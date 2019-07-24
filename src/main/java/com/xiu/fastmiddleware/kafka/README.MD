# kafka架构

## 1.kafka整体架构图

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastTech/kafka/image/kafkaarch.jpg)

kafka整体架构是分布式架构，producer,broker,consumer,zookeeper都可以有多个：

	1.Producer cluster:生产者集群。一般由实际业务项目组成，其不断往kafka集群写入数据
	
	2.kafka Cluster:kafka服务器集群。负责接收生产者写入的数据，并将其持久化到文件里，最终
	将消息提供给Consumer Cluster
	
	3.zookeeper Cluster:zookeeper集群。负责维护整个kafka集群的topic，kafka controller等信息
	
	4.consumer cluster 消费者集群。由多个实际业务项目组成，不断从kafka服务器集群读取数据

## 2.重要概念：

topic:指kafka处理消息源的不同分类

partition:topic物理分区，一个topic可分为多个partition。每个partition都是一个有序队列。partition中
的每条消息被分配一个有序的id

	replicas:partition的副本集，保障partition的高可用
	leader:replicas中的一个角色，producer和consumer只跟leader交互
	follower：replicas中的一个角色，从leader中复制数据，作为副本，一旦leader挂掉
	会从它的followers中选举出一个新的leader继续提供服务。

message：消息，通信的基本单位，每个producer可向一个topic发布一些消息

producers:消息和数据生产者，向kafka的一个topic发布消息的过程

consumers:消息和数据消费者，订阅topic，并处理发布消息过程

	consumer group:每个consumer都属于一个consumer group，每条消息只能被consumer group
	中的一个consumer消费，但可被多个consumer group消费

broker:kafka集群中的服务器

	controller:kafka集群中，通过zookeeper选举某个broker作为controller,用来进行leader election
	及各种failover(故障转移)
	
zookeeper：kafka通过zookeeper存储集群的topic partition等原信息

## 3.kafka消息从生产者到消费者的整个流程

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastTech/kafka/image/kafkamsg.jpg)

主要经历以下几个阶段

	1.服务器启动阶段
	
	2.生产者发送消息阶段
	
	3.kafka存储消息阶段
	
	4.消费者拉取消息阶段
	
### 3.1服务器启动阶段

首先启动zookeeper服务器集群。

接着启动kafka服务器集群，kafka server会向zookeeper服务器注册信息，接着启动线程池监听
客户端连接请求。

最后，启动生产者，消费者，从zookeeper服务器









