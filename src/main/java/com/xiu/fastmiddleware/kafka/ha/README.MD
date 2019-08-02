# kafka如何保证高可用性

kafka架构中包括zookeeper集群，broker集群，producer,consumer集群

1.zookeeper，部署2N+1节点，形成zookeeper集群，保证高可用

2.kafka broker部署集群，基于副本机制。每个topic有多个partition（分区），每个partition（分区）有多个副本，partition中只有一个leader负责与
producer以及consumer通信，当leader所在服务器上宕机后，会从副本中选举一个为leader。

3.producer，producer与业务服务部署在一起。producer从zookeeper拉取到topic的元数据后，选择对应topic的leader分区,进行消息写入。
broker根据producer的request.required.acks配置。是写入自己完成就响应给producer成功，还是写入所有broker完成再响应。

	request.required.acks：
	
	0：生产者producer不等待来自broker同步完成的确认继续发送下一条消息。 此选项提供最低的延迟，但服务器发生故障时，会造成某些数据丢失，如
		producer并不知情，发出去的消息broker收不到
		
	1：producer在leader成功收到消息并确认后发送下一条message。但如果写入死亡leader但尚未复制，会丢失消息
	
	-1：producer在follower副本确认接收到消息后才算一次发送完成，只要有一个分区副本存活，可以保证消息不会丢失

	三种机制，性能递减，但数据健壮性递增	
	

4.kafka consumer部署集群。同一个consumer group中每个consumer分配对应的topic partition,
consumer只从leader分区拉取消息，当有新的consumer加入或老的consumer离开会将topic partition 再均衡，重新分配给consumer

总的来说，kafka和rocketmq的高可用方式类似，主要差异在kafka broker 的 副本机制，rocketmq broker 的主从复制方式。























