<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">动态代理</h3>

> *Spring* 框架的核心是 IoC容器和AOP编程，

![代理者模式]()

jdk动态代理与CGLib



> 比如我们想购买 *迪士尼*  的门票（也可能是退票），一般情况我们会直接到迪士尼官网注册账号、绑定手机号、也有可能要实名认证、支付时还要绑定银行卡、退款这么麻烦的操作。现在我可以使用比如携程、飞猪等代理商，就可以解决上面的麻烦操作。
>
> 下面通过 三种 代理方式来看看各自的实现：

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、静态代理</h3>

##### 1、定义购票/退票功能的接口：

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * 售票服务
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public interface TicketService {

    /**
     * 购票
     *
     * @param money 付款金额
     * @return 是否成功购票
     */
    boolean buyTicket(BigDecimal money);

    /**
     * 退票
     *
     * @param orderNo 退票订单号
     * @return 是否成功退票
     */
    boolean refundTicket(String orderNo);

}
```



##### 2、具体的实现购票/退票的类 --迪士尼官网票务系统：

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * 具体的实现购票/退票的类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class DisneyOfficialWebsite implements TicketService {
    @Override
    public boolean buyTicket(BigDecimal money) {
        System.out.println("迪士尼票务系统收到" + money + "元，已购票");
        return true;
    }

    @Override
    public boolean refundTicket(String orderNo) {
        System.out.println("迪士尼票务系统正在为订单号：" + orderNo + "退票，已退票");
        return true;
    }
}
```



##### 3、代理商（比如携程）：

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * 携程票务代理商
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class CtripTicketProxy {
    /**
     * 静态代理者所依赖售票/退票服务
     */
    private TicketService ticketService;

    /**
     * 【关键】通过构造方法注入被代理的对象
     *
     * @param ticketService 被代理对象
     */
    public CtripTicketProxy(TicketService ticketService) {
        this.ticketService = ticketService;
    }

    // ---------------- 具体使用被代理对象的方法功能 ----------------

    /**
     * 购票
     *
     * @param money 付款金额
     * @return 是否成功购票
     */
    public boolean buyTicket(BigDecimal money) {
        System.out.println("携程开始代理购票:");
        boolean result = this.ticketService.buyTicket(money);
        if (result) {
            System.out.println("携程代理购票成功，已出票。");
        } else {
            System.out.println("携程代理购票失败，无法出票。");
        }

        return result;
    }

    /**
     * 退票
     *
     * @param orderNo 退票订单号
     * @return 是否成功退票
     */
    public boolean refundTicket(String orderNo) {
        System.out.println("携程开始代理退票：");
        boolean result = this.ticketService.refundTicket(orderNo);
        if (result) {
            System.out.println("携程代理退票成功，已退票。");
        } else {
            System.out.println("携程代理退票失败，无法退票。");
        }
        return result;
    }

}

```



##### 4、客户端购票/退票

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * 客户端
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class Client {
    public static void main(String[] args) {
        // 创建一个携程代理商，并注入迪士尼官网票务系统
        CtripTicketProxy proxy = new CtripTicketProxy(new DisneyOfficialWebsite());
        // 携程代理商买票
        proxy.buyTicket(new BigDecimal("9999"));
        // 携程代理商退票
        proxy.refundTicket("AAA809876");
    }
}
```

执行结果：

```mysql
携程开始代理购票:
迪士尼票务系统收到9999元，已购票
携程代理购票成功，已出票。
携程开始代理退票：
迪士尼票务系统正在为订单号：AAA809876退票，已退票
携程代理退票成功，已退票。

Process finished with exit code 0
```



##### 总结：

> 静态代理模式解决了客户端与服务端的耦合度，但是写法较麻烦，每一个实际执行的方法都需要代理类中一一写出代理方法，如果在方法多的情况下，实在是太麻烦，而且灵活性低。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、jdk动态代理</h3>

> 原理：通过反射动态获取类信息，被代理的类一定要有接口，而且被执行的方法一定实现至接口。

##### 1、定义购票/退票功能的接口：（与上面静态代理一样）

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * 售票服务
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public interface TicketService {

    /**
     * 购票
     *
     * @param money 付款金额
     * @return 是否成功购票
     */
    boolean buyTicket(BigDecimal money);

    /**
     * 退票
     *
     * @param orderNo 退票订单号
     * @return 是否成功退票
     */
    boolean refundTicket(String orderNo);

}
```



##### 2、具体的实现购票/退票的类 --迪士尼官网票务系统：（与上面静态代理一样）

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * 具体的实现购票/退票的类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class DisneyOfficialWebsite implements TicketService {
    @Override
    public boolean buyTicket(BigDecimal money) {
        System.out.println("迪士尼票务系统收到" + money + "元，已购票");
        return true;
    }

    @Override
    public boolean refundTicket(String orderNo) {
        System.out.println("迪士尼票务系统正在为订单号：" + orderNo + "退票，已退票");
        return true;
    }
}
```



##### 3、代理商（比如携程）：（关键点）

```java
package com.foovv.example;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * jdk动态代理类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class CtripTicketJdkProxy implements InvocationHandler {
    /**
     * 被代理对象
     */
    private Object object;

    /**
     * 构造方法注入被代理对象（这点和静态代理一样）
     */
    public CtripTicketJdkProxy(Object object) {
        this.object = object;
    }

    /**
     * 获取代理者的实例对象
     */
    public Object getProxyInstance() {
        return Proxy.newProxyInstance(this.object.getClass().getClassLoader(),
                this.object.getClass().getInterfaces(),
                this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 增加日志功能
        System.out.println("携程代理商开始执行" + method.getName() + "操作：");
        Object result = method.invoke(this.object, args);
        System.out.println("携程代理商执行" + method.getName() + "操作结束。");
        return result;
    }
}

```



##### 4、客户端购票/退票

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * 客户端
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class Client {
    public static void main(String[] args) {
        // 创建一个携程代理商，并注入被代理的实际迪士尼官方票务系统
        // 然后获取一个携程代理商的实例对象
       TicketService proxy = (TicketService) new CtripTicketJdkProxy(new DisneyOfficialWebsite()).getProxyInstance();
        // 携程代理商买票
        proxy.buyTicket(new BigDecimal("999"));
        // 携程代理商退票
        proxy.refundTicket("c67394279");
    }
}
```

执行结果：

```mysql
携程代理商开始执行buyTicket操作：
迪士尼票务系统收到999元，已购票
携程代理商执行buyTicket操作结束。
携程代理商开始执行refundTicket操作：
迪士尼票务系统正在为订单号：c67394279退票，已退票
携程代理商执行refundTicket操作结束。

Process finished with exit code 0
```



##### 总结：

> jdk动态代理较静态代理更加灵活，不用每次都写所有代理方法，而且还可以 `invoke` 方法中拦截方法、动态修改参数等。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">三、CGLib动态代理</h3>

> 原理：被代理的方法不能被 final 修饰

一定要引入第三方CGLib包

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.4</version>
</dependency>
```



##### 1、具体的购票/退票类 --迪士尼官网票务系统：（不需要实现接口）

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class DisneyOfficialWebsite {
    /**
     * 购票
     *
     * @param money 付款金额
     * @return 是否成功购票
     */
    public boolean buyTicket(BigDecimal money) {
        System.out.println("迪士尼票务系统收到" + money + "元，已购票");
        return true;
    }

    /**
     * 退票
     *
     * @param orderNo 退票订单号
     * @return 是否成功退票
     */
    public boolean refundTicket(String orderNo) {
        System.out.println("迪士尼票务系统正在为订单号：" + orderNo + "退票，已退票");
        return true;
    }
}

```



##### 2、代理商（比如携程）：（关键点）

```java
package com.foovv.example;


import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * CGLib动态代理类
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class CtripTicketCGLibProxy implements MethodInterceptor {
    /**
     * 被代理的对象
     */
    private Object object;

    /**
     * 构造方法注入被代理对象
     *
     * @param object 被代理的对象
     */
    public CtripTicketCGLibProxy(Object object) {
        this.object = object;
    }

    public Object getProxyInstance() {
        // 创建一个增强器
        Enhancer enhancer = new Enhancer();
        // 设置代理者的父类
        enhancer.setSuperclass(this.object.getClass());
        // 设置回调对象
        enhancer.setCallback(this);
        return enhancer.create();
    }


    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("携程代理商开始执行" + method.getName() + "操作：");
        Object result = method.invoke(this.object, objects);
        System.out.println("携程代理商执行" + method.getName() + "操作结束。");
        return result;
    }
}

```



##### 3、客户端购票/退票

```java
package com.foovv.example;

import java.math.BigDecimal;

/**
 * 客户端
 *
 * @author 代码风水师 https://github.com/about-cloud/JavaCore
 * @version jdk1.8
 */
public class Client {
    public static void main(String[] args) {
        // 创建一个方法拦截器的子类 -- 也就是动态代理类，并注入被代理的对象
        // 然后获取动态代理类的实例对象
        DisneyOfficialWebsite proxy = (DisneyOfficialWebsite) new CtripTicketCGLibProxy(new DisneyOfficialWebsite())
                .getProxyInstance();
		// 携程代理商买票
        proxy.buyTicket(new BigDecimal("666.90"));
        // 携程代理商退票
        proxy.refundTicket("AAA8708980");

    }
}

```

执行结果：

```mysql
携程代理商开始执行buyTicket操作：
迪士尼票务系统收到666.90元，已购票
携程代理商执行buyTicket操作结束。
携程代理商开始执行refundTicket操作：
迪士尼票务系统正在为订单号：AAA8708980退票，已退票
携程代理商执行refundTicket操作结束。

Process finished with exit code 0
```

