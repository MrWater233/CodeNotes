# 一.简介

> Redis（Remote Dictionary Server 远程字典服务器）是一个高性能的（K-V）分布式内存数据库，他是单进程的。

特点：

1. 支持数据持久化
2. 不仅支持简单的K-V类型，还提供list、set、zset、hash等数据结构的存储
3. 支持数据备份，即主从模式（master-slave）的数据备份

# 二.基础命令

1. `select 库的序号`：切换数据库
2. `dbsize`：查看当前数据库的`key`数量
3. `flushdb`：清空当前库
4. `flushall`：清空所有库

# 三.五大数据类型

## 0.Key关键字

1. `keys *`：查询当前库中所有key
2. `exist key`：查询某个是否存在（1：存在；0：不存在）
3. `move key 库的序号`：将某个键值对移动到另一个库
4. `expire key`：为给定的key设置过期时间
5. `ttl key`：查看key还有多少秒过期（-1：永不过期；-2：已过期）
6. `type key`：查看key的类型

## 1.字符串（String）

> Redis最基本的类型，单值单value

1. `set/get/del/append/strlen`：增/查/删/追加/查看长度

2. `incr key`：value（数字类型）加一

   `decr key`：：value（数字类型）减一

   `incrby key 增加大小`：value（数字类型）增加指定大小

   `decrby key 减少大小`：value（数字类型）减少指定大小

3. `getrange key 开始位置 结束位置`：获取value指定范围的值

   `setrange key 开始位置 值`：在value指定位置开始替换值

4. `setex key 时间 value`：增加键值对并设置过期时间

   `setnx key value`：在key不存在时才插入

5. `mset key1 value1 key2 value2...`：增加多个键值对

   `msetnx key1 value1 key2 value2...`：只有所有的key都不存在才插入

   `mget key1 key2...`：获取多个值

6. `getset key value`：先获取值再设置value

## 2.列表（List）

> 字符串链表，左右都可以插入添加
>
> 如果值全部移除，对应的键也消失了

1. `lpush key l1 l2 l3...`：增加list，顺序（`l3 l2 l1`）

   `rpush key l1 l2 l3...`：增加list，顺序（`l1 l2 l3`）

   `lrange key 开始位置 结束位置`：查看list范围之间的值，若为`0 -1`则查看整个list

2. `lpop key`：弹出栈顶元素

   `rpop key`：弹出栈底元素

3. `index key 索引下标`：获取指定索引下标的值，从0开始

4. `len key`：获取数组长度

5. `lrem key n value`：删除n个value

6. `ltrim key 开始位置 结束位置`：截取范围内的值并重新赋给key

7. `rpoplpush 源列表 目的列表`：源列表弹出栈底元素并加入到目的列表栈顶

8. `lset key index value`：在指定位置赋值

9. `linsert key before/after value1 value2`：在value1之前/后插入value2

## 3.集合（Set）

> String类型集合，无序无重复的集合，通过HashTable实现

1. `sadd key val1 val2 val3...`：增加一个set，有重复会去重

   `smenbers key `：查看指定set的所有值

   `sismenber key value`：查看指定set是否存在value

2. `scard key`：获取集合元素个数

3. `srem key value`：删除集合中的元素

4. `srandmember key 某个整数`：随机出几个数字

5. `spop key`：随机出栈一个元素

6. `smove key1 key2 key1中的某个值`：将key1里的某个值赋值给key2

7. 数学集合类

   1. `sdiff key1 key2`：差集，得到第一个set中但不在第二个set中的值
   2. `sinter key1 key2`：交集
   3. `sunion key1 key2`：并集

## 4.哈希（Hash）

> 类似Java中的`Map<String,Object>`
>
> K-V模式不变，但是V是一个键值对

1. `hset key k v `：增加一个键值对

   `hget key k`：查找一个键值对

   `hmset key k1 v1 k2 v2...`：增加多个键值对

   `hmget key k1 k2...`：查找多个键值对

   `hgetall key`：查找的所有键值对

   `hdel key k`：删除一个键值对

2. `hlen key`：查找键值对数目

3. `hexists key k`：key中的某个key是否存在

4. `hkeys key`：查找该key下的所有key

   `hvals key`：查找该key下的所有value

5. `hincrby key v 增加的整数`：value增加指定整数

   `hincrbyfloat key v 增加的小数`：vakue增加指定小数

6. `hsetnx key k v`：如果k不存在则增加

## 5.有序集合（Zset：sorted set）

> 每一个元素都关联一个double类型分数，按照分数从小到大排序。Zset元素唯一但是分数可以重复

1. `zadd key score1 value1 score2 value2...`：增加一个zset

2. `zrange key 开始位置 结束位置`：取出指定范围的值，为`0 1`则取出所有

   `zrevrange key 开始位置 结束位置`：逆序取出指定范围的值

   `zrange key 开始位置 结束位置 withscores`：将value和score一起取出

3. `zrangebyscore key 开始score 结束score`：取出指定score范围的值

   - `(开始score`表示不包含开始score

   - 结尾加入`limie num1 num2`类似分页

   `zrevrangebyscore key 结束score 开始score`：你去取出指定score范围的值

4. `zrem key value`：删除元素

5. `zcard key`：统计总个数

   `zcount key score1 score2`：统计score区间内的值的个数

6. `zscore key value`：获取value的score

7. `zrank key value`：得到value的下标

   `zrevrank key value`：逆序获得下标

