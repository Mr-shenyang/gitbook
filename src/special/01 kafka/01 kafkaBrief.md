# 1 概念
Apache Kafka是 一个分布式流处理平台.其具备如下特性：
* 快速持久化：可以在 O(1) 的系统开销下进行消息持久化
* 高吞吐：在一台普通的服务器上既可以达到10W/s的吞吐速率
* 完全的分布式系统：Broker、Producer和Consumer都原生自动支持分布式，自动实现负载均衡
* 零拷贝技术(zero-copy)：减少IO操作步骤，提高系统吞吐量
* 支持同步和异步复制两种高可用机制
* 丰富的消息拉取模型，支持数据批量发送和拉取
* 数据迁移、扩容对用户透明
* 无需停机即可扩展机器
* 高效订阅者水平扩展、实时的消息订阅、亿级的消息堆积能力、定期删除机制

**名词解释**
|名词|释义|
|---|---|
|Topic|主题，逻辑概念上的队列|
|Producer|消息的生产方，生产端|
|Consumer|消息的消费方，消费端|
|Consumer Group|Kafka是按消费组来消费消息，一个消费组下面的所有机器可以组成一个Consumer Group，每条消息只能被该Consumer Group一个Consumer消费，不同Consumer Group可以消费同一条消息。|
|Broke|存储实际消息的地方，一台服务器|
|集群|多个Broker，组成的一个集群|
|Partition|Kafka的Broker端支持消息分区,Producer可以决定把消息发到哪个Partition,Partition是Kafka高吞吐量的重要保证；|

![brief](/images/framework/mafka/brief.png)
