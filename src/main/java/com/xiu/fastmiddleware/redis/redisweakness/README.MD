# redis优缺点

## 1.redis优点

1.1.速度快

纯内存操作，Redis 的性能非常出色，每秒可以处理超过 10 万次读写操作，
是已知性能最快的 Key-Value 数据库
	
1.2.支持丰富数据类型

支持 String ，List，Set，Sorted Set，Hash 

1.3.丰富的特性

订阅发布 Pub / Sub 功能
Key 过期策略
事务
支持多个 DB
计数

1.4.持久化存储

Redis 提供 RDB 和 AOF 两种数据的持久化存储方案，解决内存数据库最担心
的万一 Redis 挂掉，数据会消失掉

## 2缺点

2.1. Redis 是内存数据库，所以，单台机器，存储的数据量，跟机器本身的内存大小。
虽然 Redis 本身有 Key 过期策略，但是还是需要提前预估和节约内存。如果内存增长过快，需要定期删除数据

	可使用 Redis Cluster、Codis 等方案，对 Redis 进行分区，从单机 Redis 变成集群 Redis
	
2.2.如果进行完整重同步，由于需要生成 RDB 文件，并进行传输，会占用主机的 CPU ，并会消耗现网的带宽

2.3.修改配置文件，进行重启，将硬盘中的数据加载进内存，时间比较久。在这个过程中，Redis 不能提供服务

# Redis 和 Memcached 的区别有哪些?

 1. Redis 支持复杂的数据结构
 
 2. Redis 原生支持集群模式
 
 Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据。
 
 3.性能对比
 
 Redis 只使用单核，而 Memcached 可以使用多核，所以平均每一个核上 Redis在存储小数据时比 Memcached 性能更高。
在 100k 以上的数据中，Memcached 性能要高于 Redis 。虽然 Redis 最近也在存储大数据的性能上进行优化，但是比起 Memcached，还是稍有逊色。

4.性能对比

简单的 Key-Value 存储的话，Memcached 的内存利用率更高，可以使用类似内存池。
如果 Redis 采用 hash 结构来做 key-value 存储，由于其组合式的压缩， 其内存利用率会高于 Memcached 。

5.网络 IO 模型

Memcached 是多线程，非阻塞 IO 复用的网络模型，原型上接近 Nignx 。
Redis 使用单线程的 IO 复用模型，自己封装了一个简单的 AeEvent 事件处理框架，主要实现了 epoll, kqueue 和 select ，更接近 Apache 早期的模式。

6.持久化存储

Memcached 不支持持久化存储，重启时，数据被清空。
Redis 支持持久化存储，重启时，可以恢复已持久化的数据。
 
 




















