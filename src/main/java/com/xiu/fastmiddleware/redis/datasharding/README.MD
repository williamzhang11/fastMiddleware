# 数据分片三种方案

## 1.客户端实现数据分片


即客户端自己计算数据的key应该在哪个机器上存储和查找。

此方法的好处是降低了服务器集群的复杂度，客户端实现数据分片时，服务器是独立的，服务器之前没有任何关联。
多数redis客户端库实现了此功能，也叫sharding。

这种方式的缺点是客户端需要实时知道当前集群节点的联系信息，同时，当添加一个新的节点时，客户端要支持动
态sharding.，多数客户端实现不支持此功能，需要重启redis。另一个弊端是redis的HA需要额外考虑。

## 2.服务器实现数据分片

其理论是，客户端随意与集群中的任何节点通信，服务器端负责计算某个key在哪个机器上，当客户端访问某台机器时，
服务器计算对应的key应该存储在哪个机器，然后把结果返回给客户端，客户端再去对应的节点操作key，是一个重定向
的过程，此方式是redis3.0正在实现，目前处于beta版本， Redis 3.0的集群同时支持HA功能，某个master节
点挂了后，其slave会自动接管。

服务器端实现集群需要客户端语言实现服务器集群的协议，目前java，php，ruby语言多数有redis-cluster客户端
实现版本。 


## 3.通过代理服务器实现数据分片

此方式是借助一个代理服务器实现数据分片，客户端直接与proxy联系，proxy计算集群节点信息，并把请求发送到对应的集群节点。
降低了客户端的复杂度，需要proxy收集集群节点信息。Twemproxy是twitter开源的，实现这一功能的proxy。这个实现方式在
客户端和服务器之间加了一个proxy，但这是在redis 3.0稳定版本出来之前官方推荐的方式。结合redis-sentinel的HA方案，
是个不错的组合。




