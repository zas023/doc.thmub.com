[toc]
>《Redis开发与运维》 付磊 张益军

### 1. 背景

#### 1.1 NoSQL
> 单机MySQL数据存储的瓶颈：数据量的总大小一台机器放不下，数据索引（B+树）一台机器的内存不足，访问量（读写混合）一个实例不够。

常见的解决方案：
Memcached（缓存）+垂直拆分
MySQL主从读写分离
分库分表+s水平拆分+MySQL集群
##### 1.1.1 概念
**定义**：NoSQL==Not Only SQL，泛指**非关系型数据库**，这些类型的数据存储不需要固定的模式，无需多余的操作就可以横向拓展。
**特点**：易拓展（数据间无关系），大数据量高性能，数据模型灵活多样（增删字段）。
**实现**：Redis，Memcache，Mongdb。

##### 1.1.2 关键词
**3V**：海量Volume，多样Variety，实时Velocity。
**3高**：高并发，高可拓，高性能。

#### 1.2 数据模型
KV键值对
BSON（文档型数据库）
列族（列存储数据库）
图形（图关系数据库）

#### 1.3 CAP
Consistency：强一致性
Avaliability：可用性
Partition tolerance：分区容错性

**CAP的核心理论**：一个分布式系统不可能同时满足和好的一致性，可用性和分区容错性这三个需求，最多能够较好的满足两个。因此把NoSQL数据库分成满足CA，CP和AP原则的三大类。

CA：单点集群，通常在可拓展性上不太强大，传统关系型数据库
CP：性能不高，Redis、Mongodb
AP：对一致性要求较低，大多数网站架构的选择

虽说CAP理论可以三选二，但由于当前网络硬件肯定会出现丢包和延迟等问题，所以分区容错性是我们必须实现的。

#### 1.4 BASE

BASE就是为了关系型数据库强一致性而引起可用性降低的解决方案。

Basically Avaliable：基本可用
Soft state：软状态
Eventually consistent：最终一致

#### 1.5 分布式与集群

**分布式**：不同的服务器上面部署不同的服务模块或工程，它们之间通过RPC/RMI通信和调用，对外提供服务和组内合作；
**集群**：不同的多台服务器部署相同的服务模块，通过分布式调度软件进行统一的调度，对外提供服务和访问。

#### 1.6 Redis

Redis的8个特性：速度快、基于键值对的数据结构服务器、功能丰富、简单稳定、客户端语言多、持久化、主从复制、支持高可用和分布式。
Redis适合场景：缓存、排行榜、计数器（视频播放量、商品浏览量）、社交网络（踩赞、粉丝）、消息队列。
生产环境中选取稳定版本，并使用配置文件启动Redis。
Redis3.0是重要的里程碑，发布了Redis官方的分布式实现Redis Cluster。

### 2. API
> Redis提供了5种数据结构。
#### 2.1 字符串
##### 2.1.1 常用命令
1. 设置值

``` shell
set key value [ex seconds] [px milliseconds] [nx|xx] 
```
| 选项 | 描述 |
|---|---|
| ex | 为键设置秒级有效期 |
| px | 为键设置毫秒级有效期 |
| nx | 等效于 setnx ，键必须不存在才能设置成功，用于添加|
| xx | 等效于 setxx ，键必须存在才能设置成功，用于更新 |

2. 获取值

``` shell
get key
```
若要获取的键不存在，返回空`nil`。

3. 批量操作

``` shell
# 批量设置值
mset key value [key value ...]

# 批量获取值
mget key [key ...]
```
Redis可以支撑每秒数万的读写操作，执行一次redis的命令延迟瓶颈往往来自于网络传输，所以批量操作命令可以有效提升redis性能。

4. 计数

``` shell
incr key
```
`incr`命令用于对返回值做自增操作，若返回值不是整数，则返回错误，键不存在则返回1。

同时，Redis提供`decr`(自减)、`incrby`(自增指定自然数)、`decrby`(自减指定自然数)、`incrbyfloat`(自增指定浮点数)。

5. 不常用命令

``` shell
# 追加值
append key value

# 字符串长度
strlen key

# 设置并返回原值
getset key value

# 设置指定位置的字符
setrange key offeset value

# 获取部分字符串
getrange key start end
```
##### 2.1.2 内部编码

``` shell
127.0.0.1:6379> set key 8653
OK
127.0.0.1:6379> object encoding key
"int"
```
字符串类型有3中内部编码：
| 编码 | 说明 |
| -- | -- |
| int | 8个字节的整型 |
| embstr | 小于等于39字节的字符串 |
| raw | 大于39字节的字符串 |
Redis会根据当前值的类型和长度决定使用对应的内部编码实现。

##### 2.1.3 典型场景

1. 缓存功能
2. 计数
3. 共享Session
4. 限速：限制接口的访问频率

#### 2.2 哈希

##### 2.2.1 常用命令

``` shell
# 1. 设置值
hset key field value

# 2. 获取值，键或field不存在时返回空
hget key field

# 3. 删除filed，返回成功删除的个数
hdel key field [field ...]

# 4. 计算field个数
hlen key

# 5. 批量操作
hmget key field [filed ...]
hmset key field value [field value ...]

# 6. 判断field是否存在
hexists key field

# 7. 获取所有field
hkeys key

# 8. 获取所有value
hvals key

# 9. 获取所有field-value
hgetall key
```
##### 2.2.2 内部编码
**ziplist**（压缩列表）：当元素个数小于hash-max-ziplist-entries配置（默认512个）、同时所有值都小于hash-max-ziplist-value配置（默认64 字节）时使用。ziplist采用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀。
**hashtable**（哈希表）：当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而 hashtable的读写时间复杂度为O（1）。

##### 2.2.3 典型场景
1. NoSQL

#### 2.3 列表

#### 2.4 集合

#### 2.5 有序集合

#### 2.7 键管理
##### 2.7.1 单个键管理
1. 键重命名
``` shell
rename key newkey
```
重命名键期间会执行del命令删除旧的键，如果键对应的值比较大，存在阻塞Redis的可能性。

如果`rename`之前newkey已经存在，则newkey的值将被覆盖，故Redis提供了`renamenx`命令，确保只有newKey不存在时候才被覆盖。

同时Redis3.2之后支持key==newkey。

1. 随机返回一个键

``` shell
127.0.0.1:6379> dbsize
1000
127.0.0.1:6379> randomkey
"hello"
```
`randomkey`命令会随机从当前数据库中挑选一个键。

3. 键过期

键过期命令有3种返回值：大于等于0的整数表示键剩余的过期时间；-1表示键没有设置过期时间；-2表示键不存在。
``` shell
# 键在seconds秒后过期
expire key seconds

#键在秒级时间戳timestamp后过期
expireat key timestamp

# 键在milliseconds毫秒后过期
pexpire key milliseconds

# 键在毫秒级时间戳timestamp后过期
pexpireat key milliseconds-timestamp

# 查询过期时间
ttl key
```
`ttl`命令和pttl都可以查询键的剩余过期时间，但是pttl精度更高可以达到毫秒级。

但无论是使用过期时间还是时间戳，秒级还是毫秒级，在Redis内部最终使用的都是`pexpireat`。

4. 迁移键

迁移键功能非常重要，因为有时候我们只想把部分数据由一个Redis迁移到另一个Redis（例如从生产环境迁移到测试环境）。
``` shell
# 1. Redis内部数据库间迁移
move key db

# 2. Redis实例之间进行数据迁移

# dump命令会将键值序列化成RDB格式
dump key
# restore命令将上面序列化的值进行复原，其中ttl参数代表过期时间
restore key ttl value
```
关于dump+restore需要注意整个迁移过程并非原子性的，而是通过客户端分步完成的。
``` shell
# 3. Redis实例间进行数据迁移
 migrate host port key|"" destination-db timeout [COPY] [REPLACE] [KEYS key]
```
| 参数 | 描述 | 说明 |
| -- | -- | -- |
| host | 目标Redis的IP地址 |
| port | 目标Redis的端口 |
| key\|"" | 迁移的键 | 3.0.6版本之后支持迁移多个键,此处为空字符串 |
| destination-db | 目标Redis的数据库索引 |

实际上`migrate`命令就是将`dump`、`restore`、`del`三个命令进行组合，从而简化了操作流程，且具有原子性，有效地提高了迁移效率。

##### 2.6.2 遍历键

``` shell
# 1. 全量遍历键，pattern为正则
keys pattern

# 2. 渐进式遍历
scan cursor [match pattern] [count number]
```
渐进式遍历可以有效的解决`keys`命令可能产生的阻塞问题，但再遍历过程中键发生变化，则`scan`并不能保证完整的遍历出来所有的键

##### 2.6.3 数据库管理

1. 切换数据库

``` shell
select dbIndex
```
2. 数据库清理

``` shell
# 1. 清除当前数据库
flushdb

# 2. 清除所有数据库
flushall
```

#### 2.7 总结

Redis提供字符串、哈希、列表、集合、有序集合5种数据结构，每种数据结构都有多种内部编码实现。 
纯内存存储、IO多路复用技术、单线程架构是造就Redis高性能的三个因素。
由于Redis的单线程架构，所以需要每个命令被快速执行，否则会存在阻塞。理解Redis单线程命令处理机制是开发和运维Redis的核心之一。
批量操作（如mget、mset、hmset等）能够有效提高命令执行的效率，但要注意每次批量操作的个数和字节数。 
了解每个命令的时间复杂度在开发中至关重要，例如在使用keys、hgetall、smembers、zrange等时间复杂度较高的命令时，需要考虑数据规模 对于Redis的影响。
persist命令可以删除任意类型键的过期时间，但是set命令也会删除字符串类型键的过期时间，这在开发时容易被忽视。
move、dump+restore、migrate是Redis发展过程中三种迁移键的方式，其中move命令基本废弃，migrate命令用原子性的方式实现了 dump+restore，并且支持批量操作，是Redis Cluster实现水平扩容的重要工具。
scan命令可以解决keys命令可能带来的阻塞问题，同时Redis还提供hscan、sscan、zscan渐进式地遍历hash、set、zset。

### 3. 小功能大用处

慢查询
1）慢查询中的两个重要参数slowlog-log-slower-than和slowlog-maxlen。
2）慢查询不包含命令网络传输和排队时间。
3）有必要将慢查询定期存放。

Redis Shell
1）redis-cli一些重要的选项，例如--latency、–-bigkeys、-i和-r组合
2）redis-benchmark的使用方法和重要参数。

Pipeline
1）Pipeline可以有效减少RTT次数，但每次Pipeline的命令数量不能无节制。

事务与Lua
1）Redis可以使用Lua脚本创造出原子、高效、自定义命令组合。
2）Redis执行Lua脚本有两种方法：eval和evalsha。

Bitmaps
1）Bitmaps可以用来做独立用户统计，有效节省内存。
2）Bitmaps中setbit一个大的偏移量，由于申请大量内存会导致阻塞。

HyperLogLog
1）HyperLogLog虽然在统计独立总量时存在一定的误差，但是节省的内存量十分惊人。

发布订阅
1）Redis的发布订阅机制相比许多专业的消息队列系统功能较弱，不具备堆积和回溯消息的能力，但胜在足够简单。

GEO
1）Redis3.2提供了GEO功能，用来实现基于地理位置信息的应用，但 底层实现是zset。

### 4. 客户端

RESP（Redis Serialization Protocol Redis）保证客户端与服务端的正
常通信，是各种编程语言开发客户端的基础。
要选择社区活跃客户端，在实际项目中使用稳定版本的客户端。
区分Jedis直连和连接池的区别，在生产环境中，应该使用连接池。
Jedis.close（）在直连下是关闭连接，在连接池则是归还连接。
Jedis客户端没有内置序列化，需要自己选用。
客户端输入缓冲区不能配置，强制限制在1G之内，但是不会受到maxmemory限制。
客户端输出缓冲区支持普通客户端、发布订阅客户端、复制客户端配置，同样会受到maxmemory限制。
Redis的timeout配置可以自动关闭闲置客户端，tcp-keepalive参数可以周期性检查关闭无效TCP连接。
monitor命令虽然好用，但是在大并发下存在输出缓冲区暴涨的可能性。
info clients帮助开发和运维人员找到客户端可能存在的问题。
理解Redis通信原理和建立完善的监控系统对快速定位解决客户端常见问题非常有帮助。

### 5. 持久化
Redis提供了两种持久化方式：RDB和AOF。

#### 5.1 RDB
> RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据快照。

RDB使用一次性生成内存快照的方式，产生的文件紧凑压缩比更高，因此读取RDB恢复速度更快。由于每次生成RDB开销较大，无法做到实时持久化，一般用于数据冷备和复制传输。
save命令会阻塞主线程不建议使用，bgsave命令通过fork操作创建子进程生成RDB避免阻塞。

#### 5.2 AOF
> Append only File：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的

AOF通过追加写命令到文件实现持久化，通过appendfsync参数可以控制实时/秒级持久化。因为需要不断追加写命令，所以AOF文件体积逐渐 变大，需要定期执行重写操作来降低文件体积。
子进程执行期间使用copy-on-write机制与父进程共享内存，避免内存消耗翻倍。AOF重写期间还需要维护重写缓冲区，保存新的写入命令避免 数据丢失。
持久化阻塞主线程场景有：fork阻塞和AOF追加阻塞。fork阻塞时间跟内存量和系统有关，AOF追加阻塞说明硬盘资源紧张。

#### 5.3 多实例部署
> Redis单线程架构导致无法充分利用CPU多核特性，通常的做法是在一台机器上部署多个实例。

单机下部署多个实例时，为了防止出现多个子进程执行重写操作，建议做隔离控制，避免CPU和IO资源竞争。

### 6. 复制
> 在分布式系统中为了解决单点问题，通常会把数据复制多个副本部署到其他机器，满足故障恢复和负载均衡等需求。

复制功能是高可用Redis的基础，也是Redis日常运维的常见维护点。

#### 6.1 配置
Redis通过复制功能实现主节点的多个副本。从节点可灵活地通过slaveof命令建立或断开复制流程。 

#### 6.2 拓扑
1. 一主一从结构
是最简单的复制拓扑结构，用于主节点出现宕机时从节点提供故障转移支持。

1. 一主多从结构
又称为星形拓扑结构，使得应用端可以利用多个从节点实现读写分离。

1. 树状结构
又称为树状拓扑结构，从节点可以复制另一个从节点，实现一层层向下的复制流。Redis2.8之后复制的流程分为：全量复制和部分复制。全量复制需要同步全部主节点的数据集，大量消耗机器和网络资源。而部分复制有 效减少因网络异常等原因造成的不必要全量复制情况。通过配置合理的复制 积压缓冲区尽量避免全量复制。

#### 6.3 原理
1. 心跳
主从节点之间维护心跳和偏移量检查机制，保证主从节点通信正常和数据一致。
2. 异步复制
为了保证高性能复制过程是异步的，写命令处理完后直接返回给客户端，不等待从节点复制完成。因此从节点数据集会有延迟情况。

#### 6.4 开发运维
1. 读写分离
当使用从节点用于读写分离时会存在数据延迟、过期数据（支持惰性删除和定时删除两种策略）、从节点可用性等问题，需要根据自身业务提前作出规避。
2. 主从配置不一致
3. 规避全量复制
需要进行全量复制的场景：第一次建立复制、节点运行ID不匹配、复制积压缓冲区不足。
4. 复制风暴
在运维过程中，主节点存在多个从节点或者一台机器上部署大量主节点的情况下，会有复制风暴的风险（复制风暴是指大量从节点对同一主节点或者对同一台机器的多个主节点短时间内发起全量复制的过程）。

### 7. 阻塞
> 客户端最先感知阻塞等Redis超时行为
