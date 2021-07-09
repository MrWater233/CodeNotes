1. 禁用nouveau

   1. 终端执行以下命令

      ```shell
      sudo vi /etc/modprobe.d/blacklist.conf
      ```

   2. 以下内容复制到文件中

      ```shell
      blacklist nouveau   
      blacklist lbm-nouveau   
      options nouveau modeset=0 
      alias nouveau off   
      alias lbm-nouveau off
      ```

      保存并退出

   3. 执行以下命令让配置文件生效

      ```shell
      sudo update-initramfs -u
      ```

   4. 重启

      ```shell
      sudo reboot
      ```

2. 安装英伟达驱动

   ```shell
   sudo apt install nvidia-driver
   ```

3. 安装nvidia-smi

   ```shell
   sudo apt update -y && sudo apt install nvidia-smi -y
   ```

   如果遇到依赖报错就安装相应依赖

   ```shell
   sudo apt install plymouth-themes console-setup
   ```

4. 安装集显和独显的切换插件

   https://github.com/zty199/dde-dock-switch_graphics_card

5. 重启