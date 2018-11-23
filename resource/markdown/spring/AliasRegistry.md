<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">AliasRegistry 接口</h3>

> 管理别名的通用接口。作为 *org.springframework.beans.factory.support.BeanDefinitionRegistry* 的父接口。

```java
package org.springframework.core;

/** @since 2.5.2 */
public interface AliasRegistry {

	/**
	 * 给定一个名称，用以注册一个别名。
	 * @param name 规范的名称
	 * @param alias 要注册的别名
	 * @throws IllegalStateException 如果别名已经在使用中，可能不会被覆盖
	 */
	void registerAlias(String name, String alias);

	/**
	 * 从这个注册表中删除指定的别名。
	 * @param alias 要删除的别名
	 * @throws IllegalStateException 如果没有找到这样的别名
	 */
	void removeAlias(String alias);

	/**
	 * 判断此给定名称是否是定义的别名
	 * @param name 要检查的名称
	 * @return 给定的名称是否为别名
	 */
	boolean isAlias(String name);

	/**
	 * 如果定义了名称，则返回给定名称的别名列表。
	 * @param name 检查别名的名称
	 * @return 别名。如果没有别名，则返回为空数组
	 */
	String[] getAliases(String name);
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">BeanDefinitionRegistry 接口</h3>

> *BeanDefinitionRegistry* 用于保存bean定义的注册中心的接口。

```java
package org.springframework.beans.factory.support;

import org.springframework.beans.factory.BeanDefinitionStoreException;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.core.AliasRegistry;

/**
 * 包含bean定义的注册接口，
 * 例如RootBeanDefinition 和 ChildBeanDefinition 实例。
 * 通常由内部使用AbstractBeanDefinition层次结构的beanfactory实现。
 *
 * 这是Spring 的 bean工厂包中唯一一个封装了bean描述的注册接口。
 * 标准的BeanFactory接口只覆盖对 完全配置工厂实例 的访问。
 *
 * Spring的bean定义读者希望处理这个接口的实现。
 * Spring核心中已知的实现者是DefaultListableBeanFactory和GenericApplicationContext。
 *
 * @author Juergen Hoeller
 * @since 26.11.2003
 * @see org.springframework.beans.factory.config.BeanDefinition
 * @see AbstractBeanDefinition
 * @see RootBeanDefinition
 * @see ChildBeanDefinition
 * @see DefaultListableBeanFactory
 * @see org.springframework.context.support.GenericApplicationContext
 * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
 * @see PropertiesBeanDefinitionReader
 */
public interface BeanDefinitionRegistry extends AliasRegistry {

	/**
	 * 使用此注册器注册一个新的bean定义(描述)对象。
	 * 必须支持RootBeanDefinition和ChildBeanDefinition。
	 * @param beanName 要注册的bean实例的名称
	 * @param beanDefinition 要注册的bean实例的订单
	 * @throws BeanDefinitionStoreException 如果此 BeanDefinition 无效的话
	 * @throws BeanDefinitionOverrideException 如果指定的bean名称已经有bean定义，并且不允许覆盖它
	 * @see GenericBeanDefinition
	 * @see RootBeanDefinition
	 * @see ChildBeanDefinition
	 */
	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	/**
	 * 删除给定名称的bean定义。
	 * @param beanName 要注册的bean实例的名称
	 * @throws NoSuchBeanDefinitionException如果没有这样的bean定义
	 */
	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 返回给定bean名称的bean定义。
	 * @param beanName 要查找定义的bean的名称
	 * @return 给定名称对应的此 BeanDefinition（非null）
	 * @throws NoSuchBeanDefinitionException 如果没有对应的 bean定义
	 */
	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 检查此注册表是否包含具有给定名称的bean定义。
	 * @param beanName 要查找的bean的名称
	 * @return 如果此注册表包含具有给定名称的bean定义
	 */
	boolean containsBeanDefinition(String beanName);

	/**
	 * 返回此注册表中定义的所有bean的名称，以String数组的形式返回。
	 * @return the names of all beans defined in this registry,
	 * or an empty array if none defined
	 */
	String[] getBeanDefinitionNames();

	/**
	 * 返回注册表中定义的bean的数量。
	 * @return 注册表中定义的bean的数量
	 */
	int getBeanDefinitionCount();

	/**
	 * 确定给定的bean名是否已经在此注册中心中使用，
	 * 例如，是否有本地bean或别名注册在这个名称下。
	 * @param beanName 要检查的名称
	 * @return 给定的bean名称是否已经在使用
	 */
	boolean isBeanNameInUse(String beanName);

}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">SimpleAliasRegistry 类</h3>

> *SimpleAliasRegistry*  类是 *AliasRegistry* 接口的简单实现。

```java
package org.springframework.core;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.util.Assert;
import org.springframework.util.StringUtils;
import org.springframework.util.StringValueResolver;

/**
 * 
 * Serves as base class for
 * 作为 org.springframework.beans.factory.support.BeanDefinitionRegistry 实现的基类
 *
 * @author Juergen Hoeller
 * @since 2.5.2
 */
public class SimpleAliasRegistry implements AliasRegistry {

	/** 可用于子类的logger */
	protected final Log logger = LogFactory.getLog(getClass());

	/** 别名到标准名称的映射Map */
	private final Map<String, String> aliasMap = new ConcurrentHashMap<>(16);

	/**
     * 注册别名
     * 名称和别名都不允许为空(null和空字符串)
     */
	@Override
	public void registerAlias(String name, String alias) {
		Assert.hasText(name, "'name' must not be empty");
		Assert.hasText(alias, "'alias' must not be empty");
        // 对名称、别名映射器加锁
		synchronized (this.aliasMap) {
			if (alias.equals(name)) {
                // 如果别名和标准名称相同，就删除。没必要存在相同的名称
				this.aliasMap.remove(alias);
				if (logger.isDebugEnabled()) {
					logger.debug("Alias definition '" + alias + "' ignored since it points to same name");
				}
			}
			else {
                // 通过别名作为map的key（就是标准名称），来获取已注册的名称
                // （言外之意就是判断是否存在与此别名相同的标准名称）
				String registeredName = this.aliasMap.get(alias);
				if (registeredName != null) {
                    // 如果已注册的标准名称不为空，并且和当前要注册的指定名称相同，就直接返回
					if (registeredName.equals(name)) {
						// 因为已存在相同的别名，不需要再注册了，
                        // 根据map的key的特性，会覆盖value（也就是别名）
						return;
					}
                    // 已注册的名称和当前要注册的名称不相同，判断别名是否允许被覆盖
                    // 这里有点绕，我来解释一下：
                    // 就是要注册的别名alias虽然和要注册的标准名称name不相同，
                    // 但要注册的别名alias和其它已注册的标准名称相同了，这里问是否可以不允许覆盖
					if (!allowAliasOverriding()) {
						throw new IllegalStateException("Cannot define alias '" + alias + "' for name '" +
								name + "': It is already registered for name '" + registeredName + "'.");
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Overriding alias '" + alias + "' definition for registered name '" +
								registeredName + "' with new target name '" + name + "'");
					}
				}
                // 检查名称是否被其他地方使用，防止循环引用
				checkForAliasCircle(name, alias);
                // 上面的检查不抛出异常的话，说明不存在循环引用，
                // 下面就可以将标准的名称和别名添加到Map映射集合中
				this.aliasMap.put(alias, name);
				if (logger.isTraceEnabled()) {
					logger.trace("Alias definition '" + alias + "' registered for name '" + name + "'");
				}
			}
		}
	}

	/**
	 * 返回是否允许别名重写。
	 * 默认为 true
	 */
	protected boolean allowAliasOverriding() {
		return true;
	}

	/**
	 * 判断给定的标准名称name是否已注册了给定的别名alias。
	 * 在这个别名注册映射集合aliasMap中，不管是标准名称,还是别名，只能存在一个，
	 * 如果在名称集中出了，那么它不能在别名集中出现，而且是唯一，反之亦然。
	 * @param name 要检查的名称
	 * @param alias 要查找的别名
	 * @since 4.2.1
	 */
	public boolean hasAlias(String name, String alias) {
        // 迭代每个map映射项
		for (Map.Entry<String, String> entry : this.aliasMap.entrySet()) {
            // 获取别名
			String registeredName = entry.getValue();
            // 判断已存放的别名与当前要注册的标准名称是否相同
			if (registeredName.equals(name)) {
                // 已存放的别名与当前要注册的标准名称相同的话，就判断与已注册的别名是否相同
				String registeredAlias = entry.getKey();
				if (registeredAlias.equals(alias) || hasAlias(registeredAlias, alias)) {
					return true;
				}
			}
		}
		return false;
	}

    /**
     * 通过给定的别名删除已注册的别名，
     * 如果要删除的别名从来没有注册过，会抛出下面的IllegalStateException异常
     */
	@Override
	public void removeAlias(String alias) {
		synchronized (this.aliasMap) {
			String name = this.aliasMap.remove(alias);
			if (name == null) {
				throw new IllegalStateException("No alias '" + alias + "' registered");
			}
		}
	}

	@Override
	public boolean isAlias(String name) {
		return this.aliasMap.containsKey(name);
	}

	@Override
	public String[] getAliases(String name) {
		List<String> result = new ArrayList<>();
		synchronized (this.aliasMap) {
			retrieveAliases(name, result);
		}
		return StringUtils.toStringArray(result);
	}

	/**
	 * 临时检索给定名称的所有别名。
	 * @param name 要查找别名的目标名称
	 * @param result 结果别名列表
	 */
	private void retrieveAliases(String name, List<String> result) {
		this.aliasMap.forEach((alias, registeredName) -> {
			if (registeredName.equals(name)) {
				result.add(alias);
				retrieveAliases(alias, result);
			}
		});
	}

	/**
	 * 通过指定的 StringValueResolver 来解析在此工厂已注册的所有别名目标名称和别名。
	 * 例如，值解析器可以解析目标bean名甚至别名中的占位符。
	 * （StringValueResolver 是一种用于解析字符串值的简单策略。）
	 * @param valueResolver 要应用的StringValueResolver
	 */
	public void resolveAliases(StringValueResolver valueResolver) {
        // 值解析器不允许为null
		Assert.notNull(valueResolver, "StringValueResolver must not be null");
        // 依旧加锁
		synchronized (this.aliasMap) {
            // 深拷贝
			Map<String, String> aliasCopy = new HashMap<>(this.aliasMap);
			aliasCopy.forEach((alias, registeredName) -> {
				String resolvedAlias = valueResolver.resolveStringValue(alias);
				String resolvedName = valueResolver.resolveStringValue(registeredName);
				if (resolvedAlias == null || resolvedName == null || resolvedAlias.equals(resolvedName)) {
					this.aliasMap.remove(alias);
				}
				else if (!resolvedAlias.equals(alias)) {
					String existingName = this.aliasMap.get(resolvedAlias);
					if (existingName != null) {
						if (existingName.equals(resolvedName)) {
							// 指向现有别名——只需删除占位符
							this.aliasMap.remove(alias);
							return;
						}
						throw new IllegalStateException(
								"Cannot register resolved alias '" + resolvedAlias + "' (original: '" + alias +
								"') for name '" + resolvedName + "': It is already registered for name '" +
								registeredName + "'.");
					}
					checkForAliasCircle(resolvedName, resolvedAlias);
					this.aliasMap.remove(alias);
					this.aliasMap.put(resolvedAlias, resolvedName);
				}
				else if (!registeredName.equals(resolvedName)) {
					this.aliasMap.put(alias, resolvedName);
				}
			});
		}
	}

	/**
	 * 检查给定名称是否被其他地方使用此别名，
	 * 预先捕获循环引用并抛出相应的IllegalStateException。
	 * @param name 候选名称
	 * @param alias 候选别名
	 * @see #registerAlias
	 * @see #hasAlias
	 */
	protected void checkForAliasCircle(String name, String alias) {
		if (hasAlias(alias, name)) {
			throw new IllegalStateException("Cannot register alias '" + alias +
					"' for name '" + name + "': Circular reference - '" +
					name + "' is a direct or indirect alias for '" + alias + "' already");
		}
	}

	/**
	 * 确定原始的名称，将别名解析为标准名称。
	 * @param name 用户指定名称
	 * @return 转换后名称
	 */
	public String canonicalName(String name) {
        // 标准名称
		String canonicalName = name;
		// 解析后的别名
		String resolvedName;
		do {
            // 通过标准名称获取aliasMap中对应的名称
			resolvedName = this.aliasMap.get(canonicalName);
            // 如果得到的名称不为null，意味着已存在此名称，那么后面不会循环再次判断
			if (resolvedName != null) {
				canonicalName = resolvedName;
			}
		}
		while (resolvedName != null); // 如果没有找到对应的名称，则一直循环下去
		return canonicalName;
	}
}
```



其他：

* @GenericBeanDefinition
* @see RootBeanDefinition
* @see ChildBeanDefinition