<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Java内存模型之：final关键字</h3>



#### 本文关注点
* **final使用**
* **final底层实现原理**

---
#### 一、final使用
> * **修饰变量**：被final修饰的变量是一个常量只能赋值一次，既可以修饰成员变量，有可以修饰局部变量。
> * **修饰方法**：被final修饰的方法不可以被复写。
> * **修饰类**：被final修饰的类不可以被继承。为了避免被继承，被子类复写功能。

#### 二、final底层实现原理
##### 对于final域，编译器和处理器要遵守两个重排序规则：
> * 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
> * 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。
>   （总之是先赋值再使用）

> 参考资料：
> [深入理解Java内存模型（六）——final](http://www.infoq.com/cn/articles/java-memory-model-6)