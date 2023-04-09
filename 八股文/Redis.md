# Redis使用指南

## 安装与环境配置

## 基本指令

### 数据库 | 全局

- `redis-cli -h 172.19.34.103 -p 12379`连接redis数据库
- `Auth "OyIKglTFsDGEIlBM"`登录，后面是数据库密码
- `select <索引0-15>`切换到指定数据库
- `dbsize` 查看所有key的数目
- `flushdb` 删除当前选择数据库中的所有key
- `flushall` 删除所有数据库中的所有key
- `save`: 将数据同步保存到磁盘
- `bgsave`: 异步保存
- `lastsave`: 上次成功保存到磁盘的Unix时间戳
- `info`: 查询server信息
- `ping` pong检测redis服务器是否启动

### Key相关

- `keys *` 查看当前数据库所有的Key
- `type(s) key`指定字段类型
- `get key`获取指定key
- `rename key <newKey>`修改key的名称
- `del key`删除key
- `expire key <second>`设置key过期时间
- `TTL key`剩余生存时间
- `exists key`判断key是否存在

- `keys aaa*` 查看前缀是aaa的key ；可能会使服务器卡顿，而出现事故，用下面那个
- `SCAN  0  MATCH aaa*  COUNT  5` 表示从游标0开始查询aaa开头的key，每次返回5条，但是这个5条不一定，只是给Redis打了个招呼，具体返回数量看Redis心情
- `keys *aaa*` 查看包括aaa的key
- keys *aaa 查看后缀是aaa的key

### string相关

```cpp
普通字符串的基本操作：
    # 设置 key-value 类型的值
    > SET name lin
    OK
    # 根据 key 获得对应的 value
    > GET name
    "lin"
    # 判断某个 key 是否存在
    > EXISTS name
    (integer) 1
    # 返回 key 所储存的字符串值的长度
    > STRLEN name
    (integer) 3
    # 删除某个 key 对应的值
    > DEL name
    (integer) 1
批量设置 :
	# 批量设置 key-value 类型的值
    > MSET key1 value1 key2 value2 
    OK
    # 批量获取多个 key 对应的 value
    > MGET key1 key2 
    1) "value1"
    2) "value2"
```

### hash相关

```
hset myhash name ljh      添加一个键值对
hget myhash name          取出值
hmset myhash name ljh age 20 note "i am notes"      批量键值对
hmget myhash name age note   
hgetall myhash               获取所有的键值对
hexists myhash name          是否存在
hsetnx myhash score 100      不存在的话创建，存在的话修改
hdel myhash name             删除
hkeys myhash                 只取key
hvals myhash                 只取value
hlen myhash                  长度
```

### list相关

```
lpush mylist a b c  左插入
rpush mylist x y z  右插入
lrange mylist 0 -1  查询所有的数据
lpop mylist  弹出一个元素
rpop mylist  弹出一个元素
llen mylist  长度
lrem mylist count value  根据值来删除（count表示要删除多少个值为value的元素）
lindex mylist 2          指定索引的值
linsert mylist before 1 value   在值为1前面插入value值
linsert mylist after 1 value    在值为1的后面插入value值
rpoplpush list list2     将list的最后一个元素移到list2中
```

### set相关

```
sadd myset e            添加一个元素
smembers myset       查询所有的数据
srem myset e        删除
sismember myset e 判断元素是否在集合中
scard key_name       获取set集合中元素个数
sdiff | sinter | sunion <key> <key> 操作：集合间运算：差集 | 交集 | 并集
srandmember          随机获取集合中的元素
spop                 从集合中弹出一个元素
```

### zset相关

```
zadd zset 1 one          添加一个元素（1表示score，用作排序使用）
zadd zset 2 two            
zadd zset 3 three
zincrby zset 1 one              分数+1
zscore zset two                  获取分数
 zrange zset 0 -1                   获取全部的值
zrange zset 0 -1 withscores     获取全部值并附带分数
zrangebyscore zset 10 25 withscores 分数在某个范围的值
zrangebyscore zset 10 25 withscores limit 1 2 分页
Zrevrangebyscore zset 10 25 withscores  指定范围的值从大到小排序
zcard zset  元素数量
Zcount zset 获得指定分数范围内的元素个数
Zrem zset one two        删除一个或多个元素
Zremrangebyrank zset 0 1  按照排名范围删除元素
Zremrangebyscore zset 0 1 按照分数范围删除元素
```

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20230325143951.png)

## Redis 与 Memcached **区别**：

- Redis 支持的数据类型更丰富（String、Hash、List、Set、ZSet），而 Memcached 只支持最简单的 key-value 数据类型；
- Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用，而 Memcached 没有持久化功能，数据全部存在内存之中，Memcached 重启或者挂掉后，数据就没了；
- Redis 原生支持集群模式，Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；
- Redis 支持发布订阅模型、Lua 脚本、事务等功能，而 Memcached 不支持

## Redis 五种数据类型的应用场景：

- String 类型的应用场景：缓存对象、常规计数、分布式锁、共享 session 信息等。
- List 类型的应用场景：消息队列（但是有两个问题：1. 生产者需要自行实现全局唯一 ID；2. 不能以消费组形式消费数据）等。
- Hash 类型：缓存对象、购物车等。
- Set 类型：聚合计算（并集、交集、差集）场景，比如点赞、共同关注、抽奖活动等。
- Zset 类型：排序场景，比如排行榜、电话和姓名排序等。

Redis 后续版本又支持四种数据类型，它们的应用场景如下：

- BitMap（2.2 版新增）：二值状态统计的场景，比如签到、判断用户登陆状态、连续签到用户总数等；
- HyperLogLog（2.8 版新增）：海量数据基数统计的场景，比如百万级网页 UV 计数等；
- GEO（3.2 版新增）：存储地理位置信息的场景，比如滴滴叫车；
- Stream（5.0 版新增）：消息队列，相比于基于 List 类型实现的消息队列，有这两个特有的特性：自动生成全局唯一消息ID，支持以消费组形式消费数据。

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20230325144131.png)

## Redis 单线程模式是怎样的？

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20230325144218.png)

之所以 Redis 采用单线程（网络 I/O 和执行命令）那么快，有如下几个原因：

- Redis 的大部分操作**都在内存中完成**，并且采用了高效的数据结构，因此 Redis 瓶颈可能是机器的内存或者网络带宽，而并非 CPU，既然 CPU 不是瓶颈，那么自然就采用单线程的解决方案了；
- Redis 采用单线程模型可以**避免了多线程之间的竞争**，省去了多线程切换带来的时间和性能上的开销，而且也不会导致死锁问题。
- Redis 采用了 **I/O 多路复用机制**处理大量的客户端 Socket 请求，IO 多路复用机制是指一个线程处理多个 IO 流，就是我们经常听到的 select/epoll 机制。简单来说，在 Redis 只运行单线程的情况下，该机制允许内核中，同时存在多个监听 Socket 和已连接 Socket。内核会一直监听这些 Socket 上的连接请求或数据请求。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO 流的效果。

## Redis 持久化

Redis 共有三种数据持久化的方式：

- **AOF 日志**：每执行一条写操作命令，就把该命令以追加的方式写入到一个文件里；
- **RDB 快照**：将某一时刻的内存数据，以二进制的方式写入磁盘；
- **混合持久化方式**：Redis 4.0 新增的方式，集成了 AOF 和 RBD 的优点

Redis 为了避免 AOF 文件越写越大，提供了 **AOF 重写机制**，当 AOF 文件的大小超过所设定的阈值后，Redis 就会启用 AOF 重写机制，来压缩 AOF 文件。

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20230325144342.png)

那问题来了，重写 AOF 日志过程中，如果主进程修改了已经存在 key-value，那么会发生写时复制，此时这个 key-value 数据在子进程的内存数据就跟主进程的内存数据不一致了，这时要怎么办呢？

为了解决这种数据不一致问题，Redis 设置了一个 **AOF 重写缓冲区**，这个缓冲区在创建 bgrewriteaof 子进程之后开始使用。

在重写 AOF 期间，当 Redis 执行完一个写命令之后，它会**同时将这个写命令写入到 「AOF 缓冲区」和 「AOF 重写缓冲区」**。



Redis 的快照是**全量快照**，也就是说每次执行快照，都是把内存中的「所有数据」都记录到磁盘中。所以执行快照是一个比较重的操作，如果频率太频繁，可能会对 Redis 性能产生影响。如果频率太低，服务器故障时，丢失的数据会更多。



混合持久化工作在 **AOF 日志重写过程**，当开启了混合持久化时，在 AOF 重写日志时，fork 出来的重写子进程会先将与主线程共享的内存数据以 RDB 方式写入到 AOF 文件，然后主线程处理的操作命令会被记录在重写缓冲区里，重写缓冲区里的增量命令会以 AOF 方式写入到 AOF 文件，写入完成后通知主进程将新的含有 RDB 格式和 AOF 格式的 AOF 文件替换旧的的 AOF 文件。

也就是说，使用了混合持久化，AOF 文件的**前半部分是 RDB 格式的全量数据，后半部分是 AOF 格式的增量数据**。

## Redis 过期删除策略和内存淘汰策略有什么区别？

### 过期删除策略有哪些？

常见的三种过期删除策略：

- 定时删除；
- 惰性删除；
- 定期删除；

**Redis 选择「惰性删除+定期删除」这两种策略配和使用**，以求在合理使用 CPU 时间和避免内存浪费之间取得平衡。

Redis 在访问或者修改 key 之前，都会调用 expireIfNeeded 函数对其进行检查，检查 key 是否过期：

- 如果过期，则删除该 key，至于选择异步删除，还是选择同步删除，根据 `lazyfree_lazy_expire` 参数配置决定（Redis 4.0版本开始提供参数），然后返回 null 客户端；
- 如果没有过期，不做任何处理，然后返回正常的键值对给客户端；

细说说 Redis 的定期删除的流程：

1. 从过期字典中随机抽取 20 个 key；
2. 检查这 20 个 key 是否过期，并删除已过期的 key；
3. 如果本轮检查的已过期 key 的数量，超过 5 个（20/4），也就是「已过期 key 的数量」占比「随机抽取 key 的数量」大于 25%，则继续重复步骤 1；如果已过期的 key 比例小于 25%，则停止继续删除过期 key，然后等待下一轮再检查。

为了保证定期删除不会出现循环过度，导致线程卡死现象，为此增加了定期删除循环流程的时间上限，默认不会超过 25ms。

### 内存淘汰策略

当 Redis 的运行内存已经超过 Redis 设置的最大内存之后，则会使用内存淘汰策略删除符合条件的 key，以此来保障 Redis 高效的运行

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E8%BF%87%E6%9C%9F%E7%AD%96%E7%95%A5/%E5%86%85%E5%AD%98%E6%B7%98%E6%B1%B0%E7%AD%96%E7%95%A5.jpg)

**LRU** 全称是 Least Recently Used 翻译为**最近最少使用**，会选择淘汰最近最少使用的数据。

Redis 实现的是一种**近似 LRU 算法**，目的是为了更好的节约内存，它的**实现方式是在 Redis 的对象结构体中添加一个额外的字段，用于记录此数据的最后一次访问时间**。

但是 LRU 算法有一个问题，**无法解决缓存污染问题**，比如应用一次读取了大量的数据，而这些数据只会被读取这一次，那么这些数据会留存在 Redis 缓存中很长一段时间，造成缓存污染。

LFU 全称是 Least Frequently Used 翻译为**最近最不常用**，LFU 算法是根据数据访问次数来淘汰数据的，它的核心思想是“如果数据过去被访问多次，那么将来被访问的频率也更高”。

**在 LFU 算法中**，Redis对象头的 24 bits 的 lru 字段被分成两段来存储，高 16bit 存储 ldt(Last Decrement Time)，低 8bit 存储 logc(Logistic Counter)。

- ldt 是用来记录 key 的访问时间戳；
- logc 是用来记录 key 的访问频次，它的值越小表示使用频率越低，越容易淘汰，每个新加入的 key 的logc 初始值为 5。

注意，logc 并不是单纯的访问次数，而是访问频次（访问频率），因为 **logc 会随时间推移而衰减的**
