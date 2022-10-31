---
title: RabbitMQ基础知识
date: 2022-10-29 21:29:02
tags: 
- RabbitMQ
- Erlang
index_img: 
banner_img: /img/article8.webp
categories:
- RabbitMQ
comment: waline
---

# RabbitMQ介绍

## 前言

消息系统允许软件、应用相互连接和扩展。这些应用可以相互链接起来组成一个更大的应用，或者将用户设备和数据进行连接。消息系统通过将消息的发送和接收分离来实现应用程序的异步和解偶；消息队列(Message Queue，简称MQ)：是在消息的传输过程中保存消息的容器。用于分布式系统之间进行通信。

RabbitMQ是一个消息代理 —— 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。

## 各种选型和对比

|            | RabbitMQ               | ActiveMQ                    | RocketMQ                 | Kafka                                          |
| ---------- | ---------------------- | --------------------------- | ------------------------ | ---------------------------------------------- |
| 公司/社区  | Rabbit                 | Apache                      | 阿里                     | Apache                                         |
| 开发语言   | Erlang                 | Java                        | Java                     | Scala&Java                                     |
| 协议       | AMQP                   | OpenWire、AUTO、Stomp、MQTT | 自定义                   | 自定义                                         |
| 单机吞吐量 | 万级                   | 万级(最差)                  | 十万级                   | 十万级                                         |
| 消息延迟   | 微妙级                 | 毫秒级                      | 毫秒级                   | 毫秒以内                                       |
| 特性       | 并发能力很强，延时很低 | 老牌产品，文档较多          | MQ功能比较完备，扩展性佳 | 只支持主要的MQ功能，毕竟是为大数据领域准备的。 |

中小型软件公司，建议选RabbitMQ

大型软件公司，根据具体使用在rocketMq和kafka之间二选一

RabbitMQ，RocketMQ，Kafka--区别/对比/选型推荐阅读🔗[链接](https://blog.51cto.com/knifeedge/5011115)

## 基于AMQP协议RabbitMQ

[AMQP](https://www.amqp.org/)即 Advanced Message Queuing Protocol(高级消息队列协议)，是一个网络协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。2006年，AMQP规范发布。类比HTTP。

2007年，Rabbit技术公司基于AMQP标准开发的RabbitMQ1.0发布。RabbitMQ采用Erlang 语言开发。

## RabbitMQ作用

### 应用解耦

传统模式：系统间耦合性太强，系统A在代码中直接调用系统B和系统C的代码，如果将来D系统接入，系统A还需要修改代码，过于麻烦！

中间件模式：将消息写入消息队列，需要消息的系统自己从消息队列中订阅，从而系统A不需要做任何修改。

举例：购买一件商品，需要先支付，再扣减库存，但这两个操作必须是在同一事务中，即保证操作的原子性，但是这样做的话效率是极其低下的，如果使用RabbitMQ，只需要将消息发送给各自的队列来进行消息处理，支付和扣减库存的操作之间没有了关联性，这样支付系统和库存系统之间就进行了解耦。

### 异步处理

传统模式: 一些非必要的业务逻辑以同步的方式运行，太耗费时间

中间件模式: 将消息写入消息队列，非必要的业务逻辑以异步的方式运行，加快响应速度

举例：异步发送短信、推送消息、日志记录等

### 流量削峰

传统模式：并发量大的时候，所有的请求直接怼到数据库，造成数据库连接异常

中间件模式: 系统A慢慢的按照数据库能处理的并发量，从消息队列中慢慢拉取消息

在访问量急剧增大的时候，RabbitMQ可以减少并发访问的压力，比较常见的业务场景就是秒杀和签到系统，一般来说流量的削峰有两个处理方式：

a、上游队列缓冲，限速发送；

b、下游队列缓冲，限速执行。

常见的场景通常都采用第二种方式，为了不影响客户使用的响应速度和使用体验等。

# 安装RabbitMQ

## Mac环境安装

```sh
# 执行安装
brew install rabbitmq

# 启动
brew services start rabbitmq
# 关闭
brew services stop rabbitmq
```

备选通过原生方式操作

```sh
# 安装完之后配置环境变量
vim ~/.bash_profile
# 添加RabbitMQ环境
export RABBIT_HOME=/usr/local/Cellar/rabbitmq/version
export PATH=$PATH:$RABBITMQ_HOME/sbin
# 生效
source ~/.bash_profile


# 有些版本默认 rabbitmq 是没有安装 web 端的客户端的插件
rabbitmq-plugins enable rabbitmq management
# 启动
rabbitmq-server
# 后台启动 rabbitMQ
rabbitmq-server -detached
# 查看状态
rabbitmqctl status
# 停止服务
rabbitmqctl stop
```

## CentOS7 环境安装

### 安装Erlang环境

注意[erlang和rabbitmq的版本对应关系](https://www.rabbitmq.com/which-erlang.html)

```sh
# 通过rpm安装Erlang
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
# yum安装
yum install -y erlang
# 检查Erlang的版本号
erl
```

### 安装RabbitMQ

```sh
# 先导入两个key
rpm --import https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
rpm --import https://packagecloud.io/gpg.key
# 完成RabbitMQ的前置条件配置
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
# 配置环境
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
# 安装socat
yum -y install epel-release
yum -y install socat

# 下载rpm安装包
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.5/rabbitmq-server-3.8.5-1.el7.noarch.rpm
# 使用命名安装
rpm -ivh rabbitmq-server-3.8.5-1.el7.noarch.rpm

# 启用管理平台插件，启用插件后，可以可视化管理RabbitMQ
rabbitmq-plugins enable rabbitmq_management
# 启动RabbitMQ
systemctl start rabbitmq-server


# 开机启动RabbtMQ
chkconfig rabbitmq-server on
```

## 创建远程账户

RabbitMQ 3.3.1以后的版本，账号guest具有所有的操作权限，并且又是默认账号，出于安全因素的考虑，guest用户只能通过localhost登陆

```sh
# 创建admin账户，密码111 
rabbitmqctl  add_user admin 1111
# 设置admin账户角色
rabbitmqctl  set_user_tags admin  administrator
# 设置admin账户权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
# 查看所有账户(需要开启服务)
rabbitmqctl list_users
```

## 测试

浏览器输入：serverip:15672。其中serverip是RabbitMQ-Server所在主机的ip，15672是RabbitMQ-Server的端口号

# 管理界面

## 主页总览

connections：无论生产者还是消费者，都需要与RabbitMQ建立连接后才可以完成消息的生产和消费，在这里可以查看连接情况

channels：通道，建立连接后，会形成通道，消息的投递获取依赖通道。

Exchanges：交换机，用来实现消息的路由

Queues：队列，即消息队列，消息存放在队列中，等待消费，消费后被移除队列。



端口：

5672: rabbitMq的编程语言客户端连接端口

15672：rabbitMq管理界面端口

25672：rabbitMq集群的端口

## 添加用户界面

1、 超级管理员(administrator)

可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

2、 监控者(monitoring)

可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)

3、 策略制定者(policymaker)

可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。

4、 普通管理者(management)

仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。

5、 其他

无法登陆管理控制台，通常就是普通的生产者和消费者。

## Virtual Hosts

虚拟主机：类似于mysql中的database。他们都是以“/”开头

## 设置权限

操作包括对资源进行配置、写、读。
配置权限可创建、删除、资源并修改资源的行为，写权限可向资源发送消息，读权限从资源获取消息。

# RabbitMQ消息模型

RabbitMQ提供了6种[消息模型](https://www.rabbitmq.com/getstarted.html)，但是第6种其实是RPC，并不是MQ，因此不予讨论

但是其实[Publish/Subscribe](https://www.rabbitmq.com/tutorials/tutorial-three-python.html)、[Routing](https://www.rabbitmq.com/tutorials/tutorial-four-python.html)、 [Topics](https://www.rabbitmq.com/tutorials/tutorial-five-python.html)这三种都属于订阅模型，只不过进行路由的方式不同。

## Simple-简单模型/ "Hello World!

RabbitMQ是一个消息代理：它接受和转发消息。 你可以把它想象成一个邮政信箱

RabbitMQ与邮局的主要区别是它不处理纸张，而是接收，存储和转发数据消息的二进制数据块。

[图示](https://www.rabbitmq.com/img/tutorials/python-one.png)

P（producer/ publisher）：生产者，一个发送消息的用户应用程序。

C（consumer）：消费者，消费和接收有类似的意思，消费者是一个主要用来等待接收消息的用户应用程序

队列（红色区域）：rabbitmq内部类似于邮箱的一个概念。虽然消息流经rabbitmq和你的应用程序，但是它们只能存储在队列中。队列只受主机的内存和磁盘限制，实质上是一个大的消息缓冲区。许多生产者可以发送消息到一个队列，许多消费者可以尝试从一个队列接收数据。

总之：

生产者将消息发送到队列，消费者从队列中获取消息，队列是存储消息的缓冲区。

### 生产者发送消息

```java
public class Sender {
    public static void main(String[] args) throws IOException, TimeoutException {
        //连接rabbitmq的通道
        Connection connection = ConnectionUtil.getConnection();

        //channel：轻量级的connection，是connection的复用。是完成大部分api的地方
        Channel channel = connection.createChannel();

        //声明queue
        String QUEUE_NAME = "simple_queue";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);


        //发送消息
        String msg = "你好！！！";
        channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
        System.out.println("sender：" + msg);

        //关闭通道和连接
        channel.close();
        connection.close();
    }
}
```

### 消费者获取消息

```java
public class Receiver {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        String QUEUE_NAME = "simple_queue";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
               
                    String msg = new String(body);
                    System.out.println(msg);
            }
        };
        /**
         * 接收消息
         * String queue：queue名称
         * boolean autoAck：是否自动ack，只有ack后消息才会从queue中删除
         * Consumer callback：收到消息后的业务逻辑
         */
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

### 消息确认机制ACK

消息一旦被消费者接收，队列中的消息就会被删除。

那么问题来了：RabbitMQ怎么知道消息被接收了呢？

如果消费者领取消息后，还没执行操作就挂掉了呢？或者抛出了异常？消息消费失败，但是RabbitMQ无从得知，这样消息就丢失了！

因此，RabbitMQ有一个[ACK机制](https://www.rabbitmq.com/confirms.html#consumer-acks-api-elements)。当消费者获取消息后，会向RabbitMQ发送回执ACK，告知消息已经被接收。不过这种回执ACK分两种情况：

- 自动ACK：消息一旦被接收，消费者自动发送ACK
- 手动ACK：消息接收后，不会发送ACK，需要手动调用

哪种更好呢？

这需要看消息的重要性：

- 如果消息不太重要，丢失也没有影响，那么自动ACK会比较方便
- 如果消息非常重要，不容丢失。那么最好在消费完成后手动ACK，否则接收消息后就自动ACK，RabbitMQ就会把消息从队列中删除。如果此时消费者宕机，那么消息就丢失了。

上面设置的是自动ACK，如果要手动ACK，需要改动消费者Receiver的代码：

```java
				 /**
         * 接收消息
         * String queue：queue名称
         * boolean autoAck：是否自动ack，只有ack后消息才会从queue中删除
         * Consumer callback：收到消息后的业务逻辑
         */
channel.basicConsume(QUEUE_NAME, false, consumer);
```

### 自动ACK存在的问题

上面虽然设置了手动ACK，但是代码中并没有进行消息确认！所以消息并未被真正消费掉。

当我们关掉这个消费者，消息的状态再次称为Ready

设置手动ACK：

```java
public class Receiver {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        String QUEUE_NAME = "simple_queue";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println(msg);
                    //业务逻辑...
                    //业务逻辑处理失败
                    int a = 6 / 0;
                    /**
                     * long deliveryTag：当前消息的索引
                     * boolean multiple：是否处理多条消息
                     */
                    channel.basicAck(envelope.getDeliveryTag(), false);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        /**
         * 接收消息
         * String queue：queue名称
         * boolean autoAck：是否自动ack，只有ack后消息才会从queue中删除
         * Consumer callback：收到消息后的业务逻辑
         */
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

 `channel.basicAck(envelope.getDeliveryTag(), false);`第二个参数multiple设置为false；当出现异常手动进行消息确认

## Work Queues工作模型

[Work Queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)主要思想就是避免执行资源密集型任务时，必须等待它执行完成。相反我们稍后完成任务，我们将任务封装为消息并将其发送到队列。 在后台运行的工作进程将获取任务并最终执行作业。当你运行许多消费者时，任务将在他们之间共享，但是**一个消息只能被一个消费者获取**。

流程：

- P：生产者：任务的发布者

- C1：消费者，领取任务并且完成任务，假设完成速度较快

- C2：消费者2：领取任务并完成任务，假设完成速度慢

如何避免消息堆积？

1）采用workqueue，多个消费者监听同一队列。

2）接收到消息以后，而是通过线程池，异步消费。

### 生产者

循环发送50条消息

```java
public class Sender {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();

        Channel channel = connection.createChannel();

        String QUEUE_NAME = "work_queue";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        // 发送50次
        for (int i = 0; i < 50; i++) {
            String msg = "你好！！！"+i;
            channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
            System.out.println("sender：" + msg);
        }
        channel.close();
        connection.close();
    }
}
```

### 消费者1

```java
public class Receiver1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        String QUEUE_NAME = "work_queue";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 设置每次只拿一条消息
        //channel.basicQos(1);

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println("Receiver1：" + msg);
                    channel.basicAck(envelope.getDeliveryTag(), false);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

### 消费者2

```java
public class Receiver2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();

        Channel channel = connection.createChannel();

        String QUEUE_NAME = "work_queue";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 设置每次只拿一条消息
        //channel.basicQos(1);

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println("Receiver2：" + msg);
                    channel.basicAck(envelope.getDeliveryTag(), false);
                    //线程睡眠
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

第一次设置与消费者1基本相同，都不增加`channel.basicQos(1);`此行代码，消费者2增加线程睡眠；

接下来，两个消费者一同启动，然后发送50条消息：

可以发现，两个消费者各自消费了25条消息，而且各不相同，这就实现了任务的分发；但是消费者1已经执行完毕，消费者2仍旧工作ing

### 能者多劳

上面的状态属于是把任务平均分配，耗时不一致；正确的做法应该是消费越快的人，消费的越多。下面是实现方式

第二次消费者增加`channel.basicQos(1);`代码；告诉RabbitMQ一次不要向工作人员发送多于一条消息。 或者换句话说，不要向工作人员发送新消息，直到它处理并确认了前一个消息。 相反，它会将其分派给不是仍然忙碌的下一个工作人员。

再次测试结果为能者多干，摸鱼的少干

## Fanout-广播模型

Fanout，称为广播；也有的称为发布/订阅模式：[Publish/Subscribe](https://www.rabbitmq.com/tutorials/tutorial-three-python.html)一次向多个消费者发送消息

消息发送流程是这样的：

-   可以有多个消费者
-   每个**消费者有自己的queue**（队列）
-   每个**队列都要绑定到Exchange**（交换机）
-   **生产者发送的消息，只能发送到交换机**，交换机来决定要发给哪个队列，生产者无法决定。
-   交换机把消息发送给绑定过的所有队列
-   队列的消费者都能拿到消息。实现一条消息被多个消费者消费

### 生产者

- 声明Exchange，不再声明Queue
- 发送消息到Exchange，不再发送到Queue

```java
public class Sender {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();

        // 声明exchange，指定类型为fanout
        String EXCHANGE_NAME = "fanout_exchange";
        channel.exchangeDeclare(EXCHANGE_NAME, ExchangeTypes.FANOUT);
				// 消息内容
        String msg = "你好！！！3 fanout" ;
        // 发布消息到Exchange
        channel.basicPublish(EXCHANGE_NAME, "", null, msg.getBytes());
        System.out.println("sender：" + msg);
        channel.close();
        connection.close();
    }
}
```

### 消费者1

```java
public class Receiver1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        String QUEUE_NAME = "fanout_queue_1";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 把queue绑定到exchange
        String EXCHANGE_NAME = "fanout_exchange";
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"");

        channel.basicQos(1);
			  // 定义队列的消费者
        Consumer consumer = new DefaultConsumer(channel) {
           // 获取消息，并且处理，这个方法类似事件监听，如果有消息的时候，会被自动调用
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    // body 即消息体
                    String msg = new String(body);
                    System.out.println("Receiver1：" + msg);
                    // false 仅确认提供的交货标签，处理单条
                    channel.basicAck(envelope.getDeliveryTag(), false);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

​	注意代码中：**队列需要和交换机绑定**

### 消费者2

```java
public class Receiver2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();

        Channel channel = connection.createChannel();

        String QUEUE_NAME = "fanout_queue_2";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 把queue绑定到exchange
        String EXCHANGE_NAME = "fanout_exchange";
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME, "");

        channel.basicQos(1);

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println("Receiver2：" + msg);
                    channel.basicAck(envelope.getDeliveryTag(), false);

                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

发送一条消息，两个都能够接收；注意需要绑定；同时也可以在web管理界面Exchanges中查看Bindings观察

## Direct-定向模型

Direct-定向模型也叫路由模式[Routing](https://www.rabbitmq.com/tutorials/tutorial-four-python.html);在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。在Direct模型下，队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）

消息的发送方在向Exchange发送消息时，也必须指定消息的routing key。

P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。

X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列

C1：消费者，其所在队列指定了需要routing key 为 error 的消息

C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息

### 生产者

`channel.basicPublish(EXCHANGE_NAME, "error", null, msg.getBytes());`中第二个参数指定routingKey

```java
public class Sender {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();

        // 声明exchange
        String EXCHANGE_NAME = "direct_exchange";
        channel.exchangeDeclare(EXCHANGE_NAME, ExchangeTypes.DIRECT);

        String msg = "direct你好！！！" ;
        // 指定routingKey
        channel.basicPublish(EXCHANGE_NAME, "error", null, msg.getBytes());
        System.out.println("sender：" + msg);
        channel.close();
        connection.close();
    }
}
```

### 消费者1

` channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"error");`中第二个参数指定routingKey

```java
public class Receiver1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();

        Channel channel = connection.createChannel();

        String QUEUE_NAME = "direct_queue_1";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //把queue绑定到exchange
        String EXCHANGE_NAME = "direct_exchange";
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"error");

        channel.basicQos(1);

        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println("Receiver1：" + msg);
                    channel.basicAck(envelope.getDeliveryTag(), false);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

### 消费者2

同样指定多个RoutingKey

```java
public class Receiver2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();

        Channel channel = connection.createChannel();

        String QUEUE_NAME = "direct_queue_2";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //把queue绑定到exchange
        String EXCHANGE_NAME = "direct_exchange";
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME, "info");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME, "warning");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME, "error");

        channel.basicQos(1);

        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println("Receiver2：" + msg);
                    channel.basicAck(envelope.getDeliveryTag(), false);

                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

测试发现只有生产者和消费者拥有相同的RoutingKey参数才能够接收消息进行消费

## Topic-主题模型

[Topics](https://www.rabbitmq.com/tutorials/tutorial-five-python.html)类型的`Exchange`与`Direct`相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过`Topic`类型`Exchange`可以让队列在绑定`Routing key` 的时候使用通配符！

Routingkey 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如： item.insert

 通配符规则：

- `#`：匹配一个或多个词
- `*`：匹配不多不少恰好1个词

举例：

`audit.#`：能够匹配`audit.irs.corporate` 或者 `audit.irs`

`audit.*`：只能匹配`audit.irs`

### 生产者

```java
public class Sender {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();

        //声明exchange
        String EXCHANGE_NAME = "topic_exchange";
        channel.exchangeDeclare(EXCHANGE_NAME, ExchangeTypes.TOPIC);

        String msg = "topic_exchange你好！！！" ;
        channel.basicPublish(EXCHANGE_NAME, "item.delete", null, msg.getBytes());
        System.out.println("sender：" + msg);
        channel.close();
        connection.close();
    }
}
```

### 消费者1

```java
public class Receiver1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();

        Channel channel = connection.createChannel();

        String QUEUE_NAME = "topic_queue_1";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //把queue绑定到exchange
        String EXCHANGE_NAME = "topic_exchange";
      	// topic
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"item.delete");

        channel.basicQos(1);

        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println("Receiver1：" + msg);
                    channel.basicAck(envelope.getDeliveryTag(), false);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

### 消费者2

```java
public class Receiver2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();

        Channel channel = connection.createChannel();

        String QUEUE_NAME = "topic_queue_2";
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //把queue绑定到exchange
        String EXCHANGE_NAME = "topic_exchange";
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME, "item.*");

        channel.basicQos(1);

        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println("Receiver2：" + msg);
                    channel.basicAck(envelope.getDeliveryTag(), false);

                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

# 持久化/durable

如何避免消息丢失？

1）  消费者的手动ACK机制。可以防止业务处理失败。

2）  但是，如果在消费者消费之前，MQ就宕机了，消息就没了。

是否可以将消息进行持久化呢？

要将消息持久化，前提是：队列、Exchange都持久化

### 生产者

```java
public class Sender {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();

        //声明exchange
        String EXCHANGE_NAME = "durable_exchange";
        channel.exchangeDeclare(EXCHANGE_NAME, ExchangeTypes.TOPIC,true);

        for (int i = 0; i < 50; i++) {
            String msg = "durable_exchange你好！！！" + i;
            channel.basicPublish(EXCHANGE_NAME, "item.delete", MessageProperties.PERSISTENT_TEXT_PLAIN, msg.getBytes());
            System.out.println("sender：" + msg);
        }

        channel.close();
        connection.close();
    }
}
```

### 消费者

```java
public class Receiver {
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = ConnectionUtil.getConnection();

        Channel channel = connection.createChannel();

        String QUEUE_NAME = "durable_queue";
        // 队列持久化durable true
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);

        //把queue绑定到exchange
        String EXCHANGE_NAME = "durable_exchange";
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"item.*");

        channel.basicQos(1);

        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                try {
                    String msg = new String(body);
                    System.out.println("Receiver1：" + msg);
                    channel.basicAck(envelope.getDeliveryTag(), false);

                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

### 交换机持久化

```java
// exchangeDeclare声明方法第三个参数durable设置为true
channel.exchangeDeclare(EXCHANGE_NAME, ExchangeTypes.TOPIC,true);
```

### 队列持久化

```java
// queueDeclare声明方法第二个参数durable设置为true
channel.queueDeclare(QUEUE_NAME, true, false, false, null);
```

### 消息持久化

```java
// basicPublish中第三个属性prop设置为MessageProperties.PERSISTENT_TEXT_PLAIN
channel.basicPublish(EXCHANGE_NAME, "item.delete", MessageProperties.PERSISTENT_TEXT_PLAIN, msg.getBytes());
```

### 测试

​	分别测试持久化和非持久化：

​	1、Send给Recv发送50条消息

​	2、Recv收到一条消息sleep1秒钟，收到前几条消息后立即关闭

​	3、重启RabbitMQ观察消息是否丢失

推荐阅读[RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)

<div>
<hr>
<script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script> 
<link
  rel="stylesheet"
  href="https://unpkg.com/@waline/client@v2/dist/waline.css"
/>
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>

