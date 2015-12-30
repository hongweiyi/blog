title: Redis五大数据结构
tags:
  - NoSQL
  - Redis
  - 数据结构
id: 451
categories:
  - 技术分享
date: 2012-03-07 17:30:26
---

### 1、Redis介绍

Redis是REmote DIctionary Server的缩写，作者定位于一个内存KV存储数据库（In-memory key-value Store），让Redis自豪的并不是那每秒10K的读写速度，而是它那可以应对很多情况的数据结构，我这里就简单的介绍一下它五大数据结构，也可以方便的让自个翻翻API，并给以后翻阅源码打下一个基础。

<!--more-->

### 2、Strings

#### 1）简介

String是Redis最基本的数据结构，它的String是二进制安全的，即String中可以存放任意的二进制数据，比如说JPG图片、序列化对象等。String值长度最大可到512mb。

#### 2）结构定义

``` c
struct sdshdr{  
    long len;  
    long free;  
    char buf[];  
}
```

#### 3）支持命令

[APPEND](http://redis.io/commands/append)、[GET](http://redis.io/commands/get)、[GETBIT](http://redis.io/commands/getbit)、[GETRANGE](http://redis.io/commands/getrange)、[GETSET](http://redis.io/commands/getset)、[STRLEN](http://redis.io/commands/strlen)

[MGET](http://redis.io/commands/mget)、[MSET](http://redis.io/commands/mset)、[MSETNX](http://redis.io/commands/msetnx)、[SET](http://redis.io/commands/set)、[SETBIT](http://redis.io/commands/setbit)、[SETEX](http://redis.io/commands/setex)、[SETNX](http://redis.io/commands/setnx)、[SETRANGE](http://redis.io/commands/setrange)

[INCR](http://redis.io/commands/incr)、[INCRBY](http://redis.io/commands/incrby)、[DECR](http://redis.io/commands/decr)、[DECRBY](http://redis.io/commands/decrby)

### 3、Hashes

#### 1）简介

Hashes中存放了多个键值对（field/value），所以Hash结构可方便的表示一个对象。如：

`HMSET user:00001 username wikie password  gender male`

一个Hash可以存放2^32 – 1个键值对。Hash对象是用zipmap存储的，查找、删除均为O(n)，但一般来说对象的field对象不会大多，所以说操作评价还是近似O(1)。如果field/value的大小超过一定限制后，Redis会在内部自动将zipmap替换成正常的Hash实现，可在配置文件中指定：

hash-max-zipmap-entries 64 # 字段最多64个

hash-max-zipmap-value 512 # value最大为512字节

#### 2）结构定义

``` c
//Please check in dict.h  
typedef struct dictht {  
    dictEntry table;  
    unsigned long size;  
    unsigned long sizemask;  
    unsigned long used;  
} dictht;
```

#### 3）支持命令

[HDEL](http://redis.io/commands/hdel)、[HEXISTS](http://redis.io/commands/hexists)、[HGET](http://redis.io/commands/hget)、[HGETALL](http://redis.io/commands/hgetall)、[HINCRBY](http://redis.io/commands/hincrby)、[HKEYS](http://redis.io/commands/hkeys)、[HLEN](http://redis.io/commands/hlen)

[HMGET](http://redis.io/commands/hmget)、[HMSET](http://redis.io/commands/hmset)、[HSET](http://redis.io/commands/hset)、[HSETNX](http://redis.io/commands/hsetnx)、[HVALS](http://redis.io/commands/hvals)

### 4、Lists

#### 1）简介

Lists是一个简单的strings类型的双向链表，按照插入顺序排序。

最大长度支持2^32-1，可以通过命令从头部或者尾部添加删除元素，即可很方便的实现栈与队列操作。List还可以阻塞，很容易就实现了一个工作队列，而不用轮询。

#### 2）结构定义

``` c
// Check in adlist.h  
typedef struct listNode {  
    struct listNode *prev;  
    struct listNode *next;  
    void *value;  
} listNode;  

typedef struct listIter {  
    listNode *next;  
    int direction;  
} listIter;  

typedef struct list {  
    listNode *head;  
    listNode *tail;  
    void *(*dup)(void *ptr);  
    void (*free)(void *ptr);  
    int (*match)(void *ptr, void *key);  
    unsigned int len;  
} list;
```

#### 3）支持命令

[BLPOP](http://redis.io/commands/blpop) 、[BRPOP](http://redis.io/commands/brpop) 、[BRPOPLPUSH](http://redis.io/commands/brpoplpush)、[LINDEX](http://redis.io/commands/lindex)、[LINSERT](http://redis.io/commands/linsert)、[LLEN](http://redis.io/commands/llen)

[LPOP](http://redis.io/commands/lpop)、[LPUSH](http://redis.io/commands/lpush)、[LPUSHX](http://redis.io/commands/lpushx)、[LRANGE](http://redis.io/commands/lrange)、[LREM](http://redis.io/commands/lrem)、[LSET](http://redis.io/commands/lset)、[LTRIM](http://redis.io/commands/ltrim)

[RPOP](http://redis.io/commands/rpop)、[RPOPLPUSH](http://redis.io/commands/rpoplpush)、[RPUSH](http://redis.io/commands/rpush)、[RPUSHX](http://redis.io/commands/rpushx)

### 5、Sets

#### 1）简介

与数学的中的集合概念类似，没有重复的值，对其有添加删除操作，可对都个结合求交、并等操作，key理解为集合的名字。新浪微博中的：“我和她都关注了”只需要一个SINTER命令就可以实现。

Sets通过Hash Table实现，添加删除的时间复杂度均为O(n)，HashTable会随着添加或者删除自动调整大小。需要注意的是，调整HashTable大小需要同步（获取写锁）阻塞读写操作，后期可能会采用SkipList（无序如何使用SkipList？）实现。

和其它类型一样，最大支持2^32-1个元素。

#### 2）结构定义

与Hashes中的dict一致。

#### 3）支持的方法

[SADD](http://redis.io/commands/sadd)、[SCAR](http://redis.io/commands/scard)、[SDIFF](http://redis.io/commands/sdiff)、[SDIFFSTORE](http://redis.io/commands/sdiffstore)、[SINTER](http://redis.io/commands/sinter)、[SISMEMBER](http://redis.io/commands/sismember)

[SMEMBERS](http://redis.io/commands/smembers)、[SMOVE](http://redis.io/commands/smove)、[SPOP](http://redis.io/commands/spop)、[SRANDMEMBER](http://redis.io/commands/srandmember)、[SREM](http://redis.io/commands/srem)

[SUNION](http://redis.io/commands/sunion)、[SUNIONSTORE](http://redis.io/commands/sunionstore)

### 6、ZSets

#### 1）简介

ZSets为Set的升级版本，即排序的Sets，在Set的基础之上增加了顺序（Score）属性，每次插入均需要指定，且会自动重新调整值的顺序。Score为double类型，ZSets实现为SkipList与HashTable的混合体。

元素到Score的映射是添加在HashTable中的，所以给定一个元素获取Score开销为O(1)，Score到元素的映射则为SkipList。

#### 2）结构定义

``` c
/* ZSETs use a specialized version of Skiplists */
typedef struct zskiplistNode {  
    robj *obj;  
    double score;  
    struct zskiplistNode *backward;  
    struct zskiplistLevel {  
        struct zskiplistNode *forward;  
        unsigned int span;  
    } level[];  
} zskiplistNode;  

typedef struct zskiplist {  
    struct zskiplistNode *header, *tail;  
    unsigned long length;  
    int level;  
} zskiplist;  

typedef struct zset {  
    dict *dict;    // Value to Score  
    zskiplist *zsl;  // Score to Value  
} zset;
```

#### 3）支持命令

[ZADD](http://redis.io/commands/zadd)、[ZCARD](http://redis.io/commands/zcard)、[ZCOUNT](http://redis.io/commands/zcount)、[ZINCRBY](http://redis.io/commands/zincrby)、[ZINTERSTORE](http://redis.io/commands/zinterstore)

[ZRANGE](http://redis.io/commands/zrange)、[ZRANGEBYSCORE](http://redis.io/commands/zrangebyscore)、[ZRANK](http://redis.io/commands/zrank)、[ZREM](http://redis.io/commands/zrem)

[ZREMRANGEBYRANK](http://redis.io/commands/zremrangebyrank)、[ZREMRANGEBYSCORE](http://redis.io/commands/zremrangebyscore)、[ZREVRANGE](http://redis.io/commands/zrevrange)

[ZREVRANGEBYSCORE](http://redis.io/commands/zrevrangebyscore)、[ZREVRANK](http://redis.io/commands/zrevrank)、[ZSCORE](http://redis.io/commands/zscore)、[ZUNIONSTORE](http://redis.io/commands/zunionstore)

> 参考资料：
>
> [Redis.io](http://redis.io)
>
> [The Little Redis Book](http://openmymind.net/2012/1/23/The-Little-Redis-Book/)
