<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">GenericApplicationContext 源码分析</h3>

> 通用的 *ApplicationContext* 实现，它持有一个内部 *org.springframe .beans.factory.support。DefaultListableBeanFactory* 实例，并且不采用特定的bean definition 定义格式。实现了 *org.springframework.beans.factory.support.DefaultListableBeanFactory* 接口，以便允许将任何bean definition定义读取器应用到它。
>
> 与其他为每次刷新创建一个新的内部 BeanFactory 实例的 *ApplicationContext* 实现不同，该上下文的内部BeanFactory从一开始就可用，以便能够在其上注册bean定义 refresh() 只能调用一次。
>
> 对于应该以可刷新方式读取特殊bean definition 定义格式的自定义应用程序上下文实现，请考虑从 *AbstractRefreshableApplicationContext* 基类派生。

```java
package org.springframework.context.support;

import java.io.IOException;
import java.lang.reflect.Constructor;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.function.Supplier;

import org.springframework.beans.BeanUtils;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanDefinitionStoreException;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanDefinitionCustomizer;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.ApplicationContext;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.core.io.support.ResourcePatternResolver;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;

/**
 *
 * 用法:
 *
 * GenericApplicationContext ctx = new GenericApplicationContext();
 * XmlBeanDefinitionReader xmlReader = new XmlBeanDefinitionReader(ctx);
 * xmlReader.loadBeanDefinitions(new ClassPathResource("applicationContext.xml"));
 * PropertiesBeanDefinitionReader propReader = new PropertiesBeanDefinitionReader(ctx);
 * propReader.loadBeanDefinitions(new ClassPathResource("otherBeans.properties"));
 * ctx.refresh();
 *
 * MyBean myBean = (MyBean) ctx.getBean("myBean");
 *
 * @author Juergen Hoeller
 * @author Chris Beams
 * @since 1.1.2
 * @see #registerBeanDefinition
 * @see #refresh()
 * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
 * @see org.springframework.beans.factory.support.PropertiesBeanDefinitionReader
 */
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {
	/** 所依赖的 DefaultListableBeanFactory */
	private final DefaultListableBeanFactory beanFactory;
	/** 资源加载器 */
	@Nullable
	private ResourceLoader resourceLoader;
	/** 是否是自定义类加载器 */
	private boolean customClassLoader = false;
	/** 是否可刷新的标识符 */
	private final AtomicBoolean refreshed = new AtomicBoolean();


	/**
	 * 创建一个新的 GenericApplicationContext。
	 * 同时将工厂设置为 DefaultListableBeanFactory
	 * @see #registerBeanDefinition
	 * @see #refresh
	 */
	public GenericApplicationContext() {
		this.beanFactory = new DefaultListableBeanFactory();
	}

	/**
	 * 创建一个新的 GenericApplicationContext，并指定DefaultListableBeanFactory
	 * @param beanFactory 用于此上下文的DefaultListableBeanFactory实例，不允许为null
	 * @see #registerBeanDefinition
	 * @see #refresh
	 */
	public GenericApplicationContext(DefaultListableBeanFactory beanFactory) {
		Assert.notNull(beanFactory, "BeanFactory must not be null");
		this.beanFactory = beanFactory;
	}

	/**
	 * 创建一个新的 GenericApplicationContext，并指定父上下文
	 * @param parent 父上下文，可为null
	 * @see #registerBeanDefinition
	 * @see #refresh
	 */
	public GenericApplicationContext(@Nullable ApplicationContext parent) {
		this();
		setParent(parent);
	}

	/**
	 * 创建一个新的 GenericApplicationContext，并指定 DefaultListableBeanFactory
	 * @param beanFactory 用于此上下文的DefaultListableBeanFactory实例
	 * @param parent 父上下文
	 * @see #registerBeanDefinition
	 * @see #refresh
	 */
	public GenericApplicationContext(DefaultListableBeanFactory beanFactory, ApplicationContext parent) {
		this(beanFactory);
		setParent(parent);
	}


	/**
	 * 设置此应用程序上下文的父节点，
	 * 也相应地设置内部BeanFactory的父节点。
	 * @see org.springframework.beans.factory.config.ConfigurableBeanFactory#setParentBeanFactory
	 */
	@Override
	public void setParent(@Nullable ApplicationContext parent) {
		super.setParent(parent);
		this.beanFactory.setParentBeanFactory(getInternalParentBeanFactory());
	}

	/**
	 * 设置是否允许它通过注册具有相同名称的不同定义来覆盖bean定义，从而自动替换前者
	 * 。否则，将引发异常。默认设置是“true”。
	 * @since 3.0
	 * @see org.springframework.beans.factory.support.DefaultListableBeanFactory#setAllowBeanDefinitionOverriding
	 */
	public void setAllowBeanDefinitionOverriding(boolean allowBeanDefinitionOverriding) {
		this.beanFactory.setAllowBeanDefinitionOverriding(allowBeanDefinitionOverriding);
	}

	/**
	 * 设置是否允许bean之间的循环引用，并自动尝试解析它们。
	 * 默认为"true". 当遇到循环引用时，关闭它以抛出异常，完全禁止它们。
	 * @since 3.0
	 * @see org.springframework.beans.factory.support.DefaultListableBeanFactory#setAllowCircularReferences
	 */
	public void setAllowCircularReferences(boolean allowCircularReferences) {
		this.beanFactory.setAllowCircularReferences(allowCircularReferences);
	}

	/**
	 * 设置用于此上下文的 ResourceLoader 。
	 * 如果设置，此上下文将所有的 getResource 方法调用委托给指定的 ResourceLoader。
	 * 如果未设置，将应用默认资源加载器。
	 * @see #getResource
	 * @see org.springframework.core.io.DefaultResourceLoader
	 * @see org.springframework.core.io.FileSystemResourceLoader
	 * @see org.springframework.core.io.support.ResourcePatternResolver
	 * @see #getResources
	 */
	public void setResourceLoader(ResourceLoader resourceLoader) {
		this.resourceLoader = resourceLoader;
	}


	//---------------------------------------------------------------------
	// 如果有需要，可以重写 ResourceLoader / ResourcePatternResolver
	//---------------------------------------------------------------------

	/**
	 * 如果设置了此上下文的ResourceLoader，
	 * 则此实现将委托给该上下文的ResourceLoader，返回到默认的超类行为.。
	 * @see #setResourceLoader
	 */
	@Override
	public Resource getResource(String location) {
		if (this.resourceLoader != null) {
			return this.resourceLoader.getResource(location);
		}
		return super.getResource(location);
	}

	/**
	 * 从指定的位置获取资源
	 * @see #setResourceLoader
	 */
	@Override
	public Resource[] getResources(String locationPattern) throws IOException {
		if (this.resourceLoader instanceof ResourcePatternResolver) {
			return ((ResourcePatternResolver) this.resourceLoader).getResources(locationPattern);
		}
		return super.getResources(locationPattern);
	}
	/** 设置类加载器 */
	@Override
	public void setClassLoader(@Nullable ClassLoader classLoader) {
		super.setClassLoader(classLoader);
        // 并将标识符设置为true
		this.customClassLoader = true;
	}

	@Override
	@Nullable
	public ClassLoader getClassLoader() {
		if (this.resourceLoader != null && !this.customClassLoader) {
			return this.resourceLoader.getClassLoader();
		}
		return super.getClassLoader();
	}


	//---------------------------------------------------------------------
	// AbstractApplicationContext 的模板方法实现
	//---------------------------------------------------------------------

	/**
	 * 什么也不做:我们持有一个内部bean工厂，
	 * 并依赖调用者通过我们的公共方法(或bean工厂的方法)注册bean。
	 * @see #registerBeanDefinition
	 */
	@Override
	protected final void refreshBeanFactory() throws IllegalStateException {
		if (!this.refreshed.compareAndSet(false, true)) {
			throw new IllegalStateException(
					"GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
		}
		this.beanFactory.setSerializationId(getId());
	}

	@Override
	protected void cancelRefresh(BeansException ex) {
		this.beanFactory.setSerializationId(null);
		super.cancelRefresh(ex);
	}

	/**
	 * 关闭bean工厂
	 */
	@Override
	protected final void closeBeanFactory() {
		this.beanFactory.setSerializationId(null);
	}

	/**
	 * 返回该上下文中保存的单个内部bean工厂(ConfigurableListableBeanFactory).
	 */
	@Override
	public final ConfigurableListableBeanFactory getBeanFactory() {
		return this.beanFactory;
	}

	/**
	 * 返回此上下文的底层bean工厂，可用于注册bean定义。
	 * 注意：您需要调用 refresh() 来使用应用程序上下文语义初始化bean工厂
	 * 及其包含的bean(自动检测 BeanFactoryPostProcessors 等)。
	 * @return 内部的bean工厂(DefaultListableBeanFactory)
	 */
	public final DefaultListableBeanFactory getDefaultListableBeanFactory() {
		return this.beanFactory;
	}

	@Override
	public AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException {
		assertBeanFactoryActive();
		return this.beanFactory;
	}


	//---------------------------------------------------------------------
	// BeanDefinitionRegistry 的实现
	//---------------------------------------------------------------------

	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		this.beanFactory.registerBeanDefinition(beanName, beanDefinition);
	}

	@Override
	public void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
		this.beanFactory.removeBeanDefinition(beanName);
	}

	@Override
	public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
		return this.beanFactory.getBeanDefinition(beanName);
	}

	@Override
	public boolean isBeanNameInUse(String beanName) {
		return this.beanFactory.isBeanNameInUse(beanName);
	}

	@Override
	public void registerAlias(String beanName, String alias) {
		this.beanFactory.registerAlias(beanName, alias);
	}

	@Override
	public void removeAlias(String alias) {
		this.beanFactory.removeAlias(alias);
	}

	@Override
	public boolean isAlias(String beanName) {
		return this.beanFactory.isAlias(beanName);
	}


	//---------------------------------------------------------------------
	// 注册单个bean的便捷的方法
	//---------------------------------------------------------------------

	/**
	 * 从给定的bean类注册一个bean，可选地定制其bean定义元数据
	 * (通常声明为lambda表达式或方法引用)。
	 * @param beanClass bean的类
	 * @param customizers 自定义工厂的 BeanDefinition 的一个或多个回调，
	 * 例如设置一个延迟初始化或主标志
	 * @since 5.0
	 * @see #registerBean(String, Class, Supplier, BeanDefinitionCustomizer...)
	 */
	public final <T> void registerBean(Class<T> beanClass, BeanDefinitionCustomizer... customizers) {
		registerBean(null, beanClass, null, customizers);
	}

	/**
	* 从给定的bean类注册一个bean，使用给定的提供者
	* 获取一个新实例(通常声明为lambda表达式或方法引用)，
	* 可选地定制其bean定义元数据(同样通常声明为lambda表达式或方法引用)。
	 * @param beanName bean的名称，可为空
	 * @param beanClass bean的类
	 * @param customizers 自定义工厂的 BeanDefinition 的一个或多个回调，
	 * 例如设置一个延迟初始化或主标志
	 * @since 5.0
	 * @see #registerBean(String, Class, Supplier, BeanDefinitionCustomizer...)
	 */
	public final <T> void registerBean(
			@Nullable String beanName, Class<T> beanClass, BeanDefinitionCustomizer... customizers) {

		registerBean(beanName, beanClass, null, customizers);
	}

	/**
	 * 从给定的bean类注册一个bean，使用给定的提供者获取一个新实例
	 * (通常声明为lambda表达式或方法引用)，可选地定制其bean定义元数据
	 * (同样通常声明为lambda表达式或方法引用)。
	 * @param beanClass bean的类
	 * @param supplier 用于创建bean实例的回调
	 * @param customizers 自定义工厂的 BeanDefinition 的一个或多个回调，
	 * 例如设置一个延迟初始化或主标志
	 * @since 5.0
	 * @see #registerBean(String, Class, Supplier, BeanDefinitionCustomizer...)
	 */
	public final <T> void registerBean(
			Class<T> beanClass, Supplier<T> supplier, BeanDefinitionCustomizer... customizers) {

		registerBean(null, beanClass, supplier, customizers);
	}

	/**
	 * 从给定的bean类注册一个bean，使用给定的提供者获取一个新实例
	 * (通常声明为lambda表达式或方法引用)，可选地定制其bean定义元数据
	 * (同样通常声明为lambda表达式或方法引用)。
	 * 可以重写此方法，以适应所有registerBean 方法的注册机制(因为它们都委托给此方法)。
	 * @param beanName bean名称，可能为空
	 * @param beanClass bean的类
	 * @param supplier用于创建bean实例的回调
	 * @param customizers 自定义工厂的 BeanDefinition 的一个或多个回调，
	 * 例如设置一个延迟初始化或主标志
	 * @since 5.0
	 */
	public <T> void registerBean(@Nullable String beanName, Class<T> beanClass,
			@Nullable Supplier<T> supplier, BeanDefinitionCustomizer... customizers) {

		ClassDerivedBeanDefinition beanDefinition = new ClassDerivedBeanDefinition(beanClass);
		if (supplier != null) {
			beanDefinition.setInstanceSupplier(supplier);
		}
		for (BeanDefinitionCustomizer customizer : customizers) {
			customizer.customize(beanDefinition);
		}

		String nameToUse = (beanName != null ? beanName : beanClass.getName());
		registerBeanDefinition(nameToUse, beanDefinition);
	}


	/**
	 * RootBeanDefinition标记子类，用于基于 registerBean 的注册，
	 * 为公共构造函数提供灵活的自动装配。
	 */
	@SuppressWarnings("serial")
	private static class ClassDerivedBeanDefinition extends RootBeanDefinition {

		public ClassDerivedBeanDefinition(Class<?> beanClass) {
			super(beanClass);
		}

		public ClassDerivedBeanDefinition(ClassDerivedBeanDefinition original) {
			super(original);
		}

		@Override
		@Nullable
		public Constructor<?>[] getPreferredConstructors() {
			Class<?> clazz = getBeanClass();
			Constructor<?> primaryCtor = BeanUtils.findPrimaryConstructor(clazz);
			if (primaryCtor != null) {
				return new Constructor<?>[] {primaryCtor};
			}
			Constructor<?>[] publicCtors = clazz.getConstructors();
			if (publicCtors.length > 0) {
				return publicCtors;
			}
			return null;
		}

		@Override
		public RootBeanDefinition cloneBeanDefinition() {
			return new ClassDerivedBeanDefinition(this);
		}
	}
}
```

