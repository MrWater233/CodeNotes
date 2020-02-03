从远程仓库分支拉取代码到本地分支进行合并的步骤：

1. 查看远程仓库
   
   ```
   git remote -v
   ```
   
2. 从远程master分支获取最新版本到本地
   
   ```
   git fetch origin master:temp
   ```
   
3. 比较远程分支和本地分支的区别

   ```
   git diff temp
   ```

4. 合并远程分支到本地

   ```
   git merge temp
   ```

   <font color=red>如果merge过程出现冲突无法合并，就进入冲突文件修改文件，再重新commit即可</font>

5. 删除temp分支

   ```
   git branch -d temp
   ```

