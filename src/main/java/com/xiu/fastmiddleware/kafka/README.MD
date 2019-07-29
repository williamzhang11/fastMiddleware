# kafka架构

## 1.kafka整体架构图

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/kafkaarch.png)

kafka整体架构是分布式架构，producer,broker,consumer,zookeeper都可以有多个：

	(1).Producer cluster:生产者集群。一般由实际业务项目组成，其不断往kafka集群写入数据
	
	(2).kafka Cluster:kafka服务器集群。负责接收生产者写入的数据，并将其持久化到文件里，最终
	将消息提供给Consumer Cluster
	
	(3).zookeeper集群:kafka集群依赖zookeeper来保存集群的的元信息，来保证系统的可用性
	
	(4).consumer cluster 消费者集群。由多个实际业务项目组成，不断从kafka服务器集群读取数据

## 2.重要概念：

	producer:生产者，消息的生产者
	
	kafka cluster:
		broker：broker是kafka实例，每个服务器上有一个多或多个kafka实例。每个kafka集群中的broker都有一个不重复的编号，如broker-0,broker-1等
		topic:消息的主题，可理解为消息的分类，kafka数据保存在topic。每个broker可创建多个topic
		partition:topic的分区，每个topic可以有多个分区，分区作用是做负载，提高kafka的吞吐量。同一个topic在不同分区的数据是不重复的。
			表现就是一个个的文件夹
		replication:每个分区都有多个副本。当主分区（Leader）故障时会选择一个follower,成为leader。kafka最大副本数量是10个。且
			副本数量不能大于broker数量，follwer和leader在不同机器上，同一机器对同一分区只能存放一个副本
		message:每一条发送的消息主体
		
	consumer:消费者，即消息的消费方
	
	consumer group：可以将多个消费组成一个消费者组，在kafka设计中同一个分区的数据只能被消费者组中的某一个消费者消费。同一个消费者组的消费者可以
	消费同一个topic的不同分区数据。提高kafka的吞吐量
	
	zookeeper:kafka集群依赖zookeeper保存集群的元数据信息，保证系统的可用性。


## 3.kafka消息从生产者到消费者的整个流程

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastTech/kafka/image/kafkamsg.jpg)

主要经历以下几个阶段



























