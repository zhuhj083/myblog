---
title: Spring源码分析4-BeanFactory与DefaultListableBeanFactory
date: 2019-08-31 16:31:19
tags: [java,spring]
categories: spring
---



Spring Beans模块是Spring中的一个重要模块，所有的应用都需要使用，功能主要有

* 访问配置文件
* 创建和管理bean
* 进行IoC/DI相关操作

# 1. BeanFactory接口

BeanFactory顾名思义就是生产bean的工厂。

首先来看BeanFactory定义：

```java
public interface BeanFactory {

	/**
	 * FactoryBean这种在注入bean的时候会在beanName前添加一个"&"修饰符
	 * 例如，如果名为myJndiObject的bean是FactoryBean，
	 * 则获取＆myJndiObject将返回这个工厂，而不是工厂返回的实例。
	 */
	String FACTORY_BEAN_PREFIX = "&";
  
	Object getBean(String name) throws BeansException;

	<T> T getBean(String name, Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;
  
	<T> T getBean(Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);

	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
  
	boolean containsBean(String name);

  boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
  
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
  
	String[] getAliases(String name);

}
```



# 2.DefaultListableBeanFactory

DefaultListableBeanFactory间接实现了BeanFactory，是整个bean加载的核心部分，是Spring注册以及加载bean的默认实现。

DefaultListableBeanFactory继承了AbstractAutowireCapableBeanFactory，并实现了ConfigurableListableBeanFactory，以及BeanDefinitionRegister接口。

<!-- more -->

## 2.1首先看下类结构图和类继承关系

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/DefaultListableBeanFactory_hierarchy.png)



![UML](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/DefaultListableBeanFactory_UML.png)



各个接口和类的作用：

* BeanFactory：定义获取bean及bean的各种属性

  

* AliasRegistry接口：定义对alias的简单增删改查等操作

* SimpleAliasRegistry：主要使用`map`作为alias的缓存，并对接口AliasRegister进行实现

* SingletonBeanRegistry：定义对单例的注册及获取

* DefaultSingletonBeanRegistry：对接口SingletonBeanRegistry各函数的实现



* BeanDefinitionRegistry：定义对BeanDefinition的各种增删改查操作



* ListableBeanFactory：根据各种条件获取bean的配置清单（可以预加载bean definition）
* HierarchicalBeanFactory：继承BeanFactory，也就是在BeanFactory定义的功能的基础上增加对parentFactory的支持
* FactoryBeanRegisterSupport：DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。
* ConfigurableBeanFactory：提供配置Factory的各种方法
* AbstractBeanFactory：综合FactoryBeanRegisterSupport和ConfigurableBeanFactory的功能
* AutowireCapableBeanFactory：提供创建bean、自动注入、初始化以及应用bean的后处理器
* AbstractAutowireCapableBeanFactory：综合AbstractBeanFactory并AutowireCapableBeanFactory接口进行实现
* ConfigurableListableBeanFactory：Bean配置清单，指定忽略类型及接口等
* DefaultListableBeanFactory：综合上面所有功能，主要对Bean注册后的处理。



## 2.2 主要父接口和父类详细介绍

### 2.2.1 AliasRegiser接口、SimpleAliasRegistry类、SingletonBeanRegistry接口、DefaultSingletonBeanRegistry类

见上一篇博文



### 2.2.2 BeanDefinitionRegistry接口

定义对BeanDefinition的各种增删改查操作,并且继承了AliasRegistry接口

```java
public interface BeanDefinitionRegistry extends AliasRegistry {

	/**
	 * 注册一个新的BeanDefinition
	 */
	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	/**
	 * 根据给定的名字，移除注册的BeanDefinition
	 */
	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 根据给定名字，查找注册的BeanDefinition
	 */
	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 检查registry中是否包含一个给定名字的BeanDefinition
	 */
	boolean containsBeanDefinition(String beanName);

	/**
	 * 返回registry中注册的所有BeanDefinition的名字数组
	 */
	String[] getBeanDefinitionNames();

	/**
	 * 返回registry中注册的所有BeanDefinition的个数
	 */
	int getBeanDefinitionCount();

	/**
	 * 判断指定的名字是否已经在registry使用了
	 */
	boolean isBeanNameInUse(String beanName);

}
```



### 2.2.3 ListableBeanFactory接口
**ListableBeanFactory**是**BeanFactory**接口的扩展接口，它可以枚举所有的bean实例，而不是客户端通过名称一个一个的查询得出所有的实例。要预加载所有的bean定义的BeanFactory可以实现这个接口。

