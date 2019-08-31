---
title: Spring中的BeanPostProcessor和BeanFactoryPostProcessor
date: 2018-08-06 20:26:50
tags: [spring,java]
categories: spring
---
BeanPostProcessor和BeanFactoryPostProcessor是Spring初始化Bean时对外暴露的2个接口，两个接口看起来很相似，但是作用和使用场景去不同。

参考博文：https://blog.csdn.net/caihaijiang/article/details/35552859



# BeanFactoryPostProcessor

## 接口的定义

```java
public interface BeanFactoryPostProcessor {

	//实现该接口，可以在spring的bean创建之前，修改bean的定义属性。
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

实现该接口，可以在spring的bean创建之前，修改bean的定义属性。

也就是说，Spring允许BeanFactoryPostProcessor在容器实例化任何其它bean之前读取配置元数据，并可以根据需要进行修改。例如可以把bean的scope从singleton改为prototype，也可以把property的值给修改掉。



可以同时配置多个BeanFactoryPostProcessor，并通过设置`order`属性来控制各个BeanFactoryPostProcessor的执行次序。

> 注意：BeanFactoryPostProcessor是在spring容器加载了bean的定义文件之后，在bean实例化之前执行的。接口方法的入参是ConfigurrableListableBeanFactory，使用该参数，可以获取到相关bean的定义信息



# BeanPostProcessor简介

BeanPostProcessor是Spring IOC容器给我们提供的一个扩展接口。接口声明如下：
```java
public interface BeanPostProcessor {
    //bean初始化方法调用前被调用
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
    //bean初始化方法调用后被调用
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```
BeanPostProcessor接口有2个回调方法，当一个BeanPostProcessor的实现类注册到Spring IOC容器后，对于该Spring IOC 容器所创造的每一个bean实例的初始化方法（如afterPropertiesSet和自定义init-method方法）调用前，将会调用BeanPostProcessor中的postProcessBeforeInitialization方法；而在bean实例初始化方法调用完成后，则会调用BeanPostProcessor中的postProcessAfterInitialization方法。

整个调用过程简单示意如下：
> Spring IOC容器实例化Bean

> 调用BeanPostProcessor的postProcessBeforeInitialization方法

> 调用bean实例的初始化方法

> 调用BeanPostProcessor的postProcessAfterInitialization方法

可以看到，Spring容器通过BeanPostProcessor给了我们一个机会对Spring管理的bean进行再加工。比如：我们可以修改bean的属性，可以给bean生成一个动态代理实例等等。

一些Spring AOP的底层处理也是通过实现BeanPostProcessor来执行代理包装逻辑的。

<!-- more -->

BeanPostProcessor，可以在spring容器实例化bean之后，在执行bean的初始化方法前后，添加一些自己的处理逻辑。这里说的初始化方法，指的是下面两种：

* bean实现了InitializingBean接口，对应的方法为afterPropertiesSet

* 在bean定义的时候，通过init-method设置的方法



**注意：BeanPostProcessor是在spring容器加载了bean的定义文件并且实例化bean之后执行的。BeanPostProcessor的执行顺序是在BeanFactoryPostProcessor之后。**



spring中，有内置的一些BeanPostProcessor实现类，例如：

* org.springframework.context.annotation.CommonAnnotationBeanPostProcessor：支持@Resource注解的注入

* org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor：支持@Required注解的注入

* org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor：支持@Autowired注解的注入

* org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor：支持@PersistenceUnit和@PersistenceContext注解的注入

* org.springframework.context.support.ApplicationContextAwareProcessor：用来为bean注入ApplicationContext等容器对象

  > 这些注解类的BeanPostProcessor，在spring配置文件中，可以通过这样的配置
  >
  >  <context:component-scan base-package="*.*" /> ，自动进行注册。
  >
  > （spring通过ComponentScanBeanDefinitionParser类来解析该标签）
  > 



# 示例

1.定义一个Bean

```java
package com.zhj.postprocessor.bean;


import org.springframework.beans.factory.InitializingBean;

public class MyJavaBean implements InitializingBean {

    private String desc;
    private String remark;

    public MyJavaBean() {
        System.out.println("--------MyJavaBean的构造函数被执行啦--------");
    }

    public String getDesc() {
        return desc;
    }
    public void setDesc(String desc) {
        System.out.println("---------调用setDesc方法，desc="+desc);
        this.desc = desc;
    }
    public String getRemark() {
        return remark;
    }
    public void setRemark(String remark) {
        System.out.println("--------调用setRemark方法，remark="+remark);
        this.remark = remark;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("--------调用afterPropertiesSet方法----------");
        this.desc = "在初始化方法中修改之后的描述信息";
    }

    public void initMethod() {
        System.out.println("--------调用initMethod方法------------");
    }

    @Override
    public String toString() {
        return "MyJavaBean{" +
                "desc='" + desc + '\'' +
                ", remark='" + remark + '\'' +
                '}';
    }
}
```



2.自定义一个BeanFactoryPostProcessor

```java
package com.zhj.postprocessor;

import org.springframework.beans.BeansException;
import org.springframework.beans.MutablePropertyValues;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;

/**
 * @author zhj on 2019-08-23.
 */
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

        System.out.println();
        System.out.println();

        System.out.println("---------------调用MyBeanFactoryPostProcessor#postProcessBeanFactory---------------");
        BeanDefinition bd = beanFactory.getBeanDefinition("myJavaBean");
        MutablePropertyValues propertyValues = bd.getPropertyValues();

        System.out.println("属性值：" + propertyValues.toString());

        if (propertyValues.contains("remark")) {
            propertyValues.addPropertyValue("remark", "把备注信息修改一下");
        }
        bd.setScope(BeanDefinition.SCOPE_PROTOTYPE);

        System.out.println("---------------end---------------");
        System.out.println();
        System.out.println();
    }
}

```



3.自定义一个BeanPostProcessor

```java
package com.zhj.postprocessor;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

/**
 * @author zhj on 2019-08-23.
 */

public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("BeanPostProcessor，对象" + beanName + "调用初始化方法之前的数据： " + bean.toString());
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("BeanPostProcessor，对象" + beanName + "调用初始化方法之后的数据：" + bean.toString());
        return bean;
    }
}
```



4.Spring 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">

    <bean id="myJavaBean" class="com.zhj.postprocessor.bean.MyJavaBean" init-method="initMethod">
        <property name="desc" value="原始的描述信息" />
        <property name="remark" value="原始的备注信息" />
    </bean>

    <bean id="myBeanFactoryPostProcessor" class="com.zhj.postprocessor.MyBeanFactoryPostProcessor"/>

    <bean id="myBeanPostProcessor" class="com.zhj.postprocessor.MyBeanPostProcessor"/>

</beans>
```



5.测试类

```java
package com.zhj.postprocessor;

import com.zhj.postprocessor.bean.MyJavaBean;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author zhj on 2019-08-23.
 */
public class PostProcessorTest {

    @Test
    public void test() {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-processor.xml");
        MyJavaBean bean = (MyJavaBean) context.getBean("myJavaBean");

        System.out.println("-------------------下面输出结果----------------");
        System.out.println("描述：" + bean.getDesc());
        System.out.println("备注：" + bean.getRemark());

    }
}
```



6.执行结果

```sh
---------------调用MyBeanFactoryPostProcessor#postProcessBeanFactory---------------
属性值：PropertyValues: length=2; bean property 'desc'; bean property 'remark'
---------------end---------------


--------MyJavaBean的构造函数被执行啦--------
---------调用setDesc方法，desc=原始的描述信息
--------调用setRemark方法，remark=把备注信息修改一下
BeanPostProcessor，对象myJavaBean调用初始化方法之前的数据： MyJavaBean{desc='原始的描述信息', remark='把备注信息修改一下'}
--------调用afterPropertiesSet方法----------
--------调用initMethod方法------------
BeanPostProcessor，对象myJavaBean调用初始化方法之后的数据：MyJavaBean{desc='在初始化方法中修改之后的描述信息', remark='把备注信息修改一下'}
-------------------下面输出结果----------------
描述：在初始化方法中修改之后的描述信息
备注：把备注信息修改一下

Process finished with exit code 0
```



从上面的结果可以看出，BeanFactoryPostProcessor在bean实例化之前执行，之后实例化bean（调用构造函数，并调用set方法注入属性值），然后在调用两个初始化方法前后，执行了BeanPostProcessor。初始化方法的执行顺序是，先执行afterPropertiesSet，再执行init-method。