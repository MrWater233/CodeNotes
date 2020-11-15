1. 添加远程仓库

   ```shell
   git remote add <name> <url>
   ```

   比如

   ```shell
   git remote add origin https://github.com/xhiteam/x-auth.git
   ```

   这里的`origin`只是我们给远程仓库取得名字，用来区分我们多个远程仓库

2. 上传和下拉

   ```shell
   git pull <name> <branch>
   ```

   ```shell
   git push <name> <branch>
   ```

   比如

   ```shell
   git pull origin master
   ```

   ```shell
   git push origin master
   ```

   这里的`origin`就是我们第一步添加的远程仓库名

3. 查看远程仓库信息

   ```shell
   git remote -v
   ```