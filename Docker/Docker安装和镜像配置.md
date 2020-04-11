1. [Centos安装Docker](https://docs.docker.com/engine/install/centos/)

2. 设置Docker开机自启

   ```shell
   sudo systemctl enable docker
   ```

3. 配置阿里云镜像加速

   1. 进入阿里云控制台，找到`容器镜像服务`

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200408221421.png)

   2. 镜像中心->镜像加速器

   3. 找到Centos并执行相关命令