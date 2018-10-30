# HBaseTree
HBase 技术研究

![](https://i.imgur.com/XKgWQsD.png)

![](https://i.imgur.com/9JukYh2.jpg)

<pre>
列式存储数据库以列为单位聚合数据，然后将列值顺序的存入磁盘，这种存储方法不同于行式存储的传统数据库库，行式存储数据库连续的
存储整行。

列式存储的出现主要基于这样一种假设：对于特定的查询，不是所有的值都是必须的，尤其是在分析型数据库里，这种情况很常见。

在这种设计中，减少I/O只是众多主要因素之一，它还有其它的优点：
    因为列的数据类型天生是相似的，即便逻辑上每一行之间有轻微的不同，但仍旧比按行存储的结构聚集在一起的数据更利于压缩，因为大多数的压缩只关注有限的压缩窗口
</pre>

![](https://i.imgur.com/N4Q4UaC.jpg)

<pre>
一行由若干列组成，若干列又构成一个列族，一个列族的所有列存储在同一个底层的存储文件里，这个列族文件叫做HFile

列族需要在表创建时就定义好，并且不能修改得太频繁，数量也不能太多。

常见的引用列的格式为  family:qualifier，列族的数量最多只限于几十，实际更小，然而一个列族中的列的数量没有限制。

HBase是按照BigTable模型实现的，是一个稀疏的，分布式的，持久化的，多维的映射，由行键，列键，时间戳索引。

存储文件通常保存在hadoop分布式文件系统HDFS中，HDFS提供一个可扩展的，持久的，冗余的HBase存储层。存储文件通过将更改写入到可配置数目的物理服务器中，以保证数据不丢失。

每次更新数据时，都会先将数据记录在（commit log）提交日志中，在HBase中这叫做预写日志（write-ahead log, WAL），然后才会将这些数据写入内存中的
memstore中，一旦内存保存的写入数据的累积超过一个给定的最大值，系统就会将这些数据移出内存作为HFile文件刷写到磁盘中，数据移出内存之后，系统就会
丢弃对应的提交日志，只保留未持久化到磁盘中的提交日志，在系统将数据移出memstore写入磁盘的过程中，可以不必阻塞系统的读写，通过滚动内存中的memstore
就可以达到这个目的，即用空的新的memstore获取更新数据，将满的旧的memstore转换成一个文件。
</pre>

<pre>
HBase自动分区
    HBase中扩展和负载均衡的基本单元称为region，region本质上是以行键排序的连续存储的区间，如果region太大，系统就会把它们动态拆分，相反地，就把多个region合并，以减少存储文件数量。

HBase是一个分布式的，持久的，强一致性的存储系统，具有近似最优的写性能和出色的读性能，它充分利用了磁盘空间，支持特定列族切换可压缩算法。
HBase继承自BigTable模型，只考虑单一的索引，类似于RDBMS中的主键，提供了服务器端钩子，可以实施灵活的辅助索引解决方案，此外，它还提供了过滤器功能，减少了网络传输的数据量。
HBase并未将说明性查询语言作为核心实现的一部分，对事务的支持也有限，但行原子性和“读-修改-写”操作在实践中弥补了这个缺陷，它们覆盖了大部分的使用场景并消除了在其它系统中经历过的死锁，等待等问题。
HBase在进行负载均衡和故障恢复时对客户端是透明的，在生产系统中，系统的可扩展性体现在系统自动 伸缩的过程中，更改集群并不涉及重新全量负载均衡和数据重分区，但整个处理过程完全是自动化的。
</pre>