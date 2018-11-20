集群、分布式和微服务有很多相似之处，又有所不同。集群为什么过时？微服务为何兴起？

原文持续更新地址：https://github.com/about-cloud/JavaCore



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、集群</h3>

![集群]()

> 一个小型应用部署在一台性能不错的服务器上，如果访问量和计算量不大，基本上没啥压力。比如个人博客运行在低配置上的服务器，也是能够满足日常使用。随着时间推移，网站的访问量越来越大，业务也越来越复杂，应用程序和服务器的性能受到挑战，传统的做法是 *重构系统* 和升级服务器配置。但这并不是长久之计，因为单台服务器的性能还是有限的。解决办法是加入多台服务器构成 **多处理系统**，同一个应用部署在多台服务器上，每个服务器都会处理完整的任务，这就是 **集群**。每个请求具体交给哪台服务器去处理，通常是配合 *负载均衡/请求分配/集群调度* 来协调工作。这样将较多的访问量分流到多个服务器，每个服务器的压力就小了。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、分布式</h3>

![分布式]()

> *集群* 解决了单机压力问题，但是随着业务的增长，单应用变的越来越复杂，越来越巨大（也就是 *巨型应用* ），维护和开发成本也会变的更高，而且单机不一定有能力处理一次任务，比如要处理 TB级别 日志。将原来单个应用根据实际情况拆分成多个小应用，然后部署在多个服务器上，这些小应用共同组成一个大应用，整个系统就是 **分布式系统** 。集群中的每个应用都是 “大应用”，每个 “大应用” 都能处理完整的业务。而分布式系统中的每个应用都是小应用，每个小应用只能处理特点的任务，但它更加灵活，比如一个简单任务可以仅需要较少的应用组成，如搭积木般灵活。

![block toy](https://images.unsplash.com/photo-1527689638836-411945a2b57c?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=35ac1f0be2205e93f13bf6d2d006c7f1&auto=format&fit=crop&w=500&q=60)

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、微服务</h3>

![微服务]()

*微服务* 是一种架构风格，不一定是单机支撑不了整个应用，而更多强调的是业务上的拆分。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、区别</h3>