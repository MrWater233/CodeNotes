1. 下载镜像

   ```shell
   docker pull redis
   ```

2. 创建配置文件

   ```shell
   mkdir -p /mydata/redis/conf
   touch /mydata/redis/conf/redis.conf
   ```

3. 创建并运行容器

   ```shell
   docker run -p 6379:6379 --name redis \
   -v /mydata/redis/data:/data \
   -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
   -d redis redis-server /etc/redis/redis.conf \
   --requirepass "123456"
   ```

   - 可以在Docker中直接使用Redis

     ```shell
     docker exec -it redis redis-cli
     ```

4. 开启aof持久化

   ```shell
   vi /mydata/redis/conf/redis.conf
   ```

   添加以下内容

   ```text
   appendonly yes
   ```

5. 重启Redis

   ```shell
   docker restart redis
   ```

6. 设置Docker启动容器自启

   ```shell
   docker update redis --restart=always
   ```

   

