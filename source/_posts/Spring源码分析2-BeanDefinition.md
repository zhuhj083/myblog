---
title: Spring源码分析2-BeanDefinition
date: 2019-08-31 17:47:20
tags: [java,spring]
categories: spring
---


# 1. BeanDefinition简介

BeanDefinition描述了一个bean实例，是配置文件\<bean\>元素标签在容器中内部表示形式。

它具有**属性值**，**构造函数参数值**以及具体实现提供的更多信息。
这是一个最小化的接口，主要的意向是允许一个BeanFactoryPostProcessor来修改它的属性值和其他的元数据。



BeanDefinition是一个接口，在Spring中有三种实现，三种实现均继承了AbstractBeanDefinition。BeanDefinition是配置文件\<bean\>元素标签在容器中的内部表现形式，<bean\>元素标签拥有class、scope、lazy-init等配置属性，BeanDefinition则提供了相应的beanClass、scope、lazyInit属性。BeanDefinition和\<bean\>是一一对应的。

* RootBeanDefinition——最常用的实现类，它对应一般性的\<bean\>元素
* ChildBeanDefinition
* GenericBeanDefinition——2.5版本后加入的bean文件配置属性定义类，是一站式服务类



Spring通过BeanDefinition将配置文件中的\<bean\>配置信息转换为容器的内部表示，并将这些BeanDefinition注册到BeanDefinitionRegistry中（主要以map的形式保存）。



#  2. BeanDefinition的父接口

BeanDefinition继承了AttributeAccessor，说明它具有处理属性的能

BeanDefinition继承了BeanMetadataElement，说明它可以持有**Bean元数据元素**，作用是可以持有XML文件的一个bean标签对应的Object。

继承关系：

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/spring_BeanDefinition_uml.png)

下面看BeanDefinition继承的接口。

##  1.1 BeanMetadataElement接口

```java
/**
 * 由携带配置源对象的bean元数据元素实现的接口。
 */
public interface BeanMetadataElement {

	/**
	 * 返回此元数据元素的配置源
	 */
	@Nullable
	default Object getSource() {
		return null;
	}

}
```



### 1.1.1 BeanMetadataAttribute类

BeanMetadataAttribute类是BeanMetadataElement接口的一个实现。通常作为BeanDefinition的一部分。

以key-value的形式持有属性。

```java
public class BeanMetadataAttribute implements BeanMetadataElement {

	// 键值对中的key
	private final String name;

	// 键值对中的value
	@Nullable
	private final Object value;

	@Nullable
	private Object source;


	/**
	 * Create a new AttributeValue instance.
	 * @param name the name of the attribute (never {@code null})
	 * @param value the value of the attribute (possibly before type conversion)
	 */
	public BeanMetadataAttribute(String name, @Nullable Object value) {
		Assert.notNull(name, "Name must not be null");
		this.name = name;
		this.value = value;
	}


	/**
	 * Return the name of the attribute.
	 */
	public String getName() {
		return this.name;
	}

	/**
	 * Return the value of the attribute.
	 */
	@Nullable
	public Object getValue() {
		return this.value;
	}

	/**
	 * Set the configuration source {@code Object} for this metadata element.
	 * <p>The exact type of the object will depend on the configuration mechanism used.
	 */
	public void setSource(@Nullable Object source) {
		this.source = source;
	}

	@Override
	@Nullable
	public Object getSource() {
		return this.source;
	}


	@Override
	public boolean equals(@Nullable Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof BeanMetadataAttribute)) {
			return false;
		}
		BeanMetadataAttribute otherMa = (BeanMetadataAttribute) other;
		return (this.name.equals(otherMa.name) &&
				ObjectUtils.nullSafeEquals(this.value, otherMa.value) &&
				ObjectUtils.nullSafeEquals(this.source, otherMa.source));
	}

	@Override
	public int hashCode() {
		return this.name.hashCode() * 29 + ObjectUtils.nullSafeHashCode(this.value);
	}

	@Override
	public String toString() {
		return "metadata attribute '" + this.name + "'";
	}

}
```



##  1.2  AttributeAccessor接口

```java
/**
 * 定义用于向/从任意对象附加和访问元数据的通用契约的接口。
 */
public interface AttributeAccessor {

	/**
	 * 将属性名设置为给定的值value。如果value是null，这个属性会被移除
	 * 为防止属性名重叠，最好用类或者包名作为前缀。
	 */
	void setAttribute(String name, @Nullable Object value);

	/**
	 * 根据属性名获取属性值
	 */
	@Nullable
	Object getAttribute(String name);

	/**
	 * 移除属性名，并返回它的值
	 */
	@Nullable
	Object removeAttribute(String name);

	/**
	 * 属性是否存在
	 */
	boolean hasAttribute(String name);

	/**
	 * 返回所有的属性名数组
	 */
	String[] attributeNames();

}
```

<!-- more -->

### 1.2.1 AttributeAccessorSupport虚类

Abstract AttributeAccessorSupport类是AccessorSupport的一个实现。实现了AccessorSupport接口的中的方法。

这个类很简单，就是使用了一个map来存储属性，key为String，value为任意对象。

```java
public abstract class AttributeAccessorSupport implements AttributeAccessor, Serializable {

	//使用一个map来存储属性值。
	/** Map with String keys and Object values. */
	private final Map<String, Object> attributes = new LinkedHashMap<>();


	@Override
	public void setAttribute(String name, @Nullable Object value) {
		Assert.notNull(name, "Name must not be null");
		if (value != null) {
			this.attributes.put(name, value);
		}
		else {
			removeAttribute(name);
		}
	}

	@Override
	@Nullable
	public Object getAttribute(String name) {
		Assert.notNull(name, "Name must not be null");
		return this.attributes.get(name);
	}

	@Override
	@Nullable
	public Object removeAttribute(String name) {
		Assert.notNull(name, "Name must not be null");
		return this.attributes.remove(name);
	}

	@Override
	public boolean hasAttribute(String name) {
		Assert.notNull(name, "Name must not be null");
		return this.attributes.containsKey(name);
	}

	@Override
	public String[] attributeNames() {
		return StringUtils.toStringArray(this.attributes.keySet());
	}


	/**
	 * Copy the attributes from the supplied AttributeAccessor to this accessor.
	 * @param source the AttributeAccessor to copy from
	 */
	protected void copyAttributesFrom(AttributeAccessor source) {
		Assert.notNull(source, "Source must not be null");
		String[] attributeNames = source.attributeNames();
		for (String attributeName : attributeNames) {
			setAttribute(attributeName, source.getAttribute(attributeName));
		}
	}


	@Override
	public boolean equals(@Nullable Object other) {
		return (this == other || (other instanceof AttributeAccessorSupport &&
				this.attributes.equals(((AttributeAccessorSupport) other).attributes)));
	}

	@Override
	public int hashCode() {
		return this.attributes.hashCode();
	}

}
```



### 1.2.2 BeanMetadataAttributeAccessor类

BeanMetadataAttributeAccessor是AttributeAccessorSupport的一个子类，并且实现了BeanMetadataElement接口。

以BeanMetadataAttribute对象的形式来持有属性，以便跟踪 定义源。

BeanMetadataAttribute中以key-value的形式持有了一个属性。

```java
public class BeanMetadataAttributeAccessor extends AttributeAccessorSupport implements BeanMetadataElement {

	@Nullable
	private Object source;


	/**
	 * Set the configuration source {@code Object} for this metadata element.
	 * <p>The exact type of the object will depend on the configuration mechanism used.
	 */
	public void setSource(@Nullable Object source) {
		this.source = source;
	}

	@Override
	@Nullable
	public Object getSource() {
		return this.source;
	}


	/**
	 * Add the given BeanMetadataAttribute to this accessor's set of attributes.
	 * @param attribute the BeanMetadataAttribute object to register
	 */
	public void addMetadataAttribute(BeanMetadataAttribute attribute) {
		//父类AttributeAccessorSupport 来处理，存储到父类AttributeAccessorSupport的map中
		super.setAttribute(attribute.getName(), attribute);
	}

	/**
	 * Look up the given BeanMetadataAttribute in this accessor's set of attributes.
	 * @param name the name of the attribute
	 * @return the corresponding BeanMetadataAttribute object,
	 * or {@code null} if no such attribute defined
	 */
	@Nullable
	public BeanMetadataAttribute getMetadataAttribute(String name) {
		return (BeanMetadataAttribute) super.getAttribute(name);
	}

	@Override
	public void setAttribute(String name, @Nullable Object value) {
		super.setAttribute(name, new BeanMetadataAttribute(name, value));
	}

	@Override
	@Nullable
	public Object getAttribute(String name) {
		BeanMetadataAttribute attribute = (BeanMetadataAttribute) super.getAttribute(name);
		return (attribute != null ? attribute.getValue() : null);
	}

	@Override
	@Nullable
	public Object removeAttribute(String name) {
		BeanMetadataAttribute attribute = (BeanMetadataAttribute) super.removeAttribute(name);
		return (attribute != null ? attribute.getValue() : null);
	}

}
```



# 3. BeanDefinition源码分析

将Bean的定义信息存储到这个BeanDefinition相应的属性中，后面对Bean的操作就直接对BeanDefinition进行，例如拿到这个BeanDefinition后，可以**根据里面的类名、构造函数、构造函数参数，使用反射进行对象创建**。



下面看BeanDefinition源码：

```java
/**
 * BeanDefinition描述了一个bean实例，它具有属性值、构造函数参数值、以及更多这个接口的具体实现类提供的更多信息
 *
 * 这是一个最小化的接口，主要的意向是允许一个BeanFactoryPostProcessor来修改它的属性值和其他的元数据。
 */
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

	/**
	 * 单例
	 */
	String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;

	/**
	 * 原型
	 */
	String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;


	/**
	 * 角色提示指示
	 * 通常对应一个用户自定义的bean
	 */
	int ROLE_APPLICATION = 0;

	/**
	 * 角色提示指示
	 */
	int ROLE_SUPPORT = 1;

	/**
	 * 角色提示指示
	 *
	 * 完全是后台角色，与终端用户无关
	 * 内部工作使用
	 */
	int ROLE_INFRASTRUCTURE = 2;


	// Modifiable attributes

	/**
	 * 设置当前bean definition的父bean definition
	 */
	void setParentName(@Nullable String parentName);

	/**
	 * 返回父bean definition
	 */
	@Nullable
	String getParentName();

	/**
	 * 指定当前bean definition的类名
	 * 在 bean工厂后处理 期间可以修改类名，通常用解析后的变体替换原始类名。
	 */
	void setBeanClassName(@Nullable String beanClassName);

	/**
	 * 返回此bean定义的当前bean类名
	 */
	@Nullable
	String getBeanClassName();

	/**
	 * 设置范围
	 * @see #SCOPE_SINGLETON
	 * @see #SCOPE_PROTOTYPE
	 */
	void setScope(@Nullable String scope);

	@Nullable
	String getScope();

	/**
	 * 设置是否延迟初始化。
	 */
	void setLazyInit(boolean lazyInit);

	boolean isLazyInit();

	/**
	 *
	 * 设置此bean依赖于初始化的bean的名称
	 * bean工厂将保证首先初始化这些bean。
	 */
	void setDependsOn(@Nullable String... dependsOn);

	/**
	 * Return the bean names that this bean depends on.
	 */
	@Nullable
	String[] getDependsOn();

	/**
	 * 设置此bean是否可以自动注入到其他bean中
	 * 只会影响 按类自动注入，不影响按名自动注入
	 */
	void setAutowireCandidate(boolean autowireCandidate);

	/**
	 * Return whether this bean is a candidate for getting autowired into some other bean.
	 */
	boolean isAutowireCandidate();

	/**
	 * 设置此bean是否为主要autowire候选者。
	 */
	void setPrimary(boolean primary);

	boolean isPrimary();

	/**
	 * 指定factory bean
	 */
	void setFactoryBeanName(@Nullable String factoryBeanName);

	/**
	 * 返回当前bean的factory bean
	 */
	@Nullable
	String getFactoryBeanName();

	/**
	 * 指定工厂方法
	 */
	void setFactoryMethodName(@Nullable String factoryMethodName);

	/**
	 * Return a factory method, if any.
	 */
	@Nullable
	String getFactoryMethodName();

	/**
	 * 返回构造器参数对象
	 */
	ConstructorArgumentValues getConstructorArgumentValues();

	/**
	 *  返回 是否构造器参数 已经定义了
	 */
	default boolean hasConstructorArgumentValues() {
		return !getConstructorArgumentValues().isEmpty();
	}

	/**
	 *
	 * 返回属性对象
	 */
	MutablePropertyValues getPropertyValues();

	/**
	 * 返回是否为此bean定义了属性值
	 */
	default boolean hasPropertyValues() {
		return !getPropertyValues().isEmpty();
	}

	/**
	 * 设置初始化方法的名称。
	 */
	void setInitMethodName(@Nullable String initMethodName);

	@Nullable
	String getInitMethodName();

	/**
	 * 设置destroy方法
	 */
	void setDestroyMethodName(@Nullable String destroyMethodName);

	/**
	 * 返回destroy方法
	 */
	@Nullable
	String getDestroyMethodName();

	/**
	 * 设置角色提示
	 * @see #ROLE_APPLICATION
	 * @see #ROLE_SUPPORT
	 * @see #ROLE_INFRASTRUCTURE
	 */
	void setRole(int role);

	/**
	 * @see #ROLE_APPLICATION
	 * @see #ROLE_SUPPORT
	 * @see #ROLE_INFRASTRUCTURE
	 */
	int getRole();

	/**
	 * 设置一个人类易读的描述
	 * Set a human-readable description of this bean definition.
	 * @since 5.1
	 */
	void setDescription(@Nullable String description);

	/**
	 * 返回描述
	 * Return a human-readable description of this bean definition.
	 */
	@Nullable
	String getDescription();


	// Read-only attributes

	/**
	* 返回当前beanDefinition的一个ResolvableType
	* 见Spring源码分析5-ResolvableType
	 */
	ResolvableType getResolvableType();

	/**
	 * 返回是否是一个单例bean
	 */
	boolean isSingleton();

	/**
	 * 返回是否是一个原型bean
	 */
	boolean isPrototype();

	/**
	 * 返回当前bean是否是abstract，如果是，则不会被实例化
	 */
	boolean isAbstract();

	/**
	 * 返回此bean定义的资源的描述
	 */
	@Nullable
	String getResourceDescription();

	/**
	 * 返回原始BeanDefinition
	 */
	@Nullable
	BeanDefinition getOriginatingBeanDefinition();

}

```

可以看到上面的很多属性和方法都很熟悉，例如**类名**、**scope**、**属性**、**构造函数参数列表**、**依赖的bean**、**是否是单例类**、**是否是懒加载**等



# 4. ConstructorArgumentValues类

持有构造器的参数值，通常是一个BeanDefiniton的一部分

```java
public class ConstructorArgumentValues {


	//Integer为参数在构造器中的顺序值
	private final Map<Integer, ValueHolder> indexedArgumentValues = new LinkedHashMap<>();

	private final List<ValueHolder> genericArgumentValues = new ArrayList<>();


	/**
	 * Create a new empty ConstructorArgumentValues object.
	 */
	public ConstructorArgumentValues() {
	}

	/**
	 * 深拷贝
	 * Deep copy constructor.
	 * @param original the ConstructorArgumentValues to copy
	 */
	public ConstructorArgumentValues(ConstructorArgumentValues original) {
		addArgumentValues(original);
	}


	/**
	 * Copy all given argument values into this object, using separate holder
	 * instances to keep the values independent from the original object.
	 * <p>Note: Identical ValueHolder instances will only be registered once,
	 * to allow for merging and re-merging of argument value definitions. Distinct
	 * ValueHolder instances carrying the same content are of course allowed.
	 */
	public void addArgumentValues(@Nullable ConstructorArgumentValues other) {
		if (other != null) {
			other.indexedArgumentValues.forEach(
				(index, argValue) -> addOrMergeIndexedArgumentValue(index, argValue.copy())
			);
			other.genericArgumentValues.stream()
					.filter(valueHolder -> !this.genericArgumentValues.contains(valueHolder))
					.forEach(valueHolder -> addOrMergeGenericArgumentValue(valueHolder.copy()));
		}
	}


	/**
	 * 在构造函数参数列表中为给定索引添加参数值。
	 *
	 * Add an argument value for the given index in the constructor argument list.
	 * @param index the index in the constructor argument list
	 * @param value the argument value
	 */
	public void addIndexedArgumentValue(int index, @Nullable Object value) {
		addIndexedArgumentValue(index, new ValueHolder(value));
	}

	/**
	 * 在构造函数参数列表中为给定索引添加参数值。
	 * Add an argument value for the given index in the constructor argument list.
	 * @param index the index in the constructor argument list
	 * @param value the argument value
	 * @param type the type of the constructor argument
	 */
	public void addIndexedArgumentValue(int index, @Nullable Object value, String type) {
		addIndexedArgumentValue(index, new ValueHolder(value, type));
	}

	/**
	 * 在构造函数参数列表中为给定索引添加参数值。
	 * Add an argument value for the given index in the constructor argument list.
	 * @param index the index in the constructor argument list
	 * @param newValue the argument value in the form of a ValueHolder
	 */
	public void addIndexedArgumentValue(int index, ValueHolder newValue) {
		Assert.isTrue(index >= 0, "Index must not be negative");
		Assert.notNull(newValue, "ValueHolder must not be null");
		addOrMergeIndexedArgumentValue(index, newValue);
	}

	/**
	 * 在构造函数参数列表中为给定索引添加参数值，将新值（通常是集合）与当前值合并
	 *
	 * Add an argument value for the given index in the constructor argument list,
	 * merging the new value (typically a collection) with the current value
	 * if demanded: see {@link org.springframework.beans.Mergeable}.
	 * @param key the index in the constructor argument list
	 * @param newValue the argument value in the form of a ValueHolder
	 */
	private void addOrMergeIndexedArgumentValue(Integer key, ValueHolder newValue) {
		ValueHolder currentValue = this.indexedArgumentValues.get(key);
		if (currentValue != null && newValue.getValue() instanceof Mergeable) {
			Mergeable mergeable = (Mergeable) newValue.getValue();
			if (mergeable.isMergeEnabled()) {
				newValue.setValue(mergeable.merge(currentValue.getValue()));
			}
		}
		this.indexedArgumentValues.put(key, newValue);
	}

	/**
	 * 检查是否已为给定索引注册参数值。
	 *
	 * Check whether an argument value has been registered for the given index.
	 * @param index the index in the constructor argument list
	 */
	public boolean hasIndexedArgumentValue(int index) {
		return this.indexedArgumentValues.containsKey(index);
	}

	/**
	 * 获取构造函数参数列表中给定索引的参数值。
	 *
	 * Get argument value for the given index in the constructor argument list.
	 * @param index the index in the constructor argument list
	 * @param requiredType the type to match (can be {@code null} to match
	 * untyped values only)
	 * @return the ValueHolder for the argument, or {@code null} if none set
	 */
	@Nullable
	public ValueHolder getIndexedArgumentValue(int index, @Nullable Class<?> requiredType) {
		return getIndexedArgumentValue(index, requiredType, null);
	}

	/**
	 * 获取构造函数参数列表中给定索引的参数值。
	 *
	 * Get argument value for the given index in the constructor argument list.
	 * @param index the index in the constructor argument list
	 * @param requiredType the type to match (can be {@code null} to match
	 * untyped values only)
	 * @param requiredName the type to match (can be {@code null} to match
	 * unnamed values only, or empty String to match any name)
	 * @return the ValueHolder for the argument, or {@code null} if none set
	 */
	@Nullable
	public ValueHolder getIndexedArgumentValue(int index, @Nullable Class<?> requiredType, @Nullable String requiredName) {
		Assert.isTrue(index >= 0, "Index must not be negative");
		ValueHolder valueHolder = this.indexedArgumentValues.get(index);
		if (valueHolder != null &&
				(valueHolder.getType() == null ||
						(requiredType != null && ClassUtils.matchesTypeName(requiredType, valueHolder.getType()))) &&
				(valueHolder.getName() == null || "".equals(requiredName) ||
						(requiredName != null && requiredName.equals(valueHolder.getName())))) {
			return valueHolder;
		}
		return null;
	}

	/**
	 * 返回索引参数值的映射（只读的）
	 *
	 * Return the map of indexed argument values.
	 * @return unmodifiable Map with Integer index as key and ValueHolder as value
	 * @see ValueHolder
	 */
	public Map<Integer, ValueHolder> getIndexedArgumentValues() {
		return Collections.unmodifiableMap(this.indexedArgumentValues);
	}


	/**
	 * 添加要按类型匹配的通用参数值。
	 *
	 * Add a generic argument value to be matched by type.
	 * <p>Note: A single generic argument value will just be used once,
	 * rather than matched multiple times.
	 * @param value the argument value
	 */
	public void addGenericArgumentValue(Object value) {
		this.genericArgumentValues.add(new ValueHolder(value));
	}

	/**
	 * 添加要按类型匹配的通用参数值。
	 *
	 * Add a generic argument value to be matched by type.
	 * <p>Note: A single generic argument value will just be used once,
	 * rather than matched multiple times.
	 * @param value the argument value
	 * @param type the type of the constructor argument
	 */
	public void addGenericArgumentValue(Object value, String type) {
		this.genericArgumentValues.add(new ValueHolder(value, type));
	}

	/**
	 * 添加要通过类型或名称匹配的通用参数值（如果可用）。
	 *
	 * Add a generic argument value to be matched by type or name (if available).
	 * <p>Note: A single generic argument value will just be used once,
	 * rather than matched multiple times.
	 * @param newValue the argument value in the form of a ValueHolder
	 * <p>Note: Identical ValueHolder instances will only be registered once,
	 * to allow for merging and re-merging of argument value definitions. Distinct
	 * ValueHolder instances carrying the same content are of course allowed.
	 */
	public void addGenericArgumentValue(ValueHolder newValue) {
		Assert.notNull(newValue, "ValueHolder must not be null");
		if (!this.genericArgumentValues.contains(newValue)) {
			addOrMergeGenericArgumentValue(newValue);
		}
	}

	/**
	 * 添加通用参数值，将新值（通常是集合）与当前值合并（如果需要）
	 *
	 * Add a generic argument value, merging the new value (typically a collection)
	 * with the current value if demanded: see {@link org.springframework.beans.Mergeable}.
	 * @param newValue the argument value in the form of a ValueHolder
	 */
	private void addOrMergeGenericArgumentValue(ValueHolder newValue) {
		if (newValue.getName() != null) {
			for (Iterator<ValueHolder> it = this.genericArgumentValues.iterator(); it.hasNext();) {
				//遍历所有通用参数
				ValueHolder currentValue = it.next();
				if (newValue.getName().equals(currentValue.getName())) {
					if (newValue.getValue() instanceof Mergeable) {
						Mergeable mergeable = (Mergeable) newValue.getValue();
						if (mergeable.isMergeEnabled()) {
							newValue.setValue(mergeable.merge(currentValue.getValue()));
						}
					}
					it.remove();
				}
			}
		}
		this.genericArgumentValues.add(newValue);
	}

	/**
	 * 查找与给定类型匹配的泛型参数值。
	 *
	 * Look for a generic argument value that matches the given type.
	 * @param requiredType the type to match
	 * @return the ValueHolder for the argument, or {@code null} if none set
	 */
	@Nullable
	public ValueHolder getGenericArgumentValue(Class<?> requiredType) {
		return getGenericArgumentValue(requiredType, null, null);
	}

	/**
	 * Look for a generic argument value that matches the given type.
	 * @param requiredType the type to match
	 * @param requiredName the name to match
	 * @return the ValueHolder for the argument, or {@code null} if none set
	 */
	@Nullable
	public ValueHolder getGenericArgumentValue(Class<?> requiredType, String requiredName) {
		return getGenericArgumentValue(requiredType, requiredName, null);
	}

	/**
	 * 查找与给定类型匹配的下一个 泛型参数值，忽略已在当前解析过程中使用的参数值。
	 *
	 * Look for the next generic argument value that matches the given type,
	 * ignoring argument values that have already been used in the current
	 * resolution process.
	 * @param requiredType the type to match (can be {@code null} to find
	 * an arbitrary next generic argument value)
	 * @param requiredName the name to match (can be {@code null} to not
	 * match argument values by name, or empty String to match any name)
	 * @param usedValueHolders a Set of ValueHolder objects that have already been used
	 * in the current resolution process and should therefore not be returned again
	 * @return the ValueHolder for the argument, or {@code null} if none found
	 */
	@Nullable
	public ValueHolder getGenericArgumentValue(@Nullable Class<?> requiredType, @Nullable String requiredName, @Nullable Set<ValueHolder> usedValueHolders) {
		for (ValueHolder valueHolder : this.genericArgumentValues) {

			// 忽略已在当前解析过程中使用的参数值
			if (usedValueHolders != null && usedValueHolders.contains(valueHolder)) {
				continue;
			}


			//参数名字相同 忽略
			if (valueHolder.getName() != null && !"".equals(requiredName) &&
					(requiredName == null || !valueHolder.getName().equals(requiredName))) {
				continue;
			}

			// 类型不匹配，忽略
			if (valueHolder.getType() != null &&
					(requiredType == null || !ClassUtils.matchesTypeName(requiredType, valueHolder.getType()))) {
				continue;
			}

			// 需要的值不能 分配 给指定的requiredType
			if (requiredType != null && valueHolder.getType() == null && valueHolder.getName() == null &&
					!ClassUtils.isAssignableValue(requiredType, valueHolder.getValue())) {
				continue;
			}

			//返回找到的参数值持有者对象
			return valueHolder;
		}
		return null;
	}

	/**
	 * 返回只读的通用参数值持有者List集合
	 *
	 * Return the list of generic argument values.
	 * @return unmodifiable List of ValueHolders
	 * @see ValueHolder
	 */
	public List<ValueHolder> getGenericArgumentValues() {
		return Collections.unmodifiableList(this.genericArgumentValues);
	}


	/**
	 * 根据索引和类别获取 值持有者对象
	 *
	 * Look for an argument value that either corresponds to the given index
	 * in the constructor argument list or generically matches by type.
	 * @param index the index in the constructor argument list
	 * @param requiredType the parameter type to match
	 * @return the ValueHolder for the argument, or {@code null} if none set
	 */
	@Nullable
	public ValueHolder getArgumentValue(int index, Class<?> requiredType) {
		return getArgumentValue(index, requiredType, null, null);
	}

	/**
	 * 根据索引、类别和参数名获取 值持有者对象
	 *
	 * Look for an argument value that either corresponds to the given index
	 * in the constructor argument list or generically matches by type.
	 * @param index the index in the constructor argument list
	 * @param requiredType the parameter type to match
	 * @param requiredName the parameter name to match
	 * @return the ValueHolder for the argument, or {@code null} if none set
	 */
	@Nullable
	public ValueHolder getArgumentValue(int index, Class<?> requiredType, String requiredName) {
		return getArgumentValue(index, requiredType, requiredName, null);
	}

	/**
	 * Look for an argument value that either corresponds to the given index
	 * in the constructor argument list or generically matches by type.
	 * @param index the index in the constructor argument list
	 * @param requiredType the parameter type to match (can be {@code null}
	 * to find an untyped argument value)
	 * @param requiredName the parameter name to match (can be {@code null}
	 * to find an unnamed argument value, or empty String to match any name)
	 * @param usedValueHolders a Set of ValueHolder objects that have already
	 * been used in the current resolution process and should therefore not
	 * be returned again (allowing to return the next generic argument match
	 * in case of multiple generic argument values of the same type)
	 * @return the ValueHolder for the argument, or {@code null} if none set
	 */
	@Nullable
	public ValueHolder getArgumentValue(int index, @Nullable Class<?> requiredType, @Nullable String requiredName, @Nullable Set<ValueHolder> usedValueHolders) {
		Assert.isTrue(index >= 0, "Index must not be negative");
		//分别从indexedArgumentValues 和 genericArgumentValues中寻找
		ValueHolder valueHolder = getIndexedArgumentValue(index, requiredType, requiredName);
		if (valueHolder == null) {
			valueHolder = getGenericArgumentValue(requiredType, requiredName, usedValueHolders);
		}
		return valueHolder;
	}

	/**
	 * Return the number of argument values held in this instance,
	 * counting both indexed and generic argument values.
	 */
	public int getArgumentCount() {
		return (this.indexedArgumentValues.size() + this.genericArgumentValues.size());
	}

	/**
	 * Return if this holder does not contain any argument values,
	 * neither indexed ones nor generic ones.
	 */
	public boolean isEmpty() {
		return (this.indexedArgumentValues.isEmpty() && this.genericArgumentValues.isEmpty());
	}

	/**
	 * 清空
	 * Clear this holder, removing all argument values.
	 */
	public void clear() {
		this.indexedArgumentValues.clear();
		this.genericArgumentValues.clear();
	}


	@Override
	public boolean equals(@Nullable Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof ConstructorArgumentValues)) {
			return false;
		}
		ConstructorArgumentValues that = (ConstructorArgumentValues) other;
		if (this.genericArgumentValues.size() != that.genericArgumentValues.size() ||
				this.indexedArgumentValues.size() != that.indexedArgumentValues.size()) {
			return false;
		}
		Iterator<ValueHolder> it1 = this.genericArgumentValues.iterator();
		Iterator<ValueHolder> it2 = that.genericArgumentValues.iterator();
		while (it1.hasNext() && it2.hasNext()) {
			ValueHolder vh1 = it1.next();
			ValueHolder vh2 = it2.next();
			if (!vh1.contentEquals(vh2)) {
				return false;
			}
		}
		for (Map.Entry<Integer, ValueHolder> entry : this.indexedArgumentValues.entrySet()) {
			ValueHolder vh1 = entry.getValue();
			ValueHolder vh2 = that.indexedArgumentValues.get(entry.getKey());
			if (!vh1.contentEquals(vh2)) {
				return false;
			}
		}
		return true;
	}

	@Override
	public int hashCode() {
		int hashCode = 7;
		for (ValueHolder valueHolder : this.genericArgumentValues) {
			hashCode = 31 * hashCode + valueHolder.contentHashCode();
		}
		hashCode = 29 * hashCode;
		for (Map.Entry<Integer, ValueHolder> entry : this.indexedArgumentValues.entrySet()) {
			hashCode = 31 * hashCode + (entry.getValue().contentHashCode() ^ entry.getKey().hashCode());
		}
		return hashCode;
	}


	/**
	 * 内部类，持有一个构造器参数的值
	 * 使用可选的type属性指示实际构造函数参数的目标类型
	 *
	 * Holder for a constructor argument value, with an optional type
	 * attribute indicating the target type of the actual constructor argument.
	 */
	public static class ValueHolder implements BeanMetadataElement {

		@Nullable
		private Object value;

		@Nullable
		private String type;

		@Nullable
		private String name;

		@Nullable
		private Object source;

		// 转换标志
		private boolean converted = false;

		@Nullable
		private Object convertedValue;

		/**
		 * 构造函数
		 * Create a new ValueHolder for the given value.
		 * @param value the argument value
		 */
		public ValueHolder(@Nullable Object value) {
			this.value = value;
		}

		/**
		 * 构造函数
		 * Create a new ValueHolder for the given value and type.
		 * @param value the argument value
		 * @param type the type of the constructor argument
		 */
		public ValueHolder(@Nullable Object value, @Nullable String type) {
			this.value = value;
			this.type = type;
		}

		/**
		 * 构造函数
		 * Create a new ValueHolder for the given value, type and name.
		 * @param value the argument value
		 * @param type the type of the constructor argument
		 * @param name the name of the constructor argument
		 */
		public ValueHolder(@Nullable Object value, @Nullable String type, @Nullable String name) {
			this.value = value;
			this.type = type;
			this.name = name;
		}

		/**
		 * Set the value for the constructor argument.
		 */
		public void setValue(@Nullable Object value) {
			this.value = value;
		}

		/**
		 * Return the value for the constructor argument.
		 */
		@Nullable
		public Object getValue() {
			return this.value;
		}

		/**
		 * Set the type of the constructor argument.
		 */
		public void setType(@Nullable String type) {
			this.type = type;
		}

		/**
		 * Return the type of the constructor argument.
		 */
		@Nullable
		public String getType() {
			return this.type;
		}

		/**
		 * Set the name of the constructor argument.
		 */
		public void setName(@Nullable String name) {
			this.name = name;
		}

		/**
		 * Return the name of the constructor argument.
		 */
		@Nullable
		public String getName() {
			return this.name;
		}

		/**
		 * Set the configuration source {@code Object} for this metadata element.
		 * <p>The exact type of the object will depend on the configuration mechanism used.
		 */
		public void setSource(@Nullable Object source) {
			this.source = source;
		}

		@Override
		@Nullable
		public Object getSource() {
			return this.source;
		}

		/**
		 * Return whether this holder contains a converted value already ({@code true}),
		 * or whether the value still needs to be converted ({@code false}).
		 */
		public synchronized boolean isConverted() {
			return this.converted;
		}

		/**
		 * Set the converted value of the constructor argument,
		 * after processed type conversion.
		 */
		public synchronized void setConvertedValue(@Nullable Object value) {
			this.converted = (value != null);
			this.convertedValue = value;
		}

		/**
		 * Return the converted value of the constructor argument,
		 * after processed type conversion.
		 */
		@Nullable
		public synchronized Object getConvertedValue() {
			return this.convertedValue;
		}

		/**
		 * 是否相等
		 * Determine whether the content of this ValueHolder is equal
		 * to the content of the given other ValueHolder.
		 * <p>Note that ValueHolder does not implement {@code equals}
		 * directly, to allow for multiple ValueHolder instances with the
		 * same content to reside in the same Set.
		 */
		private boolean contentEquals(ValueHolder other) {
			return (this == other ||
					(ObjectUtils.nullSafeEquals(this.value, other.value) && ObjectUtils.nullSafeEquals(this.type, other.type)));
		}

		/**
		 * 确定此ValueHolder的内容的哈希码。
		 *
		 * Determine whether the hash code of the content of this ValueHolder.
		 * <p>Note that ValueHolder does not implement {@code hashCode}
		 * directly, to allow for multiple ValueHolder instances with the
		 * same content to reside in the same Set.
		 */
		private int contentHashCode() {
			return ObjectUtils.nullSafeHashCode(this.value) * 29 + ObjectUtils.nullSafeHashCode(this.type);
		}

		/**
		 * Create a copy of this ValueHolder: that is, an independent
		 * ValueHolder instance with the same contents.
		 */
		public ValueHolder copy() {
			ValueHolder copy = new ValueHolder(this.value, this.type, this.name);
			copy.setSource(this.source);
			return copy;
		}
	}

}

```



# 5. MutablePropertyValues类

MutablePropertyValues类实现了PropertyValues接口。

MutablePropertyValues是PropertyValues接口的默认实现，允许对属性进行简单操作，并提供构造函数以支持Map中的深层复制和构造。



## 5.1 PropertyValue类

PropertyValue用于保存单个bean属性的信息和值的对象。
使用PropertyValue对象，而不是仅将所有属性存储在键为属性名的mao中，可以更灵活，并且能够以更优化的方式处理索引属性等。

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/spring-PropertyValue_uml.png)



```java
/**
 * 用于保存单个bean属性的信息和值的对象
 * 使用PropertyValue对象，而不是仅将所有属性存储在键为属性名的mao中，可以更灵活，并且能够以更优化的方式处理索引属性等。
 *
 * Object to hold information and value for an individual bean property.
 * Using an object here, rather than just storing all properties in
 * a map keyed by property name, allows for more flexibility（灵活性）, and the
 * ability to handle indexed properties etc in an optimized（优化） way.
 *
 * <p>Note that the value doesn't need to be the final required type:
 * A {@link BeanWrapper} implementation should handle any necessary conversion,
 * as this object doesn't know anything about the objects it will be applied to.
 *
 * @author Rod Johnson
 * @author Rob Harrop
 * @author Juergen Hoeller
 * @since 13 May 2001
 * @see PropertyValues
 * @see BeanWrapper
 */
@SuppressWarnings("serial")
public class PropertyValue extends BeanMetadataAttributeAccessor implements Serializable {

	private final String name;

	@Nullable
	private final Object value;

	private boolean optional = false;

	private boolean converted = false;

	@Nullable
	private Object convertedValue;

	/** Package-visible field that indicates whether conversion is necessary. */
	@Nullable
	volatile Boolean conversionNecessary;

	/** Package-visible field for caching the resolved property path tokens. */
	@Nullable
	transient volatile Object resolvedTokens;


	/**
	 * Create a new PropertyValue instance.
	 * @param name the name of the property (never {@code null})
	 * @param value the value of the property (possibly before type conversion)
	 */
	public PropertyValue(String name, @Nullable Object value) {
		Assert.notNull(name, "Name must not be null");
		this.name = name;
		this.value = value;
	}

	/**
	 * Copy constructor.
	 * @param original the PropertyValue to copy (never {@code null})
	 */
	public PropertyValue(PropertyValue original) {
		Assert.notNull(original, "Original must not be null");
		this.name = original.getName();
		this.value = original.getValue();
		this.optional = original.isOptional();
		this.converted = original.converted;
		this.convertedValue = original.convertedValue;
		this.conversionNecessary = original.conversionNecessary;
		this.resolvedTokens = original.resolvedTokens;
		setSource(original.getSource());
		copyAttributesFrom(original);
	}

	/**
	 * Constructor that exposes a new value for an original value holder.
	 * The original holder will be exposed as source of the new holder.
	 * @param original the PropertyValue to link to (never {@code null})
	 * @param newValue the new value to apply
	 */
	public PropertyValue(PropertyValue original, @Nullable Object newValue) {
		Assert.notNull(original, "Original must not be null");
		this.name = original.getName();
		this.value = newValue;
		this.optional = original.isOptional();
		this.conversionNecessary = original.conversionNecessary;
		this.resolvedTokens = original.resolvedTokens;
		setSource(original);
		copyAttributesFrom(original);
	}


	/**
	 * Return the name of the property.
	 */
	public String getName() {
		return this.name;
	}

	/**
	 * Return the value of the property.
	 * <p>Note that type conversion will <i>not</i> have occurred here.
	 * It is the responsibility of the BeanWrapper implementation to
	 * perform type conversion.
	 */
	@Nullable
	public Object getValue() {
		return this.value;
	}

	/**
	 * Return the original PropertyValue instance for this value holder.
	 * @return the original PropertyValue (either a source of this
	 * value holder or this value holder itself).
	 */
	public PropertyValue getOriginalPropertyValue() {
		PropertyValue original = this;
		Object source = getSource();
		while (source instanceof PropertyValue && source != original) {
			original = (PropertyValue) source;
			source = original.getSource();
		}
		return original;
	}

	/**
	 * Set whether this is an optional value, that is, to be ignored
	 * when no corresponding property exists on the target class.
	 * @since 3.0
	 */
	public void setOptional(boolean optional) {
		this.optional = optional;
	}

	/**
	 * Return whether this is an optional value, that is, to be ignored
	 * when no corresponding property exists on the target class.
	 * @since 3.0
	 */
	public boolean isOptional() {
		return this.optional;
	}

	/**
	 * Return whether this holder contains a converted value already ({@code true}),
	 * or whether the value still needs to be converted ({@code false}).
	 */
	public synchronized boolean isConverted() {
		return this.converted;
	}

	/**
	 * Set the converted value of this property value,
	 * after processed type conversion.
	 */
	public synchronized void setConvertedValue(@Nullable Object value) {
		this.converted = true;
		this.convertedValue = value;
	}

	/**
	 * Return the converted value of this property value,
	 * after processed type conversion.
	 */
	@Nullable
	public synchronized Object getConvertedValue() {
		return this.convertedValue;
	}


	@Override
	public boolean equals(@Nullable Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof PropertyValue)) {
			return false;
		}
		PropertyValue otherPv = (PropertyValue) other;
		return (this.name.equals(otherPv.name) &&
				ObjectUtils.nullSafeEquals(this.value, otherPv.value) &&
				ObjectUtils.nullSafeEquals(getSource(), otherPv.getSource()));
	}

	@Override
	public int hashCode() {
		return this.name.hashCode() * 29 + ObjectUtils.nullSafeHashCode(this.value);
	}

	@Override
	public String toString() {
		return "bean property '" + this.name + "'";
	}

}
```




## 5.2 PropertyValues接口

PropertyValues即PropertyValue的集合管理类,MutablePropertyValues是其实现类

```java
public interface PropertyValues extends Iterable<PropertyValue> {

	/**
	 * 迭代器
	 * Return an {@link Iterator} over the property values.
	 * @since 5.1
	 */
	@Override
	default Iterator<PropertyValue> iterator() {
		return Arrays.asList(getPropertyValues()).iterator();
	}

	/**
	 * 并行遍历迭代器
	 *
	 * Return a {@link Spliterator} over the property values.
	 * @since 5.1
	 */
	@Override
	default Spliterator<PropertyValue> spliterator() {
		return Spliterators.spliterator(getPropertyValues(), 0);
	}

	/**
	 * Return a sequential {@link Stream} containing the property values.
	 * @since 5.1
	 */
	default Stream<PropertyValue> stream() {
		return StreamSupport.stream(spliterator(), false);
	}

	/**
	 * Return an array of the PropertyValue objects held in this object.
	 */
	PropertyValue[] getPropertyValues();

	/**
	 * Return the property value with the given name, if any.
	 * @param propertyName the name to search for
	 * @return the property value, or {@code null} if none
	 */
	@Nullable
	PropertyValue getPropertyValue(String propertyName);

	/**
	 * Return the changes since the previous PropertyValues.
	 * Subclasses should also override {@code equals}.
	 * @param old old property values
	 * @return the updated or new properties.
	 * Return empty PropertyValues if there are no changes.
	 * @see Object#equals
	 */
	PropertyValues changesSince(PropertyValues old);

	/**
	 * Is there a property value (or other processing entry) for this property?
	 * @param propertyName the name of the property we're interested in
	 * @return whether there is a property value for this property
	 */
	boolean contains(String propertyName);

	/**
	 * Does this holder not contain any PropertyValue objects at all?
	 */
	boolean isEmpty();

}
```



## 5.3 MutablePropertyValues类

MutablePropertyValues实现了PropertyValues。

```
public class MutablePropertyValues implements PropertyValues, Serializable {

   private final List<PropertyValue> propertyValueList;

   @Nullable
   private Set<String> processedProperties;

   private volatile boolean converted = false;


   /**
    * Creates a new empty MutablePropertyValues object.
    * <p>Property values can be added with the {@code add} method.
    * @see #add(String, Object)
    */
   public MutablePropertyValues() {
      this.propertyValueList = new ArrayList<>(0);
   }

   /**
    * Deep copy constructor. Guarantees PropertyValue references
    * are independent, although it can't deep copy objects currently
    * referenced by individual PropertyValue objects.
    * @param original the PropertyValues to copy
    * @see #addPropertyValues(PropertyValues)
    */
   public MutablePropertyValues(@Nullable PropertyValues original) {
      // We can optimize this because it's all new:
      // There is no replacement of existing property values.
      if (original != null) {
         PropertyValue[] pvs = original.getPropertyValues();
         this.propertyValueList = new ArrayList<>(pvs.length);
         for (PropertyValue pv : pvs) {
            this.propertyValueList.add(new PropertyValue(pv));
         }
      }
      else {
         this.propertyValueList = new ArrayList<>(0);
      }
   }

   /**
    * Construct a new MutablePropertyValues object from a Map.
    * @param original a Map with property values keyed by property name Strings
    * @see #addPropertyValues(Map)
    */
   public MutablePropertyValues(@Nullable Map<?, ?> original) {
      // We can optimize this because it's all new:
      // There is no replacement of existing property values.
      if (original != null) {
         this.propertyValueList = new ArrayList<>(original.size());
         original.forEach((attrName, attrValue) -> this.propertyValueList.add(
               new PropertyValue(attrName.toString(), attrValue)));
      }
      else {
         this.propertyValueList = new ArrayList<>(0);
      }
   }

   /**
    * Construct a new MutablePropertyValues object using the given List of
    * PropertyValue objects as-is.
    * <p>This is a constructor for advanced usage scenarios.
    * It is not intended for typical programmatic use.
    * @param propertyValueList a List of PropertyValue objects
    */
   public MutablePropertyValues(@Nullable List<PropertyValue> propertyValueList) {
      this.propertyValueList =
            (propertyValueList != null ? propertyValueList : new ArrayList<>());
   }


   /**
    * Return the underlying List of PropertyValue objects in its raw form.
    * The returned List can be modified directly, although this is not recommended.
    * <p>This is an accessor for optimized access to all PropertyValue objects.
    * It is not intended for typical programmatic use.
    */
   public List<PropertyValue> getPropertyValueList() {
      return this.propertyValueList;
   }

   /**
    * Return the number of PropertyValue entries in the list.
    */
   public int size() {
      return this.propertyValueList.size();
   }

   /**
    * Copy all given PropertyValues into this object. Guarantees PropertyValue
    * references are independent, although it can't deep copy objects currently
    * referenced by individual PropertyValue objects.
    * @param other the PropertyValues to copy
    * @return this in order to allow for adding multiple property values in a chain
    */
   public MutablePropertyValues addPropertyValues(@Nullable PropertyValues other) {
      if (other != null) {
         PropertyValue[] pvs = other.getPropertyValues();
         for (PropertyValue pv : pvs) {
            addPropertyValue(new PropertyValue(pv));
         }
      }
      return this;
   }

   /**
    * Add all property values from the given Map.
    * @param other a Map with property values keyed by property name,
    * which must be a String
    * @return this in order to allow for adding multiple property values in a chain
    */
   public MutablePropertyValues addPropertyValues(@Nullable Map<?, ?> other) {
      if (other != null) {
         other.forEach((attrName, attrValue) -> addPropertyValue(
               new PropertyValue(attrName.toString(), attrValue)));
      }
      return this;
   }

   /**
    * Add a PropertyValue object, replacing any existing one for the
    * corresponding property or getting merged with it (if applicable).
    * @param pv the PropertyValue object to add
    * @return this in order to allow for adding multiple property values in a chain
    */
   public MutablePropertyValues addPropertyValue(PropertyValue pv) {
      for (int i = 0; i < this.propertyValueList.size(); i++) {
         PropertyValue currentPv = this.propertyValueList.get(i);
         // 已经存在一个同名属性 replacing 或者 merged
         if (currentPv.getName().equals(pv.getName())) {
            pv = mergeIfRequired(pv, currentPv);
            setPropertyValueAt(pv, i);
            return this;
         }
      }
      // 添加一个新的PropertyValue
      this.propertyValueList.add(pv);
      return this;
   }

   /**
    * Overloaded version of {@code addPropertyValue} that takes
    * a property name and a property value.
    * <p>Note: As of Spring 3.0, we recommend using the more concise
    * and chaining-capable variant {@link #add}.
    * @param propertyName name of the property
    * @param propertyValue value of the property
    * @see #addPropertyValue(PropertyValue)
    */
   public void addPropertyValue(String propertyName, Object propertyValue) {
      addPropertyValue(new PropertyValue(propertyName, propertyValue));
   }

   /**
    * Add a PropertyValue object, replacing any existing one for the
    * corresponding property or getting merged with it (if applicable).
    * @param propertyName name of the property
    * @param propertyValue value of the property
    * @return this in order to allow for adding multiple property values in a chain
    */
   public MutablePropertyValues add(String propertyName, @Nullable Object propertyValue) {
      addPropertyValue(new PropertyValue(propertyName, propertyValue));
      return this;
   }

   /**
    * 修改指定位置的值
    *
    * Modify a PropertyValue object held in this object.
    * Indexed from 0.
    */
   public void setPropertyValueAt(PropertyValue pv, int i) {
      this.propertyValueList.set(i, pv);
   }

   /**
    * Merges the value of the supplied 'new' {@link PropertyValue} with that of
    * the current {@link PropertyValue} if merging is supported and enabled.
    * @see Mergeable
    */
   private PropertyValue mergeIfRequired(PropertyValue newPv, PropertyValue currentPv) {
      Object value = newPv.getValue();
      if (value instanceof Mergeable) {
         Mergeable mergeable = (Mergeable) value;
         if (mergeable.isMergeEnabled()) {
            // 可以合并则合并
            Object merged = mergeable.merge(currentPv.getValue());
            return new PropertyValue(newPv.getName(), merged);
         }
      }
      //不可以合并，返回新的PropertyValue
      return newPv;
   }

   /**
    * 根据值来移除
    * Remove the given PropertyValue, if contained.
    * @param pv the PropertyValue to remove
    */
   public void removePropertyValue(PropertyValue pv) {
      this.propertyValueList.remove(pv);
   }

   /**
    * 根据名来移除
    *
    * Overloaded version of {@code removePropertyValue} that takes a property name.
    * @param propertyName name of the property
    * @see #removePropertyValue(PropertyValue)
    */
   public void removePropertyValue(String propertyName) {
      //先拿到值，再根据值来移除
      this.propertyValueList.remove(getPropertyValue(propertyName));
   }


   @Override
   public Iterator<PropertyValue> iterator() {
      return Collections.unmodifiableList(this.propertyValueList).iterator();
   }

   @Override
   public Spliterator<PropertyValue> spliterator() {
      return Spliterators.spliterator(this.propertyValueList, 0);
   }

   @Override
   public Stream<PropertyValue> stream() {
      return this.propertyValueList.stream();
   }

   @Override
   public PropertyValue[] getPropertyValues() {
      return this.propertyValueList.toArray(new PropertyValue[0]);
   }

   @Override
   @Nullable
   public PropertyValue getPropertyValue(String propertyName) {
      for (PropertyValue pv : this.propertyValueList) {
         if (pv.getName().equals(propertyName)) {
            return pv;
         }
      }
      return null;
   }

   /**
    * Get the raw property value, if any.
    * @param propertyName the name to search for
    * @return the raw property value, or {@code null} if none found
    * @since 4.0
    * @see #getPropertyValue(String)
    * @see PropertyValue#getValue()
    */
   @Nullable
   public Object get(String propertyName) {
      PropertyValue pv = getPropertyValue(propertyName);
      return (pv != null ? pv.getValue() : null);
   }

   @Override
   public PropertyValues changesSince(PropertyValues old) {
      MutablePropertyValues changes = new MutablePropertyValues();
      if (old == this) {
         return changes;
      }

      // for each property value in the new set
      for (PropertyValue newPv : this.propertyValueList) {
         // if there wasn't an old one, add it
         PropertyValue pvOld = old.getPropertyValue(newPv.getName());
         if (pvOld == null || !pvOld.equals(newPv)) {
            changes.addPropertyValue(newPv);
         }
      }
      return changes;
   }

   @Override
   public boolean contains(String propertyName) {
      return (getPropertyValue(propertyName) != null ||
            (this.processedProperties != null && this.processedProperties.contains(propertyName)));
   }

   @Override
   public boolean isEmpty() {
      return this.propertyValueList.isEmpty();
   }


   /**
    * Register the specified property as "processed" in the sense
    * of some processor calling the corresponding setter method
    * outside of the PropertyValue(s) mechanism.
    * <p>This will lead to {@code true} being returned from
    * a {@link #contains} call for the specified property.
    * @param propertyName the name of the property.
    */
   public void registerProcessedProperty(String propertyName) {
      if (this.processedProperties == null) {
         this.processedProperties = new HashSet<>(4);
      }
      this.processedProperties.add(propertyName);
   }

   /**
    * Clear the "processed" registration of the given property, if any.
    * @since 3.2.13
    */
   public void clearProcessedProperty(String propertyName) {
      if (this.processedProperties != null) {
         this.processedProperties.remove(propertyName);
      }
   }

   /**
    * Mark this holder as containing converted values only
    * (i.e. no runtime resolution needed anymore).
    */
   public void setConverted() {
      this.converted = true;
   }

   /**
    * Return whether this holder contains converted values only ({@code true}),
    * or whether the values still need to be converted ({@code false}).
    */
   public boolean isConverted() {
      return this.converted;
   }


   @Override
   public boolean equals(@Nullable Object other) {
      return (this == other || (other instanceof MutablePropertyValues &&
            this.propertyValueList.equals(((MutablePropertyValues) other).propertyValueList)));
   }

   @Override
   public int hashCode() {
      return this.propertyValueList.hashCode();
   }

   @Override
   public String toString() {
      PropertyValue[] pvs = getPropertyValues();
      if (pvs.length > 0) {
         return "PropertyValues: length=" + pvs.length + "; " + StringUtils.arrayToDelimitedString(pvs, "; ");
      }
      return "PropertyValues: length=0";
   }

}
```

