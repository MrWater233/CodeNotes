1. 下载JDK

   [Oracle官网](https://www.oracle.com/java/technologies/javase-jdk8-downloads.html)

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200405175817.png)

2. 进入`usr/local/java`目录，使用`rz`命令上传JDK

3. 解压JDK

   ```shell
   tar -zxvf jdk-8u241-linux-x64.tar.gz
   ```

4. 配置环境变量

   ```shell
   vi /etc/profile
   ```

   在最后添加

   ```js
   #Java Env
   export JAVA_HOME=/usr/local/java/jdk1.8.0_241
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   export PATH=$PATH:$JAVA_HOME/bin
   ```

5. 刷新配置

   ```shell
   source /etc/profile
   ```

6. 检查JDK是否安装成功

   ```shell
   java -version
   ```

   

