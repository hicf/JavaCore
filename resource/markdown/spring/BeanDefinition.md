#### bean元数据元素
> 何为元数据？用于描述数据的数据。

```java
package org.springframework.beans;

import org.springframework.lang.Nullable;

/**
 * 由携带源配置对象的bean元数据元素实现此接口。
 *
 * @author Juergen Hoeller
 * @since 2.0
 */
public interface BeanMetadataElement {

	/**
	 * 返回此元数据元素的配置源对象
	 * (may be {@code null}).
	 */
	@Nullable
	Object getSource();

}

```



#### 属性访问器
>用于访问bean的属性。

```java
package org.springframework.core;

import org.springframework.lang.Nullable;

/**
 * 接口定义用于向任意对象增加和访问元数据的通用契约。
 *
 * @author Rob Harrop
 * @since 2.0
 */
public interface AttributeAccessor {

	/**
	 * 通过名称设置属性值。
	 * 通常，用户应该使用完全限定的名称(可能使用类或包名称作为前缀)来防止与其他元数据属性重叠。
	 * @param name 唯一属性键
	 * @param value 要附加的属性值
	 */
	void setAttribute(String name, @Nullable Object value);

	/**
	 * 获取由名称标识的属性的值
	 * 如果属性值不存在，则返回null
	 * @param name 唯一属性键
	 * @return 如果有的话，就返回属性的当前值
	 */
	@Nullable
	Object getAttribute(String name);

	/**
	 * 删除由名称标识的属性并返回其值。
	 * 如果未找到名称下的属性，则返回NULL。
	 * @param name 唯一属性键
	 * @return 如果有的话，返回最后一个属性值
	 */
	@Nullable
	Object removeAttribute(String name);

	/**
	 * 判断指定属性名称对应的数值是否存在
	 * @param name 唯一的属性键
	 */
	boolean hasAttribute(String name);

	/**
	 * 返回所有属性的名称。
	 */
	String[] attributeNames();
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">BeanDefinition</h3>

> *BeanDefinition* 用于描述一个Bean的实例对象，它具有属性值、构造方法参数值和由它提供的进一步信息
>
> 具体的实现。这只是一个最小的接口:主要目的是允许 *BeanFactoryPostProcessor* 这样的*PropertyPlaceholderConfigurer* 来自行修改属性值和其他bean元数据。

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.BeanMetadataElement;
import org.springframework.beans.MutablePropertyValues;
import org.springframework.core.AttributeAccessor;
import org.springframework.lang.Nullable;

/**
 * @author Juergen Hoeller
 * @author Rob Harrop
 * @since 19.03.2004
 * @see ConfigurableListableBeanFactory#getBeanDefinition
 * @see org.springframework.beans.factory.support.RootBeanDefinition
 * @see org.springframework.beans.factory.support.ChildBeanDefinition
 */
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

	/**
	 * 标准单例范围的范围标识符: "singleton".
	 * 请注意，扩展的bean工厂可能支持更多的范围。
	 * @see #setScope
	 */
	String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;

	/**
	 * 标准原型范围的范围标识符: "prototype".
	 * 请注意，扩展的bean工厂可能支持更多的范围。
	 * @see #setScope
	 */
	String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;


	/**
	 * bean对象的角色表示，0表示是bean定义是应用程序的主要部分。通常对应于用户定义的bean。
	 */
	int ROLE_APPLICATION = 0;

	/** 表示角色是 较大的配置(通常是外部配置) */
	int ROLE_SUPPORT = 1;

	/** 内部使用的角色 */
	int ROLE_INFRASTRUCTURE = 2;


	// 可更改的属性

	/** 如果当前bean描述器存在父描述器名称的话，就设置父描述器的名称 */
	void setParentName(@Nullable String parentName);

	/** 如果当前bean描述器存在父描述器名称的话，就设获取父描述器的名称 */
	@Nullable
	String getParentName();

	/**
	 * 指定此bean定义的bean类名。
	 * 类名可以在bean工厂的后置处理过程中修改，通常用经过解析的类名变体替换原来的类名。
	 * @see #setParentName
	 * @see #setFactoryBeanName
	 * @see #setFactoryMethodName
	 */
	void setBeanClassName(@Nullable String beanClassName);

	/**
	 * 返回此bean描述器类名。
	 * 注意，如果子类定义覆盖/继承了父类的类名，那么它不必是运行时使用的实际类名。
	 * 而且，这可能只是调用工厂方法的类，或者在调用方法的工厂bean引用的情况下，它甚至可能是空的。
	 * 因此，而不是认为它是运行时的最终bean类型，而只是将其用于在单个bean定义级别进行解析。
	 * @see #getParentName()
	 * @see #getFactoryBeanName()
	 * @see #getFactoryMethodName()
	 */
	@Nullable
	String getBeanClassName();

	/**
	 * 指定一个新的作用域，用以覆盖当前bean的作用域
	 * @see #SCOPE_SINGLETON
	 * @see #SCOPE_PROTOTYPE
	 */
	void setScope(@Nullable String scope);

	/**
	 * 获取当前bean的作用域名称
	 * 如果不存在的话，则返回null
	 */
	@Nullable
	String getScope();

	/**
	 * 设置是否应该延迟初始化此bean。
	 * 如果设置为false，bean将在启动时由执行单例初始化的bean工厂进行实例化。
	 */
	void setLazyInit(boolean lazyInit);

	/** 返回是否延迟初始化此bean */
	boolean isLazyInit();

	/**
	 * 设置此bean依赖于初始化的bean的名称。
	 * bean工厂将保证首先初始化这些bean。
	 */
	void setDependsOn(@Nullable String... dependsOn);

	/** 返回此bean所依赖的bean名称 */
	@Nullable
	String[] getDependsOn();

	/**
	 * 设置此bean是否是自动连接到其他bean的候选bean。
	 * 注意，此标志仅用于影响基于类型的自动连接。
	 * 它不影响按名称显式引用，即使指定的bean没有标记为自动连接候选，也会解析该引用。
	 * 因此，如果名称匹配，按名称自动连接将注入bean。
	 */
	void setAutowireCandidate(boolean autowireCandidate);

	/**
	 * 返回此bean是否是自动连接到其他bean的候选bean。
	 */
	boolean isAutowireCandidate();

	/**
	 * 设置此bean是否为主要自动连接候选bean。
	 * 如果这个值是true，那么对于多个匹配候选bean中会选择其中一个bean，它将起到关键作用。
	 */
	void setPrimary(boolean primary);

	/** 返回此bean是否为主要自动连接候选对象 */
	boolean isPrimary();

	/**
	 * 也可以通过此方式来设置指定的bean工厂。
	 * 只需要传入被指的的bean工厂字符串名称即可
	 * @see #setFactoryMethodName
	 */
	void setFactoryBeanName(@Nullable String factoryBeanName);

	/**
	 * 如果该bean有bean工厂的话，可以通过此方法获取
	 */
	@Nullable
	String getFactoryBeanName();

	/**
	 * （如果有工厂的方法）可以指定工厂方法。
	 * 将使用构造方法参数调用此方法，如果没有指定参数，则不使用参数。
	 * (如果有的话)方法将在指定的工厂bean上调用，或者作为本地bean类上的静态方法调用。
	 * @see #setFactoryBeanName
	 * @see #setBeanClassName
	 */
	void setFactoryMethodName(@Nullable String factoryMethodName);

	/** 如果此bean有工厂方法的化话，此方法可以返回这个工厂方法 */
	@Nullable
	String getFactoryMethodName();

	/**
	 * 返回此bean的构造方法参数值
	 * 返回的实例可以在bean工厂后处理期间修改。
	 * @return the ConstructorArgumentValues 对象(非null)
	 */
	ConstructorArgumentValues getConstructorArgumentValues();

	/**
	 * 如果此bean定义了构造函数参数值，则返回true。
	 * @since 5.0.2
	 */
	default boolean hasConstructorArgumentValues() {
		return !getConstructorArgumentValues().isEmpty();
	}

	/**
	 * 返回要应用于bean的新实例的属性值。
	 * 返回的实例可以在bean工厂后置处理期间修改。
	 * @return the MutablePropertyValues 对象(非null)
	 */
	MutablePropertyValues getPropertyValues();

	/**
	 * 如果为该bean定义了属性值，则返回true。
	 * @since 5.0.2
	 */
	default boolean hasPropertyValues() {
		return !getPropertyValues().isEmpty();
	}


	// 只读的属性

	/**
	 * 判断此bean的作用域是否是单例作用域
	 * @see #SCOPE_SINGLETON
	 */
	boolean isSingleton();

	/**
	 * 判断此bean的作用域是否是原型作用域
	 * @since 3.0
	 * @see #SCOPE_PROTOTYPE
	 */
	boolean isPrototype();

	/** 返回此bean是否“抽象”，即不打算实例化。 */
	boolean isAbstract();

	/**
	 * 获取这个bean的角色
	 * @see #ROLE_APPLICATION
	 * @see #ROLE_SUPPORT
	 * @see #ROLE_INFRASTRUCTURE
	 */
	int getRole();

	/**
	 * 返回此bean定义的可读描述。
	 */
	@Nullable
	String getDescription();

	/**
	 * 获取此bean定义的资源描述(以便在出现错误时显示在上下文)。
	 */
	@Nullable
	String getResourceDescription();

	/**
	 * 返回原始的BeanDefinition，如果没有，则返回null
	 * 允许检索修饰后的bean定义
	 */
	@Nullable
	BeanDefinition getOriginatingBeanDefinition();

}
```

