#### kafka
基于发布订阅的消息系统。

#### 一对多模式：一个生产者生产的信息被多人订阅。关键：topic
1.通信过程：

生产者(Producer)制造消息发给Kafka的一个特定主题(Topic)，订阅这个主题的消费者(Consumer)获得Topic的消息。

2.应用场景：

1. 某个go服务(生产者)生成了很多日志，把日志发往一个名为"go日志"的主题，有一些消费者如elasticseach订阅了这个主题，相当于elasticseach获得了这些日志。

3.kafka组成成分及其功能

- broker：接收生产者发来的消息并存到磁盘。
- 消费者群组：包含了多个消费者。
- partition: 主题的分区，消息的实际承载体。
- replication-factor: 主题下partition的副本个数。

4.kafka重要配置
broker端配置
- broker.id: kafka节点在集群中的标志。
- port: kafka启动端口
- zookeeper.connect: localhost:2181
- hostname: zookeeper所在机器的ip
- log.dirs: 指定kafka处理消息片段的存放点
- num.recovery.threads.per.data.dir: kafka启动的线程个数去处理日志片段。

主题端配置
... https://cloud.tencent.com/developer/article/1547380

5.kafka运维
```
# 创建主题
./kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 -partitions 1 --topic looklook-log   // replication-factor表topic的副本个数，partitions表

```

#### 一对一模式。关键：队列(queue)
1.通信过程

生产者生产消息 -> kafka(queue) -> 消费者