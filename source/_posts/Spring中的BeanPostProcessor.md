---
title: Spring中的BeanPostProcessor简介
date: 2018-08-06 20:26:50
tags: [spring,java]
categories: java
---
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
