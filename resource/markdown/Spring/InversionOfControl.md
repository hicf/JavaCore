<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">控制反转IoC</h3>

> IoC，全称 *Inversion of Control* ，中文称 控制反转。

![Control](https://images.pexels.com/photos/935019/pexels-photo-935019.jpeg?auto=compress&cs=tinysrgb&h=350)

首先要知道，什么是 *控制(Control)* ？要控制谁？或者说谁要被控制？在面向对象编程中，对象A内部依赖对象B，那么就需要在对象A中实例化一个对象B，到底谁来实例化对象B，那么谁就是对象B的 **控制者**(controller)。我们通常的做法是在对象A中实例化一个对象B，那么 *控制权* 也就在对象A中，因为对象A也可以选择不去实例化，控制权在需求方。



![new](https://images.pexels.com/photos/12733/pexels-photo-12733.jpeg?auto=compress&cs=tinysrgb&h=350)

独木不成林。面向对象编程的特点就是以 **对象** 为基本单位。大家本着 *谁用谁创建* 的原则。比如：发布评论时，要先剔除敏感词，然后才能发布，发布功能依赖敏感词剔除功能：

```java
package com.foovv.example;

/**
 * 评论功能类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class Comment {
    /**
     * 引入评论语句预处理工具类，并主动实例化对象
     */
    private CommentPretreatmentUtils commentPretreatmentUtils = new CommentPretreatmentUtils();

    /**
     * 发布评论
     *
     * @param comment 评语
     * @return 返回true表示评论成功，返回false表示评论失败
     */
    public boolean uploadComment(String comment) {

        // 剔除敏感词
        this.commentPretreatmentUtils.removesensitiveWords(comment);
		... 其他操作 ...
    }
}

```

```java
package com.foovv.example;

/**
 * 评论预处理工具类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class CommentPretreatmentUtils {
    /**
     * 删除评语中的敏感词
     *
     * @param comment 评语
     * @return 删除明敏感词后的评论
     */
    public String removesensitiveWords(String comment) {
        ... 其他操作 ...
        return ...;
    }

}

```

 *Comment* 类 **控制** (control) 着所依赖对象的实例化的工作，它们之间的耦合度大大增加，万一依赖的类发生了变化，对于被依赖类无疑是致命一击。对于 *Comment* 来说，它的工作就是发布或删除评论，至于要剔除敏感词或者增加某些语句与它无关，那么至于要注入什么对象实例，也不应该与它有关，它只需要使用被注入对象实例的方法罢了。

![集权](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542703412920&di=153ab0debf76c2d790da56b32a307df2&imgtype=0&src=http%3A%2F%2F07imgmini.eastday.com%2Fmobile%2F20181030%2F20181030075129_90828cc2c29cf13e1327e9b7165881fd_10.jpeg)

控制权的交接给统一的系统负责，该系统负责对象的实例化、传递引用、初始化值等。控制权就发生了变化，外国人称为 **Inversion of Control**，IoC，中文意思是 **控制反转**。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">依赖注入DI</h3>

> DI，全称 *Dependency Injection* ，中文称 依赖注入。

控制实例化对象的权利交给 *统一管理系统* 来负责了，如果我们需要其他被依赖的对象时怎么 ~~获取~~ 呢？错错错，压根不需要主动去获取，而是等待被注入所依赖的对象实例。



#### 依赖注入的三种方式：

> 接着上面评论功能的代码，被依赖的 *CommentPretreatmentUtils* 不变，修改 *Comment* 类。

#### 1、构造方法注入

```java
package com.foovv.example;

/**
 * 评论功能类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class Comment {
    /**
     * 引入评论语句预处理工具类，并主动实例化对象
     */
    private CommentPretreatmentUtils commentPretreatmentUtils;
    
    /**
     * 构造方法注入依赖对象
     *
     * @param commentPretreatmentUtils 被依赖对象
     */
    public Comment(CommentPretreatmentUtils commentPretreatmentUtils) {
        this.commentPretreatmentUtils = commentPretreatmentUtils;
    }
    
    /**
     * 发布评论
     *
     * @param comment 评语
     * @return 返回true表示评论成功，返回false表示评论失败
     */
    public boolean uploadComment(String comment) {

        // 剔除敏感词
        this.commentPretreatmentUtils.removesensitiveWords(comment);
		... 其他操作 ...
    }
}
```

使用时：

```java
public class Client {
    public static void main(String[] args) {
        // 通过构造方法直接注入被依赖的对象实例
        Comment comment = new Comment(new CommentPretreatmentUtils());
        comment.uploadComment("文章很不错，更多文章在：https://github.com/about-cloud/JavaCore");

    }
}
```



这样还是有很强的依赖：

```java
public class Comment {
    /**
     * Comment类还是依赖CommentPretreatmentUtils类，
      * 如果要切换其他类还是很麻烦
     */
    private CommentPretreatmentUtils commentPretreatmentUtils;
		... 其他操作 ...
}
```

---

根据 [迪米特法则](https://github.com/about-cloud/JavaCore) 现在要将被依赖的类以接口化编程实现，降低耦合度，只对外暴露方法服务，屏蔽实现细节，UML图如下：

![UML]()

**评论预处理工具对外服务的接口**

```java
package com.foovv.example;

/**
 * 评论预处理工具对外服务类
 * 
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public interface CommentPretreatmentService {
    /**
     * 删除评论中的敏感词
     *
     * @param comment 评语
     * @return 删除明敏感词后的评论
     */
    String removesensitiveWords(String comment);
}
```

**评论预处理工具的实现类** 

```java
package com.foovv.example;

/**
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class CommentPretreatmentUtils implements CommentPretreatmentService {

    /**
     * 删除评论中的敏感词
     *
     * @param comment 评语
     * @return 删除明敏感词后的评论
     */
    @Override
    public String removesensitiveWords(String comment) {
        return comment;
    }

}
```

**评论功能类：**

```java
package com.foovv.example;

/**
 * 评论功能类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class Comment {
    /**
     * 评论预处理服务类
     */
    private CommentPretreatmentService commentPretreatmentService;

    /**
     * 构造方法注入依赖对象
     *
     * @param commentPretreatmentService 被依赖对象
     */
    public Comment(CommentPretreatmentService commentPretreatmentService) {
        this.commentPretreatmentService = commentPretreatmentService;
    }

    /**
     * 发布评论
     *
     * @param comment 评论
     * @return 返回true表示评论成功，返回false表示评论失败
     */
    public boolean uploadComment(String comment) {

        // 剔除敏感词
        this.commentPretreatmentService.removesensitiveWords(comment);
		... 其他操作 ...
    }
}
```

上面的 *Comment* 最多也就知道 *CommentPretreatmentService* 接口中的方法（这里不考虑静态变量和default语句），不会知道具体实现，而且切换具体实现类也很方法。

使用时：

```java
public class Client {
    public static void main(String[] args) {
        // 通过构造方法直接注入被依赖的对象实例
        // 被注入的实例对象一定要实现于CommentPretreatmentService接口
        Comment comment = new Comment(new CommentPretreatmentUtils());
        comment.uploadComment("文章很不错，更多文章在：https://github.com/about-cloud/JavaCore");

    }
}
```



#### 2、setter方法注入

接上面的构造方法注入，区别是将构造方法注入改成setter方法注入，客户端使用时通过调用setter方法注入即可：

**评论功能类：**

```java
package com.foovv.example;

/**
 * 评论功能类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class Comment {
    /**
     * 评论预处理服务类
     */
    private CommentPretreatmentService commentPretreatmentService;

    /**
     * CommentPretreatmentService 的 setter 方法
     *
     * @param commentPretreatmentService set 注入实例对象
     */
    public void setCommentPretreatmentService(CommentPretreatmentService commentPretreatmentService) {
        this.commentPretreatmentService = commentPretreatmentService;
    }

    /**
     * 发布评论
     *
     * @param comment 评论
     * @return 返回true表示评论成功，返回false表示评论失败
     */
    public boolean uploadComment(String comment) {

        // 剔除敏感词
        this.commentPretreatmentService.removesensitiveWords(comment);
		... 其他操作 ...
    }
}
```

使用时：

```java
public class Client {
    public static void main(String[] args) {
        // 评论功能实例
        Comment comment = new Comment();
        // setter 注入被依赖的实例对象
        comment.setCommentPretreatmentService(new CommentPretreatmentUtils());
        comment.uploadComment("文章很不错，更多文章在：https://github.com/about-cloud/JavaCore");

    }
}
```



##### 3、接口注入



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">手写简易版的IoC容器</h3>

首先在使用前通过文件或其他方式来映射 *依赖对象* 与 *被依赖对象* 之间的关系，也包括属性默认值的填充等等，最后*统一管理系统*  会去处理，然后 *统一管理系统* 生成大量的 **实例对象**，屏蔽其他细节，在外界看来它就像一个存放对象的容器，所以常称为 **IoC容器**。最后 *IoC容器* 这个统一管理系统会负责管理依赖关系。



