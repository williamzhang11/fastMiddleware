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

(3)副本（replication）

同一个partition会有多个,没有replication的情况，一旦broker宕机，其上所有partition的数据都不可被消费，同时producer也不能再将数据存于
partition上。引入replication后，同一个partition可能有多个replication，这需要在这些replication之间选出一个leader,producer和
consumer只与这个leader交互，其它replication作为follower从leader中复制数据。

（4）写入流程

producer写入消息流程：

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/producer.png)

	1)producer先从zookeeper的/brokers/../state 节点找到该partition的leader
	
	2)producer将消息发送给该leader
	
	3)leader将消息写入本地log
	
	4)followers从leader pull消息，写入本地log后向leader发送ack
	
	5)leader收到所有ISR中的replication的ack后，增加HW(high watermark 最后commit的offset)
	并向producer发送ack
	
## 3.2broker保存消息

（1）存储方式

物理上把topic分成一个或多个partition，每个partition物理上对应一个文件夹（存储partition的消息和索引文件）

（2）存储策略

不论消息是否被消费，kafka都会保留所有消息。有2种策略可以删除旧数据：

	1）基于时间：log.retention.hours=168
	2)基于大小：log.retention.bytes=1073741824 
	
因为kafka读取特定消息的时间复杂度为O(1),即与文件大小无关，所以这里删除过期文件与提高kafka性能无关

（3）zookeeper存储结构

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/kafkazookeeper.png)

producer不在zk种注册，消费者再zk中注册

## 3.3kafka消费

消息的消费模型有2种，推送模型（push）和拉取模型（pull）

推送模型：

基于推送的消息系统，由消息代理记录消费者的消费状态。消息代理在将消息推送到消费者后，标记这条消息为已消费。
但这种方法无法很好保证消息被处理。比如消息代理把消息发送出去后，当消费进程挂掉或者由于网络原因没有收到
这条消息时，可能造成消息丢失（因为消息代理把这条消息标记为已消费，实际上这条消息并没有被实际处理）。如果
要保证消息被处理，消息代理发送完消息后，要设置状态为已发送，只有收到消费者确认请求后，才更新为已消费。
这就要求消息代理种记录所有消费状态，这种做法显然时不可取的。

kafka采用拉取模型，由消费者自己记录消费状态，每个消费者互相独立顺序读取每个分区消息。consumer
可自主控制消费消息的速率，同时consumer可以自己控制消费方式——即可批量消费也可逐条消费，
同时还能选择不同的提交方式从而实现不同的传输语义。

pull模式不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直等待数据到达。
为了避免这种情况，我们在我们的拉请求中有参数，允许消费者请求在等待数据到达的“长轮询”
中进行阻塞（并且可选地等待到给定的字节数，以确保大的传输大小）。

如下图，有2个消费者（不同消费者组）拉取同一个主题的消息，消费者A的消费进度是3，消费者b的消费进度是6.消费者拉取
的最大上限通过最高watermark控制。如果生产者最新写入的消息如果还没达到备份数量，对消费者是不可见的。
这种消费者控制偏移量的优点是：消费可以按照任意顺序消费消息。如：消费者可以重置到旧的偏移量，重新处理之前已经消费
过的消息；或者直接跳到最近位置，从当前时刻开始消费。
![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/consumer.png)


在一些消息系统中，消息代理会在消息被消费后立即删除消息。如果有不同类型的消费者订阅同一个主题，
消息代理可能需要冗余存储同一消息；或者等所有消费者都消费完才删除，这就需要消息代理跟踪每个消费者的消费状态
这种设计很大程度限制了消息系统的整体吞吐量和处理延迟。

kafka是生产者发布的所有消息都会保存在集群中，不管消息有没有被消费。用户可以设置保留时间来清理过期数据。


高级api不需要自行管理offset,系统通过zookeeper自行管理
不需要管理分区，副本等情况，系统自动管理。
消费者断线会自动根据上一次记录在zookeeper中的offset去获取数据（默认设置1分钟更新一下zookeeper存的
offset）
可以使用group区分对同一个topic的不同程序访问分离开来（不同group记录不同的offset，这样不同
程序读取同一个topic才不会因为offset相互影响）











































