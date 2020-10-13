# 一.基本配置

1. 拉取`nginx`

   ```shell
   docker pull nginx
   ```

2. 启动`nginx`

   ```shell
   docker run --name nginx \
           -p 443:443\
           -p 80:80 \
           -v /home/docker/nginx/data:/usr/share/nginx/html:rw\
           -v /home/docker/nginx/config/nginx.conf:/etc/nginx/nginx.conf:rw\
           -v /home/docker/nginx/config/conf.d/default.conf:/etc/nginx/conf.d/default.conf:rw\
           -v /home/docker/nginx/logs:/var/log/nginx/:rw\
           -v /home/docker/nginx/ssl:/ssl/:rw\
           -d nginx
   ```

   第一次可能会报错，提示需要文件而不是文件夹，输入以下命令将`nginx.conf`和`default.conf`替换为文件

   ```shell
   docker rm nginx
   
   rm -rf /home/docker/nginx/config/nginx.conf/ /home/docker/nginx/config/conf.d/default.conf/
   
   touch /home/docker/nginx/config/nginx.conf /home/docker/nginx/config/conf.d/default.conf
   ```

3. 编辑`nginx.conf`

   `vim /home/docker/nginx/config/nginx.conf`

   ```shell
   #运行nginx的用户
   user  nginx;
   #启动进程设置成和CPU数量相等
   worker_processes  1;
    
   #全局错误日志及PID文件的位置
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;
    
   #工作模式及连接数上限
   events {
       #单个后台work进程最大并发数设置为1024
       worker_connections  1024;
   }
    
    
   http {
       #设定mime类型
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;
    
       #设定日志格式
       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
    
       access_log  /var/log/nginx/access.log  main;
    
       sendfile        on;
       #tcp_nopush     on;
    
       #设置连接超时的事件
       keepalive_timeout  65;
    
       #开启GZIP压缩
       #gzip  on;
    
       include /etc/nginx/conf.d/*.conf;
   }
   ```

4. 编辑`default.conf`修改其中的域名

   `vim /home/docker/nginx/config/conf.d/default.conf`

   ```shell
   server {
       listen    80;       #侦听80端口，如果强制所有的访问都必须是HTTPs的，这行需要注销掉
       server_name  *.mrwater.top mrwater.top;             #域名
    
       charset utf-8;
       #access_log  /var/log/nginx/host.access.log  main;
    
       # 定义首页索引目录和名称
       location / {
           root   /usr/share/nginx/html;
           index  index.html index.htm;
       }
    
       # 定义错误提示页面
       #error_page  404              /404.html;
    
       # 重定向错误页面到 /50x.html
       error_page   500 502 503 504  /50x.html;
       location = /50x.html {
           root   /usr/share/nginx/html;
       }
   }
   ```

5. 在`/home/docker/nginx/data`目录下创建`index.html`文件

6. 可以通过http访问`index.html`

# 二.配置证书

1. 申请CSR

   1. 下载[KeyManager](https://keymanager.org)，并配置密码

   2. 访问https://freessl.cn/

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013224526.png)

   3. 输入邮箱创建证书

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013224546.png)

   4. 等待KeyManager生成完成

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013223223.png)

2. DNS验证

   1. 查看站点提供的DNS验证

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013222719.png)

   2. 将DNS到阿里云域名控制台进行解析配置

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013222732.png)

3. 导出证书

   1. ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013223653.png)

   2. ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013224227.png)

   3. ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013224452.png)

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013224654.png)

   4. 将导出的`crt`和`key`文件复制到`/home/docker/nginx/ssl`目录下

   5. 重新编辑`default.conf`文件

      ```shell
vim /home/docker/nginx/config/conf.d/default.conf
      ```
      
      ```shell
      # 配置443端口的https
      server {
          #listen    80;       #侦听80端口，如果强制所有的访问都必须是HTTPs的，这行需要注销掉
          listen    443 ssl;
          server_name  *.mrwater.top mrwater.top;             #域名
       
          # 增加ssl
          #ssl on;        #如果强制HTTPs访问，这行要打开
          ssl_certificate /ssl/mrwater.top_chain.crt;
          ssl_certificate_key /ssl/mrwater.top_key.key;
          ssl_session_timeout 5m;
          ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
          ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
          ssl_prefer_server_ciphers on;
       
          # 定义首页索引目录和名称
          location / {
              root   /usr/share/nginx/html;
              index  index.html index.htm;
          }
       
          # 重定向错误页面到 /50x.html
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   /usr/share/nginx/html;
          }
      }
      
      # 配置http跳转到https
      server {
          listen 80;
          # 这里可以不具体指定
          server_name  _;
          # 重写地址
          rewrite ^(.*)$ https://$host$1 permanent; 
}
      ```
      
      

