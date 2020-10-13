>- 以下为CentOS7下的操作
>- 系统时区应为UTC，可以通过`date`查看系统时区

# 一.安装Crontab

```shell
yum install vixie-cron crontabs
```

# 二.Cron自启用

```shell
systemctl is-enabled crond.service
```

结果

```shell
enable  #表示已启用自启动
disable #标识未启用自启动
```

- 如果未启用，开启cron自启用

  ```shell
  systemctl enable crond.service
  ```

- 如果已启用，想要关闭自启动

  ```shell
  systemctl disable crond.service
  ```

# 三.Cron服务操作

- 查看cron服务状态

  ```shell
  systemctl status crond.service
  ```

  只有`acrive running`才表示服务已经启动

- 启动Cron服务（无提示）

  ```shell
  systemctl start crond.service
  ```

- 停止Cron服务（无提示）

  ```shell
  systemctl stop crond.service
  ```

- 重启Cron服务（无提示）

  ```shell
  systemctl restart crond.service
  ```

# 四.定时任务操作

- 编辑定时任务

  ```shell
  crontab -e
  ```

  添加定时任务的配置

  如：想要每天`7:00`执行`/root/script/test.sh`脚本并保存日志

  ```shell
  00 07 * * * /root/script/test.sh >> /root/log
  ```

- 查看已编辑的定时任务

  ```shell
  crontab -l
  ```

- 删除已编辑的所有定时任务（不建议使用）

  ```shell
  crontab -r
  ```

# 五.日志和检测

- 查看定时任务执行日志

  ```shell
  tail -f -n 200 /var/log/cron
  ```

  ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201013101822.png)

- 有时候Cron会把报错发送到root的邮箱，可以通过以下命令查看

  ```shell
  cat /var/spool/mail/root
  ```

  



