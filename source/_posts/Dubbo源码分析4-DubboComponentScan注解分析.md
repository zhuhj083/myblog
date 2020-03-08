---
title: Dubbo源码分析4-DubboComponentScan注解分析
date: 2019-09-01 02:59:53
tags: [rpc,dubbo]
categories: dubbo
---

# 一. DubboComponentScan注解类分析

## 1. 首先看DubboComponentScan源码

DubboComponentScan注解类

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(DubboComponentScanRegistrar.class)
public @interface DubboComponentScan {

    String[] value() default {};

    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

}
```

看到@Import了DubboComponentScanRegistrar类。这个类也是实现了Spring的ImportBeanDefinitionRegistrar接口，直接看registerBeanDefinitions()方法

```java
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        // 根据importingClassMetadata找到贴有DubboComponentScan注解的类的信息，
        // 从中获取需要扫描的包
        Set<String> packagesToScan = getPackagesToScan(importingClassMetadata);

        // 注册ServiceAnnotationBeanPostProcessor，会传入packagesToScan
        registerServiceAnnotationBeanPostProcessor(packagesToScan, registry);

        // 注册ReferenceAnnotationBeanPostProcessor
        registerReferenceAnnotationBeanPostProcessor(registry);

    }
```

上面的主要的工作就是对需要扫描的包或者类的处理，统一处理为包路径。然后注册两个Bean(ServiceAnnotationBeanPostProcessor和ReferenceAnnotationBeanPostProcessor)

<!-- more -->

## 2. ServiceAnnotationBeanPostProcessor分析

ServiceAnnotationBeanPostProcessor实现了BeanDefinitionRegistryPostProcessor。

```java
public class ServiceAnnotationBeanPostProcessor implements BeanDefinitionRegistryPostProcessor, EnvironmentAware,
        ResourceLoaderAware, BeanClassLoaderAware {
          //...代码略
}
```

ServiceAnnotationBeanPostProcessor的类图

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/ServiceAnnotationBeanPostProcessor.png)

BeanDefinitionRegistryPostProcessor的作用是：允许在正常的BeanFactoryPostProcessor检测开始之前注册更多的自定义bean。

Spring容器中所有的Bean注册之后，回调postProcessBeanDefinitionRegistry方法，开始扫描@Service注解并注入容器。

看ServiceAnnotationBeanPostProcessor#postProcessBeanDefinitionRegistry方法。

```java
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {

        //// 获取扫描包，扫描包可以是多个，所以这里使用了Set集合
        Set<String> resolvedPackagesToScan = resolvePackagesToScan(packagesToScan);

        // 判断需要扫描的包是否为空
        if (!CollectionUtils.isEmpty(resolvedPackagesToScan)) {
            // 不为空则调用方法进行解析
            registerServiceBeans(resolvedPackagesToScan, registry);
        } else {
            if (logger.isWarnEnabled()) {
                logger.warn("packagesToScan is empty , ServiceBean registry will be ignored!");
            }
        }
    }
```



继续看ServiceAnnotationBeanPostProcessor#registerServiceBeans方法

```java
    /**
     * Registers Beans whose classes was annotated {@link Service}
     *
     * @param packagesToScan The base packages to scan
     * @param registry       {@link BeanDefinitionRegistry}
     */
    private void registerServiceBeans(Set<String> packagesToScan, BeanDefinitionRegistry registry) {

        //定义扫描对象，该类继承了ClassPathBeanDefinitionScanner类
        DubboClassPathBeanDefinitionScanner scanner =
                new DubboClassPathBeanDefinitionScanner(registry, environment, resourceLoader);

        // beanName解析器
        BeanNameGenerator beanNameGenerator = resolveBeanNameGenerator(registry);

        scanner.setBeanNameGenerator(beanNameGenerator);

        //这行代码很重要，添加了一个注解过滤器，用来过滤出来写了@Service注解的对象
        scanner.addIncludeFilter(new AnnotationTypeFilter(Service.class));

        /**
         * 添加旧版Dubbo @Service的兼容性
         *
         * Add the compatibility for legacy Dubbo's @Service
         *
         * The issue : https://github.com/apache/dubbo/issues/4330
         * @since 2.7.3
         */
        scanner.addIncludeFilter(new AnnotationTypeFilter(com.alibaba.dubbo.config.annotation.Service.class));

        //扫描正式开始，遍历包
        for (String packageToScan : packagesToScan) {

            // Registers @Service Bean first
            scanner.scan(packageToScan);

            // 开始查找添加了@Service注解的类
            // Finds all BeanDefinitionHolders of @Service whether @ComponentScan scans or not.
            Set<BeanDefinitionHolder> beanDefinitionHolders =
                    findServiceBeanDefinitionHolders(scanner, packageToScan, registry, beanNameGenerator);

            //找到了则不为空
            if (!CollectionUtils.isEmpty(beanDefinitionHolders)) {

                for (BeanDefinitionHolder beanDefinitionHolder : beanDefinitionHolders) {
                    // 注册serviceBean定义，并做数据绑定和解析
                    registerServiceBean(beanDefinitionHolder, registry, scanner);
                }

                if (logger.isInfoEnabled()) {
                    logger.info(beanDefinitionHolders.size() + " annotated Dubbo's @Service Components { " +
                            beanDefinitionHolders +
                            " } were scanned under package[" + packageToScan + "]");
                }

            } else {

                if (logger.isWarnEnabled()) {
                    logger.warn("No Spring Bean annotating Dubbo's @Service was found under package["
                            + packageToScan + "]");
                }

            }

        }

    }
```

上面的代码，主要作用就是通过扫描过滤器，扫描包中添加了@Service注解的类。最后得到BeanDefinitionHolder对象，调用registerServiceBean来注册ServiceBean



看registerServiceBean方法

```java
    /**
     * Registers {@link ServiceBean} from new annotated {@link Service} {@link BeanDefinition}
     *
     * @param beanDefinitionHolder
     * @param registry
     * @param scanner
     * @see ServiceBean
     * @see BeanDefinition
     */
    private void registerServiceBean(BeanDefinitionHolder beanDefinitionHolder, BeanDefinitionRegistry registry,
                                     DubboClassPathBeanDefinitionScanner scanner) {

        // 有@Service注解的，实现了某个接口的 服务类对象
        Class<?> beanClass = resolveClass(beanDefinitionHolder);

        // 找到@service注解
        Annotation service = findServiceAnnotation(beanClass);

        /**
         * The {@link AnnotationAttributes} of @Service annotation
         */
        AnnotationAttributes serviceAnnotationAttributes = getAnnotationAttributes(service, false, false);

        // 服务类实现的接口对象
        Class<?> interfaceClass = resolveServiceInterfaceClass(serviceAnnotationAttributes, beanClass);

        // 得到bean的名称
        String annotatedServiceBeanName = beanDefinitionHolder.getBeanName();

        // 构建ServiceBean对象的BeanDefinition,通过Service注解对象，以及接口服务的实现类生成ServiceBean
        AbstractBeanDefinition serviceBeanDefinition =
                buildServiceBeanDefinition(service, serviceAnnotationAttributes, interfaceClass, annotatedServiceBeanName);

        // 构建ServuceBean的名称
        // ServiceBean Bean name
        String beanName = generateServiceBeanName(serviceAnnotationAttributes, interfaceClass);

        // 校验Bean是否重复
        if (scanner.checkCandidate(beanName, serviceBeanDefinition)) { // check duplicated candidate bean
            // 调用BeanDefinitionRegistry注册。
            registry.registerBeanDefinition(beanName, serviceBeanDefinition);

            if (logger.isInfoEnabled()) {
                logger.info("The BeanDefinition[" + serviceBeanDefinition +
                        "] of ServiceBean has been registered with name : " + beanName);
            }

        } else {

            if (logger.isWarnEnabled()) {
                logger.warn("The Duplicated BeanDefinition[" + serviceBeanDefinition +
                        "] of ServiceBean[ bean name : " + beanName +
                        "] was be found , Did @DubboComponentScan scan to same package in many times?");
            }

        }

    }
```



最后看buildServiceBeanDefinition方法

```java
    private AbstractBeanDefinition buildServiceBeanDefinition(Annotation serviceAnnotation,
                                                              AnnotationAttributes serviceAnnotationAttributes,
                                                              Class<?> interfaceClass,
                                                              String annotatedServiceBeanName) {

        // 生成ServiceBean对象BeanDefinitionBuilder
        BeanDefinitionBuilder builder = rootBeanDefinition(ServiceBean.class);

        // 获取beanDefinition
        AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();

        // 属性集合
        MutablePropertyValues propertyValues = beanDefinition.getPropertyValues();

        // 忽略的属性,指的是AnnotationPropertyValuesAdapter中，不把以下属性放到PropertyValues中去
        String[] ignoreAttributeNames = of("provider", "monitor", "application", "module", "registry", "protocol",
                "interface", "interfaceName", "parameters");

        propertyValues.addPropertyValues(new AnnotationPropertyValuesAdapter(serviceAnnotation, environment, ignoreAttributeNames));

        //// 设置ref属性
        // References "ref" property to annotated-@Service Bean
        addPropertyReference(builder, "ref", annotatedServiceBeanName);

        // 设置服务的接口
        // Set interface
        builder.addPropertyValue("interface", interfaceClass.getName());
        // Convert parameters into map
        builder.addPropertyValue("parameters", convertParameters(serviceAnnotationAttributes.getStringArray("parameters")));

        // Add methods parameters
        List<MethodConfig> methodConfigs = convertMethodConfigs(serviceAnnotationAttributes.get("methods"));
        if (!methodConfigs.isEmpty()) {
            builder.addPropertyValue("methods", methodConfigs);
        }

        /**
         * 获取注解中的provider属性
         * Add {@link org.apache.dubbo.config.ProviderConfig} Bean reference
         */
        String providerConfigBeanName = serviceAnnotationAttributes.getString("provider");
        if (StringUtils.hasText(providerConfigBeanName)) {
            addPropertyReference(builder, "provider", providerConfigBeanName);
        }

        /**
         * 获取注解中的monitor属性
         * Add {@link org.apache.dubbo.config.MonitorConfig} Bean reference
         */
        String monitorConfigBeanName = serviceAnnotationAttributes.getString("monitor");
        if (StringUtils.hasText(monitorConfigBeanName)) {
            addPropertyReference(builder, "monitor", monitorConfigBeanName);
        }

        /**
         * 获取注解中的application属性
         * Add {@link org.apache.dubbo.config.ApplicationConfig} Bean reference
         */
        String applicationConfigBeanName = serviceAnnotationAttributes.getString("application");
        if (StringUtils.hasText(applicationConfigBeanName)) {
            addPropertyReference(builder, "application", applicationConfigBeanName);
        }

        /**
         * Add {@link org.apache.dubbo.config.ModuleConfig} Bean reference
         */
        String moduleConfigBeanName = serviceAnnotationAttributes.getString("module");
        if (StringUtils.hasText(moduleConfigBeanName)) {
            addPropertyReference(builder, "module", moduleConfigBeanName);
        }


        /**
         * 获取注解中的注册中心属性
         * Add {@link org.apache.dubbo.config.RegistryConfig} Bean reference
         */
        String[] registryConfigBeanNames = serviceAnnotationAttributes.getStringArray("registry");

        List<RuntimeBeanReference> registryRuntimeBeanReferences = toRuntimeBeanReferences(registryConfigBeanNames);

        if (!registryRuntimeBeanReferences.isEmpty()) {
            builder.addPropertyValue("registries", registryRuntimeBeanReferences);
        }

        /**
         * 获取注解中的协议属性
         * Add {@link org.apache.dubbo.config.ProtocolConfig} Bean reference
         */
        String[] protocolConfigBeanNames = serviceAnnotationAttributes.getStringArray("protocol");

        List<RuntimeBeanReference> protocolRuntimeBeanReferences = toRuntimeBeanReferences(protocolConfigBeanNames);

        if (!protocolRuntimeBeanReferences.isEmpty()) {
            builder.addPropertyValue("protocols", protocolRuntimeBeanReferences);
        }

        return builder.getBeanDefinition();

    }
```



最终dubbo解析出了ServiceBean对象的beandefinition放入了spring的容器。

ServiceBean对于每个暴露的服务来说很重要，每个暴露的服务都拥有一个ServiceBean，这个类里面包含了服务发布，服务下线等操作，算是一个很核心的类。



## 3. ReferenceAnnotationBeanPostProcessor分析

Dubbo中做`属性注入`是通过ReferenceAnnotationBeanPostProcessor处理的，主要做以下几件事情

* 获取类中标注的@Reference注解的字段和方法
* 反射设置字段或方法对应的引用



### 3.1 类的继承关系

```java
public class ReferenceAnnotationBeanPostProcessor extends AnnotationInjectedBeanPostProcessor implements
        ApplicationContextAware, ApplicationListener {
}

public abstract class AnnotationInjectedBeanPostProcessor extends
        InstantiationAwareBeanPostProcessorAdapter implements MergedBeanDefinitionPostProcessor, PriorityOrdered,
        BeanFactoryAware, BeanClassLoaderAware, EnvironmentAware, DisposableBean {
}
```



ReferenceAnnotationBeanPostProcessor是一个BeanPostProcessor，该类实现了`InstantiationAwareBeanPostProcessor` ，所以Spring的Bean中初始化前会触发postProcessPropertyValues方法，该方法允许我们做进一步处理，比如增加属性和属性值修改等。从而达到解析@Reference并实现依赖注入（@Autowired注入一样的原理）。



>InstantiationAwareBeanPostProcessor：实例化Bean后置处理器（继承BeanPostProcessor），有以下三个方法
>
>- postProcessBeforeInstantiation：在实例化目标对象之前执行，可以自定义实例化逻辑，如返回一个代理对象等。
>- postProcessAfterInitialization： Bean实例化完毕后执行的**后处理操作**，所有初始化逻辑、装配逻辑之前执行，如果返回false将阻止其他的InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation的执行。
>- postProcessPropertyValues：完成其他定制的一些**依赖注入**和**依赖检查**等，如：
>  - AutowiredAnnotationBeanPostProcessor执行@Autowired注解注入
>  - CommonAnnotationBeanPostProcessor执行@Resource等注解的注入
>  - RequiredAnnotationBeanPostProcessor执行@ Required注解的检查等等。



###  3.2  源码分析

首先看AnnotationInjectedBeanPostProcessor#postProcessPropertyValues方法

```java
    @Override
    public PropertyValues postProcessPropertyValues(
            PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeanCreationException {

        // InjectionMetadata metadata是注入元数据 ，包含了
        //      目标Bean的Class对象
        //      和注入元素（InjectionElement）的集合
        // 查找Bean所有标注了指定注解（@Reference）的字段和方法
        InjectionMetadata metadata = findInjectionMetadata(beanName, bean.getClass(), pvs);
        try {
            // 通过反射来给bean设置值
            metadata.inject(bean, beanName, pvs);
        } catch (BeanCreationException ex) {
            throw ex;
        } catch (Throwable ex) {
            throw new BeanCreationException(beanName, "Injection of @" + getAnnotationType().getSimpleName()
                    + " dependencies is failed", ex);
        }
        return pvs;
    }
```



注入元数据InjectionMetadata类

```java
public class InjectionMetadata {

  //目标Bean的Class对象
	private final Class<?> targetClass;

  //注入元素（InjectionElement）集合
  //注入元素可能是属性也可能是方法
	private final Collection<InjectedElement> injectedElements;

	public InjectionMetadata(Class<?> targetClass, Collection<InjectedElement> elements) {
		this.targetClass = targetClass;
		this.injectedElements = elements;
	}

  //...其他代码略
}
```



#### 3.2.1 收集注入元数据

通过findInjectionMetadata找到@Reference，并解析得到 元数据对象。

AnnotationInjectedBeanPostProcessor#findInjectionMetadata()

```java
    private InjectionMetadata findInjectionMetadata(String beanName, Class<?> clazz, PropertyValues pvs) {

        // 通过类名作为缓存的key
        // Fall back to class name as cache key, for backwards compatibility with custom callers.
        String cacheKey = (StringUtils.hasLength(beanName) ? beanName : clazz.getName());

        // 从缓存中的injectionMetadataCache根据类名获取 AnnotatedInjectionMetadata元数据
        // Quick check on the concurrent map first, with minimal locking.
        AnnotationInjectedBeanPostProcessor.AnnotatedInjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
        // 判断metadata是否需要刷新（metadata为空以及class对象不等于ReferenceInjectionMetadata时，则需要进行刷新）
        if (InjectionMetadata.needsRefresh(metadata, clazz)) {
            synchronized (this.injectionMetadataCache) {
                metadata = this.injectionMetadataCache.get(cacheKey);
                // 双重检查机制
                if (InjectionMetadata.needsRefresh(metadata, clazz)) {
                    if (metadata != null) {
                        metadata.clear(pvs);
                    }
                    try {

                        // 构建InjectionMetadata元数据
                        metadata = buildAnnotatedMetadata(clazz);

                        // 加入到缓存
                        this.injectionMetadataCache.put(cacheKey, metadata);
                    } catch (NoClassDefFoundError err) {
                        throw new IllegalStateException("Failed to introspect object class [" + clazz.getName() +
                                "] for annotation metadata: could not find class that it depends on", err);
                    }
                }
            }
        }
        return metadata;
    }
```



构建InjectionMetadata元数据，

AnnotationInjectedBeanPostProcessor#buildAnnotatedMetadata()方法源码：

```java
    private AnnotationInjectedBeanPostProcessor.AnnotatedInjectionMetadata buildAnnotatedMetadata(final Class<?> beanClass) {

        // 获取属性上的指定注解（@Reference注解）
        Collection<AnnotationInjectedBeanPostProcessor.AnnotatedFieldElement> fieldElements = findFieldAnnotationMetadata(beanClass);

        // 获取方法上的指定注解（@Reference注解）
        Collection<AnnotationInjectedBeanPostProcessor.AnnotatedMethodElement> methodElements = findAnnotatedMethodMetadata(beanClass);

        return new AnnotationInjectedBeanPostProcessor.AnnotatedInjectionMetadata(beanClass, fieldElements, methodElements);

    }
```



获取属性上的@Reference注解详解

AnnotationInjectedBeanPostProcessor#findFieldAnnotationMetadata()方法源码：

```java
    private List<AnnotationInjectedBeanPostProcessor.AnnotatedFieldElement> findFieldAnnotationMetadata(final Class<?> beanClass) {

              final List<AnnotationInjectedBeanPostProcessor.AnnotatedFieldElement> elements = new LinkedList<AnnotationInjectedBeanPostProcessor.AnnotatedFieldElement>();

        ReflectionUtils.doWithFields(beanClass, field -> {
            // getAnnotationTypes 拿到需要寻找的注解类（@Reference注解）
            for (Class<? extends Annotation> annotationType : getAnnotationTypes()) {

                //合并和解析占位符后，获取AnnotationAttributes注释属性
                AnnotationAttributes attributes = getMergedAttributes(field, annotationType, getEnvironment(), true);

                if (attributes != null) {

                    // 不能是static方法
                    if (Modifier.isStatic(field.getModifiers())) {
                        if (logger.isWarnEnabled()) {
                            logger.warn("@" + annotationType.getName() + " is not supported on static fields: " + field);
                        }
                        return;
                    }

                    elements.add(new AnnotatedFieldElement(field, attributes));
                }
            }
        });

        return elements;

    }
```




#### 3.2.2 对字段、方法进行反射绑定
元数据收集好了，接下来就是调用`metadata.inject(bean, beanName, pvs);`这个方法了。

```java
	public void inject(Object target, String beanName, PropertyValues pvs) throws Throwable {
		// 获取metadata中的InjectedElement
    Collection<InjectedElement> elementsToIterate =
				(this.checkedElements != null ? this.checkedElements : this.injectedElements);
		if (!elementsToIterate.isEmpty()) {
			boolean debug = logger.isDebugEnabled();
      // 循环设值
			for (InjectedElement element : elementsToIterate) {
        //触发方法或字段值的注入
				element.inject(target, beanName, pvs);
			}
		}
	}
```



InjectionMetadata#inject()方法

```java
		protected void inject(Object target, String requestingBeanName, PropertyValues pvs) throws Throwable {
			if (this.isField) {
				Field field = (Field) this.member;
				ReflectionUtils.makeAccessible(field);
				field.set(target, getResourceToInject(target, requestingBeanName));
			}
			else {
				if (checkPropertySkipping(pvs)) {
					return;
				}
				try {
					Method method = (Method) this.member;
					ReflectionUtils.makeAccessible(method);
					method.invoke(target, getResourceToInject(target, requestingBeanName));
				}
				catch (InvocationTargetException ex) {
					throw ex.getTargetException();
				}
			}
		}
```



属性注入：

```java
    public class AnnotatedFieldElement extends InjectionMetadata.InjectedElement {

        private final Field field;

        private final AnnotationAttributes attributes;

        private volatile Object bean;

        protected AnnotatedFieldElement(Field field, AnnotationAttributes attributes) {
            super(field, null);
            this.field = field;
            this.attributes = attributes;
        }

        @Override
        protected void inject(Object bean, String beanName, PropertyValues pvs) throws Throwable {

            Class<?> injectedType = field.getType();

            Object injectedObject = getInjectedObject(attributes, bean, beanName, injectedType, this);

            ReflectionUtils.makeAccessible(field);

            field.set(bean, injectedObject);

        }

    }
```



方法注入：

```java
    private class AnnotatedMethodElement extends InjectionMetadata.InjectedElement {

        private final Method method;

        private final AnnotationAttributes attributes;

        private volatile Object object;

        protected AnnotatedMethodElement(Method method, PropertyDescriptor pd, AnnotationAttributes attributes) {
            super(method, pd);
            this.method = method;
            this.attributes = attributes;
        }

        @Override
        protected void inject(Object bean, String beanName, PropertyValues pvs) throws Throwable {

            Class<?> injectedType = pd.getPropertyType();

            Object injectedObject = getInjectedObject(attributes, bean, beanName, injectedType, this);

            ReflectionUtils.makeAccessible(method);

            method.invoke(bean, injectedObject);

        }

    }
```



消费者每引用的一种服务，都会创建一个ReferenceBean， 如果多个地方使用@Reference引用同一个服务，需要看他们的的缓存key是否一样，如果都是一样的，那么就只会创建一个ReferenceBean，如果有些配置不一样，比如版本号不一致，则会创建创建不同的ReferenceBean对象，这也是他版本号能够起到的作用把。至此@Reference注解已经解析完毕，并且服务引用的对象也已经创建了。
