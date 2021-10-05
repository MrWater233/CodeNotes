## 安装

- 安装`acme.sh`

  ```shell
  curl https://get.acme.sh | sh -s email=my@example.com
  ```

  ```shell
  source ~/.bashrc
  ```

- 创建API Token（这里以DNSPOD为例）

  https://console.dnspod.cn/account/token/token

  ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/202110051342673.png)

  将API Token加入环境变量中

  ```shell
  vim ~/.bashrc
  ```

  ```shell
  export DP_Id="xxx"
  export DP_Key="xxx"
  ```

  ```shell
  source ~/.bashrc
  ```

- 申请通配符证书（先添加@和*的DNS解析）

  ```shell
  acme.sh --issue --dns dns_dp -d example.com -d *.example.com
  ```

- 将证书安装到相应位置（位置可以根据自己需要可以进行更改）

  ```shell
  mkdir /etc/nginx/ssl
  ```

  ```shell
  acme.sh --installcert -d domain.com -d *.domain.com \
  --key-file /etc/nginx/ssl/key.pem \
  --fullchain-file /etc/nginx/ssl/cert.pem \
  --reloadcmd "service nginx force-reload"
  ```

- 配置Nginx

  ```shell
  server {
      listen       80;
      server_name  test.domain.com;
      return 301 https://$server_name$request_uri;
  }
  
  server {
      listen       443 ssl http2;
      # 域名，多个以空格分开
      server_name  test.domain.com;
      
      # ssl证书地址
      ssl_certificate     /etc/nginx/ssl/cert.pem;  # pem文件的路径
      ssl_certificate_key  /etc/nginx/ssl/key.pem; # key文件的路径
      
      # ssl验证相关配置
      ssl_session_timeout  5m;    #缓存有效期
      ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;    #加密算法
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;    #安全链接可选的加密协议
      ssl_prefer_server_ciphers on;   #使用服务器端的首选算法
  
      location / {
      	proxy_set_header X-Real-IP $remote_addr;
      	proxy_set_header X-Forwarded-For $remote_addr;
      	proxy_set_header Host $host;
      	proxy_pass http://127.0.0.1:8080;
      }
  }
  ```

  重载Nginx：

  ```shell
  nginx -s reload
  ```

## 常用命令

- `acme.sh --list`：列出证书

- `acme.sh remove xxx`：删除证书

- 更新`acme.sh`

  - 升级到最新版 

    ```shell
    acme.sh --upgrade
    ```

  - 开启自动更新

    ```shell
    acme.sh  --upgrade  --auto-upgrade
    ```

  - 关闭自动更新

    ```shell
    acme.sh --upgrade  --auto-upgrade  0
    ```

## 参考

- https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E
- https://segmentfault.com/a/1190000022044741
- https://segmentfault.com/a/1190000022044741