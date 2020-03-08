---

title: Spring源码解析6-BeanDefinitionReader
date: 2019-09-03 09:00:21
tags: [java,spring]
categories: spring
---

BeanDefinitionReader接口有定义资源文件读取并转换为BeanDefinition的功能。

使用Resource和String位置参数指定加载方法。

# 1. BeanDefinitionReader接口

BeanDefinitionReader定义

```java
public interface BeanDefinitionReader {

	/**
	 * 返回BeanFactory 以注册bean definitions
	 */
	BeanDefinitionRegistry getRegistry();

	/**
	 * 返回资源文件加载器
	 */
	@Nullable
	ResourceLoader getResourceLoader();

	/**
	 * 返回类加载器
	 */
	@Nullable
	ClassLoader getBeanClassLoader();

	/**
	 * Return the BeanNameGenerator to use for anonymous beans
	 * (without explicit bean name specified).
	 */
	BeanNameGenerator getBeanNameGenerator();


	/**
	 * 从指定的资源里加载bean，以Resource形式给出
	 * 返回加载的bean定义数量
	 */
	int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException;

	/**
	 * 从多个资源处加载bean definitions
	 */
	int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException;

	/**
	 * 从指定的资源里加载bean，以String形式给出
	 * 返回加载的bean定义数量
	 */
	int loadBeanDefinitions(String location) throws BeanDefinitionStoreException;

	/**
	 * 从多个资源处加载bean definitions
	 */
	int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException;

}
```

# 2. BeanDefinitionReader的常见实现类

常见的BeanDefinitionReader的实现见下图。

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/spring_BeanDefinitionReader_hierarchy.png)



## 2.1 AbstractBeanDefinitionReader类

AbstractBeanDefinitionReader类实现了BeanDefinitionReader。

提供常用属性，例如要处理的bean工厂和用于加载bean类的类加载器。

```java
public abstract class AbstractBeanDefinitionReader implements BeanDefinitionReader, EnvironmentCapable {

	/** Logger available to subclasses. */
	protected final Log logger = LogFactory.getLog(getClass());

	// 注册器
	private final BeanDefinitionRegistry registry;

	// 资源加载器
	@Nullable
	private ResourceLoader resourceLoader;

	// 类加载器
	@Nullable
	private ClassLoader beanClassLoader;

	// 环境
	private Environment environment;

	// bean名称生成器
	private BeanNameGenerator beanNameGenerator = DefaultBeanNameGenerator.INSTANCE;


	/**
	 * 使用给定的bean factory创建一个AbstractBeanDefinitionReader
	 */
	protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		this.registry = registry;

		// 确定要使用的ResourceLoader。
		if (this.registry instanceof ResourceLoader) {
			// 如果构造函数的参数registry 也实现了ResourceLoader，那么就使用这个registry作为资源加载器
			this.resourceLoader = (ResourceLoader) this.registry;
		}
		else {
			// 否则，使用PathMatchingResourcePatternResolver作为资源加载器
			this.resourceLoader = new PathMatchingResourcePatternResolver();
		}

		// 如果可能，继承Environment
		if (this.registry instanceof EnvironmentCapable) {
			this.environment = ((EnvironmentCapable) this.registry).getEnvironment();
		}
		else {
			// 否则默认的是StandardEnvironment
			this.environment = new StandardEnvironment();
		}
	}


	public final BeanDefinitionRegistry getBeanFactory() {
		return this.registry;
	}

	@Override
	public final BeanDefinitionRegistry getRegistry() {
		return this.registry;
	}

	public void setResourceLoader(@Nullable ResourceLoader resourceLoader) {
		this.resourceLoader = resourceLoader;
	}

	@Override
	@Nullable
	public ResourceLoader getResourceLoader() {
		return this.resourceLoader;
	}

	public void setBeanClassLoader(@Nullable ClassLoader beanClassLoader) {
		this.beanClassLoader = beanClassLoader;
	}

	@Override
	@Nullable
	public ClassLoader getBeanClassLoader() {
		return this.beanClassLoader;
	}

	public void setEnvironment(Environment environment) {
		Assert.notNull(environment, "Environment must not be null");
		this.environment = environment;
	}

	@Override
	public Environment getEnvironment() {
		return this.environment;
	}

	public void setBeanNameGenerator(@Nullable BeanNameGenerator beanNameGenerator) {
		// 默认的是 DefaultBeanNameGenerator
		this.beanNameGenerator = (beanNameGenerator != null ? beanNameGenerator : DefaultBeanNameGenerator.INSTANCE);
	}

	@Override
	public BeanNameGenerator getBeanNameGenerator() {
		return this.beanNameGenerator;
	}


	@Override
	public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
		Assert.notNull(resources, "Resource array must not be null");
		int count = 0;
		for (Resource resource : resources) {
			count += loadBeanDefinitions(resource);
		}
		return count;
	}

	@Override
	public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(location, null);
	}

	public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					"Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");
		}

		if (resourceLoader instanceof ResourcePatternResolver) {
			// Resource pattern matching available.
			try {
				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
				int count = loadBeanDefinitions(resources);
				if (actualResources != null) {
					Collections.addAll(actualResources, resources);
				}
				if (logger.isTraceEnabled()) {
					logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");
				}
				return count;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"Could not resolve bean definition resource pattern [" + location + "]", ex);
			}
		}
		else {
			// 只能从使用绝对URL路径的Resource中加载
			// Can only load single resources by absolute URL.
			Resource resource = resourceLoader.getResource(location);
			int count = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			if (logger.isTraceEnabled()) {
				logger.trace("Loaded " + count + " bean definitions from location [" + location + "]");
			}
			return count;
		}
	}

	@Override
	public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
		Assert.notNull(locations, "Location array must not be null");
		int count = 0;
		for (String location : locations) {
			count += loadBeanDefinitions(location);
		}
		return count;
	}

}
```



# 2.2 XmlBeanDefinitionReader类

各个类的功能

* ResourceLoader：定义资源加载器，主要应用于根据给定的资源文件地址返回对应的Resource

* BeanDefinitionReader：主要定义资源文件读取并转换为BeanDefinition的各个功能

* EnvironmentCapable：定义获取Environment的方法

* DocumentLoader：定义从资源文件加载到转换为Document的功能

* AbstractBeanDefinitionReader：对EnvironmentCapable、BeanDefinitionReader类定义的功能进行实现

* BeanDefinitionDocumentReader：定义读取Document并注册BeanDefinition功能

* BeanDefinitionParserDelegate：定义解析Element的各种方法

  

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/spring_XmlBeanDefinitionReader_uml.png)



在XMLBeanDefinitionReader中主要包含以下几步的处理：

1. 通过继承AbstractBeanDefinitionReader中的方法，来使用ResourceLoader将资源文件路径转换为对应的Resource文件
2. 通过DocumentLoader对Resource文件进行转换，将Resource文件转换为Document文件
3. 通过实现接口BeanDefinitionDocumentReader的DefaultBeanDefinitionDocumentReader类对Document进行解析，并使用BeanDefinitionParserDelegate对Element进行解析

