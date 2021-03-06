# spring

springbean生命周期：实例化、属性赋值、初始化、使用、销毁。

spring是一些模块的集合，包括核心容器（DI，验证，类型转换，资源，国际化，数据绑定，事件，事务），数据访问DAO/集成，Web，AOP，工具，消息JMS，测试模块。

> spring core，spring aspects，spring aop，spring jdbc，spring jms，orm，web，test。
>
> spring aop是运行时增强，aspects是编译时增强。
>
> 注入对象可以是singleton、prototype，request，session。

**spring框架中用到哪些设计模式**

1. 工厂设计模式：beanFactory、ApplicationContext创建bean对象。
2. 代理设计模式：AOP
3. 单例设计模式：默认都是单例的注入对象。
4. 模板方法模式：spring中的jdbctemplate、hibernatetemplate，redistemplate等。
5. 包装器模式：链接多个数据库，不同客户每次访问不同数据库。
6. 策略模式：自动选择JDK动态代理还是GCLIB。

**spring包括自动装配和手动装配**，手动装配基于Xml、构造方法、setter方法等，**自动装配**：no[不进行自动装配，显式设置ref属性进行装配]，**byName**[通过参数名自动装配，例如autowire设置成byName，容器就会试图匹配相同名字的bean]，**byType**[通过参数类型自动装配，例如autowire设置成byType，容器试图匹配和该bean属性具有相同类型的bean，如果有多个就会报错]，**constructor**构造器参数，**autodetect**[先constructor，再byType]

**事件**：我们可以创建bean用来监听在ApplicationContext中发布的事件，ApplicationEvent类和在ApplicationContext接口中处理的事件，如果一个bean实现了ApplicationListener，当一个ApplicationEvent被发布以后，bean会自动被通知。有上下文更新、开始、停止、关闭事件、请求处理事件、自定义事件。

**spring AOP**：

使用的是代理模式，如果代理的是一个接口的话，就用java动态代理，如果是其他的类，使用CGLIB。

**核心概念**

1. 切面：对横切关注点的抽象
2. 横切关注点：对哪些方法进行拦截，拦截后怎么处理。
3. 连接点：被拦截到的点
4. 切入点
5. 通知：前置，后置，异常，最终，环绕通知
6. 目标对象：代理的目标对象。
7. 织入：将切面应用到目标对象并导致代理对象创建的过程。
8. 引入：不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段。

**spring事务**：

* 事务的特性：ACID(原子性、一致性、隔离性、持久性)

* 并发事务带来的问题：脏读、丢失修改、不可重复读、幻读。

  TransactionDefinition接口定义了五个表示隔离级别的常量。

  **TransactionDefinition.ISOLATION_DEFAULT:**	使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.

  **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**

  **TransactionDefinition.ISOLATION_READ_COMMITTED:** 	允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**

  **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 	对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**

  **TransactionDefinition.ISOLATION_SERIALIZABLE:** 	最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

  **事务传播行为**：

  当事务方法被另一个事务方法调用是，必须指定事务应该如何传播。

  ​    **支持当前事务的情况：**

  - **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
  - **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
  - **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

  ​    **不支持当前事务的情况：**

  - **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
  - **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
  - **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

  **其他情况：**

  - **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

  **@Transaction失效场景**：

  1. 注解加在了非public方法上。
  2. 注解属性设置错误，就是事务传播行为设置了PROPAGATION_NOT_SUPPORTED这类的。
  3. rollbackFor设置错误，抛出异常不用处理到。
  4. 同一个类方法调用，导致失效，例如类有A/B两个方法，B加了注解，A调B，则会事务失效。
  5. 异常被catch掉了，所以不抛出异常不处理事务。
  6. 数据库引擎不支持事务。
  7. 多数据源下没有设置datasource。

  