<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">GET和POST方法区别</h3>

#### 设计初衷：

- GET -- **从** 指定的资源定位符处请求数据。
- POST -- **向** 指定的资源定位符处提交要被处理的数据。

#### 最直观的区别：

- GET - 把参数包含在URL中。
- POST - 把参数包含在请求体重。

> 注意两个谬论：
>
> 1. GET 方法的参数最多2083字节？：这是早期浏览器地址栏的限制，不是GET请求的限制，不同的浏览器对地址栏url限制长度不一样；
> 2. POST方法比GET方法安全？：只不过片面看待此问题，GET方法传递加密后的参数一样安全，POST方法未加密的请求体一样不安全。



对于GET方法请求，回将整个请求一并发送至目的地；而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。