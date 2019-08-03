# kafka如何保证消息传输可靠性

kafka是否会丢失数据，主要取决于怎么使用 

## 1.消费端

场景：

消费端可能丢失数据的唯一情况是：消费到了某个消息，然后消费者自动提交了offset，broker就认为已经
消费好了这个消息，但消费端还没来得及处理就挂了，这是消息就丢失了。

方法：在kafka中可以关闭自动提交，使用手动提交，处理完后再提交offset就可以保证数据不会消失。

## 2.broker

场景:

kafka某个broker宕机，重新选举partition的leader。此时其他follower刚好有些数据没有
同步，此时挂了，选举某个follower成leader后，也就丢了一些数据。

方法：

（1）给topic设置分区副本数，replication.factor参数。这个值必须大于1，要求每个partition必须
至少有2个副本

（2）broker服务端设置min.insync.replicas参数，这个值必须大于1，要求一个leader至少
感知到至少有一个follower还跟自己保持联系，即leader挂了，还有个follower。

（3）producer端设置acks=all,要求每条数据，必须写入所有replica后，才能认为是成功。

（4）producer端设置retries=max（一个很大的值），一旦写入写入失败，就无限重试。

这样配置可以保证broker端leader所在broker发送故障，进行leader切换时，数据不会丢失。

## 3.生产者

producer设置了acks=all，一定不会丢失，leader收到消息，所有的follower都同步到消息后，才认为
写成功，没有满足就不断重试。






