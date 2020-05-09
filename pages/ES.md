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
> * 接收请求的node返回document给coordinatingnode协调节点。
> * 协调节点返回document给客户端。

**es搜索数据**：最强大的是做全文检索。

> * 客户端发送请求到一个coordinating node协调节点。
> * 协调节点将搜索请求转发到所有的shard对应的primary node或replica node。
> * query phase：每个shard将自己的搜索结果（其实就是一些doc id）返回给协调节点，由协调节点继续数据的合并、排序、分页等操作，产出最终结果。
> * fetch phase：接着由协调节点根据doc id去各个节点上拉取实际的document数据，最终返回给客户端

**总结一下写数据**，数据先写入内存 buffer，然后每隔 1s，将数据 refresh 到 os cache，到了 os cache 数据就能被搜索到（所以我们才说 es 从写入到能被搜索到，中间有 1s 的延迟）。每隔 5s，将数据写入 translog 文件（这样如果机器宕机，内存数据全没，最多会有 5s 的数据丢失），translog 大到一定程度，或者默认每隔 30mins，会触发 commit 操作，将缓冲区的数据都 flush 到 segment file 磁盘文件中。

**删除/更新数据底层原理**：删除操作，commit的时候会生成一个.del文件，将某个doc标识为deleted状态，搜索的时候根据.del文件就知道这个doc是否被删除了。更新是将原来的doc标识为deleted状态，然后新写入一条数据。segment file默认一秒钟一个，越来越多，定期merge，这是deleted状态doc物理删除。

**数据量大之后查询慢**：单纯去查segment file磁盘文件，速度得按秒级来，但是es有机制是将查询数据缓存到filesystem cache，理论上filesystem cache够大缓存所有，那么速度可以毫秒级。最佳是至少一半数据放到filesystem cache，当然也可以把索引数据都放到filesystem cache，例如一行数据三十个字段，只cache其中三个要查的数据，采用es+mysql/hbase，一般建议es+hbase。如果还是多，那就数据预热、冷热分离（冷热数据各写入一个索引，不同机器以使尽可能保证cache热数据）。

document模型设计避免复杂的关联查询，大表设计这样。

**es分页很差**：翻页越深，性能越差，因为分布式汇总数据的原因。scroll api/search_after像下拉微博一样。

