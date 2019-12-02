# 安装青岛OJ

### 安装操作系统

略

### 获取root权限

```sh
sudo passwd root   #然后输入密码
su root 
```

### 更换源

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak #备份源文件
sudo gedit /etc/apt/sources.list #将里面的内容换为阿里云的
# 阿里云的镜像
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

sudo apt-get update  #更新源
```

### 安装相关软件

```shell
sudo apt-get update && sudo apt-get install -y vim python-pip curl git
pip install docker-compose
sudo curl -sSL https://get.daocloud.io/docker | sh #安装docker
```

### 更换docker镜像

```shell
sudo vim /etc/docker/daemon.json #新建文件，加入以下内容
###
{
    "registry-mirrors":["https://docker.mirrors.ustc.edu.cn"]
}
###
#重启守护进程
sudo systemctl daemon-reload
sudo systemctl restart docker
```



### 安装OJ

```shell
#到空间大的位置
git clone -b 2.0 https://github.com/QingdaoU/OnlineJudgeDeploy.git && cd OnlineJudgeDeploy #克隆代码
docker-compose up -d #启动服务，在root用户下运行
```

### 安装ssh

```sh
sudo apt-get update
sudo apt-get install openssh-server
gedit /etc/ssh/ssh_config
#注释掉 PermitRootLogin without-password
#加入 PermitRootLogin yes
service ssh start #启动
```



### 管理

```shell

docker ps  #显示所有的容器
docker stop name
docker start name

docker ps # 查看所有正在运行容器
docker stop containerId # containerId 是容器的ID

docker ps -a # 查看所有容器
docker ps -a -q # 查看所有容器ID

docker stop $(docker ps -a -q) #  stop停止所有容器
docker  rm $(docker ps -a -q) #   remove删除所有容器

sudo /etc/init.d/nginx stop #停止nginx
#强制停止
sudo ps -ef | grep nginx  # 查询nginx PID 此处为28444
#sudo netstat -a | grep 28444
sudo kill -quit 28444 #关闭nginx

sudo /etc/init.d/nginx start #启动ngnix

```

### 导入旧版本数据

* zip的测试数据包移动到OnlineJudgeDeploy/data/backend/test_case下解压

```shell
unzip   testcase.zip  #解压
```

* 复制old_json到某一目录(root权限)

```shell
docker cp old_data.json oj-backend:/app/utils/ 
docker exec -it oj-backend /bin/sh
cd utils
python3 migrate_data.py old_data.json
```

###  跟踪网络节点

```shell
tracert  www.baidu.com
```

