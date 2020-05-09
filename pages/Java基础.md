# java基础

* **执行顺序**：父类静态变量 -> 父类静态代码块 -> 子类静态变量 -> 子类静态代码块 -> 父类非静态变量 -> 父类构造函数 -> 子类非静态变量 -> 子类构造函数。
* **内部类**：静态内部类、成员内部类、局部内部类、匿名内部类。
* **重写**(override)要求子类的返回值小于父类（类型相同），修饰符使用范围也大于父类，抛出异常小于父类。
* **静态方法**不能调用类非静态方法，因为静态方法可以在不生成对象的时候直接调用。
* **默认构造方法**，因为子类构造的时候也调用super()父类，所以需要增加一个默认构造函数，避免编译出错。
* **java只有值传递**：按值调用，按引用调用（其实也是按值调用，因为传的是一个指针地址）
* **线程状态**：初始状态->就绪状态->运行状态、阻塞状态、等待状态、超时等待状态->终止状态。
* **try**：不能单独用，可以不加catch，但是最后要finally{}。
* **synchronized**:优化加了偏向锁，自旋锁/适应性自旋锁，轻量级锁，锁粗化，锁消除

​       synchronized加在类方法上是锁的对象锁，加在静态方法上锁的是类锁。

* **volatile**：直接从内存中取值，同时禁止指令重排，与JMM模型有关。
* **线程池的好处**：1.减少创建和销毁线程的开销、2.任务不用等创建线程即可运行，3.管理，调优，监控。

包括newFixedThreadExecutor（有等待队列过大造成OOM可能）、ScheduleThreadExecutor、newSingleThreadExecutor，newCacheThreadExecutor（有创建线程过大导致OOM风险）。满载后拒绝策略有直接抛异常，抛弃最老的，直接丢弃，由调用者自己执行。

**创建线程的方式**：1.继承thread、2.实现Runable、3.实现callable。

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

**thread拥有的方法**：setPriority(.)，sleep(线程阻塞，让出时间片)，join（让调取它的线程等待本线程执行完），yield（暂停当前执行线程，执行其他同等或更高优先级线程），interrupt（中断线程）,isAlive（测试线程是否处于活动状态）。

**线程安全的理解**：是有全局变量及静态变量引起的，只读没问题，但是写的要考虑线程同步。常量不会改没事，线程内的局部变量或者调用方法前每一个实例都是新建的，那也是线程安全的，因为不会访问共享的资源。

**ReentrantLock 实现原理**：基于AQS来实现。

**Synchronized和Lock的异同**：前者利用Object的notify/wait之类的调度，后者用Condition进行线程间调度。再者前者是可以加在方法/代码片段给JVM执行，后者直接代码段通过代码控制。前者自动释放，后者手动解锁。

**Queue**：实现类LinkedList，ArrayBlockingQueue, LinkedBlockingQueue，PriorityQueue, PriorityBlockingQueue，ConcurrentLinkedQueue。

**Map的多线程安全版本**：

1. Collections.synchronizedMap(Map)，原理是传入一个map和object然后对object加锁操作map。
2. Hashtable，继承Dictionary，初始11，拓容2N+1。与HashMap迭代器是快速失败（fast-fail）不同，Hashtable是安全失败（fail-safe），迭代时可以增加删除修改。
3. ConcurrentHashMap（常用）。

**安全失败（fail—safe）**：java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。

**hashMap拓容**：默认16，负载因子0.75，拓容为2的N次方（使用2次方除了数据重新分配比较匀称之外，还将取余%运算转化成&运算）。

**ArrayList拓容**：默认10，拓容为2.5N，就是newN = N + N * 2；

**Java内存模型（Java Memory Model ,JMM）**就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范。Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。而JMM就作用于工作内存和主存之间数据同步过程。他规定了如何做数据同步以及什么时候做数据同步。

**ThreadLocal**：例如平时单机服务器上验证完登录用户后，把用户信息存放在ThreadLocal中，需要用的时候可以直接get出来。ThreadLocal的作用就是为当前运行线程准备一个专属数据，其他线程访问不到，只有自己可以，也不会存在什么并发问题，但是用完记得要remove掉，因为key为弱引用，value不是，可能value不被GC会造成内存泄露或堆栈溢出。内存泄露？像栈底元素一直被引用着但又一直不用，map中key被改了hashcode，value就找不回了等等。githubPageHelper中记录分页参数也是用的ThreadLocal。

**异常**：都是Throwable的子类，Error基本就是程序挂掉了。Exception也分运行时和非运行时异常，非运行时异常需要catch处理，例如IO操作。运行时异常，例如下标越界，空指针，类不存在，方法不存在，数字格式异常。

**Comparable和Comparator**：Comparable是被类实现的，实现compareto方法，可以在collections.sort(list)直接排序，而Comparator是Collections.sort(list, new Comparator)做一个外部排序器。

**JDK8新特性**：lamda表达式、stream流计算、optional、接口默认实现、新的时间日期类。

JDK8的应用，可以使用**Instant**代替Date，**LocalDateTime**代替Calendar，**DateTimeFormatter**代替SimpleDateFormat。