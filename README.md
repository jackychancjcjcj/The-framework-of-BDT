# The framework of big data technology
大数据架构详解-note
## 综述
例如：运营商数字化转型衍生业务
* SQM（运维质量管理）  
* CSE (客户体验提升)
* MSS（市场运维支撑）
* DMP（数据管理平台）
## 数据获取
info:数据获取涉及技术
  * 探针（电信特有）  
    主要涉及到了IB`（infinite band）`技术
  * 爬虫  
  最基本的系统的结构：  
    > 数据中心  
    >> 服务器  
    >>> 爬虫程序  
    * 在数据中心的不同服务器里，协同工作的方式一般有：
      * 主从式
      一个master管理多个slave，瓶颈出现在master上
      * 对等式
      一致性哈希算法对url域名分析，分配给每台服务器。
  * Flume（采集日志数据）  
  flume的数据流由事件贯穿始终，事件是flume的基本数据单位。这些event由agent的外部source捕捉，进行特定格式化，然后把事件推入channel中。channel可以看作缓冲区，保存事件至sink处理完事件，sink负责持久化日志或者把事件推向另一个source。
  * Kafka（消息中间件）  
  数据采集后需要送到后端进行分析，kafka负责消息转发，保障信息可靠性，匹配前后端的速度差。  
  整个架构主要三个角色，生产者、代理（核心）、消费者。   
  kafka给producer和consumer提供注册的接口，broker承担中间缓存和分发作用。
    * kafka高效性：
      * 直接使用linux文件系统的cache缓存数据  
      * 采用 linux zero-copy 提高发送性能。  
           传统数据发送需要四次的上下文切换，现在数据直接在内核态进行交换，系统上下文切换减少2次，性能提高60%。数据在磁盘的存取代价为o（1）。  
    * 消息管理  
    topic，topic下包含多个partition，每个partition对应逻辑log，由多个segment组成。  
    一般是以物理偏移地址作为index。kafka为每个分区创建文件夹，文件中每个segment会由一个index（索引）和log（message信息）组成。  
## 流处理
根据实时性的不同，也可以分为以IBM InfoSphere Streams为代表的，消息立刻处理；另一种是Spark Streaming，将数据存在内存中，较小的批处理模拟流处理（设置窗口大小Dstream）。  
### Storm  
Twitter开源的分布式实时数据处理系统。
 * Nimbus  
 负责资源分配和任务调度
 * Supervisor  
 负责接收Nimbus分配的任务，启动和停止属于自己管理的worker进程
 * Worker
 运行具体处理组件逻辑的进程
 * Task  
 Worker中每一个Spout/Bolt的线程称为一个task（同一个Spout/Bolt可能会共用一个物理线程，称为executor）
 * Topology  
 Storm中运行的实时应用程序
 * Spout  
 在Topology中产生源数据流的组件，从外部读取数据转化为内部的源数据。是一个主动角色（不断调用接口nextTuple（））
 * Bolt  
 Topology中接受数据然后执行处理的组件。（过滤、函数合并、写数据库等操作）是一个被动角色（有tuple input才会执行）。
 * Tuple  
 消息传递的基本单元。
 * Stream  
 源源不断的Tuple就形成了Stream
#### Storm记录级别容错（亮点一）
容错的意思是，Storm会告诉用户每个消息单元是否在指定时间内被完全处理了。即一个messageId绑定的源Tuple及由该源Tuple后续生成的Tuple经过了Topology中每一个该到达的Bolt的处理。  
在Storm的Topology中有一个系统级组件Acker，用于追踪Tuple的处理路径。（亦或定理，0 xor 0 = 0， 1 xor 0 = 1）  
#### Storm的事务拓扑（亮点二）
事务拓扑的目的是为了满足对消息的严格精确处理，将消息分为一批批（Batch），同批次内消息以及不同批次可以并行处理。
### Spark Streaming
是以Mini-Batch模拟实现流处理，业界质疑其不是真正的流处理，笔者认为不重要，主要看适用的场景。
* 计算流程  
Spark Streaming（SS）将流式计算分为一系列短小的批处理作业，SS将输入数据按照Batch Size（如1秒）分为一段一段的数据（Discretized Stream，DStream），每一段数据都转换为Spark中的RDD，RDD操作的中间结果都会保存到内存中（默认好像是3份）。
* 容错性  
设计到RDD的容错机制，RDD的transformation是惰性的，即不会马上进行变换，而是通过lineage记录变换模式，因此RDD中任意分区（Partition）出错了，都可以通过原始数据和lineage转换操作计算得出。 
目前版本的Spark Streaming在0.5s~2s之间（storm是100ms）。  
就`吞吐量`而言，SS是Storm的五倍左右。
* SS编程  
 * 初始化  
 new一个StreamingContext
 * 输入  
 磁盘输入（HDFS）、网络流输入（Kafka、Flume、TCP Socket）
 * 转换操作  
 与RDD类似的，常用操作包括：Map、Filter、Flatmap、Join等，需要shuffle的操作有groupByKey/reduceByKey。
 * 输出  
 * 启动
### Flink
起源于德国，是Apache顶级项目。相比于Spark，特点是原生流系统。  
实际上是基于流处理机制的batch处理引擎。
### CEP
Complex event processing，流式处理的核心技术。
### Eagle
eBay开源的分布式实时安全监控方案，通过离线机器学习训练模型和实时流引擎，监控出对敏感数据的访问或恶意的操作。
## 交互式分析
定义：基于历史数据的交互式查询（Interactive Query），通常的时间跨度在数十秒到数分钟间
通常来说特点如下：
* 时延低
* 查询条件复杂
* 查询范围大
* 返回结果小
* 并发要求高
* 需要sql等接口
传统解决方案，数据库所引、内存缓存、cube（数据预聚合），接下来讨论下新的解决方案。
### MPP DB技术
MPP是系统架构的一种服务器分类方法，海量并行处理架构（Massive Parallel Processing，MPP）。目前有三大商用服务器，SMP、NUMA、MPP。
#### SMP（Symmetric Multi-Processor）
* 对称多处理器结构，是指服务器中多个CPU对称工作，无主次或从属关系。  
* 各cpu共享相同的物理内存，每个cpu访问内存中任何地址时间都是相同的，也被称为一致性存储器访问结构（Uniform Memory Access，UMA）。对SMP扩展的方式有增加内存、更快的cpu、增加cpu、扩充I/O及更多的外部设备等等。  
SMP服务器主要特征是共享，这导致SMP的扩展能力非常有限，每个共享环节都有可能造成瓶颈，最受限制的是内存。由于每个cpu必须通过相同的内存总线访问相同的内存资源，因此cpu过多会造成cpu资源浪费。  
#### NUMA（Non-Uniform Memory Access）
* 为了改善SMP扩展能力差，NUMA应运而生，可以把上百个cpu部署到一台服务器上。  
* NUMA服务器的基本特征是拥有多个CPU模块，每个模块由多个CPU组成。节点之间可以通过互联模块进行信息交换，每个cpu都可以访问整个系统的内存。  
* 缺陷是，由于访问异地内存时延远远高于本地内存，因此cpu数量增加也不能使系统性能线性增加。
#### MPP 
* 多台SMP服务器通过一定节点互联网络进行连接，每个节点之访问自己的本地资源，是一种完全无共享结构，扩展能力最强，理论上无限。目前可达512节点互联，几千个cpu。  
* 与NUMA不同，不存在异地访问问题，每个节点访问自己内容。不同节点间通过节点互联网络信息交互，这个过程称为数据重分配（Data Redistribution）  
* 但MPP服务器需要一种复杂机制调度和平衡各个节点的负载和并行。现在一般是通过系统级软件来屏蔽这种复杂性（如NCR的Teradata)
* OLAP因为有大量数据交互要选择MPP，OLTP只是大吞吐而已选择NUMA即可。
MPP架构可以分为share disk和share nothing两种：
* share disk：每个处理单元使用私有cpu和memory，共享磁盘。当磁盘接口达到瓶颈是，增加节点无用。
* share nothing：每个处理单元私有cpu、memory和磁盘。
share nothin数据同步与故障恢复是灾难，因为元数据存储在不同服务器上。
### 典型的MPP数据库
#### Greenplum架构
* 最早采用MPP架构的是Teradata数据库，整体采用sharenothin架构进行组织。早期在postgreSQL基础采用MPP架构，后期为了兼容hadoop上台，推出了HAWQ，上层分析还是原本的greenplum高性能引擎，下层存储采用HDFS。
* 因为MPP架构需要在不同处理单元之间传递信息，因此效率会比SMP差些。但当需要处理的事务达到一定规模的时候，MPP效率会比较高。这需要视通信时间占用计算时间比例而定，通信时间比较多时，MPP不占优势。
#### DB2 DPF
* IBM推出的ISAS装载的就是DB2 DPF（database partitioning feature）。每个数据独立，服务器之间通过万兆交换机交换数据，服务器内部通过share_memory实现相互访问。  
* 与greenplum类似，都是通过hash算法实现表分区，实现并行处理问题。  
### MPP调优
linux方便的东西--略。
### MPP DB适用场景
从性能来讲，MPP DB在多维复杂查询方面的性能优于Hive、Hbase、Impala等。但有两个致命缺点。
* 扩展性  
架构本身。MPP DB是基于DB扩展而来，DB中天然追求一致性，必然会带来分区容错性较差。但集群规模变大、业务数据变多时，MPP DB的元数据管理就是灾难，一旦出错难以恢复。
* 并发的支持  
MPP DB的核心原理是将一个大的查询分解成一个个子查询，分布到底层执行，最后合并结果，就是通过多线程并发来暴力扫描带来高速。但整个系统支持的并发数必然不多，最多支持50~100的并发能力。MPP DB适合小集群（100）低并发（50）场景
### SQL on Hadoop
指的是Hadoop生态里一系列支持SQL接口的组件和技术。
#### Hive
![Hive架构](https://github.com/jackychancjcjcj/The-framework-of-BDT/blob/master/hive.png)  
组件可以分为两大类：服务器端组件和客户端组件
* 服务器端组件
 * driver组件：complier、optimizer、executor，其作用是将hiveql语句进行解析、编译优化then生成计划，调用底层mapreduce计算框架。
 * metastore组件：元数据服务组件，存储hive的元数据。hive支持把metastore服务独立出来，安装到远程服务器上，使得hive更健壮。
 * thrift服务：是facebook开发的一个软件框架，用来进行可扩展且跨语言的服务开发。hive集成该服务，使不同程序语言可以调用hive接口。
* 客户端组件
 * CLI：command line interface，命令行接口。
 * thrift客户端
 * web gui：hive客户端提供了一种通过网页的方式访问hive所提供的服务。  
 ##### metastore组件
 metastore包括两部分：服务和后台数据的存储，后台存储是关系型数据库。服务是建立在后台数据存储介质之上的。在默认情况下，metastore和hive是安装在一起，也可以将metastore从hive中脱离出来，独立安装在一个集群中，这样就可以把元数据放到防火墙中，客户端可以访问hive。使用远程的metastore服务，可以让metastore服务和hive运行在不同的进程里，保证了hive稳定性和hive效率。  
 一句话描述hive：hive是基于hadoop的数据仓库工具，可以将结构化的数据文件映射成一张数据库表，提供完整的sql查询功能，将sql语句转换为mapreduce任务运行。hive类sql，最大缺点是慢，短、平、快的业务不符合hadoop批量的特性。
 #### Hbase
 hbase是分布式、面向列的开源数据库。是一个适合非结构化数据存储的数据库，而且HBase是基于列而不是基于行的。  
 ![HBase架构](https://github.com/jackychancjcjcj/The-framework-of-BDT/blob/master/hbase_framework.png)  
 HBase的核心是将数据抽象成表，表中只有rowkey和columnfamily。rowkey存储主键，columnfamily存储实际数据，仅支持单行事务，主要用来存储非结构化和半结构化的松散数据。  
 正是由于HBase这种结构，应对查询中带了主键的应用非常有效，查询返回结果非常。HBase自身的协处理器，遇到不带rowkey的查询，由协处理器通过线程并行扫描。  
 HBase不支持SQL语法。诞生了Phoenix。
 #### Impala
 Impala是实时交互SQL大数据查询工具。没有采用缓慢的Hive+Mapreduce批处理，在架构上使用了与传统并行关系数据库类似的分布式查询引擎，可以从HDFS或HBase中用select、join等查询数据，大大降低了延迟
 ### 大数据仓库
 前面讲到的MPP DB、SQL on Hadoop本质上解决的都是传统数据仓库的多维查询问题，但为什么大家不用数据仓库解决呢？  
 1. 在大数据时代，数据量大小已经超过了传统数据库能解决的量。  
 2. 成本，相比数据仓库的高性能硬件，Hadoop技术一般使用大量廉价硬件。  
 传统数据库为了追求性能，在设计的时候将存储引擎与查询引擎耦合在一起，从而扩展性不佳。大数据仓库的设计思路是对存储引擎和查询引擎分别进行扩展和优化。
 ### 本章小结
 相比传统数据仓库，大数据技术：
 * 优势：  
   支持非结构化、扩展性强、成本降低
 * 劣势：  
   小数据量时比传统MPP差/大数据时又不能满足交互式分析秒级响应的需求、对sql支持不充分。
## 批处理技术
通常的时间跨度时几分钟到数小时之间。
### 批处理技术发展
数据批处理一开始源于传统的ETL过程，通常由数据库承担，传统的数据库扩展性遇到瓶颈后，就出现了MPP技术。然后发明了Mapreduce，使得大规模扩展成为可能。Spark一开始为了替代Mapreduce。除了迭代式计算外，大规模机器学习需要另外的框架。批处理为了提高吞吐量，cpu的利用率是关键。
### MPP DB技术
MPP类数据库突破了传统数据库单点的瓶颈，扩展性得到一定的提升，在一定规模的数据量（通常是TB级的数据处理）。在数据量持续上升的情况下（PB级别以上），MPP由于架构上的限制，遇到了明显的扩展瓶颈。Hadoop的出现，解决了扩展性的问题。另外MPP的计算和存储过程是耦合的，这方面比不上MapReduce、HDFS的分离设计。分离设计的最大优点，除了MapReduce引擎下，还可以根据业务需求选择其他框架。从目前的应用来说，一份数据选择多个引擎以应对多个业务的必然选择。
### MapReduce编程框架
#### MapReduce简介
起源于Google的几篇论文，可以把MapReduce理解为：把一堆杂乱无章的数据按照某种特征归纳起来，然后处理并得到最后的结果。Map面对的杂乱无章的、互不相关的数据，它解析每天数据，从中提取出Key和Value，也就是提取数据的特征。经过MapReduce的Shuffle阶段之后，在Reduce阶段看到的已经是归纳好的数据，在这基础下可以做进一步处理已得到结果。
#### MapReduce原理
MapReduce是一种云计算的核心计算模式，是一种分布式运算技术，主要用于大规模并行程序并行问题。  
MapReduce模式的主要思想是自动将一个大的计算（如程序）拆解成Map（映射）和Reduce（化简）的方式。  
数据被分割后，通过Map函数将数据映射成不同的区块，分配给计算机集群进行处理，以达到分布式运算的效果，再通过Reduce函数将结果汇整，从而输出开发者所需的结果。  
MapReduce借鉴了函数式程序设计语言的设计思想，其软件实现是指定一个Map函数，把键值对（key/value）映射成新的键值对，形成一系列中间结果形式的键值对，然后把他们传递给Reduce（规约）函数，把具有相同中间形式key的value合并在一起，map和reduce函数具有一定的关联性。  
MapReduce致力于解决大数据处理问题，在处理时，每个节点就近读取存储在本地的数据处理（map），将处理后的数据进行合并（combine）、排序(shuffle and sort)后再分发至（reduce节点），从而避免了大数据传输，降低效率。
#### shuffle
shuffle过程是mapreduce核心所在，shuffle的大致范围就是怎么样把maptask的输出结果有效地传送到reduce端。  
在hadoop里，大部分maptask和reducetask的执行在不同的节点上。当然，很多情况下reduce在执行时需要跨节点去拉去其他节点的maptask结果。  
