<!-- GFM-TOC -->
* [目录](#目录)
* [Redis简介](#Redis简介)
  * [Redis与其他数据库和软件的对比](#Redis与其他数据库和软件的对比)
  * [Redis的持久化方法](#Redis的持久化方法)
  * [Redis的主从复制特性](#Redis主从复制特性)
    * [Redis中的字符串](#Redis中的字符串)
    * [Redis中的列表](#Redis中的列表)
    * [Redis中的集合](#Redis中的集合)
    * [Redis中的散列](#Redis中的散列)
    * [Redis的有序集合](#Redis的有序集合)  
* [Redis命令](#Redis命令)
  * [字符串](#字符串)
  * [列表](#列表)
  * [集合](#集合)
  * [散列](#散列)
  * [有序集合](#有序集合)
  * [发布与订阅](#发布与订阅)
  * [其他命令](#其他命令)
    * [排序](#排序)
    * [基本的Redis事务](#基本的Redis事务)
    * [键的过期时间](#键的过期时间)
<!-- GFM-TOC -->

# Redis简介

>Redis是一个速度非常块的非关系数据库，可以存储键与5种不同类型的值之间的映射，可以将存储在内存中的键值对数据持久化到硬盘，可以使用复制特性来扩展读写性能，还可以使用客户端分片来扩展写性能

## Redis与其他数据库和软件的对比

|名称|类型|数据存储选项|查询类型|附加功能|
|:-|:-|:-|:-|:-|
|Redis|使用内存存储的非关系数据库|字符串、列表、集合、散列表、有序集合|每种数据类型都自己的专属命令，另外还有批量操作(bulk operation)和不完全(partial)的事务支持|发布与订阅，主从复制(master/slave replication),持久化，脚本(存储过程,stored procedure)|
|memcached|使用内存存储的键值缓存|键值之间的映射|创建命令、读取命令、更新命令、删除命令以及其他几个命令|为提升性能而设的多线程服务器|
|MySQL|关系数据库|每个数据库可以包含多个表,每个表可以包含多个行;可以处理多个表的视图(View);支持空间(spatial)和第三方扩展|SELECT、INSERT、UPDATE、DELETE、函数、存储过程|支持ACID性质(需要使用InnoDB),主从复制和主主复制(master/master replication)|
|PostgreSQL|关系数据库|每个数据库可以包含多个表，每个表可以包含多个行；可以处理多个表的视图；支持空间和第三方扩展；支持可定制类型|SELECT、INSERT、UPDATE、DELETE、内置函数、自定义的存储过程|支持ACID性质、主从复制,由第三方支持的多主复制(multi-master replication)|
|MongoDB|使用硬盘存储和非关系文档存储|每个数据库可以包含多个表,每个表可以包含多个schema(schema-less)的BSON文档|创建命令、读取命令、更新命令、删除命令、条件查询命令等|支持map-reduce操作，主从复制，分片，空间索引(spatial index)|  

## Redis的持久化方法

* 时间点转储
     > 转储操作既可以在“指定时间段内有指定数量的写操作执行”这一条件被满足时执行,又可以通过调用两条转储到硬盘(dump-to-disk)命令中的任何一条来执行
* 写入只追加文件(append-only)文件
     > 将所有修改了数据库的命令都写入一个只追加(append-only)文件里面，用户可以根据数据的重要程度，将只追加写入设置为从不同步(sync)、每秒同步一次或者每写入一个命令就同步一次。

## Redis主从复制特性

>执行复制的从服务器会连接上主服务器，接受主服务器发送的整个数据库的初始副本;之后主服务器执行的写命令,都会被发送给所有连接着的从服务器去执行,从而实时地更新从服务器的数据集。因为从服务器包含的数据会不断进行更新，所以客户端可以向任意一个从服务器发送读请求，以此来避免对主服务器进行集中式的访问。

## Redis数据结构简介

Redis提供了五种不同的数据结构类型之间的映射，这五种数据结构类型分别为STRING(字符串)、LIST(列表)、SET(集合)、HASH(散列)和ZSET(有序集合)。有一部分Redis命令对这5种结构都是通用的，如DEL、TYPE、RENAME等;但也有一部分Redis命令只能对特定的一种或者两种结构使用。

|结构类型|结构存储的值|结构的读写能力|
|:-|:-|:-|
|STRING|可以是字符串、整数或者浮点数|对整个字符串或者字符串的其中一部分执行操作;对整数和浮点数执行自增(increment)或者自减(decrement)操作|
|LIST|一个链表,链表上的每个节点都包含了一个字符串|从链表的两端推入或者弹出元素;根据偏移量对链表进行修剪(trim);读取单个或者多个元素;根据值查找或者移除元素|
|SET|包含字符串的无序收集器(unordered collection),并且被包含的每个字符串都是独一无二、各不相同的|添加、获取、移除单个元素;检查一个元素是否存在于集合中;计算交集、并集、差集;从集合里面随机获取元素|
|HASH|包含键值对的无序散列表|添加、获取、移除单个元素;检查一个元素是否存在于集合当中;计算交集、并集、差集;从集合中随机获取元素|
|ZSET|字符串成员与浮点数分值之间的有序映射,元素的排列顺序由分值大小决定|添加、获取、删除单个元素;根据分值范围或者成员来获取元素|

#### Redis中的字符串

>常用字符串命令

|命令|行为|
|:-|:-|
|GET|获取存储在给定键中的值|
|SET|设置存储在给定键中的值|
|DEL|删除存储在给定键中的值(该命令可以用于所有类型)|

#### Redis中的列表

>常用列表命令

|命令|行为|
|:-|:-|
|RPUSH|将给定值推入列表的右端|
|LPUSH|将给定值推入列表的左端|
|LPOP|从列表的左侧弹出一个值,并返回被弹出的值|
|RPOP|从列表的右侧弹出一个值,并返回被弹出的值|
|LRANGE|获取列表在给定范围上的所有值|
|LINDEX|获取列表在给定位置上的一个元素|

#### Redis中的集合

>常用集合命令

|命令|行为|
|:-|:-|
|SADD|将给定元素添加到集合|
|SMEMBERS|返回集合包含的所有元素|
|SISMEMBER|检查给定元素是否存在于集合中|
|SREM|如果给定的元素存在于集合中,那么移除这个元素|

#### Redis中的散列

>常用散列命令

|命令|行为|
|:-|:-|
|HSET|在散列里面关联起给定的键值对|
|HGET|获取指定散列键的值|
|HGETALL|获取散列包含的所有键值对|
|HDEL|如果给定键存在于散列里面，那么移除这个键|

#### Redis的有序集合
 
>常用有序集合命令

|命令|行为|
|:-|:-|
|ZADD|将一个带有给定分值的成员添加到有序集合里面|
|ZRANGE|根据元素在有序排列中所处的位置,从有序集合里面获取多个元素|
|ZRANGEBYSCORE|获取有序集合在给定分值范围内的所有元素|
|ZREM|如果给定成员存在于有序集合,那么移除这个成员|

# Redis命令

## 字符串
Redis的字符串就是一个由字节组成的序列,在Redis里面,字符串可以存储以下三种类型的值:

>> * 字节串(byte string)
>> * 整数
>> * 浮点数

> 用户可以通过给定一个任意的数值，对存储着整数或者浮点数的字符串执行自增(increment)或者自减(decrement)操作，在有需要的时候,Redis还会将整数转换成浮点数。整数的取值范围和系统的长整数(long integer)的取值范围相同(在32位系统上，整数就是32位有符号整数,在64位系统上，整数就是64位有符号整数),而浮点数的取值范围和精度则与IEEE 754标准的双精度浮点数(double)相同。Redis明确的区分字节串、整数和浮点数的做法是一种优势，比起只能够存储字节串的做法,Redis的做法在数据表现方面具有强大的灵活性。

对Redis执行自增和自减操作的命令

|命令|用例|描述|
|:-|:-|:-|
|INCR|INCR key-name|将键存储的值加上1|
|DECR|DECR key-name|将键存储的值减去1|
|INCRBY|INCRBY key-name amount|将键存储的值加上整数amount|
|DECRBY|DECRBY key-name amount|将键存储的值减去整数amount|
|INCRBYFLOAT|INCRBYFLOAT key-name amount|将键存储的值加上浮点数amount,redis版本2.6或以上可用|

> 当用户将一个值存储到Redis字符串中时，如果该值可以被解释成十进制或者是浮点数，那么Redis会察觉到这一点，并允许用户对这个字符串执行各种INCR和DECR操作，若对一个不存在的键或者保存了空串的键执行自增或者自减的操作，那么Redis在执行时会将该键的值当做是0来处理。如果用户尝试对一个值无法解释为整数或者浮点数的字符串键执行自增或者自减操作，那么Redis将向用户返回一个错误。

供Redis处理子串和二进制位的命令

|命令|用例|描述|
|:-|:-|:-|
|APPEND|APPEND key-name value|将值value追加到给定键key-name当前存储的值得末尾|
|GETRANGE|GETRANGE key-name start end|获取一个由偏移量start至偏移量end范围内所有字符组成的子串，包括start和end|
|SETRANGE|SETRANGE key-name offset value|将从offset偏移量开始的子串设置为给定值|
|GETBIT|GETBIT key-name offset|将字符串看做是二进制位串，并返回位串中偏移量为offset的二进制位的值|
|SETBIT|SETBIT key-name offset value|将字符串看作是二进制位串，并将位串中偏移量为offset的二进制位的值设置为value|
|BITCOUNT|BITCOUNT key-name [start end]|统计二进制位串里面值为1的二进制位的数量，如果给定了可选的start偏移量和end偏移量，那么只对偏移量指定范围内的二进制位进行统计|
|BITOP|BITOP operation dest-key key-name [key-name ...]|对一个或多个二进制位串执行包括并(AND)、或(OR)、异或(XOR)、非(NOT)在内的任意一种按位运算操作，并将计算得到的结果保存在dest-key键里面|

在使用SETBIT或者SETRANGE对字符串进行写入的时候,如果字符串长度不满足写入的要求,Redis会用空字节(null)将字符串扩展至所需的长度,然后才执行写入或者更新操作。在使用GETRANGE读取字符串的时候,超出字符串末尾的数据会被视为是空串,而在使用GETBIT读取二进制位串的时候,超出字符串末尾的二进制位会被视为是0.

## 列表

> Redis列表允许用户从序列的两端推入或者弹出元素,获取列表元素,以及执行各种常见的列表操作。

> 一些常用的列表命令

|命令|用例|描述|
|:-|:-|:-|
|RPUSH|RPUSH key-name value [value ...]|将一个或多个值推入列表的右端,并返回列表当前的长度|
|LPUSH|LPUSH key-name value [value ...]|将一个或多个值推入列表的左端,并返回列表当前的长度|
|RPOP|RPOP key-name|移除并返回列表最右端的元素|
|LPOP|LPOP key-name|移除并返回列表最左端的元素|
|LINDEX|LINDEX key-name offset|返回列表中偏移量为offset的元素|
|LRANGE|LRANGE key-name start end|返回列表从start偏移量到end偏移量范围内的所有元素,其中偏移量为start和偏移量为end的元素也会包含在被返回的元素之内|
|LTRIM|LTRIM key-name end|对列表进行修剪,只保留从start偏移量到end偏移量范围内的元素,其中偏移量为start和偏移量为end的元素也会被保留|
|RPOPLPUSH|RPOPLPUSH source-key dest-key|从source-key列表中弹出位于最右端的元素,然后将这个元素推入dest-key列表的最左端,并向用户返回这个元素|

> 阻塞式的列表弹出命令以及在列表之间移动元素的命令

|命令|用例|描述|
|:-|:-|:-|
|BLPOP|BLPOP key-name [key-name ...] timeout|从第一个非空列表中弹出位于最左端的元素,或者在timeout秒之内阻塞并等待可弹出的元素出现|
|BRPOP|BRPOP key-name [key-name ...] timeout|从第一个非空列表中弹出位于最右端的元素,或者在timeout秒之内阻塞并等待可弹出的元素出现|
|BRPOPLPUSH|BRPOPLPUSH source-key dest-key timeout|从source-key列表中弹出位于最右端的元素,然后将这个元素推入dest-key列表的最左端,并向用户返回这个元素；如果source-key为空，那么timeout秒之内阻塞并等待可弹出的元素出现|

## 集合

> 一些常用的集合命令

|命令|用例|描述|
|:-|:-|:-|
|SADD|SADD key-name item [item ...]|将一个或多个元素添加到集合里面,并返回被添加元素当中原本并不存在于集合里面的元素数量|
|SREM|SREM key-name item [item ...]|从集合里面移除一个或者多个元素，并返回被移除元素的数量|
|SISMEMBER|SISMEMBER key-name item|检查元素item是否存在于集合key-name里|
|SCARD|SCARD key-name|返回集合包含的元素的数量|
|SMEMBERS|SMEMBERS key-name|返回集合包含的所有元素|
|SRANDMEMBER|SRANDMEMBER key-name [count]|从集合中随机的返回一个或多个元素。当count为正数时，命令返回的随机元素不会重复；当count为负数时，命令返回的随机元素可能会出现重复|
|SPOP|SPOP key-name|随机地移除集合中的一个元素，并返回被移除的元素|
|SMOVE|SMOVE source-key dest-key item|如果集合source-key包含元素item，那么从集合source-key里面移除元素item，并将元素item添加到集合dest-key中；如果item被成功移除，那么命令返回1，否则返回0|

> 用于组合和处理多个集合的Redis命令

|命令|用例|描述|
|:-|:-|:-|
|SDIFF|SDIFF key-name [key-name ...]|返回存在于第一个集合、但不存在于其他集合中的元素(数学上的差集运算)|
|SDIFFSTORE|SDIFFSTORE dest-key dest-name [key-name ...]|将存在于第一个集合但并不存在于其他集合中的元素存储到dest-key键里面|
|SINTER|SINTER key-name [key-name ...]|返回那些同时存在于所有集合中的元素(数学上的交集运算)|
|SINTERSTORE|SINTERSTORE dest-key key-name [key-name ...]|将那些同时存在于所有集合中的元素存储到dest-key键里面|
|SUNION|SUNION key-name [key-name ...]|返回那些至少存在于一个集合中的元素(数学上的并集计算)|
|SUNIONSTORE|SUNIONSTORE dest-key key-name [key-name ...]|将那些至少存在于一个集合中的元素存储到dest-key键里面|

## 散列

> 用于添加和删除键值对的散列操作

|命令|用例|描述|
|:-|:-|:-|
|HMGET|HMGET key-name key [key ...]|从散列里面获取一个或多个键的值|
|HMSET|HMSET key-name key value [key value ...]|为散列里面的一个或多个键设置值|
|HDEL|HDEL key-name key [key ...]|删除散列里面的一个或多个键值对,返回成功找到并删除的键值对数量|
|HLEN|HLEN key-name|返回散列包含的键值对数量|

> 展示Redis散列的更高级特性

|命令|用例|描述|
|:-|:-|:-|
|HEXISTS|HEXISTS key-name key|检查给定键是否存在于散列中|
|HKEYS|HKEYS key-name|获取散列包含的所有键|
|HVALS|HVALS key-name|获取散列包含的所有值|
|HGETALL|HGETALL key-name|获取散列包含的所有键值对|
|HINCRBY|HINCRBY key-name key increment|将键key存储的值加上整数increment|
|HINCRBYFLOAT|HINCRBYFLOAT key-name key increment|将键key存储的值加上浮点数increment|

## 有序集合

> 一些常用的有序集合命令

|命令|用例|描述|
|:-|:-|:-|
|ZADD|ZADD key-name score member [score member ...]|将带有给定分值的成员添加到有序集合里面|
|ZREM|ZREM key-name member [member ...]|从有序集合里面移除给定的成员,并返回被移除成员的数量|
|ZCARD|ZCARD key-name|返回有序集合包含的成员数量|
|ZINCRBY|ZINCRBY key-name increment member|将member成员的分值加上increment|
|ZCOUNT|ZCOUNT key-name min max|返回分值介于min和max之间的成员数量|
|ZRANK|ZRANK key-name member|返回成员member在有序集合中的排名|
|ZSCORE|ZSCORE key-name member|返回成员member的分值|
|ZRANGE|ZRANGE key-name start stop [WITHSCORES]|返回有序集合中排名介于start和stop之间的成员,如果给定了可选的WITHSCORES选项,那么命令会将成员的分值也一并返回|

> 有序集合的范围型数据获取命令和范围型数据删除命令，以及并集命令和交集命令

|命令|用例|描述|
|:-|:-|:-|
|ZREVRANK|ZREVRANK key-name member|返回有序集合里成员member的排名,成员按照分值从大到小排列|
|ZREVRANGE|ZREVRANGE key-name start stop [WITHSCORES]|返回有序集合给定排名范围内的成员，成员按照分值从大到小排列|
|ZRANGEBYSCORE|ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]|返回有序集合中,分值介于min和max之间的所有成员|
|ZREMRANGEBYRANK|ZREMRANGEBYRANK key-name start stop|移除有序集合中排名介于start和stop之间的所有成员|
|ZREMRANGEBYSCORE|ZREMRANGEBYSCORE key-name min max|移除有序集合中分值介于min和max之间的所有成员|
|ZINTERSTORE|ZINTERSTORE dest-key key-count key [key ...][WEIGHTS weight [weight ...]][AGGREGATE SUM/MIN/MAX]|对给定的有序集合执行类似于集合交集运算|
|ZUNIONSTORE|ZUNIONSTORE dest-key key-count key [key ...][WEIGHTS weight [weight ...]][AGGREGATE SUM/MIN/MAX]|对给定的有序集合执行类似于集合并集运算|

> * WEIGHTS：使用使用 WEIGHTS 选项，你可以为 每个 给定有序集 分别 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 score 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子
> * AGGREGATE：使用 AGGREGATE 选项，你可以指定并集的结果集的聚合方式。默认使用的参数 SUM ，可以将所有集合中某个成员的 score 值之 和 作为结果集中该成员的 score 值；使用参数 MIN ，可以将所有集合中某个成员的 最小 score 值作为结果集中该成员的 score 值；而参数 MAX 则是将所有集合中某个成员的 最大 score 值作为结果集中该成员的 score 值。

## 发布与订阅     

> Redis提供了5个发布和订阅命令

|命令|用例|描述|
|:-|:-|:-|
|SUBSCRIBE|SUBSCRIBE channel [channel ...]|订阅给定的一个或多个频道|
|UNSUBSCRIBE|UNSUBSCRIBE [channel [channel ...]]|退订给定的一个或多个频道,如果执行时没有给定任何频道,那么退订所有频道|
|PUBLISH|PUBLISH channel message|给定频道发送消息|
|PSUBSCRIBE|PSUBSCRIBE pattern [pattern ...]|订阅与给定模式相匹配的所有频道|
|PUNSUBSCRIBE|PUNSUBSCRIBE [pattern [pattern ...]]|退订给定的模式,如果执行时没有给定任何模式,那么退订所有模式|

> 对于旧版Redis来说,如果一个客户端订阅了某个或某些频道,但是它读取消息的速度不够快的话,那么不断积压的消息就会使得Redis输出缓冲区的体积变得越来越大,这可能会导致Redis的速度变慢,甚至直接崩溃。也可能会导致Redis被操作系统强制杀死,甚至导致操作系统本身不可用。新版的Redis不会出现这种问题,因为它会自动断开不符合client-output-buffer-limit pubsub配置选项要求的订阅客户端

> 任何网络系统在执行操作时都可能会遇上断线情况,而断线产生的连接错误通常会使网络连接两端中的其中一端进行重新连接。但是如果客户端在执行订阅操作的过程中断线,那么客户端将丢失在断线期间发送的所有消息,因此依靠频道来接收消息的用户可能会对Redis提供的PUBLISH命令和SUBSCRIBE命令的语义感到失望。

## 其他命令

### 排序

Redis的排序操作和其他编程语言和排序操作一样，都可以根据某种比较规则对一系列元素进行有序排列。SORT可以看做是SQL语言中的order by。

>SORT命令的定义

|命令|用例|描述|
|:-|:-|:-|
|SORT|SORT source-key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC/DESC] [ALPHA] [STORE dest-key]|根据给定的选项,对输入列表、集合或者有序集合进行排序,然后返回或者存储排序的结果|

### 基本的Redis事务

Redis的基本事务需要用到MULTI命令和EXEC命令,这种事务可以让一个客户端在不被其他客户端打断的情况下执行多个命令。和关系数据库那种可以在执行的过程中进行回滚的事务不同,在Redis里面,被MULTI命令和EXEC命令包围的所有命令会一个接一个地执行,直到所有命令都执行完毕为止。当一个事务执行完毕之后,Redis才会处理其他客户端的命令。
要在Redis中执行事务,我们首先需要执行MULTI命令,然后输入那些我们想要在事务,然后再执行EXEC命令。当Redis从一个客户端那里接收到MULTI命令时,Redis会将这个客户端之后发送的所有命令都放入到一个队列里面,直到这个客户端发送EXEC命令为止,然后Redis就会在不被打断的情况下,一个接一个地执行存储在队列里面的命令。 

### 键的过期时间

在使用Redis存储数据的时候,有些数据可能在某个时间点之后就不再有用了,用户可以使用DEL命令显示地删除这些无用的数据,也可以通过Redis的过期时间特性来让一个键在给定的时限之后自动被删除。在常用的命令中,只有少数的几个命令可以原子的为键设置过期时间,并且对于列表、集合、散列和有序集合这样的容器来说,键过期命令只能为整个键设置过期时间,而没办法为键里面的单个元素设置过期时间。

> 用于处理过期时间的Redis命令

|命令|示例|描述|
|:-|:-|:-|
|PERSIST|PERSIST key-name|移除键的过期时间|
|TTL|TTL key-name|查看给定键距离过期还有多少秒|
|EXPIRE|EXPIRE key-name|让给定键在指定的秒数之后过期|
|EXPIREAT|EXPIREAT key-name timestamp|将给定键的过期时间设置为给定的UNIX时间戳|
|PTTL|PTTL key-name|查看给定键距离过期时间还有多少毫秒,在Redis2.6或以上版本可用|
|PEXPIRE|PEXPIRE key-name milliseconds|让给定键在指定的毫秒之后过期,在Redis2.6或以上版本可用|
|PEXPIREAT|PEXPIREAT key-name timestamp-milliseconds|将一个毫秒级精度的UNIX时间戳设置为给定键的过期时间,这个命令在Redis2.6或以上版本可用|
