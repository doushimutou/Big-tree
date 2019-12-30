# RocketMQ-Consumer

RocketMQ两种方式，一种是DefaultMQPushConsumer，另一种是DefaultMQPullConsumer;本文将仔细剖析这两种方式的区别。

## DefaultMQPushConsumer

### 创建一个pushConsumer

废话少说直接上示例代码：

```
public class Consumer {

	public static void main(String[] args) throws MQClientException {

		DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("first_consumer");
		consumer.setNamesrvAddr("118.25.127.220:9876");
		/**
		 * 默认为CLUSTERING
		 */
		consumer.setMessageModel(MessageModel.CLUSTERING);

		/**
		 * 消费的起始位置
		 */
		consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
		//设置消费的toppic，tag
		consumer.subscribe("TopicDemo1", "*");

		consumer.registerMessageListener(new MessageListenerConcurrently() {
			public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
				System.out.println(Thread.currentThread().getName() + " Receive messages :" + msgs);
				return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
			}
		});
		consumer.start();
		System.out.println("========= Consumer start =========");
	}
}
```

> DefauleMQPushConsumer，需要设置三个参数：

#### GroupName:

-  用于把过个Consumer组织到一起，提高并发能力

  - GroupName 需要和MessageModel配合使用.

  **Clustering模式：**

  同一个consumerGroup中，每个consumer只能订阅消息的一部分内容，同一个consumerGroup里的所有consumer消费的内容合起来才是所订阅的Topic内容的整体，以达到负载均衡的目的。

  **Broadcasting模式：**

  同一个consumerGroup里面的每个consumer能订阅到所有消息。一个消息被多次分发，被多个consumer消费。

#### NameServer:

地址+端口号，可以写多个

#### Topic:

标识消息类型，需要提前创建。topic下还可以使用tag来过滤消息。

**举个栗子**：

生产环境部署了n个consumer，n个producer；发送了100条消息；采用clustering（集群）模式，就是这n个consumer一起来消费这100条消息；采用Broadcasting（广播）模式，就是每个consumer都可以拿到完整的100条消息。



### DefaultMQPushConsumer处理：

DefaultMQPushConsumer 是通过`pullRequest` 长轮询来实现Push的效果。

- push方式：

  server接到消息后，主要把消息推给Client端，实时性高。但是队列服务的Server，push会加大server性能开销。

- pull方式

  从server拉取消息，主动权在client中；主要问题在于消息间隔时间不好设定。



## DefaultMQPullConsumer

老规矩废话少说，示例代码：

```

public class PullConsumer {
	private static  final Map<MessageQueue ,Long> offsetTable = new HashMap<MessageQueue, Long>();
	public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
		//new pullConsumer
		DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("pull_consumer");
		consumer.setNamesrvAddr("118.25.127.220:9876");
		consumer.start();
		//获取所有消息，遍历
		Set<MessageQueue> mqs = consumer.fetchSubscribeMessageQueues("TopicDemo1");
		for (MessageQueue mq: mqs ) {
			PullResultExt pullResult = (PullResultExt) consumer.pullBlockIfNotFound(mq, null,
					getMessageQueueOffset(mq), 32);
			putMessageQueueOffset(mq, pullResult.getNextBeginOffset());
			switch (pullResult.getPullStatus()) {

				case FOUND:

					List<MessageExt> messageExtList = pullResult.getMsgFoundList();
					for (MessageExt m : messageExtList) {
						System.out.println("收到了消息:" + new String(m.getBody()));
					}
					break;

				case NO_MATCHED_MSG:
					break;

				case NO_NEW_MSG:
					break;

				case OFFSET_ILLEGAL:
					break;
				default:
					break;
			}
		}
	}
	private static void putMessageQueueOffset(MessageQueue mq, long offset) {
		offsetTable.put(mq, offset);
	}

	private static long getMessageQueueOffset(MessageQueue mq) {
		Long offset = offsetTable.get(mq);
		if (offset != null)
			return offset;
		return 0;
	}
}

```

### 过程解析：

- 从topic获取到全量的message queue后遍历
- 维护一个offsetstroe，用于记录拉取消息的节点位置
- 根据不同的消息状态做不同的处理



## consumer重启和关闭

pullconsumer，使用者本身主动权很高，做好offset的记录保存，可以根据需要重启和关闭消费过程。

pushConsumer，退出要调用shutdown（）函数，释放资源，保存offset。

