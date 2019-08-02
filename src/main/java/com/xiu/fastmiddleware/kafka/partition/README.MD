# kafka为什么对topic进行分区

（1）topic是逻辑概念，面向的是producer和consumer，partition是物理概念。如果不进行分区，将topic内的消息全部存储到一个broker
那么该topic的读写都由这一个broker处理，吞吐量很容易陷入瓶颈，不符合高吞吐量场景
（2）有了分区概念后，如果一个topic被分为10个partitions，kafka会根据一定算法将partition尽可能均匀分布到不同broker上
（3）producer发布消息时producer客户端可采用random，key-hash及轮询等算法选定目标partition，若不指定，kafka也将根据一定算法将其置于一分区上
（4）consumer拉取消息时，consumer客户端可采用range,轮询等算法分配partition,从不同broker拉取对应partition的leader分区

因此partition机制可以极大提高吞吐量，并使系统具备良好的水平扩展能力。

## 分区原则

(1)指定了partition,则直接使用
(2)未指定partition，但指定key，通过对key的value进行hash出一个partition
(3)partition和key都未指定，使用轮询选出一个partition



























