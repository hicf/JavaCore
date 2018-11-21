> 🏃本文基于 `Spring5.1.2.RELEASE` 版本分析



> 上篇文章 [分析了 *BeanFactory* 接口工厂和其两个子接口](https://github.com/about-cloud/JavaCore) ，它们定义最顶层的关于 *bean对象* 通用方法，本文的重点，*ApplicationContext* 接口也是间接继承至 *BeanFactory* 接口，但它扩展了更多的、面向实际应用功能。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">ApplicationContext体系结构</h3>

![ApplicationContext体系结构]()

从 *ApplicationContext* 类继承结构可以看出，它扩展了分层Bean工厂、可罗列Bean工厂、国际化信息、应用事件发布、资源加载、资源模式解析、能够处理环境等功能。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">1、MessageSource</h3>

#### 什么是国际化？

> 国际化（internationalization）是设计和制造容易适应不同区域要求的产品的一种方式。它要求从产品中抽离所有[地域](https://baike.baidu.com/item/%E5%9C%B0%E5%9F%9F/4437973)语言，国家/地区和文化相关的[元素](https://baike.baidu.com/item/%E5%85%83%E7%B4%A0/373563)。换言之，应用程序的功能和代码设计考虑在不同地区运行的需要，其[代码](https://baike.baidu.com/item/%E4%BB%A3%E7%A0%81/86048)简化了不同本地版本的生产。开发这样的程序的过程，就称为国际化。（来源于百度百科）

```java
package org.springframework.context;

import java.util.Locale;

import org.springframework.lang.Nullable;
/** 用于解决消息的策略接口，支持这些消息的参数化和i18n国际化。 */
public interface MessageSource {
	// ------------ 根据特定的参数获取并返回String类型的消息 ------------
	@Nullable
	String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);

	String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;

	String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;

}

```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">2、ApplicationEventPublisher</h3>

> 应用事件发布器，封装事件发布功能的接口。

#### 什么是应用事件？

> 关于应用事件的问题，不适宜写在本篇文章中，我单独写了一篇关于 **应用事件及事件监听器** 的文章，原文地址：https://github.com/about-cloud/JavaCore

```java
package org.springframework.context;

@FunctionalInterface
public interface ApplicationEventPublisher {
	// --------------------- 发布事件的方法 ---------------------
	default void publishEvent(ApplicationEvent event) {
		publishEvent((Object) event);
	}

	void publishEvent(Object event);
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">3、ResourceLoader</h3>

> 资源加载器：用于加载资源的策略接口，通过类路径或文件系统加载资源。当在 *ApplicationContext* 中运行时，可以使用特定上下文的资源加载策略从字符串填充Resource和Resource数组类型的Bean属性。

```java
package org.springframework.core.io;

import org.springframework.lang.Nullable;
import org.springframework.util.ResourceUtils;

/**
 * @see Resource
 * @see org.springframework.core.io.support.ResourcePatternResolver
 * @see org.springframework.context.ApplicationContext
 * @see org.springframework.context.ResourceLoaderAware
 */
public interface ResourceLoader {

	/** 用于从类路径加载的伪URL前缀："classpath:"。所谓伪URL、伪路径都是相对URL、相对路径 */
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;
	/** 从指定的 location 位置处加载资源，路径必须是全限定路径 */
	Resource getResource(String location);
	/** 获取类加载器 */
	@Nullable
	ClassLoader getClassLoader();

}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">4、ResourcePatternResolve</h3>

> *ResourcePatternResolver* 接口继承至 *ResourceLoader* ，它的作用是加载多个Resource。

```java
package org.springframework.core.io.support;

import java.io.IOException;

import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;

/**
 * @see org.springframework.core.io.Resource
 * @see org.springframework.core.io.ResourceLoader
 * @see org.springframework.context.ApplicationContext
 * @see org.springframework.context.ResourceLoaderAware
 */
public interface ResourcePatternResolver extends ResourceLoader {
	/** 用于从类路径加载的伪URL前缀："classpath:"。所谓伪URL、伪路径都是相对URL、相对路径 */
	String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
	/**
	 * 将指定的位置模式解析为资源对象。
	 * 应该尽可能避免指向同一物理资源的重叠资源项。
	 * 结果应该具有set语义。。
	 */
	Resource[] getResources(String locationPattern) throws IOException;

}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">5、EnvironmentCapable</h3>

> 与环境Environment有关的接口

```java
package org.springframework.core.env;

/**
 * @author Chris Beams
 * @since 3.1
 * @see Environment
 * @see ConfigurableEnvironment
 * @see org.springframework.context.ConfigurableApplicationContext#getEnvironment()
 */
public interface EnvironmentCapable {

	/** 返回与此组件关联的环境Environment。 */
	Environment getEnvironment();

}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">WebApplicationContext</h3>

> *WebApplicationContext* 接口扩展了*ApplicationContext* ，它是专门为 Web 应用设计的，但并意味着只有它才能为 Web应用服务。

```java
package org.springframework.web.context;

import javax.servlet.ServletContext;

import org.springframework.context.ApplicationContext;
import org.springframework.lang.Nullable;

public interface WebApplicationContext extends ApplicationContext {

	/**
	 * 上下文属性要绑定到WebApplicationContext根应用才能成功启动。
	 * 注意: 如果根上下文的启动失败，则该属性可以包含异常或错误作为值。
	 * 使用WebApplicationContextUtils可以方便的查找WebApplicationContext根应用。
	 * @see org.springframework.web.context.support.WebApplicationContextUtils#getWebApplicationContext
	 * @see org.springframework.web.context.support.WebApplicationContextUtils#getRequiredWebApplicationContext
	 */
	String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";

	/**
	 * request范围的范围标识符: "request".
	 * 除了标准scope之外，还支持“"singleton"单例模式 和"prototype"原型模式。
	 */
	String SCOPE_REQUEST = "request";

	/**
	 * session范围的范围标识符: "session".
	 * 除了标准scope之外，还支持“"singleton"单例模式 和"prototype"原型模式。
	 */
	String SCOPE_SESSION = "session";

	/**
	 * 全局web application范围的范围标识符: "application".
	 * 除了标准scope之外，还支持“"singleton"单例模式 和"prototype"原型模式。
	 */
	String SCOPE_APPLICATION = "application";

	/**
	 * 工厂中的 ServletContext 环境的bean的名称。
	 * @see javax.servlet.ServletContext
	 */
	String SERVLET_CONTEXT_BEAN_NAME = "servletContext";

	/**
	 * 工厂中的 ServletContext 初始化参数环境的bean的名称。
	 * 注意: 可能与ServletConfig参数合并。
	 * ServletConfig参数会覆盖同名的ServletContext参数。
	 * @see javax.servlet.ServletContext#getInitParameterNames()
	 * @see javax.servlet.ServletContext#getInitParameter(String)
	 * @see javax.servlet.ServletConfig#getInitParameterNames()
	 * @see javax.servlet.ServletConfig#getInitParameter(String)
	 */
	String CONTEXT_PARAMETERS_BEAN_NAME = "contextParameters";

	/**
	 * 工厂中的 ServletContext 属性环境bean的名称。
	 * @see javax.servlet.ServletContext#getAttributeNames()
	 * @see javax.servlet.ServletContext#getAttribute(String)
	 */
	String CONTEXT_ATTRIBUTES_BEAN_NAME = "contextAttributes";


	/**
	 * 获取并返回这个应用程序的标准Servlet API Servlet上下文。
	 */
	@Nullable
	ServletContext getServletContext();

}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">ConfigurableApplicationContext</h3>

> *ConfigurableApplicationContext* ，是可配置的 *ApplicationContext* ，用户可以根据自己的配置应用上下文，所以在使用 应用上下文 时，有了更多的灵活性。

*ConfigurableApplicationContext* 继承至 *ApplicationContext* 接口和 *Lifecycle* 接口，下面简单介绍一下比较陌生的  *Lifecycle* 接口:

> Spring容器中的 bean对象 都有生命周期， *Lifecycle* 接口则定义了每个对象的最有关键的行为，而实现类扩展的更为丰富。

```java
package org.springframework.context;

public interface Lifecycle {

	/**
	 * 启动此组件。
	 * 如果组件已经在运行，则不应该抛出异常。
	 * 对于容器，这将把开始信号传播到所有应用的组件。
	 * @see SmartLifecycle#isAutoStartup()
	 */
	void start();

	/**
	 * 通常以同步方式停止此组件。
	 * 以便组件的方法在改返回的地方返回。
	 * 如果必须要异步的方式停止此组件，可以使用stop(Runnable)方法。
	 * 
	 * @see SmartLifecycle#stop(Runnable)
	 * @see org.springframework.beans.factory.DisposableBean#destroy()
	 */
	void stop();

	/**
	 * 检查该组件是否正在运行。
	 * 在容器的该实例下，只有当应用的所有组件都在运行时，才会返回true。
	 */
	boolean isRunning();

}
```

##### *ConfigurableApplicationContext* 源码分析：

```java
package org.springframework.context;

import java.io.Closeable;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.Environment;
import org.springframework.core.io.ProtocolResolver;
import org.springframework.lang.Nullable;

public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable {

	/** 当存在多个配置时，此符号用于分割多个配置的位置 */
	String CONFIG_LOCATION_DELIMITERS = ",; \t\n";

	/**
	 * 在此工厂中，ConversionService 服务 bean的名称。
	 * 如果没有提供，则使用此处的值作为默认转换规则。
	 * @see org.springframework.core.convert.ConversionService
	 */
	String CONVERSION_SERVICE_BEAN_NAME = "conversionService";

	/**
	 * 在此工厂中，LoadTimeWeaver bean 的名称。如果提供这样的bean，
	 * 为了允许LoadTimeWeaver处理所有实际的bean类,
	 * 上下文将使用一个临时类加载器进行类型匹配。
	 * LoadTimeWeaver 类提供可代码织入的功能。
	 * @see org.springframework.instrument.classloading.LoadTimeWeaver
	 */
	String LOAD_TIME_WEAVER_BEAN_NAME = "loadTimeWeaver";

	/**
	 * 在此系统中，此Environment环境bean的名称。
	 */
	String ENVIRONMENT_BEAN_NAME = "environment";

	/**
	 * 在此工厂中，此系统属性bean的名称。
	 * @see java.lang.System#getProperties()
	 */
	String SYSTEM_PROPERTIES_BEAN_NAME = "systemProperties";

	/**
	 * 在此工厂中，此系统环境bean的名称。
	 * @see java.lang.System#getenv()
	 */
	String SYSTEM_ENVIRONMENT_BEAN_NAME = "systemEnvironment";


	/**
	 * 设置此应用程序上下文的惟一id。
	 */
	void setId(String id);

	/**
	 * 设置此应用程序上下文的父ApplicationContext。
	 * 注意，父类不应该被更改:只有在创建该类的对象时(例如WebApplicationContext设置时)不可用时，
	 * 它才应该在构造函数之外设置。
	 * @see org.springframework.web.context.ConfigurableWebApplicationContext
	 */
	void setParent(@Nullable ApplicationContext parent);

	/**
	 * 为这个应用程序上下文设置Environment环境。
	 * @param environment the new environment
	 */
	void setEnvironment(ConfigurableEnvironment environment);

	/**
	 * 以可配置的形式返回此应用程序上下文的Environment环境，允许进一步定制。
	 */
	@Override
	ConfigurableEnvironment getEnvironment();

	/**
	 * 添加一个新的BeanFactoryPostProcessor，
	 * 该BeanFactoryPostProcessor将在刷新时应用于此应用程序上下文的内部bean工厂，
	 * 然后计算任何bean定义。在上下文配置期间调用。
	 */
	void addBeanFactoryPostProcessor(BeanFactoryPostProcessor postProcessor);

	/**
	 * 添加一个新的ApplicationListener，它将在上下文事件(如上下文刷新和上下文关闭)上得到通知。
	 * 注意，在此注册的任何ApplicationListener都将在刷新时应用，如果上下文尚未激活，
	 * 或者在已经激活的上下文中使用当前事件多路传播。
	 * @see org.springframework.context.event.ContextRefreshedEvent
	 * @see org.springframework.context.event.ContextClosedEvent
	 */
	void addApplicationListener(ApplicationListener<?> listener);

	/**
	 * Register the given protocol resolver with this application context,
	 * allowing for additional resource protocols to be handled.
	 * <p>Any such resolver will be invoked ahead of this context's standard
	 * resolution rules. It may therefore also override any default rules.
	 */
	void addProtocolResolver(ProtocolResolver resolver);

	/**
	 * 加载或刷新一个持久化的配置，可能是XML文件、属性文件或关系数据库模式。
	 * 由于这是一种启动方法，如果失败，应该销毁已经创建的单例，以避免悬空资源。
	 * 换句话说，在调用该方法之后，要么全部实例化，要么完全不实例化。
	 * @throws 如果bean工厂无法初始化，则抛出 BeansException 异常
	 * @throws 如果已经初始化且不支持多次刷新，则会抛出 IllegalStateException 异常
	 */
	void refresh() throws BeansException, IllegalStateException;

	/**
	 * 向JVM运行时注册一个关闭链接，在JVM关闭时关闭这个上下文，除非此时它已经关闭。
	 * 这个方法可以多次调用。每个上下文实例只注册一个关闭链接(最大)。
	 * @see java.lang.Runtime#addShutdownHook
	 * @see #close()
	 */
	void registerShutdownHook();

	/**
	 * 关闭此应用程序上下文，释放实现可能持有的所有资源和锁。
	 * 这包括销毁所有缓存的单例bean。
	 * 注意：不能在父上下文中调用close；父上下文有它们自己的独立生命周期。
	 * 此方法可以多次调用，没有副作用：对已经关闭的上下文的后续关闭调用将被忽略。
	 */
	@Override
	void close();

	/**
	 * 确定此应用程序上下文是否处于活动状态，即是否至少刷新过一次并且尚未关闭。
	 * @return 上下文是否仍处于活动状态
	 * @see #refresh()
	 * @see #close()
	 * @see #getBeanFactory()
	 */
	boolean isActive();

	/**
	 * 返回此应用程序上下文的内部bean工厂。可用于访问基础工厂的特定功能。
	 * 注意:不要使用此处来对bean工厂进行后置处理;单例在以前就已经被实例化过了。
	 * 在接触bean之前，使用BeanFactoryPostProcessor拦截BeanFactory设置过程。
	 * 通常，这个内部工厂只能在上下文处于活动状态时访问，即处于 refresh() 和 close() 之间。
	 * 可以调用 isActive() 来检查上下文是否处于可用的状态。
	 * @see #isActive()
	 * @see #refresh()
	 * @see #close()
	 * @see #addBeanFactoryPostProcessor
	 */
	ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;

}
```

