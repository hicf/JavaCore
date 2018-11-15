<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">HTTP协议</h3>

> **HTTP** 协议是大家耳熟能详、也是计算机网络流行广泛的网络协议 -- *HTTP*，HyperText Transfer Protocol)**超文本传输协议**。它诞生于1991年（比我年纪还大），是为传输 HTML（超文本标记语言）格式的文本字符串。HTTP 是建立在TCP之上的协议。它还有一个特点 -- 无状态。

#### 基本工作流程：

> 客户端发起HTTP请求，请求里封装一些信息表明目的地和想要的资源。服务端收到请求后开始分析含义，得知客户端想要访问资源时，服务端就把资源响应给客户端。



#### HTTP/0.9

> 最早比较简单的HTTP/0.9版本，起初只能支持Get这种拉模式的请求，默认端口号80，只支持文本，也没有HTTP头。

#### HTTP/1.0

> 经过5年历练发展之后，到了1996年，又发布了新版本的HTTP/1.0协议：新增支持图片视频等格式的数据、新增HTTP头，头里添加了状态码status。关于状态码大全及含义参考：https://github.com/about-cloud/JavaCore。为了提高效率，HTTP 1.0规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个TCP连接，服务器完成请求处理后立即断开TCP连接，服务器不跟踪每个客户也不记录过去的请求。这里有个问题，一个页面包含多个url（比如css、script、图片、音频、视频等），每个url都需要重新发起一次请求、建立单独连接，每次建立连接都要消耗资源，存在的问题就是 **不能复用连接**。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、HTTP/1.1</h3>

> 为了解决不能 **复用连接** 等其他问题，HTTP/1.1 支持持久连接（可复用连接）、管道化机制、多种缓存机制，而且还添加了 OPTIONS,PUT, DELETE, TRACE, CONNECT方法 等。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、HTTPS</h3>

> **HTTPS**，超文本传输安全协议（Hypertext Transfer Protocol Secure，HTTPS，HTTP over TLS，HTTP over SSL或HTTP Secure）。
>
> 传统的HTTP存在安全问题，HTTPS可以称为安全版本的HTTP，实现是HTTP + SSL/TLS，加密安全的任务就交给SSL/TLS。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、HTTP/2.0</h3>

2015年发布的HTTP/2.0有很多新特点：

##### 1.多路复用技术

![多路复用技术]()

##### 2.服务端推送



##### 3.头部压缩技术



##### 4.增加二进制分帧层

