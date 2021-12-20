> Kubernetes是一个开源的**容器编排引擎**，用来对容器化应用进行**自动化部署**、**缩容**和**管理**。

# 一.安装

## 1.1 Docker安装

1. [Centos安装Docker](https://docs.docker.com/engine/install/centos/)

2. 设置Docker开机自启

   ```shell
   sudo systemctl enable docker
   ```

3. 配置镜像加速

   1. 进入阿里云控制台，找到`容器镜像服务`

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200408221421.png)

   2. 镜像中心->镜像加速器

   3. 找到Centos并执行相关命令

## 1.2 MiniKube安装

> Minikube是一种轻量级的Kubernetes实现，可在本地计算机上创建VM并部署仅包含一个节点的简单集群

1. [MiniKube安装](https://minikube.sigs.k8s.io/docs/start/)

2. 启动MiniKube

   ```shell
   minikube start --image-mirror-country='cn'
   ```

   如果启动失败，就先执行`minikube delete`再重新执行上面这个命令

3. 查看MiniKube的版本

   ```shell
   minikube version
   ```
   
3. 设置`kubectl`的alias

   修改文件`vim ~/.bashrc`追加

   ```shell
   alias kubectl="minikube kubectl --"
   ```

   让配置生效

   `source ~/.bashrc`

# 二.核心概念

## 2.1 Kubernetes集群

**Kubernetes能够协调一个高可用计算机集群，每个计算机作为独立单元互相连接工作**

一个 Kubernetes 集群包含两种类型的资源:

- **Control Plane(Master)**：负责调度整个集群。它负责协调集群中的所有活动如调度应用、维护应用状态、应用扩容和应用更新。
- **Nodes**： 负责运行应用，他被`Control Plane`所管理。他可以是一个虚拟机也可以是一个物理机。每个`Node`都有`Kubelet`，它是管理`Node`和`Control Plane`与`Node`之间通信管理的代理。

一个生产级流量的Kubernetes集群至少应该有三个`Node`

![img](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112171401213.svg)

在K8S上部署应用的时候只需要告诉`Control Plane`启动应用的容器的命令，`Control Plane`就会编排容器在集群的`Node`上运行。

## 2.2 Deployment

完成了K8S集群部署我们就能在上面部署应用容器了，这时候就会用到`Deployment`

`Deployment`负责指挥K8S如何创建和更新应用程序的实例。

我们创建一个`Deployment`以后`Control Plane`就会调度包含`Deployment`的应用实例部署到各个`Node`上。

创建`Deployment`以后`K8S Deployment Controller`会持续监视这些`Node`上的实例，如果实例的`Node`被关闭或者删除，那么`K8S Deployment Controller`会将实例重新部署到集群中的其他`Node`上。

![img](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112171154439.svg)





可以通过`kubectl get deployments`查看`Deployment`

## 2.3 Node

`Node`可以是虚拟机也可以是物理主机，它依赖于集群。`Node`都被`Control Plane`所管理，每个`Node`都能够有多个`Pod`，`Control Plane`通过`Node`自动地处理调度`Pod`。

每个运行的`Node`至少包含以下组件：

- `Kubelet`：它是负责`Control Plane`和`Node`之间通信的进程；它管理`Pod`和容器在机器上运行
- 容器运行时环境：如Docker。它负责从仓库拉取`container image`，解压缩`container`并运行应用

![img](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112171507634.svg)

## 2.4 Pod

`Pod`相当于**逻辑主机**的概念，运行在`Node`之上，负责托管应用实例，它包括了一个或者多个应用程序容器，这些容器共享主机资源，这些资源包括：

- 存储
- 网络，Pod中的容器共享IP地址和端口
- 每个容器的运行信息

在创建`Deployment`的时候，K8S就会创建`Pod`来管理我们的应用实例。

![img](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112171510428.svg)

`Pod`的声明周期转瞬即逝，当所依赖的`Node`挂掉之后，运行在它之上的`Pod`也会消亡，但系统会根据你的副本集设置重新创建新的`Pod`来让集群回归到目标状态。这也就是说这些Pod都是可以替换的。

## 2.5 Service

尽管每个`Pod`都有唯一的IP地址，但是这些IP地址不会暴露在集群外部，所以K8S抽象出了`Service`的概念

K8S中的`Service`是一个抽象的概念，它定义了一组`Pod`的逻辑集，并为这些`Pod`提供外部流量暴露、负载均衡和服务的发现。

![img](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112171546142.svg)



`Service`通过`Label`和`selecor`(他们是K8S中对进行进行逻辑操作的一种分组原语)来匹配一组`Pod`。

`Label`是附加在对象（如`Pod`）上的键值对，每个键对于给定的对象必须是唯一的。他有如下的使用场景：

- 用于标记开发、测试、生产的对象
- 标记版本
- 将对象进行分类

`Label`创建并附加到对象上之后可以随时进行修改

![img](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112171546653.svg)

# 三.Hello K8S

## 3.1 创建集群

- 查看集群的信息

  ```shell
  kubectl cluster-info
  ```

  ![image-20211217162409779](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112171624837.png)

- 查看集群中的Node信息

  ```shell
  kubectl get nodes
  ```

  ![](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112171624425.png)

  可以发现Minikube创建了一个单节点的简单集群

## 3.2 部署应用

### 3.2.1 制作程序镜像

- 应用程序的代码

  `main.go`

  ```go
  package main
  
  import (
  	"fmt"
  	"log"
  	"net/http"
  	"os"
  )
  
  func helloK8S(w http.ResponseWriter, r *http.Request) {
  	host, _ := os.Hostname()
    _, err := fmt.Fprintf(w, "Hello K8S-V1! I'm running on %s\n", host)
  	if err != nil {
  		return
  	}
  }
  
  func main() {
  	http.HandleFunc("/", helloK8S)
  	err := http.ListenAndServe(":8080", nil)
  	if err != nil {
  		log.Fatal("ListenAndServe: ", err)
  	}
  }
  ```

- 编写`Dockerfile`文件并放在`main.go`的同级目录下

  ```dockerfile
  FROM golang:alpine
  RUN mkdir /app
  COPY . /app
  WORKDIR /app
  RUN go build -o main . 
  CMD ["/app/main"]
  ```

- 编写`go.mod`文件并放在`main.go`同级目录下

  ```go
  module hello-k8s
  
  go 1.17
  ```

- 构建镜像

  ```shel
  docker build -t hello-k8s .
  ```

- 验证镜像

  ```shell
  docker run -d -p 8080:8080 --rm --name hello-k8s hello-k8s
  ```

### 3.2.2 推送镜像到DockerHub

- 本地登录docker

  ```shell
  docker login
  ```

- 制作镜像

  镜像命名规范是`注册用户名/镜像名`

  ```shell
  docker build -t mrwater233/hello-k8s:v1 .
  ```

- 推送镜像

  ```shell
  docker push mrwater233/hello-k8s:v1
  ```

- 查看推送上去的镜像

  https://hub.docker.com/

### 3.2.3 K8S部署应用

- 编写部署清单`deployment.yaml`

  [K8S YAML文件详解](https://blog.csdn.net/BigData_Mining/article/details/88535356)

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: hello-k8s
  spec: # Deployment的规格说明
    replicas: 1
    selector:
      matchLabels:
        app: hello-k8s
    template: # Pod的模版
      metadata:
        labels:
          app: hello-k8s
      spec: # Pod的规格说明
        containers:
          - name: hello-k8s-container
            image: mrwater233/hello-k8s:v1
            ports:
              - containerPort: 8080
  ```

- 部署应用

  ```shell
  kubectl create -f deployment.yaml
  
  kubectl get deployments
  
  kubectl get pods
  ```

- 暴露应用

  应用部署完后还不能从外部直接访问，需要把刚才`Deployment`对象运行的应用程序作为`Kubernetes`的一个`Service`对外暴露。

  ```shell
  kubectl expose deployment/hello-k8s --type=NodePort --port=8000 --target-port=8080 --name=hello-k8s-svc
  ```

  - `--type=NodePort`：指定类型为集群外部访问
  - `--port`：指定集群内访问的端口
  - `--target-port`：指定容器内跑服务的端口

  这里附上NodePost这种服务方式的引导图

  ![2.png](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112181019564.png)

  

  接下去获取nodeport暴露的端口

  ```shell
  kubectl get svc
  ```

  ![image-20211218102155074](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112181021131.png)

  

  获取minikube的ip地址

  ```shell
  minikube ip
  ```

  ![image-20211218102228121](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112181022204.png)

  

  访问

  ```shel
  curl ${minikube ip}:${service port}
  ```

  ![image-20211218102341105](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202112181023191.png)

- 配置nginx

  安装：

  ```shell
  sudo apt install nginx
  ```

  配置`vim /etc/nginx/conf.d/minikube.conf`

  ```shell
  server{
    listen 80;
    server_name 127.0.0.1; # 公网ip
    index  index.php index.html index.htm;
  
    location / {
      proxy_pass  http://127.0.0.1:8080; # minikube ip:service port
      proxy_set_header Host $proxy_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  }
  ```

## 3.3 伸缩容

伸缩容是通过改变`Deployment`的副本数量来实现的

[官网图例例子 ](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/scale/scale-intro/)

- 查看我们当前的`Deployments`

  ```shell
  kubectl get deployments
  ```

  ```shell
  NAME        READY   UP-TO-DATE   AVAILABLE   AGE
  hello-k8s   1/1     1            1           16h
  ```

  - NAME：列出了我们集群中`Deployments`的名字
  - READY：展示了目前`CURRENT/DESIRED`（现实/理想）副本数
  - UP-TO-DATE：展示了有多少副本已经更新到我们想要达到的状态（完成更新）
  - AGE：启动的时间

- 查看副本

  ```shell
  kubectl get rs
  ```

  ```shell
  NAME                   DESIRED   CURRENT   READY   AGE
  hello-k8s-7dd5648ff9   1         1         1       16h
  ```

  - 这里副本的NAME始终按照`[DEPLOYMENT-NAME]-[RANDOM-STRING]`的格式展示
  - DESIRED：你定义的副本数
  - CURRENT：当前实际运行的副本数

- 扩容

  ```shell
  kubectl scale deployments/hello-k8s --replicas=4
  ```

  查看状态

  ```shell
  kubectl get deployments
  
  kubectl get pods -o wide
  ```

- 负载均衡

  进行扩容之后再去访问K8S就会帮我们进行负载均衡

- 缩容

  ```shell
  kubectl scale deployments/hello-k8s --replicas=1
  ```

## 3.4 滚动更新

K8S的采用的更新方式就是滚动更新，[滚动更新图例](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/update/update-intro/)

有三种不同的更新方法[三种不同的更新方法](https://www.jianshu.com/p/8c2eafc46104)，这里采用kubectl的方式

- 进行滚动更新

  ```shell
  kubectl set image deployments/hello-k8s hello-k8s-container=mrwater233/hello-k8s:v2
  ```

  查看`Pod`状态

  ```shell
  kubectl get pods
  ```

- 确认更新

  ```shell
  kubectl rollout status deployments/hello-k8s
  ```

- 回滚

  ```shell
  kubectl rollout undo deployments/hello-k8s
  ```

  

