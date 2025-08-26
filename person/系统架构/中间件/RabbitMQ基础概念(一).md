# RabbitMQ基础概念

> 来源：<https://www.yuque.com/wyg0405/rabbitmq/rabbitmq-getstart>

## RabbitMQ 基础

![RabbitMQ基础概念.RabbitMQ基本架构](figures/RabbitMQ基础概念.RabbitMQ基本架构.png)

## 1. RabbitMQ 特点

RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现。
AMQP ：Advanced Message Queue，高级消息队列协议。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。
RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。具体特点包括：

1. 可靠性（Reliability） RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认。
2. 消息集群（Clustering） 多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。
3. 高可用（Highly Available Queues） 队·1列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。
4. 多种协议（Multi-protocol） RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。
5. 多语言客户端（Many Clients） RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。
6. 管理界面（Management UI） RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。
7. 跟踪机制（Tracing） 如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。
8. 插件机制（Plugin System） RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

## 2. RabbitMQ 中的概念模型

灵活的路由（Flexible Routing） 在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange

### 2.1. 消息模型

所有 MQ 产品从模型抽象上来说都是一样的过程： 消费者（consumer）订阅某个队列。生产者（producer）创建消息，然后发布到队列（queue）中，最后将消息发送到监听的消费者。

![RabbitMQ基础概念.RabbitMQ基本消息模型](figures/RabbitMQ基础概念.RabbitMQ基本消息模型.png)

#### 2.1.1. RabbitMQ 基本概念

上面只是最简单抽象的描述，具体到 RabbitMQ 则有更详细的概念需要解释。上面介绍过 RabbitMQ 是 AMQP 协议的一个开源实现，所以其内部实际上也是 AMQP 中的基本概念：

RabbitMQ 内部结构

![RabbitMQ基础概念.RabbitMQ内部结构](figures/RabbitMQ基础概念.RabbitMQ内部结构.png)

1. Brooker 表示消息队列服务器实体。
2. Message 消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。
3. Virtual Host 虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。
4. Publisher 消息的生产者，也是一个向交换器发布消息的客户端应用程序。
5. Exchange 交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
6. Binding 绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
7. Queue 消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
8. Connection 网络连接，比如一个TCP连接。
9. Channel 信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。
10. Consumer 消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

#### 2.1.2. AMQP 中的消息路由

AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了 Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

#### 2.1.3. Exchange 类型

![RabbitMQ基础概念.RabbitMQExchange类型](figures/RabbitMQ基础概念.RabbitMQExchange类型.png)

Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers 。headers 匹配 AMQP 消息的 header 而不是路由键，此外 headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：

##### 2.1.3.1. direct

![RabbitMQ基础概念.RabbitMQExchange类型之direct.png](figures/RabbitMQ基础概念.RabbitMQExchange类型之direct.png)

消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、单播的模式。

##### 2.1.3.2. fanout

![RabbitMQ基础概念.RabbitMQExchange类型之fanout.png](figures/RabbitMQ基础概念.RabbitMQExchange类型之fanout.png)

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。

##### 2.1.3.3. topic

![RabbitMQ基础概念.RabbitMQExchange类型之topic.png](figures/RabbitMQ基础概念.RabbitMQExchange类型之topic.png)

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。
它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。
它同样也会识别两个通配符：符号`#`和符号`*`。`#`匹配0个或多个单词，`*`匹配不多不少一个单词。
