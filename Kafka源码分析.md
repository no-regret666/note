# Kafka源码分析

Kafka是一个开源的、分布式的事件流处理平台。

Kafka采用“Producer -> Server -> Consumer”的业务模型。

1. Producer：生产者，事件源通过该组件上报事件信息；
2. Server：对上报来的事件数据作持久化存储，并通过精心设计的机制保证高吞吐量；
3. Consumer：分责从Server端实时拉取事件数据，以执行相应的业务处理。

### 架构

1. ##### 基本框架

   ![img](https://pica.zhimg.com/v2-a11a02ed897165b55605b653261a1500_1440w.jpeg)

2. ##### Topic

   一类事件流分配一个主题(Topic)。Producer可将事件发到某个Topic下，Consumer可以订阅其感兴趣的Topic，从而可以处理对应的事件流。

3. ##### Partition 和 Consumer Group

   为了提高吞吐量，增加并行度。Kafka将一个Topic下的数据横向分成了多个“分区”，而每个Partition只允许一个Consumer来消费。

   引入了ConsumerGroup的概念，即将订阅同一个Topic的多个Consumer打成“组”，然后将Topic内的Partition通过一定的算法分配给组内的Consumer。

4. ##### Replica

   为了提高可用性，防止单点故障，Kafka引入了Replica的概念：

   1. 一个Partition的数据会同时保存N份，即N个Replica；
   2. Replica之间有"主从"之分，Producer将数据写入主Replica中，从Replica异步到主Replica拉数据以实现同步；
   3. Producer在产生数据时可以指定acks参数，表示本次写入需要有多少个从Replica完成同步才视为成功；
   4. 当主Replica损坏或宕机时，其中一个从Replica会被选举为主Replica；

5. ##### Broker & KafkaController

   Kafka Server端有多个节点组成，每个节点都有一个名字叫Broker。其中一个Broker会被选举称为KafkaController，用于监测所有Broker的状态，发现故障后启动故障转移过程。



## 生产者

### 线程模型

<img src="https://pic4.zhimg.com/v2-69951ed2f5d1eb64a0e696d2b2d198cf_1440w.jpg" alt="img" style="zoom:67%;" />

这里主要存在两个线程：主线程 和 Sender线程。主线程即调用KafkaProducer.send方法的线程。当send方法被调用时，消息并没有真正被发送，而是暂存到RecordAccumulator。Sender线程在满足一定条件后，会去RecordAccumulator中取消息并发送到Kafka Server端。

暂存的好处：

1. 可以将多条消息通过一个ProducerRequest批量发送出去；
2. 提高数据压缩效率（一般压缩算法都是数据量越大越能接近预期的压缩效果）