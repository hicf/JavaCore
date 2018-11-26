<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">SingletonBeanRegistry 接口</h3>

> *SingletonBeanRegistry* 定义共享bean实例注册表的接口。可以通过 *org.springframework.beans.factory.BeanFactory* 实现以统一方式去暴露出它们的单例管理器。*ConfigurableBeanFactory* 扩展了此接口。

```java
package org.springframework.beans.factory.config;

import org.springframework.lang.Nullable;

/**
 * @author Juergen Hoeller
 * @since 2.0
 * @see ConfigurableBeanFactory
 * @see org.springframework.beans.factory.support.DefaultSingletonBeanRegistry
 * @see org.springframework.beans.factory.support.AbstractBeanFactory
 */
public interface SingletonBeanRegistry {

	/**
	 * 在bean注册表中以给定的bean名称将给定的现有对象注册为单例。
	 * 给定的实例应该是完全被实例化的。
	 * 注册表不会执行任何初始化回调
	 * （尤其是，它不会调用InitializingBean 的 afterPropertiesSet 方法）。
	 * 给定实例也不会接收任何销毁回调（如DisposableBean的 destroy 方法）。
	 * 当运行在一个完整的BeanFactory：
	 * 如果bean应该接收初始化/销毁的回调，则注册bean定义而不是现有的实例。
	 * 通常在注册表配置期间调用，但是也可以用于单例的运行时注册。
	 * 因此，注册表实现应该同步单例访问；
	 * 如果它支持BeanFactory的单例延迟初始化，则无论如何都必须同步单例访问。
	 * @param beanName bean的名称
	 * @param singletonObject 已有的单例对象
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.DisposableBean#destroy
	 * @see org.springframework.beans.factory.support.BeanDefinitionRegistry#registerBeanDefinition
	 */
	void registerSingleton(String beanName, Object singletonObject);

	/**
	 * 返回以给定名称对应已注册的(未加工的)单例对象。
	 * 只检查已经实例化的单例；不返回尚未实例化的单例bean定义的对象。
	 * 此方法的主要目的是访问手动注册的单例（参见 #registerSingleton 方法）。
	 * 还可以用于以原始方式访问由已经创建的bean定义定义的单例。
	 * 注意：此查找方法不知道 FactoryBean 的前缀或别名。
	 * 在获得单例实例之前，需要首先解析规范bean名称。
	 * @param beanName 要查找的bean的名称
	 * @return 注册的单例对象，如果没有找到，则返回null
	 * @see ConfigurableListableBeanFactory#getBeanDefinition
	 */
	@Nullable
	Object getSingleton(String beanName);

	/**
	 * 检查此注册表是否包含具有给定名称的单例实例。
	 * 仅当已存在对应的单例对象，则返回true
	 * 其他注意事项参考 getSingleton 方法的描述。
	 * @see #registerSingleton
	 * @see org.springframework.beans.factory.ListableBeanFactory#containsBeanDefinition
	 * @see org.springframework.beans.factory.BeanFactory#containsBean
	 */
	boolean containsSingleton(String beanName);

	/**
	 * 返回在这个注册中心注册的单例bean的名称，以字符串数组的形式返回。
	 * 其他注意事项参考 getSingleton 方法的描述。
	 * @see #registerSingleton
	 * @see org.springframework.beans.factory.support.BeanDefinitionRegistry#getBeanDefinitionNames
	 * @see org.springframework.beans.factory.ListableBeanFactory#getBeanDefinitionNames
	 */
	String[] getSingletonNames();

	/**
	 * 返回注册表中注册的单例bean的数量。
	 * 其他注意事项参考 getSingleton 方法的描述。
	 * @see #registerSingleton
	 * @see org.springframework.beans.factory.support.BeanDefinitionRegistry#getBeanDefinitionCount
	 * @see org.springframework.beans.factory.ListableBeanFactory#getBeanDefinitionCount
	 */
	int getSingletonCount();

	/**
	 * 返回注册表(用于外部协作者)使用的单例互斥锁。返回这个注册表使用的单例互斥量(用于外部协调)
	 * @return 互斥对象，非null
	 * @since 4.2
	 */
	Object getSingletonMutex();

}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">DefaultSingletonBeanRegistry接口</h3>

> *SingletonBeanRegistry* 接口的默认实现类。允许注册为注册表中所有调用者共享的单例实例，这些单例实例将通过bean名称获得。
>
> 还支持在注册表关闭时销毁 ~~org.springframework.beans.factory.DisposableBean~~ 的实例(可能对应于已注册的单例)。可以注册bean之间的依赖关系来强制执行适当的关闭顺序。
>
> 这个类主要用作 org.springframework.beans.factory.BeanFactory 接口实现的基类，分解出单例bean实例的公共管理行为。
>
> 注意， org.springframework.beans.factory.config.ConfigurableBeanFactory 接口也扩展了 SingletonBeanRegistry 接口。
>
> 请注意，这个类既不假定bean定义的概念，也不假定bean实例的特定创建过程，这与 *AbstractBeanFactory* 和 *DefaultListableBeanFactory* (继承自它)形成了对比。也可以作为一个嵌套的助手来委托。

```java
package org.springframework.beans.factory.support;

import java.util.Collections;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.LinkedHashSet;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

import org.springframework.beans.factory.BeanCreationException;
import org.springframework.beans.factory.BeanCreationNotAllowedException;
import org.springframework.beans.factory.BeanCurrentlyInCreationException;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.config.SingletonBeanRegistry;
import org.springframework.core.SimpleAliasRegistry;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;
import org.springframework.util.StringUtils;

/**
 * @author Juergen Hoeller
 * @since 2.0
 * @see #registerSingleton
 * @see #registerDisposableBean
 * @see org.springframework.beans.factory.DisposableBean
 * @see org.springframework.beans.factory.config.ConfigurableBeanFactory
 */
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

	/** 单例对象的缓存容器：从bean名称映射其bean实例对象 */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	/** 单例工厂的缓存容器：从bean名称映射其 ObjectFactory. */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	/** 早期单例对象的缓存: 从bean名称映射其bean实例对象 */
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

	/** 已注册的单例对象bean名称Set集：按注册顺序存放bean名称。 */
	private final Set<String> registeredSingletons = new LinkedHashSet<>(256);

	/** 当前正在创建的bean的名称。 */
	private final Set<String> singletonsCurrentlyInCreation =
			Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** 当前在创建检查中被排除的bean的名称。 */
	private final Set<String> inCreationCheckExclusions =
			Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** suppressedExceptions异常列表，可用于关联相关原因。 */
	@Nullable
	private Set<Exception> suppressedExceptions;

	/** 当前是否正在处于destroySingletons销毁单例对象的状态标识符。 */
	private boolean singletonsCurrentlyInDestruction = false;

	/** 可被销毁的bean实例：bean名称与实例的映射 */
	private final Map<String, Object> disposableBeans = new LinkedHashMap<>();

	/** 包含bean名称的映射Map:bean名称与包含bean的bean名称Set集合的映射。 */
	private final Map<String, Set<String>> containedBeanMap = new ConcurrentHashMap<>(16);

	/** bean与bean之间存在依赖，用map映射来表示其依赖关系 */
	private final Map<String, Set<String>> dependentBeanMap = new ConcurrentHashMap<>(64);

	/** 一个bean依赖多个bean，这些多个bean构成一个Set集合，然后用Map存放(映射)它们的依赖关系 */
	private final Map<String, Set<String>> dependenciesForBeanMap = new ConcurrentHashMap<>(64);


    /** 通过bean的名称将bean对象注册到存放单例对象的缓存容器singletonObjects */
	@Override
	public void registerSingleton(String beanName, Object singletonObject) throws IllegalStateException {
		Assert.notNull(beanName, "Bean name must not be null");
		Assert.notNull(singletonObject, "Singleton object must not be null");
        // 锁住缓存容器singletonObjects
		synchronized (this.singletonObjects) {
            // 根据要注册的bean名称来说判断缓存容器singletonObjects中存在已存在此bean
			Object oldObject = this.singletonObjects.get(beanName);
			if (oldObject != null) {
                // 如果存在，直接抛出异常，
                // 一个注册表singletonObjects只允许存在一个bean对象（准确的说是一个bean名称）
				throw new IllegalStateException("Could not register object [" + singletonObject +
						"] under bean name '" + beanName + "': there is already object [" + oldObject + "] bound");
			}
            // 如果缓存容器singletonObjects没有该bean名称对应的bean对象，
            // 则将其添加到缓存容器singletonObjects中
			addSingleton(beanName, singletonObject);
		}
	}

	/**
	 * 将给定的单例对象添加到该工厂的单例缓存中。
	 * 被要求紧急注册单例。
	 * @param beanName bean名称
	 * @param singletonObject the singleton object
	 */
	protected void addSingleton(String beanName, Object singletonObject) {
        // 加锁操作
		synchronized (this.singletonObjects) {
            // 添加到单例对象缓存中
			this.singletonObjects.put(beanName, singletonObject);
            // 从单例工厂容器中删除该bean映射项
			this.singletonFactories.remove(beanName);
            // 从早期单例对象的缓存中删除
			this.earlySingletonObjects.remove(beanName);
            // 将bean名称添加到已注册的单例对象bean名称Set集
			this.registeredSingletons.add(beanName);
		}
	}

	/**
	 * 如果有需要的话，添加指定的单例工厂来构建指定的单例对象。
	 * 被要求注册的单例。
	 * 比如：能够解析循环引用。
	 * @param beanName bean名称
	 * @param singletonFactory 单例对象的工厂（不允许为null）
	 */
	protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
            // 如果当前单例对象的缓存容器不包括此bean名称，则进行下面的操作，
            // 否则删除都不操作
			if (!this.singletonObjects.containsKey(beanName)) {
                // 添加到单例工厂缓存中
				this.singletonFactories.put(beanName, singletonFactory);
                // 从早期单例对象的缓存中删除
				this.earlySingletonObjects.remove(beanName);
                // 将bean名称添加到已注册的单例对象bean名称Set集
				this.registeredSingletons.add(beanName);
			}
		}
	}

	@Override
	@Nullable
	public Object getSingleton(String beanName) {
		return getSingleton(beanName, true);
	}

	/**
	 * 返回以给定名称注册的(未加工的)单例对象。
	 * 检查已经实例化的单例对象，并允许对当前创建的单例对象进行前期引用(解析循环引用)。
	 * @param beanName 要查找的bean的名称
	 * @param allowEarlyReference 是否应该创建前期引用
	 * @return 注册的单例对象，如果没有找到，则返回为null
	 */
	@Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        // 首先根据bean名称获取bean的实例对象
		Object singletonObject = this.singletonObjects.get(beanName);
        // 如果实例对象为null，并且该实例对象正在创建中
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
                // 判断是否是前期引用
				singletonObject = this.earlySingletonObjects.get(beanName);
                // singletonObject == null 意味着不是前期单例对象
				if (singletonObject == null && allowEarlyReference) {
                    // 然后从单例工厂缓存容器中查找
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                    // 然后再判断是否存在单例工厂缓存中
					if (singletonFactory != null) {
                        // 如果存在单例工厂缓存中，则获取单例对象
						singletonObject = singletonFactory.getObject();
                        // 并它这个工厂单例对象放入前期对象缓存池中
						this.earlySingletonObjects.put(beanName, singletonObject);
                        // 放完之后，就要重单例工厂缓存池中，将这个单例工厂对象删除
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}

	/**
	 * 返回以给定名称注册的(未加工的)单例对象，
	 * 如果没有的话，就注册一个
	 * @param beanName bean的名称
	 * @param singletonFactory 对象工厂来延迟创建单例对象
	 * with, if necessary
	 * @return 已注册的单例对象
	 */
	public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
			Object singletonObject = this.singletonObjects.get(beanName);
            // 检查是否存在此单例对象
			if (singletonObject == null) {
                // 如果不存在，检查该对象当前是否正在销毁中
				if (this.singletonsCurrentlyInDestruction) {
					throw new BeanCreationNotAllowedException(beanName,
							"Singleton bean creation not allowed while singletons of this factory are in destruction " +
							"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
				}
                // 单例对象创建之前的回调处理（详见下面的源码分析）
				beforeSingletonCreation(beanName);
                // 是否新建单例对象的标识
				boolean newSingleton = false;
                // 记录异常的容器不存在的标识符
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				if (recordSuppressedExceptions) {
                    // 如果不存在则创建一个容器
					this.suppressedExceptions = new LinkedHashSet<>();
				}
				try {
                    // 获取单例工厂
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					// 单例对象是否同时隐式出现 ->
					// 如果是，则继续进行，因为异常指示该状态。
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
					if (recordSuppressedExceptions) {
						this.suppressedExceptions = null;
					}
                    // 单例对象创建之前的回调处理（详见下面的源码分析）
					afterSingletonCreation(beanName);
				}
				if (newSingleton) {
                    // 如果是新建的单例对象则添加到缓存容器中
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}

	/** 记录异常 */
	protected void onSuppressedException(Exception ex) {
		synchronized (this.singletonObjects) {
			if (this.suppressedExceptions != null) {
				this.suppressedExceptions.add(ex);
			}
		}
	}

	/**
	 * 从该工厂的单例缓存中删除具有给定名称的bean，
	 * 如果创建失败，则能够清除单例注册。
	 * @param beanName bean名称
	 * @see #getSingletonMutex()
	 */
	protected void removeSingleton(String beanName) {
		synchronized (this.singletonObjects) {
            // 关于集合对象的意义见上文
			this.singletonObjects.remove(beanName);
			this.singletonFactories.remove(beanName);
			this.earlySingletonObjects.remove(beanName);
			this.registeredSingletons.remove(beanName);
		}
	}

	@Override
	public boolean containsSingleton(String beanName) {
		return this.singletonObjects.containsKey(beanName);
	}

	@Override
	public String[] getSingletonNames() {
		synchronized (this.singletonObjects) {
			return StringUtils.toStringArray(this.registeredSingletons);
		}
	}

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
	 * 检查指的bean名称是否正在创建中
	 * (整个工厂).
	 * @param beanName bean名称
	 */
	public boolean isSingletonCurrentlyInCreation(String beanName) {
		return this.singletonsCurrentlyInCreation.contains(beanName);
	}

	/**
	 * 单例对象创建之前的回调。
	 * 默认实现注册为单例对象
	 * @param beanName 将要被创建的单例对象的名称
	 * @see #isSingletonCurrentlyInCreation
	 */
	protected void beforeSingletonCreation(String beanName) {
		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
			throw new BeanCurrentlyInCreationException(beanName);
		}
	}

	/**
	 * 单例对象创建之后的回调。
	 * 默认实现标记此单例对象不再创建
	 * @param beanName 已创建的单例的名称
	 * @see #isSingletonCurrentlyInCreation
	 */
	protected void afterSingletonCreation(String beanName) {
		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
			throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
		}
	}

	/**
	 * 将给定的bean添加到此注册表中的可被销毁的bean列表中。
	 * @param beanName bean名称
	 * @param bean bean实例
	 */
	public void registerDisposableBean(String beanName, DisposableBean bean) {
		synchronized (this.disposableBeans) {
			this.disposableBeans.put(beanName, bean);
		}
	}

	/**
	 * 在两个bean之间建立一个包含关系，
	 * 在内部bean与其包含外部bean之间。
	 * 还根据销毁顺序将包含的bean注册为依赖于包含的bean。
	 * @param containedBeanName 被包含(内部)bean的名称
	 * @param containingBeanName 所包含(外部)bean的名称
	 * @see #registerDependentBean 
	 */
	public void registerContainedBean(String containedBeanName, String containingBeanName) {
		synchronized (this.containedBeanMap) {
			Set<String> containedBeans =
					this.containedBeanMap.computeIfAbsent(containingBeanName, k -> new LinkedHashSet<>(8));
			if (!containedBeans.add(containedBeanName)) {
				return;
			}
		}
		registerDependentBean(containedBeanName, containingBeanName);
	}

	/**
	 * 为给定的bean注册一个依赖bean
	 * 依赖bean在给定的bean销毁前销毁
	 * @param beanName bean的名称
	 * @param dependentBeanName 依赖bean的名称
	 */
	public void registerDependentBean(String beanName, String dependentBeanName) {
		String canonicalName = canonicalName(beanName);

		synchronized (this.dependentBeanMap) {
			Set<String> dependentBeans =
					this.dependentBeanMap.computeIfAbsent(canonicalName, k -> new LinkedHashSet<>(8));
			if (!dependentBeans.add(dependentBeanName)) {
				return;
			}
		}

		synchronized (this.dependenciesForBeanMap) {
			Set<String> dependenciesForBean =
					this.dependenciesForBeanMap.computeIfAbsent(dependentBeanName, k -> new LinkedHashSet<>(8));
			dependenciesForBean.add(canonicalName);
		}
	}

	/**
	 * 检查指的bean与依赖bean是否存在依赖关系（或者传递依赖关系）。
	 * @param beanName 要检查的bean的名称
	 * @param dependentBeanName 依赖bean的名称
	 * @since 4.0
	 */
	protected boolean isDependent(String beanName, String dependentBeanName) {
		synchronized (this.dependentBeanMap) {
			return isDependent(beanName, dependentBeanName, null);
		}
	}

	private boolean isDependent(String beanName, String dependentBeanName, @Nullable Set<String> alreadySeen) {
		if (alreadySeen != null && alreadySeen.contains(beanName)) {
			return false;
		}
        // 从依赖缓存容器中查找、比对
		String canonicalName = canonicalName(beanName);
		Set<String> dependentBeans = this.dependentBeanMap.get(canonicalName);
		if (dependentBeans == null) {
			return false;
		}
		if (dependentBeans.contains(dependentBeanName)) {
			return true;
		}
		for (String transitiveDependency : dependentBeans) {
			if (alreadySeen == null) {
				alreadySeen = new HashSet<>();
			}
			alreadySeen.add(beanName);
			if (isDependent(transitiveDependency, dependentBeanName, alreadySeen)) {
				return true;
			}
		}
		return false;
	}

	/**
	 * 检查bean是否有依赖的bean
	 * @param beanName 要检查的bean的名称
	 */
	protected boolean hasDependentBean(String beanName) {
		return this.dependentBeanMap.containsKey(beanName);
	}

	/**
	 * （如果存在的话，）返回依赖于指定bean的所有bean的名称的集合。
	 * （谁依赖于此指定bean，就返回谁）
	 * @param beanName bean名称
	 * @return 依赖bean名称的数组，如果没有，则为空数组
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
	 * （如果存在的话，）返回指定bean所依赖的所有bean的名称
	 * （指定的bean依赖谁，就返回谁）
	 * @param beanName bean的名称
	 * @return bean所依赖的bean名称数组，如果没有，则为空数组
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
	/** 销毁bean集合 */
	public void destroySingletons() {
		if (logger.isTraceEnabled()) {
			logger.trace("Destroying singletons in " + this);
		}
		synchronized (this.singletonObjects) {
			this.singletonsCurrentlyInDestruction = true;
		}

		String[] disposableBeanNames;
		synchronized (this.disposableBeans) {
			disposableBeanNames = StringUtils.toStringArray(this.disposableBeans.keySet());
		}
		for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
            // 一个一个销毁bean
			destroySingleton(disposableBeanNames[i]);
		}
		// 情况相关集合，释放引用
		this.containedBeanMap.clear();
		this.dependentBeanMap.clear();
		this.dependenciesForBeanMap.clear();
		// 情况单例对象缓存池（注册表）
		clearSingletonCache();
	}

	/**
	 * 清除注册表中所有缓存的单例实例。
	 * @since 4.3.15
	 */
	protected void clearSingletonCache() {
		synchronized (this.singletonObjects) {
			this.singletonObjects.clear();
			this.singletonFactories.clear();
			this.earlySingletonObjects.clear();
			this.registeredSingletons.clear();
			this.singletonsCurrentlyInDestruction = false;
		}
	}

	/**
	 * 如果找到了bean名称对应的对象，则销毁它，
	 * 实际是委托 disposableBean 来销毁的。
	 * @param beanName bean的名称
	 * @see #destroyBean
	 */
	public void destroySingleton(String beanName) {
		// 先从注册表（对象缓存）中删除
		removeSingleton(beanName);

		// 使用 DisposableBean 的对象实例来完成销毁操作
		DisposableBean disposableBean;
		synchronized (this.disposableBeans) {
			disposableBean = (DisposableBean) this.disposableBeans.remove(beanName);
		}
        // 委托 destroyBean 方法，详见下面
		destroyBean(beanName, disposableBean);
	}

	/**
	 * 销毁指定bean名称的bean。
	 * 注意：先销毁被依赖的bean
	 * @param beanName the name of the bean
	 * @param bean the bean instance to destroy
	 */
	protected void destroyBean(String beanName, @Nullable DisposableBean bean) {
		// 首先触发依赖bean的销毁…
		Set<String> dependencies;
		synchronized (this.dependentBeanMap) {

			dependencies = this.dependentBeanMap.remove(beanName);
		}
		if (dependencies != null) {
			if (logger.isTraceEnabled()) {
				logger.trace("Retrieved dependent beans for bean '" + beanName + "': " + dependencies);
			}
            // 一个一个销毁
			for (String dependentBeanName : dependencies) {
				destroySingleton(dependentBeanName);
			}
		}

		// 现在才是实际销毁指的bean的地方
		if (bean != null) {
			try {
				bean.destroy();
			}
			catch (Throwable ex) {
				if (logger.isInfoEnabled()) {
					logger.info("Destroy method on bean with name '" + beanName + "' threw an exception", ex);
				}
			}
		}

		// 触发对包含的bean的销毁…
		Set<String> containedBeans;
		synchronized (this.containedBeanMap) {
			
			containedBeans = this.containedBeanMap.remove(beanName);
		}
		if (containedBeans != null) {
			for (String containedBeanName : containedBeans) {
				destroySingleton(containedBeanName);
			}
		}

		// 从其他bean的依赖中删除已销毁的bean
		synchronized (this.dependentBeanMap) {
            // 基本思路，遍历删除
			for (Iterator<Map.Entry<String, Set<String>>> it = this.dependentBeanMap.entrySet().iterator(); it.hasNext();) {
				Map.Entry<String, Set<String>> entry = it.next();
				Set<String> dependenciesToClean = entry.getValue();
				dependenciesToClean.remove(beanName);
				if (dependenciesToClean.isEmpty()) {
					it.remove();
				}
			}
		}

		// 删除已销毁bean的依赖信息
		this.dependenciesForBeanMap.remove(beanName);
	}

	/**
	 * 将单例互斥体暴露给子类，
	 * 因为子类也有自己的单例互斥体，所以以此避免在延迟初始化的情况下可能出现死锁问题。
	 */
	public final Object getSingletonMutex() {
		return this.singletonObjects;
	}
}
```

