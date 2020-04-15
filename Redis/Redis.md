# 一.简介

> Redis（Remote Dictionary Server 远程字典服务器）是一个高性能的（K-V）分布式内存数据库，他是单进程多线程的

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

# 四.配置文件

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
    daemonize no
2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
    pidfile /var/run/redis.pid
3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
    port 6379
4. 绑定的主机地址
    bind 127.0.0.1
5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
    timeout 300
6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
    loglevel verbose
7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
    logfile stdout
8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
    databases 16
9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
    save <seconds> <changes>
    Redis默认配置文件中提供了三个条件：
    save 900 1
    save 300 10
    save 60 10000
    分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
    rdbcompression yes
11. 指定本地数据库文件名，默认值为dump.rdb
    dbfilename dump.rdb
12. 指定本地数据库存放目录
    dir ./
13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
    slaveof <masterip> <masterport>
14. 当master服务设置了密码保护时，slav服务连接master的密码
    masterauth <master-password>
15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
    requirepass foobared
16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
    maxclients 128
17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
    maxmemory <bytes>
18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
    appendonly no
19. 指定更新日志文件名，默认为appendonly.aof
      appendfilename appendonly.aof
20. 指定更新日志条件，共有3个可选值： 
    no：表示等操作系统进行数据缓存同步到磁盘（快） 
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
    everysec：表示每秒同步一次（折衷，默认值）
    appendfsync everysec
21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
      vm-enabled no
22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
      vm-swap-file /tmp/redis.swap
23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
      vm-max-memory 0
24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
      vm-page-size 32
25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
      vm-pages 134217728
26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
      vm-max-threads 4
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
    glueoutputbuf yes
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
    hash-max-zipmap-entries 64
    hash-max-zipmap-value 512
29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
    activerehashing yes
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
    include /path/to/local.conf
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
        include /path/to/local.conf

# 五.持久化

> 当AOF和RDB同时存在时，Redis优先使用AOF

## RDB（Redis DataBase）

> 指定的时间间隔将内存中的数据集快照写入磁盘
>
> Redis运行会单独fork一个进程来进行持久化，先将数据保存进临时文件，数据保存完成后再替换上次的持久化文件，整个过程主进程不进行IO操作，确保了高性能

- 持久化配置：相关配置在`SNAPSHOUTTING`中去配置，详见第4.8

  默认配置：

  - 一分钟更改一万次触发
  - 五分钟更改10次触发
  - 十五分钟更改1次触发

- 立刻进行备份指令：

  - `save`：阻塞保存
  - `bgsave`：异步保存

- 停止RDB：`redis-cli config set save ""`

- 缺点：

  - 在备份时间间隔里如果遇到突发情况，那么数据就会丢失
  - fork会占用双倍内存

**注意**：当关闭redis的时候会迅速将内存中的数据进行备份产生新的`dump.rdb`文件

## AOF（Append Only File）

> 用日志的形式记录每一个写操作，将Redis执行过的写指令记录下来

- 配置：相关配置在`APPEND ONLY MODE`中去配置，详见4.17、4.19

  ` appendfsync everysec`：

  - `always`：同步持久化，每次数据变更都会立刻记录，性能较差但是完整性好
  - `everysec`：异步操作，每秒记录，如果一秒内宕机，会有数据丢失
  - `no`：表示等操作系统进行数据缓存同步到磁盘

- 当AOF日志文件出错，可以使用同级目录的`redis-check-aof`去修复

  ```shell
  redis-check-aof --fix appendonly.aof
  ```

- 重写机制：

  - 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令`bgrewriteaof`

  - Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

# 六.事务

> 一个队列中，一次性、顺序性、排他性的执行一系列命令

- 执行流程

   1. 开启事务

      ```shell
      MULTI
      ```

   2. 一系列操作。。。

   3. 提交事务

      ```shell
      EXEC
      ```

      放弃事务

      ```shell
      DISCARD
      ```

- Redis是部分支持事务，

   - 当进行操作时检查出语法错误如果立即报错，此时提交事务，那么这些操作不会执行
   - 如果在提交以后某个操作出现了错误，那么只是错误的操作不会执行，其他操作继续执行

- 乐观锁和悲观锁

   - 乐观锁：不会给数据表上锁，但是每一条记录有一个版本号，每次更新数据表的时候会通过版本号前后对比判断该条数据是否已经更新。即提交的版本必须大于当前记录的版本才能更新
   - 悲观锁：每次操作数据表，将整个数据表上锁，保持了数据库的一致性，但是并发差

- 监控：

   - watch指令：

      类似乐观锁，事务提交时，如果Key的值已被别的客户端改变，那么事务就会失败
      比如某个list已被别的客户端push/pop过了，整个事务队列都不会被执行

   - unwatch指令：

      放弃watch对**所有**key的监控

   - 一旦执行exec之前所有的监控锁会被取消

# 七.主从复制

> 主机数据更新以后自动同步到备机，Master以写为主，Slave以读为主
>
> 配从（库）不配主（库）

## 命令

- `info replication`：查看当前Redis的主从信息
- `slaveof ip port`：配为从机并关联主机
- `slaveof no one`：从机变为主机

## 常用配置关系

1. 一主二从

   一个主机两个从机的配置

2. 薪火相传

   一个主机，两个从机进行接力的配置

3. 反客为主

   主机宕机，从机通过`slaveof no one`，成为主机

## 注意

1. 从机不能写入数据
2. 主机宕机，从机待命 ，直到主机恢复
3. 从机宕机，重启后，如果没有在配置文件中配置主从关系，会重新变为主机

## 哨兵模式

> 哨兵监控主机是否故障，如果主机故障，从机根据投票数自动转为主机

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200411212402.png)

- 如果主机宕机，从机完成选举以后，主机重新上线后会变为从机
- 一组哨兵能够监控多组master