完整的事件操作有三部门组成： 事件 、事件监听器、事件发布器。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">应用事件</h3>

*Spring* 上下文应用事件为bean 与 bean之间传递消息。一个bean处理完了希望其余一个接着处理，这时我们就需要其余的一个bean监听当前bean所发送的事件。*Spring* 框架定义了 *ApplicationEvent* 抽象类作为应用事件的半成品，由所有应用程序事件扩展的类。抽象的，因为直接发布通用事件是没有意义的。它属于上下文。

```java
package org.springframework.context;

import java.util.EventObject;

public abstract class ApplicationEvent extends EventObject {
	private static final long serialVersionUID = 7099057708183571937L;

	/** 事件发生时的系统时间。 */
	private final long timestamp;
	/**
	 * 创建一个新的ApplicationEvent。
	 * @param source 事件最初产生的事件对象(决不为null)
	 */
	public ApplicationEvent(Object source) {
		super(source);
		this.timestamp = System.currentTimeMillis();
	}


	/** 在事件发生时以毫秒为单位返回系统时间。 */
	public final long getTimestamp() {
		return this.timestamp;
	}
}
```

上下文应用事件功能很少，构造方法注入 *应用事件*、记录应用事件的创建时间、获取应用事件的创建时间。它还扩展至 *java.util.EventObject* 工具类。让我们一探究竟：

```java
package java.util;

/**
 * 此类是所有事件类的根类。（除其本身，所有的事件类均由其派生）
 * 所有事件都是通过引用对象来构造的，“源”在逻辑上被认为是事件最初发生的对象。
 */

public class EventObject implements java.io.Serializable {

    private static final long serialVersionUID = 5516075349620653480L;

    /**
     * 事件最初产生的对象。
     */
    protected transient Object  source;

    /**
     * 构建一个原型事件。
     *
     * @param    source    事件最初产生的对象。
     * @exception  IllegalArgumentException  如果源对象为null
     */
    public EventObject(Object source) {
        if (source == null)
            throw new IllegalArgumentException("null source");
        this.source = source;
    }

    /** 获取事件最初产生的对象 */
    public Object getSource() {
        return source;
    }

    /** 返回这个时间对象的字符串表现形式  */
    public String toString() {
        return getClass().getName() + "[source=" + source + "]";
    }
}
```



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">应用事件监听器</h3>

> 事件的触发是由 *事件发布器* 来完成，事件的获取是由 *事件监听器* 来完成。

所有事件侦听器接口都必须扩展至此 *java.util.EventListener* 接口：

```java
package java.util;

/**
 * 此接口的作用是标记为应用事件监听器。
 * @since JDK1.1
 */
public interface EventListener {
}

```



*Spring* 中的应用监听器 *ApplicationListener<E extends ApplicationEvent>*，它是使用观察者模式来完成事件监听任务：

```java
package org.springframework.context;

import java.util.EventListener;

/**
 * @see org.springframework.context.event.ApplicationEventMulticaster
 */
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

	/**
	 * 处理应用程序事件。
	 * @param event 要响应的事件
	 */
	void onApplicationEvent(E event);
}
```

