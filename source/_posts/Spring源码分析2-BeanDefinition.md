---
title: Spring源码分析2-BeanDefinition
date: 2019-08-31 17:47:20
tags: [java,spring]
categories: spring
---

BeanDefinition描述了一个bean实例，是配置文件\<bean\>元素标签在容器中内部表示形式。

它具有**属性值**，**构造函数参数值**以及具体实现提供的更多信息。
这是一个最小化的接口，主要的意向是允许一个BeanFactoryPostProcessor来修改它的属性值和其他的元数据。


# 1 继承关系

BeanDefinition继承了AttributeAccessor，说明它具有处理属性的能力

BeanDefinition继承了BeanMetadataElement，说明它可以持有**Bean元数据元素**，作用是可以持有XML文件的一个bean标签对应的Object。



继承关系：

![](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/spring_BeanDefinition_uml.png)



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

# 2 BeanDefinition分析

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



# 3 ConstructorArgumentValues类

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

			//返回找到的参数持有者对象
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



# 4 MutablePropertyValues类

待完成