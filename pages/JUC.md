其实就是指java.util.concurrent/.atomic包下的东西，例如常见的有volatile<特性禁止指令重排(1-开辟空间，2-空间初始化,3-空间指向地址变量)、适用于共享变量的可见、一写多读>、cas、concurrentHashMap、CountDownLatch、cyclicBarrier，Semaphore、实现Callable接口创建线程、ReentrantLock同步锁、ReadWriterLock读写锁、线程池。

**ReentrantLock**：Synchroized原理是JVM中的mintorenter和mintorexit对被锁对象的头加标记。而ReentrantLock也是类似，利用AQS实现，里面用了volatie修饰一些变量，保存可见。

**ReentrantLock与Synchroized差异**：前者等待可中断即等待的线程可以选择超时放弃。可以判断是否有线程在排队，可以响应中断，实现公平锁，释放锁要unlock。

**CAS**：compare and swap，通过保存一个旧值，更新时用旧值来比较当前值，没有改变则更新为更新后的值。像atomicInteger就是利用volatile修饰值，然后使用上面说的cas这个方法来做。

**AQS**:位于java.util.concurrent.locks包下，AQS是一个用来构建锁和同步器的框架，例如ReentrantLock和Semaphore等等，**原理**：被请求的共享资源空闲，那么当前空闲的线程就会被标记为运行状态，资源也被标记为被锁定，需要一套线程阻塞等待以及被唤醒时锁分配的机制，AQS使用CLH队列锁（自旋锁，先来先服务，保证无饥饿）实现，即将暂时获取不到锁的线程放在队列中。

**concurrentHashMap**是怎么保证线程安全的？与Hashtable不同，concurrentHashMap不会锁住整个表，JDK7及之前是做了一个分段锁，将表分成16段，每段单独加锁，每段的里面是一个不会互相干扰的数组。JDK8以及之后，分段锁的做法成了CAS加synchroized，段里面的数组超过8个之后会成为红黑树。