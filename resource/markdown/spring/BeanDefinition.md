#### bean元数据元素
>BeanMetadataElement

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
>

```java
package org.springframework.core;

import org.springframework.lang.Nullable;

/**
 * 接口定义用于向任意对象附加和访问元数据的通用契约。
 *
 * @author Rob Harrop
 * @since 2.0
 */
public interface AttributeAccessor {

	/**
	 * 通过名称设置属性值。
	 * 通常，用户应该使用完全限定的名称(可能使用类或包名称作为前缀)来防止与其他元数据属性重叠。
	 * @param name the unique attribute key
	 * @param value the attribute value to be attached
	 */
	void setAttribute(String name, @Nullable Object value);

	/**
	 * Get the value of the attribute identified by {@code name}.
	 * Return {@code null} if the attribute doesn't exist.
	 * @param name the unique attribute key
	 * @return the current value of the attribute, if any
	 */
	@Nullable
	Object getAttribute(String name);

	/**
	 * Remove the attribute identified by {@code name} and return its value.
	 * Return {@code null} if no attribute under {@code name} is found.
	 * @param name the unique attribute key
	 * @return the last value of the attribute, if any
	 */
	@Nullable
	Object removeAttribute(String name);

	/**
	 * Return {@code true} if the attribute identified by {@code name} exists.
	 * Otherwise return {@code false}.
	 * @param name the unique attribute key
	 */
	boolean hasAttribute(String name);

	/**
	 * Return the names of all attributes.
	 */
	String[] attributeNames();
}
```





<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">BeanDefinition</h3>

