# javaLearn
迷你版java面试笔试总结，面向有基础的同学，涵盖基础知识、框架、网络及中间件知识点总结，继续更新中。

**部分知识来源JavaGuide等网络学习资源。
在线阅读：https://1471914707.github.io/
下载地址：链接: https://pan.baidu.com/s/1MxKXIZtoBd57pTwTIDyrgA 提取码: 3arb
# java基础

* **内部类**：静态内部类、成员内部类、局部内部类、匿名内部类。
* **重写**(override)要求子类的返回值小于父类（类型相同），修饰符使用范围也大于分类，抛出异常小于父类。
* **静态方法**不能调用类非静态方法，因为静态方法可以在不生成对象的时候直接调用。
* **默认构造方法**，因为子类构造的时候也调用super()父类，所以需要增加一个默认构造函数，避免编译出错。
* **java只有值传递**：按值调用，按引用调用（其实也是按值调用，因为传的是一个指针地址）
* **线程状态**：初始状态->就绪状态->运行状态、阻塞状态、等待状态、超时等待状态->终止状态。
* **try**：不能单独用，可以不加catch，但是最后要finally{}
* **synchronized**:优化加了偏向锁，自旋锁/适应性自旋锁，轻量级锁，锁粗化，锁消除

​       synchronized加在类方法上是锁的对象锁，加在静态方法上锁的是类锁。

* **volatile**:直接从内存中取值，同时禁止指令重排。
* **线程池的好处**：1.减少创建和销毁线程池的开销、2.任务不用等创建线程即可运行，3.管理，调优，监控。

包括newFixedThreadExecutor、ScheduleThreadExecutor、newSingleThreadExecutor，newCacheThreadExecutor。满载后拒绝策略有直接抛异常，抛弃最老的，丢弃，有调用者自己执行。

**创建线程的方式**：1.继承thread、2.实现Runable、3.实现callable

​        executor.submit（）返回执行结果Future（get()返回结果会阻塞线程去执行完成），executor.execute()不返回执行结果

```java
public ThreadPoolExecutor(int corePoolSize,//核心线程数量
                          int maximumPoolSize,//最大线程数量
                          long keepAliveTime, //超过核心线程数量时，多出的线程等待keepAliveTime                                                 时间后自行销毁 
                          TimeUnit unit,//上面时间的单位
                          BlockingQueue<Runnable> workQueue,//等待队列
                          ThreadFactory threadFactory,//创建新线程的时候用到
                          RejectedExecutionHandler handler //拒绝策略
                         ){...}
```

thread拥有的方法：setPriority(.)，sleep()，join（让调取它的线程等待本线程执行完），yield（暂停当前执行线程，执行其他线程），interrupt（中断线程）,isAlive（测试线程是否处于活动状态）

线程安全的理解**：是有全局变量及静态变量引起的，只读没问题，但是写的要考虑线程同步。常量不会改没事，线程内的局部变量或者调用方法前每一个实例都是新建的，那也是线程安全的，因为不会访问共享的资源。

**ReentrantLock 实现原理**：基于AQS来实现。

**Synchronized和Lock的异同**：前者利用Object的notify/wait之类的调度，后者用Condition进行线程间调度。再者前者是可以加在方法/代码片段给JVM执行，后者直接代码段通过代码控制。前者自动释放，后者手动解锁。

**Queue**：实现类LinkedList，ArrayBlockingQueue, LinkedBlockingQueue，PriorityQueue, PriorityBlockingQueue，ConcurrentLinkedQueue。

**Java内存模型（Java Memory Model ,JMM）**就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范。Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。而JMM就作用于工作内存和主存之间数据同步过程。他规定了如何做数据同步以及什么时候做数据同步。

JDK8的应用，可以使用**Instant**代替Date，**LocalDateTime**代替Calendar，**DateTimeFormatter**代替SimpleDateFormat。

# JAVA的JUC、AQS

其实就是指java.util.concurrent/.atomic包下的东西，例如常见的有volatile<特性禁止指令重排(1-开辟空间，2-空间初始化,3-空间指向地址变量)>、cas、concurrentHashMap、ContDownLatch、实现Callable接口创建线程、ReentrantLock同步锁、ReadWriterLock读写锁、线程池

**cas**:compare and swap，比较旧的值，没有改变则更新为新的值。atomicInteger就是利用cas、volatile和native方法

**AQS**:位于java.util.concurrent.locks包下，AQS是一个用来构建锁和同步器的框架，例如ReentrantLock和Semaphore等等，**原理**：被请求的共享资源空闲，那么当前空闲的线程就会被标记为运行状态，资源也被标记为被锁定，需要一套线程阻塞等待以及被唤醒时锁分配的机制，AQS使用CLH队列锁（自旋锁，先来先服务，保证无饥饿）实现，即将暂时获取不到锁的线程放在队列中。



# JVM

* **堆**：对象和数组分配内存，是垃圾收集器管理的主要区域，又称为GC堆，1.7之后运行时常量池也在此

* **方法区/元空间/直接内存**：储存已被虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码数据

  **线程独有**

  * **程序计数器**：保存下条指令、分支，循环，跳转、异常处理，线程恢复的位置，唯一不会OOM
  * **虚拟机栈**：一个个栈帧，保存局部变量，操作数栈，动态链接，方法出入口信息
  * **本地方法栈**：执行native方法

  > 运行时常量池：放class编译信息。
  >
  > 字符串常量池：7之前放方法区，之后放堆。

#### 垃圾回收

基本采用分代垃圾回收算法，粗分为新生代和老年代，细分为eden，from survivor, to survivor,老年区

**堆内存常见分配策略**：对象优先在eden区分配，大对象直接进入老年代，长期存活的对象将进入老年代。

**Minor GC**：新生代垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快。

**FULL GC/Major GC**: 发生在老年代的GC，速度比Minor GC慢10倍以上。

**判断对象死亡**：引用计数法, 可达性分析法（GC ROOT，找出一些root节点然后构造树，需要两次成为不可达对象，才面临回收，GCROOT对象：虚拟机栈中引用的对象、方法区类静态属性引用的对象-方法区常量池引用的对象、本地方法栈JNI引用的对象。一般被标记两次才会真正被回收）

**无用的类**：所有实例被回收，加载该类的classLoader已被回收，对应的java.lang.class没有在任何地方被引用。

**分代收集**：新生代使用复制算法，老年代使用标记-清除或标记整理。

> 新生代收集器：Serial、ParNew、Parallel Scavenge(注重吞吐量)；
>
> 老年代收集器：Serial Old、Parallel Old、CMS；
>
> 整堆收集器：G1；
>
> CMS收集器**：Concurrent mark Sweep（并发标记清除） ,一种以获取最短回收停顿时间为目标的收集器，基本实现垃圾回收和用户线程同时工作。（缺点:对CPU资源敏感,无法处理浮动垃圾，标记清除有碎片）
>
> 1. 初始标记：暂停其他所有线程，记录直接和root相连的对象
> 2. 并发标记：同时开启GC和用户线程，记录可达对象（不能保证包含当前所有可达对象，因为还在更新）
> 3. 重新标记：暂停用户线程，修正并发标记期间产生变动的那一部分标记记录
> 4. 并发清楚：开启用户线程，同时GC线程开始对标记的区域做清扫。
>
> **G1收集器**：Garbage-First 面向服务器的垃圾收集器，主要针对多颗处理器及大容量内存的机器，以高概率满足GC停顿时间的要求，还具备高吞吐量性能特征。使用标记整理、可预测的停顿时间模型。维护一个优先列表，优先回收价值最大的region
>
> 1. 初始标记
> 2. 并发标记
> 3. 最终标记
> 4. 筛选回收

**finalize()**：可达性分析中，第一个标记会进行一次筛选，如果类有覆盖这个方法，会放到一个F-Queue中，虚拟机触发一个线程去执行，但不会承诺一直等待它运行完避免死锁从而内存回收系统崩溃，GC对处于F-Queue中的对象进行第二次被标记，这时该对象将会被移除“即将回收”集合，等待回收。

**内存泄露**：对象没有被使用了但是因为某些原因不能被回收。例如：栈中push进去的元素一直没有拿出来使用；HashMap的key改了hascode导致之前的找不回来。

Full GC本身不会先进行Minor GC，我们可以配置-XX:+ScavengeBeforeFullGC（非CMS回收算法）、CMSScavengeBeforeRemark（CMS回收算法）可以，**让Full GC之前先进行一次Minor GC**，因为老年代很多对象都会引用到新生代的对象，先进行一次Minor GC可以提高老年代GC的速度。


#### 对象创建过程 

1. **类加载检查**：虚拟机遇到一个new指令的时候，首先去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过，没有的话，必须执行相应的类加载过程。

2. **分配内存**：类加载过就可以确定所需内存大小，为对象分配内存就是在java堆中划分出来一块确定带下的内存。方式有“指针碰撞"和”空闲列表“，选择哪个方式取决于JAVA堆是否规整，而java堆是否规整又由采用的垃圾收集器是否带有压缩整理功能决定。涉及线程安全（采用CAS+失败重试，TLAB:每个线程预先分配内存，则首先在TLAB分配，大于剩余内存或快用完，就用上面的CAS）

> 指针碰撞：内存规整下，通过一个指针区分已使用和未使用的内存的位置，需要使用时，指针移动划分。
>
> 空闲列表：内存不规整，虚拟机维护一空闲列表，需要多大内存空闲列表又显示有就拿去用，如cms收集器

2. **初始化零值**：内存空间初始化，类字段初始值
3. **设置对象头**:类的元数据信息，对象的哈希码，GC分代年龄
4. **执行init方法**

**对象访问定位方式**

1.使用句柄：堆中划分一块内存作为句柄，引用（变量名）储存的是对象的句柄地址，句柄包含对象实例数据地址指针和类型数据指针地址。对象移动只需要改变句柄的实例数据指针。

2.直接指针：引用储存的时候直接对象的地址，直接对象中包含到对象类型数据的指针和对象实例数据。速度快。

**PermGen space异常**：永久代溢出，一般是加载的class太多超出默认的4m，-XX:PermSize=64M-XX:MaxPermSize=128m，但是在java8之后，这个是元空间（直接内存）了。

**-XX:MaxNewSize**：新生代内存大小，与-Xmn一样，Sun建议占总的JVM内存3/8。

**-XX：NewRatio=4**：年轻代与老年代比例1：4。

**-XX：SurvivorRatio=4**：年轻代的eden和一个survior区的比例1 ： 4。和两个survior比就是2:4。

**JVM垃圾回收器参数**：

| 参数               | 描述                                                       |
| ------------------ | ---------------------------------------------------------- |
| UseSerialGC        | 运行在client模式下的默认值，Serial + Serial Old            |
| UseParNewGC        | parNew + Serial Old                                                          |
| UseConcMarkSweepGC | ParNew + CMS + Serial old（此为CMS出现失败后的后备收集器） |
| UseParallelGC      | 运行在Server模式下，Parallel Scavenge + Serial Old         |
| UseParallelOldGC   | Parallel Scavenge + Parallel Old，jdk8默认                 |



### CLASS文件

.class文件包含以下：

![image-20200309174450475](C:\Users\lin\AppData\Roaming\Typora\typora-user-images\image-20200309174450475.png)

* **魔数**：确定这个class文件是否能被虚拟机接收。
* **class文件版本**: class文件的版本号，保证编译正常执行。

## 类加载过程

类加载过程： **加载**（文件二进制流到可使用的数据结构）->**链接**->**初始化**（init方法）。连接过程分为：**验证**（检查class文件的正确性）->**准备**（初始化零值）->**解析**（符号引用【因为一开始不知道引用的是什么，所以会使用特定符号来表示】转直接引用）

1. **加载** ：1.通过类名获取定义此类的二进制文字流（还可自定义类加载器）。2.字节流代表的静态储存结构转换为方法区的运行时数据结构。3.在内存中生成一个代表该类的Class对象，作为方法区这些数据的访问入口。

*数组类型不通过类加载器创建，它由Java虚拟机直接创建。

>***类加载器**：JVM内置了三个重要的ClassLoader，除了BootstrapClassLoader，其他都是由Java实现自java.lang.classloader。
>
>* **BootstrapClassLoader（启动类加载器）**: 最顶层的加载类，有C++实现，负责加载%JAVA_HOME%/lib目录下的jar包和类或者被-Xbootclasspath参数指定的路径的所有类。
>* **ExtensionClassLoader（拓展类加载器）**: 主要负责加载目录%JER_HOME%/lib/ext目录下的jar包和类，或被java.ext.dirs系统变量所指定的路径下的jar包。
>* **AppClassLoader（应用程序类加载器）**: 面向我们用户的加载器，负责加载当前应用classpath下所有的jar包和类。 

> ***双亲委派模型**（并非父母两个，而是指父母这一辈的）: 每一个类都有一个对应它的类加载器，系统中的ClassLoader在协同工作的时候会默认使用双亲委派模型，即在类加载的时候，系统会首先判断当前类是否被加载过，已经被加载过类会直接返回，否则才会被尝试加载，加载的时候，首先会把该请求委派该父类加载器的loadClass()处理，因此所有的请求最终都应该传到最顶层的启动类加载器bootstrapClassLoader中。当父类加载器无法处理时，才由自己来处理，当父类加载器为null时，会启动类加载器BootstrapClassLoader作为父类加载器。这中的父子是按优先级定的，不是继承关系。
>
> 好处：保证Java程序的稳定运行，可以避免类的重复加载（JVM区分不同类的方式不仅仅根据类名，系统的类文件被不同的类加载器加载产生的是两个不同的类），保证了Java的核心API不被篡改，如果没有使用双亲委派模型，而是每个类加载器都加载自己的话就会出现一些问题，比较我们编写一个称为java.lang.Object类的话，那么程序运行的时候，系统就会出现不同的Object类。反正意思就是向上寻找是否已经加载过了。
>
> bootstrapClassLoader必用因为它在虚拟机实现，其他不想用的话自己定义一个类加载器，重载loadClass()即可，就是继承ClassLoader

**隐式加载**指的是程序在使用new等方式创建对象时，会隐式地调用类的加载器把对应的类加载到JVM中。**显式加载**指的是通过直接调用class.forName()方法把所需的类加载到JVM中。

# mybatis

#{}是静态替换变量有注入风险，${}是利用PreparedStatement的？替换，相对于加''，防止注入。

select|update|delete|insert|resultMap|parameterMap|sql|include|selectKey|

动态标签trim|where|set|foreach|if|choose|when|otherwise|bind

> bind的作用是拼接字符串等操作，可以应对改数据库，sql注入
>
> <!-- and user_name like concat('%',#{userName},'%') -->               
>
> <bind name="userNameLike" value=" '%' + userName + '%' "/>
>
> and user_name like #{userNameLike}

不同的xml写一样的namespace和id，可以但会覆盖，因为用的map<namespace+id，object>，没有写namespace的话相同id则会报错。

**mybatis的executor执行器**：（严格限制在SqlSession生命周期范围）

1. **SimpleExecutor**：执行一次update或select，就开启一个Statement对象，用完立即关闭。
2. **ReuseExecutor**：执行update或select,以sql作为key查找Statement对象，存在就使用，不存在就创建，使用完不关闭，而是放在Map<String, Statement>重复利用。
3. **BatchExecutor**：执行update（jdbc批处理不支持select），将所有sql添加到批处理中(addBatch)，等待统一执行（executeBatch），它缓存多个Statement对象，先addBatch，再executeBatch

使用**#{}**经过预编译，是安全的，防止Sql注入。

拦截器只能拦截四种类型的接口：Executor、StatementHander、ParameterHandler、ReusltSetHandler。想要支持其他接口得重写configuration，可以对这四个接口所有方法进行拦截。



### 缓存

* mybatis默认开启一级缓存，一级缓存是查询数据库前的最后一层缓存，使用PerpetualCache（即HashMap缓存），利用自动生成的cacheKey查询出数据，update/delete/insert提交回滚事务时会删除缓存。SqlSession级别的缓存。关闭方式有配置文件，插件，<insert ...flushCache="true">。
* 二级缓存比较复杂，涉及事务脏读，需要在**-mapping.xml配置<cache />来开启。mapper级别的缓存。两个Mapper的namespace如果相同，那么这两个Mapper执行的sql查询会被缓存在同一个二级缓存中。要开启二级缓存需要在配置文件中设置cacheEnabled属性为true。

# Spring

springbean生命周期：实例化、属性赋值、初始化、使用、销毁。

spring是一些模块的集合，包括核心容器（DI，验证，类型转换，资源，国际化，数据绑定，事件），数据访问/集成，Web，AOP，工具，消息，测试模块。

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

**spring包括自动装配和手动装配**，手动装配基于Xml、构造方法、setter方法等，**自动装配**：no[不进行自动装配，显式设置ref属性进行装配]，**byName**[通过参数名自动装配，例如autowire设置成byName，容器就会试图匹配相同名字的bean]，**byType**[通过参数类型自动装配，例如autowire设置成byType，容器试图匹配和该bean属性具有相同类型的bean，如果有多个就会报错]，**constructor**构造器参数，**autodetect**[先constructor，再byType]

**事件**：我们可以创建bean用来监听在ApplicationContext中发布的事件，ApplicationEvent类和在ApplicationContext接口中处理的事件，如果一个bean实现了ApplicationListener，当一个ApplicationEvent被发布以后，bean会自动被通知。有上下文更新、开始、停止、关闭事件、请求处理事件、自定义事件。

### spring AOP

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

### spring事务

* 事务的特性：ACID(原子性、一致性、隔离性、持久性)

* 并发事务带来的问题：脏读、丢失修改、不可重复读、幻读。

  ###### TransactionDefinition接口定义了五个表示隔离级别的常量

  **TransactionDefinition.ISOLATION_DEFAULT:**	使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.

  **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**

  **TransactionDefinition.ISOLATION_READ_COMMITTED:** 	允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**

  **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 	对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**

  **TransactionDefinition.ISOLATION_SERIALIZABLE:** 	最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

  ### 事务传播行为

  当事务方法被另一个事务方法调用是，必须指定事务应该如何传播。

  **支持当前事务的情况：**

  - **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
  - **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
  - **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

  **不支持当前事务的情况：**

  - **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
  - **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
  - **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

  **其他情况：**

  - **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

  **@Transaction失效场景**：

  1. 注解加在了非public方法上。
  2. 注解属性设置错误，就是事务传播行为设置了PROPAGATION_NOT_SUPPORTED这类的。
  3. 
  4. rollbackFor设置错误，抛出异常不用处理到。
  5. 同一个类方法调用，导致失效，例如类有A/B两个方法，B加了注解，A调B，则会事务失效。
  6. 异常被catch掉了，所以不抛出异常不处理事务。
  7. 数据库引擎不支持事务。

  **分布式事务**：

  1. 两阶段提交2PC：在计算机网络以及数据库领域内，为了使基于分布式系统架构下的所有节点在进行事务提交时保持一致性而设计的一种算法。当一个事务跨越多个节点时，为了保持事务的ACID特性，需要引入一个作为协调者的组件来统一掌握所有节点，并最终指示这些节点是否要把操作结果进行真正的提交。概括为：参与者将操作成败通知协调者，再由协调者根据所有参与者的反馈情况决定各参与者是否提交操作还是中止操作。准备阶段（各系统准备好但不提交）-统一提交（失败还向每个参与者发送回滚）释放锁。问题会有阻塞、脑裂（网络问题协调者通知不到位，最后数据部分不一致）、宕机后再恢复不知道之前commit之类通知的情况。
  2. 三阶段提交3PC：加入超时机制，CanCommit(协调者向参与者发送commit请求，参与可以就返回YES/NO)--PreCommit(有一个NO则中断，全部YES就会下一步)--doCommit(真正的事务提交，协调者发送提交请求，参与者提交事务，参与者提交完成发送ack响应，协调者确定完成事务)。

  **CAP**：一致性（分布式系统中的所有数据备份在同一时刻是否同样的值）、可用性（集群某一部分节点故障后整体是否还能响应客户端读写请求）、分区容错性（分区相当于对通信时限要求，系统如果不能在时限内达成数据一致性，意味着发生了分区情况，必须就当前操作在C和A之间做出选择）。

  **BASE**：基本可用（Basically Available），柔性状态（Soft State）、最终一致性。

  **柔性事务**：两阶段型、补偿型（TCC）、异步确保型（最终消息一致性）、最大努力通知型。

  

# springboot

* 相对于spring更加纯注解开发，减少配置和大量样板代码。
* 与spring生态系统的天然结合。
* 可以使用内置工具(如Maven和Gradle)开发管理jar和测试Spring Boot应用程序。
* 提供内嵌的HTTP服务器。
* ~~提供命令行接口工具~~。

### 1. 读取配置文件信息

* **@Value("${property}")**  不被推荐
* **@ConfigurationProperties(prefix = "library")** 与bean绑定，改bean注解@Component
* **@EnableConfigurationProperties(ProfileProperties.class)** 用于启动类上， 配置类用@ConfigurationProperties("my-profile")，这个可以加@notnull之类的校验
* **@PropertySource("classpath:website.properties")** 读取指定文件，加@Component

### 2.全局异常

* @ControllerAdvice(assignableTypes = {ExceptionController.class})用于类、@ExceptionHandler(value = Exception.class)用于方法。
* @ResponseStatus(code = HttpStatus.NOT_FOUND) 返回特定的码，用于异常类。
* 方法里throw new ResponseStatusException，没效果。

### 过滤器

* @Configuration类下生成FilterRegistrationBean<MyFilter>的@Bean

```java
@Bean
    public FilterRegistrationBean<MyFilter> setUpMyFilter() {
        FilterRegistrationBean<MyFilter> filterRegistrationBean = new                FilterRegistrationBean<>();
        filterRegistrationBean.setOrder(2);
        filterRegistrationBean.setFilter(myFilter);
        filterRegistrationBean.setUrlPatterns(new ArrayList<>(Arrays.asList("/api/*")));

        return filterRegistrationBean;
    }
```

多个过滤器按指定顺序执行的话需要用上面的方法。

* @WebFilter(filterName = "MyFilterWithAnnotation", urlPatterns = "/api/*")，启动类要加@ServletComponentScan

### 参数校验

springboot引进来的基础包上已经包含所要用的校验注解，其中在方法上使用在类的，加@Valid，单独参数的@Valid @NotNull之类（此时controller需要加注解@Validated）。值得一提的是，因为之前有同事说处理参数校验出错的时候，不用在代理里处理bindResult，其实这种说法并不对，因为不单独处理的话时候会抛出异常MethodArgumentNotValidException，如果使用全局异常处理的话，就可以集中处理这类问题。

除了contoller之外，service这些也可以使用

```java
@Service
@Validated
public class PersonService {

    public void validatePerson(@Valid Person person){
        // do something
    }
}
```

* 自定义校验，实现的一个注解@interface,再实现一个具体类实现ConstraintValidator接口。

* 校验组，例@NotNull(groups = DeletePersonGroup.class)，DeletePersonGroup是接口，在方法上使用@Validated(AddPersonGroup.class)即可生效。



# 设计模式

**SOLID模式**：

1. 单一原则：对象单一功能
2. 开闭原则：对拓展开放，对修改关闭
3. 里氏替换：程序中的对象应该是可以在不改变程序正确性的前提下被它的子类所替换的
4. 接口隔离：多个特定客户端接口要好于一个宽泛用途的接口
5. 依赖倒转：代码应当取决于抽象概念，而不是具体实现

**工厂模式**：主要功能都是帮助我们把对象的实例化部分抽取出来，降低耦合度，增强拓展性。

**简单工厂模式**：实现对象的创建和对象使用分离，将对象的创建交给专门的工厂类负责，缺点在于工厂类不够灵活，增加新的具体产品需要修改成功类的判断逻辑，多的时候就非常复杂。例如传进来一个字符串，case判断得到一个所需的指定对象。

**工厂方法模式**：通过定义一个抽象的核心工厂类，并定义创建产品对象的接口。创建具体产品实例工作延迟到其工厂子类去完成。增加一个产品的时候，只需要增加一个具体实现的工厂子类。简单来说，客户端需要什么样的产品可以调子类方法创建对应的产品。

**抽象工厂模式**：处理一个工厂产生不同的产品，定义一个抽象类包括不同产品的创建。

单例模式、适配器模式、观察者模式、模板方法模式、代理模式、策略模式、外观模式、建造者模式、原型模式、修饰模式、状态模式、备忘录模式、组合模式、迭代器模式、桥接模式、命令模式、职责链模式、中介者模式、享元模式、解释器模式、访问者模式。



# 数据库

**优化方向**：数据库表设计，良好的SQL、分库分表，读写分离、缓存、搜索引擎、硬件升级、索引使用、系统配置

**SQL调优**：

1. 命中查询缓存
2. explain使用索引
3. limit 1
4. 搜索字段走索引
5. join字段走索引（属性相同，字符集系统）
6. 避免使用order by rand()
7. 避免select *，消耗网络传输、需要查数据字典、走不了索引字段需要回表，客户端映射字段多有消耗。

第一范式是字段最小不可拆解，第二范式是字段和主键关联依赖，第三范式是字段和主键无间接关联（传递依赖）无冗余。

**索引类型**：普通索引、唯一索引、主键索引、组合索引。

**主从复制**：主机更新记录到二进制日志binlog中，从服务器从主服务器中不断读取binlog日志到自己的中继日志（replay log），从服务器重做中继日志的时间，把更新应用到自己的数据库上。

MVCC 一词代表的是多版本并发控制，只在READ COMMITED和REPEATABLE READ两个隔离级别下工作。

**mysql存储引擎**

1. **MyISAM**：性能极佳，全文索引，压缩。但不支持事务和行级锁，只有表级锁，查询较快，崩溃无法恢复，不支持外键，不支持MVCC
2. **InnoDB**：支持外键，事务/回滚，行级锁，崩溃可恢复，支持MVCC

**mysql索引** ：有Hash索引（单个查询）、BTree索引，全文索引（full text)。

1. **MyISAM**：B+Tree树叶节点的data域就是数据记录的地址，在索引检索的时候，先走B+Tree搜索索引，如果指定KEY存在，就取出data域的值，然后更新域值存的地址读取相应的数据记录。非聚簇索引
2. **InnoDB**：其数据文件本身就是索引文件，相比MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按B+Tree组织的一个索引结构，树的叶节点data域保存完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引，这称为聚簇索引。其他索引是辅助索引，叶节点为主键key和索引字段数据。

**数据库事务**

ACID：原子性、一致性、隔离性、持久性

> 导致问题，1.脏读（读到别的事务尚未提交的数据），2.丢失修改（当一个事务修改是，另一个事务也在修改覆盖，导致前面事务修改丢失），3.不可重复读（两次读的数据不一致）,4.幻读（读着的数据行数变化）
>
> **SQL标准定义四个隔离级别**：
>
> 1. READ-UNCOMMITTED（读取未提交）,允许读取尚未提交的数据。导致脏读，幻读，不可重复读
> 2. READ-COMMITTED（读取已提交）,允许读取并发事务已经提交的数据，还会幻读，不可重复读
> 3. REPEATABLE-READ（可重复读）,对同一个字段多次读取结果完全一致除非自己改，还会幻读
> 4. SERIALIZABLE（可串行化），最高隔离级别，完全ACID隔离，一个个事务依次执行
>
> **mysql默认的是可重复读**，使用next-key lock锁算法避免幻读。
>
> InnoDB存储引擎锁算法：record lock（行锁）、Gap lock（间隙锁，锁一个范围，不包括记录本身），Next-key lock（record+gap锁定一个范围，包括记录本身）



# redis

**应用场景**：

> list队列做一个FIFO双向队列，实现一个轻量级高性能消息队列服务。
>
> 用它的Set做高性能的tag系统。
>
> 会话缓存。
>
> 全页缓存。
>
> 排行榜/计数器。
>
> 发布/订阅。

使用redis的好处是，一处缓存多处使用，同时有丰富的数据类型。redis是单线程的，但是IO多路复用。

数据类型：String、List、Hash、Set、Sorted Set、bitmap位图，geo地理位置，hyperloglog等

过期删除：定期扫描随机删除，惰性删除查的时候检查是否要删。

提供了一些内存淘汰机制，一般用allkey-lru（内存不足时，删除最近最少使用的key）

> 1. noeviction:默认策略，不淘汰，如果内存已满，添加数据是报错。
> 2. allkeys-lru:在所有键中，选取最近最少使用的数据抛弃。类似LinkedHashMap。
> 3. allkeys-random: 在所有键中，随机抛弃。
> 4. volatile-lru:在设置了过期时间的所有键中，选取最近最少使用的数据抛弃。
> 5. volatile-random: 在设置了过期时间的所有键，随机抛弃。
> 6. volatile-ttl:在设置了过期时间的所有键，抛弃存活时间最短的数据。

持久化机制：RDB、AOF(数据有改就写、每秒写一次、让操作系统决定)

**缓存雪崩**：本身就要随机地设置过期时间。尽量保证redis集群高可用性，发现机器宕机尽快补上，选择合适的内存淘汰策略。事中本地ehcache缓存+hystrix限流&降级，避免Mysql崩掉。事后利用redis持久化机制恢复缓存。

还可以加锁或者队列来控制读数据库写缓存的线程数量，或者做二级缓存，A1缓存没有查A2（设置长期）。

**缓存击穿**：一个key失效了被大量访问：永不过期+异步刷新，互斥锁只能有一个线程更新缓存。

**缓存穿透**：null也缓存时间短些，布隆过滤器。

**缓存一致**：肯定不能读写串行化，一般是先更新数据库，再删除缓存（更新复杂，尤其涉及多表。写频繁就改频繁，再者也是赌你不会那么快读数据），但是在更新数据库的过程中，缓存的数据还是旧的。面对高并发之下的不一致（先删缓存再更数据），更新数据的时候，根据数据的唯一标识，将操作路由之后，发送到一个jvm内部队列中。读取数据的时候，如果发现数据不在缓存中，那么将重新执行"读取数据+更新缓存"的操作，根据唯一标识路由之后，也发送到同一个jvm内部队列中。一个队列对应一个工作线程，每个工作线程串行拿到对应的操作，然后一条一条的执行（一般就是会先执行更新数据库操作），这样的话，一个数据变更的操作，先删缓存，然后再去更新数据库，但是还没完成更新，如果一个读请求过来，没读到缓存，那么可以先将缓存更新的请求发送到队列中，此时会在队列中积压，然后同步等待缓存更新完成，最好不重复放到队列中。

**redis分布式锁**：

> 1. **加锁**：lua脚本，加锁的key，传入设置默认生存时间30s和加锁的客户端id，存在的时候不加锁。一般用hset key uid:1，用的hash结果，存的是1应该表示第一次加锁。
> 2. **锁互斥**：第一步判断key是否存在，已经存在的话，判断uid是否和自己的一样，不一样证明不是（一样的话就可以重入），同时返回锁剩余的生存时间，以此来决定是否需要while循环不停地尝试加锁。
> 3. **watch dog自动延期机制**：场景是30s时间到了还想拥有锁，加锁成功的时候，就会启动一个看门狗，它是一个后台线程，每隔10s检查一下，如果还持有锁，那么就会不断延长锁的生存时间。
> 4. **释放锁**：重入锁次数减一，为0了就del。
> 5. **缺点**：主节点宕机的时候，可能导致多个客户端同时完成加锁。

**一亿个key，查出10w个以某前缀开头的**：keys命令不可以阻塞了，用scan无阻塞但数据有一定的重复。

**实现延迟队列**：用sorted set，时间戳做score，消费者获取指定时间的时间戳的数据。



# 网络问题

**浏览器输入URL发生什么**：

1. 查询本机或本浏览器或路由器的域名缓存，查出域名对应的IP和端口，如果没有则请求DNS服务器。
2. 根据得到的IP和端口发起HTTP请求，达到指定的服务器。
3. 服务器根据请求的内容， 做出处理，然后响应返回到浏览器。
4. 浏览器根据返回的内容或HTML渲染显示页面。

**网络分层**:

1. **应用层**：通过应用进程间交互来完成特定网络应用。DNS服务、HTTP服务等等，交互的数据单元称为报文。

2. **运输层**：负责向两台主机进程之间的通信提供通用的数据传输服务。

   > 传输控制协议TCP--提供面向连接、可靠的数据传输服务。保证可靠传输如下：
   >
   > >* 应用数据被分割成TCP认为最适合发送的数据块。
   > >* TCP给发送的每个包进行编号，接收方对数据包顺序排序，把有序数据传到应用层。
   > >* 校验和：TCP将保持它首部和数据的校验和。这是端到端的校验和，目的是检测数据在传输过程中的任何变化，有差错则丢弃这个报文段和不确认收到此报文段。
   > >* TCP的接收端会丢弃重复的数据。
   > >* 流量控制：TCP链接的每一方都有固定大小的缓冲空间，只接受能接纳的数据，过量则会提示发送方降低发送的速率，防止包丢失。
   > >* 拥塞控制：当网络拥塞时，减少数据的发送。
   > >* ARQ协议：每发完一个分组就停止发送，等待对方确认，收到确认后再发下一个分组。
   > >* 超时重传：启动一个定时器，等待目的端确认收到这个报文段，不能及时就重发。
   >
   > 用户数据协议UDP--提供无连接、尽最大努力的数据传输服务。（不保证数据传输的可靠性）

3. **网络层**：两个计算机之间经过很多数据链路，也可能通过很多通信子网，就是选择合适的网间路由，确保数据及时发送。把运输层报文段或用户数据报封装成分组和包，使用IP协议，因此分组也叫IP数据报。

4. **数据链路层**：链路层，两台主机的数据传输，总是在一段一段的链路上传送的，将IP数据报组装成帧，其为了数据正确传输，包含了数据验证和纠错。

5. **物理层**：数据单位是比特，实现相邻计算机节点之间比特流的透明传送，尽可能屏蔽具体传输介质和物理设备的差异。

**三次握手**：A-(SYN)-B，B-(SYN/ACK)-A，A-(ACK)-B。目的建立可靠的通信信道，确认双方发送接收正常。TCP用

**四次挥手**：A-(FIN)-B，B-(ACK)-A，B-(FIN)-A，A-(ACK)-B

**HTTP长连接,短连接**：http1.0短连接，每一个请求都建立一次链接。http1.1长连接保持连接性，请求头加上Connection:keep-alive，但是需要服务器也支持长连接。http1.1还增加了一些错误状态效应码，缓存处理更多，允许请求资源的某个部分。

**URI**：统一资源标志符，就像身份证号码

**URL**：统一资源定位符，提供资源的路径，就像个人地址。

**HTTP和HTTPS的区别** : 1.端口，http用80，https用443。2.http运行在tcp之上，传输内容是明文，客户端和服务器端无法验证对方的身份。https是运行在SSL/TLS之上的HTTP协议，SSL/TLS运行在TCP之上，所有的内容都经过加密，但是HTTPS消耗资源给HTTP耗费更多。



# 认证授权

**认证**：你是谁，验证你的身份凭据，系统你知道你这个用户存在。

**授权**：发生在认证之后，访问系统的权限。

**Spring Session**：一般与redis使用，可以在不同的项目中共享session。支持一个域名，同一个项目部署多台tomcat，也支持一个域名多个项目，支持同一根域名多个子域名，不支持不同根域名实现session共享。

> 原理：实现了一个filter，继承自OncePerRequestFilter（确保一次请求只通过一次filter），并且@Order优先级被优先执行

**CSRF**：跨站请求伪造，攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求，利用受害者已经获取的注册凭证，绕过后台的用户验证，达到冒充用户被攻击的网站执行某项操作的目的。

​         解决办法：同源策略，token（麻烦在某个请求都要验证）

**OAuth2.0**：主要用来授权第三方应用获取有限的权限。实际上它就是一种授权机制，它的最终目的是为第三方应用颁发一个有时效性的令牌token，使得第三方应用能够通过该令牌获取相关的资源，也常用于第三方登录，当你的网站接入了第三方登录的时候一般就是使用的OAuth2.0协议。

**SSO**：单点登录，一个平台登录，可以自动登入其他关联平台。解决公司多平台访问。

# Nginx

六种负载均衡方式：（作用也是避免请求过了集中到一台机器）

> 1. 轮询
> 2. weight权重方式
> 3. ip_hash根据ip分配方式，可以让同一个用户访问同一个资源。
> 4. least_conn最少链接方式
> 5. fair响应时间方式，请求向响应时间较短的机器发。
> 6. url_hash依据URL分配方式，根据url请求到同一个服务器，避免缓存资源多次请求。

**配置相关**

1. event标签里面，有**worker_processer**（表示连接处理请求的线程个数，一般和cpu核数保持一致），**worker_connections**（每一个worker最大连接数，一般和ulimit -a中的openfile参数[这个参数表示操作系统最大打开文件数量，最好改大些，我见过netty实战中直接把它改成100万了]除以worker_processer数）。
2. **keepalive_timeout、send timeout **请求和响应的超时时间。
3. **default_type  application/octet-stream;**将一些路径下的例如文件通知到浏览器是文件流。
4. **client_max_body_size 10m;**上传文件大小限制。
5. 转发配置


	server{
		  listen 80;
		  charset utf-8;
		  server_name www.ican.com;
		  access_log logs/host.access.log;
		  location /{
			  proxy_pass http://127.0.0.1:7001;
		  }
	}

# linux

**磁盘满了怎么办**：一般是文件过大造成的，可以先用**df -h**查看一下是哪个地方占用较大，其次**du -h --max-depth=1**查询具体文件夹占用大小，以此类推，查出实际占用大的文件删除。**ls -lhS**文件大小排序查看。

**查看web应用的jvm信息**：1. jps查询java应用基础信息，jinfo -flags PID。2. jmap -heap PID查看分代情况， jmap -histo:live <pid>可迫使JVM进行一次full gc。3. jstat -gcutil pid:查看每个代区域使用的百分比情况。



# 其他

**OOM怎么排查**：如果JVM配置了-XX:+HeapDumpOnOutOfMemoryError，可以拿到发生OOM的时候，堆里面都有些什么。

**一致性Hash**：简单来说，就是一些节点发布在一个圆上，按顺时针找到第一个节点作为处理节点，会涉及增加节点和减少节点。一开始节点太少的时候，可以通过虚拟节点（即同一个节点A，虚拟节点A1,A2..但都是A处理）。

**分布式事务**：

> 1. 两阶段提交方案/XA方案。意思就是各个数据库都准备好一个有错了全部回滚，但是一般一个服务对应一个库，所以操作上比较难控。（spring+JTA）
> 2. TCC方案（Try Confirm Cancel）：Try阶段对各个服务的资源做检测以及对资源进行锁定或预留。Confirm阶段是在各个服务中执行实际的操作。Cancel如果任何一个服务的业务方法执行出错，那么这里就需要进行补偿，就是执行已经执行成功的业务逻辑回滚。实际上是写代码回滚动和补偿。
> 3. 本地消息表：依赖数据库的消息表来管理事务.A、B事务通知，失败又通知..............复杂到不想说
> 4. 可靠消息最终一致性方案：本地消息表改成用MQ，A系统发一个准备消息到mq(发送失败就直接取消操作不执行了)，发送成功后，执行本地事务，如果成功就告诉mq发送确认消息，如果失败就告诉mq回滚动消息，如果发送了确认消息，此时B系统会接收到确认消息，然后执行本地的事务，mq会自动定时论询所有准备消息回调你的接口，问你这个消息是不是本地事务处理失败了，所有没发送确认的消息，是继续重试还是回滚？一般来说这里你可以查下数据库看之前本地事务是否执行，如果回滚了，那么也回滚，这个就是避免本地执行成功了而确认消息却发送失败。B失败就回滚，通知A或直接抛错通知人员。
> 5. 最大努力通知：A本地事务执行完，发送个消息到MQ，这里会有个专门消费MQ的最大努力通知服务，这个服务会消费MQ然后写入到数据库记录下来，或者放到个内存队列也可以，接着调用B的接口，B成功就ok，失败了那么最大努力通知服务就定时尝试重新调用系统B，反复N次，最后还是不行就放弃。



# lombok

[原文路径](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485385&idx=2&sn=a7c3fb4485ffd8c019e5541e9b1580cd&chksm=cea24802f9d5c1144eee0da52cfc0cc5e8ee3590990de3bb642df4d4b2a8cd07f12dd54947b9&token=1381785723&lang=zh_CN#rd)

* **val**: 用在局部变量上，有点像js的val。
* **@cleanup**: 释放资源

```java
    try {
        @Cleanup InputStream inputStream = new FileInputStream(args[0]);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
```

* **@NonNull**: 用于方法参数上，遇到空的值会抛出空指针。
* **@ToString**: 自动覆盖toString方法，例如@ToString(exclude=”id”)排除id属性，或者@ToString(callSuper=true, includeFieldNames=true)调用父类的toString方法，包含所有属性
* **@EqualsAndHashCode**： 用在类上，自动生成equals方法和hashCode方法**
* **@NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor**：用在类上，自动生成无参构造和使用所有参数的构造函数以及把所有@NonNull属性作为参数的构造函数，如果指定staticName = “of”参数，同时还会生成一个返回类对象的静态工厂方法，比使用构造函数方便很多
* **@Data**： 注解在类上，相当于同时使用了`@ToString`、`@EqualsAndHashCode`、`@Getter`、`@Setter`和`@RequiredArgsConstrutor`这些注解
* **@Value**：用在类上，是@Data的不可变形式，相当于为属性添加final声明，只提供getter方法，而不提供setter方法
* **@Bulider**: 用在类上，链式生成类
* **@Log**: @CommonsLog、@Log、@Log4j、@Log4j2、@Slf4j、@XSlf4j
* **@SneakyThrows**：自动抛受检异常，而无需显式在方法上使用throws语句
* **@Synchronized**：用在方法上，将方法声明为同步的，并自动加锁，而锁对象是一个私有的属性$lock或$LOCK
* **@Getter(lazy=true)**：可以替代经典的Double Check Lock样板代码


# 消息队列

比较核心的场景是：**解耦**，**异步**，**削峰**

> 场景：给第三方发送商品，走mq，避免直接rpc请求超时；保证业务逻辑无异常，出异常就异常池，人介入
>
> 买基金有待确认，敏感数据处理耗时复杂，大宗交易涉及审核，异步处理。
>
> 同步binlog数据，新增-修改-删除是按顺序的，不能就弄成了删除-修改-新增等等。

缺点：

1. 系统可用性降低：引入的外部依赖越多，越容易挂掉，万一MQ挂了怎么办，牵连系统都崩溃。

2. 系统复杂度提高：消息重复消费，消息丢失，消息传递顺序性。
3. 一致性问题：A系统成功了，BCD系统有失败的，数据就不一致了。

| 特性       | ActiveMQ               | RabbitMQ                                         | RocketMQ                                | Kafka                                                        |
| ---------- | ---------------------- | ------------------------------------------------ | --------------------------------------- | ------------------------------------------------------------ |
| 单机吞吐量 | 万级                   | 万级                                             | 10万级，高吞吐                          | 10万级，高吞吐，配置大数据类系统进行实时数据计算，日志采集   |
| topic数量  |                        |                                                  | 几百/几千，吞吐量有较小幅度下降（优势） | 几十/几百，吞吐量大幅下降，所以不宜过多，否则要增加机器资源  |
| 时效       | ms级                   | 微秒级，延迟最低                                 | ms级                                    | ms级以内                                                     |
| 可用性     | 基于主从架构时效高可用 | 基于主从架构实现高可用                           | 非常高，分布式架构                      | 非常高，分布式，一个数据多个副本，不会丢失数据，不会不可用   |
| 消息可靠性 | 较低概率丢失           | 基本不丢                                         | 参数优化配置，0丢失                     | 参数优化配置，0丢失                                          |
| 功能支持   | MQ领域的功能极其完备   | 基于erlang开发，并发能力很强，性能极好，延时很低 | MQ功能比较完善，分布式，拓展性好        | 只支持简单的MQ功能，在大数据领域实时计算和采集日志大规模使用 |

**RabbitMQ的exchange类型**：direct、fanout、topic、headers。

**RabbitMQ的高可用**：是基于主从（非分布式）做高可用。三种模式如下

1. 单机模式
2. 普通集群模式（无高可用性）：在多台机器启动多个RabbitMQ实例，每个机器启动一个，创建的queue，只会放在一个RabbitMQ实例上，但是每个实例都同步queue的元数据（元数据可以认为是queue的一些配置信息，通过元数据，可以找到queue所在实例），消费的时候，实际链接到了另一个实例，那个实例会从queue所在实例上拉取数据过来。缺点一来数据传输频繁，二来queue所在节点挂掉就找不回了（可开持久化）。
3. 镜像集群模式（高可用性）：创建的queue，无论元数据还是queue里的消息都会存在于多个实例中，就是每个RabbitMQ节点都有这个queue一个完整镜像，包含queue的全部数据，写消息时，自动同步到多个实例queue上。缺点是同步开销大，数据大到装不下就麻烦了。

**Kafka的高可用**：message包括：消息长度、版本号、CRC校验码、具体消息。

Kafka基本架构：由多个broker组成，每个broker是一个节点，你创建的topic，这个topic可以划分为多个partition，每个partition可以存在于不同的broker上，每个partition就放一部分数据。

这就是天然的分布式消息队列，就是说一个topic的数据，是分散放在多个机器上的，每个机器就放一部分数据。

实际上Kafka的高可用，就是在于给这些broker创建在不同机器上的副本，只有一个leader，leader负责把数据同步到所有的follower，读写数据都只在leader。写数据时需要leader和follower全部成功，消费只有leader，但是也需要所有follower同步成功返回ack，这个数据才会被消费者读到。

**重复消费问题**：例如消费完数据要ack一下的，但是进程被kill没有发，误以为还没消费，重启项目之后又收到了。涉及到消费的幂等性，避免数据有重复。解决方向有下面这些：

> 1. 数据库写库，根据主键查下，这条数据有了，就不插入了，update下就好。
> 2. 写redis的，就没事，因为每次都是set，天然幂等性。
> 3. 上面两个都不是，就让生产者发送每条数据加个全局唯一id，类似订单id之类的，到redis查下之前是否消费过，没有消费过，现在消费了，就再写到redis。
> 4. 基于数据库的唯一键来保证重复数据不会插入多条。

**消息丢失**：

> 丢失的可能：1.消息在传入过程中丢失，2.MQ挂了，消息跟着没了，3.消费者收到消息还没处理就挂了。

第一个，RabbitMQ有事务channel.txSelect，发送消息没被mq成功收到就回滚事务channel.txRollback，然后重试发送消息，收到了，那么可以提交事务channel.txCommit。这么弄基本上吞吐量会下来，因为太耗性能。为了不丢，可以开启confirm模式，生产者设置开启confirm模式之后，每次写消息都会分配一个唯一的id，写入RabbitMQ了，MQ会回传一个ack消息，说ok了，如果RabbitMQ没能处理这个消息，会回调你的一个nack接口，说失败了，你可以重试，而且你可以结合这个机制在内存唯一每个消息id的状态，超过一定时间没有收到回调，就重发。（waitForConfirm/回调）

第二个在MQ做持久化，步骤：1. 创建queue时设置持久化元数据，2.发送消息deliveryMode设置为2才行。（创建channel时有参数设置true）

第三个也是要做ack，关闭RabbitMQ自动ack，让消费者完主动ack回MQ（basicAck方法）。

kafka避免数据丢失做法则是要求partition至少两个副本，leader至少感知一个follower没掉队，生产者写入数据必须要写入所有副本之后才是写成功，写入失败，无限重试。

**消息顺序执行**：

1. RabbitMQ：拆分到多个queue，每个queue一个consumer；或者一个queue对应一个consumer，然后这个consumer内部用内存队列做排队，然后分发给底层不同的worker来处理。
2. Kafka：一个topic一个partition一个consumer内部单线程消费但不会用。写N个内存queue，具有相同key的数据都到同一个内存queue；对于N个线程，每个线程分别消费一个内存queue即可。

**消息大量堆积**：

* 先修复consumer，确保其恢复消费速度。
* 新建一个topic，partition是原来的10倍，临时建立好原先10倍的queue数量。
* 然后写一个临时的分发数据的consumer程序，这个程序部署上去消费积压的数据，消费之后不做耗时的处理，直接均匀轮询写入临时建立好的10倍数量的queue。
* 临时征用10倍的机器来部署consumer，每一批consumer消费一个临时queue的数据，这种做法相当于临时将queue资源和consumer资源扩大10倍，以正常10倍的速度来消费数据。
* 等快速消费完积压数据之后，得恢复原先部署的架构，重新用原来的consumer机器来消费消息。

**mq中的消息过期失效了**：有些数据设置了过期时间，但是大量积压下没有被消费，则被直接丢失了。一般可以在不活跃时间，重查出数据再跑。

**mq写满了**：按消息大量积压的来不及了，只能写程序消费一个丢弃一个，然后在按不活跃数据重查数据再跑。

**mq消息确认**：Rabbitmq生产者有channel.waitForConfirms()机制/事务（但是回滚动会把全部都回滚，性能一般）。消费者有1-自动应答，2-手动应答（channel.basicAck(envelope.getDeliveryTag(), false);，失败basicReject（..））。自动应答存在缺陷，消息被消费者接收到就应答，但是如果消费者消费消息失败，这时候会造成消息丢失。手动应答我们可以手工控制，在消息被消费者消息完之后再手工给予应答，这样消息不会丢失。

# ES

分布式的文档储存引擎，分布式的搜索引擎和分析引擎，支持PB级的数据。秒级。

Cluster集群->多个Node节点->shard分片和replica副本。

与数据库类比，则index是数据库，type是数据表，document是行数据。

**原理**：基于lucene，核心思想是在多台机器上启动多个ES进程实例，组成一个集群。es中储存数据的基本单位是索引，一个索引差不多就是相当于是mysql里的一张表。

分多个shard的好处一是支持横向扩展（数据越多就加到新机器），二是提高性能，一个查询多个机器并行执行。

**es写数据过程**：

> * 客户端选择一个node发送请求过去，这个node就是coordinating node协调节点。
> * 协调节点对document进行路由，将请求转发给对应的node（有primary shard）。
> * 实际的node上的primary shard处理请求，然后将数据同步到replica node。
> * 协调节点如果发现primary node和所有的replica node都搞定之后，就返回响应结果给客户端。

**es读数据过程**：通过doc id来查询，会根据doc id进行hash，判断分配到哪个shard。

> * 客户端发送请求到任意一个node，成为coordinating node协调节点。
> * 协调节点对doc id进行哈希路由，将请求转发到对应的node，此时会使用round-robin随机轮询算法，在primary shard以及其所有replica中随机选择一个，让读请求负载均衡。
> * 接收请求的node返回ducument给coordinatingnode协调节点。
> * 协调节点返回document给客户端。

**es搜索数据**：最强大的是做全文检索。

> * 客户端发送请求到一个coordinating node协调节点。
> * 协调节点将搜索请求转发到所有的shard对应的primary node或replica node。
> * query phase：每个shard将自己的搜索结果（其实就是一些doc id）返回给协调节点，由协调节点继续数据的合并、排序、分页等操作，产出最终结果。
> * fetch phase：接着由协调节点根据doc id去各个节点上拉取实际的document数据，最终返回给客户端

**总结一下**，数据先写入内存 buffer，然后每隔 1s，将数据 refresh 到 os cache，到了 os cache 数据就能被搜索到（所以我们才说 es 从写入到能被搜索到，中间有 1s 的延迟）。每隔 5s，将数据写入 translog 文件（这样如果机器宕机，内存数据全没，最多会有 5s 的数据丢失），translog 大到一定程度，或者默认每隔 30mins，会触发 commit 操作，将缓冲区的数据都 flush 到 segment file 磁盘文件中。

**删除/更新数据底层原理**：删除操作，commit的时候会生成一个.del文件，将某个doc标识为deleted状态，搜索的试试根据.del文件就知道这个doc是否被删除了。更新是将原来的doc标识为deleted状态，然后新写入一条数据。segment file默认一秒钟一个，越来越多，定期merge，这是deleted状态doc物理删除。

**数据量大之后查询慢**：单纯去查segment file磁盘文件，速度得按秒级来，但是es有机制是将查询数据缓存到filesystem cache，理论上filesystem cache够大缓存所有，那么速度可以毫秒级。最佳是至少一半数据放到filesystem cache，当然也可以把索引数据都放到filesystem cache，例如一行数据三十个字段，只cache其中三个要查的数据，采用es+mysql/hbase，一般建议es+hbase。如果还是多，那就数据预热、冷热分离（冷热数据各写入一个索引，不同机器以使尽可能保证cache热数据）。

document模型设计避免复杂的关联查询，大表设计这样。

**es分页很差**：翻页越深，性能越差，因为分布式汇总数据的原因。scroll api/search_after像下拉微博一样。



# 缓存

用途：高性能、高并发。

> 像商品SPU做了缓存，每个商家的SPU可能还得缓存不一样，因为查商品需要，看订单也要，比较频繁。
>
> 对外的物流信息查询，一般短时间内变化过少，可以缓存半个小时，避免多次查询刷新。

与memcached区别：redis支持复杂的数据结构，memcached只是String；redis原生支持集群模式，memcached需要依靠客户端实现集群发片写入数据。redis是单核，小数据性能更高，memcached储存100k以上的数据，性能更高。

**线程模型**：redis内部使用文件事件处理器file event handler，这个文件事件处理器是单线程的，它采用IO多路复用机制同时监听多个socket，将产生事件的socket压人内存队列中，事件反派器根据socket上的事件类型来选择对应的事件处理器进行处理。文件事件处理器的结构包括4个部分：多个socket、IO多路复用程序、文件事件分派器、事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）。

**效率高原因**：纯内存操作，非阻塞IO多路复用，C语言实现，单线程避免多线程繁杂的上下文切换和竞争。

**过期策略**：定期删除、惰性删除。

**内存淘汰机制**：以下情况均是当内存不足以容纳新写入数据时

* noeviction：新写入数据操作会报错。
* allkeys-lru：在键空间中，移除最近最少使用的key(最常用)。
* allkeys-random：在键空间中，随机移除某个key。
* volatile-lru：设置了过期时间的键空间之中，移除最近最少使用的key。
* volatile-random：设置了过期时间的键空间中，随机移除某个key。
* volatile-ttl：设置了过期时间的键空间中，有更早过期时间的key优先移除。

*LinkedHashMap构造函数accessOrder=true，在get/put的时候把数据放到最后，旧数据还在前，实现LRU。

主从结构、哨兵模式、集群模式（一致性hash，每个节点保存一份元数据）。

单机redis能过承载QPS大概就在上万到几万不等。

**RDB**：

* 一个时刻生成一个redis数据，冷备份，可以存到本地或CDN上。
* fork子进程去执行，对主程影响较小。
* 恢复上更快。
* 一般会丢失5分钟数据。
* fork子进程如果遇上数据文件淘宝多，会导致对客户端提供服务暂停。

**AOF**：

* 一般一秒执行一次fsync，最多丢失1秒钟数据。
* AOF日志文件以append-only模式写入，没有任何磁盘寻址的开销，写入性能高文件不容易损坏，易修复。
* AOF过大会重写，对指令进行压缩，创建一份恢复数据最小日志。旧日志还会写，后面交换。
* 做紧急修复，flushall清空了所有数据，把AOF中flushall命令删掉再恢复。
* AOF数据大小给RDB大。
* AOF给RDB性能低，因为一秒一次fsync，但还是很高的。
* 为了避免错漏，AOF重做的数据是基于当时内存中的数据进行指令的重新构建。

**避免缓存击穿**：

* 缓存不会更新则尝试将该热点数据设置为永不过期。
* 若缓存更新不频繁，做互斥锁，少量请求到数据库，形成缓存，后面请求都缓存。
* 缓存频繁或构建时间长，利用定时线程过期前重新构建和延后过期时间。

**Cache Aside Pattern**：先读缓存，再读数据库；更新时更新数据库，再删缓存。

为什么不更新缓存：跨表字段复杂，是否每次更新都得更新缓存，读未必频繁。

但是会遇到后删缓存失败，办法是先删缓存，再更新数据库，这样就算数据库更新失败了，缓存是空的，再读取的时候，依旧是旧数据，再更新到缓存中。访问高发的时候会出现缓存不一致（数据库还没更新完缓存又生成了），办法：更新数据时，根据数据的唯一标识，发送到相应JVM内部队列（重复可去重），读取数据不在缓存中，那么将重新执行“读取数据+更新缓存”的操作， 根据唯一标识路由后，也发送到同一个JVM内部队列中。一个对列对应一个工作线程，每个工作线程串行拿到对应的操作，然后一条一条执行。（注意超时处理、内存积压）。

**CAS问题**：对同一个key多次写，但是顺序错了，造成最后数据有问题。一来分布式锁，写的时候只能一个写，二来数据库数据加时间戳，时间戳没有更晚就不覆盖缓存。

**分布式锁**：setnx加锁，lua解锁（检查加锁的value是否一样，避免删掉别的程序加的锁），这种方式在单机/主从故障时有问题，于是采用redlock算法（过半redis主节点加锁成功才成功）。

**缓存与数据库不一致怎么办**：先删缓存再写库则会造成从库数据还没完成，别的线程读到旧数据，解决办法是当从库有数据更新之后，把缓存删除。

**主从数据库不一致如何解决**：忽略、强制读主库、选择性读主库（缓存必须都主库的key，过期时间大概为主从同步时间，当缓存有这个数据，直接读取主库，不然从库读取）。



# 分库分表

**Sharding-jdbc**：SQL语法支持比较多，支持分库分表，读写分离，分布式id，柔性事务（最大努力送达型事务、TCC事务）。不用部署，运维成本低，不需要代理层的二次转发请求，性能很高。升级则需要各个系统都升级。

**Mycat**：缺点在于需要部署，运维成本高，对各个项目是透明的，升级只需要升级中间件。

> rang：按时间分，例如一个月一个库，但是一般都访问最新数据的，会有大量请求到新的库去。
>
> hash：平均每个库的数据量和请求压力。扩容比较麻烦，会有数据迁移过程，重新计算hash。

**怎么迁移**：停机迁移方案（把旧库数据导到新库）；双写迁移方案（新数据进入新旧库，后台数据迁移临时从旧到新，根据gmt_modify最新保留新旧库重复数据）

**MySQL主从复制原理**：主库将变更写入binglog日志，然后从库链接到主库之后，从库有一个IO线程，将主库的binlog日志拷贝到自己本地，写入一个relay中继日志中，接着从库中有一个SQL线程会从中继日志读取binlog，然后执行binlog日志中的内容，也就是在本地再执行一遍SQL，这样就可以保证自己跟主库数据一样。会存在延迟，所以mysql有两个机制，**半同步复制**（要求主库接到从库写入本地relaylog返回ack才算成功，解决主库数据丢失），**并行复制**：从库开启多个线程，并行读取relaylog中不同库的日志，然后并行重放不同库的日志。

**延时较严重解决办法**：

* 分库，主库多了承受的写办法减少，主从延迟降低。
* 打开mysql支持的并行复制，但是单库写并发达到2000/s，没有意义。
* 代码避免改完再查，再做逻辑查处理。
* 代码上是改完要再查出来执行，这个查询改直连主库，不太推荐。

# 设计高并发系统

系统拆分微服务、缓存、MQ、分库分表、读写分离。

# zookeeper

* 分布式协调：A系统注册监听一个节点，B系统改变该节点，A系统收到通知做出反应。
* 分布式锁。
* 元数据/配置信息管理：像微服务的注册信息。
* HA高可用性：主备两个，主的挂了通过zookeeper系统切换到备。

# 限流

* 计数器
* 滑动窗口
* 漏桶：规定固定容量的桶，对于流进的水我们无法估计，对于流出的水我们可以控制。
* 令牌桶：一个线程往令牌桶加token，满则不加，请求是取走一个，空了就拒绝请求。
* spring cloud gateway
* sentinel：结合spring-cloud-alibaba。

# 微服务

拆分微服务的时候应该从1.停止挖掘（就是新的功能开始做在微服务上，增加胶水代码，即不要被单体应用污染到微服务），2.前后端分离。3.抽出服务（最好抽的先开始，获益大的开始，资源消耗大的开始）



# Tomcat调优

1. JVM调优：-Xms/JVM初始化堆的大小，-Xmx/JVM堆最大值。

2. 禁用DNS查询：在Connector标签中使用enableLookups="false"。

3. 启动NIO模式

   修改server.xml里的Connector节点(就是那个写了port为8080的地方)，修改protocol为org.apache.coyote.http11.Http11NioProtocol。

4. 执行器优化（线程池）

​      在tomcat中每一个用户请求都是一个线程，所以可以使用线程池提高性能。配置<Executor>标签。Connector标签中使用executor指向上面。

acceptCount最大等待数、maxConnections最大连接数，maxThreads最大线程数。

5. 禁用AJP协议：本来是用于重用链接的，但用了nginx后不需要，8009端口那个protocol协议。
6. 配置gzip压缩（HTTP压缩）。https协议下还可能走8443端口还得改这个端口的配置。也可以放在nginx处理
7. JVM调优：堆大小、垃圾回收收集器。



# HBASE数据库

一个列式储存数据库
