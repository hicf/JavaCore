

前面的[章节](https://github.com/about-cloud/JavaCore)已经讲过，TCP是面向连接的传输协议。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、3次握手过程</h3>

![TCP三次握手](https://i.loli.net/2019/01/10/5c3768734d0d2.png)

#### 第一次握手

>  客户端发起请求，发送 **SYN** (seq=i)到服务器，表示请求连接，之后客户端处于 **SYN_SNET** 状态；

#### 第二次握手

>  服务端接收到客户端的TCP连接请求，最起码能表示双方具有连接能力，然后响应 **ACK**(ack=i+1) + **SYN**(seq=j)表示已确认客户端的请求，并表示可以连接，之后服务端处于 **SYN_RCVD** 状态；

#### 第三次握手

> 客户端收到服务端消息，并回复 **ACK** (ack=j+1, seq=i+1,)确认已收到服务端的消息。之后双端处于  **ESTABLISHED** 状态表示已成功建立 **连接**。

*SYN 是Synchronize Sequence Numbers,同步序列号的缩写；

   ACK是Acknowledgement,确认、答复的缩写。

   SENT 表示发送；

   RCVD 是received缩写，表示已收到；

   ESTABLISHED 表示已建立。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、4次挥手过程</h3>

![TCP四次挥手](https://i.loli.net/2019/01/10/5c376853c0109.png)

#### 第一次挥手

>从双方建立连接的ESTABLISHED状态，客户端想要断开连接，向服务端发送 **FIN** 标识报文段，表示客户端要 **安全地** 断开连接。客户端进入 **FIN_WAIT_1** 状态。

#### 第二次挥手

>服务端收到客户端的终止连接的请求，并回复确认的 **ACK** 报文段，此后服务端进入 **CLOSE_WAIT** 状态，并且内部正在处理数据。

#### 第三次挥手

>待服务端处理好任务，才能发送 **FIN** 结束标识的状态，并发送 **ACK** 报文，确认FIN。

#### 第四次挥手

>客户端收到断开连接FIN标识后，向服务端发送**ACK**确认应答，此时服务端进入TIME-WAIT状态。该状态会持续**2MSL**时间，若该时间段内没有服务端的重发请求的话，就进入CLOSED状态，撤销TCB。当B收到确认应答后，也便进入CLOSED状态，撤销TCB。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、为什么？</h3>

**为什么连接建立需要三次握手，而不是两次握手？** 

> 为了防止已经失效的连接请求报文段突然又传到服务端，因而产生错误。



**为什么A要先进入TIME-WAIT状态，等待2MSL时间后才进入CLOSED状态？** 

> 为了保证服务端能收到客户端的确认应答。 
> 若客户端发完确认应答后直接进入CLOSED状态，那么如果该应答丢失，服务端等待超时后就会重新发送连接释放请求，但此时客户端已经关闭了，不会作出任何响应，因此服务端永远无法正常关闭。