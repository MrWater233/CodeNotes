1. 下载镜像

   `docker pull mysql`

2. 创建数据卷

   `docker volume create mysql-vol`

3. 启动容器，将实际路径跟数据卷映射并将主机的3306跟容器的3306端口映射

   ```shell
   docker run --rm -d -e MYSQL_ROOT_PASSWORD=123456 \
         -v mysql-vol:/var/lib/mysql \
         -p 3306:3306 mysql
   ```

   