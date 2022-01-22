# RabbitMQ

## MQ（Message Queue）

**作用**

+ 解耦。
+ 冗余〈存储)：有些情况下处理数据的过程会失败，造成数据丢失，可使用消息中间件进行数据持久化；
+ 扩展性:消息中间件解耦了应用的处理过程，所以提高消息入队和处理的效率是很容易的，只要另外增加处理过程即可，不需要改变代码，也不需要调节参数。
+ 削峰: 在访问量剧增的情况下，程序不会因为突发的超负荷请求而崩溃。
+ 可恢复性: 当系统一部分组件失效时，不会影响到整个系，消息中间件降低了进程间的耦合度，所以即使 个处理消息的进程挂掉，加入消息中间件中的消息仍然可以在系统恢复后进行处理。
+ 顺序保证: 在大多数使用场景下，数据处理的顺序很重要，大部分消息中间件支持 定程度上的顺序性。
+ 缓冲: 在任何重要的系统中，都会存在需要不同处理时间的元素。消息中间件通过 个缓冲层来帮助任务最高效率地执行，写入消息中间件的处理会尽可能快速 该缓冲层有助于控制和优化数据流经过系统的速度。
+ 异步通信: 在很多时候应用不想也不需要立即处理消息 消息中间件提供了异步处理机制，允许应用把 些消息放入消息中间件中，但并不立即处理它，在之后需要的时候再慢慢处理

核心问题：解耦，异步，削峰。

![消息队列-解耦](http://oss.jankinwu.com/img/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97-%E8%A7%A3%E8%80%A6.jpg)

## 消息队列协议

**1. AMQP（Advance Message Queuing Protocol）**

特点：AMQP 十分可靠且功能强大。当然它及它的实现并不是足够轻量级。

应用场景：当简单的发布－订阅模型不能满足使用要求。

典型实现：RabbitMQ是AMQP消息队列最有名的开源实现。RabbitMQ同时还可以通过插件支持STOMP、MQTT等协议接入。Kafka、RocketMQ 均使用自定义的协议。

**2. MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）**

发布－订阅这一基本功能外，也提供一些其它特性：不同的消息投递保障（delivery guarantee），“至少一次”和“最多一次”。通过存储最后一个被确认接受的消息来实现重连后的消息恢复。

特点：它非常轻量级，并且从设计和实现层面都适合用于不稳定的网络环境中。

应用场景：物联网（IoT）场景中更适合，支持几乎所有语言进行开发，并且浏览器也可通过 WebSocket 来发送和接收 MQTT 消息。

**3. OpenMessage协议**

是近几年由阿里、雅虎和滴滴出行、 Stremalio等公司共同参与创立的分布式消息中间件、流处理等领域的应用开发标准。
特点：

+ 结构简单
+ 解析速度快
+ 支持事务和持久化设计。

## 消息分发策略

![消息分发策略1](http://oss.jankinwu.com/img/%E6%B6%88%E6%81%AF%E5%88%86%E5%8F%91%E7%AD%96%E7%95%A51.png)

**轮询分发**

在轮询分发的场景下，交换机并不知道后面消费者的消费能力，就两个消费者一人一个这样轮询
缺点：不同消费者处理任务的时间是不一样的，这样会造成性能浪费

**公平分发**

公平分发的本质就是能者多劳
1. 使用公平分发，必须关闭自动应答ack，然后改成手动应答方式。
2. 每个消费者发送确认消息之前，消息队列不发送下一个消息到消费者，一次只处理一个消息。
限制发送给同一个消费者不得超过1条消息。
3. 消费者增加了channel.basicQos(1)控制同一消息消费次数。

## 安装RabbitMQ

### 使用docker安装

```sh
docker run -dit --name Myrabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:management
```

## RabbitMQ 角色分类

**RabbitMQ各类角色描述：**

**1. none**

+ 不能访问 management plugin

**2. management**

+ 列出自己可以通过AMQP登入的virtual hosts
+ 查看自己的virtual hosts中的queues, exchanges 和 bindings
+ 查看和关闭自己的channels 和 connections
+ 查看有关自己的virtual hosts的“全局”的统计信息，包含其他用户在这些virtual hosts中的活动

**3. policymaker**

+ 包含management所有权限
+ 查看、创建和删除自己的virtual hosts所属的policies和parameters

**4. monitoring**

+ 包含management所有权限
+ 列出所有virtual hosts，包括他们不能登录的virtual hosts
+ 查看其他用户的connections和channels
+ 查看节点级别的数据如clustering和memory使用情况
+ 查看真正的关于所有virtual hosts的全局的统计信息

**5. administrator**
+ policymaker和monitoring可以做的任何事外加:
+ 创建和删除virtual hosts
+ 查看、创建和删除users
+ 查看创建和删除permissions
+ 关闭其他用户的connections

## RabbitMQ 核心组成

![	](http://oss.jankinwu.com/img/RabbitMQ%E6%A0%B8%E5%BF%83%E7%BB%84%E6%88%90.png)

1. Broker：消息队列服务器实体

2. Virtual Host：虚拟主机,一个broker里可以有多个Virtual Host，用作不同用户的权限分离

3. Exchange：消息交换机，指定消息规则，处理消息和队列之间的关系
    Exchange Types：direct、topic、fanout、headers

4. Queue：消息的载体,每个消息都会被投到一个或多个队列

5. Message：由Properties和Body组成，Properties是生产者添加的各种属性的集合，包括消息被哪个Message Queue接受，优先级等属性，body是需要传输的数据

6. Binding：绑定，把exchange和queue按照路由规则绑定起来

7. Routing Key：路由关键字,exchange根据这个关键字进行消息投递

8. connection：客户端与RabbitMQ server(broker)之间的TCP连接

9. channel：信道，打开信道才能进行通信，一个channel代码一个会话任务，它是真实 TCP 连接之上的虚拟连接

   

## AMQP

​		AMQP，即 Advanced Message Queuing Protocol（高级消息队列协议），是一个网络协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。2006年，AMQP 规范发布。

### AMQP 生产者流转过程

![02326f516e8720d9c8f7d628555ccfc8](http://oss.jankinwu.com/img/02326f516e8720d9c8f7d628555ccfc8.png)

### AMQP 消费者流转过程

![AMQP消费者流转过程](http://oss.jankinwu.com/img/AMQP%E6%B6%88%E8%B4%B9%E8%80%85%E6%B5%81%E8%BD%AC%E8%BF%87%E7%A8%8B.png)

## RabbitMQ 模式

### 简单模式 Hello World

![rabbitMQ简单模式](http://oss.jankinwu.com/img/rabbitMQ%E7%AE%80%E5%8D%95%E6%A8%A1%E5%BC%8F.png)

一个生产者P发送消息到队列Q,一个消费者C接收。

### 工作队列模式 Work Queue

![img](http://oss.jankinwu.com/img/20181113170522132.png)

一个生产者，多个消费者，每个消费者获取到的消息唯一，多个消费者只有一个队列。

### 发布/订阅模式 Publish/Subscribe

![img](http://oss.jankinwu.com/img/20181113170540476.png)

一个生产者发送的消息会被多个消费者获取。一个生产者、一个交换机、多个队列、多个消费者。

生产者端发布消息到交换机，使用“fanout”方式发送，即广播消息，不需要使用queue，发送端不需要关心谁接收。

### 路由模式 Routing

![img](http://oss.jankinwu.com/img/20181113170555287.png)

生产者发送消息到交换机并且要指定路由key，消费者将队列绑定到交换机时需要指定路由key。如不指定交换机，则会分配一个默认交换机，默认交换机type为direct。

消费者端和发布订阅模式的区别：
1、exchange 的 type 为 direct
2、发送消息的时候加入了 routing key

### 通配符模式 Topics 

![img](http://oss.jankinwu.com/img/20181113170631174.png)

生产者端不只按固定的routing key发送消息，而是按字符串“匹配”发送，消费者端同样如此。
与之前的路由模式相比，它将信息的传输类型的key更加细化，以“key1.key2.keyN…”的模式来指定信息传输的key的大类型和大类型下面的小类型，让消费者端可以更加精细的确认自己想要获取的信息类型。而在消费者端，不用精确的指定具体到哪一个大类型下的小类型的key，而是可以使用类似正则表达式(但与正则表达式规则完全不同)的通配符在指定一定范围或符合某一个字符串匹配规则的key，来获取想要的信息。“通配符交换机”（Topic Exchange）将路由键和某模式进行匹配。此时队列需要绑定在一个模式上。符号“#”匹配零个或多个词，符号“*”仅匹配一个词。

### RPC 远程调用模式

![img](http://oss.jankinwu.com/img/7b9b26a328f1373b600ab3f0f5e38f1a.png)
