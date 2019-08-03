# kafka如何保证消息不被重复消费

任何消息中间件都有可能出现消息重复消费的问题，这是不可避免的，这需要由开发保证。

比如kafka中offset的概念，每个消息写入进入，都有一个offset,代表消息的序号。然后consumer
消费数据后，每个一段时间，会把消费者消费消息的offset提交一下。但是如果消费者消费了某个
消息，这是如果应用被kill掉，因为没有来的及提交offset。下次重启消费者应用时，kafka还是会把
刚刚的消息发送过来，从而导致重复消费。

重复消费其实不可怕，可怕的是出现重复消费时，怎么处理。只要保证消息幂等性就可以了。

保证幂等性方法：

1.如果是写数据库，插入数据库之前，根据消息的特点去数据库查一下，如果有就update,没有则插入

2.如果是写redis，直接用set，是天然幂等性

3.如果都不是的话，让生产者发送每条数据时里面加一个全局唯一的id,消费到之后，先根据这个id去
比如redis查一下，之前消费过没，如果没消费过，就处理，然后写redis。
如果消费过了，就不用处理了。

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/repeat.jpg)














