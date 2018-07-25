---
title: kafka 生产者和消费者
date: 2018-07-25 16:08:46
tags: [kafka,java]
categories: java
---
# maven配置
```xml
  <dependency>
		   <groupId>org.apache.kafka</groupId>
		   <artifactId>kafka_2.11</artifactId>
		   <version>0.11.0.2</version>
	</dependency>
	<dependency>
		   <groupId>org.apache.kafka</groupId>
		   <artifactId>kafka-clients</artifactId>
		   <version>0.11.0.2</version>
	</dependency>
```

# KafkaProducer
KafkaProducer初始化
```java
public static KafkaProducer<String, String> kafkaProducer = null ;
static {
		try {
	        Properties props = new Properties();
	        props.put("bootstrap.servers",QueueConstants.BROKER_LIST);
	        props.put("acks", "all");
	        props.put("retries", 1);
	        props.put("batch.size", 16384);
	        props.put("linger.ms", 1);
	        props.put("buffer.memory", 33554432);
	        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        kafkaProducer = new KafkaProducer<String,String>(props);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```
KafkaProducer send消息(异步方式)
```java
QueueConstants.kafkaProducer.send(new ProducerRecord<String, String>(topic, msg), new Callback() {
			@Override
			public void onCompletion(RecordMetadata metadata, Exception exception) {
				System.out.println("MessageProducer sended,metadata:"+metadata.toString());
			}
	});
```

# KafkaConsumer
```java
    private  KafkaConsumer<String, String> createConsumer(){
    	Properties props = new Properties();
        props.put("bootstrap.servers", QueueConstants.BROKER_LIST );
        props.put("group.id", groupId);	//必须要使用别的组名称， 如果生产者和消费者都在同一组，则不能访问同一组内的topic数据
        props.put("enable.auto.commit", false);
        props.put("auto.commit.interval.ms", "1000");//自动确认offset的时间间隔
        props.put("session.timeout.ms", "30000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    	return new KafkaConsumer<>(props);
     }
```

消费消息
```java
	@SuppressWarnings("unchecked")
	@Override
    public void run() {
        KafkaConsumer<String, String> consumer =createConsumer();
        consumer.subscribe(Arrays.asList(topic));
        try {
	        while (true) {
	        	 ConsumerRecords<String, String> records = consumer.poll(1000);
	        	 for (ConsumerRecord<String, String> record : records) {
	        		 consumer.commitAsync(new OffsetCommitCallback() {
							@Override
							public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets,
									Exception exception) {
									if (null == exception) {
										  //表示偏移量成功提交
				              System.out.println("IosPushySender commit succ");
									} else {
				              //表示提交偏移量发生了异常，根据业务进行相关处理
				              System.out.println("IosPushySender commit exception ,"+exception.toString());
				           }
							}
						});
	        		 if(_.nonEmpty(record.value())){
		                 String message  = new String(record.value().getBytes(), "UTF-8");
		                 //...
                     //消费消息
	        		 }
	            }
	        }
		}catch (Exception e) {
			e.printStackTrace();
		}finally{
			consumer.close();
		}
  }
```
