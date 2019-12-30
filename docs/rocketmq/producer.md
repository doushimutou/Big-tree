# Producer&NameServer

## DefaultMQProducer

**消息发动步骤：**

1. 设置producer的GroupName;
2. 这种instanceName,当一个jvm需要启动多个producer的时候，需要设置instanceName来区分；
3. 设置发送失败重试的次数
4. 设置NameServer的地址
5. 组装消息并发送

## 延迟消息

延时消息的使用是`setDelayTimelevel(int level)` 设置延迟时间，然后把消息发送出去。

## 自定义发送规则

- 一个Topic有多个Message queue，使用producer的默认规则是轮流向各个message queue发送消息。consumer在消费的时候根据负载均衡策略，消费被分配到的Message queue。

- 如果需要把消息发送到指定的message queue ，可以设置Message queueSelector.

## 存储Offset的队列位置信息

### offset存储机制：

Offset：一种类型的消息会放到一个topic里面，为了能够并行，一个topic会有多个message queue ，offset是指某个topic下的一条消息在某个message  queue的位置，通过这个offset来定位这条消息。或则让consumer从这条消息后开始处理。

>  **DefaultMQPushConsumer:**

对于常用的DefaultMQPushConsumer来说，默认clustering 模式，由Broker存储和控制offset的值，使用remoterBrokerOffsetStore结构。

DefaultMQPushConsumer，BroadCasting模式，使用localFileOffsetStore模式，把offset存本地。

> **DefaultMQPullConsumer:**

使用DefaultMQPullConsumer来说，就要自己维护offset，把offset写到内存中，没有持久化，实际不推荐。

### 设置offsetfrom：

DefaultMQPushConsumer.setConsumerFromWhere()，设置从哪儿开始消费。ConsumerFromWhere.CONSUME_FROM_FIRST_OFFSET，从最小的offset开始读取。但是如果从开始就到有用的消息之间由很大的距离，就不适合。可以通过ConsumerFromWhere.CONSUME_FROM_TIMESTAMP，时间戳是精确到秒的。



## NameServer

​	 对于一个消息队列集群来说，系统由很多台机器，每台机器的IP、角色不同，又是动态变动的。每增加一个Producer或则Consumer,如何配置链接信息？NameServer的作用就是维护这些配置信息，状态信息，其他角色都通过nameServer系统执行。

​    为什么RocketMQ要重新造轮子，而不用开源的注册中心比如：zookeeper  ,主要是RocketMQ不需要自动master选举，不需要复杂的功能，一个轻量的元数据服务就足够了；减少中间件的依赖，减少维护成本。

​	具体NameServer如何维护状态数据，以及底层的通信机制本文不做讨论。