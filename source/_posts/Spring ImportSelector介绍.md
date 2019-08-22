---
title: Spring ImportSelector介绍.md
date: 2019-08-22 16:32:51
tags: [spring,java]
categories: java
---

ImportSelecto的用途比较简单，可以根据启动的相关环境配置来决定让哪些类能够被Spring容器初始化。

Spring中，在`@Configuration`标注的Class（配置类）上可以使用`@Import`引入其它的配置类，其实它还可以引入`org.springframework.context.annotation.ImportSelector`的实现类。

# 一.ImportSelector接口说明

ImportSelector接口只定义了一个`selectImports()`，用于指定需要注册为bean的Class名称。

当在`@Configuration`标注的Class上使用`@Import`引入了一个ImportSelector实现类后，会把实现类中返回的Class名称都定义为bean。	

## 1.源码解读

下面是ImportSelector的源码

```java
package org.springframework.context.annotation;

import org.springframework.core.type.AnnotationMetadata;

/**
 * Interface to be implemented by types that determine which @{@link Configuration}
 * class(es) should be imported based on a given selection criteria, usually one or more
 * annotation attributes.
 *
 * <p>An {@link ImportSelector} may implement any of the following
 * {@link org.springframework.beans.factory.Aware Aware} interfaces, and their respective
 * methods will be called prior to {@link #selectImports}:
 * <ul>
 * <li>{@link org.springframework.context.EnvironmentAware EnvironmentAware}</li>
 * <li>{@link org.springframework.beans.factory.BeanFactoryAware BeanFactoryAware}</li>
 * <li>{@link org.springframework.beans.factory.BeanClassLoaderAware BeanClassLoaderAware}</li>
 * <li>{@link org.springframework.context.ResourceLoaderAware ResourceLoaderAware}</li>
 * </ul>
 *
 * <p>ImportSelectors are usually processed in the same way as regular {@code @Import}
 * annotations, however, it is also possible to defer selection of imports until all
 * {@code @Configuration} classes have been processed (see {@link DeferredImportSelector}
 * for details).
 *
 * @author Chris Beams
 * @since 3.1
 * @see DeferredImportSelector
 * @see Import
 * @see ImportBeanDefinitionRegistrar
 * @see Configuration
 */
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

接口文档上写明，主要作用是收集需要导入的配置类，如果该接口的实现类同时实现下列接

* EnvironmentAware
* BeanFactoryAware
* BeanClassLoaderAware
* ResourceLoaderAware

那么在调用其selectImports方法之前先调用上述接口中对应的方法，如果需要在所有的@Configuration处理完再导入时可以实现DeferredImportSelector接口。



<!-- more -->



## 2.举例说明



### 2.1 定义一个接口以及2个实现类

HelloService.java

```java
package com.zhj.service;

/**
 * @author zhj on 2019-08-22.
 */
public interface HelloService {
    void doSomething();
}
```

HelloServiceImplA.java

```java
package com.zhj.service.impl;

import com.zhj.service.HelloService;

/**
 * @author zhj on 2019-08-22.
 */
public class HelloServiceImplA implements HelloService {

    @Override
    public void doSomething() {
        System.out.println("Hello A");
    }
}
```

HelloServiceImplB.java

```java
package com.zhj.service.impl;

import com.zhj.service.HelloService;

/**
 * @author zhj on 2019-08-22.
 */
public class HelloServiceImplB implements HelloService {

    @Override
    public void doSomething() {
        System.out.println("Hello B");
    }
}
```



### 2.2 ImportSelector实现

#### 2.2.1 实现一

定义了一个ImportSelector实现类HelloImportSelector1，直接指定了需要把HelloService接口的实现类HelloServiceA和HelloServiceB定义为bean。

```java
package com.zhj.importSelector;

import com.zhj.service.impl.HelloServiceImplA;
import com.zhj.service.impl.HelloServiceImplB;
import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;

/**
 * @author zhj on 2019-08-22.
 */
public class HelloImportSelector1 implements ImportSelector {

    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{HelloServiceImplA.class.getName(), HelloServiceImplB.class.getName()};
    }
}
```

然后定义了`@Configuration`配置类HelloConfiguration1，指定了`@Import`的是HelloImportSelector1。

HelloConfiguration1.java

```java
package com.zhj.importSelector;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

/**
 * @author zhj on 2019-08-22.
 */
@Configuration
@Import(HelloImportSelector1.class)
public class HelloConfiguration1 {

}
```

这样当加载配置类HelloConfiguration1的时候会一并把HelloServiceA和HelloServiceB注册为Spring bean。可以

测试：

```java
    @Test
    public void test1(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(HelloConfiguration1.class);
        Map<String,HelloService> helloServices = annotationConfigApplicationContext.getBeansOfType(HelloService.class);

        for (String key : helloServices.keySet()){
            System.out.println(key);
            helloServices.get(key).doSomething();
        }
    }
```

输出：

```sh
com.zhj.service.impl.HelloServiceImplA
Hello A
com.zhj.service.impl.HelloServiceImplB
Hello B
```



#### 2.2.2 直接在HelloConfiguration中定义bean或者import。

看到这里可能你会觉得其实它也没什么用，因为整一个ImportSelector实现类那么麻烦，还不如直接在HelloConfiguration中定义bean或者import。

在不引入ImportSelector的情况下，下面的两种方式都可以达到相同的效果。

```java
//直接在HelloConfiguration中import
@Configuration
@Import({HelloServiceImplA.class, HelloServiceImplB.class})
public class HelloConfigurationDirect {

}

//直接在HelloConfiguration中定义bean
@Configuration
public class HelloConfigurationDefineBean {

    @Bean
    public HelloServiceImplA helloServiceImplA(){
        return new HelloServiceImplA();
    }

    @Bean
    public HelloServiceImplB helloServiceImplB(){
        return new HelloServiceImplB();
    }

}
```

测试：

```java
    @Test
    public void testDirect(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(HelloConfigurationDirect.class);
        Map<String,HelloService> helloServices = annotationConfigApplicationContext.getBeansOfType(HelloService.class);

        for (String key : helloServices.keySet()){
            System.out.println(key);
            helloServices.get(key).doSomething();
        }
    }

    @Test
    public void testDefineBean(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(HelloConfigurationDefineBean.class);
        Map<String,HelloService> helloServices = annotationConfigApplicationContext.getBeansOfType(HelloService.class);

        for (String key : helloServices.keySet()){
            System.out.println(key);
            helloServices.get(key).doSomething();
        }
    }
```





如果直接是固定的bean定义，那完全可以用上面的方式代替。

但如果需要动态的带有逻辑性的定义bean，则使用ImportSelector还是很有用处的。因为在它的`selectImports()`你可以实现各种获取bean Class的逻辑，通过其参数`AnnotationMetadata importingClassMetadata`可以获取到`@Import`标注的Class的各种信息，包括其Class名称，实现的接口名称、父类名称、添加的其它注解等信息，通过这些额外的信息可以辅助我们选择需要定义为Spring bean的Class名称。

#### 2.2.3 @ComponentScan扫描方法

现假设我们在HelloConfiguration上使用了`@ComponentScan`进行bean定义扫描我们期望HelloImportSelector也可以扫描`@ComponentScan`指定的Package下HelloService实现类并把它们定义为bean，则HelloImportSelector和HelloConfiguration可以改为如下这样：

HelloConfigurationWithScan.java

```java
package com.zhj.importSelector;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

/**
 * @author zhj on 2019-08-22.
 */
@Configuration
@ComponentScan("com.zhj.service.impl")
@Import(HelloImportSelector4.class)
public class HelloConfigurationWithScan {
}
```

HelloImportSelectorWithScan.java

```java
package com.zhj.importSelector;

import com.zhj.service.HelloService;
import org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.filter.AssignableTypeFilter;
import org.springframework.core.type.filter.TypeFilter;

import java.util.HashSet;
import java.util.Map;
import java.util.Set;

/**
 * @author zhj on 2019-08-22.
 */
public class HelloImportSelectorWithScan implements ImportSelector {

    /**
     * 可以看到在HelloImportSelector的实现中获取了HelloConfiguration类上标注的@ComponentScan的basePackages属性值
     * 并使用ClassPathScanningCandidateComponentProvider进行了扫描。
     *
     * 可能有的时候你不希望依赖于配置类上的@ComponentScan，而期望直接扫描配置类所在的包。
     * 此时可以通过importingClassMetadata.getClassName()获取配置类的Class名称，进而获取其package名称。
     *
     * @param importingClassMetadata
     * @return
     */
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {

        Map<String, Object> annotationAttributes = importingClassMetadata.getAnnotationAttributes(ComponentScan.class.getName());
        String[] basePackages = (String[]) annotationAttributes.get("basePackages");
        for (int i = 0; i < basePackages.length; i++) {
            System.out.println("basePackages_"+i+"："+basePackages[i]);
        }

        ClassPathScanningCandidateComponentProvider scanner = new ClassPathScanningCandidateComponentProvider(false);
        TypeFilter helloServiceFilter = new AssignableTypeFilter(HelloService.class);
        scanner.addIncludeFilter(helloServiceFilter);

        Set<String> classes = new HashSet<>();
        for (String basePackage : basePackages) {
            scanner.findCandidateComponents(basePackage).forEach(beanDefinition -> classes.add(beanDefinition.getBeanClassName()));
        }

        //set转换为array
        return classes.toArray(new String[classes.size()]);
    }

}
```

测试：

```java
    @Test
    public void testWithScan(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(HelloConfigurationWithScan.class);
        Map<String,HelloService> helloServices = annotationConfigApplicationContext.getBeansOfType(HelloService.class);

        for (String key : helloServices.keySet()){
            System.out.println(key);
            helloServices.get(key).doSomething();
        }
    }
```



#### 2.2.4 期望直接扫描配置类所在的包

可以看到在HelloImportSelector的实现中获取了HelloConfiguration类上标注的`@ComponentScan`的basePackages属性值，并使用ClassPathScanningCandidateComponentProvider进行了扫描。可能有的时候你不希望依赖于配置类上的`@ComponentScan`，而期望直接扫描配置类所在的包。

此时可以通过`importingClassMetadata.getClassName()`获取配置类的Class名称，进而获取其package名称。

```java
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        String packageName = null;
        try {
            packageName = Class.forName(importingClassMetadata.getClassName()).getPackage().getName();
            System.out.println("packageName="+packageName);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        String[] basePackages = new String[] {packageName};

        for (int i = 0; i < basePackages.length; i++) {
            System.out.println("4_basePackages_"+i+"："+basePackages[i]);
        }

        ClassPathScanningCandidateComponentProvider scanner = new ClassPathScanningCandidateComponentProvider(false);
        TypeFilter helloServiceFilter = new AssignableTypeFilter(HelloService.class);
        scanner.addIncludeFilter(helloServiceFilter);

        Set<String> classes = new HashSet<>();
        for (String basePackage : basePackages) {
            scanner.findCandidateComponents(basePackage).forEach(beanDefinition -> classes.add(beanDefinition.getBeanClassName()));
        }
        return classes.toArray(new String[classes.size()]);
    }
```



#### 2.2.5 更通用一点的做法

更通用一点的做法可能你还是期望扫描的package跟`@Configuration`上的`@ComponentScan`的basePackages保持一致或者在没有指定`@ComponentScan`时扫描配置类所在的package。

@ComponentScan`的basePackages如果没有指定，默认是把配置类当前所在的package当做basePackage。所以为了满足这些需求，我们的HelloImportSelector可以定义为如下这样:

```java
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        String[] basePackages = null;
        if (importingClassMetadata.hasAnnotation(ComponentScan.class.getName())) {
            Map<String, Object> annotationAttributes = importingClassMetadata.getAnnotationAttributes(ComponentScan.class.getName());
            basePackages = (String[]) annotationAttributes.get("basePackages");
        }
        if (basePackages == null || basePackages.length == 0) {//ComponentScan的basePackages默认为空数组
            String basePackage = null;
            try {
                basePackage = Class.forName(importingClassMetadata.getClassName()).getPackage().getName();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
            basePackages = new String[] {basePackage};
        }
        ClassPathScanningCandidateComponentProvider scanner = new ClassPathScanningCandidateComponentProvider(false);
        TypeFilter helloServiceFilter = new AssignableTypeFilter(HelloService.class);
        scanner.addIncludeFilter(helloServiceFilter);
        Set<String> classes = new HashSet<>();
        for (String basePackage : basePackages) {
            scanner.findCandidateComponents(basePackage).forEach(beanDefinition -> classes.add(beanDefinition.getBeanClassName()));
        }
        return classes.toArray(new String[classes.size()]);
    }
```



### 2.3 为ImportSelector定义特定的注解

当我们觉得在`@Configuration`配置类上使用`@Import(HelloImportSelector.class)`太麻烦，或者是需要在ImportSelector实现类中使用一些特定的配置时就可以考虑为ImportSelector实现类定义一个特定的注解，在该注解上使用`@Import(HelloImportSelector.class)`。

如下针对上面的HelloImportSelector定义了一个`@HelloServiceScan`注解，用于扫描HelloService实现类。

```java
package com.zhj.importSelector.annotation;

import com.zhj.importSelector.HelloImportSelector1;
import org.springframework.context.annotation.Import;
import org.springframework.core.annotation.AliasFor;

import java.lang.annotation.*;

/**
 * @author zhj on 2019-08-22.
 */


@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(HelloImportSelector1.class)
public @interface HelloServiceScan {

    @AliasFor("value")
    String[] basePackages() default {};

    @AliasFor("basePackages")
    String[] value() default {};
}
```

此时，我们的HelloConfiguration类可以改为如下这样，效果跟之前的一样的。

```java
package com.zhj.importSelector;

import com.zhj.importSelector.annotation.HelloServiceScan;
import org.springframework.context.annotation.Configuration;

/**
 * @author zhj on 2019-08-22.
 */
@Configuration
@HelloServiceScan("com.zhj.service.impl")
public class HelloScanHelloConfiguration {
}
```



这样HelloImportSelector在进行bean扫描时可以通过`@HelloServiceScan`的basePackages属性获取需要扫描的basePackage。

```java
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        Map<String, Object> annotationAttributes = importingClassMetadata.getAnnotationAttributes(HelloServiceScan.class.getName());
        String[] basePackages = (String[]) annotationAttributes.get("basePackages");

        //HelloServiceScan的basePackages默认为空数组
        if (basePackages == null || basePackages.length == 0) {
            String basePackage = null;
            try {
                basePackage = Class.forName(importingClassMetadata.getClassName()).getPackage().getName();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
            basePackages = new String[] {basePackage};
        }


        ClassPathScanningCandidateComponentProvider scanner = new ClassPathScanningCandidateComponentProvider(false);
        TypeFilter helloServiceFilter = new AssignableTypeFilter(HelloService.class);
        scanner.addIncludeFilter(helloServiceFilter);
        Set<String> classes = new HashSet<>();
        for (String basePackage : basePackages) {
            scanner.findCandidateComponents(basePackage).forEach(beanDefinition -> classes.add(beanDefinition.getBeanClassName()));
        }

        return classes.toArray(new String[classes.size()]);
    }
```





参考博文：https://www.iteye.com/blog/elim-2428994