> :dizzy:本文基于 `Spring5.0.10.RELEASE` 版本

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">BeanFactory</h3>

![bean](https://images.unsplash.com/photo-1538391382455-5a95e0b549de?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=cc89611e38144bf02dcdb52f637d97cf&auto=format&fit=crop&w=500&q=60)



> *Java* 名称源于 Java咖啡:coffee:，*bean* 意为 “豆” ，JavaBean 意为 “Java咖啡豆”。在Java编程世界了，万物皆对象，*bean* 就是对象的称呼，JavaBean就是java对象。
>
> *Spring* 框架最为核心的莫过于 *IoC容器*，这个容器在 *Spring* 框架中是通过 *Factory工厂* 来生成 *bean对象* 。IoC容器最顶层的对象工厂就是 **BeanFactory** 接口，它定义了生产对象的工厂对基本的行为功能：

```java
package org.springframework.beans.factory;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;
import org.springframework.lang.Nullable;

public interface BeanFactory {
	/** 标识符。如果以&开头表示获取FactoryBean，否则是FactoryBean创建的实例 */
	String FACTORY_BEAN_PREFIX = "&";
    
    // --------------------- 获取bean对象实例 ---------------------
	/** 根据bean名称获取bean对象实例 */
	Object getBean(String name) throws BeansException;
	/** 根据bean名称和必须的bean类型获取bean对象实例 */
	<T> T getBean(String name, @Nullable Class<T> requiredType) throws BeansException;
	/** 根据bean名称和参数来获取bean对象实例 */
	Object getBean(String name, Object... args) throws BeansException;
	/** 根据类型获取对象实例 */
	<T> T getBean(Class<T> requiredType) throws BeansException;
	/** 根据类型和参数获取对象实例 */
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
    
    // ------------------ 判断是否存在bean对象实例 -----------------
	/** 根据名称判断是否存在相同的bean对象 */
	boolean containsBean(String name);
    
    // --------------- 判断是对象是单例模式还是原型模式 ---------------
    /** 判断创建对象的模式是否是单例模式 */
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
	 /** 判断创建对象的模式是否是原型模式 */
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
    
    // ---------------------- 判断类型是否匹配 ----------------------
	/**
	 * 判断指定bean的名称和可解析类型是否匹配。
	 * ResolvableType 类是为了解决泛型丢失问题而发明的一个类，它可以很好的解决泛化参数问题。
	 */
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
	/** 指定名称的bean的类型是否匹配 */
	boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
    
    // ---------------------------- 其他 ----------------------------
	/** 根据bean名称获取其类型 */
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	/** 根据bean名称获取其别名 */
	String[] getAliases(String name);
}

```

通过上面 *BeanFactory* 接口的源码可以看出，它定义了 **获取、判断** bean的基本行为，并未涉及创建bean的行为。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">bean工厂体系结构</h3>

BeanFactory接口和子接口、子实现类以及其他接口、类共同组成了一个对象工厂与应用上下文体系：

![BeanFactory]()

丰富的 *工厂与应用上下文体系* 不仅仅具有生产bean对象这么单一的工作，它还负责查看Bean的个数、获取Bean的配置、Bean的属性编辑、设置类装载器，而且提供了基于Xml、注解的等多种依赖关系映射处理，当前还有其它更多的功能。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">HierarchicalBeanFactory接口</h3>

*HierarchicalBeanFactory* 接口继承于 *BeanFactory* 接口，是bean工厂分层结构的一部分。可参见*org.springframework.beans.factory.config.ConfigurableBeanFactory#setParentBeanFactory* 设置父级bean工厂的方法。

```java
package org.springframework.beans.factory;

import org.springframework.lang.Nullable;

public interface HierarchicalBeanFactory extends BeanFactory {
	/** 返回其父级bean工厂，如果不存在则返回null。 */
	@Nullable
	BeanFactory getParentBeanFactory();
	/**
     * 返回本地bean工厂是否包含给定名称的bean，忽略祖先上下文中定义的bean。
     * 这是一种替代bean的方法，忽略来自祖先bean工厂的给定名称的bean。
	 * @param name 要查询的bean的名称
	 * @return 具有指定名称的bean是否在本地工厂中定义
	 * @see BeanFactory#containsBean
	 */
	boolean containsLocalBean(String name);
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">ListableBeanFactory接口</h3>

> *ListableBeanFactory* 也是 *BeanFactory* 接口的子接口，它提供工厂容器内bean实例对象的可罗列功能，但不会列举父容器内的实例对象。

```java
package org.springframework.beans.factory;

import java.lang.annotation.Annotation;
import java.util.Map;
import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;
import org.springframework.lang.Nullable;

public interface ListableBeanFactory extends BeanFactory {
	/** 判断是否包含指定bean名称的bean */
	boolean containsBeanDefinition(String beanName);
	/** 获取当前工厂中bean实例对象的数量 */
	int getBeanDefinitionCount();
	/** 获取当前工厂中bean的名称，以String数组的集合形式返回 */
	String[] getBeanDefinitionNames();
	/** 获取指定可解析类型的bean名称列表，以String数组的形式返回。 */
	String[] getBeanNamesForType(ResolvableType type);
	/** 获取指定类型的bean名称列表，以String数组的形式返回。 */
	String[] getBeanNamesForType(@Nullable Class<?> type);
	/** 获取指定类型的、是否单例模式的、是否允许被初始化的bean名称列表，以String数组的形式返回。 */
	String[] getBeanNamesForType(@Nullable Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);
	/** 获取指定类型的bean集合，以Map的形式返回 */
	<T> Map<String, T> getBeansOfType(@Nullable Class<T> type) throws BeansException;
	/** 获取指定类型的、是否单例模式下的、是否允许初始化的bean对象实例的集合，以Map的方式返回 */
	<T> Map<String, T> getBeansOfType(@Nullable Class<T> type, boolean includeNonSingletons, boolean allowEagerInit)
			throws BeansException;
	/** 根据指定的注解类型，获取bean名称的集合，以String数组的方式返回 */
	String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType);
	/** 根据指定的注解类型，获取bean对象实例的集合，以Map的方式返回 */
	Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException;
	/** 查找指定bean名称的、指定注解类型的注解 */
	@Nullable
	<A extends Annotation> A findAnnotationOnBean(String beanName, Class<A> annotationType)
			throws NoSuchBeanDefinitionException;

}
```

*ListableBeanFactory* 接口的意义在于罗列关于bean的集合。