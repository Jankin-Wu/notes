# Kafka

## 部署 Kafka 集群

### 使用 docker 部署（不成功）

```sh
docker run -d -v /usr/local/docker/kafka:/data/kafka-data --name kafka \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=hadoop102:2181,hadoop103:2181,hadoop104:2181/ka \
-e KAFKA_LOG_RETENTION_HOURS=5 \
-e KAFKA_LOG_DIRS=/data/kafka-data \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.8.0.3:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka
```

### 使用 docker-compose 部署

#### 部署 zookeeper + kafka 集群

```yaml
version: '3.8'
services:
  zk1:
    image: 'zookeeper:3.5.7'
    restart: always
    container_name: zk1
    hostname: zk1
    networks:
      - zookeeper-kafka
    ports:
      - "2181:2181"
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
  zk2:
    image: 'zookeeper:3.5.7'
    restart: always
    container_name: zk2
    hostname: zk2
    networks:
      - zookeeper-kafka
    ports:
      - "2182:2181"
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
 
  zk3:
    image: 'zookeeper:3.5.7'
    restart: always
    container_name: zk3
    hostname: zk3
    networks:
      - zookeeper-kafka
    ports:
      - "2183:2181"
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
      
  kafka1:
    image: 'bitnami/kafka:latest'
    restart: always
    container_name: kafka1
    networks:
      - zookeeper-kafka
    ports:
      - '9093:9093'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zk1:2181,zk2:2181,zk3:2181/kafka
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka1:9092,EXTERNAL://192.168.31.230:9093
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9093
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zk1
      - zk2
      - zk3

  kafka2:
    image: 'bitnami/kafka:latest'
    restart: always
    container_name: kafka2
    networks:
      - zookeeper-kafka
    ports:
      - '9094:9094'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zk1:2181,zk2:2181,zk3:2181/kafka
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka2:9092,EXTERNAL://192.168.31.230:9094
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9094
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zk1
      - zk2
      - zk3

  kafka3:
    image: 'bitnami/kafka:latest'
    restart: always
    container_name: kafka3
    networks:
      - zookeeper-kafka
    ports:
      - '9095:9095'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zk1:2181,zk2:2181,zk3:2181/kafka
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka3:9092,EXTERNAL://192.168.31.230:9095
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9095
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zk1
      - zk2
      - zk3

networks:
  zookeeper-kafka:
```

 **docker-compose 文件解释**

+ zookeeper 的配置见zookeeper文章。

+ `KAFKA_CFG_ZOOKEEPER_CONNECT`：zookeeper 连接配置，`/kafka`指的是在zookeeper根节点下建立一个`kafka`目录，方便管理。
+ `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP`：监听器名称和安全协议之间的映射关系集合。`PLAINTEXT`为不需要授权,非加密通道。
+ `KAFKA_CFG_LISTENERS`：就是主要用来定义Kafka Broker的Listener的配置项。为了使用内部和外部客户端访问 Apache Kafka 代理，需要为每种客户端配置一个侦听器。Kafka是由Zookeeper进行管理的，由Zookeeper负责Leader选举，Broker Rebalance等工作。所以External Producer和External Consumer其实是通过Zookeeper中提供的信息和Broker通信交互的。所以`listeners`中配置的信息都会发布到Zookeeper中，但是这样就会把Broker的所有Listener信息都暴露给了外部Clients，在安全上是存在隐患的，我们希望只把给外部Clients使用的Listener暴露出去，此时就需要用到下面这个配置项了。
+ `KAFKA_CFG_ADVERTISED_LISTENERS`：这个参数的作用就是将Broker的Listener信息发布到Zookeeper中，供Clients（Producer/Consumer）使用。如果配置了`KAFKA_CFG_ADVERTISED_LISTENERS`，那么就不会将`listeners`配置的信息发布到Zookeeper中去了。这里在Zookeeper中发布了供External Clients（Producer/Consumer）使用的Listener——`EXTERNAL`。所以`KAFKA_CFG_ADVERTISED_LISTENERS`配置项实现了只把给外部Clients使用的Listener暴露出去的需求。所以这里`EXTERNAL`配置的是宿主机的地址和端口。
+ `KAFKA_INTER_BROKER_LISTENER_NAME`：指定一个或一类Listener的名称，将它作为Internal Listener。这个Listener专门用于Kafka集群中Broker之间的通信。

## Kafka 命令行操作

```sh
# 进入docker容器内
docker exec -it kafka1 /bin/bash
# 创建一个名称为 “first”，有一个分区，两个副本的topic
kafka-topics.sh --bootstrap-server kafka1:9092 --topic first --create --partitions 1 --replication-factor 3
# 修改分区数（分区数只能增加，不能减少）
kafka-topics.sh --bootstrap-server kafka1:9092 --alter --topic first --partitions 3
# 查看 topic 列表
kafka-topics.sh --bootstrap-server kafka1:9092 --list
# 生产消息
kafka-console-producer.sh --bootstrap-server kafka1:9092 --topic first
# 消费消息
kafka-console-consumer.sh --bootstrap-server kafka1:9092 --topic first --from-beginning
# 删除 topic
kafka-topics.sh --bootstrap-server kafka1:9092 --delete --topic first
```

## Kafka 发送端参数

+ BOOTSTRAP_SERVERS_CONFIG：连接集群的ip+端口
+ ACKS_CONFIG：发出消息持久化机制参数
  - acks=0： 表示producer不需要等待任何broker确认收到消息的回复，就可以继续发送下一条消息。性能最高，但是最容易丢消息。
  - acks=1： 至少要等待leader已经成功将数据写入本地log，但是不需要等待所有follower是否成功写入。就可以继续发送下一条消息。这种情况下，如果follower没有成功备份数据，而此时leader又挂掉，则消息会丢失。
  - acks=-1或all： 需要等待 min.insync.replicas(默认为1，推荐配置大于等于2) 这个参数配置的副本个数都成功写入日志，这种策略会保证只要有一个备份存活就不会丢失数据。这是最强的数据保证。一般除非是金融级别，或跟钱打交道的场景才会使用这种配置。
+ BUFFER_MEMORY_CONFIG：设置发送消息的本地缓冲区，如果设置了该缓冲区，消息会先发送到本地缓冲区，可以提高消息发送性能，默认值是33554432，即32MB。
+ BATCH_SIZE_CONFIG：kafka本地线程会从缓冲区取数据，批量发送到broker，设置批量发送消息的大小，默认值是16384，即16kb，就是说一个batch满了16kb就发送出去。
+ LINGER_MS_CONFIG：发送消息的延时配置，默认值是0，意思就是消息必须立即被发送，但这样会影响性能，一般设置10毫秒左右，就是说这个消息发送完后会进入本地的一个batch，如果10毫秒内，这个batch满了16kb就会随batch一起被发送出去，如果10毫秒内，batch没满，那么也必须把消息发送出去，不能让消息的发送延迟时间太长。
+ RETRIES_CONFIG：发送失败的重试次数，默认重试间隔100ms，重试能保证消息发送的可靠性，但是也可能造成消息重复发送，比如网络抖动，所以需要在接收者那边做好消息接收的幂等性处理。
+ RETRY_BACKOFF_MS_CONFIG：重试时间间隔设置。
+ KEY_SERIALIZER_CLASS_CONFIG：把发送的key从字符串序列化为字节数组。
+ VALUE_SERIALIZER_CLASS_CONFIG：把发送消息value从字符串序列化为字节数组。

## 使用 Kraft 模式部署Kafka集群

```yaml
version: '3.8'
services:
  kafka1:
    image: 'bitnami/kafka:latest'
    restart: always
    container_name: kafka1
    networks:
      - kafka
    ports:
      - '9093:9093'
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_KRAFT_CLUSTER_ID=LelM2dIFQkiUFvXCEcqRWA
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka1:9092,EXTERNAL://192.168.31.230:9093
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9093
      - ALLOW_PLAINTEXT_LISTENER=yes

  kafka2:
    image: 'bitnami/kafka:latest'
    restart: always
    container_name: kafka2
    networks:
      - kafka
    ports:
      - '9094:9094'
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_KRAFT_CLUSTER_ID=LelM2dIFQkiUFvXCEcqRWA
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka2:9092,EXTERNAL://192.168.31.230:9094
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9094
      - ALLOW_PLAINTEXT_LISTENER=yes

  kafka3:
    image: 'bitnami/kafka:latest'
    restart: always
    container_name: kafka3
    networks:
      - kafka
    ports:
      - '9095:9095'
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_KRAFT_CLUSTER_ID=LelM2dIFQkiUFvXCEcqRWA
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka3:9092,EXTERNAL://192.168.31.230:9095
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9095
      - ALLOW_PLAINTEXT_LISTENER=yes

networks:
  kafka:
```

### 配置说明

+ KAFKA_CFG_PROCESS_ROLES：使用Kraft模式来运行kafka集群的话，必须配置。有以下值可以配置：
  - KAFKA_CFG_PROCESS_ROLES=Broker, 服务器在KRaft模式中充当Broker。
  - KAFKA_CFG_PROCESS_ROLES=Controller, 服务器在KRaft模式下充当Controller。
  - KAFKA_CFG_PROCESS_ROLES=Broker,Controller，服务器在KRaft模式中同时充当Broker和Controller。
  - 如果KAFKA_CFG_PROCESS_ROLES没有设置。那么集群就假定是运行在ZooKeeper模式下。