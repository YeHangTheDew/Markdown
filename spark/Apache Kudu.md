# Apache Kudu

## 一、诞生背景

在KUDU之前，大数据主要以两种方式存储；

（1）静态数据：以HDFS引擎作为存储引擎，适用于高吞吐量的离线大数据分析场景。这类存储的局限性是数据无法进行随机的读写。

（2）动态数据：以HBase、Cassandra作为存储引擎，适用于大数据随机读写场景。局限性是批量读取吞吐量远不如HDFS，不适用于批量数据分析的场景。从上面分析可知，这两种数据在存储方式上完全不同，进而导致使用场景完全不同，但在真实的场景中，边界可能没有那么清晰，面对既需要随机读写，又需要批量分析的大数据场景，该如何选择呢？

​		这个场景中，单种存储引擎无法满足业务需求，我们需要通过多种大数据工具组合来满足这一需求，如下图所示

![image-20200812233618635](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200812233618635.png)

如上图所示，数据实时写入 HBase，实时的数据更新也在 HBase 完成，为了应对 OLAP 需求，我们定时将 HBase 数据写成静态的文件（如：Parquet）导入到 OLAP 引擎（如：Impala、hive）。这一架构能满足既需要随机读写，又可以支持 OLAP 分析的场景，但他有如下缺点：

(1) 架构复杂。从架构上看，数据在 HBase、消息队列、HDFS 间流转，涉及环节太多，运维成本很高。并且每个环节需要保证高可用，都需要维护多个副本，存储空间也有一定的浪费。最后数据在多个系统上，对数据安全策略、监控等都提出了挑战。
(2) 时效性低。数据从 HBase 导出成静态文件是周期性的，一般这个周期是一天（或一小时），在时效性上不是很高。
(3) 难以应对后续的更新。真实场景中，总会有数据是延迟到达的。如果这些数据之前已经从 HBase 导出HDFS，新到的变更数据就难以处理了，一个方案是把原有数据应用上新的变更后重写一遍，但这代价又很高。为了解决上述架构的这些问题，KUDU 应运而生。KUDU 的定位是 Fast Analytics on Fast Data，是一个既支持随机读写、又支持 OLAP 分析的大数据存储引擎。

![image-20200812233744134](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200812233744134.png)

从上图可以看出，KUDU 是一个折中的产品，在 HDFS 和 HBase 这两个偏科生中平衡了随机读写和批量分析的性能。

### 1. 什么是kudu？

​			Apache Kudu 是由 Cloudera 开源的存储引擎，可以同时提供低延迟的随机读写和高效的数据分析能力。它是一个融合 HDFS 和 HBase 的功能的新组件，具备介于两者之间的新存储组件。Kudu 支持水平扩展，并且与 Cloudera Impala 和 Apache Spark 等当前流行的大数据查询和分析工具结合紧密。

### 2. 应用场景？

​	适用于那些既有随机访问，也有批量数据扫描的复合场景
​	高计算量的场景
​	使用了高性能的存储设备，包括使用更多的内存
​	支持数据更新，避免数据反复迁移
​	支持跨地域的实时数据备份和查询

## 二 、kudu架构

​			与 HDFS 和 HBase 相似，Kudu 使用单个的 Master 节点，用来管理集数据，并且使用任意数量的 Tablet Server（类似 HBase 中的 RegionSer色）节点用来存储实际数据可以部署多个 Master 节点来提高容错性

![image-20200813002334297](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200813002334297.png)

### 1． Table

​			表（Table）是数据库中用来存储数据的对象，是有结构的数据集合。kudu中的表具有 schema（纲要）和全局有序的 primary key（主键）。kudu 中一个table 会被水平分成多个被称之为 tablet 的片段。

### 2． Tablet

​           一个 tablet 是一张 table 连续的片段，tablet 是 kudu 表的水平分区，类似于 HBase 的 region。每个 tablet 存储着一定连续 range 的数据（key），且 tablet 两两间的 range 不会重叠。一张表的所有 tablet 包含了这张表的所有 key 空间。tablet 会冗余存储。放置到多个 tablet server 上，并且在任何给定的时间点，其中一个副本被认为是 leader tablet,其余的被认之为 follower tablet。每个 tablet 都可以进行数据的读请求，但只有 Leader tablet 负责写数据请求。

​	元数据：存储自己的元信息，例如表明、列名、列类型

​	数据：存放于表中的数据

### 3． Tablet Server

​			tablet server 集群中的小弟，负责数据存储，并提供数据读写服务一个 tablet server 存储了 table 表的 tablet，向 kudu client 提供读取数据服务。对于给定的 tablet，一个 tablet server 充当 leader，其他tablet server 充当该 tablet 的 follower 副本。只有 leader 服务写请求，然而 leader 或 followers 为每个服务提供读请求 。一个 tablet server 可以服务多个 tablets ，并且一个 tablet 可以被多个 tablet servers 服务着。

- Tablet server也是tablet，但是其中存储的是表数据

- 功能：存储、访问、压缩，还负责将数据复制到其他机器

- 限制：

  1. Kudu最多支持300个服务器，建议Tablet server最多不超过100个

  2. 建议每个Table server至多包含2000个tablet（ 包含Follower）

  3. 建议每个表在每个Tablet server中之多包含60个tablet

  4. 每个tablet server 至多管理8TB数据

  5. 理想环境下，一个tablet leader应该对应一个CPU核心，以

     保证最优的扫描性能（侧面说明硬件要求比hadoop高）

### 4． Master Server

集群中的老大，负责集群管理、元数据管理等功能。

## 三、kudu安装使用

## 四、kudu分区

​		为了提供可扩展性，Kudu 表被划分为称为 tablet 的单元，并分布在许多tablet servers 上。行总是属于单个 tablet 。将行分配给 tablet 的方法由在表创建期间设置的表的分区决定。 kudu 提供了 3 种分区方式。

### 1. Range Partitioning ( 范围分区 )

​		范围分区可以根据存入数据的数据量，均衡的存储到各个机器上，防止机器出现负载不均衡现象.

### 2. Hash Partitioning（哈希分区）

​		哈希分区通过哈希值将行分配到许多 buckets ( 存储桶 )之一； 哈希分区是一种有效的策略，当不需要对表进行有序访问时。哈希分区对于在 tablet 之间随机散布这些功能是有效的，这有助于减轻热点和 tablet 大小不均匀。

### 3. Multilevel Partitioning ( 多级分区 )

​		Kudu 允许一个表在单个表上组合多级分区。 当正确使用时，多级分区可以保留各个分区类型的优点，同时减少每个分区的缺点

## 五、Kudu 原理

### 1． table 与 schema

​		Kudu 设计是面向结构化存储的，因此，Kudu 的表需要用户在建表时定义它的 Schema 信息，这些 Schema 信息包含：列定义（含类型），Primary Key 定义（用户指定的若干个列的有序组合）。数据的唯一性，依赖于用户所提供的Primary Key 中的 Column 组合的值的唯一性。Kudu 提供了 Alter 命令来增删列，但位于 Primary Key 中的列是不允许删除的。从用户角度来看，Kudu 是一种存储结构化数据表的存储系统。在一个 Kudu集群中可以定义任意数量的 table，每个 table 都需要预先定义好 schema。每个table 的列数是确定的，每一列都需要有名字和类型，每个表中可以把其中一列或多列定义为主键。这么看来，Kudu 更像关系型数据库，而不是像 HBase、Cassandra 和 MongoDB 这些 NoSQL 数据库。不过 Kudu 目前还不能像关系型数据一样支持二级索引。Kudu 使用确定的列类型，而不是类似于 NoSQL 的“everything is byte”。带来好处：确定的列类型使 Kudu 可以进行类型特有的编码,可以提供元数据给其他上层查询工具。

### 2． kudu 底层数据模型

​		Kudu 的底层数据文件的存储，未采用 HDFS 这样的较高抽象层次的分布式文
件系统，而是自行开发了一套可基于 Table/Tablet/Replica 视图级别的底层存
储系统。
这套实现基于如下的几个设计目标：
• 可提供快速的列式查询
• 可支持快速的随机更新
• 可提供更为稳定的查询性能保障

![image-20200812235931589](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200812235931589.png)

​		一张 table 会分成若干个 tablet，每个 tablet 包括 MetaData 元信息及若干个 RowSet。RowSet 包含一个 MemRowSet 及若干个 DiskRowSet，DiskRowSet 中包含一个BloomFile、Ad_hoc Index、BaseData、DeltaMem 及若干个 RedoFile 和 UndoFile。MemRowSet：用于新数据 insert 及已在 MemRowSet 中的数据的更新，一个MemRowSet 写满后会将数据刷到磁盘形成若干个 DiskRowSet。默认是 1G 或者或者 120S。
​		DiskRowSet：用于老数据的变更，后台定期对 DiskRowSet 做 compaction，以删除没用的数据及合并历史数据，减少查询过程中的 IO 开销。
​		BloomFile：根据一个 DiskRowSet 中的 key 生成一个 bloom filter，用于
快速模糊定位某个 key 是否在 DiskRowSet 中。
​		Ad_hocIndex：是主键的索引，用于定位 key 在 DiskRowSet 中的具体哪个偏
移位置。
​		BaseData 是 MemRowSet flush 下来的数据，按列存储，按主键有序。
​		UndoFile 是基于 BaseData 之前时间的历史数据，通过在 BaseData 上 apply UndoFile 中的记录，可以获得历史数据。
​		RedoFile 是基于 BaseData 之后时间的变更记录，通过在 BaseData 上 RedoFile 中的记录，可获得较新的数据。
​		DeltaMem 用于 DiskRowSet 中数据的变更，先写到内存中，写满后 fl
磁盘形成 RedoFile。

​		EDO 与 UNDO 与关系型数据库中的 REDO 与 UNDO 日志类似（在关系型数据库
中，REDO 日志记录了更新后的数据，可以用来恢复尚未写入 Data File 的已成
功事务更新的数据。而 UNDO 日志用来记录事务更新之前的数据，可以用来在事
务失败时进行回滚）

![image-20200813000033566](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200813000033566.png)

​		MemRowSets 可以对比理解成 HBase 中的 MemStore, 而 DiskRowSets 可理解成 HBase 中的 HFile。MemRowSets 中的数据被 Flush 到磁盘之后，形成 DiskRowSets。DisRowSets中的数据，按照 32MB 大小为单位，按序划分为一个个的 DiskRowSet。DiskRowSet中的数据按照 Column 进行组织，与 Parquet 类似。这是 Kudu 可支持一些分析性查询的基础。每一个 Column 的数据被存储在一个相邻的数据区域，而这个数据区域进一步被细分成一个个的小的 Page 单元，与 HBase File 中的 Block 类似，对每一个 Column Page 可采用一些 Encoding 算法，以及一些通用的 Compression 算法。既然可对 Column Page 可采用 Encoding以及 Compression 算法，那么，对单条记录的更改就会比较困难了。

​	前面提到了 Kudu 可支持单条记录级别的更新/删除，是如何做到的？

​		与 HBase 类似，也是通过增加一条新的记录来描述这次更新/删除操作的。
DiskRowSet 是不可修改了，那么 KUDU 要如何应对数据的更新呢？在 KUDU 中，
把 DiskRowSet 分为了两部分：base data、delta stores。base data 负责存储
基础数据，delta stores 负责存储 base data 中的变更数据.

![image-20200813000110397](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200813000110397.png)

​	如上图所示，数据从 MemRowSet 刷到磁盘后就形成了一份 DiskRowSet（只包 含 base data ）， 每 份 DiskRowSet 在 内 存 中 都 会 有 一 个 对 应 的DeltaMemStore，负责记录此 DiskRowSet 后续的数据变更（更新、删除）。DeltaMemStore 内部维护一个 B-树索引，映射到每个 row_offset 对应的数据变更。DeltaMemStore 数据增长到一定程度后转化成二进制文件存储到磁盘，形成一个 DeltaFile，随着 base data 对应数据的不断变更，DeltaFile 逐渐增长。

### 3． tablet 发现过程

​		当创建 Kudu 客户端时，其会从主服务器上获取 tablet 位置信息，然后直接与服务于该 tablet 的服务器进行交谈。为了优化读取和写入路径，客户端将保留该信息的本地缓存，以防止他们在每个请求时需要查询主机的 tablet 位置信息。随着时间的推移，客户端的缓存可能会变得过时，并且当写入被发送到不再是 tablet 领导者的 tablet 服务器时，则将被拒绝。然后客户端将通过查询主服务器发现新领导者的位置来更新其缓存。

![image-20200813000145432](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200813000145432.png)

### 4． kudu 写流程

​		当 Client 请求写数据时，先根据主键从 Master Server 中获取要访问的目标 Tablets，然后到依次对应的 Tablet 获取数据。因为 KUDU 表存在主键约束，所以需要进行主键是否已经存在的判断，这里就涉及到之前说的索引结构对读写的优化了。一个 Tablet 中存在很多个 RowSets，为了提升性能，我们要尽可能地减少要扫描的 RowSets 数量。首先，我们先通过每个 RowSet 中记录的主键的（最大最小）范围，过滤掉一批不存在目标主键的 RowSets，然后在根据 RowSet 中的布隆过滤器，过滤掉确定不存在目标主键的 RowSets，最后再通过 RowSets 中的 B-树索引，精确定位目标主键是否存在。如果主键已经存在，则报错（主键重复），否则就进行写数据（写 MemRowSet）。

![image-20200813000218453](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200813000218453.png)

### 5． kudu 读流程

​		数据读取过程大致如下：先根据要扫描数据的主键范围，定位到目标的Tablets，然后读取 Tablets 中的 RowSets。在读取每个 RowSet 时，先根据主键过滤要 scan 范围，然后加载范围内的base data，再找到对应的 delta stores，应用所有变更，最后 union 上 MemRowSet中的内容，返回数据给 Client。

![image-20200813000308736](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200813000308736.png)

### 6． kudu 更新流程

​		数据更新的核心是定位到待更新数据的位置，这块与写入的时候类似，就不
展开了，等定位到具体位置后，然后将变更写到对应的 delta store 中。

![image-20200813000331932](C:\Users\yechh\AppData\Roaming\Typora\typora-user-images\image-20200813000331932.png)

## 六、kudu 帮手系列

### 1. impala

#### 使用场景：

​		Impala 类似hive，主要是提供数据分析人员来进行数据分析和查询的工具，虽然提供了impala通过jdbc访问kudu的api。

（kudu不支持sql）

#### 与hive的区别：

1. impala的执行速度比hive要快的多，但是架构决定了impala无法执行长时间的sql

2. Hive执行计划使用MR,MR专门为大规模的数据分析而设计，MR在数据规模上有显著优势
3. impala有自己的执行计划，是一个MPP架构，MPP大规模并行（MPP架构往往会把节点分开，每个节点对应每个数据存储的位置）

#### 为什么impala会跟hive整合呢？

目的：为了使用hive表来操作kudu。

Impala和hive是强依赖关系，Impala依赖Hive的Metastore



### 2. spark

#### 基础认识   

1. scala是面对对象函数式编程语言,且基于jvm上运行。

2. spark是分布式流式计算引擎。

3. SparkSession 是spark程序的入口点（2.x）

4. RDD: 分布式弹性数据集，对数据的一种抽象，Row

5. DataFrame：与RDD类似，分布式并行计算的弹性数据集



kudu主页：https://kudu.apache.org/docs/index.html

kudu的分区详细信息：https://kudu.apache.org/docs/schema_design.html

操作kudu的各种形式：https://kudu.apache.org/docs/developing.html#_viewing_the_api_documentation

kudu python客户端源代码：https://github.com/apache/kudu/blob/master/python/kudu/client.pyx

kudu scala spark操作详细例子：https://blog.cloudera.com/blog/2017/02/up-and-running-with-apache-spark-on-apache-kudu/