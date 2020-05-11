1. 创建文件夹`nginx/项目名/`

2. 打包项目，并将打包生成的`dist`文件放在文件夹中

   ```shell
   npm run build
   ```

3. 安装Nginx

   ```shell
   docker pull nginx
   ```

4. 在文件夹中创建文件`default.conf`

   ```yaml
   server {
       listen       80;
       server_name  localhost;
   
       #charset koi8-r;
       access_log  /var/log/nginx/host.access.log  main;
       error_log  /var/log/nginx/error.log  error;
   
       location / {
           root   /usr/share/nginx/html;
           index  index.html index.htm;
           try_files $uri $uri/ /index.html;
       }
   
       #error_page  404              /404.html;
   
       # redirect server error pages to the static page /50x.html
       #
       error_page   500 502 503 504  /50x.html;
       location = /50x.html {
           root   /usr/share/nginx/html;
       }
   }
   ```

   如果vue-router使用的是history模式，一定要加`try_files $uri $uri/ /index.html;`

5. 在文件夹中创建`Dockerfile`

   ```yaml
   # 设置基础镜像
   FROM nginx
   # 定义作者
   MAINTAINER MrWater <xxxxx@qq.com>
   # 将dist文件中的内容复制到 /usr/share/nginx/html/ 这个目录下面
   COPY dist/  /usr/share/nginx/html/
   #用本地的 default.conf 配置来替换nginx镜像里的默认配置
   COPY default.conf /etc/nginx/conf.d/default.conf
   ```

6. 在`Dockerfile`同级目录输入命令构建Vue应用镜像

   ```shell
   docker build -t 镜像名 .
   ```

   - `-t`用来给镜像命名
   - `.`是基于当前目录的`Dockerfile`来构建镜像

7. 启动镜像

   ```shell
   docker run \
   -p 80:80 \
   -d --name 容器名 \
   镜像名/镜像ID
   ```

   

   

