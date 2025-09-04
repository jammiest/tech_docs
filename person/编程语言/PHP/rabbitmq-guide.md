# README

## composer安装

> 官方文档：<https://www.rabbitmq.com/tutorials/tutorial-one-php.html>  
> composer包：<https://packagist.org/packages/php-amqplib/php-amqplib>  
> RabbitMQ核心内容：<https://mp.weixin.qq.com/s/OfxE6cx1hRTM_WkilT8uiQ>

**消息确认**ack包括

- `Producer`-`Broker`： 生产者端（可在消费者端进行发送消息的`ack`确认操作）
- `Brocker`：RabbitMQ服务器（严格来说，这部分并不算是`ack`可控制的部分，但是由于会影响到消息投递的可靠性，因此把它列在这里）,可以通过持久化进一步确保该环节消息丢失
- `Brocker`-`Consumer`：消费者端（可在消费者端进行接收消息的`ack`确认操作）

> 以上三部分，每部分都是彼此独立  
要确保整个消息投递的完全可靠，必须要保证这每各环节的完全确认

!> 消息确认会对队列性能造成一定影响，可根据实际业务选择消息确认

## RabbitMQ持久化内容

- Exchange持久化设置
- Queue持久化设置
- Message持久化设置

> 没有给的持久化都要单独进行设置
> Message持久化生效的必要条件是**Queue必须设置为持久化**


```shell
composer require php-amqplib/php-amqplib
```

## 消费者(Consumer)

### 确认模式(队列持久化，非确认模式)

```php
use PhpAmqpLib\Connection\AMQPStreamConnection;

    try {
        $connection = AMQPStreamConnection::create_connection(array_map(function ($host, $port) use ($user, $password) {
            return  ['host' => $host, 'port' => $port, 'user' => $user, 'password' => $password];
        }, $hosts, $ports), $options);

        $channel = $connection->channel();
        // queue_declare($queue='',$passive=false,$durable=false,$exclusive=false,$auto_delete=true,$nowait=false,$arguments=array(),$ticket=null)
        $channel->queue_declare('my_queue', false, true, false, false); //@1 @2

        //basic_consume($queue='',$consumer_tag='',$no_local=false,$no_ack=false,$exclusive=false,$nowait=false,$callback=null,$ticket=null,$arguments=array())
        $channel->basic_consume('my_queue', '', false, false, false, false, function ($msg) { // @4
            // 消息体：$msg->body
            // 其他处理流程
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']); //消息确认，不返回这表明消费者确认消息
        });

        while ($channel->is_consuming()) {
            $channel->wait();
        }
    } catch (\Exception $e) {
        echo $e->getMessage();
    }
```

## 生产者(Producer)

### 确认模式（队列持久化，消息持久化，Exchange持久化）

```php
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

        $user = 'guest';
        $password ='guest';
        $vhost = ['192.168.10.240']; //单机模式，支持多主机模式，追加IP到数组即可
        $ports = [9672]; //端口与vhost保持一致
        $options = [
            // 'insist' => false,
            // 'login_method' => 'AMQPLAIN',
            // 'login_response' => null,
            // 'locale' => 'en_US',
            // 'connection_timeout' => 3.0,
            // 'read_write_timeout' => 10.0,
            // 'context' => null,
            // 'keepalive' => false,
            // 'heartbeat' => 5
        ];
        $connection = AMQPStreamConnection::create_connection(array_map(function ($host, $port) use ($user, $password, $vhost) {
            return  ['host' => $host, 'port' => $port, 'user' => $user, 'password' => $password ,'vhost'=> $vhost];
        }, $hosts, $ports), $options);

        $channel = $connection->channel();

        $channel->set_ack_handler( //ack回调
            function (AMQPMessage $message) {
                echo "<br>acked " . $message->body . PHP_EOL; //处理消息
            }
        );
        $channel->set_nack_handler( //noack回调
            function (AMQPMessage $message) {
                echo "<br>nacked " . $message->body . PHP_EOL; //处理消息
            }
        );

        $channel->confirm_select(); // 启用确认模式

        // queue_declare($queue='',$passive=false,$durable=false,$exclusive=false,$auto_delete=true,$nowait=false,$arguments=array(),$ticket=null)
        $channel->queue_declare('my_queue', false, true, false, false); //声明持久化队列

        // exchange_declare($exchange,$type,$passive=false,$durable=false,$auto_delete=true,$internal=false,$nowait=false,$arguments=array(),$ticket=null)
        $channel->exchange_declare('my_exchange','fanout',false,true,false);   //声明持久化Exchange

        $channel->queue_bind('my_queue', 'my_exchange');     //将Exchange与队列绑定

        for ($i = 0; $i < 100; $i++) {      //模拟消息
            $msgTxt = str_pad($i, 3, 0, STR_PAD_LEFT). ' Hello World! ' . md5('time.' . $i);  //模拟设置消息内容
            $properties = [];
            $properties = ['delivery_mode'=> AMQPMessage::DELIVERY_MODE_PERSISTENT];  //设置消息持久化
            $msg = new AMQPMessage($msgTxt , $properties);          //消息封装
            $channel->basic_publish($msg, 'my_exchange', 'test.rabbitmq'); //消息投递
            usleep(mt_rand(1,20000)); //模拟推送的随机延时
        }

        $channel->wait_for_pending_acks();

        $channel->close();
```
