<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Tomcat</h3>

#### Servlet是什么？

Servlet的全称Server Applet，是Java Servlet的简称，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。Servlet运行于支持Java的应用服务器中。从原理上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。

#### Server Applet是什么？

Server Applet是一种服务器端的Java应用程序，具有独立于平台和协议的特性，可以生成动态的Web页面。它担当客户请求（Web浏览器或其他HTTP客户程序）与服务器响应（HTTP服务器上的数据库或应用程序）的中间层。Servlet是位于Web服务器内部的服务器端的Java应用程序，与传统的从命令启动的Java应用程序不同，Servlet由服务器加载，该Web服务器必须包含支持Servlet的Java虚拟机。

#### Tomcat是什么？

Tomcat 服务器是一个免费的开放源代码的Web应用服务器，Servlet容器，Catalina容器。

#### Catalina是什么？

这个Web容器的代号（他称、小名）。



---

#### 支持Servlet的Web容器是如何工作的？

1、创建一个request对象，接受请求并填充可能被servlet用到的信息，比如：URI、头信息、cookies、参数等等。一个request对象是 `javax.servlet.ServletRequest` 或 `javax.servlet.http.ServletRequest` 接口的一个实例对象。

2、创建一个response对象，填充数据，它所引用的servlet使用它给客户端发送响应。一个response对象是 `javax.servlet.ServletResponse`  或 `javax.servlet.http.ServletResponse` 接口的一个实例对象。 

3、调用servlet的service方法，并传入request和response对象，从request对象里面取值，处理，将处理后的结果写入response。



### Tomcat的架构

![Tomcat的架构]()

Tomcat容器最重要的两个组件是 **连接器**`Connector` 和 **容器**`Container` 



1、网络基础：

HTTP服务器，java.net.Socket和java.net.ServerSocket源码与原理分析。

2、Servlet容器工作机制

Servlet是如何工作的，创建request和response对象，并传递给service方法。

3、连接器Connector

4、容器Container

这里的容器是org.apache.catalina.Container接口，有四种类型的Container：Engine、Host、Context、Wrapper。主要是Context和Wrapper。

5、生命周期Lifecycle接口

是如何start/stop启动和停止组件的。

6、Tomcat日志框架

7、加载器

加载servlet和第一个web应用所需的其他类。

8、管理器Manager

管理会话、管理会话信息。如何把会话持久化。使用StandardManager实例来运行一个使用会话对象进行储值的servlet。

9、Web应用程序的安全性

实例：主角（principals）、角色（Roles）、登录配置，认证等等。

10、用servlet的org.apache.catalina.core.StandardWrapper类的实例对象来表示一个Web应用

过滤器filter和service方法是如何调用的

11、用servlet的org.apache.catalina.core.StandardContext类的实例对象来表示一个Web应用

StandardContext是如何配置的，每次HTTP请求发生了什么，如何支持自动重新加载。如何处理共享的定时任务。

12、Engine和Host容器

它们的标准实现：`org.apache.catalina.core.StandardHost` 和 `org.apache.catalina.core.StandardEngine` 。

13、服务器和服务组件

服务器为 servlet 容器提供了优雅的启动和停止机制，而服务为容器和一个或多个连接器提供了一个支架。

14、通过 Digester 来配置Web应用

使用Digester库把Xml转换成Java对象。然后解释成一个StandardContext对象。

15、shutdown hook钩子

Tomcat使用它总能获得一个机会用户 clean-up，而无论用户怎么停止它（既适当的发送一个shutdown命令或者不适当的简单关闭控制台）。





#### Tomcat的IO模型



#### conf/server.xml配置