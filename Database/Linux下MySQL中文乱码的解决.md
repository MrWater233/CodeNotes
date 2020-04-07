1. 登陆Mysql后可以使用`SHOW VARIABLES LIKE 'character%'`查看字符集

   ```shell
   +--------------------------+----------------------------+
   | Variable_name | Value |
   +--------------------------+----------------------------+
   | character_set_client | utf8 |
   | character_set_connection | utf8 |
   | character_set_database | latin1 |
   | character_set_filesystem | binary |
   | character_set_results | utf8 |
   | character_set_server | latin1 |
   | character_set_system | utf8 |
   | character_sets_dir | /usr/share/mysql/charsets/ |
   +--------------------------+----------------------------+
   ```

   如上所示有一些编码是`latin1`编码格式，需要修改Mysql配置文件

2. 查看Mysql默认配置文件路径

   命令：

   ```shell
   mysql *--help|grep 'my.cnf'*
   ```

   结果：

   ```shell
                         order of preference, my.cnf, $MYSQL_TCP_PORT,
   /etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf ~/.my.cnf
   ```

   `/etc/my.cnf`，`/etc/mysql/my.cnf,`，`/usr/local/etc/my.cnf`，`~/.my.cnf` 这些就是Mysql默认会搜寻`my.cnf`的目录，顺序排前的优先。

3. 进入`/etc/my.cnf`文件

   增加或者修改以下内容

   ```cnf
   # 1、在[client]字段里加入default-character-set=utf8，如下：
    
   [client]
   port = 3306
   socket = /var/lib/mysql/mysql.sock
   default-character-set=utf8
    
   # 2、在[mysqld]字段里加入character-set-server=utf8，如下：
    
   [mysqld]
   port = 3306
   socket = /var/lib/mysql/mysql.sock
   character-set-server=utf8
    
   # 3、在[mysql]字段里加入default-character-set=utf8，如下：
    
   [mysql]
   no-auto-rehash
   default-character-set=utf8
   ```

4. 重启Mysql

   `service mysql restart`

5. 再次登陆Mysql通过`SHOW VARIABLES LIKE 'character%'`查看编码发现都变为utf8就成功了