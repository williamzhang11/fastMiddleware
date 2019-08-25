# zookeeper部署方式

Zookeeper 有两种部署方式：

1、单机
2、集群

Zookeeper 集群，是一个由多个 Server 组成，一个 Leader，多个 Follower。
（这个不同于我们常见的 Master/Slave 模式）Leader 为客户端服务器提供读写服务，
除了 Leader 外其他的机器只能提供读服务。

每个 Server 保存一份数据副本全数据一致，分布式读 Follower ，写由 Leader 实施更
新请求转发，由 Leader 实施更新请求顺序进行，来自同一个 Client 的更新请求按其发送顺
序依次执行数据更新原子性，一次数据更新要么成功，要么失败。

全局唯一数据视图，Client 无论连接到哪个 Server，数据视图都是一致的实时性，在一定事
件范围内，Client 能读到最新数据。
![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/image/zookeeper1.JPG)

