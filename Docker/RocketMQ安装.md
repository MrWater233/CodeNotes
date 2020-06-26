[RocketMQ DockerHub](https://hub.docker.com/r/foxiswho/rocketmq)

[安装教程](https://www.cnblogs.com/xxbiao/p/12119733.html)

1. pull镜像

   ```shell
   docker pull foxiswho/rocketmq:server-4.7.0
   ```

   ```shell
   docker pull foxiswho/rocketmq:broker-4.7.0
   ```

   ```shell
   docker pull styletang/rocketmq-console-ng:latest
   ```

   启动nameserver容器

   ```shell
   docker run -d -p 9876:9876 --name rmqnamesrv foxiswho/rocketmq:server-4.7.0
   ```

2. 启动broker容器

   ```shell
   docker run -d -p 10911:10911 -p 10909:10909 --name rmqbroker --link rmqnamesrv:namesrv -e "NAMESRV_ADDR=namesrv:9876" -e "JAVA_OPTS=-Duser.home=/opt" foxiswho/rocketmq:broker-4.7.0
   ```

3. 启动console

   ```shell
   docker run -d --name rmqconsole -p 8180:8080 --link rmqnamesrv:namesrv -e "JAVA_OPTS=-Drocketmq.namesrv.addr=namesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -t styletang/rocketmq-console-ng
   ```

