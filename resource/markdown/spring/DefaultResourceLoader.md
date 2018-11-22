*AbstractApplicationContext* 抽象类依赖/扩展了很多其他的实现类，所以在分析 *AbstractApplicationContext*  源码之前是有必要分析其依赖/扩展类的源码。但有些依赖/扩展类在 [前面的文章](https://github.com/about-cloud/JavaCore) 中已经分析过，这里就不在重复。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">默认资源加载器</h3>

> *DefaultResourceLoader*  ，默认资源加载器，是 *ResourceLoader* 接口的默认实现类，用于加载资源配置。*AbstractApplicationContext* 直接继承此类。

```java
package org.springframework.core.io;

import java.net.MalformedURLException;
import java.net.URL;
import java.util.Collection;
import java.util.LinkedHashSet;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;
import org.springframework.util.ClassUtils;
import org.springframework.util.ResourceUtils;
import org.springframework.util.StringUtils;

/**
 * 通过ResourceEditor来使用，作为一个基础类来服务于
 * org.springframework.context.support.AbstractApplicationContext
 *
 * 如果location 的值是一个 URL，将返回一个UrlResource资源；
 * 如果location 的值非URL路径或是一个"classpath:"相对URL，则返回一个ClassPathResource.
 *
 * @author Juergen Hoeller
 * @since 10.03.2004
 * @see FileSystemResourceLoader
 * @see org.springframework.context.support.ClassPathXmlApplicationContext
 */
public class DefaultResourceLoader implements ResourceLoader {

    /**
     * 可为空的类加载器。
     * 默认情况下，类加载器将在这个ResourceLoader初始化时使用线程上下文类加载器进行访问。
     */
	@Nullable
	private ClassLoader classLoader;
	/** 路径协议解析的Set集合 */
	private final Set<ProtocolResolver> protocolResolvers = new LinkedHashSet<>(4);

    /** 用于资源缓存的Map的映射集 */
	private final Map<Class<?>, Map<Resource, ?>> resourceCaches = new ConcurrentHashMap<>(4);


	/**
	 * 无参构造方法创建一个新的DefaultResourceLoader对象，并设置类加载器.
	 * @see java.lang.Thread#getContextClassLoader()
	 */
	public DefaultResourceLoader() {
        /**
         * 底层实现是按以下递进获取，有则返回：
         * 当前线程的类加载、ClassUtils.class 的加载器、系统类加载
         */
		this.classLoader = ClassUtils.getDefaultClassLoader();
	}

	/**
	 * 有参构造方法，主动注入一个类加载
	 * @param classLoader 用来加载类路径资源的类加载器,因为这个类加载器可以null，所以可传null值
	 */
	public DefaultResourceLoader(@Nullable ClassLoader classLoader) {
		this.classLoader = classLoader;
	}


	/**
	 * setter 方法注入一个(可为null)的类加载器
	 */
	public void setClassLoader(@Nullable ClassLoader classLoader) {
		this.classLoader = classLoader;
	}

	/**
	 * 返回用于加载类资源的类加载器。
	 * 如果未null，则通过ClassUtils.getDefaultClassLoader()重新获取。
	 * @see ClassPathResource
	 */
	@Override
	@Nullable
	public ClassLoader getClassLoader() {
		return (this.classLoader != null ? this.classLoader : ClassUtils.getDefaultClassLoader());
	}

	/**
	 * 用这个资源加载器注册给定的解析器（前面的文章也讲说，解析器用于解析多个资源路径）。
	 * 任何这样的解析器都会在这个加载程序的标准解析规则之前被调用。
	 * 因此，它还可能覆盖任何默认规则。
	 * @since 4.3
	 * @see #getProtocolResolvers()
	 */
	public void addProtocolResolver(ProtocolResolver resolver) {
		Assert.notNull(resolver, "ProtocolResolver must not be null");
		this.protocolResolvers.add(resolver);
	}

	/**
	* 返回当前注册的协议解析器集合，允许进行自行修改。
	 * @since 4.3
	 */
	public Collection<ProtocolResolver> getProtocolResolvers() {
		return this.protocolResolvers;
	}

	/**
	 * 获取由 Resource 键入的给定值类型的缓存。
	 * @param valueType 值类型, 比如一个 ASM MetadataReader
	 * @return Map缓存, 在ResourceLoader级别进行分享
	 * @since 5.0
	 */
	@SuppressWarnings("unchecked")
	public <T> Map<Resource, T> getResourceCache(Class<T> valueType) {
		return (Map<Resource, T>) this.resourceCaches.computeIfAbsent(valueType, key -> new ConcurrentHashMap<>());
	}

	/**
	 * 清空此资源加载器中所有的资源缓存
	 * @since 5.0
	 * @see #getResourceCache
	 */
	public void clearResourceCaches() {
		this.resourceCaches.clear();
	}

	/**
	 * 从指定的路径获取资源，不能为null
	 *
	 * 获取资源的优先策略从高到底依次是：
	 * 1.从协议解析器集合遍历每个协议器尝试解析指定位置的资源；
	 * 2.从"/"开头的绝对路径尝试解析资源；
	 * 3.从"classpath:"的类相对路径解析资源；
	 * 4.最后从网络URL处解析资源。
	 */
	@Override
	public Resource getResource(String location) {
		Assert.notNull(location, "Location must not be null");
		// 遍历协议解析器的集合
		for (ProtocolResolver protocolResolver : this.protocolResolvers) {
            // 然后用当前的协议解析来解析指定路径的资源
			Resource resource = protocolResolver.resolve(location, this);
			if (resource != null) {
                // 如果在当前协议解析器下解析的资源不为空，则返回该资源
				return resource;
			}
		}
		// 程序运行到这里，要么说明协议解析器集合为空，
        // 要么说明协议解析器集合中的所有协议解析器解析指定位置的资源都为null
        // 那么就对资源路径判断
		if (location.startsWith("/")) {
            // 此方法是本类中的方法，下面分析
			return getResourceByPath(location);
		}
        // 如果资源路径不以 "/" 为开始的绝对路径标识，
        // 则判断是否以默认方式 "classpath:" 开头
		else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
            // 如果以默认的"classpath:"开头，则说明资源是类路径的相对资源
			return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
		}
        // 在不然，就尝试从远处网络URL资源加载
		else {
			try {
				// 尝试将路径位置解析为URL…
				URL url = new URL(location);
				return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
			}
			catch (MalformedURLException ex) {
				// 没有URL可以解析为资源路径
				return getResourceByPath(location);
			}
		}
	}

	/**
	 * 返回给定路径上的资源地址引用。
	 * 默认实现支持类路径位置。 这应该适用于独立实现，但可以被重写，
	 * 比如：用于针对Servlet容器的实现。
	 * @param path 资源的路径
	 * @return 对应的资源引用
	 * @see ClassPathResource
	 * @see org.springframework.context.support.FileSystemXmlApplicationContext#getResourceByPath
	 * @see org.springframework.web.context.support.XmlWebApplicationContext#getResourceByPath
	 */
	protected Resource getResourceByPath(String path) {
		return new ClassPathContextResource(path, getClassLoader());
	}


	/**
	 * 类路径库，通过实现上下文源接口显式地表示上下文相关路径。
	 */
	protected static class ClassPathContextResource extends ClassPathResource implements ContextResource {

		public ClassPathContextResource(String path, @Nullable ClassLoader classLoader) {
			super(path, classLoader);
		}

		@Override
		public String getPathWithinContext() {
			return getPath();
		}

		@Override
		public Resource createRelative(String relativePath) {
			String pathToUse = StringUtils.applyRelativePath(getPath(), relativePath);
			return new ClassPathContextResource(pathToUse, getClassLoader());
		}
	}
}
```


