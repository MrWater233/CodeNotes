# 主从复制

1. 安装wegt

   ```shell
   yum -y install wget
   ```

2. 下载redis配置文件

   ```shell
   wget http://download.redis.io/redis-stable/redis.conf
   ```

3. 并复制三份，两份为从，一份为主

   ```shell
   [root@localhost conf]# ls
   redis.conf  redis-master.conf  redis-slave1.conf  redis-slave2.conf
   ```

4. 修改配置文件

   - 主机配置文件

     ```yaml
     # 允许所有IP访问Redis
     bind 0.0.0.0
     ```

   - 两个从机配置文件

     ```yaml
     # 允许所有IP访问Redis
     bind 0.0.0.0
     # 设置主机的地址：IP 端口。接下去通过--lin进行地址解析，docker自动将master解析为地址
     replicaof master 6379
     ```

     Tip：第二个配置也可以不在配置文件中指定，而在redis0-cli中输入命令`slaveof master 6379`，但是这样每次从机重启都要重新输入命令指定主机

5. 启动容器

   - 主机

     ```shell
     docker run --name redis-master \
     -v /mydata/redis-test/conf/redis-master.conf:/usr/local/etc/redis/redis.conf \
     -d redis redis-server /usr/local/etc/redis/redis.conf
     ```

   - 从机1

     ```shell
     docker run --name redis-slave1 \
     -v /mydata/redis-test/conf/redis-slave1.conf:/usr/local/etc/redis/redis.conf \
     --link redis-master:master \
     -d redis redis-server /usr/local/etc/redis/redis.conf
     ```

   - 从机2

     ```shell
     docker run --name redis-slave2 \
     -v /mydata/redis-test/conf/redis-slave2.conf:/usr/local/etc/redis/redis.conf \
     --link redis-master:master \
     -d redis redis-server /usr/local/etc/redis/redis.conf
     ```

6. 查看Redis主从状态

   ```shell
# 进入哨兵容器
   docker exec -it 容器名/容器ID redis-cli
# 查看主从状态
   info replication
   ```

`--link`：`--link 关联容器名/关联容器ID:alias`。其中，alias是源容器在link下的别名。

这样容器可以通过alias也可以通过关联容器名（关联容器ID）进行通信，因为Docker会自动进行DNS解析

# 哨兵模式

## 单哨兵

1. 下载哨兵配置文件

   ```shell
   wget http://download.redis.io/redis-stable/sentinel.conf
   ```

2. 修改配置文件`sentinel.conf`

   ```yaml
   # 参数说明:mymaster：哨兵名称；master 6379：被监控主机IP和端口；1：执行故障恢复操作前至少需要几个哨兵节点同意
   # 因为当前只配了一个哨兵，所以设为1，一般以一主二仆要配置三个哨兵，那么最后一个值就配为2
   sentinel monitor mymaster master 6379 1
   
   # 配置日志文件，默认会在容器/data下，如果不需要可以不管
   logfile "sentinel.log"
   ```

3. 运行哨兵

   ```shell
   docker run --name redis-sentinel \
   -v /mydata/redis-test/conf/sentinel.conf:/usr/local/etc/redis/sentinel.conf \
   --link redis-master:master \
   -d redis redis-sentinel /usr/local/etc/redis/sentinel.conf
   ```

以上方法针对只有一个哨兵的情况，这时候哨兵也有可能宕机，导致无法及时切换主机

## 多哨兵

> 以下演示配置三个哨兵

1. 下载哨兵配置文件

   ```shell
   wget http://download.redis.io/redis-stable/sentinel.conf
   ```

2. 修改配置文件

   ```yaml
   # 关闭安全模式
   protected-mode no
   
   # 最后一个值修改为2，即需要两个哨兵节点投票同意才能故障恢复
   sentinel monitor mymaster master 6379 2
   ```

3. 复制三份

   ```shell
   [root@localhost conf]# ls
   sentinel1.conf  sentinel2.conf  sentinel3.conf
   ```

4. 启动三个哨兵容器

   ```shell
   docker run --name redis-sentinel1 \
   -v /mydata/redis-test/conf/sentinel1.conf:/usr/local/etc/redis/sentinel.conf \
   --link redis-master:master \
   -d redis redis-sentinel /usr/local/etc/redis/sentinel.conf
   ```

   ```shell
   docker run --name redis-sentinel2 \
   -v /mydata/redis-test/conf/sentinel2.conf:/usr/local/etc/redis/sentinel.conf \
   --link redis-master:master \
   --link redis-sentinel1:sentinel \
   -d redis redis-sentinel /usr/local/etc/redis/sentinel.conf
   ```

   ```shell
   docker run --name redis-sentinel3 \
   -v /mydata/redis-test/conf/sentinel3.conf:/usr/local/etc/redis/sentinel.conf \
   --link redis-master:master \
   --link redis-sentinel2:sentinel \
   -d redis redis-sentinel /usr/local/etc/redis/sentinel.conf
   ```

   <font color=red>注意后两个容器需要多配置和前一个哨兵的`--link`，否则哨兵之间无法进行通信</font>

5. 查看哨兵运行情况

   ```shell
   # 进入哨兵容器
   docker exec -it 哨兵容器名/哨兵容器ID bash
   # 进入哨兵
   redis-cli -p 26379
   # 查看哨兵运行情况
   info sentinel
   ```

   



