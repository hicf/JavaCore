<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">反射机制</h3>

> *Java反射机制* 是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种 **动态** 获取的信息以及动态调用对象的方法的功能称为java语言的 **反射机制**。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">获取Class类对象的三种方式</h3>

> 万物皆对象，包括 *类* 也有对象，称为 **类对象**。要想获取类的信息首先要获取 **类**。

#### Class类的与反射有关的重要方法：

| 返回类型                   | 方法名                                                       | 含义                                                         |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| static Class<?>            | forName(String className)                                    | 返回给定字符串(全限定类名)的对应类的类对象                   |
| static Class<?>            | forName(String name, boolean initialize, ClassLoader loader) | 给定是否初始化、类加载器，返回给定字符串(全限定类名)的对应类的类对象 |
| AnnotatedType[]            | getAnnotatedInterfaces()                                     | 返回 AnnotatedType对象的数组。                               |
| AnnotatedType              | getAnnotatedSuperclass()                                     | 返回父类的注解的AnnotatedType                                |
| <A extends Annotation> A   | getAnnotation(Class<A> annotationClass)                      | 返回指定类型的注释，如果不存在则返回null。                   |
| Annotation[]               | getAnnotations()                                             | 过去该对象上的注解，以注解数组的形式返回。                   |
| <A extends Annotation> A[] | getAnnotationsByType(Class<A> annotationClass)               | 返回指定类型的注解数组。                                     |
| Class<?>[]                 | getClasses()                                                 | 返回类/接口内部类的、从父类/接口继承的内部类的数组。         |
| ClassLoader                | getClassLoader()                                             | 返回该类的类加载器。                                         |
| Class<?>                   | getComponentType()                                           | 返回表示数组组件类型的 Class。如果此类表示数组类，返回表示此类组件类型的 Class。否则返回null。 |
| Constructor<T>             | getConstructor(Class<?>... parameterTypes)                   | 通过指定参数类型，返回一个 Constructor 公共构造方法类的类对象。 |
| Constructor<?>[]           | getConstructors()                                            | 返回一个包含所有 Constructor 公共构造方法类的类对象数组。    |
| <A extends Annotation> A   | getDeclaredAnnotation(Class<A> annotationClass)              | 返回直接存在于此元素上的该类型注释。                         |
| Annotation[]               | getDeclaredAnnotations()                                     | 返回直接存在于此元素上的所有注释。                           |
| <A extends Annotation> A[] | getDeclaredAnnotationsByType(Class<A> annotationClass)       | 返回指定注解类型的存在于此元素上的注解数组。                 |
| Class<?>[]                 | getDeclaredClasses()                                         | 返回类定义的公共的内部类，不包括继承的。                     |
| Constructor<T>             | getDeclaredConstructor(Class<?>... parameterTypes)           | 通过指定参数类型，返回一个 Constructor 公共构造方法类的类对象。可获取非公共构造方法。 |
| Constructor<?>[]           | getDeclaredConstructors()                                    | 返回 Constructor 对象的一个数组，这些对象反映此 Class 对象表示的类声明的所有构造方法。不仅是公共的。 |
| Field                      | getDeclaredField(String name)                                | 返回一个 Field 对象，该对象反映此 Class 对象所表示的类或接口的指定已声明字段。 |
| Field[]                    | getDeclaredFields()                                          | 返回 Field 对象的一个数组，这些对象反映此 Class 对象所表示的类或接口所声明的所有字段。 |
| Method                     | getDeclaredMethod(String name, Class<?>... parameterTypes)   | 返回一个 Method 对象，该对象反映此 Class 对象所表示的类或接口的指定已声明方法。 |
| Method[]                   | getDeclaredMethods()                                         | 返回 Method 对象的一个数组，这些对象反映此 Class 对象表示的类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。 |
| Class<?>                   | getDeclaringClass()                                          | 如果此 Class 对象所表示的类或接口是另一个类的成员，则返回的 Class 对象表示该对象的声明类。即返回所在类的class对象，对成员class对象有效。 |
| Class<?>                   | getEnclosingClass()                                          | 返回底层类的立即封闭类。                                     |
| Constructor<?>             | getEnclosingConstructor()                                    | 如果该 Class 对象表示构造方法中的一个本地或匿名类，则返回 Constructor 对象，它表示底层类的立即封闭构造方法。 |
| Method                     | getEnclosingMethod()                                         | 如果此 Class 对象表示某一方法中的一个本地或匿名类，则返回 Method 对象，它表示底层类的立即封闭方法。 |
| T[]                        | getEnumConstants()                                           | 如果此 Class 对象不表示枚举类型，则返回枚举类的元素或 null。 |
| Field                      | getField(String name)                                        | 返回一个 Field 对象，它反映此 Class 对象所表示的类或接口的指定公共成员字段。 |
| Field[]                    | getFields()                                                  | 返回一个包含某些 Field 对象的数组，这些对象反映此 Class 对象所表示的类或接口的所有可访问公共字段。 |
| Type[]                     | getGenericInterfaces()                                       | 返回表示某些接口的 Type，这些接口由此对象所表示的类或接口直接实现。 |
| Type                       | getGenericSuperclass()                                       | 返回表示此 Class 所表示的实体（类、接口、基本类型或 void）的直接超类的 Type。 |
| Class<?>[]                 | getInterfaces()                                              | 获取此类/接口对象所的实现接口。                              |
| Method                     | getMethod(String name, Class<?>... parameterTypes)           | 返回一个 Method 对象，它反映此 Class 对象所表示的类或接口的指定公共成员方法。 |
| Method[]                   | getMethods()                                                 | 返回一个包含某些 Method 对象的数组，这些对象反映此 Class 对象所表示的类或接口（包括那些由该类或接口声明的以及从超类和超接口继承的那些的类或接口）的公共 member 方法。 |
| int                        | getModifiers()                                               | 返回此类或接口以整数编码的 Java 语言修饰符。                 |
| String                     | getName()                                                    | 以 String 的形式返回此 Class 对象所表示的实体（类、接口、数组类、基本类型或 void）名称。 |
| Package                    | getPackage()                                                 | 获取此类的包。                                               |
| ProtectionDomain           | getProtectionDomain()                                        | 返回该类的 ProtectionDomain。                                |
| URL                        | getResource(String name)                                     | 查找带有给定名称的资源。                                     |
| InputStream                | getResourceAsStream(String name)                             | 查找具有给定名称的资源。                                     |
| Object[]                   | getSigners()                                                 | 获取此类的标记。                                             |
| String                     | getSimpleName()                                              | 返回源代码中给出的底层类的简称。                             |
| Class<? super T>           | getSuperclass()                                              | 返回表示此 Class 所表示的实体（类、接口、基本类型或 void）的超类的 Class。 |
| String                     | getTypeName()                                                | 返回该类型的名称                                             |
| TypeVariable<Class<T>>[]   | getTypeParameters()                                          | 按声明顺序返回 TypeVariable 对象的一个数组，这些对象表示用此 GenericDeclaration 对象所表示的常规声明来声明的类型变量。 |
| boolean                    | isAnnotation()                                               | 如果此 Class 对象表示一个注释类型则返回 true。               |
| boolean                    | isAnnotationPresent(Class<? extends Annotation> annotationClass) | 如果指定类型的注释存在于此元素上，则返回 true，否则返回 false。 |
| boolean                    | isAnonymousClass()                                           | 当且仅当底层类是匿名类时返回 true。                          |
| boolean                    | isArray()                                                    | 判定此 Class 对象是否表示一个数组类。                        |
| boolean                    | isAssignableFrom(Class<?> cls)                               | 判定此 Class 对象所表示的类或接口与指定的 Class 参数所表示的类或接口是否相同，或是否是其超类或超接口。 |
| boolean                    | isEnum()                                                     | 当且仅当该类声明为源代码中的枚举时返回 true。                |
| boolean                    | isInstance(Object obj)                                       | 判定指定的 Object 是否与此 Class 所表示的对象赋值兼容。      |
| boolean                    | isInterface()                                                | 判定指定的 Class 对象是否表示一个接口类型。                  |
| boolean                    | isLocalClass()                                               | 当且仅当底层类是本地类时返回 true。                          |
| boolean                    | isMemberClass()                                              | 当且仅当底层类是成员类时返回 true。                          |
| boolean                    | isPrimitive()                                                | 判定指定的 Class 对象是否表示一个基本类型。                  |
| boolean                    | isSynthetic()                                                | 如果此类是复合类，则返回 true，否则 false。有匿名类部类的类可以称作为复合类。 |
| T                          | newInstance()                                                | 调用默认构造方法创建此 Class 对象所表示的类的一个新实例。    |
| String                     | toGenericString()                                            | 返回该对象的描述，包含标识符等。                             |



#### 获取Class类对象方式一：

```java
// 通过类的全限定类名(包名+类名)获取类
Class clazz = Class.forName("类的全限定类名(包名+类名)");
```

#### 获取Class类对象方式二：

```java
// 直接获取类
Class clazz = Object.class;
```

#### 获取Class类对象方式三：

```java
// 通过实例对象获取其类
Class clazz = object.getClass();
```

*根据类加载机制，运行时一个类仅有一个类对象。



<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">java.lang.reflect 包中的类</h3>

> 万物皆对象，不仅 *类* 有类对象，类的方法、字段，甚至修饰符都可以有 *对象*。
>
> *java.lang.reflect* 包提供了用于通过反射获取 *类信息(方法、属性、修饰符、参数等)*的类、接口、异常。下面主重介绍一下有关反射的*类*：

| 类                | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| AccessibleObject  | AccessibleObject类是Field，Method和Constructor对象的基类。 它提供了将反射的对象标记为在使用时取消默认 Java 语言访问控制检查的能力。对于公共成员、默认（包）访问成员、受保护成员和私有成员，在分别使用 Field、Method 或 Constructor 对象来设置或获得字段、调用方法，或者创建和初始化类的新实例的时候，会执行访问检查。 |
| Array             | Array类提供静态方法来动态创建和访问Java数组。它允许在获取或设置操作期间扩大转换，但如果发生缩小转换，则抛出IllegalArgumentException 。 |
| Constructor       | Constructor提供了一个类的单个构造函数的信息和访问。          |
| Executable        | 用于 Method 和 Constructor的公共功能的共享超类。（jdk1.8 新加的类） |
| Field             | Field提供有关类或接口的单个字段的信息和动态访问。 反射的字段可以是类（静态）字段或实例字段。 |
| Method            | Method 提供有关类和接口上单一方法的信息和访问权限。 反映的方法可以是类方法或实例方法（包括抽象方法）。 |
| Modifier          | Modifier类提供了static方法和常量来解码类和成员访问修饰符。 修饰符集合被表示为具有表示不同修饰符的不同位位置的整数。 |
| Parameter         | 有关方法参数的信息。 Parameter提供了有关方法参数的信息，包括其名称和修饰符。 它还提供了获取参数属性的替代方法。 |
| Proxy             | Proxy提供了创建动态代理类和实例的静态方法，它也是由这些方法创建的所有动态代理类的超类。（*关于Proxy的使用及动态代理模式，我会单独拿出一篇文章讲解，详见：https://github.com/about-cloud/JavaCore） |
| ReflectPermission | 反射操作的权限类。                                           |

#### 

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">反射的实际操作</h3>

#### 案例1：

> 现在有个需求，想获取一个 用户实体 中所有的字段。

```java
package com.foovv.example;
/**
 * 用户实体类
 *
 * @author 代码风水师
 * @version jdk1.8
 */
public class User {
    /** 用户id */
    private String id;

    /** 用户名称 */
    private String userName;

    /** 用户性别 */
    private String userGender;

    /** 用户年龄 */
    private Integer userAge;

    /** 用户邮箱 */
    private String email;

    /* getter & setter 方法略 */
}
```

```java
package com.foovv.example;

/** 要反射类中的字段，就要引入reflect.Field类 */
import java.lang.reflect.Field;

/**
 * 反射字段的demo
 *
 * @author 代码风水师
 * @version jdk1.8
 */
public class ReflectDemo {
    public static void main(String[] args) throws ClassNotFoundException {
		// 1、要想获取类的字段，就先要获取该类。下面通过全限定类名来获取的。
        Class user = Class.forName("com.foovv.example.User");
		// 2、上面已得到User类的类对象，那么下面就获取其所有字段的类对象数组
        Field[] fields = user.getDeclaredFields();

        // 3、遍历字段的数组集合，一个一个字段去处理
        for (Field field : fields) {
            // 此处的是打印User类对象中的字段名称
            System.out.println(field.getName());
        }

    }
}
```

输出结果：

```mysql
id
userName
userGender
userAge
email

Process finished with exit code 0
```



#### 案例2：

> 在很多框架中，常使用注解来标记类、方法、字段的特征。比如持久化框架常用来反射生成SQL，下面就动手实战一下：

实体：

```java
package com.foovv.example;
/** 描述略 */
public class User {
    private String id;

    private String userName;

    private String userGender;

    private Integer userAge;

    private String email;

    /* getter & setter 方法略 */
}
```

通过反射生成SQL语句：

```mysql
SELECT id, user_name, user_gender, user_age,email FROM user;
```



1、定义两个注解类，一个注解类Table标注实体对应的表名，另一个注解类Column标注对应的数据表的列名：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Table {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Column {
    String value();
}
```

2、对实体类及其字段使用注解标注：

```java
@Table("user")
public class User {
    @Column("id")
    private String id;

    @Column("user_name")
    private String userName;

    @Column("user_gender")
    private String userGender;

    @Column("user_age")
    private Integer userAge;

   @Column("email")
    private String email;

    /* getter & setter 方法略 */
}
```



3、上面做了处理，下面开始对这个实体类做反射、获取注解：

```java
public class ReflectSql {
    public static void main(String[] args) throws ClassNotFoundException {
        // 拼接的SQL
        StringBuilder sql = new StringBuilder();

        // 1、要想获取类的字段，就先要获取该类。下面通过全限定类名来获取的。
        Class user = Class.forName("com.foovv.example.User");

        // 2、获取实体类上的Table注解，然后通过value()获取对应的表名
        // 当前你也可以使用getDeclaredAnnotations()获取注解数组，然后遍历
        Table table = (Table) user.getDeclaredAnnotation(Table.class);
        String tableName = table.value();

        // 3、获取字段上的注解，然后获取通过value()获取对应的数据表的列名
        Field[] fields = user.getDeclaredFields();
        sql.append("SELECT ");
        // 遍历每个字段
        for (int i = 0; i < fields.length; i++) {
            // 获取字段上Column注解
            Column column = fields[i].getAnnotation(Column.class);
            // 进而获取value()值，既对应指定的数据表列名
            // 拼接SQL
            sql.append(column.value());
            if (i < fields.length - 1) {
                sql.append(", ");
            }
        }
        sql.append(" FROM ");
        sql.append(tableName);
        sql.append(";");

        System.out.println(sql);
    }
}
```

运行结果：

```mysql
SELECT id, user_name, user_gender, user_age, email FROM user;

Process finished with exit code 0
```