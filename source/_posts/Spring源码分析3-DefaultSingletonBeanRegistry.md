---
title: Spring源码分析3-DefaultSingletonBeanRegistry
date: 2019-08-31 23:08:27
tags: [java,spring]
categories: spring
---



DefaultSingletonBeanRegistry类

* 继承SimpleAliasRegistry——主要实现了对单例Bean的注册和获取
* 实现了SingletonBeanRegistry接口——具有对alias的简单增删改查等功能

# 1.DefaultSingletonBeanRegistry的继承关系图

DefaultSingletonBeanRegistry类继承关系

![继承图](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/spring_DefaultSingletonBeanRegistry_hierarchy.png)



DefaultSingletonBeanRegistry类图

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/spring_DefaultSingletonBeanRegistry_uml.png)



如果所示DefaultSingletonBeanRegistry继承SimpleAliasRegistry类和实现了SingletonBeanRegistry接口。

因此这个类可以有别名注册的功能和单例bean注册的功能，并且他还支持注册DisposableBean实例



它依赖ObjectFactory接口和DisposableBean接口(关闭注册表时调用到了destroy方法)。

- **ObjectFactory** : 这个接口通常用于封装一个通用的工厂，它只有一个方法getObject() ，它调用getObject()方法返回一个新的实例，一些在每次调用的目标对象（原型）.
- **DisposableBean :** 接口实现为beans要销毁释放资源。只有一个方法destroy()，由一个破坏一个singleton的BeanFactory调用。



# 2.  父接口与父类

## 2.1 AliasRegiser接口

```java
package org.springframework.core;

/**
 * 管理别名的通用接口，是BeanDefinitionRegistry的父接口
 * @since 2.5.2
 */
public interface AliasRegistry {

	/**
	 * 给定 名称，为其注册别名。
	 */
	void registerAlias(String name, String alias);

	/**
	 * 从 registry 中删除指定的别名。
	 */
	void removeAlias(String alias);

	/**
	 * 确定此 给定名称 是否定义为别名
	 */
	boolean isAlias(String name);

	/**
	 * 返回给定名称的别名的数组
	 */
	String[] getAliases(String name);

}
```



## 2.2 SimpleAliasRegistry类

SimpleAliasRegistry实现了AliasRegistry。使用一个map作为别名的缓存

```java
public class SimpleAliasRegistry implements AliasRegistry {
	//map作为缓存，key为alias，value为name
	/** Map from alias to canonical name. */
	private final Map<String, String> aliasMap = new ConcurrentHashMap<>(16);

  // ... 其他代码略 
}
```



## 2.3 SingletonBeanRegistry接口

定义对单例的注册及获取

```java
public interface SingletonBeanRegistry {

	/**
	 * 在bean registry（注册器）中以给定的名字注册现有的对象为单例
	 * 这个对象应该完全被初始化了，注册器不会执行它的任何初始化回调（例如：afterPropertiesSet方法）
	 * 也不会收到任何销毁函数的回调（destroy method）。
	 */
	@Nullable
	Object getSingleton(String beanName);
  
	boolean containsSingleton(String beanName);

	/**
	 * 返回所有的已经注册的单例名字
	 */
	String[] getSingletonNames();

	/**
	 * 返回注册的单例个数
	 */
	int getSingletonCount();

	/**
	 * mutex：互斥
	 * 返回此注册表使用的单例互斥锁
	 * @since 4.2
	 */
	Object getSingletonMutex();
}
```

<!-- more --> 


# DefaultSingletonBeanRegistry源码注释

 DefaultSingletonBeanRegistry分析：

主要通过**几个缓存**来完成单例bean的注册与获取

* 3个主要map

  * **singletonObjects**：存放singleton对象的缓存
  * **singletonFactories**：是存放生产singleton的工厂对象的缓存
  * **earlySingletonObjects**：是存放singletonFactory 制造出来的 singleton 的缓存，然后由
* **registeredSingletons** 注册表
* 各个SingletonObject之间的关系
	* **containedBeanMap**：bean的包含关系
	* **dependentBeanMap**：被依赖关系，当前bean被Set\<String\>里的bean依赖
	* **dependenciesForBeanMap**：依赖关系:当前bean依赖的Set\<String\>
* disposableBeans存放一次性bean的缓存

在注册两个bean包含关系的时候，同时要注册他们的依赖关系。



源码注释：

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

	// 存放singleton对象的缓存
	/** Cache of singleton objects: bean name to bean instance. */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	// 是存放生产singleton的工厂对象的缓存
	/** Cache of singleton factories: bean name to ObjectFactory. */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	// 是存放singletonFactory 制造出来的 singleton 的缓存
	/** Cache of early singleton objects: bean name to bean instance. */
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

	/**
	 * 单例注册表
	 */
	/** Set of registered singletons, containing the bean names in registration order. */
	private final Set<String> registeredSingletons = new LinkedHashSet<>(256);

	// 以上四个缓存是这个类存放单例bean的主要Map

	// 目前正在创建中的单例bean的名字的集合
	/** Names of beans that are currently in creation. */
	private final Set<String> singletonsCurrentlyInCreation =
			Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	// 创建检查排除 的bean的名字集合。
	/** Names of beans currently excluded from in creation checks. */
	private final Set<String> inCreationCheckExclusions =
			Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	// 存放异常出现的相关的原因的集合
	/** List of suppressed Exceptions, available for associating related causes. */
	@Nullable
		private Set<Exception> suppressedExceptions;

	// 标志，指示我们目前是否在销毁单例中
	/** Flag that indicates whether we're currently within destroySingletons. */
	private boolean singletonsCurrentlyInDestruction = false;

	// 存放一次性bean的缓存
	/** Disposable bean instances: bean name to disposable instance. */
	private final Map<String, Object> disposableBeans = new LinkedHashMap<>();

	// 外部bean与被包含在外部bean的所有内部bean集合包含关系的缓存
	/** Map between containing bean names: bean name to Set of bean names that the bean contains. */
	private final Map<String, Set<String>> containedBeanMap = new ConcurrentHashMap<>(16);

	// dependentBeanMap存放的是  依赖当前bean的 所有的bean的集合
	/** Map between dependent bean names: bean name to Set of dependent bean names. */
	private final Map<String, Set<String>> dependentBeanMap = new ConcurrentHashMap<>(64);

	//	dependenciesForBeanMap中存放的则是  当前Bean所依赖的Bean的集合。
	// 当前bean的 beanName 在Set里面
	/** Map between depending bean names: bean name to Set of bean names for the bean's dependencies. */
	private final Map<String, Set<String>> dependenciesForBeanMap = new ConcurrentHashMap<>(64);


	//SingletonBeanRegistry接口的registerSingleton方法的实现
	@Override
	public void registerSingleton(String beanName, Object singletonObject) throws IllegalStateException {
		Assert.notNull(beanName, "Bean name must not be null");
		Assert.notNull(singletonObject, "Singleton object must not be null");
		synchronized (this.singletonObjects) {
			Object oldObject = this.singletonObjects.get(beanName);
			// 该名称已被占用
			if (oldObject != null) {
				throw new IllegalStateException("Could not register object [" + singletonObject +
						"] under bean name '" + beanName + "': there is already object [" + oldObject + "] bound");
			}
			// 真正的注册操作在这里实现
			addSingleton(beanName, singletonObject);
		}
	}

	/**
	 * Add the given singleton object to the singleton cache of this factory.
	 * <p>To be called for eager registration of singletons.
	 * @param beanName the name of the bean
	 * @param singletonObject the singleton object
	 */
	protected void addSingleton(String beanName, Object singletonObject) {
		synchronized (this.singletonObjects) {

			this.singletonObjects.put(beanName, singletonObject);

			// beanName已被注册存放在singletonObjects缓存，那么singletonFactories不应该再持有名称为beanName的工厂
			this.singletonFactories.remove(beanName);
			// beanName已被注册存放在singletonObjects缓存，那么earlySingletonObjects不应该再持有名称为beanName的bean。
			this.earlySingletonObjects.remove(beanName);

			// beanName放进单例注册表中
			this.registeredSingletons.add(beanName);
		}
	}

	/**
	 * 添加 名称为beanName的singletonFactory对象
	 *
	 * Add the given singleton factory for building the specified singleton
	 * if necessary.
	 * <p>To be called for eager registration of singletons, e.g. to be able to
	 * resolve circular references.
	 * @param beanName the name of the bean
	 * @param singletonFactory the factory for the singleton object
	 */
	protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
			if (!this.singletonObjects.containsKey(beanName)) {
				// 判断singletonObjects内名字为beanName是否被占用，若没有，进行注册操作
				this.singletonFactories.put(beanName, singletonFactory);
				this.earlySingletonObjects.remove(beanName);
				this.registeredSingletons.add(beanName);
			}
		}
	}

	//SingletonBeanRegistry接口的getSingleton方法的实现
	@Override
	@Nullable
	public Object getSingleton(String beanName) {
		return getSingleton(beanName, true);
	}

	/**
	 * Return the (raw) singleton object registered under the given name.
	 * <p>Checks already instantiated singletons and also allows for an early
	 * reference to a currently created singleton (resolving a circular reference).
	 * @param beanName the name of the bean to look for
	 * @param allowEarlyReference whether early references should be created or not
	 * @return the registered singleton object, or {@code null} if none found
	 */
	@Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		// 如果是null，则判断是否正在创建
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				// 如果earlySingletonObjects指定的beanName的对象是不存在的且allowEarlyReference是允许的
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					// 如果存在指定beanName的singletonFactory对象
					if (singletonFactory != null) {
						// singletonFactory创建指定的单例对象
						singletonObject = singletonFactory.getObject();

						// 这里可以看出earlySingletonObjects缓存应该是存放singletonFactory产生的singleton
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}

	/**
	 * 返回在给定名称下注册的（原始）单例对象，
	 * 如果尚未注册，则使用执行的工厂类创建并注册
	 *
	 * Return the (raw) singleton object registered under the given name,
	 * creating and registering a new one if none registered yet.
	 * @param beanName the name of the bean
	 * @param singletonFactory the ObjectFactory to lazily create the singleton
	 * with, if necessary
	 * @return the registered singleton object
	 */
	public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				// 如果目前在销毁singellton
				if (this.singletonsCurrentlyInDestruction) {
					throw new BeanCreationNotAllowedException(beanName,
							"Singleton bean creation not allowed while singletons of this factory are in destruction " +
							"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
				}

				// 单例对象创建前的回调
				// singletonsCurrentlyInCreation中添加beanName
				beforeSingletonCreation(beanName);

				boolean newSingleton = false;

				// 判断存储异常相关原因的集合是否已存在
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);

				// 若没有，刚创建异常集合的实例
				if (recordSuppressedExceptions) {
					this.suppressedExceptions = new LinkedHashSet<>();
				}

				try {
					// 由参数给定的singletonFactory创建singleton对象,getObject方法的具体由ObjectFactory的实现类决定
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					// Has the singleton object implicitly appeared in the meantime ->
					// if yes, proceed with it since the exception indicates that state.
					//在此期间是否隐含地出现了单例对象,如果是，则继续执行，因为异常表示该状态。
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						throw ex;
					}
				}
				catch (BeanCreationException ex) {
					if (recordSuppressedExceptions) {
						for (Exception suppressedException : this.suppressedExceptions) {
							ex.addRelatedCause(suppressedException);
						}
					}
					throw ex;
				}
				finally {
					// 结束前，将异常集合销毁掉
					if (recordSuppressedExceptions) {
						this.suppressedExceptions = null;
					}

					// 单例创建之后的回调,
					//singletonsCurrentlyInCreation中移除beanName
					afterSingletonCreation(beanName);
				}
				if (newSingleton) {
					// 注册创建后的单例
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}

	/**
	 * 注册发生在singeton bean实例创建期间发生的异常
	 *
	 * Register an Exception that happened to get suppressed during the creation of a
	 * singleton bean instance, e.g. a temporary circular reference resolution problem.
	 * @param ex the Exception to register
	 */
	protected void onSuppressedException(Exception ex) {
		synchronized (this.singletonObjects) {
			if (this.suppressedExceptions != null) {
				this.suppressedExceptions.add(ex);
			}
		}
	}

	/**
	 * 移除名称为beanName的单例,主要在四个集合中移除，
	 *
	 * Remove the bean with the given name from the singleton cache of this factory,
	 * to be able to clean up eager registration of a singleton if creation failed.
	 * @param beanName the name of the bean
	 * @see #getSingletonMutex()
	 */
	protected void removeSingleton(String beanName) {
		synchronized (this.singletonObjects) {
			this.singletonObjects.remove(beanName);
			this.singletonFactories.remove(beanName);
			this.earlySingletonObjects.remove(beanName);
			this.registeredSingletons.remove(beanName);
		}
	}

	// SingletonBeanRegistry接口的containsSingleton方法实现
	@Override
	public boolean containsSingleton(String beanName) {
		return this.singletonObjects.containsKey(beanName);
	}

	// SingletonBeanRegistry接口的getSingletonNames方法实现
	@Override
	public String[] getSingletonNames() {
		// 对singletonObjects加锁，可能是为了防止registeredSingletons和singletonObjects出现不一致的问题
		synchronized (this.singletonObjects) {
			return StringUtils.toStringArray(this.registeredSingletons);
		}
	}

	// SingletonBeanRegistry接口的getSingletonCount方法实现
	@Override
	public int getSingletonCount() {
		synchronized (this.singletonObjects) {
			return this.registeredSingletons.size();
		}
	}


	public void setCurrentlyInCreation(String beanName, boolean inCreation) {
		Assert.notNull(beanName, "Bean name must not be null");
		if (!inCreation) {
			this.inCreationCheckExclusions.add(beanName);
		}
		else {
			this.inCreationCheckExclusions.remove(beanName);
		}
	}

	public boolean isCurrentlyInCreation(String beanName) {
		Assert.notNull(beanName, "Bean name must not be null");
		return (!this.inCreationCheckExclusions.contains(beanName) && isActuallyInCreation(beanName));
	}

	protected boolean isActuallyInCreation(String beanName) {
		return isSingletonCurrentlyInCreation(beanName);
	}

	/**
	 * 指定的bean是否正在被创建
	 *
	 * Return whether the specified singleton bean is currently in creation
	 * (within the entire factory).
	 * @param beanName the name of the bean
	 */
	public boolean isSingletonCurrentlyInCreation(String beanName) {
		return this.singletonsCurrentlyInCreation.contains(beanName);
	}

	/**
	 * Callback before singleton creation.
	 * <p>The default implementation register the singleton as currently in creation.
	 * @param beanName the name of the singleton about to be created
	 * @see #isSingletonCurrentlyInCreation
	 */
	protected void beforeSingletonCreation(String beanName) {
		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
			throw new BeanCurrentlyInCreationException(beanName);
		}
	}

	/**
	 * Callback after singleton creation.
	 * <p>The default implementation marks the singleton as not in creation anymore.
	 * @param beanName the name of the singleton that has been created
	 * @see #isSingletonCurrentlyInCreation
	 */
	protected void afterSingletonCreation(String beanName) {
		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
			throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
		}
	}


	/**
	 * 一次性bean注册，存放在disponsableBeans集合中
	 *
	 * Add the given bean to the list of disposable beans in this registry.
	 * <p>Disposable beans usually correspond to registered singletons,
	 * matching the bean name but potentially being a different instance
	 * (for example, a DisposableBean adapter for a singleton that does not
	 * naturally implement Spring's DisposableBean interface).
	 * @param beanName the name of the bean
	 * @param bean the bean instance
	 */
	public void registerDisposableBean(String beanName, DisposableBean bean) {
		synchronized (this.disposableBeans) {
			this.disposableBeans.put(beanName, bean);
		}
	}

	/**
	 * 注册两个bean之间的包含关系,例如内部bean和包含其的外部bean之间
	 *
	 * Register a containment relationship between two beans,
	 * e.g. between an inner bean and its containing outer bean.
	 * <p>Also registers the containing bean as dependent on the contained bean
	 * in terms of destruction order.
	 * @param containedBeanName the name of the contained (inner) bean
	 * @param containingBeanName the name of the containing (outer) bean
	 * @see #registerDependentBean
	 */
	public void registerContainedBean(String containedBeanName, String containingBeanName) {
		synchronized (this.containedBeanMap) {

			// 从containedBeanMap缓存中查找外部bean名为containingBeanName的内部bean集合
			// 如果没有，刚新建一个存放内部bean的集合，并且存放在containedBeanMap缓存中
			Set<String> containedBeans =
					this.containedBeanMap.computeIfAbsent(containingBeanName, k -> new LinkedHashSet<>(8));

			// 将名为containedBeanName的内部bean存放到内部bean集合
			if (!containedBeans.add(containedBeanName)) {
				return;
			}
		}

		// 紧接着调用注册内部bean和外部bean的依赖关系的方法
		registerDependentBean(containedBeanName, containingBeanName);
	}

	/**
	 * 注册给定bean的一个依赖bean，给定的bean销毁之前被销毁。
	 *
	 * Register a dependent bean for the given bean,
	 * to be destroyed before the given bean is destroyed.
	 * @param beanName the name of the bean
	 * @param dependentBeanName the name of the dependent bean
	 */
	public void registerDependentBean(String beanName, String dependentBeanName) {

		// 调用SimpleAliasRegistry的canonicalName方法，这方法是将参数beanName当做别名寻找到注册名，并依此递归
		String canonicalName = canonicalName(beanName);

		synchronized (this.dependentBeanMap) {
			// 从dependentBeanMap缓存中找到依赖名为canonicalName这个bean的 依赖bean集合
			Set<String> dependentBeans =
					this.dependentBeanMap.computeIfAbsent(canonicalName, k -> new LinkedHashSet<>(8));

			// 依赖bean集合添加参数2指定的dependentBeanName
			if (!dependentBeans.add(dependentBeanName)) {
				return;
			}
		}

		synchronized (this.dependenciesForBeanMap) {
			// 从dependenciesForBeanMap缓存中找到 dependentBeanName 要依赖的所有bean集合
			Set<String> dependenciesForBean =
					this.dependenciesForBeanMap.computeIfAbsent(dependentBeanName, k -> new LinkedHashSet<>(8));

			dependenciesForBean.add(canonicalName);
		}
	}

	/**
	 *
	 * beanName
	 *
	 * Determine whether the specified dependent bean has been registered as
	 * dependent on the given bean or on any of its transitive dependencies.
	 * @param beanName the name of the bean to check
	 * @param dependentBeanName the name of the dependent bean
	 * @since 4.0
	 */
	protected boolean isDependent(String beanName, String dependentBeanName) {
		synchronized (this.dependentBeanMap) {
			return isDependent(beanName, dependentBeanName, null);
		}
	}

	// dependentBeanName是否依赖于 当前beanName
	private boolean isDependent(String beanName, String dependentBeanName, @Nullable Set<String> alreadySeen) {
		if (alreadySeen != null && alreadySeen.contains(beanName)) {
			return false;
		}
		String canonicalName = canonicalName(beanName);

		// 拿到依赖于当前的bean 的所有的beanName集合
		Set<String> dependentBeans = this.dependentBeanMap.get(canonicalName);
		if (dependentBeans == null) {
			return false;
		}

		// 依赖当前bean
		if (dependentBeans.contains(dependentBeanName)) {
			return true;
		}


		// 递归处理间接依赖
		for (String transitiveDependency : dependentBeans) {
			if (alreadySeen == null) {
				alreadySeen = new HashSet<>();
			}
			alreadySeen.add(beanName);
			if (isDependent(transitiveDependency , dependentBeanName, alreadySeen)) {
				return true;
			}
		}
		return false;
	}

	/**
	 * 确定是否还存在 名为beanName的 被依赖关系
	 *
	 * Determine whether a dependent bean has been registered for the given name.
	 * @param beanName the name of the bean to check
	 */
	protected boolean hasDependentBean(String beanName) {
		return this.dependentBeanMap.containsKey(beanName);
	}

	/**
	 * 返回 依赖于指定的bean的所有bean的名称，如果有的话。
	 *
	 * Return the names of all beans which depend on the specified bean, if any.
	 * @param beanName the name of the bean
	 * @return the array of dependent bean names, or an empty array if none
	 */
	public String[] getDependentBeans(String beanName) {
		Set<String> dependentBeans = this.dependentBeanMap.get(beanName);
		if (dependentBeans == null) {
			return new String[0];
		}
		synchronized (this.dependentBeanMap) {
			return StringUtils.toStringArray(dependentBeans);
		}
	}

	/**
	 *
	 * 返回 指定的bean依赖的 所有的bean的名称，如果有的话。
	 *
	 * Return the names of all beans that the specified bean depends on, if any.
	 * @param beanName the name of the bean
	 * @return the array of names of beans which the bean depends on,
	 * or an empty array if none
	 */
	public String[] getDependenciesForBean(String beanName) {
		Set<String> dependenciesForBean = this.dependenciesForBeanMap.get(beanName);
		if (dependenciesForBean == null) {
			return new String[0];
		}
		synchronized (this.dependenciesForBeanMap) {
			return StringUtils.toStringArray(dependenciesForBean);
		}
	}

	/**
	 * 销毁单例
	 */
	public void destroySingletons() {
		if (logger.isTraceEnabled()) {
			logger.trace("Destroying singletons in " + this);
		}
		synchronized (this.singletonObjects) {
			// 单例目前销毁标志开始
			this.singletonsCurrentlyInDestruction = true;
		}

		// 销毁disposableBeans缓存中所有单例bean
		String[] disposableBeanNames;
		synchronized (this.disposableBeans) {
			disposableBeanNames = StringUtils.toStringArray(this.disposableBeans.keySet());
		}
		for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
			destroySingleton(disposableBeanNames[i]);
		}


		// containedBeanMap缓存清空
		// dependentBeanMap缓存清空
		// dependenciesForBeanMap缓存清空
		this.containedBeanMap.clear();
		this.dependentBeanMap.clear();
		this.dependenciesForBeanMap.clear();

		clearSingletonCache();
	}

	/**
	 * Clear all cached singleton instances in this registry.
	 * @since 4.3.15
	 */
	protected void clearSingletonCache() {
		synchronized (this.singletonObjects) {
			//销毁四个map缓存
			this.singletonObjects.clear();
			this.singletonFactories.clear();
			this.earlySingletonObjects.clear();
			this.registeredSingletons.clear();

			// 单例目前正在销毁标志为结束
			this.singletonsCurrentlyInDestruction = false;
		}
	}

	/**
	 * Destroy the given bean. Delegates to {@code destroyBean}
	 * if a corresponding disposable bean instance is found.
	 * @param beanName the name of the bean
	 * @see #destroyBean
	 */
	public void destroySingleton(String beanName) {
		// Remove a registered singleton of the given name, if any.
		removeSingleton(beanName);

		// 销毁相应的DisposableBean实例
		// Destroy the corresponding DisposableBean instance.
		DisposableBean disposableBean;
		synchronized (this.disposableBeans) {
			disposableBean = (DisposableBean) this.disposableBeans.remove(beanName);
		}
		destroyBean(beanName, disposableBean);
	}

	/**
	 * 销毁给定的bean，在销毁之前，必须销毁所有依赖于给定的bean的bean
	 *
	 * Destroy the given bean. Must destroy beans that depend on the given
	 * bean before the bean itself. Should not throw any exceptions.
	 * @param beanName the name of the bean
	 * @param bean the bean instance to destroy
	 */
	protected void destroyBean(String beanName, @Nullable DisposableBean bean) {
		// 触发销毁所有  依赖当前bean 的bean们
		// Trigger destruction of dependent beans first...
		Set<String> dependencies;
		synchronized (this.dependentBeanMap) {
			// Within full synchronization in order to guarantee a disconnected Set
			dependencies = this.dependentBeanMap.remove(beanName);
		}
		if (dependencies != null) {
			if (logger.isTraceEnabled()) {
				logger.trace("Retrieved dependent beans for bean '" + beanName + "': " + dependencies);
			}
			for (String dependentBeanName : dependencies) {
				// 遍历销毁
				destroySingleton(dependentBeanName);
			}
		}

		// 销毁当前的bean
		// Actually destroy the bean now...
		if (bean != null) {
			try {
				bean.destroy();
			}
			catch (Throwable ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Destruction of bean with name '" + beanName + "' threw an exception", ex);
				}
			}
		}

		// 触发销毁 包含的beans
		// Trigger destruction of contained beans...
		Set<String> containedBeans;
		synchronized (this.containedBeanMap) {
			// Within full synchronization in order to guarantee a disconnected Set
			containedBeans = this.containedBeanMap.remove(beanName);
		}
		if (containedBeans != null) {
			for (String containedBeanName : containedBeans) {
				destroySingleton(containedBeanName);
			}
		}

		// Remove destroyed bean from other beans' dependencies.
		// 移除当前 Bean所依赖的Bean的集合，
		synchronized (this.dependentBeanMap) {
			for (Iterator<Map.Entry<String, Set<String>>> it = this.dependentBeanMap.entrySet().iterator(); it.hasNext();) {
				Map.Entry<String, Set<String>> entry = it.next();
				Set<String> dependenciesToClean = entry.getValue();
				dependenciesToClean.remove(beanName);
				if (dependenciesToClean.isEmpty()) {
					it.remove();
				}
			}
		}

		// 最后 从dependenciesForBeanMap缓存中移除要销毁的bean
		// Remove destroyed bean's prepared dependency information.
		this.dependenciesForBeanMap.remove(beanName);
	}

	public final Object getSingletonMutex() {
		return this.singletonObjects;
	}

}
```

