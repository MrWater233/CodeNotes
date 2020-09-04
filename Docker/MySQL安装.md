1. 下载镜像

   `docker pull mysql:5.7`

2. 创建实例并启动

   ```shell
   docker run -p 3306:3306  --name mysql \
   -v /mydata/mysql/log:/var/log/mysql \
   -v /mydata/mysql/data:/var/lib/mysql \
   -v /mydata/mysql/conf:/etc/mysql \
   -e MYSQL_ROOT_PASSWORD=123456 \
   -d mysql:5.7
   ```

   - `-p`：端口映射
   - `-v /mydata/mysql/log:/var/log/mysql`：将日志文件挂载到主机
   - `-v /mydata/mysql/data:/var/lib/mysql`：将持久化数据文件挂载到主机
   - `-v /mydata/mysql/conf:/etc/mysql`：将配置文件夹挂载到主机
   - `-e MYSQL_ROOT_PASSWORD=12345`：初始化root用户名和密码

3. 更改字符编码

   ```shell
   vi /mydata/mysql/conf/my.cnf
   ```

   添加以下内容

   ```text
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
   ```

4. 重启容器

   ```shell
   docker restart mysql
   ```

5. 设置Docker启动容器自启

   ```shell
   docker update mysql --restart=always
   ```

   