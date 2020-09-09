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
