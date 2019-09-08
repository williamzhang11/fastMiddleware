# redis五大数据类型及底层实现

redis支持五种数据类型:string(字符串),hash(哈希),list(列表),set(集合),zset(有序集合)

## 1.1 String

是一个键值对,如果key存在则覆盖
一个key对应一个value

String 类型是二进制安全,redis的String可以包含任何数据.如图片,序列化对象

命令: set/get

场景:
	
	(1) 应用memcache的所有场景,字符串和数据直接存取.结构化数据需要先序列化,再set到value.get到value
	也需要反序列化
	
	(2) 可利用redis的incr,incrby,decr,decrby等指令来实现原子计数效果.可以实现业务上的统计计数需求
	也可以生成全局唯一id
	
	(3)存放session key, 实现一个分布式session系统.redis的key可以方便设置过期时间,用于实现session key的
	自动过期.验证skey时先根据uid路由到对应redis,如果取步到skey,表示skey已过期,需要重新登录;如果取到skey
	且校验通过,则升级此skey的过期时间即可
	
	(4)setnx,仅当key不存在时才set.可以用来选举master或实现分布式锁:所有client不断尝试使用setnx
	master myName抢注master,成功的那位不断使用expire刷新它的过期时间.如果master挂掉key就会失效,
	剩下的节点发送新一轮抢夺.
	
	(5)redis2.6开始支持lua脚本,可以实现分布式锁
	
	(6)getset设置新值,返回旧值.比如实现一个计数器可以用getset获取计数并重置为0
	

底层实现:
String再redis内部存储默认是一个字符串,当遇到incr,desr等操作时会转为数值型进行计算

## 1.2 hash

hset key field value

redis hash是一个键值对集合,如果field存在则覆盖

命令:hget,hset

场景:

	(1)存放结构化数据,如用户信息.对于用户信息比如用户的昵称,年龄,性别,积分等,redis的hash结构可以像
	在数据库中update一个属性只修改某一项属性值.
	key是用户id,value是一个map,这个map的key是成员的属性名,value是属性值
	
	(2)可用来建索引.如user对象,除了id有时还要按name查询,可以建一个key为user:name:id的hash
	在插入user对象时(set user:101{"id":101,"name":calvin}),顺便往这个hash插入一条
	(hset user:name:id calvin 101).按name查询的时候,用hget user:name:id calvin可以
	从名为calvin的key取出id.假设需要使用多种索引查找某条数据,一个hash key就可以,避免多个String
	key 存放索引值
	
	(3)hincrby 可以实现分布式id.相对String类型的idmaker 每个类型需要一个key,hash类型的用一个
	key即可
	
实现:
	redis hash对应的value内部时一个hashmap(哈希表),有2种不同实现.这个hash的成员较少时redis为了节省空间
	采用类似一维数组的方式紧凑存储,不会采用真正的hashmap结构,成员数量较大时会自动转成hashmap
	
	相对于对象序列化存储为string字符串类型，将对象存储在hash哈希类型中会占用更少的内存
	
	
	
	https://www.jianshu.com/p/28d4198085c8
	
	
# 1.3 list :

	list是一个双向链表,支持双向pop/push,一般是左端push,右端pop
	
	lpush students "zhangsan" "lisi" "wangwu" 
	
常用命令：lpush,rpush,lpop,rpop,lrange等

lpush redis.net.cn redis

场景:

	(1)各种列表,如twitter的关注列表,粉丝列表,最新消息排行,每篇文章的评论也可以用redis的list
	结构实现.
	
	(2)消息队列,可以利用Lists的push操作,将任务存在lists,然后工作线程再用pop操作将任务取出执行.这里的
	消息队列并没有ack机制,如果消费者把任务给pop走了又没处理完就死机了.
	解决方法1:多加一个sorted set分发时同时发到list与sorted set,以分发时间为score,用户把任务
	做完之后用zrem消掉sorted set里的job,并且定时从sorted set种取出没有完成的任务,重新放回list.


	(3)利用lrange可以很方便的实现list内容分页功能
	
	(4)取最新的N个数据的操作:LPUSH用来插入一个内容id,作为关键字存储在队列头部.LTRIM用来限制列表
	种的项目最多为5000.如果用户需要的检索数据量超越这个缓存容量,这时需要把请求发送到数据库
	
	
实现:redis list是一个双向链表即支持反向查找和遍历,操作方便,不过带来部分额外内存开销.
	
	
	
## 1.4 set

 是一种无需集合,集合中的元素没有先后顺序,不重复,重复的元素放入set会自动去重
 
 常用命令:
 
 sadd  key  value1  value2
 
sadd,spop,smembers,sunion等

场景:
	
	(1)某些需要去重列表
	
	(2)可存储一些集合性的数据,如再微博应用中,可将一个用户所有的关注人存在一个集合中,将其所有粉丝
	存在一个集合.Redis还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、
	共同喜好、二度好友等功能
	(3)想要知道某些特定的注册用户或ip地址,他们到底有多少访问了某个页面
	Add page:day1:<page_id> <user_id>
	
实现:
set的内部实现是一个value永远为null的hashmap,通过计算hash的方式快速排重.

## 1.5 zset

有序集合,相比set,元素放入集合中要提供该元素的分数,可根据分数自动排序

常用命令:

ZADD runoobkey 1 redis

zadd,zrange,zrem,zcard

场景:

(1)存放一个有序的不重复的集合列表,如twitter的public timeline可以以发表时间作为score存储,这样
获取时就是自动按时间排好序

(2)可以做带权重的队列,如普通消息的score为1,重用的score为2,然后工作线程可选择按score的倒序
获取工作任务.让重要任务有限进行

(3)排行榜相关: ZADD leaderboard <score> <username>.得到前100名高分用户很简单.
Zrevrange leaderboard 0 99.

(4) 新闻按用户投票和时间排序,zadd的score=points/time^alpha ,这样用户的投票会把相应大的新闻
挖出来,但时间会按照一定的指数将新闻埋下去

(5)过期项目处理:使用unix时间作为关键字,用来保持列表能够按时间排序.对current_time和time_to_live
进行检索,完成查找过期项目的艰巨任务.

实现方式:

redis sorted set的内部使用hashmap和跳跃表(skiplist)保证数据的存储和有序,
hashmap放的是成员到score的映射,而跳跃表存放的是所有成员,排序依据是hashmap存放score
,使用跳跃表结构可以获得比较高的查找效率.
	
	
	
	
	
	
	
	
	
	
	









