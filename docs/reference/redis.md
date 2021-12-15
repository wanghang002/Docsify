
## Redis的数据类

- String：最基本的数据类型，二进制安全（512M）。
```shell
Local_Redis:0>set name "redis"
"OK"
Local_Redis:0>get name
"redis"
Local_Redis:0>set name "memcache"
"OK"
Local_Redis:0>get name
"memcache"
Local_Redis:0>set count 1
"OK"
Local_Redis:0>get count
"1"
Local_Redis:0>incr count
"2"
```

- List：列表，按照String元素插入的顺序排序。
```shell
Local_Redis:0>LPUSH mylist aaa
"1"
Local_Redis:0>LPUSH mylist bbb
"2"
Local_Redis:0>LPUSH mylist ccc
"3"
Local_Redis:0>LRANGE mylist 0 10
 1)  "ccc"
 2)  "bbb"
 3)  "aaa"
Local_Redis:0>LPOP mylist
"ccc"
Local_Redis:0>LRANGE mylist 0 10
 1)  "bbb"
 2)  "aaa"
```

- Set：String元素组成的无序集合，通过哈希表实现，不允许重复。
```shell
Local_Redis:0>SADD myset 111
"1"
Local_Redis:0>SADD myset 222
"1"
Local_Redis:0>SADD myset 222
"0"
Local_Redis:0>SMEMBERS myset
 1)  "111"
 2)  "222"
```

- Sorted Set：通过分数来为集合中的成员进行从小到大的排序。
```shell
Local_Redis:0>ZADD myzset 3 abc
"0"
Local_Redis:0>ZADD myzset 3 abc
"1"
Local_Redis:0>ZADD myzset 1 abd
"1"
Local_Redis:0>ZADD myzset 2 abb
"1"
Local_Redis:0>ZADD myzset 2 abb
"0"
Local_Redis:0>ZADD myzset 1 bgg
"1"
Local_Redis:0>ZRANGEBYSCORE myzset 0 10
 1)  "abd"
 2)  "bgg"
 3)  "abb"
 4)  "abc"
```

- Hash：String元素组成的字典，适用于存储对象。
```shell
Local_Redis:0>HMSET wanghang name WangHang age 18 title Senior
"OK"
Local_Redis:0>HGET wanghang name
"WangHang"
Local_Redis:0>HGET wanghang age
"18"
Local_Redis:0>HSET wanghang age 20
"0"
Local_Redis:0>HGET wanghang age
"20"
Local_Redis:0>HGETALL wanghang
 1)  "name"
 2)  "WangHang"
 3)  "age"
 4)  "20"
 5)  "title"
 6)  "Senior"
```

## 从Redis里查询出某一固定前缀的Key
- 少量数据
 - KEYS pattern：查找所有符合给定模式pattern的key
>KEYS指令一次性返回所有匹配的key<br/>键的数量过大会使服务卡顿

```shell
Local_Redis:0>keys my*
 1)  "myset"
 2)  "mystream"
 3)  "mylist"
 4)  "myzset"
```
- 海量数据
 - SCAN cursor [MATCH pattern] [COUNT count]
>基于游标的迭代器，需要基于上一次的游标延续之前的迭代过程<br/>以0作为游标开始一次新的迭代，直到命令返回游标0完成一次遍历<br/>不保证每次执行都返回某个给定数量的元素，支持模糊查询<br/>一次返回的数量不可控，只能大概率符合count参数

```shell
Local_Redis:0>SCAN 0 MATCH my* COUNT 2
 1)  "6"
 2)    1)   "myset"
Local_Redis:0>SCAN 6 MATCH my* COUNT 2
 1)  "5"
 2)  null
Local_Redis:0>SCAN 5 MATCH my* COUNT 2
 1)  "7"
 2)    1)   "mystream"
 2)   "mylist"
```

## Redis为什么这么快
Redis的性能非常之高，每秒可以承受10W+的QPS，它如此优秀的性能主要取决于以下几个方面：
- 纯内存操作

数据存在内存中，绝大部分请求是纯粹的内存操作，非常快速，跟传统的磁盘文件数据存储相比，避免了通过磁盘IO读取到内存这部分的开销
- 数据结构简单

Redis中的数据结构是专门进行设计的，每种数据结构都有一种或多种数据结构来支持。Redis正是依赖这些灵活的数据结构，来提升读取和写入的性能。
- 采用单线程

省去了很多上下文切换的时间以及CPU消耗，不存在竞争条件，不用去考虑各种锁的问题，不存在加锁释放锁操作，也不会出现死锁而导致的性能消耗
- 使用IO多路复用技术

“多路”指的是多个网络连接，“复用”指的是复用同一个线程。

Redis可以在单线程中监听多个Socket的请求，在任意一个Socket可读/可写时，Redis去读取客户端请求，在内存中操作对应的数据，然后再写回到Socket中。整个过程非常高效，Redis利用了IO多路复用技术的事件驱动模型，保证在监听多个Socket连接的情况下，只针对有活动的Socket采取反应。

