<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">类加载机制</h3>

从写好的代码到被执行完成，这都与类加载器机制密不可分。就像我们吃食物一样，先用筷子/勺子/叉子将食物放入嘴中，然后通过舌头上的味觉器官验证食物好不好吃、有没有变馊。

首先， Java 编译器将Java源代码（.java 文件）编译成 Java字节码（.class 文件）。类加载器负责读取 .class 文件，并转换成 `java.lang.Class`类的一个实例。每个这样的实例用来表示一个 class类。所有的类加载器都是 `java.lang.ClassLoader`类的一个实例。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、类加载过程</h3>

![类加载过程](https://upload-images.jianshu.io/upload_images/11476758-1bc7770bbd72e069.png)

#### 1.1、加载



#### 1.2、验证



#### 1.3、准备



#### 1.4、解析



#### 1.5、初始化



#### 1.6、使用



#### 1.7、卸载





<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、双亲委派模型</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、Tomcat类加载器</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、OSGi：Java动态化模块化</h3>