#### kafka
基于发布订阅的消息系统。

1.通信过程：

生产者(Producer)制造消息发给Kafka的一个特定主题(Topic)，订阅这个主题的消费者(Consumer)获得Topic的消息。

2.应用场景：

1. 某个go服务(生产者)生成了很多日志，把日志发往一个名为"go日志"的主题，有一些消费者如elasticseach订阅了这个主题，相当于elasticseach获得了这些日志。