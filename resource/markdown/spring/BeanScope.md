<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">SpringBean 的作用域</h3>



| 作用域类型         | 模式         | 描述                                                         |
| :----------------- | :----------- | ------------------------------------------------------------ |
| **singleton**      | **单例模式** | bean对象以单例的模式存在IoC容器中，既在一个容器中，相同的对象只存在一个实例。 |
| **prototype**      | **原型模式** | 每次从容器中获取bean时，都会产生并返回新创建的对象实例。     |
| request            | 请求模式     | 每次HTTP请求时才会创建一个新的bean，该作用域仅适用于WebApplicationContext上下文的环境。 |
| session            | 会话模式     | 同一个HTTP session 共享一个bean，不同的HTTP session的bean对象实例不同。 |
| globalSession      | 全局会话模式 | 同一个全局 session 共享一个bean，一般用于 Portlet 应用环境。该作用域仅适用于 WebApplicationContext上下文的环境。 |
| 自定义bean的作用域 |              |                                                              |