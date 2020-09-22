有三种方法

1. 关闭SELinux

   1. 查看SELinux状态

      ```shell
      sestatus -v
      getenforce
      ```

   2. 关闭SELinux

      ```shell
      setenforce 0
      ```

2. 创建容器时给容器增加权限

   ```shell
    --privileged=true
   ```

   如

   ```shell
   docker run -p 3306:3306 --privileged=true --name mysql \
   -v /mydata/mysql/log:/var/log/mysql \
   -v /mydata/mysql/data:/var/lib/mysql \
   -v /mydata/mysql/conf:/etc/mysql \
   -e MYSQL_ROOT_PASSWORD=123456 \
   -d mysql:5.7
   ```

3. 在SELinux添加规则，修改挂载目录de

