1. 下载Nodejs编译好的压缩包

   ```shell
   wget https://nodejs.org/dist/v12.19.0/node-v12.19.0-linux-x64.tar.xz
   ```

2. 解压缩文件

   ```shell
   tar xvf node-v12.19.0-linux-x64.tar.xz
   ```

3. 移动Node文件位置

   ```shell
   mv ./node-v12.19.0-linux-x64 /usr/local/nodejs
   ```

4. 建立软连接，使node和npm命令能够全局使用

   ```shell
   ln -s /usr/local/nodejs/bin/node /usr/local/bin
   ln -s /usr/local/nodejs/bin/npm /usr/local/bin
   ```

5. 配置环境变量（Shell脚本执行命令时使用）

   1. 修改`/etc/profile`文件

   2. 在末尾添加

      ```shell
      export NODE_HOME=/usr/local/nodejs  //Node所在路径
      export PATH=$NODE_HOME/bin:$PATH
      ```

   3. 更新环境变量

      ```shell
      source /etc/profile
      ```

6. 查看是否安装成功

   ```shell
   node -v
   npm -v
   ```

   

