# redis数据结构实现

redis 有5种数据类型 分别是string，list，hash，set，zset

redis底层有6种数据结构，分别是简单动态字符串(SDS),链表，字典，跳跃表，整数集合，压缩列表。

每种数据类型都有着2种以上的数据结构实现，在不同状态下会进行数据结构的转换

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/image/redis.JPG)
## 简单动态字符串(SDS)---String

Redis 是用 C 语言写的，但是对于Redis的字符串，却不是 C 语言中的字符串（即以空字符’\0’结尾的字符数组），它是自己构建了一种名为 简单动态字符串（simple dynamic string,SDS）的抽象类型，并将 SDS 作为 Redis的默认字符串表示。

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/image/sds.JPG)

对于 SDS 数据类型的定义：

　　1、len 保存了SDS保存字符串的长度

　　2、buf[] 数组用来保存字符串的每个元素

　　3、free 记录了 buf 数组中未使用的字节数量
好处：
	(1)常数复杂度获取字符串长度
	(2)杜绝缓冲区溢出



## 字典  ---hash

字典又称为符号表或者关联数组、或映射（map），是一种用于保存键值对的抽象数据结构。字典中的每一个键 key 都是唯一的，通过 key 可以对值来进行查找或修改。C 语言中没有内置这种数据结构的实现，所以字典依然是 Redis自己构建的

Redis 的字典使用哈希表作为底层实现(类似hashmap,链地址法解决哈希冲突)

## 跳跃表 ---zset

跳表是一种随机化的数据结构	。它的效率和红黑树以及 AVL 树不相上下。它的效率和红黑树以及 AVL 树不相上下

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/image/skiplist.JPG)

跳表具有如下性质： 
(1) 由很多层结构组成 
(2) 每一层都是一个有序的链表 
(3) 最底层(Level 1)的链表包含所有元素 
(4) 如果一个元素出现在 Level i 的链表中，则它在 Level i 之下的链表也都会出现。 
(5) 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素。

1.搜索：从最高层的链表节点开始，如果比当前节点要大和比当前层的下一个节点要小，那么则往下找，也就是和当前层的下一层的节点的下一个节点进行比较，以此类推，一直找到最底层的最后一个节点，如果找到则返回，反之则返回空。

2.插入：首先确定插入的层数，有一种方法是假设抛一枚硬币，如果是正面就累加，直到遇见反面为止，最后记录正面的次数作为插入的层数。当确定插入的层数k后，则需要将新元素插入到从底层到k层。

3.删除：在各个层中找到包含指定值的节点，然后将节点从链表中删除即可，如果删除以后只剩下头尾两个节点，则删除这一层。

https://www.cnblogs.com/wangshaowei/p/9520403.html

https://www.cnblogs.com/ysocean/p/9080942.html


## 整数集合

　整数集合（intset）是Redis用于保存整数值的集合抽象数据类型，它可以保存类型为int16_t、int32_t 或者int64_t 的整数值，并且保证集合中不会出现重复元素。

```
typedef struct intset{
     //编码方式
     uint32_t encoding;
     //集合包含的元素数量
     uint32_t length;
     //保存元素的数组
     int8_t contents[];
 
}intset;

```

## 压缩列表

压缩列表（ziplist）是Redis为了节省内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型数据结构，一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。

压缩列表的原理：压缩列表并不是对数据利用某种算法进行压缩，而是将数据按照一定规则编码在一块连续的内存区域，目的是节省内存。

## 双端链表

C 语言内部是没有内置这种数据结构的实现，所以Redis自己构建了链表的实现


## 快速列表(quickList)  --List

　quickList 是 zipList 和 linkedList 的混合体，它将 linkedList 按段切分，每一段使用 zipList 来紧凑存储，多个 zipList 之间使用双向指针串接起来。

![image](https://github.com/williamzhang11/fastMiddleware/blob/master/src/main/java/com/xiu/fastmiddleware/image/quicklist.png)

## --- set 

底层使用了intset和hashtable两种数据结构存储的，intset我们可以理解为数组，hashtable就是普通的哈希表（key为set的值，value为null）





