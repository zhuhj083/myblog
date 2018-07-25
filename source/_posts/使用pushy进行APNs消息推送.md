---
title: 使用pushy进行APNs消息推送
date: 2018-07-25 16:33:26
tags: [pushy,java]
categories: java
---
之前做了一个iOS消息推送的平台，主要是通过调用苹果提供的APNs接口进行消息的推送。最后采用了Pushy框架来进行推送。
# pushy简介
Pushy是用于发送APN（iOS、MacOS和Safari）推送通知的Java库，由Turo工程师编写和维护。

可以在github上获取源码和介绍：https://github.com/relayrides/pushy

官方文档：https://github.com/relayrides/pushy/wiki

# 使用pushy
## 首先引入jar
```xml
	  <!-- ====================================== pushy ====================================== -->
		<dependency>
		    <groupId>com.turo</groupId>
		    <artifactId>pushy</artifactId>
			<version>0.12.0</version>
		</dependency>
    <!-- 下面2个是监控的jar -->
		<dependency>
    		<groupId>com.turo</groupId>
    		<artifactId>pushy-dropwizard-metrics-listener</artifactId>
    		<version>0.12.0</version>
		</dependency>
		<dependency>
    		<groupId>io.dropwizard.metrics</groupId>
    		<artifactId>metrics-servlets</artifactId>
    		<version>3.1.0</version>
		</dependency>
```

## 建立连接

下面这段代码主要用户建立服务器与苹果服务器之间的链接，当我们每次执行推送消息的时候，会去检查这个链接是不是还存在，如果存在就直接使用，否则执行下面的代码建立链接。与苹果的服务器的链接长时间没有数据发送过去的话，苹果服务器会主动将它断掉。

同时也可以通过setApnsServer函数来指定是开发环境还是生产环境。


是基于Netty的，通过ApnsClientBuilder我们可以根据需要来修改ApnsClient的连接数和EventLoopGroups的线程数<br>
setConcurrentConnections：设置服务器与苹果服务器建立几个链接通道，这里是建立4个，非越多越好<br>
setEventLoopGroup：建立几个线程来处理，这里设置了4个。相当于16个线程同时处理<br>
关于连接数和EventLoopGroup线程数官不要配置EventLoopGroups的线程数超过APNs连接数。

setMetricsListener：可以设置监听器，来监听发送消息的结果
setClientCredentials：证书和密码。

```java
        if (apnsClient == null) {
            try {
                EventLoopGroup eventLoopGroup = new NioEventLoopGroup(4);
                String apnsServer = mode.equals("sandbox")?ApnsClientBuilder.DEVELOPMENT_APNS_HOST:ApnsClientBuilder.PRODUCTION_APNS_HOST;
                apnsClient = new ApnsClientBuilder().setApnsServer(apnsServer)
                		.setClientCredentials( new FileInputStream(p12_path), password )
                        .setConcurrentConnections(4)
                        .setEventLoopGroup(eventLoopGroup)
                        .setMetricsListener( metricsListener )
                        .build();
            } catch (Exception e) {
                System.err.println(_.f("[%s][ERROR]ios get pushy apns client failed!",format.print(System.currentTimeMillis())));
                e.printStackTrace();
            }
        }
```

## 发送推送
关于消息的推送，注意一定要使用异步操作，Pushy发送消息会返回一个Netty Future对象，通过它可以拿到消息发送的情况。

APNs服务器可以保证同时发送1500条消息，当超过这个限制时，Pushy会缓存消息，所以我们不必担心异步操作发送的消息过多（当我们的消息非常多，达到上亿时，我们也得做一些控制，避免缓存过大，内存不足，Pushy给出了使用Semaphore的解决方法）。
```java
  final Future<PushNotificationResponse<SimpleApnsPushNotification>> future = apnsClient.sendNotification(pushNotification);
            future.addListener(new PushNotificationResponseListener<SimpleApnsPushNotification>() {
            	@Override
            	public void operationComplete(final PushNotificationFuture<SimpleApnsPushNotification, PushNotificationResponse<SimpleApnsPushNotification>> future) throws Exception {
            	        if (future.isSuccess()) {
            	            final PushNotificationResponse<SimpleApnsPushNotification> pushNotificationResponse = future.getNow();
            	            if (pushNotificationResponse.isAccepted()) {
                                successCnt.incrementAndGet();
                            } else {
                                Date invalidTime = pushNotificationResponse.getTokenInvalidationTimestamp();
                                if (invalidTime != null) {
                                    System.out.println(_.f("[%s][ERROR]Notification deviceToken="+token+" rejected by the APNs gateway:" + pushNotificationResponse.getRejectionReason()+"\t...and the token is invalid as of " + pushNotificationResponse.getTokenInvalidationTimestamp(),format.print(System.currentTimeMillis())));
                                }else{
                                	System.out.println(_.f("[%s][ERROR]Notification deviceToken="+token+" rejected by the APNs gateway:" + pushNotificationResponse.getRejectionReason(),format.print(System.currentTimeMillis())));
                                }
                            }
            	        }else {
                            System.out.println(_.f("[%s][ERROR]send notification device token="+token+" is failed:" + future.cause().getMessage(),format.print(System.currentTimeMillis())));
                        }
                        latch.countDown();
                        semaphore.release();
                   }
            	});
```

## 完整代码
```java
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.util.concurrent.Future;

import java.io.FileInputStream;
import java.util.Date;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicLong;

import org.joda.time.format.DateTimeFormat;
import org.joda.time.format.DateTimeFormatter;

import com.alibaba.fastjson.JSONObject;
import com.turo.pushy.apns.ApnsClient;
import com.turo.pushy.apns.ApnsClientBuilder;
import com.turo.pushy.apns.PushNotificationResponse;
import com.turo.pushy.apns.metrics.dropwizard.DropwizardApnsClientMetricsListener;
import com.turo.pushy.apns.util.ApnsPayloadBuilder;
import com.turo.pushy.apns.util.SimpleApnsPushNotification;
import com.turo.pushy.apns.util.TokenUtil;
import com.turo.pushy.apns.util.concurrent.PushNotificationFuture;
import com.turo.pushy.apns.util.concurrent.PushNotificationResponseListener;

public class IOSPushClient {
    private static ApnsClient apnsClient = null;
    //APNs服务器可以保证同时发送1500条消息，当超过这个限制时，Pushy会缓存消息，所以我们不必担心异步操作发送的消息过多
    //通过Semaphore来进行流控，防止缓存过大，内存不足
    private static final Semaphore semaphore = new Semaphore(6000);
    public static  final DropwizardApnsClientMetricsListener metricsListener = new DropwizardApnsClientMetricsListener();
    static final DateTimeFormatter format = DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss.SSS");

    public void push(String mode,String p12_path ,String password, final List<String> deviceTokens, String alertTitle , String alertBody,int badgeNumber,Map<String,String> extraMap) {
        long startTime = System.currentTimeMillis();
        if (apnsClient == null) {
            try {
                EventLoopGroup eventLoopGroup = new NioEventLoopGroup(4);
                String apnsServer = mode.equals("sandbox")?ApnsClientBuilder.DEVELOPMENT_APNS_HOST:ApnsClientBuilder.PRODUCTION_APNS_HOST;
                apnsClient = new ApnsClientBuilder().setApnsServer(apnsServer)
                		.setClientCredentials( new FileInputStream(p12_path), password )
                        .setConcurrentConnections(4)
                        .setEventLoopGroup(eventLoopGroup)
                        .setMetricsListener( metricsListener )
                        .build();
            } catch (Exception e) {
                System.err.println(_.f("[%s][ERROR]ios get pushy apns client failed!",format.print(System.currentTimeMillis())));
                e.printStackTrace();
            }
        }

        long total = deviceTokens.size();
        //通过CountDownLatch来标记消息是否发送完成
        final CountDownLatch latch = new CountDownLatch(deviceTokens.size());
        final AtomicLong successCnt = new AtomicLong(0);
        long startPushTime =  System.currentTimeMillis();

        for (String deviceToken : deviceTokens) {

        	ApnsPayloadBuilder payloadBuilder = new ApnsPayloadBuilder();
            payloadBuilder.setAlertBody(alertBody);
            payloadBuilder.setAlertTitle(alertTitle);
            payloadBuilder.setBadgeNumber(badgeNumber);
            payloadBuilder.setSoundFileName("default");
            //自定义键值对，其中value是Object，可以支持多层的json字串，这个根据业务需求而定
            if(extraMap != null && extraMap.size() > 0 ){
        		for(String key : extraMap.keySet()){
        			String value = extraMap.get(key);
        			if("extraData".equals(key)){
        				payloadBuilder.addCustomProperty(key,JSONObject.parse(value));
        			}else{
        				payloadBuilder.addCustomProperty(key,value);
        			}
        		}
        	}

            String payload = payloadBuilder.buildWithDefaultMaximumLength(); //最大4k
            final String token = TokenUtil.sanitizeTokenString(deviceToken);
            SimpleApnsPushNotification pushNotification = new SimpleApnsPushNotification(token, "com.sogou.sogoureader" , payload);

            try {
                semaphore.acquire();
            } catch (InterruptedException e){
                System.out.println(_.f("[%s][ERROR]ios push get semaphore failed, deviceToken:"+deviceToken,format.print(System.currentTimeMillis())));
                e.printStackTrace();
            }

            final Future<PushNotificationResponse<SimpleApnsPushNotification>> future = apnsClient.sendNotification(pushNotification);
            future.addListener(new PushNotificationResponseListener<SimpleApnsPushNotification>() {
            	@Override
            	public void operationComplete(final PushNotificationFuture<SimpleApnsPushNotification, PushNotificationResponse<SimpleApnsPushNotification>> future) throws Exception {
            	        if (future.isSuccess()) {
            	            final PushNotificationResponse<SimpleApnsPushNotification> pushNotificationResponse = future.getNow();
            	            if (pushNotificationResponse.isAccepted()) {
                                successCnt.incrementAndGet();
                            } else {
                                Date invalidTime = pushNotificationResponse.getTokenInvalidationTimestamp();
                                if (invalidTime != null) {
                                    System.out.println("[ERROR]Notification deviceToken="+token+" rejected by the APNs gateway.");
                                }else{
                                	System.out.println("Notification deviceToken="+token+" rejected by the APNs gateway");
                                }
                            }
            	        }else {
                            System.out.println("[ERROR]send notification device token="+token+" is failed:");
                        }
                        latch.countDown();
                        semaphore.release();
                   }
            	});
        }
        try {
            latch.await(20, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            System.out.println("[ERROR]ios push latch await failed!");
            e.printStackTrace();
        }
        long endPushTime = System.currentTimeMillis();
        System.out.println("IOSPushClient pushMessage success. [total push=" + total + "][succ push=" + (successCnt.get()) + "], totalcost= " + (endPushTime - startTime) + ", pushCost=" + (endPushTime - startPushTime) , format.print(System.currentTimeMillis())));
    }
}

```
