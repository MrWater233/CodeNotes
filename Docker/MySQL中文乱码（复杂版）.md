1. 通过容器id进入容器内部

   ```shell
   docker exec -it container_id bash
   ```

2. 进入Mysql通过`SHOW VARIABLES LIKE 'character%';`查看编码，发现部分编码为`latin1`

3. 容器的系统默认为debian，而且没有安装vim，并且下载速度感人，所以先配置一波镜像

   1. 查看debian版本

      ```shell
      cat /etc/debian_version
      ```

   2. 查看该版本[阿里云的镜像配置](https://developer.aliyun.com/mirror/)

   3. 进入`/etc/apt`，`sources.list`就是镜像配置文件

   4. 备份文件

      ```shell
      cp sources.list sources.list_bak
      ```

   5. 删除`sources.list`

      ```shell
      rm sources.list
      ```

   6. 重写`sources.list`

      我用的是debian9

      ```shell
      echo "deb http://mirrors.aliyun.com/debian/ stretch main non-free contrib" > sources.list
      ```

4. 更新源库并安装vim

   ```shell
   apt-get update
   ```

   ```shell
   apt-get install vim
   ```

5. 进入`/etc/mysql`，使用vim修改`my.cnf`

   1. 因为这时候的vim不支持复制粘贴，需要用以下方法解决

      - 在普通模式下键入下面的值

         ```shell
         :set mouse-=a
         ```

         但是每次重新打开vim都需要重新设置

      - 编辑`~/.vimrc`文件，加入以下代码便可一劳永逸

         ```shell
         if has('mouse')
             set mouse-=a 
         endif
         ```

   2. 复制粘贴以下配置

      ```cnf
      [client]
      port = 3306
      socket = /var/lib/mysql/mysql.sock
      default-character-set=utf8
      
      [mysqld]
      port = 3306
      socket = /var/lib/mysql/mysql.sock
      character-set-server=utf8
      
      [mysql]
      no-auto-rehash
      default-character-set=utf8
      ```

6. 回到docker，输入以下命令重启容器

   ```shell
   docker restart container_id
   ```

   

