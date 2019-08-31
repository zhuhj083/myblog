---
title: Spring ImportBeanDefinitionRegistrar介绍.md
date: 2019-08-22 16:32:51
tags: [spring,java]
categories: spring
---



本文需要介绍的ImportBeanDefinitionRegistrar的用法和作用跟ImportSelector类似。

唯一的不同点是ImportBeanDefinitionRegistrar的接口方法`void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry)`的返回类型是`void`，且多了一个BeanDefinitionRegistry类型的参数，它允许我们直接通过BeanDefinitionRegistry对象注册bean。



# 一.源码

ImportBeanDefinitionRegistrar源码

```java

package org.springframework.context.annotation;

import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.core.type.AnnotationMetadata;

/**
 * Interface to be implemented by types that register additional bean definitions when
 * processing @{@link Configuration} classes. Useful when operating at the bean definition
 * level (as opposed to {@code @Bean} method/instance level) is desired or necessary.
 *
 * <p>Along with {@code @Configuration} and {@link ImportSelector}, classes of this type
 * may be provided to the @{@link Import} annotation (or may also be returned from an
 * {@code ImportSelector}).
 *
 * <p>An {@link ImportBeanDefinitionRegistrar} may implement any of the following
 * {@link org.springframework.beans.factory.Aware Aware} interfaces, and their respective
 * methods will be called prior to {@link #registerBeanDefinitions}:
 * <ul>
 * <li>{@link org.springframework.context.EnvironmentAware EnvironmentAware}</li>
 * <li>{@link org.springframework.beans.factory.BeanFactoryAware BeanFactoryAware}
 * <li>{@link org.springframework.beans.factory.BeanClassLoaderAware BeanClassLoaderAware}
 * <li>{@link org.springframework.context.ResourceLoaderAware ResourceLoaderAware}
 * </ul>
 *
 * <p>See implementations and associated unit tests for usage examples.
 *
 * @author Chris Beams
 * @since 3.1
 * @see Import
 * @see ImportSelector
 * @see Configuration
 */
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
```



<!-- more -->



# 二. 示例

### 1.示例一

为了扫描并注册HelloService类型的bean，我们可以自定义如下ImportBeanDefinitionRegistrar实现类。

在实现类中可以使用ClassPathBeanDefinitionScanner进行扫描并自动注册，它是ClassPathScanningCandidateComponentProvider的子类，所以还是可以添加相同的TypeFilter，然后通过`scanner.scan(basePackages)`扫描指定的basePackage下满足条件的Class并注册它们为bean。

```java
package com.zhj.importBeanDefinitionRegistrar;

import com.zhj.service.HelloService;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.ClassPathBeanDefinitionScanner;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.filter.AssignableTypeFilter;
import org.springframework.core.type.filter.TypeFilter;

import java.util.Map;

/**
 * @author zhj on 2019-08-22.
 */
public class HelloImportBeanDefinitionRegistrar1 implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        Map<String, Object> annotationAttributes = importingClassMetadata.getAnnotationAttributes(ComponentScan.class.getName());
        String[] basePackages = (String[]) annotationAttributes.get("basePackages");

        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(registry, false);

        TypeFilter helloServiceFilter = new AssignableTypeFilter(HelloService.class);
        scanner.addIncludeFilter(helloServiceFilter);
        scanner.scan(basePackages);
    }

}
```



此时我们的`@Configuration`配置类可以进行如下定义。

```java
package com.zhj.importBeanDefinitionRegistrar;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

/**
 * @author zhj on 2019-08-22.
 */
@Configuration
@ComponentScan("com.zhj.service.impl")
@Import(HelloImportBeanDefinitionRegistrar.class)
public class HelloConfiguration1 {
}
```



### 2.自定义注解



为它定义一个特定的注解也是可以的，比如下面代码为HelloImportBeanDefinitionRegistrar定义了`@HelloScan`，其value属性和basePackages属性互为别名，用于指定需要扫描的basePackage。

```java
package com.zhj.importBeanDefinitionRegistrar;

import org.springframework.context.annotation.Import;
import org.springframework.core.annotation.AliasFor;

import java.lang.annotation.*;

/**
 * @author zhj on 2019-08-22.
 */

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(HelloImportBeanDefinitionRegistrar2.class)
public @interface HelloScan {

    @AliasFor("value")
    String[] basePackages() default {};

    @AliasFor("basePackages")
    String[] value() default {};
}

```



为了满足`@HelloScan`指定扫描的basePackage的需求，我们的HelloImportBeanDefinitionRegistrar需要改造为如下这样。

```java
package com.zhj.importBeanDefinitionRegistrar;

import com.zhj.service.HelloService;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.ClassPathBeanDefinitionScanner;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.filter.AssignableTypeFilter;
import org.springframework.core.type.filter.TypeFilter;

import java.util.Map;

/**
 * @author zhj on 2019-08-22.
 */
public class HelloImportBeanDefinitionRegistrar2 implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        Map<String, Object> annotationAttributes = importingClassMetadata.getAnnotationAttributes(HelloScan.class.getName());
        String[] basePackages = (String[]) annotationAttributes.get("basePackages");

        if (basePackages == null || basePackages.length == 0) {//HelloScan的basePackages默认为空数组
            String basePackage = null;
            try {
                basePackage = Class.forName(importingClassMetadata.getClassName()).getPackage().getName();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
            basePackages = new String[] {basePackage};
        }

        for (int i = 0; i < basePackages.length; i++) {
            System.out.println("HelloScan basePackage："+basePackages[i]);
        }

        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(registry, false);
        TypeFilter helloServiceFilter = new AssignableTypeFilter(HelloService.class);
        scanner.addIncludeFilter(helloServiceFilter);
        scanner.scan(basePackages);
    }
}
```



此时我们的HelloConfiguration可以定义为如下这样，它的效果和之前是一模一样的。

```java
package com.zhj.importBeanDefinitionRegistrar;

import org.springframework.context.annotation.Configuration;

/**
 * @author zhj on 2019-08-22.
 */
@Configuration
@HelloScan("com.zhj.service.impl")
public class HelloConfiguration2 {
}
```



### Test

```java
package com.zhj.importBeanDefinitionRegistrar;

import com.zhj.service.HelloService;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import java.util.Map;

/**
 * @author zhj on 2019-08-22.
 */
public class HelloImportSelectorTest {

    @Test
    public void test1(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(HelloConfiguration1.class);
        Map<String,HelloService> helloServices = annotationConfigApplicationContext.getBeansOfType(HelloService.class);

        for (String key : helloServices.keySet()){
            System.out.println(key);
            helloServices.get(key).doSomething();
        }
    }

    @Test
    public void test2(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(HelloConfiguration2.class);
        Map<String,HelloService> helloServices = annotationConfigApplicationContext.getBeansOfType(HelloService.class);

        for (String key : helloServices.keySet()){
            System.out.println(key);
            helloServices.get(key).doSomething();
        }
    }

}
```

