# RocketMQ支持的消息队列类型及对应用途

## 一、RocketMQ核心队列模型

### 1. 普通消息队列
**基本特性**：
- 遵循发布-订阅模型
- 消息无序消费
- 自动负载均衡

**队列分配公式**：
$$
\text{QueueID} = \text{hash(messageKey)} \% \text{QueueNum}
$$

**典型用途**：
```java
// 订单创建消息发送
Message msg = new Message(
    "OrderTopic", 
    "CREATE",  // Tag过滤使用
    orderId.toString(),
    order.toString().getBytes()
);
SendResult result = producer.send(msg);
```
- 电商订单处理
- 用户行为日志收集
- 系统通知推送

### 2. 顺序消息队列
**保证机制**：
- 相同ShardingKey的消息写入同一队列
- 单队列单线程消费

**顺序性公式**：
$$
\forall m_1, m_2 \in Q, \text{if } m_1.\text{time} < m_2.\text{time} \Rightarrow C(m_1) \rightarrow C(m_2)
$$

**典型用途**：
```java
// 订单状态变更顺序消息
Message msg = new Message(
    "OrderStatusTopic",
    "UPDATE",
    order.getOrderId(),  // 使用订单ID作为ShardingKey
    order.toString().getBytes()
);
producer.send(msg, new MessageQueueSelector() {
    @Override
    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
        String orderId = (String) arg;
        int index = Math.abs(orderId.hashCode()) % mqs.size();
        return mqs.get(index);
    }
}, order.getOrderId());
```
- 订单状态变更流水
- 库存扣减操作
- 金融交易流水

### 3. 延迟消息队列
**延迟级别**：

| 级别 | 延迟时间 | 级别 | 延迟时间 |
|------|----------|------|----------|
| 1    | 1s       | 10   | 1m       |
| 2    | 5s       | 11   | 2m       |
| ...  | ...      | 18   | 2h       |

**实现原理**：
```
消息写入 → SCHEDULE_TOPIC_XX → 定时触发 → 目标Topic
```

**典型用途**：
```java
// 订单超时关闭
Message msg = new Message(
    "OrderCloseTopic",
    order.toString().getBytes()
);
msg.setDelayTimeLevel(12);  // 对应30分钟延迟
producer.send(msg);
```
- 订单超时关闭
- 定时任务触发
- 预约提醒通知

### 4. 事务消息队列
**两阶段提交流程**：
1. 发送半消息(Half Message)
2. 执行本地事务
3. 提交/回滚消息

**状态转换**：
$$
\text{半消息} \xrightarrow{\text{LocalTransSuccess}} \text{可消费消息} \\
\text{半消息} \xrightarrow{\text{LocalTransFail}} \text{丢弃}
$$

**典型用途**：
```java
// 分布式事务示例
TransactionSendResult result = producer.sendMessageInTransaction(
    new Message("PaymentTopic", payment.toString().getBytes()), 
    null
);

// 本地事务执行器
public class PaymentLocalTrans implements LocalTransactionExecuter {
    @Override
    public LocalTransactionState executeLocalTransactionBranch(Message msg, Object arg) {
        try {
            paymentService.process(payment);
            return LocalTransactionState.COMMIT_MESSAGE;
        } catch (Exception e) {
            return LocalTransactionState.ROLLBACK_MESSAGE;
        }
    }
}
```
- 跨系统资金转账
- 订单支付结算
- 需要最终一致性的业务

## 二、RocketMQ队列高级特性

### 1. 消息过滤队列
**过滤类型**：

| 类型       | 语法示例                  | 性能影响 |
|------------|---------------------------|----------|
| TAG过滤    | `TagA \|\| TagB`            | 低       |
| SQL92过滤  | `a > 5 AND b like '%abc%'` | 中       |

**典型用途**：
```java
// 消费者订阅带过滤条件的消息
consumer.subscribe("LogTopic", "ERROR || WARN");

// SQL过滤(需要Broker开启enablePropertyFilter=true)
Message msg = new Message("OrderTopic");
msg.putUserProperty("region", "EAST");
msg.putUserProperty("amount", "1000");
producer.send(msg);

// 消费者订阅
consumer.subscribe("OrderTopic", MessageSelector.bySql("region='EAST' AND amount>500"));
```
- 区域化消息处理
- 日志级别过滤
- 差异化业务处理

### 2. 重试队列与死信队列
**消息重试机制**：
$$
\text{重试次数} = 
\begin{cases} 
0 & \text{首次消费} \\
1 \sim \text{maxReconsumeTimes} & \text{渐进式重试} \\
\text{超过maxReconsumeTimes} & \text{进入死信队列}
\end{cases}
$$

**配置参数**：
```properties
# broker.conf
maxReconsumeTimes=16
```

**典型用途**：
```java
// 消费者处理逻辑
consumer.registerMessageListener(new MessageListenerConcurrently() {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
        ConsumeConcurrentlyContext context) {
        try {
            processMessage(msgs);
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        } catch (Exception e) {
            return ConsumeConcurrentlyStatus.RECONSUME_LATER;
        }
    }
});
```
- 支付结果异步通知
- 外部系统对接
- 容错性要求高的场景

### 3. 批量消息队列
**批量发送公式**：
$$
\text{吞吐量提升} = \frac{T_{\text{single}} \times n}{T_{\text{batch}}}
$$

**典型用途**：
```java
List<Message> messages = new ArrayList<>();
for(Order order : orders) {
    messages.add(new Message(
        "BatchOrderTopic",
        order.toString().getBytes()
    ));
}

// 批量发送
SendResult result = producer.send(messages);
```
- 日志批量上传
- 数据同步作业
- 大批量订单创建

## 三、RocketMQ集群队列模式

### 1. 集群消费模式
**特点**：
- 同GroupID的消费者均摊消息
- 自动负载均衡
- 消费进度共享

**分配算法**：
$$
C_i \rightarrow Q_j \quad \text{where} \quad j = i \% N
$$

**典型用途**：
```java
// 订单处理集群
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("OrderProcessGroup");
consumer.subscribe("OrderTopic", "*");
consumer.registerMessageListener(new OrderMessageListener());
consumer.start();
```
- 分布式任务处理
- 水平扩展的消费者
- 无状态服务

### 2. 广播消费模式
**特点**：
- 同GroupID的消费者都收到全量消息
- 消费进度独立维护
- 无重复消费保护

**消息流**：
```
Topic → Queue1 → Consumer1
      → Queue2 → Consumer1
      → Queue1 → Consumer2
      → Queue2 → Consumer2
```

**典型用途**：
```java
// 配置信息同步
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ConfigSyncGroup");
consumer.setMessageModel(MessageModel.BROADCASTING);
consumer.subscribe("ConfigTopic", "*");
consumer.registerMessageListener(new ConfigUpdateListener());
consumer.start();
```
- 全局配置更新
- 缓存刷新通知
- 需要全量数据的场景

## 四、行业应用案例

### 1. 电商交易系统
**架构**：
```
[订单服务] → OrderTopic → [支付服务]
                   → [库存服务]
                   → [物流服务]
```

**队列设计**：

| 队列类型       | Topic          | 用途                     |
|----------------|----------------|--------------------------|
| 顺序消息       | OrderStatus    | 订单状态变更流水         |
| 事务消息       | Payment        | 支付与订单状态同步       |
| 延迟消息       | OrderCancel    | 30分钟未支付自动取消     |
| 普通消息       | OrderLog       | 订单操作日志记录         |

### 2. 金融支付系统
**关键配置**：

```properties
# 事务消息检查间隔
transactionCheckInterval=60000
# 最大检查次数
transactionCheckTimes=15
# 消息追踪开启
traceTopicEnable=true
```

**消息轨迹**：
```java
// 开启消息轨迹
DefaultMQProducer producer = new DefaultMQProducer("PaymentProducer");
producer.setTraceDispatcher(traceDispatcher);
producer.start();
```

### 3. 物流跟踪系统
**混合消费模式**：
```java
// 实时位置更新(集群消费)
DefaultMQPushConsumer locationConsumer = new DefaultMQPushConsumer("LocationGroup");
locationConsumer.subscribe("LocationTopic", "*");

// 运单状态广播(广播消费)
DefaultMQPushConsumer waybillConsumer = new DefaultMQPushConsumer("WaybillGroup");
waybillConsumer.setMessageModel(MessageModel.BROADCASTING);
waybillConsumer.subscribe("WaybillTopic", "*");
```

## 五、性能优化指南

### 1. 队列数量配置公式
$$
\text{最优队列数} = \max(\text{生产者组数}, \text{消费者组数}) \times \text{扩容系数}
$$
其中扩容系数建议1.5~2.0

### 2. 消费并行度计算
$$
\text{并行度} = \min(\text{QueueNum}, \text{ConsumerThreadNum})
$$

**配置示例**：
```java
consumer.setConsumeThreadMin(20);
consumer.setConsumeThreadMax(64);
```

### 3. 消息大小建议
| 消息类型       | 推荐大小       | 性能影响           |
|----------------|----------------|--------------------|
| 普通消息       | <4KB           | 最优               |
| 批量消息       | <128KB         | 网络传输效率高     |
| 超大消息       | 使用OSS存储引用 | 避免Broker压力过大 |

### 4. 监控关键指标
| 指标                | 计算公式                     | 健康阈值          |
|---------------------|----------------------------|-------------------|
| 消息堆积量          | `TOPIC.queue.maxOffset - Consumer.offset` | <1000             |
| 消费TPS             | $\Delta \text{offset}/\Delta t$         | >500(视业务而定)  |
| 发送耗时            | `sendMessageTimeCost`       | P99 < 500ms       |
| 事务消息成功率      | $\frac{\text{commit}}{\text{commit}+\text{rollback}}$ | >0.99             |

RocketMQ的队列类型和特性能够满足从简单到复杂的各种消息场景需求，建议根据业务特点选择适当的队列模型，并通过合理的参数配置和监控确保系统稳定运行。对于关键业务系统，建议启用消息轨迹和事务消息等高级特性保障数据可靠性。