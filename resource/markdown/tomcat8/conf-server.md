<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Tomcat</h3>

#### Servlet是什么？

Servlet的全称Server Applet，是Java Servlet的简称，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。Servlet运行于支持Java的应用服务器中。从原理上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。

#### Server Applet是什么？

Server Applet是一种服务器端的Java应用程序，具有独立于平台和协议的特性，可以生成动态的Web页面。它担当客户请求（Web浏览器或其他HTTP客户程序）与服务器响应（HTTP服务器上的数据库或应用程序）的中间层。Servlet是位于Web服务器内部的服务器端的Java应用程序，与传统的从命令启动的Java应用程序不同，Servlet由服务器加载，该Web服务器必须包含支持Servlet的Java虚拟机。

#### Tomcat是什么？

Tomcat 服务器是一个免费的开放源代码的Web应用服务器，Servlet容器，Catalina容器。

#### Catalina是什么？

这个Web容器的代号（他称、小名）。

#### Web是什么？

Web（World Wide Web）即全球广域网，也称为万维网，它是一种基于超文本和HTTP的、全球性的、动态交互的、跨平台的分布式图形信息系统。是建立在Internet上的一种网络服务，为浏览者在Internet上查找和浏览信息提供了图形化的、易于访问的直观界面，其中的文档及超级链接将Internet上的信息节点组织成一个互为关联的网状结构。

#### Web服务器是什么？

Web服务器一般指网站服务器，是指驻留于[因特网](https://baike.baidu.com/item/%E5%9B%A0%E7%89%B9%E7%BD%91/114119)上某种类型计算机的程序，可以向浏览器等Web客户端提供文档，也可以放置网站文件，让全世界浏览；可以放置数据文件，让全世界下载。目前最主流的三个Web服务器是Apache、Nginx、IIS。



#### Apache、Nginx、IIS、Jetty区别？





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

16、通过batch批处理文件和shell脚本对Tomcat进行启动和停止

17、部署工具deployer

deployer组件负责部署和安装web应用

18、ContainerServlet 特殊接口

能够让servlet访问Catalina的内部对象。通过Manager应用来部署应用程序。

19、JMX以及Tomcat是如何为其内部对象创建MBeans 是得这些对象可管理的



#### 第一章：一个简单的web服务器

Web服务器也称谓超文本传输协议（HTTP）服务器，因为它是基于HTTP来进行客户端和服务端进行通信的。一个基于Java的Web服务器使用的两个重要的类：`java.net.Socket` 和 `java.net.ServerSocket` ，并通过HTTP消息通信。

HTTP协议是基于请求（request）和响应（response）的协议。HTTP协议又是使用可靠的TCP来连接，默认80端口（HTTPS默认使用443端口）。

**HTTP请求**

一个http请求有三个部分：

- 方法 / 统一资源标识符（URI） / 协议版本
- 请求的头head
- 请求的体body

**HTTP响应**

一个http响应有三个部分：

- 方法 / 统一资源标识符（URI） / 协议版本
- 响应的头head
- 响应的体body



**java.net.Socket 类**

套字节是网络连接的端点。套字节使得每一个应用可以从网络中写入和读取数据。在网络端点的两个不同的计算上的两个应用可以通过连接发送和接受字节流。

**Socket** 带表着客户端，**ServerSocket** 带表着服务端，服务器总是等待着客户端的连接。

Socket -- 网络与机器流

I/O流 -- 动态流动

File本地 -- 静态与动态转换



#### 第二章 ：一个简单的Servlet容器

Servlet编程需要 `javax.servlet` 和 `javax.servlet.http` 这两个包中的接口来实现。

`javax.servlet.ServletRequest` 接口封装着客户端的HTTP请求，`javax.servlet.ServletResponse` 封装了 servlet 响应。

使用 `java.net.URLClassLoader` 类加载 servlet 。









#### Tomcat的IO模型





#### conf/server.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 注意:“Server”本身并不是“Container”，
	 因此在这个级别上您可能不会定义子组件，例如“Valves”。
     文档在 /docs/config/server.html
 -->
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <!-- 安全监听器. 文档在 /docs/config/listeners.html
  <Listener className="org.apache.catalina.security.SecurityListener" />
  -->
  <!-- （Apache Portable Runtime）APR 库加载器. 文档在 /docs/apr.html -->
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <!-- 防止由于使用特定的 java/javax api 而导致的内存泄漏 -->
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <!-- 全局 JNDI 资源 （Java Naming and Directory Interface）
       文档在 /docs/jndi-resources-howto.html
  -->
  <GlobalNamingResources>
    <!-- 可编辑的用户数据库,也可以由UserDatabaseRealm用于对用户进行身份验证
    -->
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <!-- 一个“Service”是一个或多个“Collector”的集合，这些连接器共享一个“Container”。
       文档在 /docs/config/service.html
   -->
  <Service name="Catalina">

    <!-- 连接器可以使用共享的执行器，您可以定义一个或多个命名的线程池 -->
    <!--
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
    -->

    <!-- “Connector”表示接收请求和返回响应的端点。 文档在 :
         Java HTTP Connector: /docs/config/http.html
         Java AJP  Connector: /docs/config/ajp.html
         APR (HTTP/AJP) Connector: /docs/apr.html
         在端口8080上定义一个非ssl /TLS HTTP/1.1连接器
    -->
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <!-- 使用共享线程池的“Connector” -->
    <!--
    <Connector executor="tomcatThreadPool"
               port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    -->
    <!-- 在端口8443上定义SSL/TLS HTTP/1.1连接器
		 这个连接器使用NIO实现。
		 默认的SSLImplementation取决于APR/本地库
		 和AprLifecycleListener的useOpenSSL属性的存在。
		 无论选择何种SSLImplementation，都可以使用JSSE或OpenSSL样式配置。
		 下面使用JSSE样式配置。
    -->
    <!--
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true">
        <SSLHostConfig>
            <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
    -->
    <!-- 使用HTTP/2在端口8443上定义SSL/TLS HTTP/1.1连接器，
		 该连接器使用APR/native实现，该实现总是为TLS使用OpenSSL。
		 可以使用JSSE或OpenSSL风格的配置。下面将使用OpenSSL样式配置。
    -->
    <!--
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
               maxThreads="150" SSLEnabled="true" >
        <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
        <SSLHostConfig>
            <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                         certificateFile="conf/localhost-rsa-cert.pem"
                         certificateChainFile="conf/localhost-rsa-chain.pem"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
    -->

    <!-- 在端口8009上定义一个AJP 1.3连接器 -->
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />


    <!-- 引擎表示处理每个请求的入口点(在Catalina中)。
		 Tomcat的引擎实现单独分析请求中包含的HTTP头，
		 并将它们传递到适当的主机(虚拟主机)。
         文档位于 /docs/config/engine.html -->

    <!-- 您应该将jvmRoute设置为通过AJP ie支持负载平衡 :
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
    -->
    <Engine name="Catalina" defaultHost="localhost">

      <!-- 关于Tomcat集群，请参阅以下文件:
          /docs/cluster-howto.html  (使用简介)
          /docs/config/cluster.html (参考文档) -->
      <!--
      <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
      -->

      <!-- 使用 LockOutRealm 防止通过暴力攻击猜测用户密码 -->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <!-- 这个Realm使用在键“UserDatabase”下的全局JNDI资源中配置的用户数据库。
			 RealmA可以立即使用针对此UserDatabase执行的任何编辑。 -->
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <!-- SingleSignOn valve，在web应用程序之间共享身份验证
             文档位于: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- 访问日志处理的所有例子。
             文档位于: /docs/config/valve.html
             注意:使用的模式等价于使用 pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>
```

