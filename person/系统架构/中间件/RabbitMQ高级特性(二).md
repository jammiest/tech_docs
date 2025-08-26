# RabbitMQ高级特性(二)

> 来源：<https://www.yuque.com/wyg0405/rabbitmq/ifghq5>

## 生产者Confirm消息确认机制

### 什么是消息确认

消息确认，是指生产者投递消息后，如果Broker收到消息，则会给我们生产者一个应答。生产者进行接受应答，用来确认消息这条是否正常发送到Broker，这种方式也是消息的可靠性投递的核心保障！

![RabbitMQ高级特性(二).生产者Confirm消息确认机制](figures/RabbitMQ高级特性(二).生产端Confirm消息确认机制.png)

### 如何实现Confirm确认消息

1. 在channel上开启确认模式：channel.confirmSelect();
2. 在channel上添加监听: addConfirmListener,监听成功和失败返回的结果，根据结果对消息重发送或记录日志等后续处理。

### java代码实现

**生产者**

```java
package com.wyg.rabbitmq.javaclient.confirm;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConfirmListener;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * confirm消息确认机制
 */
public class Producer {
    private static final String HOST = "localhost";
    private static final int PORT = 5672;
    private static final String USERNAME = "guest";
    private static final String PASSWORD = "guest";
    private static final String EXCHANGE = "test_confirm_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setVirtualHost("/");
        factory.setPort(PORT);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE, "topic");

        // 开启confirm确认模式
        channel.confirmSelect();

        // 添加监听
        channel.addConfirmListener(new ConfirmListener() {
            @Override
            public void handleAck(long deliveryTag, boolean multiple) throws IOException {
                System.out.println("---消息投递成功，deliveryTag:" + deliveryTag);
            }

            @Override
            public void handleNack(long deliveryTag, boolean multiple) throws IOException {
                System.out.println("---消息投递失败，deliveryTag:" + deliveryTag);
            }
        });

        String msg1 = "这是一条confirm确认消息save";
        channel.basicPublish(EXCHANGE, "confirm.save", null, msg1.getBytes("UTF-8"));
        String msg2 = "这是一条confirm确认消息abc";
        channel.basicPublish(EXCHANGE, "confirm.abc", null, msg2.getBytes("UTF-8"));
    }

    // 注意，因为要等待broker的confirm消息，暂时不关闭channel和connection
}
```

运行结果

![RabbitMQ高级特性(二).生产端Confirm消息确认机制-运行结果.png](figures/RabbitMQ高级特性(二).生产端Confirm消息确认机制-运行结果.png)

注意

broker在接受消息成功后，ConfirmListener进入handleAck；失败进入handleNack，失败原因有很多，比如MQ异常，queue容量达到上限，磁盘写满了等。  
还有极端情况，ack和nack都没获取到，比如网络闪断，这时候需要用定时任务去抓取做后续处理。

**消费者**

```java
package com.wyg.rabbitmq.javaclient.confirm;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.*;

/**
 * 消息确认
 */

public class Consumer {

    private static final String HOST = "localhost";
    private static final int PORT = 5672;
    private static final String USERNAME = "guest";
    private static final String PASSWORD = "guest";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setVirtualHost("/");
        factory.setPort(PORT);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();

        String queueName = "test_confirm_queue";
        String routingKey = "confirm.#";
        String exchangeName = "test_confirm_exchange";
        // 申明队列
        channel.queueDeclare(queueName, true, false, false, null);
        // 队列绑定到exchange
        channel.queueBind(queueName, exchangeName, routingKey, null);

        DeliverCallback deliverCallback = new DeliverCallback() {
            @Override
            public void handle(String consumerTag, Delivery message) throws IOException {
                try {
                    System.out.println("---消费者：" + new String(message.getBody(), "UTF-8"));
                } finally {
                    // consumer手动 ack 给broker
                    channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                }
            }
        };
        CancelCallback cancelCallback = new CancelCallback() {
            @Override
            public void handle(String consumerTag) throws IOException {
                System.out.println("---消费者：cancelCallback");
            }
        };

        // 消费消息
        channel.basicConsume(queueName, false, deliverCallback, cancelCallback);
    }
}
```

**运行结果**

![RabbitMQ高级特性(二).生产端Confirm消息确认机制-运行结果2.png](figures/RabbitMQ高级特性(二).生产端Confirm消息确认机制-运行结果2.png)

## 生产者Return返回消息机制

### 为什么需要Return

Return Listener用于处理一些不可路由的消息

生产者通过指定Exchange和RoutingKey将消息发送达指定队列，消费者只需监听这个队列，进行消费操作。

但是在某些情况下，我们在发送消息的时候，当前Exchange不存在或者指定的路由key路由不到，这个时候我们如果需要监听这种达不到的消息，我们就需要使用Return Listener。

### 实现流程图

![RabbitMQ高级特性(二).生产端Return返回消息机制](figures/RabbitMQ高级特性(二).生产端Return返回消息机制.png)

在API中配置项：

Mandatory：如果为true，则会监听路由不可达消息，然后进行后续处理；如果为false，那么broker自动删除这条消息。

### 代码实现

**生产者**

```java
package com.wyg.rabbitmq.javaclient.producer_return;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.*;

/**
 * 生产者return机制
 */

public class Producer {
    private static final String HOST = "localhost";
    private static final int PORT = 5672;
    private static final String USERNAME = "guset";
    private static final String PASSWORD = "guset";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setVirtualHost("/");
        factory.setPort(PORT);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();
        String exchangeName = "test_return_exchange";
        String routingKey = "return.ok";
        String routingKeyError = "return.error";
        // 申明exchange
        channel.exchangeDeclare(exchangeName, "topic");
        // 添加监听
        channel.addReturnListener(new ReturnListener() {
            @Override
            public void handleReturn(int replyCode, String replyText, String exchange, String routingKey,
                AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("---生产者---");
                System.out.println("replyCode:" + replyCode);
                System.out.println("replyText:" + replyText);
                System.out.println("exchange:" + exchange);
                System.out.println("routingKey:" + routingKey);
                System.out.println("properties:" + properties);
                System.out.println("body:" + new String(body));

            }
        });

        // 设置 mandatory 为 true
        boolean mandatory = true;
        String msg = "这是一条return确认消息1,routingKey " + routingKey;
        channel.basicPublish(exchangeName, routingKey, mandatory, null, msg.getBytes("UTF-8"));
        String msg2 = "这是一条return确认消息2,routingKey " + routingKeyError;
        channel.basicPublish(exchangeName, routingKeyError, mandatory, null, msg2.getBytes("UTF-8"));

    }

    // 注意，因为要等待broker的return消息，暂时不关闭channel和connection
}
```

**消费者**

```java
package com.wyg.rabbitmq.javaclient.producer_return;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.*;

/**
 * 生产者return机制
 */

public class Consumer {

    private static final String HOST = "localhost";
    private static final int PORT = 5672;
    private static final String USERNAME = "guset";
    private static final String PASSWORD = "guset";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setVirtualHost("/");
        factory.setPort(PORT);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();

        String queueName = "test_return_queue";
        String exchangeName = "test_return_exchange";
        String routingKey = "return.ok";

        // 申明exchange
        channel.exchangeDeclare(exchangeName, "topic");
        // 申明队列
        channel.queueDeclare(queueName, true, false, false, null);
        // 队列绑定到exchange
        channel.queueBind(queueName, exchangeName, routingKey, null);

        DeliverCallback deliverCallback = new DeliverCallback() {
            @Override
            public void handle(String consumerTag, Delivery message) throws IOException {
                try {
                    System.out.println("---消费者---：" + new String(message.getBody(), "UTF-8"));
                } finally {
                    // consumer手动 ack 给broker
                    channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                }
            }
        };
        CancelCallback cancelCallback = new CancelCallback() {
            @Override
            public void handle(String consumerTag) throws IOException {
                System.out.println("---消费者：cancelCallback");
            }
        };

        // 消费消息
        channel.basicConsume(queueName, false, deliverCallback, cancelCallback);
    }
}
```

**注意**

1. 先运行Consumer,绑定正确的exchange和routingKey
2. 再启动生产者

运行结果

Consumer

![RabbitMQ高级特性(二).生产端Confirm消息确认机制-运行结果3](figures/RabbitMQ高级特性(二).生产端Confirm消息确认机制-运行结果3.png)

**Producer**，其中routingKey 为 return.error的消息无法路由，被Return Listener监听到

![RabbitMQ高级特性(二).生产端Confirm消息确认机制-运行结果4](figures/RabbitMQ高级特性(二).生产端Confirm消息确认机制-运行结果4.png)

## 消费者限流

### 为什么要进行消费者端限流

假设有个场景，RabbitMQ服务器上堆积上万条未处理的消息，我们随便打开一个消费者客户端会出现下面情况：巨量的消息同时推送过来，但是我们单个消费者客户端无法同时处理这么多数据，服务器可能卡死

### 什么是消费者限流

RabbitMQ提供了一种qos(服务质量保证)功能，即在非自动确认消息的情况下，如果一定数量的消息(通过基于consumer或者channel设置qos值)未被确认前，不消费新的消息

### 消费者限流的实现

生产者

```java
package com.wyg.rabbitmq.javaclient.consumer_limit;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * 消费者限流
 */

public class Producer {
    private static final String HOST = "localhost";
    private static final int PORT = 5672;
    private static final String USERNAME = "guset";
    private static final String PASSWORD = "guset";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setVirtualHost("/");
        factory.setPort(PORT);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();
        String exchangeName = "test_qos_exchange";
        String routingKey = "qos.abc";
        // 申明exchange
        channel.exchangeDeclare(exchangeName, "topic");

        // 发送十条消息
        for (int i = 0; i < 10; i++) {
            String msg = "这是一条 消费者限流消息," + i;
            channel.basicPublish(exchangeName, routingKey, false, null, msg.getBytes("UTF-8"));
        }
        channel.close();
        connection.close();
    }

}
```

消费者

```java
package com.wyg.rabbitmq.javaclient.consumer_limit;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.*;

/**
 * 消费者限流
 */

public class Consumer {

    private static final String HOST = "localhost";
    private static final int PORT = 5672;
    private static final String USERNAME = "guset";
    private static final String PASSWORD = "guset";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setVirtualHost("/");
        factory.setPort(PORT);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();

        String queueName = "test_qos_queue";
        String exchangeName = "test_qos_exchange";
        String routingKey = "qos.#";

        // 申明exchange
        channel.exchangeDeclare(exchangeName, "topic");
        // 申明队列
        channel.queueDeclare(queueName, true, false, false, null);
        // 队列绑定到exchange
        channel.queueBind(queueName, exchangeName, routingKey, null);

        // 单条消息的大小限制，一般设为0或不设置，不限制大小
        int prefecthSize = 0;
        // 告诉RabbitMQ不要同时给消费者推送n条消息，一旦有n个消息还没ack，则该consumer将block掉，直到有ack；注意在自动应答下不生效
        int prefecthCount = 1;
        // 表示是否应用于channel上，即是channel级别还是consumer级别
        boolean global = false;

        channel.basicQos(prefecthSize, prefecthCount, global);
        DeliverCallback deliverCallback = new DeliverCallback() {
            @Override
            public void handle(String consumerTag, Delivery message) throws IOException {
                try {
                    System.out.println("---消费者---");
                    System.out.println(new String(message.getBody(), "UTF-8"));
                } finally {
                    // consumer手动 ack 给broker
                    channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                }
            }
        };
        CancelCallback cancelCallback = new CancelCallback() {
            @Override
            public void handle(String consumerTag) throws IOException {
                System.out.println("---消费者：cancelCallback");
            }
        };

        // 消费消息,autoAck一定要设置为false
        channel.basicConsume(queueName, false, deliverCallback, cancelCallback);
    }
}
```

注意

1. 第一次我们注释掉 手动 ack给RabbitMQ应答

![RabbitMQ高级特性(二).消费端限流-手动注释RabbitMQ应答](figures/RabbitMQ高级特性(二).消费端限流-手动注释RabbitMQ应答.png)

运行结果

*发现一直卡在第一条消息，因为未给RabbitMQ手动应答，所以RabbitMQ认为消费者还未消费完，不推送新的消息*

![RabbitMQ高级特性(二).消费端限流-运行结果1.png](figures/RabbitMQ高级特性(二).消费端限流-运行结果1.png)

2. 第二次开启手动应答

![RabbitMQ高级特性(二).消费端限流-手动开启RabbitMQ应答.png](figures/RabbitMQ高级特性(二).消费端限流-手动开启RabbitMQ应答.png)

运行结果

*所有消息依次消费*

![RabbitMQ高级特性(二).消费端限流-运行结果2.png](figures/RabbitMQ高级特性(二).消费端限流-运行结果2.png)

## 消费端的ack和重回队列

### 消费端的手工ack和nack

### 什么是ack和nack

`ack`表示告知**RabbitMQ**已经成功消费消息

`nack`表示告知**RabbitMQ**消费端处理消息失败

### 手工ack和nack使用场景

- 消费端进行消费的时候，由于业务异常，我们可以进行日志记录，后续做补偿操作。
- 消费端由于服务器宕机等严重问题，比如消息消费一半时宕机，RabbitMQ既收不到ack也收不到nack，此时消费端采用手工ack，等消费端服务重启好后，RabbitMQ回重发此未能消费成功的消息，保障消息消费成功

### 消费端的重回队列

消费端重回队列是为了对没有处理成功的消息，把消息重新递给Broker

一般我们在实际应用中，都会关闭重回队列

### 代码实现

**生产者**

```java
package com.wyg.rabbitmq.javaclient.consumer_ack;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * 消费者手工ack和nack
 */

public class Producer {
    private static final String HOST = "localhost";
    private static final int PORT = 5672;
    private static final String USERNAME = "guset";
    private static final String PASSWORD = "guset";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setVirtualHost("/");
        factory.setPort(PORT);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();
        String exchangeName = "test_ack_exchange";
        String routingKey = "ack.abc";
        // 申明exchange
        channel.exchangeDeclare(exchangeName, "topic");
        for (int i = 0; i < 6; i++) {
            Map<String, Object> map = new HashMap<>();
            map.put("num", i);
            AMQP.BasicProperties props =
                new AMQP.BasicProperties.Builder().deliveryMode(2).contentEncoding("UTF-8").headers(map).build();

            String msg = "这是第" + i + "条ack消息";
            channel.basicPublish(exchangeName, routingKey, false, props, msg.getBytes("UTF-8"));
        }

        channel.close();
        connection.close();
    }

}
```

**消费者**

```java
package com.wyg.rabbitmq.javaclient.consumer_ack;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.*;

/**
 * 消费者手工ack和nack
 */

public class Consumer {

    private static final String HOST = "localhost";
    private static final int PORT = 5672;
    private static final String USERNAME = "guset";
    private static final String PASSWORD = "guset";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setVirtualHost("/");
        factory.setPort(PORT);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();

        String queueName = "test_ack_queue";
        String exchangeName = "test_ack_exchange";
        String routingKey = "ack.#";

        // 申明exchange
        channel.exchangeDeclare(exchangeName, "topic");
        // 申明队列
        channel.queueDeclare(queueName, true, false, false, null);
        // 队列绑定到exchange
        channel.queueBind(queueName, exchangeName, routingKey, null);

        DeliverCallback deliverCallback = new DeliverCallback() {
            @Override
            public void handle(String consumerTag, Delivery message) throws IOException {

                // consumer手动 ack 给broker
                int num = (int)message.getProperties().getHeaders().get("num");
                // 根据headers里的num做判断，num<3,发ack给broker，并将消息重新入队
                if (num < 3) {
                    System.out.println("---消费端nack---DeliveryTag:" + message.getEnvelope().getDeliveryTag() + ","
                        + new String(message.getBody(), "UTF-8"));
                    channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                } else {
                    // 根据headers里的num做判断，num>=3,发nack给broker，并将消息重新入队
                    System.out.println("---消费端nack---DeliveryTag:" + message.getEnvelope().getDeliveryTag() + ","
                        + new String(message.getBody(), "UTF-8"));
                    channel.basicNack(message.getEnvelope().getDeliveryTag(), false, true);

                }

            }
        };
        CancelCallback cancelCallback = new CancelCallback() {
            @Override
            public void handle(String consumerTag) throws IOException {
                System.out.println("---消费者--：cancelCallback");
            }
        };

        // 消费消息,autoAck一定要设为false，手工ack
        channel.basicConsume(queueName, false, deliverCallback, cancelCallback);
    }
}
```

运行结果

发现前3条消息成功消费，手工发ack给Broker

最后3条消息，发nack给Broker，并不断重回队列尾端，broker再将其推给消费端，一直循环消费失败

![RabbitMQ高级特性(二).消费端的ack和重回队列-运行结果.png](figures/RabbitMQ高级特性(二).消费端的ack和重回队列-运行结果.png)

## TTL队列/消息

time to live,即生存时间

RabbitMQ 支持消息的过期时间，可以在发消息是指定

RabbitMQ 支持队列的过期时间，从消息入队开始计算，只要超过了队列的超时时间配置，那么消息会自动清除

## 死信队列

消息没有任何消费者去消费就变为死信

### 消息变为死信有以下几种情况

- 消息被拒绝(basic.reject/basic.nack),并且requeue=false
- 消息TTL过期
- 队列达到最大长度

### 死信队列 Dead Letter Exchange(DLX)

#### DLX

利用DLX，当消息在一个队列中变成死信之后，它能被重新publish到另外一个exchange,这个exchange就是DLX。

DLX也是一个正常的Exchange,和一般的Exchange没有区别，它能在任何队列上被指定，实际就是设置某个队列的属性。当队列中有死信时，RabbitMQ会自动将死信消息发送到设置的DLX，进而被路由到另外一个队列，可以监听这个队列，做后续处理。

#### 死信队列设置

1. 申明死信队列的Exchange和queue，然后进行绑定
2. 申明正常队列Exchange和queue绑定，只不过要在队列上加参数 `arguments.put(x-dead-letter-exchange", "you dlx");`
