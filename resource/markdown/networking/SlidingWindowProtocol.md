![滑动窗口](https://i.loli.net/2019/01/03/5c2d78ac2d805.jpeg)

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">什么是滑动窗口协议</h3>

**滑动窗口协议**（*Sliding Window Protocol*），属于[TCP协议](https://baike.baidu.com/item/TCP%E5%8D%8F%E8%AE%AE)的一种应用，用于网络数据传输时的 **流量控制**，以 **避免拥塞** 的发生。该协议允许发送方在停止并等待确认前发送多个数据分组。由于发送方不必每发一个分组就停下来等待确认，因此该协议可以加速数据的传输，提高网络吞吐量。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">滑动窗口协议的机制</h3>

![滑动窗口协议的机制]()

