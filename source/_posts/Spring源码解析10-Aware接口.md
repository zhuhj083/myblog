---
title: Spring源码解析10-Aware接口
date: 2019-09-02 18:26:49
tags: [java,spring]
categories: spring
---



Spring中有很多接口继承自Aware接口的接口。

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/spring_aware_hierarchy.png)



aware 这个单词的意思是感知、察觉。所以这些接口从字面意思上理解就是能感知到所有**Aware**前面的内容的含义。

例如：

* BeanNameAware，实现了Aware接口，可以让该bean感知到自身的BeanName（对应Spring容器的`BeanId`属性）属性。

* 实现了ApplicationContextAware接口的类，能够获取到ApplicationContext

* 实现了BeanFactoryAware接口的类，能够获取到BeanFactory对象。



# 1. Aware源码

```java
package org.springframework.beans.factory;

/**
 * A marker superinterface indicating（说明） that a bean is eligible to（有资格） be notified by the
 * Spring container of a particular(特定的) framework object through a callback-style method.
 * The actual method signature（方法签名） is determined（确定） by individual subinterfaces but should
 * typically consist of（包括） just one void-returning method that accepts a single argument.
 */
public interface Aware {

}
```



一个超级接口标记。说明实现了这个接口的bean，可以被Spring 容器通过一个回调函数来获取到一个特定的对象。

这个回调函数由子接口来定义。通常是返回值是void，有一个参数的方法。



# 2.子接口举例 

BeanNameAware接口定义：

这个回调函数在普通bean**属性填充之后**但在**初始化回调(afterPropertiesSet或自定义init方法)之前**调用。

```java
/**
 * 需要在BeanFactory感知其bean name的beans 需要实现这个接口。
 *
 * Interface to be implemented by beans that want to be aware of their
 * bean name in a bean factory. Note that it is not usually recommended
 * that an object depends on its bean name, as this represents a potentially
 * brittle dependence on external configuration, as well as a possibly
 * unnecessary dependence on a Spring API.
 *
 * <p>For a list of all bean lifecycle methods, see the
 * {@link BeanFactory BeanFactory javadocs}.
 *
 * @author Juergen Hoeller
 * @author Chris Beams
 * @since 01.11.2003
 * @see BeanClassLoaderAware
 * @see BeanFactoryAware
 * @see InitializingBean
 */
public interface BeanNameAware extends Aware {

	/**
	 * Set the name of the bean in the bean factory that created this bean.
	 * <p>Invoked after population of normal bean properties but before an
	 * init callback such as {@link InitializingBean#afterPropertiesSet()}
	 * or a custom init-method.
	 * @param name the name of the bean in the factory.
	 * Note that this name is the actual bean name used in the factory, which may
	 * differ from the originally specified name: in particular for inner bean
	 * names, the actual bean name might have been made unique through appending
	 * "#..." suffixes. Use the {@link BeanFactoryUtils#originalBeanName(String)}
	 * method to extract the original bean name (without suffix), if desired.
	 */
	void setBeanName(String name);

}
```



ApplicationContextAware接口定义：

```java
package org.springframework.context;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.Aware;

/**
 * Interface to be implemented by any object that wishes to be notified
 * of the {@link ApplicationContext} that it runs in.
 *
 * <p>Implementing this interface makes sense for example when an object
 * requires access to a set of collaborating beans. Note that configuration
 * via bean references is preferable to implementing this interface just
 * for bean lookup purposes.
 *
 * <p>This interface can also be implemented if an object needs access to file
 * resources, i.e. wants to call {@code getResource}, wants to publish
 * an application event, or requires access to the MessageSource. However,
 * it is preferable to implement the more specific {@link ResourceLoaderAware},
 * {@link ApplicationEventPublisherAware} or {@link MessageSourceAware} interface
 * in such a specific scenario.
 *
 * <p>Note that file resource dependencies can also be exposed as bean properties
 * of type {@link org.springframework.core.io.Resource}, populated via Strings
 * with automatic type conversion by the bean factory. This removes the need
 * for implementing any callback interface just for the purpose of accessing
 * a specific file resource.
 *
 * <p>{@link org.springframework.context.support.ApplicationObjectSupport} is a
 * convenience base class for application objects, implementing this interface.
 *
 * <p>For a list of all bean lifecycle methods, see the
 * {@link org.springframework.beans.factory.BeanFactory BeanFactory javadocs}.
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @author Chris Beams
 * @see ResourceLoaderAware
 * @see ApplicationEventPublisherAware
 * @see MessageSourceAware
 * @see org.springframework.context.support.ApplicationObjectSupport
 * @see org.springframework.beans.factory.BeanFactoryAware
 */
public interface ApplicationContextAware extends Aware {

	/**
	 * Set the ApplicationContext that this object runs in.
	 * Normally this call will be used to initialize the object.
	 * <p>Invoked after population of normal bean properties but before an init callback such
	 * as {@link org.springframework.beans.factory.InitializingBean#afterPropertiesSet()}
	 * or a custom init-method. Invoked after {@link ResourceLoaderAware#setResourceLoader},
	 * {@link ApplicationEventPublisherAware#setApplicationEventPublisher} and
	 * {@link MessageSourceAware}, if applicable.
	 * @param applicationContext the ApplicationContext object to be used by this object
	 * @throws ApplicationContextException in case of context initialization errors
	 * @throws BeansException if thrown by application context methods
	 * @see org.springframework.beans.factory.BeanInitializationException
	 */
	void setApplicationContext(ApplicationContext applicationContext) throws BeansException;

}
```

