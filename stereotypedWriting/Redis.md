### Redis学习

#### 简介

Redis 全称 Remote Dictionary Server（即远程字典服务），它是一个基于内存实现的键值型非关系（NoSQL）数据库，由意大利人 Salvatore Sanfilippo 使用 C 语言编写。

Redis 遵守 BSD 协议，实现了免费开源，其最新版本是 6.20，常用版本包括 3.0 、4.0、5.0。自 Redis 诞生以来，它以其超高的性能、完美的文档和简洁易懂的源码广受好评，国内外很多大型互联网公司都在使用 Redis，比如腾讯、阿里、Twitter、Github 等等。

#### 特点

- Redis 不仅可以将数据完全保存在内存中，还可以通过磁盘实现数据的持久存储；
- Redis 支持丰富的数据类型，包括 string、list、set、zset、hash 等多种数据类型，因此它也被称为“数据结构服务器”；
- Redis 支持主从同步，即 master-slave 主从复制模式。数据可以从主服务器向任意数量的从服务器上同步，有效地保证数据的安全性；
- Redis 支持多种编程语言，包括 C、C++、Python、Java、PHP、Ruby、Lua 等语言。

#### 配置文件

在 Redis 的安装目录中有一个名为 redis.windows.conf 的配置文件，若在 Linux 中则为 redis.conf。

##### 查看配置命令

```
redis 127.0.0.1:6379> CONFIG GET 配置名称
```

通过使用`*`可以查看所有配置项，命令如下：

```
redis 127.0.0.1:6379> CONFIG GET *
```

##### 更改命令

```
edis 127.0.0.1:6379> CONFIG SET 配置项名称 配置项参数值
```

##### 配置说明

| 配置项                            | 参数                                       | 说明                                       |
| ------------------------------ | ---------------------------------------- | ---------------------------------------- |
| daemonize                      | no/yes                                   | 默认为 no，表示 Redis 不是以守护进程的方式运行，通过修改为 yes 启用守护进程。 |
| pidfile                        | 文件路径                                     | 当 Redis 以守护进程方式运行时，会把进程 pid 写入自定义的文件中。   |
| port                           | 6379                                     | 指定 Redis 监听端口，默认端口为 6379。                |
| bind                           | 127.0.0.1                                | 绑定的主机地址。                                 |
| timeout                        | 0                                        | 客户端闲置多长秒后关闭连接，若指定为 0 ，表示不启用该功能。          |
| loglevel                       | notice                                   | 指定日志记录级别，支持四个级别：debug、verbose、notice、warning，默认为 notice。 |
| logfile                        | stdout                                   | 日志记录方式，默认为标准输出。                          |
| databases                      | 16                                       | 设置数据库的数量（0-15个）共16个，Redis 默认选择的是 0 库，可以使用 SELECT 命令来选择使用哪个数据库储存数据。 |
| save[seconds][changes]         | 可以同时配置三种模式：save 900 1save 300 10save 60 10000 | 表示在规定的时间内，执行了规定次数的写入或修改操作，Redis 就会将数据同步到指定的磁盘文件中。比如 900s 内做了一次更改，Redis 就会自动执行数据同步。 |
| rdbcompression                 | yes/no                                   | 当数据存储至本地数据库时是否要压缩数据，默认为 yes。             |
| dbfilename                     | dump.rdb                                 | 指定本地存储数据库的文件名，默认为 dump.rdb。              |
| dir                            | ./                                       | 指定本地数据库存放目录。                             |
| slaveof <masterip><masterport> | 主从复制配置选项                                 | 当本机为 slave 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动与 master 主机进行数据同步。 |
| requirepass                    | foobared 默认关闭                            | 密码配置项，默认关闭，用于设置 Redis 连接密码。如果配置了连接密码，客户端连接 Redis 时需要通过<password> 密码认证。 |
| maxmemory<bytes>               | 最大内存限制配置项                                | 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会尝试清除已到期或即将到期的 Key，当此方法处理 后，若仍然到达最大内存设置，将无法再进行写入操作，但可以进行读取操作。 |
| appendfilename                 | appendonly.aof                           | 指定 AOF 持久化时保存数据的文件名，默认为 appendonly.aof。  |
| glueoutputbuf                  | yes                                      | 设置向客户端应答时，是否把较小的包合并为一个包发送，默认开启状态。        |

#### Redis-Key

| 命令                  | 说明                                       |
| ------------------- | ---------------------------------------- |
| DEL key             | 若键存在的情况下，该命令用于删除键。                       |
| DUMP key            | 用于序列化给定 key ，并返回被序列化的值。                  |
| EXISTS key          | 用于检查键是否存在，若存在则返回 1，否则返回 0。               |
| EXPIRE key          | 设置 key 的过期时间，以秒为单位。                      |
| EXPIREAT key        | 该命令与 EXPIRE 相似，用于为 key 设置过期时间，不同在于，它的时间参数值采用的是时间戳格式。 |
| PEXPIRE key         | 设置 key 的过期，以毫秒为单位。                       |
| PEXPIREAT key       | 与 PEXPIRE 相似，用于为 key 设置过期时间，采用以毫秒为单位的时间戳格式。 |
| KEYS pattern        | 此命令用于查找与指定 pattern 匹配的 key。              |
| MOVE key db         | 将当前数据库中的 key 移动至指定的数据库中（默认存储为 0 库，可选 1-15中的任意库）。 |
| PERSIST key         | 该命令用于删除 key 的过期时间，然后 key 将一直存在，不会过期。     |
| PTTL key            | 用于检查 key 还剩多长时间过期，以毫秒为单位。                |
| TTL key             | 用于检查 key 还剩多长时间过期，以秒为单位。                 |
| RANDOMKEY           | 从当前数据库中随机返回一个 key。                       |
| RENAME key newkey   | 修改 key 的名称。                              |
| RENAMENX key newkey | 如果新键名不重复，则将 key 修改为 newkey。              |
| SCAN cursor         | 基于游标的迭代器，用于迭代数据库中存在的所有键，cursor 指的是迭代游标。  |
| TYPE key            | 该命令用于获取 value 的数据类型。                     |

#### 五种常见数据类型

##### String

| 命令                            | 说明                                       |
| ----------------------------- | ---------------------------------------- |
| SET KEY VALUE                 | 用于设定指定键的值。                               |
| GET KEY VALUE                 | 用于检索指定键的值。                               |
| GETRANGE KEY VALUE            | 返回 key 中字符串值的子字符。                        |
| GETSET KEY VALUE              | 将给定 key 的值设置为 value，并返回 key 的旧值。         |
| GETBIT KEY OFFSET             | 对 key 所存储的字符串值，获取其指定偏移量上的位（bit）。         |
| MGET KEY1 KEY2                | 批量获取一个或多个 key 所存储的值，减少网络耗时开销。            |
| SETBIT KEY OFFERSET VALUE     | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。       |
| SETEX KEY SECONDE VALUE       | 将值 value 存储到 key中 ，并将 key 的过期时间设为 seconds (以秒为单位) |
| SETNX KEY VALUE               | 当 key 不存在时设置 key 的值。                     |
| SETRANGE KEY OFFSET VLUAE     | 从偏移量 offset 开始，使用指定的 value 覆盖的 key 所存储的部分字符串值。 |
| STRLEN KEY                    | 返回 key 所储存的字符串值的长度。                      |
| MSET KEY VALUE KEY2 VALUE2    | 该命令允许同时设置多个键值对。                          |
| MSETNX [key value]            | 当指定的 key 都不存在时，用于设置多个键值对。                |
| PSETEX key milliseconds value | 此命令用于设置 key 的值和有过期时间（以毫秒为单位）。            |
| INCR KEY                      | 将 key 所存储的整数值加 1。                        |
| INCRBY key increment          | 将 key 所储存的值加上给定的递增值（increment）。          |
| INCRBYFLOAT KEY INCREMENT     | key 所储存的值加上指定的浮点递增值（increment）           |
| DESC KEY                      | 将 key 所存储的整数值减 1。                        |
| DESCBY KEY decrement          | 将 key 所储存的值减去给定的递减值（decrement）。          |
| APPEND key value              | 该命令将 value 追加到 key 所存储值的末尾。              |

##### List

| 命令                                    | 说明                                       |
| ------------------------------------- | ---------------------------------------- |
| LPUSH key value1 value2               | 在列表头部插入一个或者多个值。                          |
| LRANGE key start stop                 | 获取列表指定范围内的元素。                            |
| RPUSH key value1 [value2]             | 在列表尾部添加一个或多个值。                           |
| LPUSHX key value                      | 当储存列表的 key 存在时，用于将值插入到列表头部。              |
| RPUSHX key value                      | 当存储列表的 key 存在时，用于将值插入到列表的尾部。             |
| LINDEX key index                      | 通过索引获取列表中的元素。                            |
| LINSERT key before\|after pivot value | 指定列表中一个元素在它之前或之后插入另外一个元素。                |
| LREM key count value                  | 表示从列表中删除元素与 value 相等的元素。count 表示删除的数量，为 0 表示全部移除。 |
| LSET key index value                  | 表示通过其索引设置列表中元素的值。                        |
| LTRIM key start stop                  | 保留列表中指定范围内的元素值。                          |
| LPOP key                              | 从列表的头部弹出元素，默认为第一个元素。                     |
| RPOP key                              | 从列表的尾部弹出元素，默认为最后一个元素。                    |
| LLEN key                              | 用于获取列表的长度。                               |
| RPOPLPUSH source destination          | 用于删除列表中的最后一个元素，然后将该元素添加到另一个列表的头部，并返回该元素值。 |
| BLPOP key1 [key2 \] timeout           | 用于删除并返回列表中的第一个元素（头部操作），如果列表中没有元素，就会发生阻塞，直到列表等待超时或发现可弹出元素为止。 |
| BRPOP key1 [key2 \] timeout           | 用于删除并返回列表中的最后一个元素（尾部操作），如果列表中没有元素，就会发生阻塞， 直到列表等待超时或发现可弹出元素为止。 |
| BRPOPLPUSH source destination timeout | 从列表中取出最后一个元素，并插入到另一个列表的头部。如果列表中没有元素，就会发生阻塞，直到等待超时或发现可弹出元素时为止。 |

##### Set

| 命令                                       | 说明                           |
| ---------------------------------------- | ---------------------------- |
| SADD key member1 [member2]               | 向集合中添加一个或者多个元素，并且自动去重。       |
| SCARD key]                               | 返回集合中元素的个数。                  |
| SDIFF key1 [key2\]                       | 求两个或多个集合的差集。                 |
| SDIFFSTORE destination key1 [key2]       | 求两个集合或多个集合的差集，并将结果保存到指定的集合中。 |
| SINTER key1 [key2\]                      | 求两个或多个集合的交集。                 |
| SINTERSTORE destination key1 [key2]      | 求两个或多个集合的交集，并将结果保存到指定的集合中。   |
| SISMEMBER key member                     | 查看指定元素是否存在于集合中。              |
| SMEMBERS key                             | 查看集合中所有元素。                   |
| SMOVE source destination member          | 将集合中的元素移动到指定的集合中。            |
| SPOP key [count]                         | 弹出指定数量的元素。                   |
| SRANDMEMBER key [count\]                 | 随机从集合中返回指定数量的元素，默认返回 1个。     |
| SREM key member1 [member2]               | 删除一个或者多个元素，若元素不存在则自动忽略。      |
| SUNION key1 [key2]                       | 求两个或者多个集合的并集。                |
| SUNIONSTORE destination key1 [key2]      | 求两个或者多个集合的并集，并将结果保存到指定的集合中。  |
| SSCAN key cursor [match pattern][count count] | 该命令用来迭代的集合中的元素。              |

##### Hash

| 命令                                       | 说明                                |
| ---------------------------------------- | --------------------------------- |
| HDEL key field2 [field2\]                | 用于删除一个或多个哈希表字段。                   |
| HEXISTS key field                        | 用于确定哈希表字段是否存在。                    |
| HGET key field                           | 获取 key 关联的哈希字段的值。                 |
| [HGETALL key                             | 获取 key 关联的所有哈希字段值。                |
| HINCRBY key field increment              | 给 key 关联的哈希字段做整数增量运算 。            |
| HINCRBYFLOAT key field increment         | 给 key 关联的哈希字段做浮点数增量运算 。           |
| HKEYS key                                | 获取 key 关联的所有字段和值。                 |
| HLEN key                                 | 获取 key 中的哈希表的字段数量。                |
| HMSET key field1 value1 [field2 value2 ] | 在哈希表中同时设置多个 field-value(字段-值）     |
| HMGET key field1 [field2]                | 用于同时获取多个给定哈希字段（field）对应的值。        |
| HSET key field value                     | 用于设置指定 key 的哈希表字段和值（field/value）。 |
| HSETNX key field value                   | 仅当字段 field 不存在时，设置哈希表字段的值。        |
| HVALS key                                | 用于获取哈希表中的所有值。                     |
| HSCAN key cursor                         | 迭代哈希表中的所有键值对，cursor 表示游标，默认为 0。   |

##### Zset

| 命令                                       | 说明                                    |
| ---------------------------------------- | ------------------------------------- |
| ZADD key score1 member1 [score2 member2\] | 用于将一个或多个成员添加到有序集合中，或者更新已存在成员的 score 值 |
| ZCARD key                                | 获取有序集合中成员的数量                          |
| ZCOUNT key min max                       | 用于统计有序集合中指定 score 值范围内的元素个数。          |
| ZINCRBY key increment member             | 用于增加有序集合中成员的分值。                       |
| ZINTERSTORE destination numkeys key [key ...\] | 求两个或者多个有序集合的交集，并将所得结果存储在新的 key 中。     |
| ZLEXCOUNT key min max                    | 当成员分数相同时，计算有序集合中在指定词典范围内的成员的数量。       |
| ZRANGE key start stop [WITHSCORES\]      | 返回有序集合中指定索引区间内的成员数量。                  |
| ZRANGEBYLEX key min max [LIMIT offset count\] | 返回有序集中指定字典区间内的成员数量。                   |
| ZRANGEBYSCORE key min max [WITHSCORES\] [LIMIT] | 返回有序集合中指定分数区间内的成员。                    |
| ZRANK key member                         | 返回有序集合中指定成员的排名。                       |
| ZREM key member [member ...\]]           | 移除有序集合中的一个或多个成员。                      |
| ZREMRANGEBYLEX key min max               | 移除有序集合中指定字典区间的所有成员。                   |
| ZREMRANGEBYRANK key start stop           | 移除有序集合中指定排名区间内的所有成员。                  |
| [ZREMRANGEBYSCORE key min max            | 移除有序集合中指定分数区间内的所有成员。                  |
| ZREVRANGE key start stop [WITHSCORES\]   | 返回有序集中指定区间内的成员，通过索引，分数从高到低。           |
| ZREVRANGEBYSCORE key max min [WITHSCORES] | 返回有序集中指定分数区间内的成员，分数从高到低排序。            |
| ZREVRANK key member                      | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序。   |
| ZSCORE key member                        | 返回有序集中，指定成员的分数值。                      |
| ZUNIONSTORE destination numkeys key [key ...\] | 求两个或多个有序集合的并集，并将返回结果存储在新的 key 中。      |
| ZSCAN key cursor [MATCH pattern][COUNT count] | 迭代有序集合中的元素（包括元素成员和元素分值）。              |

#### 三种特殊数据类型

##### 基数统计 Hyperloglogs

Redis 2.8.9 版本更新了 Hyperloglog 数据结构！特点是占用空间下，计算速度快。

**什么是基数**

举个例子，A = {1, 2, 3, 4, 5}， B = {3, 5, 6, 7, 9}；那么基数（不重复的元素）= 1, 2, 4, 6, 7, 9； （允许容错，即可以接受一定误差）

**作用**

HyperLogLog 也有一些特定的使用场景，它最典型的应用场景就是统计网站用户月活量，或者网站页面的 UV(网站独立访客)数据等。

UV 与 PV(页面浏览量) 不同，UV 需要去重，同一个用户一天之内的多次访问只能计数一次。这就要求用户的每一次访问都要带上自身的用户 ID，无论是登陆用户还是未登陆用户都需要一个唯一 ID 来标识。

当一个网站拥有巨大的用户访问量时，我们可以使用 Redis 的 HyperLogLog 来统计网站的 UV （网站独立访客）数据，它提供的去重计数方案，虽说不精确，但 0.81% 的误差足以满足 UV 统计的需求。

**常用命令**

| 命令                                       | 说明                             |
| ---------------------------------------- | ------------------------------ |
| PFADD key element [element ...]          | 添加指定元素到 HyperLogLog key 中。     |
| PFCOUNT key [key ...]                    | 返回指定 HyperLogLog key 的基数估算值。   |
| PFMERGE destkey sourcekey [sourcekey ...] | 将多个 HyperLogLog key 合并为一个 key。 |

##### 位图 bitmap

​     在平时开发过程中，经常会有一些 bool 类型数据需要存取。比如记录用户一年内签到的次数，签了是 1，没签是 0。如果使用 key-value 来存储，那么每个用户都要记录 365 次，当用户成百上亿时，需要的存储空间将非常巨大。为了解决这个问题，Redis 提供了位图结构。

**常用命令**

| 命令                       | 说明                                       |
| ------------------------ | ---------------------------------------- |
| SETBIT key offset value  | 用来设置或者清除某一位上的值，其返回值是原来位上存储的值。key 在初始状态下所有的位都为 0 |
| GETBIT                   | 用来获取某一位上的值。                              |
| BITCOUNT key [start end] | 统计指定位区间上，值为 1 的个数。                       |

##### 地理位置 geospatial

在 Redis 3.2 版本中，新增了存储地理位置信息的功能。

**常用命令**

| 序号   | 命令                | 说明                                       |
| ---- | ----------------- | ---------------------------------------- |
| 1    | GEOADD            | 将指定的地理空间位置（纬度、经度、名称）添加到指定的 key 中。        |
| 2    | GEOPOS            | 从 key 里返回所有给定位置元素的位置（即经度和纬度）             |
| 3    | GEODIST           | 返回两个地理位置间的距离，如果两个位置之间的其中一个不存在， 那么命令返回空值。 |
| 4    | GEORADIUS         | 根据给定地理位置坐标(经纬度)获取指定范围内的地理位置集合。           |
| 5    | GEORADIUSBYMEMBER | 根据给定地理位置(具体的位置元素)获取指定范围内的地理位置集合。         |
| 6    | GEOHASH           | 获取一个或者多个的地理位置的 GEOHASH 值。                |
| 7    | ZREM              | 通过有序集合的 zrem 命令实现对地理位置信息的删除。             |

#### Redis持久化

##### RDB

RDB 即快照模式，它是 Redis 默认的数据持久化方式，它会将数据库的快照保存在 dump.rdb 这个二进制文件中。

RDB 实际上是 Redis 内部的一个定时器事件，它每隔一段固定时间就去检查当前数据发生改变的次数和改变的时间频率，看它们是否满足配置文件中规定的持久化触发条件。当满足条件时，Redis 就会通过操作系统调用 fork() 来创建一个子进程，该子进程与父进程享有相同的地址空间。

###### 触发策略

**1.手动触发**

手动触发是通过`SAVAE`命令或者`BGSAVE`命令将内存数据保存到磁盘文件中。

**2. 自动触发策略**

动触发策略，是指 Redis 在指定的时间内，数据发生了多少次变化时，会自动执行`BGSAVE`命令。自动触发的条件包含在了 Redis 的配置文件中

- save 900 1 表示在 900 秒内，至少更新了 1 条数据，Redis 自动触发 BGSAVE 命令，将数据保存到硬盘。
- save 300 10 表示在 300 秒内，至少更新了 10 条数据，Redis 自动触 BGSAVE 命令，将数据保存到硬盘。
- save 60 10000 表示 60 秒内，至少更新了 10000 条数据，Redis 自动触发 BGSAVE 命令，将数据保存到硬盘。

###### 优点与劣势

我们知道，在 RDB 持久化的过程中，子进程会把 Redis 的所有数据都保存到新建的 dump.rdb 文件中，这是一个既消耗资源又浪费时间的操作。因此 Redis 服务器不能过于频繁地创建 rdb 文件，否则会严重影响服务器的性能。

RDB 持久化的最大不足之处在于，最后一次持久化的数据可能会出现丢失的情况。我们可以这样理解，在 持久化进行过程中，服务器突然宕机了，这时存储的数据可能并不完整，比如子进程已经生成了 rdb 文件，但是主进程还没来得及用它覆盖掉原来的旧 rdb 文件，这样就把最后一次持久化的数据丢失了。

##### AOF

AOF 被称为追加模式，或日志模式

每当有一个修改数据库的命令被执行时，服务器就将命令写入到 appendonly.aof 文件中，该文件存储了服务器执行过的所有修改命令，因此，只要服务器重新执行一次 .aof 文件，就可以实现还原数据的目的，这个过程被形象地称之为“命令重演”。

#### 内存管理

 Redis的内存管理提供了过期策略与内存淘汰

##### 过期策略

  过期策略是指对每个存储到redis中的key设置过期时间。Redis提供了 EXPIRE 与 EXPIREAT 来为 key 设置过期时间。

Redis中提供了两种过期删除策略：惰性删除和定期删除。

###### 定期删除

 定期删除类似一个守护线程，每间隔一段时间就执行一次（默认100ms一次，可以通过修改配置文件redis.conf 的 hz 选项来调整这个次数），将过期的Key进行删除，具体过程如下：

- 从过期字典中随机选出20个key；
- 删除这20个key中已经过期的key；
- 如果过期的key的比例超过了1/4，那就重复从步骤1开始执行；

###### 惰性删除

惰性删除即当查询某个Key时，判断该key是否已过期，如果已过期则从缓存中删除，同时返回空。

##### 内存淘汰

在配置文件redis.conf 中，可以通过参数 maxmemory <bytes> 来设定最大内存，当数据内存达到 maxmemory 时，便会触发redis的内存淘汰策略（我们一般会将该参数设置为物理内存的四分之三）。

- volatile-lru，针对设置了过期时间的key，使用lru算法进行淘汰。
- allkeys-lru，针对所有key使用lru算法进行淘汰。
- volatile-lfu，针对设置了过期时间的key，使用lfu算法进行淘汰。
- allkeys-lfu，针对所有key使用lfu算法进行淘汰。
- volatile-random，从所有设置了过期时间的key中使用随机淘汰的方式进行淘汰。
- allkeys-random，针对所有的key使用随机淘汰机制进行淘汰。
- volatile-ttl，针对设置了过期时间的key，越早过期的越先被淘汰。
- noeviction，不会淘汰任何数据，当使用的内存空间超过 maxmemory 值时，再有写请求来时返回错误。

###### LRU

LRU是Least Recently Used的缩写，也就是表示最近很少使用，也可以理解成最久没有使用。也就是说当内存不够的时候，每次添加一条数据，都需要抛弃一条最久时间没有使用的旧数据。

###### LFU

LFU（Least Frequently Used），表示最近最少使用，它和key的使用次数有关，其思想是：根据key最近被访问的频率进行淘汰，比较少访问的key优先淘汰，反之则保留。

#### IO模型





#### 哨兵机制





#### Redis的使用场景

1. 缓存
2. 数据共享
3. 分布式锁
4. 全局ID
5. 计数器
6. 限流    以访问者的ip和其他信息作为key，访问一次增加一次计数，超过次数则返回false


