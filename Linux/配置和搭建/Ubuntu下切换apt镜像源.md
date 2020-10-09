注：当前配置为Ubuntu16的版本

1. 备份原镜像文件

   ```shell
   mv /etc/apt/sources.list /etc/apt/sources.list.bak
   ```

2. 修改镜像源

   ```shell
   vim /etc/apt/sources.list
   ```

   添加以下内容

   ```shell
   # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
   deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
   deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
   deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
   deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
   
   # 预发布软件源，不建议启用
   # deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
   ```

   具体[不同Ubuntu版本的镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

   查看其他发行版镜像配置的方式

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200924102013.png)

3. 刷新列表

   ```shell
   sudo apt-get update
   ```

4. 更新安装的包

   ```shell
   sudo apt-get upgrade
   ```

   