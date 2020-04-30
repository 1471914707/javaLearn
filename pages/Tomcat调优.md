1. JVM调优：-Xms/JVM初始化堆的大小，-Xmx/JVM堆最大值。

2. 禁用DNS查询：在Connector标签中使用enableLookups="false"。

3. 启动NIO模式

   修改server.xml里的Connector节点(就是那个写了port为8080的地方)，修改protocol为org.apache.coyote.http11.Http11NioProtocol。

4. 执行器优化（线程池）

​      在tomcat中每一个用户请求都是一个线程，所以可以使用线程池提高性能。配置<Executor>标签。Connector标签中使用executor指向上面。

acceptCount最大等待数、maxConnections最大连接数，maxThreads最大线程数。

5. 禁用AJP协议：本来是用于重用链接的，但用了nginx后不需要，8009端口那个protocol协议。
6. 配置gzip压缩（HTTP压缩）。https协议下还可能走8443端口还得改这个端口的配置。也可以放在nginx处理