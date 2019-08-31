---
title: Dubbo源码分析3-EnableDubboConfig注解分析
date: 2019-08-30 15:07:54
tags:[spring,dubbo,rpc]
categories: dubbo

---

# 一. @EnableDubboConfig分析

## 1. EnableDubboConfig注解类

首先来看EnableDubboConfig的源码

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Import(DubboConfigConfigurationRegistrar.class)
public @interface EnableDubboConfig {
    /**
     * It indicates whether binding to multiple Spring Beans.
     *
     * @return the default value is <code>false</code>
     * @revised 2.5.9
     */
    boolean multiple() default true;

}
```

源码说明：

我们可以看到使用@Import注解引入了DubboConfigConfigurationRegistrar类，这个类实现了实现了ImportBeanDefinitionRegistrar接口，重写了接口的registerBeanDefinitions方法，这个方法返回类型是void，有一个BeanDefinitionRegistry类型的参数，允许我们直接通过BeanDefinitionRegistry注册bean。

<!-- more -->

## 2. DubboConfigConfigurationRegistrar类

DubboConfigConfigurationRegistrar的源码

```java
public class DubboConfigConfigurationRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        // 从importingClassMetadata对象（这个对象中包含启动类上的标签和其他信息）中获取EnableDubboConfig标签中的属性，
        // 并保存在map中然后封装到AnnotationAttributes中
        // AnnotationAttributes 是LinkedHashMap的子类
        AnnotationAttributes attributes = AnnotationAttributes.fromMap(
                importingClassMetadata.getAnnotationAttributes(EnableDubboConfig.class.getName()));

        //获取EnableDubboConfig标签中的multiple属性
        boolean multiple = attributes.getBoolean("multiple");

        // Single Config Bindings
        AnnotatedBeanDefinitionRegistryUtils.registerBeans(registry, DubboConfigConfiguration.Single.class);

        if (multiple) { // Since 2.6.6 https://github.com/apache/dubbo/issues/3193
            AnnotatedBeanDefinitionRegistryUtils.registerBeans(registry, DubboConfigConfiguration.Multiple.class);
        }
    }

}
```

这里根据配置的multiple的值（默认为false），来注册dubboConfigConfiguration.Single 和dubboConfigConfiguration.Multiple Bean。我们看到是通过`AnnotatedBeanDefinitionRegistryUtils`类的`registerBeans()`方法来注册bean的。



## 3. AnnotatedBeanDefinitionRegistryUtils工具类

AnnotatedBeanDefinitionRegistryUtils源码

```java
public abstract class AnnotatedBeanDefinitionRegistryUtils {

  // ... 其他代码略，并删除了一切无关代码
  
    public static void registerBeans(BeanDefinitionRegistry registry, Class<?>... annotatedClasses) {
      
        if (ObjectUtils.isEmpty(annotatedClasses)) {
            return;
        }

        // 删除已注册的所有 注解类
        // Remove all annotated-classes that have been registered
        Iterator<Class<?>> iterator = new ArrayList<>(asList(annotatedClasses)).iterator();

        while (iterator.hasNext()) {
            Class<?> annotatedClass = iterator.next();
            if (isPresentBean(registry, annotatedClass)) {
                iterator.remove();
            }
        }

        // 获取AnnotatedBeanDefinitionReader对象，这个对面里面包含了所有的 注释后处理器,
        // 比如处理有Configuration标签的ConfigurationClassPostProcessor类
        AnnotatedBeanDefinitionReader reader = new AnnotatedBeanDefinitionReader(registry);
        reader.register(annotatedClasses);
    }
}
```



## 4. DubboConfigConfiguration.Single 内部类

Single类源码

```java
    /**
     * Single Dubbo {@link AbstractConfig Config} Bean Binding
     */
    @EnableDubboConfigBindings({
            @EnableDubboConfigBinding(prefix = "dubbo.application", type = ApplicationConfig.class),
            @EnableDubboConfigBinding(prefix = "dubbo.module", type = ModuleConfig.class),
            @EnableDubboConfigBinding(prefix = "dubbo.registry", type = RegistryConfig.class),
            @EnableDubboConfigBinding(prefix = "dubbo.protocol", type = ProtocolConfig.class),
            @EnableDubboConfigBinding(prefix = "dubbo.monitor", type = MonitorConfig.class),
            @EnableDubboConfigBinding(prefix = "dubbo.provider", type = ProviderConfig.class),
            @EnableDubboConfigBinding(prefix = "dubbo.consumer", type = ConsumerConfig.class),
            @EnableDubboConfigBinding(prefix = "dubbo.config-center", type = ConfigCenterBean.class),
            @EnableDubboConfigBinding(prefix = "dubbo.metadata-report", type = MetadataReportConfig.class),
            @EnableDubboConfigBinding(prefix = "dubbo.metrics", type = MetricsConfig.class)
    })
    public static class Single {

    }
```

这类类上有注解@EnableDubboConfigBindings，以及@EnableDubboConfigBinging数组



## 5. EnableDubboConfigBindings注解类

EnableDubboConfigBindings源码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(DubboConfigBindingsRegistrar.class)
public @interface EnableDubboConfigBindings {

    /**
     * The value of {@link EnableDubboConfigBindings}
     *
     * @return non-null
     */
    EnableDubboConfigBinding[] value();

}
```

可以看到EnableDubboConfigBindings包含了 EnableDubboConfigBinding数组。并且用@Import 导入了DubboConfigBindingsRegistrar类，



## 6. DubboConfigBindingsRegistrar类

```java
public class DubboConfigBindingsRegistrar implements ImportBeanDefinitionRegistrar, EnvironmentAware {
  
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        //从importingClassMetadata对象获取EnableDubboConfigBindings注解的内容
        AnnotationAttributes attributes = AnnotationAttributes.fromMap(               importingClassMetadata.getAnnotationAttributes(EnableDubboConfigBindings.class.getName()));

        //从获取的attributes集合中找到value属性，此时value是一个EnableDubboConfigBinding注解数组
        AnnotationAttributes[] annotationAttributes = attributes.getAnnotationArray("value");

        //创建DubboConfigBindingRegistrar类，作用是注册EnableDubboConfigBinding注解中的Bean
        DubboConfigBindingRegistrar registrar = new DubboConfigBindingRegistrar();
        registrar.setEnvironment(environment);

        for (AnnotationAttributes element : annotationAttributes) {
            //遍历EnableDubboConfigBinding注解，并注册其中配置的各个config类。
            registrar.registerBeanDefinitions(element, registry);

        }
    }
}
```

DubboConfigBindingsRegistrar#registerBeanDefinitions()的主要作用就是拿到Single类上注解@EnableDubboConfigBindings所配置的@EnableDubboConfigBinding数组，再创建DubboConfigBindingRegistrar类，分别注册每个@EnableDubboConfigBinding配置的ConfigBean



##  7. EnableDubboConfigBinding注解类

首先看EnableDubboConfigBinding源码，有2个属性prefix和type类。根据prefix 和type可以去properties配置文件中读取配置，在根据配置来注册ConfigBean

```java
@Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(EnableDubboConfigBindings.class)
@Import(DubboConfigBindingRegistrar.class)
public @interface EnableDubboConfigBinding {

    String prefix();

    Class<? extends AbstractConfig> type();

    boolean multiple() default false;

}
```



## 8. DubboConfigBindingRegistrar类

最后是DubboConfigBindingRegistrar类。进入DubboConfigBindingRegistrar类中，对registerBeanDefinitions方法进行解析

```java
@Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {


        AnnotationAttributes attributes = AnnotationAttributes.fromMap(
                importingClassMetadata.getAnnotationAttributes(EnableDubboConfigBinding.class.getName()));

        registerBeanDefinitions(attributes, registry);

    }

    protected void registerBeanDefinitions(AnnotationAttributes attributes, BeanDefinitionRegistry registry) {

        // 获取EnableDubboConfigBinding注解中的prefix属性的占位符
        // 例如prefix=dubbo.application则这里最后prefix值为dubbo.application
        String prefix = environment.resolvePlaceholders(attributes.getString("prefix"));

        // 获取EnableDubboConfigBinding注解中的type表示的对象的Class对象
        Class<? extends AbstractConfig> configClass = attributes.getClass("type");

        // 获取EnableDubboConfigBinding注解中的multiple属性
        boolean multiple = attributes.getBoolean("multiple");

        //注册Dubbo的ConfigBean
        registerDubboConfigBeans(prefix, configClass, multiple, registry);

    }

    private void registerDubboConfigBeans(String prefix,
                                          Class<? extends AbstractConfig> configClass,
                                          boolean multiple,
                                          BeanDefinitionRegistry registry) {

        // 获取环境变量中的所有属性，并找到前缀为prefix的所有的属性集合；
        // 例如 dubbo.application.name=dubbo-service，则properties中的值为 name：dubbo-service
        Map<String, Object> properties = getSubProperties(environment.getPropertySources(), prefix);

        if (CollectionUtils.isEmpty(properties)) {
            if (log.isDebugEnabled()) {
                log.debug("There is no property for binding to dubbo config class [" + configClass.getName()
                        + "] within prefix [" + prefix + "]");
            }
            return;
        }

        //
        Set<String> beanNames = multiple ? resolveMultipleBeanNames(properties) :
                Collections.singleton(resolveSingleBeanName(properties, configClass, registry));

        //遍历beanNames
        for (String beanName : beanNames) {

            //使用beanName作为创建的configClass类型Bean的Name
            registerDubboConfigBean(beanName, configClass, registry);

            // 注册DubboConfigBindingBeanPostProcessor，
            // 这里使用BeanDefinitionBuilder来创建Bean，并将prefix跟beanName加到Bean的构造器参数列表中，
            // 这里会注册多个这个Bean只是构造参数不同
            registerDubboConfigBindingBeanPostProcessor(prefix, beanName, multiple, registry);

        }

        registerDubboConfigBeanCustomizers(registry);
    }
```



通过简化为一个@EnableDubboConfig注解到配置类上，就完成了Dubbo主要的ConfigBean的Spring注入。