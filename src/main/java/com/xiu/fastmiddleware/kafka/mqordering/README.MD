# kafka如何保证消息顺序性

场景：

如建立一个topic,有三个partition,生产者在写的时候，可以指定一个key,如，某个订单id作为key,
那么这个订单相关的数据，一定会被分发到同一个partition中，而且这个partition中数据一定是有顺序的

消费者从partition中取出的数据的时候，也一定是有顺序的。在消费者里可能会搞个多线程来并发处理消息。因为
单线程处理的话，如果处理比较耗时，吞吐量太低，而多线程跑顺序可能乱掉.

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/mqordering1.png)

解决方案:

1.一个topic,一个partition,一个consumer,内部单线程消费,但单线程吞吐量太低,一般不会用

2.写N个内存queue,具有相同key的数据都写到同一个内存queue;然后对N个线程,每个线程分别消费一个内存queue
即可,这样可保证顺序性.

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/kafka/image/mqordering2.png)








