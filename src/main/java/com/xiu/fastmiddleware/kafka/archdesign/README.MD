# kafka架构设计要点

## 1.高吞吐量

	(1)数据磁盘持久化:消息不在内存中cache,直接写入磁盘,充分利用磁盘的顺序读写性能
	(2)zero-copy:减少io操作,采用zero-copy提高发送性能.
	  传统数据发送需要发送4次上下文切换,采用zero-copy数据直接在内核态交换,上下文切换减少为2次.
	(3)数据批量发送
	(4)数据压缩
	(5)topic划分为多个partition,提高并行度.数据在磁盘存取速度为O(1)
	
## 2.负载均衡

	1、Producer 根据用户指定的算法，将消息发送到指定的 Partition 中。
	2、Topic 存在多个 Partition ，每个 Partition 有自己的replica ，
	每个 replica 分布在不同的 Broker 节点上。多个Partition 需要选取出 Leader 
	partition ，Leader Partition 负责读写，并由 Zookeeper 负责 fail over 。
	3、相同 Topic 的多个 Partition 会分配给不同的 Consumer 进行拉取消息，进行消费。
	
## 3.拉取系统

由于 Kafka Broker会持久化数据，Broker没有内存压力，因此， 
Consumer非常适合采取 pull的方式消费数据，具有以下几点好处：

1、简化 Kafka 设计。
2、Consumer 根据消费能力自主控制消息拉取速度。
3、Consumer 根据自身情况自主选择消费模式，例如批量，重复消费，从尾端开始消费等

## 4.可扩展性

	通过 Zookeeper管理 Broker与 Consumer 的动态加入与离开。

	当需要增加 Broker节点时，新增的 Broker会向 Zookeeper注册，而 Producer 
	及 Consumer 会根据注册在 Zookeeper上的 watcher感知这些变化，并及时作出调整。
	
	当新增和删除 Consumer节点时，相同 Topic的多个 Partition会分配给剩余的 Consumer们。






	 