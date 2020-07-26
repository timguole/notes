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



### 配置文件（ /etc/redis/redis.conf ）

#### databases <number>

数据库数量。默认为16，使用 select <db-id> 可以选择要操作的数据库。
