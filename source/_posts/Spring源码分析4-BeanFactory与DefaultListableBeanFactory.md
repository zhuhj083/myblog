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



# 2.DefaultListableBeanFactory 介绍

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

* DefaultSingletonBeanRegistry：对接口SingletonBeanRegistry各函数的实

* FactoryBeanRegisterSupport：DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。



* BeanDefinitionRegistry：定义对BeanDefinition的各种增删改查操作



* ListableBeanFactory：根据各种条件获取bean的配置清单（可以预加载bean definition）

* HierarchicalBeanFactory：继承BeanFactory，也就是在BeanFactory定义的功能的基础上增加对parentFactory的支持

* ConfigurableBeanFactory：提供配置Factory的各种方法

  

* AbstractBeanFactory：综合FactoryBeanRegisterSupport和ConfigurableBeanFactory的功能

* AutowireCapableBeanFactory：提供创建bean、自动注入、初始化以及应用bean的 后处理器

  

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

```java
public interface ListableBeanFactory extends BeanFactory {
	
  boolean containsBeanDefinition(String beanName);

	int getBeanDefinitionCount();

	String[] getBeanDefinitionNames();

	String[] getBeanNamesForType(ResolvableType type);

	String[] getBeanNamesForType(ResolvableType type, boolean includeNonSingletons, boolean allowEagerInit);

	String[] getBeanNamesForType(@Nullable Class<?> type);

	String[] getBeanNamesForType(@Nullable Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);

	<T> Map<String, T> getBeansOfType(@Nullable Class<T> type) throws BeansException;

	/**
	 *
	 * includeNonSingletons：是否包含非单例bean
	 * allowEagerInit：是否要初始化
	 */
	<T> Map<String, T> getBeansOfType(@Nullable Class<T> type, boolean includeNonSingletons, boolean allowEagerInit)
			throws BeansException;
  
	String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType);

	Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException;

	@Nullable
	<A extends Annotation> A findAnnotationOnBean(String beanName, Class<A> annotationType)
			throws NoSuchBeanDefinitionException;

}
```



### 2.2.4 HierarchicalBeanFactory接口

HierarchicalBeanFactory：继承BeanFactory，也就是在BeanFactory定义的功能的基础上增加对**parentFactory**的支持

```java
public interface HierarchicalBeanFactory extends BeanFactory {

	/**
	 * Return the parent bean factory, or {@code null} if there is none.
	 */
	@Nullable
	BeanFactory getParentBeanFactory();

	/**
	 * 返回本地BeanFactory是否包含给定名称的bean
	 * 不在父BeanFactory中找
	 */
	boolean containsLocalBean(String name);

}
```



### 2.2.5 ConfigurableBeanFactory接口

继承自HierarchicalBeanFactory接口，提供配置Factory的各种方法。

* 设置ClassLoader，BeanExpressionResolver，ConversionService
* 添加PropertyEditorRegistrar，PropertyEditor，StringValueResolver，BeanPostProcessor



### 2.2.6 FactoryBeanRegistrySupport类

DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。

使用`Map<String, Object> factoryBeanObjectCache = new ConcurrentHashMap<>(16);`缓存，存储由FactoryBean创建的bean（key是 FactoryBean的name，value是创建的bean对象）。

```java
package org.springframework.beans.factory.support;

import java.security.AccessControlContext;
import java.security.AccessController;
import java.security.PrivilegedAction;
import java.security.PrivilegedActionException;
import java.security.PrivilegedExceptionAction;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanCreationException;
import org.springframework.beans.factory.BeanCurrentlyInCreationException;
import org.springframework.beans.factory.FactoryBean;
import org.springframework.beans.factory.FactoryBeanNotInitializedException;
import org.springframework.lang.Nullable;

/**
 * 在DefaultSingletonBeanRegistry基础上增加了对 FactoryBean的特殊处理功能。
 */
public abstract class FactoryBeanRegistrySupport extends DefaultSingletonBeanRegistry {

	//缓存，存储的是由FactoryBean创建的bean（key是 FactoryBean的name，value是创建的bean对象）
	private final Map<String, Object> factoryBeanObjectCache = new ConcurrentHashMap<>(16);


	/**
	 *  返回FactoryBean创建的bean实例的类型
	 */
	@Nullable
	protected Class<?> getTypeForFactoryBean(final FactoryBean<?> factoryBean) {
		try {
			if (System.getSecurityManager() != null) {
				return AccessController.doPrivileged(
						(PrivilegedAction<Class<?>>) factoryBean::getObjectType , getAccessControlContext());
			}
			else {
				return factoryBean.getObjectType();
			}
		}
		catch (Throwable ex) {
			// Thrown from the FactoryBean's getObjectType implementation.
			logger.info("FactoryBean threw exception from getObjectType, despite（尽管） the contract saying " +
					"that it should return null if the type of its object cannot be determined yet", ex);
			return null;
		}
	}

	/**
	 * 从FactoryBeans缓存中 根据factory bean name获取 Bean对象
	 */
	@Nullable
	protected Object getCachedObjectForFactoryBean(String beanName) {
		return this.factoryBeanObjectCache.get(beanName);
	}

	/**
	 * shouldPostProcess：bean是否需要进行后处理
	 */
	protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
		//检查FactoryBean 是否作为单例形式注册到父类DefaultSingletonBeanRegistry的缓存中
		if (factory.isSingleton() && containsSingleton(beanName)) {
			synchronized (getSingletonMutex()) {
				Object object = this.factoryBeanObjectCache.get(beanName);
				if (object == null) {

					//获取对象
					object = doGetObjectFromFactoryBean(factory, beanName);

					//只有在上面的getObject（）调用期间没有进行 后处理 和 存储
					//（例如，由于自定义getBean调用触发的循环引用处理）
					Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
					if (alreadyThere != null) {
						object = alreadyThere;
					}
					else {
						//不存在于factoryBeanObjectCache
						if (shouldPostProcess) {
							// 需要后处理

							// 检查singletonsCurrentlyInCreation缓存，看是否是正在创建的bean
							if (isSingletonCurrentlyInCreation(beanName)) {
								// 正在创建的bean
								// Temporarily（暂时）return non-post-processed object, not storing it yet..
								return object;
							}

							//创建bean之前回调，添加到singletonsCurrentlyInCreation缓存
							beforeSingletonCreation(beanName);

							try {
								object = postProcessObjectFromFactoryBean(object, beanName);
							}
							catch (Throwable ex) {
								throw new BeanCreationException(beanName,
										"Post-processing of FactoryBean's singleton object failed", ex);
							}
							finally {
								//创建bean之后回调，从singletonsCurrentlyInCreation缓存移除
								afterSingletonCreation(beanName);
							}
						}
						if (containsSingleton(beanName)) {
							//添加到factoryBeanObjectCache缓存
							this.factoryBeanObjectCache.put(beanName, object);
						}
					}
				}
				return object;
			}
		}
		else {
			Object object = doGetObjectFromFactoryBean(factory, beanName);
			if (shouldPostProcess) {
				try {
					object = postProcessObjectFromFactoryBean(object, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);
				}
			}
			return object;
		}
	}

	/**
	 * 调用FactoryBean对象的getObject方法，创建一个bean（beanName是创建的bean的name）
	 */
	private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
			throws BeanCreationException {

		Object object;

		//调用FactoryBean的getObject()方法，创建一个bean对象
		try {
			if (System.getSecurityManager() != null) {
				AccessControlContext acc = getAccessControlContext();
				try {
					object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) factory::getObject, acc);
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
				object = factory.getObject();
			}
		}
		catch (FactoryBeanNotInitializedException ex) {
			throw new BeanCurrentlyInCreationException(beanName, ex.toString());
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "FactoryBean threw exception on object creation", ex);
		}

		// Do not accept a null value for a FactoryBean that's not fully
		// initialized yet: Many FactoryBeans just return null then.
		if (object == null) {
			if (isSingletonCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(
						beanName, "FactoryBean which is currently in creation returned null from getObject");
			}
			object = new NullBean();
		}
		return object;
	}

	/**
	 * 对从FactoryBean获取的给定对象进行后处理。 生成的对象将暴露给bean引用。
	 * 留给子类实现
	 */
	protected Object postProcessObjectFromFactoryBean(Object object, String beanName) throws BeansException {
		return object;
	}

	protected FactoryBean<?> getFactoryBean(String beanName, Object beanInstance) throws BeansException {
		if (!(beanInstance instanceof FactoryBean)) {
			throw new BeanCreationException(beanName,
					"Bean instance of type [" + beanInstance.getClass() + "] is not a FactoryBean");
		}
		return (FactoryBean<?>) beanInstance;
	}

	@Override
	protected void removeSingleton(String beanName) {
		synchronized (getSingletonMutex()) {
			super.removeSingleton(beanName);
			this.factoryBeanObjectCache.remove(beanName);
		}
	}

	@Override
	protected void clearSingletonCache() {
		synchronized (getSingletonMutex()) {
			super.clearSingletonCache();
			this.factoryBeanObjectCache.clear();
		}
	}

	protected AccessControlContext getAccessControlContext() {
		return AccessController.getContext();
	}

}

```



### 2.2.7 AutowireCapableBeanFactory接口

继承自BeanFactory，提供了创建bean、自动注入、初始化以及 应用bean的后处理器

`public interface AutowireCapableBeanFactory extends BeanFactory `



### 2.2.8 AbstractBeanFactory 虚类

继承自FactoryBeanRegistrySupport类，实现了ConfigurableBeanFactory接口，所以综合了FactoryBeanRegisterSupport和ConfigurableBeanFactory的功能





### 2.2.9 ConfigurableListableBeanFactory接口

ConfigurableListableBeanFactory接口继承自ListableBeanFactory接口、AutowireCapableBeanFactory接口和ConfigurableBeanFactory接口。

Bean配置清单，指定忽略类型及接口等

```
public interface ConfigurableListableBeanFactory
      extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory 
```



### 2.2.10 AbstractAutowireCapableBeanFactory类

综合AbstractBeanFactory，对AutowireCapableBeanFactory接口进行实现

```
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory
```



# 3 DefaultListableBeanFactory源码分析

待完成	



# 4. XmlBeanFactory类

XmlBeanFactory继承自DefaultListableBeanFactory，XmlBeanFactory中使用了自定义的XML读取器XmlBeanDefinitionReader，实现了个性化的BeanDefinitionReader。

```java
public class XmlBeanFactory extends DefaultListableBeanFactory {
  
	private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);


	/**
	 * Create a new XmlBeanFactory with the given resource,
	 * which must be parsable using DOM.
	 * @param resource the XML resource to load bean definitions from
	 * @throws BeansException in case of loading or parsing errors
	 */
	public XmlBeanFactory(Resource resource) throws BeansException {
		this(resource, null);
	}

	/**
	 * Create a new XmlBeanFactory with the given input stream,
	 * which must be parsable using DOM.
	 * @param resource the XML resource to load bean definitions from
	 * @param parentBeanFactory parent bean factory
	 * @throws BeansException in case of loading or parsing errors
	 */
	public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
		super(parentBeanFactory);
		this.reader.loadBeanDefinitions(resource);
	}
}
```



















