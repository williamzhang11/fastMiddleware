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


## 3.kafka消息从生产者到消费者的整个流程(工作流程)

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/kafkamsg.png)

### 3.1生产过程
（1）写入

producer采用推（push）模式将消息发布到broker，每条消息被追加到分区中，属于顺序写磁盘，保障kafka吞吐量。同一个分区只能被一个消费者订阅，但一个消费者可以订阅多个分区
因此消费者与分区之间是1对多的关系

（2）分区

kafka集群有多个消息服务器（broker）组成,发布到kafka集群的每条消息都有一个类别，用主题（topic）表示。不同应用产生不同类型的数据，
可以设置不同的主题。

kafka集群为每个主题维护了分布式的分区（partition）日志文件，物理上可以把主题看出分区的日志文件。主题的分区都是一个有序的，不可变的记录序列，
新的消息不断追加到日志中。分区中的每条消息都会按照时间顺序分配到一个递增的顺序编号，叫偏移量offset。偏移量能唯一定位当前分区中的每一条消息。

消息发送时被发送到一个topic，本质是一个目录，而topic是由一些partition log（分区日志）组成

下图topic有3个分区，每个分区偏移量从0开始，不同分区之间偏移量是独立的，不相互影响
![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/kafkapartition.png)








































