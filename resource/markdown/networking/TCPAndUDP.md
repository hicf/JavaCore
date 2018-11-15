<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">传输层协议</h3>

> TCP 和 UDP 都是传输层协议。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">TCP</h3>

> TCP, Transmission Control Protocol,传输控制协议，是面向 **连接** 的传输协议，既传输数据前先进行客户端和服务端建立 **可靠** 连接，连接成功后才进行传输数据。TCP每次连接都需要通过 **[”三次握手“](https://github.com/about-cloud/JavaCore)** 才能建立.



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">UDP</h3>

> UDP,User Data Protocol,用户数据报协议，面向 **非连接** 的传输协议，既传输数据前客户端和服务端 **无需** 建立连接，数据也可被传输，但 *被传输* 不带表一定传输成功。传世时直接将数据包丢给对方。



#### 对比

| 对比维度\传输协议 | TCP                    | UDP                  |
| :---------------- | ---------------------- | -------------------- |
| 是否需要连接      | 需要                   | 不需要               |
| 是否是可靠传输    | 是                     | 否                   |
| 速度              | 慢                     | 快                   |
| 使用环境          | 传输数据量少、可靠性高 | 传输数量多、可靠性低 |

