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

    
