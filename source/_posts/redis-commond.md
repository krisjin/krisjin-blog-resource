title: Redis 命令
date: 2015-06-09 17:30:44
categories: NoSQL
tags: [redis,nosql]
---

## 连接操作相关的命令

QUIT：关闭连接（connection）；  
AUTH：简单密码认证。  
关于密码验证：  

1. 如果 redis 监听回环IP之外的地址 任何人都可以读取其信息，所以安全问题需要考虑；
2. redis 服务器的速度众所周知，因此官方文件中提醒设置比较复杂的密码，防止机器破解；
3. 首先需要在redis的配置文件 redis.conf 中 requirepass 注释掉的内容，设置 requirepass testpassword ；
4. 重新启动 redis (service redis-server restart) 即需密码验证；
5. 验证 auth testpassword, testpassword 是配置文件中设置的 requirepass 值。

## 适合全体类型的命令

* EXISTS(key) 确认一个 key 是否存在；  
* DEL(key) 删除一个 key；  
* TYPE(key) 返回值的类型；  
* KEYS(pattern) 返回满足给定 pattern 的所有 key；  
* RANDOMKEY：随机返回 key 空间的一个key；  
* RENAME(oldname, newname) 将 key 由 oldname 重命名为 newname，若 newname 存在则删除 newname 表示的 key；  
* DBSIZE：返回当前数据库中 key 的数目；  
* EXPIRE(key,ttl) 设定一个 key 的生存时间 ttl（s）；  
* TTL(key) 获得一个 key 的活动时间；  
* SELECT(index) 按索引查询；  
* MOVE(key, dbindex) 将当前数据库中的 key 转移到有 dbindex 索引的数据库；  
* FLUSHDB：删除当前选择数据库中的所有 key；  
* FLUSHALL：删除所有数据库中的所有 key。  

## 对 STRING 操作的命令

* SET(key, value) 给数据库中名称为 key 的 string 赋予值 value；
* GET(key) 返回数据库中名称为 key 的 string 的 value；
* GETSET(key, value) 给名称为 key 的 string 赋予上一次的value；
* MGET(key1, key2,…, key{$n}) 返回库中多个 string（它们的名称为key1，key2…）的value；
* SETNX(key, value) 如果不存在名称为 key 的 string，则向库中添加 string，名称为 key，值为 value；
* SETEX(key, time, value) 向库中添加 string（名称为key，值为value）同时，设定过期时间time；
* MSET(key1, value1, key2, value2,…key{$n}, value{$n}) 同时给多个 string 赋值，名称为 key{$i} 的 string 赋值 value{$i}；
* MSETNX(key1, value1, key2, value2,…key{$n}, value{$n}) 如果所有名称为 key{$i} 的 string 都不存在，则向库中添加 string，名称 key{$i} 赋值为 value{$i}；
* INCR(key) 名称为 key 的 string 增1操作；
* INCRBY(key, integer) 名称为 key 的 string 增加 integer；
* DECR(key) 名称为 key 的 string 减1操作；
* DECRBY(key, integer) 名称为 key 的 string 减少 integer；
* APPEND(key, value) 名称为 key的 string 的值附加 value；
* SUBSTR(key, start, end) 返回名称为 key 的 string 的 value 的子串。

## 对无索引序列 LIST 操作的命令

* RPUSH(key, value) 在名称为 key 的 list 尾添加一个值为 value 的元素；
* LPUSH(key, value) 在名称为 key 的 list 头添加一个值为 value 的 元素；
* LLEN(key) 返回名称为 key 的 list 的长度；
* LRANGE(key, start, end) 返回名称为 key 的 list 中 start 至 end 之间的元素（下标从0开始，下同）；
* LTRIM(key, start, end) 截取名称为 key 的 list，保留 start 至 end 之间的元素；
* LINDEX(key, index) 返回名称为 key 的 list 中 index 位置的元素；
* LSET(key, index, value) 给名称为 key 的 list 中 index 位置的元素赋值为 value；
* LREM(key, count, value) 删除 count 个名称为 key 的 list 中值为value的元素。count 为0，删除所有值为 value 的元素，count>0从 头至尾删除 count 个值为 value 的元素，count<0从尾到头删除|count|个值为value的元素；
* LPOP(key) 返回并删除名称为key的list中的首元素；
* RPOP(key) 返回并删除名称为key的list中的尾元素；
* BLPOP(key1, key2,… key{$n}, timeout) LPOP 命令的 block 版本。即当 timeout 为0时，若遇到名称为 key{$i} 的 list 不存在或该 list 为空，则命令结束。如果 timeout>0，则遇到上述情况时，等待 timeout 秒，如果问题没有解决，则对 key{$i}+1 开始的 list 执行 pop 操作；
* BRPOP(key1, key2,… key{$n}, timeout) RPOP 的 block 版本。参考上一命令；
* RPOPLPUSH(srckey, dstkey) 返回并删除名称为 srckey 的 list 的尾元素，并将该元素添加到名称为 dstkey 的 list 的头部。

## 对有索引无序集合 SET 操作的命令

* SADD(key, member) 向名称为 key 的 set 中添加元素 member；
* SREM(key, member) 删除名称为 key 的 set 中的元素 member；
* SPOP(key) 随机返回并删除名称为 key 的 set 中一个元素；
* SMOVE(srckey, dstkey, member) 将 member 元素从名称为 srckey 的集合移到名称为 dstkey 的集合；
* SCARD(key) 返回名称为 key 的 set 的基数；
* SISMEMBER(key, member) 测试 member 是否是名称为 key 的 set 的元素；
* SINTER(key1, key2,…key{$n}) 求交集；
* SINTERSTORE(dstkey, key1, key2,…key{$n}) 求交集并将交集保存到 dstkey 的集合；
* SUNION(key1, key2,…key{$n}) 求并集；
* SUNIONSTORE(dstkey, key1, key2,…key{$n}) 求并集并将并集保存到 dstkey 的集合；
* SDIFF(key1, key2,…key{$n}) 求差集；
* SDIFFSTORE(dstkey, key1, key2,…key{$n}) 求差集并将差集保存到 dstkey 的集合；
* SMEMBERS(key) 返回名称为 key 的 set 的所有元素；
* SRANDMEMBER(key) 随机返回名称为 key 的 set 的一个元素。

## 对有序集合 ZSET（sorted set）操作的命令

* ZADD(key, score, member) 向名称为 key 的 zset 中添加元素 member，score 用于排序。如果该元素已经存在，则根据score更新该元素的顺序；
* ZREM(key, member) 删除名称为 key 的 zset 中的元素 member；
* ZINCRBY(key, increment, member) 如果在名称为 key 的 zset 中已经存在元素 member，则该元素的 score 增加 increment；否则向集合中添加该元素，其 score 的值为 increment；
* ZRANK(key, member) 返回名称为 key 的 zset（元素已按 score 从小到大排序）中 member 元素的 rank（即 index，从0开始），若没有 member 元素，返回 “null”；
* ZREVRANK(key, member) 返回名称为 key 的 zset（元素已按 score 从大到小排序）中 member元素的 rank（即 index，从0开始），若没有 member 元素，返回 “null”；
* ZRANGE(key, start, end) 返回名称为 key 的 zset（元素已按 score 从小到大排序）中的 index 从 start 到 end 的所有元素；
* ZREVRANGE(key, start, end) 返回名称为 key 的 zset（元素已按 score 从大到小排序）中的 index 从 start 到 end 的所有元素；
* ZRANGEBYSCORE(key, min, max) 返回名称为 key 的 zset 中 score >= min 且 score <= max 的所有元素；
* ZCARD(key) 返回名称为 key 的 zset 的基数；
* ZSCORE(key, element) 返回名称为 key 的 zset 中元素 element 的 score；
* ZREMRANGEBYRANK(key, min, max) 删除名称为 key 的 zset 中 rank >= min 且 rank <= max 的所有元素；
* ZREMRANGEBYSCORE(key, min, max) 删除名称为 key 的 zset 中 score >= min 且 score <= max 的所有元素；
* ZUNIONSTORE / ZINTERSTORE(dstkeyN, key1,…,keyN, WEIGHTS w1,…wN, AGGREGATE SUM|MIN|MAX) 对N个 zset 求并集和交集，并将最后的集合保存在 dstkeyN 中。对于集合中每一个元素的 score，在进行 AGGREGATE 运算前，都要乘以对于的 WEIGHT 参数。如果没有提供 WEIGHT，默认为1。默认的 AGGREGATE 是 SUM，即结果集合中元素 的 score 是所有集合对应元素进行 SUM 运算的值，而 MIN 和 MAX 是指，结果集合中元素的 score 是所有集合对应元素中最小值和最大值。

## 对有序列表 HASH 操作的命令

* HSET(key, field, value) 向名称为 key 的 hash 中添加元素 field=value；
* HGET(key, field) 返回名称为 key 的 hash 中 field 对应的 value；
* HMGET(key, field1, …,field{$n}) 返回名称为 key 的 hash 中 field{$i} 对应的 value；
* HMSET(key, field1, value1,…,field{$n}, value{$n}) 向名称为 key 的 hash 中添加元素 field{$i}=value{$i}；
* HINCRBY(key, field, integer) 将名称为 key 的 hash 中 field 的 value 增加 integer；
* HEXISTS(key, field) 名称为 key 的 hash 中是否存在键为 field 的域；
* HDEL(key, field) 删除名称为 key 的 hash 中键为 field 的域；
* HLEN(key) 返回名称为 key 的 hash 中元素个数；
* HKEYS(key) 返回名称为 key 的 hash 中所有键；
* HVALS(key) 返回名称为 key 的 hash 中所有键对应的 value；
* HGETALL(key) 返回名称为 key 的 hash 中所有的键（field）及其对应的 value。

## 持久化

* SAVE：将数据同步保存到磁盘；
* BGSAVE：将数据异步保存到磁盘；
* LASTSAVE：返回上次成功将数据保存到磁盘的 UNIX 时戳；
* SHUNDOWN：将数据同步保存到磁盘，然后关闭服务。

## 远程服务控制

* INFO：提供服务器的信息和统计；
* MONITOR：实时转储收到的请求；
* SLAVEOF：改变复制策略设置；
* CONFIG：在运行时配置 Redis 服务器。