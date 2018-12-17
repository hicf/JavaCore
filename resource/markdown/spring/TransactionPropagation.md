<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">Spring事务传播机制</h3>

**事务传播：** 事务方法与事务方法嵌套调用时，描述事务如何传播的行为。



Spring 框架的 *TransactionDefinition* 接口提供了七种事务传播的行为类型（或等级）

| 事务传播行为类型          | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| **PROPAGATION_REQUIRED**  | 如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，这个是常用的而且是默认的事务传播类型。 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。               |
| PROPAGATION_REQUIRES_NEW  | 不管当前有没有事务，都会新建事务，如果当前存在事务，把当前事务挂起。 |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
| PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

