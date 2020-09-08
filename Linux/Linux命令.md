# 一.用户

默认有一个普通用户和一个`root`用户

- 修改某个用户的密码

  ```shell
  sudo passwd [username]
  ```

- 切换用户

  ```shell
  su [username]
  ```

- 添加用户（`root`用户下）

  ```shell
  adduser [username]
  ```

  1. 输入该账号的密码
  2. 输入该用户的基本信息并确认

- 修改当前用户的密码

  ```shell
  passwd
  ```

# 二.目录

- 查看当前目录

  ```shell
  pwd
  ```

- 进入当前用户的主目录

  ```shell
  cd ~
  ```

- 进入上一级目录

  ```shell
  cd ..
  ```

- 进入根目录

  ```shell
  cd /
  ```

# 三.安装

- 更新软件源列表

  ```shell
  sudo apt-get update
  ```

- 更新安装的软件

  会将安装的软件与软件源列表的软件进行对比，如果发现已经安装的版本过低就会进行更新

  ```shell
  sudo apt-get upgrade
  ```

- 安装软件

  ```shell
  sudo apt-get install [软件包名称]
  ```



