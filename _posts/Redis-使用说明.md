---
title: Redis-使用说明
date: 2017-07-22 10:18:36
tags: Redis
categories: tool 
---

# 常用命令

## 安装Redis
win:官网下载，直接安装。
linux:  
```shell
wget http://download.redis.io/releases/redis-2.8.17.tar.gz
tar xzf redis-2.8.17.tar.gz
cd redis-2.8.17
make
```

<!--more-->

ubuntu:  
```shell
sudo apt-get update
sudo apt-get install redis-server
```

## 运行服务器
win:`redis-server.exe redis.windows.conf`
linux:`./redis-server redis.conf`
Ubuntu:`redis-server`
mac:`redis-server`


## 运行redis客户端
win:`redis-cli.exe -h 127.0.0.1 -p 6379`
linux:`./redis-cli`
Ubuntu:`redis-cli`
Ubuntu:`redis-cli`

# 配置设置
## `CONFIG GET CONFIG_SETTING_NAME`
通过 CONFIG 命令查看或设置配置项,使用`*`号获取所有配置项

## `CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE`
修改配置使用 CONFIG set 命令来修改配置，或者修改 redis.conf 文件

## 配置内容详细介绍：
redis.conf 配置项说明如下：

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程  
`daemonize no`  

2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定  
`pidfile /var/run/redis.pid`  

3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字  
`port 6379`  

4. 绑定的主机地址  
`bind 127.0.0.1`  

5. 当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能  
`timeout 300`  

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose  
`loglevel verbose`

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null  
`logfile stdout`  

8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id  
`databases 16`

9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合  
`save <seconds> <changes>`
Redis默认配置文件中提供了三个条件：
```
save 900 1
save 300 10
save 60 10000
```
分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。

10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大  
`rdbcompression yes`

11. 指定本地数据库文件名，默认值为dump.rdb  
`dbfilename dump.rdb`

12. 指定本地数据库存放目录  
`dir ./`  

13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步  
`slaveof <masterip> <masterport>`  

14. 当master服务设置了密码保护时，slav服务连接master的密码  
`masterauth <master-password>`  

15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过`AUTH <password>`命令提供密码，默认关闭  
`requirepass foobared`  

16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置`maxclients 0`，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回`max number of clients reached`错误信息  
`maxclients 128`   

17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
`maxmemory <bytes>`  

18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no  
`appendonly no`  

19. 指定更新日志文件名，默认为appendonly.aof  
`appendfilename appendonly.aof`  

20. 指定更新日志条件，共有3个可选值：
* no:表示等操作系统进行数据缓存同步到磁盘（快）  
* always:表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）  
* everysec:表示每秒同步一次（折衷，默认值）  
`appendfsync everysec`  

21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）  
`vm-enabled no`  

22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享  
`vm-swap-file /tmp/redis.swap`  

23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0  
`vm-max-memory 0`  

24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值  
`vm-page-size 32`   

25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，在磁盘上每8个pages将消耗1byte的内存。  
`vm-pages 134217728`  

26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4  
`vm-max-threads 4`  

27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启  
`glueoutputbuf yes`  

28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法  
`hash-max-zipmap-entries 64`  
`hash-max-zipmap-value 512`  

29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）  
`activerehashing yes`  

30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件  
`include /path/to/local.conf`  

# 数据类型
## string (字符串)
string类型是二进制安全的。redis的string可以包含任何数据。比如j图片.string类型是Redis最基本的数据类型，一个键最大能存储512MB。

### 设置key的值
`SET key "value"`

### 获取key的值
`GET key`  

### 截取返回字符
`GETRANGE key start end`

### 设置key的值并返回old value
`GETSET key value`

### 获取指定偏移量
`GETBIT key offset`

### 获取所有key的值
`MGET key1 [key2..]`

### 设置或清除指定偏移量
`SETBIT key offset value`

### 过期时间
`SETEX key seconds value`

### key不存在时设置
`SETNX key value`

### 覆写key储存的字符串值
`SETRANGE key offset value`

### 字符串值的长度
`STRLEN key`

### 设置多个key-value
`MSET key value [key value ...]`

### 设置多个不存在key-value
`MSETNX key value [key value ...]`

### 增加毫秒生存时间
`PSETEX key milliseconds value`

### value+1
`INCR key`

### value 加上给定的增量值
`INCRBY key increment`

### value 加上浮点增量值
`INCRBYFLOAT key increment`
### value-1
`DECR key`
### value减去给定的减量值
`DECRBY key decrement`
### 追加
`APPEND key value`

## hash (哈希)
Redis hash 是一个键名对集合。  
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。 
每个 hash 可以存储 2^32 -1 键值对（40多亿）。  

### 删除多个value
`HDEL key field2 [field2]`

### 是否存在
`HEXISTS key field`

### 获取字段的值
`HGET key field`

### 获取所有的field-value
`HGETALL key`
### 整数值加上增量
`HINCRBY key field increment`
### 浮点数值加上增量
`HINCRBYFLOAT key field increment`
### 获取所有field
`HKEYS key`
### 获取字段的数量
`HLEN key`
### 获取字段的值
`HMGET key field1 [field2]`
### 设置多个field-value
`HMSET key field1 value1 [field2 value2 ]`
### 设置一个field-value
`HSET key field value`
### field不存在时设置值
`HSETNX key field value`
### 获取所有value
`HVALS key`
### 迭代键值对
`HSCAN key cursor [MATCH pattern] [COUNT count]`


## list (列表)
Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。  
列表最多可存储 2^32 - 1 元素 (4294967295, 每个列表可存储40多亿)。  

### `BLPOP key1 [key2 ] timeout`
移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
### `BRPOP key1 [key2 ] timeout`
移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
### `BRPOPLPUSH source destination timeout`
从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

### 通过索引获取元素
`LINDEX key index`

### 在元素前或者后插入元素
`LINSERT key BEFORE|AFTER pivot value`
### 获取列表长度
`LLEN key`
### 移出并获取第一个元素
`LPOP key`
### 列表头部插入
`LPUSH key value1 [value2]`
### 列表头部插入一个值
`LPUSHX key value`
### 获取列表范围内的元素
`LRANGE key start stop`
### 移除列表元素
`LREM key count value`
### 通过索引设置元素的值
`LSET key index value`
### 缩短key list的值
`LTRIM key start stop`
### 移除并获取最后一个元素
`RPOP key`
### `RPOPLPUSH source destination`
移除列表的最后一个元素，并将该元素添加到另一个列表并返回
### 添加多个值
`RPUSH key value1 [value2]`
### 已存在的列表添加值
`RPUSHX key value`





## set (集合)
Redis的Set是string类型的无序集合。  
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。  
集合中最大的成员数为 2^32 - 1(4294967295, 每个集合可存储40多亿个成员)。 
### 添加多个成员
`SADD key member1 [member2]`

### 集合的成员数
`SCARD key`

### 集合的差集
`SDIFF key1 [key2]`
### 所有集合的差集并存储在 destination 中
`SDIFFSTORE destination key1 [key2]`
### 集合的交集
`SINTER key1 [key2]`
### 集合的交集并存储在 destination 中
`SINTERSTORE destination key1 [key2]`
### 判断元素是否在key集合中
`SISMEMBER key value`
### 返回所有成员
`SMEMBERS key`
### 将元素从key集合移动到key2集合
`SMOVE key key2 value`
### 移除并返回一个随机元素
`SPOP key`
### 返回集合多个随机数
`SRANDMEMBER key [count]`
### 移除集合多个成员
`SREM key member1 [member2]`
### 返回所有集合的并集
`SUNION key1 [key2]`
### 所有给定集合的并集存储在 destination 集合中
`SUNIONSTORE destination key1 [key2]`
### 迭代集合中的元素
`SSCAN key cursor [MATCH pattern] [COUNT count]`



## zset (sorted set 有序集合)
Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。  
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。  
zset的成员是唯一的,但分数(score)却可以重复。  


### `ZADD key score1 member1 [score2 member2]`
向有序集合添加一个或多个成员，或者更新已存在成员的分数
### `ZCARD key`
获取有序集合的成员数
### `ZCOUNT key min max`
计算在有序集合中指定区间分数的成员数
### `ZINCRBY key increment member`
有序集合中对指定成员的分数加上增量 increment
### `ZINTERSTORE destination numkeys key [key ...]`
计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
### `ZLEXCOUNT key min max`
在有序集合中计算指定字典区间内成员数量
### `ZRANGE key start stop [WITHSCORES]`
通过索引区间返回有序集合成指定区间内的成员
### `ZRANGEBYLEX key min max [LIMIT offset count]`
通过字典区间返回有序集合的成员
### `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]`
通过分数返回有序集合指定区间内的成员
### `ZRANK key member`
返回有序集合中指定成员的索引
### `ZREM key member [member ...]`
移除有序集合中的一个或多个成员
### `ZREMRANGEBYLEX key min max`
移除有序集合中给定的字典区间的所有成员
### `ZREMRANGEBYRANK key start stop`
移除有序集合中给定的排名区间的所有成员
### `ZREMRANGEBYSCORE key min max`
移除有序集合中给定的分数区间的所有成员
### `ZREVRANGE key start stop [WITHSCORES]`
返回有序集中指定区间内的成员，通过索引，分数从高到底
### `ZREVRANGEBYSCORE key max min [WITHSCORES]`
返回有序集中指定分数区间内的成员，分数从高到低排序
### `ZREVRANK key member`
返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
### `ZSCORE key member`
返回有序集中，成员的分数值
### `ZUNIONSTORE destination numkeys key [key ...]`
计算给定的一个或多个有序集的并集，并存储在新的 key 中
### `ZSCAN key cursor [MATCH pattern] [COUNT count]`
迭代有序集合中的元素（包括元素成员和元素分值）

## HyperLogLog 结构
Redis 在 2.8.9 版本添加了 HyperLogLog 结构。  
Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。  
在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。  
但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。   

### `PFADD key element [element ...]`
添加指定元素到 HyperLogLog 中。
### `PFCOUNT key [key ...]`
返回给定 HyperLogLog 的基数估算值。
### `PFMERGE destkey sourcekey [sourcekey ...]`
将多个 HyperLogLog 合并为一个 HyperLogLog 

# 常用操作
## 连接命令
### `ping `
该命令用于检测 redis 服务是否启动。返回pong。查看服务是否运行
### `redis-cli -h host -p port -a password`
在远程 redis 服务上执行命令
### `AUTH password`
验证密码是否正确
### `ECHO message`
打印字符串
### `QUIT`
关闭当前连接
### `SELECT index`
切换到指定的数据库

## 客户端连接命令
### `CLIENT LIST`
返回连接到 redis 服务的客户端列表
### `CLIENT SETNAME`
设置当前连接的名称
### `CLIENT GETNAME`
获取通过 CLIENT SETNAME 命令设置的服务名称
### `CLIENT PAUSE`
挂起客户端连接，指定挂起的时间以毫秒计
### `CLIENT KILL`
关闭客户端连接

## 服务器命令
### `BGREWRITEAOF`
异步执行一个 AOF（AppendOnly File） 文件重写操作
### `BGSAVE`
在后台异步保存当前数据库的数据到磁盘
### `CLIENT KILL [ip:port] [ID client-id]`
关闭客户端连接
### `CLIENT LIST`
获取连接到服务器的客户端连接列表
### `CLIENT GETNAME`
获取连接的名称
### `CLIENT PAUSE timeout`
在指定时间内终止运行来自客户端的命令
### `CLIENT SETNAME connection-name`
设置当前连接的名称
### `CLUSTER SLOTS`
获取集群节点的映射数组
### `COMMAND`
获取 Redis 命令详情数组
### `COMMAND COUNT`
获取 Redis 命令总数
### `COMMAND GETKEYS`
获取给定命令的所有键
### `TIME`
返回当前服务器时间
### `COMMAND INFO command-name [command-name ...]`
获取指定 Redis 命令描述的数组
### `CONFIG GET parameter`
获取指定配置参数的值
### `CONFIG REWRITE`
对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写
### `CONFIG SET parameter value`
修改 redis 配置参数，无需重启
### `CONFIG RESETSTAT`
重置 INFO 命令中的某些统计数据
### `DBSIZE`
返回当前数据库的 key 的数量
### `DEBUG OBJECT key`
获取 key 的调试信息
### `DEBUG SEGFAULT`
让 Redis 服务崩溃
### `FLUSHALL
删除所有数据库的所有key`
### `FLUSHDB
删除当前数据库的所有key`
### `INFO [section]`
获取 Redis 服务器的各种信息和统计数值
### `LASTSAVE`
返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示
### `MONITOR`
实时打印出 Redis 服务器接收到的命令，调试用
### `ROLE`
返回主从实例所属的角色
### `SAVE`
异步保存数据到硬盘
### `SHUTDOWN [NOSAVE] [SAVE]`
异步保存数据到硬盘，并关闭服务器
### `SLAVEOF host port`
将当前服务器转变为指定服务器的从属服务器(slave server)
### `SLOWLOG subcommand [argument]`
管理 redis 的慢日志
### `SYNC`
用于复制功能(replication)的内部命令


## 键(key)的管理 `COMMAND KEY_NAME`
### DEL key
该命令用于在 key 存在时删除 key。
### DUMP key
序列化给定 key ，并返回被序列化的值。
### EXISTS key
检查给定 key 是否存在。
### EXPIRE key seconds
为给定 key 设置过期时间。
### EXPIREAT key timestamp
EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
### PEXPIRE key milliseconds
设置 key 的过期时间以毫秒计。
### PEXPIREAT key milliseconds-timestamp
设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
### KEYS pattern
查找所有符合给定模式( pattern)的 key 。
### MOVE key db
将当前数据库的 key 移动到给定的数据库 db 当中。
### PERSIST key
移除 key 的过期时间，key 将持久保持。
### PTTL key
以毫秒为单位返回 key 的剩余的过期时间。
### TTL key
以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
### RANDOMKEY
从当前数据库中随机返回一个 key 。
### RENAME key newkey
修改 key 的名称
### RENAMENX key newkey
仅当 newkey 不存在时，将 key 改名为 newkey 。
### TYPE key
返回 key 所储存的值的类型。

# Redis 发布订阅
Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。   
Redis 客户端可以订阅任意数量的频道。  
下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：  
![订阅发布关系]("http://www.runoob.com/wp-content/uploads/2014/11/pubsub1.png")  
当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：  
![订阅发布关系]("http://www.runoob.com/wp-content/uploads/2014/11/pubsub2.png") 

## Redis 发布订阅操作示例
创建了订阅频道名为 redisChat:
```
SUBSCRIBE redisChat
```

打开新的客户端，并输入命令。
```
PUBLISH redisChat "Redis is a great caching technique"
PUBLISH redisChat "Learn redis by runoob.com"
```

订阅者收到消息：
```
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by runoob.com"
```

## Redis 发布订阅命令
### `PSUBSCRIBE pattern [pattern ...]`
订阅一个或多个符合给定模式的频道。
### `PUBSUB subcommand [argument [argument ...]]`
查看订阅与发布系统状态。
### `PUBLISH channel message`
将信息发送到指定的频道。
### `PUNSUBSCRIBE [pattern [pattern ...]]`
退订所有给定模式的频道。
### `SUBSCRIBE channel [channel ...]`
订阅给定的一个或多个频道的信息。
### `UNSUBSCRIBE [channel [channel ...]]`
指退订给定的频道。

# Redis 事物
Redis 事务可以一次执行多个命令， 并且带有以下两个重要的保证：  
* 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
* 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。  
  
一个事务从开始到执行会经历以下三个阶段：
* 开始事务。
* 命令入队。
* 执行事务。

## 示例操作
```
MULTI
SET book-name "Mastering C++ in 21 days"
GET book-name
SADD tag "C++" "Programming" "Mastering Series"
SMEMBERS tag
EXEC
```

## Redis事物命令
### `DISCARD`
取消事务，放弃执行事务块内的所有命令。
### `EXEC`
执行所有事务块内的命令。
### `MULTI`
标记一个事务块的开始。
### `UNWATCH`
取消 WATCH 命令对所有 key 的监视。
### `WATCH key [key ...]`
监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

# Redis 脚本
Redis 脚本使用 Lua 解释器来执行脚本。 Reids 2.6 版本通过内嵌支持 Lua 环境。执行脚本的常用命令为 EVAL。 

## 基本语法
`EVAL script numkeys key [key ...] arg [arg ...]`
例子
`EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second`

## 命令
### `EVAL script numkeys key [key ...] arg [arg ...]`
执行 Lua 脚本。
### `EVALSHA sha1 numkeys key [key ...] arg [arg ...]`
执行 Lua 脚本。
### `SCRIPT EXISTS script [script ...]`
查看指定的脚本是否已经被保存在缓存当中。
### `SCRIPT FLUSH`
从脚本缓存中移除所有脚本。
### `SCRIPT KILL`
杀死当前正在运行的 Lua 脚本。
### `SCRIPT LOAD script`
将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。

# Redis 高级操作
## 数据备份与恢复
### 备份
`save`该命令将在 redis 安装目录中创建dump.rdb文件。
`Bgsave` 该命令也可以备份文件，是在后台执行。
### 恢复
将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。
查看redis目录`CONFIG GET dir`

# Redis 安全
我们可以通过 redis 的配置文件设置密码参数，客户端连接到 redis 服务就需要密码验证，这样可以让你的 redis 服务更安全。 
##  设置密码与获取密码
```
CONFIG set requirepass "hello"
CONFIG get requirepass
```
## 验证密码
`AUTO password`

# 性能测试
Redis 性能测试是通过同时执行多个命令实现的。
##  命令与参数解释：
```
redis-benchmark [option] [option value]
```
* -h：指定服务器主机名，默认值：127.0.0.1
* -p：指定服务器端口，默认值：6379
* -s：指定服务器socket	
* -c：指定并发连接数，默认值：50
* -n：指定请求数，默认值：10000
* -d：以字节的形式指定 SET/GET 值的数据大小，默认值：2
* -k：1=keep alive 0=reconnect，默认值：1
* -r：SET/GET/INCR 使用随机 key, SADD 使用随机值	
* -P：通过管道传输 <numreq> 请求，默认值：1
* -q：强制退出 redis。仅显示 query/sec 值	
* --csv：以 CSV 格式输出	
* -l：生成循环，永久执行测试	
* -t：仅运行以逗号分隔的测试命令列表。	
* -I：Idle 模式。仅打开 N 个 idle 连接并等待。	


# Redis 管道技术
Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。这意味着通常情况下一个请求会遵循以下步骤：
* 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
* 服务端处理命令，并将结果返回给客户端。

## 示例
Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。
```
$(echo -en "PING\r\n SET liang redis\r\nGET liang\r\nINCR visitor\r\nINCR visitor\r\nINCR visitor\r\n"; sleep 10) | nc localhost 6379

+PONG
+OK
redis
:1
:2
:3
```
 以上实例中我们通过使用 PING 命令查看redis服务是否可用， 之后我们们设置了 runoobkey 的值为 redis，然后我们获取 runoobkey 的值并使得 visitor 自增 3 次。  
在返回的结果中我们可以看到这些命令一次性向 redis 服务提交，并最终一次性读取所有服务端的响应   

## 优势
管道技术最显著的优势是提高了 redis 服务的性能。   
在下面的测试中，我们将使用Redis的Ruby客户端，支持管道技术特性，测试管道技术对速度的提升效果。
```ruby
require 'rubygems' 
require 'redis'
def bench(descr) 
start = Time.now 
yield 
puts "#{descr} #{Time.now-start} seconds" 
end
def without_pipelining 
r = Redis.new 
10000.times { 
	r.ping 
} 
end
def with_pipelining 
r = Redis.new 
r.pipelined { 
	10000.times { 
		r.ping 
	} 
} 
end
bench("without pipelining") { 
	without_pipelining 
} 
bench("with pipelining") { 
	with_pipelining 
}
```
 从处于局域网中的Mac OS X系统上执行上面这个简单脚本的数据表明，开启了管道操作后，往返时延已经被改善得相当低了。
```
without pipelining 1.185238 seconds 
with pipelining 0.250783 seconds
```
如你所见，开启管道后，我们的速度效率提升了5倍。

# Redis 分区
分区是分割数据到多个Redis实例的处理过程，因此每个实例只保存key的一个子集。
## 分区的优势
* 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
* 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。
## 分区的不足
redis的一些特性在分区方面表现的不是很好：

* 涉及多个key的操作通常是不被支持的。举例来说，当两个set映射到不同的redis实例上时，你就不能对这两个set执行交集操作。
* 涉及多个key的redis事务不能使用。
* 当使用分区时，数据处理较为复杂，比如你需要处理多个rdb/aof文件，并且从多个实例和主机备份持久化文件。
* 增加或删除容量也比较复杂。redis集群大多数支持在运行时增加、删除节点的透明数据平衡的能力，但是类似于客户端分区、代理等其他系统则不支持这项特性。然而，一种叫做presharding的技术对此是有帮助的。

## 分区类型
Redis 有两种类型分区。 假设有4个Redis实例 R0，R1，R2，R3，和类似user:1，user:2这样的表示用户的多个key，对既定的key有多种不同方式来选择这个key存放在哪个实例中。也就是说，有不同的系统来映射某个key到某个Redis服务。
### 范围分区
最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例。  
比如，ID从0到10000的用户会保存到实例R0，ID从10001到 20000的用户会保存到R1，以此类推。  
这种方式是可行的，并且在实际中使用，不足就是要有一个区间范围到实例的映射表。这个表要被管理，同时还需要各 种对象的映射表，通常对Redis来说并非是好的方法。
### 哈希分区
另外一种分区方法是hash分区。这对任何key都适用，也无需是object_name:这种形式，像下面描述的一样简单：

* 用一个hash函数将key转换为一个数字，比如使用crc32 hash函数。对key foobar执行crc32(foobar)会输出类似93024922的整数。
*  对这个整数取模，将其转化为0-3之间的数字，就可以将这个整数映射到4个Redis实例中的一个了。93024922 % 4 = 2，就是说key foobar应该被存到R2实例中。注意：取模操作是取除的余数，通常在多种编程语言中用%操作符实现。

# java 使用Redis
需先安装Redis。并且加载jedis.jar

## 连接到redis服务
```
import redis.clients.jedis.Jedis;
 
public class RedisJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //查看服务是否运行
        System.out.println("服务正在运行: "+jedis.ping());
    }
}
```
## Redis Java String(字符串) 实例
```
import redis.clients.jedis.Jedis;
 
public class RedisStringJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //设置 redis 字符串数据
        jedis.set("mykey", "hello");
        // 获取存储的数据并输出
        System.out.println("redis 存储的字符串为: "+ jedis.get("mykey"));
    }
}
```
## Redis Java List(列表) 实例
```
import java.util.List;
import redis.clients.jedis.Jedis;
 
public class RedisListJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //存储数据到列表中
        jedis.lpush("site-list", "Baidu");
        jedis.lpush("site-list", "Google");
        jedis.lpush("site-list", "Taobao");
        // 获取存储的数据并输出
        List<String> list = jedis.lrange("site-list", 0 ,2);
        for(int i=0; i<list.size(); i++) {
            System.out.println("列表项为: "+list.get(i));
        }
    }
}
```
## Redis Java Keys 实例
```
import java.util.Iterator;
import java.util.Set;
import redis.clients.jedis.Jedis;
 
public class RedisKeyJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
 
        // 获取数据并输出
        Set<String> keys = jedis.keys("*"); 
        Iterator<String> it=keys.iterator() ;   
        while(it.hasNext()){   
            String key = it.next();   
            System.out.println(key);   
        }
    }
}
```

