<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">依赖注入的循环依赖问题</h3>

![循环依赖](https://img-blog.csdn.net/20170912082357749?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDg1MzI2MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 什么是循环依赖？

> 在对象的引用中，两个及两个以上的类互相引用，只要形成环形就会产生 **循环依赖**。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">依赖注入产生循环依赖的三种方式</h3>

#### 1、构造方法注入产生的循环依赖



#### 2、setter 方法单例模式注入产生的循环依赖



#### 3、setter 方法原型模式注入产生的循环依赖