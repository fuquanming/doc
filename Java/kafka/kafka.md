# 一、kafka概要

## 1、应用场景

1）活动追踪：跟踪网站用户与前端应用程序发生的交互，如：**网站PV、UV分析**

2）传递消息：系统间异步的消息交互，如：**营销活动（支持后发送券码福利）**

3）日志收集：收集系统及应用程序的度量指标及日志，如：**应用监控和告警**

4）提交日志：将数据库的更新发布到kafka上，如：**交易统计**

## 2、特点

kafka是一种分布式的，基于发布/订阅的消息系统

**集群模式**：动态增减节点，伸缩性

**多个生产者**

**多个消费者**

消息 持久化：基于磁盘的数据存储

通过横向扩展生产者，消费者和broker，kafka可以轻松处理巨大的消息流，具有很好的性能和吞吐量

## 3、核心概念

**Broker（代理）：**消息中间件处理节点，一个kafka节点就是一个broker，一个或多个Broker可以组成一个kafka集群

**Topic（主题）：**kafka根据topic对消息进行归类，发布到kafka集群的每条消息都需要指定一个topic

**Partition（分区）：**物理上的概念，一个topic可以分为多个partition，每个partition内部是有序的

**Replica（副本）：**副本，每个partition有多个副本，存储在不同broker上，保证消息的高可用

**Segment（片段）：**partition物理上由多个segment组成，每个segment存着message信息

**Message（消息）：**消息，是级别的通信单位，有一个key，一个value和时间戳构成

**Producer（生产者）：**消息生产者，向Broker发送消息的客户端

**Consumer（消费者）：**消息消费者，从Broker读取消息的客户端

**ConsumerGroup（消费者组）：**每个Consumer属于一个特定的Consumer Group，一条消息可以发送到多个不同的Consumer Group，但是一个Consumer Group中只能有一个Consumer能够消费该消息

## 4、发展

一站式的流处理平台

- 消息系统：kafka consumer & producer （生产者和消费者）
- 存储系统
- ETL工具：kafka connector（连接DB等外部数据）
- 流式计算：kafka streams

# 二、kafka生产者

## 1、生产者概述

1）kafka生产者会将消息封装成一个 ProducerRecord 向 kafka集群中的某个 topic 发送消息

2）发送的消息首先会经过序列化器进行序列化，以便在网络中传输

3）发送的消息需要经过分区器来决定该消息会分发到 topic 对应的 partition，当然如果指定了分区，那么就不需要分区器了。

4）这个时候消息离开生产者开始往kafka集群指定的 topic 和 partition 发送

5）如果写入成功，kafka集群会回应 生产者一个 RecordMetaData 的消息，如果失败会根据配置的允许失败次数进行重试，如果还是失败，那么消息写入失败，并告诉生产者。

## 2、创建kafka生产者

~~~java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");// broker地址清单
props.put("key,serializer", "org.apache.kafka.common.serialization.StringSerializer");// key序列化
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");// value序列化
Producer<String, String> producer = new KafkaProducer<>(props);
~~~

## 3、发送消息到broker

~~~java
ProductRecord<String, String> record = new RroductRecord<>("topic", "key", "value");// 多种构造方式
proucer.send(record);// 发送并忘记（fire-and-forget）
RecordMetadata rm = producer.send(record).get();// 同步发送
producer.send(record, new Callback() {// 异步发送
    public void onCompletion(RecordMetadata rm, Execption e)  {
        
    }
});
~~~

## 4、序列化器

1）内置序列化器（int/long/float/double/byte/string）

2）自定义序列化器（实现org.apache.kafka.common.serialization.Serializer接口）

3）使用Avro序列化：需要引入第三方类库

![1564914169256](image\2.4.png)

## 5、分区

1）创建消息时，指定分区

2）使用默认的分区器：DefaultPartitioner

key存在：hash（key）% numPartition

key不存在：采用round-robin算法，每个分区机会均等

3）自定义分区器

# 三、kafka消费者

## 1、基本概念

1）一个主题可以被多个组消费

2）一个主题的一个分区只能被一个组中的一个消费者消费

3）主题可以被一个组反复消费，只要消息没有被删除

4）在均衡：分区的所有权从一个消费者转移到另一个消费者

## 2、创建kafka消费者

~~~java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");// broker地址清单
props.put("key,serializer", "org.apache.kafka.common.serialization.StringDeserializer");// key反序列化
props.put("value.serializer", "org.apache.kafka.common.serialization.StringDeserializer");// value反序列化
props.put("group.id", "fcc-group"); // * 设置所属消费者群组
Consumer<String, String> consumer = new KafkaConsumer<>(props);
~~~

## 3、订阅主题

一个消费者可以同时订阅多个主题

~~~java
consumer.subscribe(Collections.singletonList("fcc_topic"));
consumer.subscribe("fcc.*");
~~~

## 4、轮询

~~~java
try {
    while (true) {// 无限循环
        ConsumerRecords<String, String> records = consumer.poll(100);// 轮询消息
        for (ConsumerRecord<String, String> r : records) {// 逐条处理
            System.out.println("offset=%d,key=%s,value=%s%n", r.offset(), r.key(), r.value());
        }
    }
} finally {
    consumer.close();// 关闭消费者
}
// poll：加入群组，接受分配到分区，轮询获取数据，发送心跳，自动提交
// close：关闭连接，主动通知GroupCorrdinator,自己已挂
~~~

## 5、提交

偏移量：offset，消息在分区中的位置

提交：commit，更新分区当前位置，避免重复消费

提交方式：1）自动提交；2）手动提交（同步VS异步提交）

## 6、反序列化器

1）内置反序列化器（int/long/float/double/byte/string）

2）自定义反序列化器（实现org.apache.kafka.common.serialization.Deserializer接口）

3）使用Avro反序列化：需要引入第三方类库

# 四、深入kafka

## 1、集群成员关系



## 2、控制器

1）控制器的作用：除具有一般broker的功能之外，还负责分区首领的选举

2）控制器的选举：各broker先ZK中/controller注册临时节点

3）分区首领的选举：broker加入时，同步副本；broker离开时，选举新的分区首领

## 3、分区复制

1）首领副本：每个分区都有一个首领副本。**首领副本负责处理所有生产者和消费者的请求**

2）跟随者副本：首领副本以外的副本都是跟随者副本。**跟随者副本不能处理来只客户端的其你去，它们唯一的任务就是从首领哪里复制消息，保存与首领一直的状态，在首领挂了后顶替首领。**

3）AR：Assigned Replicas，所有副本

4）ISR：In-Sync Replicas，已同步副本

5）OSR：Outof-Sync Replicas，掉队副本

6）AR=ISR+OSR

Kafka Manager

## 4、物理存储

1）Kafka会将数据持久化到文件，文件目录以分区来组织，每个分区下有多个文件，每个文件称作一个片段，每个片段包含1G或一周的数据，以较小的值为准

2）保留策略：数据被删除之前可以保留多次实践，或者清理数据之前可以保留的数据量大小

3）3类文件：数据文件、索引文件、时间索引文件

## 5、参数调优

http://kafka.apache.org/documentation/#producerconfigs

| 配置项           | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| acks             | acks=0：不会等待来只任何服务器的响应；1：等待来只集群首领节点的响应；all：等待所有参与复制的节点的响应 |
| retries          | 遇到发送失败，重试的次数                                     |
| max.request.size | 单个消息或单个批次大小的上限，最好与broker中的配置{message.max.bytes}一致，避免消息被拒接 |









































































































