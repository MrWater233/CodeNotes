1. 下载wget

   ```shell
   yum -y install wget
   ```

2. 进入镜像源目录

   ```shell
   cd /etc/yum.repos.d
   ```

3. 备份旧的镜像文件

   ```shell
   mv CentOS-Base.repo CentOS-Base.repo.bak
   ```

4. 下载阿里源镜像

   ```shell
   wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
   ```

5. 清理缓存

   ```shell
   yum clean all
   ```

6. 重新生成缓存

   ```shell
   yum makecache
   ```

7. 更新软件

   ```shell
   yum -y update
   ```

   

