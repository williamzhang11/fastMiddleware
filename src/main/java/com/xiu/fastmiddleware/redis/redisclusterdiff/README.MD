# redis的哨兵模式与集群模式比较


　　如果你的数据量很少，主要是承载高并发高性能的场景，比如你的缓存一般就几个G，单机足够了replication，

一个mater，多个slave，要几个slave跟你的要求的读吞吐量有关系，然后自己搭建一个sentinal集群，去保证redis主从架构的高可用性，就可以了

　　redis cluster，主要是针对海量数据+高并发+高可用的场景，海量数据，如果你的数据量很大，那么建议就用redis cluster
	