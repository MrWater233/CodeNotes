# 安装

1. 官网下载安装`VirtualBox`

2. 官网下载安装`Vagrant`

   可以使用`vagrant`命令检测是否安装成功

3. 进入`VirtualBox`修改虚拟机默认安装目录

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200923155449.png)

4. [下载并安装Vagrant box镜像](https://c4ys.com/archives/1230)

   1. 下载镜像

   2. 添加下载的`vagrant box`到`vagrant list`

      ```shell
      vagrant box add centos7 CentOS-7.box
      ```

   3. 初始化一个虚拟机并使用刚才添加的`vagrant box`

      ```shell
      vagrant init centos7
      ```

   4. 启动`vagrant box`虚拟机

      ```shell
      vagrant up
      ```

5. 连接进入虚拟机

   ```shell
   vagrant ssh
   ```

6. 退出虚拟机

   ```shell
   exit;
   ```

7. 可以在`VirtualBox`中关闭虚拟机，也可以使用命令关闭

   ```shell
   vagrant halt
   ```

8. 下次想要启动虚拟机可以在`VirtualBox`中启动

   也可以在创建`Vagrant box`的路径下输入以下命令

   ```shell
   vagrant up
   ```

---------

- 获取管理员权限

  ```shell
  su root
  ```

  默认密码为：`vagrant`


# 网络配置

> `VirutalBox`默认网络设置是端口转发，如果想要在外部系统通过端口访问虚拟机中的软件需要设置端口转发，较为麻烦

1. 使用`ipconfig`

   查看本机`VirtualBox`的网段

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200408184918.png)

2. 更改创建box路径下生成的`Vagrantfile`文件

   将35行ip改为第1步同一网段即可，如👇

   ```
   config.vm.network "private_network", ip: "192.168.56.10"
   ```

3. 重启虚拟机

   ```shell
   vagrant reload
   ```

4. 主机和虚拟机能相互ping通说明成功

