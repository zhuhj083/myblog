---
title: Dubbo源码分析2-EnableDubbo注解分析
date: 2019-08-30 15:12:14
tags: [spring,dubbo,rpc]
categories: dubbo
---

# 一. @EnableDubbo分析

首先看源码：

```java
/**
 * 发现EnableDubbo注解其实是EnableDubboConfig跟DubboComponentScan标签的合并
 */
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@EnableDubboConfig
@DubboComponentScan
public @interface EnableDubbo {
    /**
     * 用于指定扫描那些包含@Service注解的包(声明后面提到Service注解都是Dubbo的Service注解)
     */
    @AliasFor(annotation = DubboComponentScan.class, attribute = "basePackages")
    String[] scanBasePackages() default {};

    /**
     * 用于指定扫描那些包含@service注解的类
     */
    @AliasFor(annotation = DubboComponentScan.class, attribute = "basePackageClasses")
    Class<?>[] scanBasePackageClasses() default {};


    /**
     * 表示是否绑定到多个bean。默认是true
     * 举例：作用是按照不同的配置文件格式进行选择。如：dubbo.application跟dubbo.applications之间的区别
     */
    @AliasFor(annotation = EnableDubboConfig.class, attribute = "multiple")
    boolean multipleConfig() default true;
}
```

我们发现EnableDubbo注解其实是EnableDubboConfig跟DubboComponentScan标签的合并，是一种简略写法。

所以我们再分别去分析EnableDubboConfig注解和DubboComponentScan注解