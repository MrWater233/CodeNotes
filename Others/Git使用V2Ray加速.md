只对`github`进行加速：

```shell
# socks5协议，10808端口修改成自己的本地代理端口
git config --global http.https://github.com.proxy socks5://127.0.0.1:10808
git config --global https.https://github.com.proxy socks5://127.0.0.1:10808
```



其他命令：

```shell
# 查看所有配置
git config -l
# reset 代理设置
git config --global --unset http.proxy
git config --global --unset https.proxy
```

