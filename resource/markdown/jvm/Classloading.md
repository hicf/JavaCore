<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">类加载机制</h3>

从写好的代码到被执行完成，这都与类加载器机制密不可分。就像我们吃食物一样，先用筷子/勺子/叉子将食物放入嘴中，然后通过舌头上的味觉器官验证食物好不好吃、有没有变馊。

Java的 **“动态性”** 表现在依赖运行期动态加载和动态链接的特点实现。

首先， Java 编译器将Java源代码（.java 文件）编译成 Java字节码（.class 文件）。类加载器负责读取 .class 文件（不限于.class文件），并转换成 `java.lang.Class`类的一个实例。每个这样的实例用来表示一个 class类。所有的类加载器都是 `java.lang.ClassLoader`类的一个实例。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、类加载过程</h3>

![类加载过程](https://upload-images.jianshu.io/upload_images/11476758-1bc7770bbd72e069.png)

#### 1.1、加载

![加载](https://images.pexels.com/photos/188679/pexels-photo-188679.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500)

**"类加载"** 过程比较多，而加载是其中第一个步骤，负责将.class文件加载至内存，但又不仅可以从本地 `.class` 文件加载一个类或接口，也可以从JAR包、WAR包、远程网络资源获取并加载到虚拟机，通过动态代理技术可以在运行时动态生成。比如JSP文件也可以生成对应的Class类。



#### 1.2、验证

![验证](https://images.pexels.com/photos/207585/pexels-photo-207585.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500)

对加载的Class信息首先要做验证，检测文件格式、元数据、字节码等信息是否合法，是否符合虚拟机的最低要求，校验文件是否对虚拟机有害。



#### 1.3、准备

前面的文章讲过，变量所需的内存大小在编译期就已经确定，而类加载的准备阶段是为类变量分配内存并设置类变量的初始值（区别于用户指定值），这些类变量的使用的内存区域是线程共享的方法区（Method Area）。



#### 1.4、解析



#### 1.5、初始化



#### 1.6、使用



#### 1.7、卸载





<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、双亲委派模型</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、Tomcat类加载器</h3>



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">四、OSGi：Java动态化模块化</h3>