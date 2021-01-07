1. 下载镜像

   - 5.7

     `docker pull mysql:5.7`

   - 8.0

     `docker pull mysql:8.0`

2. 创建实例并启动

   - 5.7

     ```shell
     docker run -p 3306:3306  --name mysql \
     -v /home/docker/mysql/log:/var/log/mysql \
     -v /home/docker/mysql/data:/var/lib/mysql \
     -v /home/docker/mysql/conf:/etc/mysql \
     -e MYSQL_ROOT_PASSWORD=123456 \
     -d mysql:5.7
     ```

     - `-p`：端口映射
     - `-v /home/docker/mysql/log:/var/log/mysql`：将日志文件挂载到主机
     - `-v /home/docker/mysql/data:/var/lib/mysql`：将持久化数据文件挂载到主机
     - `-v /home/docker/mysql/conf:/etc/mysql`：将配置文件夹挂载到主机
     - `-e MYSQL_ROOT_PASSWORD=12345`：初始化root用户名和密码

   - 8.0

     ```shell
     docker run -p 3306:3306  --name mysql \
     -v /home/docker/mysql/log:/var/log/mysql \
     -v /home/docker/mysql/data:/var/lib/mysql \
     -v /home/docker/mysql/conf:/etc/mysql \
     -e MYSQL_ROOT_PASSWORD=123456 \
     -d mysql:8.0
     ```

3. 更改字符编码

   ```shell
   vi /home/docker/mysql/conf/my.cnf
   ```

   - 5.7

     ```txt
     [client]
     default-character-set=utf8
     
     [mysql]
     default-character-set=utf8
     
     [mysqld]
     init_connect='SET collation_connection = utf8_unicode_ci'
     init_connect='SET NAMES utf8'
     character-set-server=utf8
     collation-server=utf8_unicode_ci
     skip-character-set-client-handshake
     skip-name-resolve
     default-time_zone='SYSTEM'
     ```

   - 8.0

     ```txt
     [client]
     default-character-set=utf8
     [mysql]
     default-character-set=utf8
     
     [mysqld]
     user=mysql
     character-set-server=utf8
     default_authentication_plugin=mysql_native_password
     secure_file_priv=/var/lib/mysql
     server-id=100 
     log-bin=mysql-bin
     default-time_zone='+8:00'
     ```

     - 配置远程连接

       进入docker容器

       ```shell
       docker exec -it mysql bash
       mysql -u root -p
       use mysql
       select host, user, authentication_string, plugin from user;
       ```
       
       可以发现没有`host`为`%`的root用户，但是有一个为`localhost`的用户，需要修改它
       
       ```shell
       update user set host = '%' where user = 'root';
       FLUSH PRIVILEGES;
       alter user 'root'@'%' identified with mysql_native_password by '123456';
       FLUSH PRIVILEGES;
       ```
     
   - 参考：

     - https://blog.csdn.net/pannubi/article/details/104861672
     - https://www.cnblogs.com/tarencez/p/11841587.html

4. 重启容器

   ```shell
   docker restart mysql
   ```

5. 设置Docker启动容器自启

   ```shell
   docker update mysql --restart=always
   ```

   