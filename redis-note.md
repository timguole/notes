# Redis (Remote dictionary service)

### 基本信息

| 项目       | 内容         |
| ---------- | ------------ |
| 服务器进程 | redis-server |
| 客户端程序 | redis-cll    |
| 默认端口   | 6379/tcp     |

### 基本操作

启动服务进程

```shell
redis-server /etc/redis/redis.conf
```

连接服务进程

```shell
redis-cli -h localhost -p 6379 -a password
```

切换数据库，查看数据库大小

```shell
redis> select db-id # default values can be from 0 to 15
redis> dbsize
```

查看所有key，清空数据库

```shell
# list all keys
redis> keys *

# flush current db
redis> flushdb

# flush all db
redis> flushall
```



测试连接

```shell
redis> ping
PONG
```

关闭服务进程

```shell
redis> shutdown
redis> exit
```

性能测试

```shell
# -c: number of concurrent connections
# -n: number of request per connection
redis-benchmark -h localhost -p 6379 -c 50 -n 10000
```

------

### 基本数据类型

#### 键（Key）

```shell
redis> set key value
redis> get key
redis> setnx keyname value # set key if not exists
redis> setex keyname seconds value # set key with expiration
redis> expires keyname seconds
redis> ttl keyname
redis> mset key1 v1 key2 v2 ....
redis> mset k1 v1 k2 v2 ... # one fails, all fail
redis> getset key value # return old value and set new value
```



#### 字符串（String）

```shell
redis> append keyname value # append value to string
redis> strlen keyname
redis> getrange keyname start end # get substring

# substitue string with value from offset to len(value)
redis> setrange keyname offset value
```

#### 整数（也是字符串表示）

```shell
redis> incr keyname # increase by 1
redis> decr keyname # decrease by 1
redis> incrby keyname value
redis> decrby keyname value
```

#### 列表（List）

```shell
redis> lpush list value # prepend a value to a list
redis> rpush list value # append a value to a list
redis> lrange list start end

# get/set by index
redis> lindex list index # get value at index
redis> lset list index value # must in range of `llen list`

redis> llen list # length of a list
redis> lpop list # pop from left
redis> rpop list # pop from right
redis> lrem list count value # remove <count> values from list
redis> ltrim list start stop # trim values out of range from start to stop
redis> rpoplpush list1 list2 # move value from one list to another
redis> exists list
redis> linsert list before|after value value # just the first occurence
```

#### 集合（Set）

```shell
redis> sadd setname value
redis> srem setname value # remove a value
redis> smembers setname
redis> sismember setname value
redis> scard setname # number of members
redis> srandmember setname [num] # return 'num' members, defaut is 1
redis> spop setname # remove a member randomly
redis> smove setname1 setname2 value # move value from setname1 to setname2
redis> sdiff setname1 setname2 # return values only in setname1
redis> sinter set1 set2 # values in both set1 and set2
redis> sunion set1 set2 # all values in set1 and set2
```

#### 哈希（Hash）

```shell
redis> hset hash key value
redis> hget hash key
redish> hmset hash k1 v1 k2 v2 ...
redis> hmget hash k1 k2 ...
redis> hgetall hash
redis> hdel hash key
redis> hlen hash
redis> hexist hash key
redis> hkeys hash # return all keys
redis> hvals hash # return all values
redis> hincrby hash key value
redis> hdecrby hash key value
redis> hsetnx hash key value # set key and value if key not exists
```

#### 有序集合（Zset）

```shell
redis> zadd setname <score> <value> .... # set value with a score
redis> zrem setname value
redis> zrange setname start end # start/end is rank, not scores
redis> zrevrange setname start end # start/end is rank, not scores

# return values whose score is between start and stop
redis> zrangebyscore setname start end [withscore]
redis> zrevrangebyscore setname start end [withscores]

redis> zcard setname # length of setname
redis> zcount setname start end # number of values between star/stop scores
redis> zrank setname value # the rank of value in set
redis> zscore setname value # the score of value in set
```

#### 地理信息（geo）

geo低层数据类型为zset。

```shell
redis> geoadd key long lat name
redis> geodist key name1 name2
redis> geopos key name
redis> georadius key long lat distance unit
redis> georadiusbymember key name dist unit

# return a hash string.
# the closer of two positions, the more similar of two strings
redis> geohash key name1 name2 ...
```

#### Hyperloglog

合并结果可能有部分误差。

```shell
redis> pfadd key v1 v2 ...
redis> pfcount key
redis> pfmerge key srckey1 srckey2
```

#### 位图（bitmap）

```shell
redis> setbit key offset value
redis> getbit key offset
redis> bitcount key # options: start/end offset
```

#### 事务

- 语法错误会终止事务编辑，所有命令都没有执行
- 运行时错误只会影响当前命令，之前和之后的命令都不影响

```shell
redis> multi # begin a transaction
redis> ...
redis> ...
...
redis> exec # execute the transaction

# cancel
redis> discard # commands after multi will be discarded
```

#### 乐观锁

使用watch获取乐观锁，事务执行之前如果key改变了，事务则不执行。

```shell
redis> watch key
reids> multi
redis> some command
...
redis> exec # 'watch' will be automatically removed.

redis> unwatch # manually unwatch keys
```

#### 订阅/发布

```shell
# On subscriber side
# Multiple clients can subscribe to a channel and all wiil receive msg
redis> subscribe channel # this client will block and wait

# On publisher side
redis> publish channel message
```

------

### 配置文件（ /etc/redis/redis.conf ）

| 配置项目                    | 说明                                         |
| --------------------------- | -------------------------------------------- |
| include                     | 包含其他配置文件                             |
| bind                        | 绑定的IP地址                                 |
| port                        | 监听的端口号                                 |
| protected-mode              | 只有本地机器可以访问                         |
| databases                   | 数据库的数量，命令行使用select可以选择数据库 |
| daemonize                   | 是否后台运行                                 |
| logfile                     | 日志文件                                     |
| save                        | 保存日志的频率                               |
| rdbcompression              | rbd是否压缩                                  |
| stop-writes-on-bgsave-error | 保存数据出错时，是否停止写操作               |
| rdbchecksum                 | 是否在文件末尾添加CRC4校验值                 |
| rdbfilename                 | 数据库文件名                                 |
| dir                         | 数据库文件位置                               |
| requirepass                 | 设置客户端连接密码；auth命令或-a参数         |
| maxclients                  | 最大客户端连接数                             |
| maxmemory                   | 最大内存，单位为字节                         |
| maxmemory-policy            | 达到最大内存后的删除策略                     |
| maxmemory-samples           | 删除数据时，以多少键为一组进行比较           |
| appendonly                  | 是否开启该功能                               |

### 数据持久化

#### RDB

数据文件：/var/lib/redis/dump.rdb

过程：

1. fork一个子进程
2. 创建一个临时文件用来保存当前的数据
3. 产生rdb文件

执行flushall命令或shutdown命令都会产生rdb文件。

检查rdb文件：redis-check-rdb

#### AOF

记录所有写操作到aof文件中

检测并修复aof文件：redis-check-aof --fix

------

### 集群复制

查看节点状态

```shell
# view node states
# includes:
# node role
# connected followers
reidis> info replication
```

#### 复制集群的基本配置

首先修改配置文件（所有节点都要修改，保持password一致）

```shell
bind <host ip>
masterauth <redis-pass>
requirepass <redis-pass>
```

启动服务，在从节点上执行以下操作

```shell
redis> slaveof <master ip> <master port>
```

使用`info replication`检查集群状态。

#### 复制集群与哨兵

哨兵用于监控master节点，如果master节点无法正常工作，哨兵会挑选一个slave节点置成新master节点。sentinel可以从master节点上自动发现slave节点和其他的sentinel节点。

ubuntu 安装redis sentinel

```shell
sudo apt install redis-sentinel
```

修改配置文件（/etc/redis/sentinel.conf）

```shell
bind <host ip>
protected-mode no
sentinel auth-pass mymaster <redis-password>
sentinel monitor mymaster <master-ip> <master-port> <quorum>
```

启动进程

```shell
redis-sentinel /etc/redis/sentinel.conf
```

应该配置奇数个sentinel，保证quorum数值大于一半的sentinel数量。

------

### 多主节点集群

修改配置文件

```shell
bind <host ip>
masterauth <redis-pass>
requirepass <redis-pass>
cluster-enabled yes
```

创建集群

```shell
# --cluster-replicas <num>: how many replicas a master has.
# In this example, first 3 nodes are masters and the rest nodes are replicas.
redis-cli -a <redia-password> --cluster create \
			172.20.0.2{1..6}:6379 --cluster-replicas 1

# connect to a master node
redis-cli -h redis1 -a <redis-password> -c
```

添加新节点

TODO

重新分配slot

TODO