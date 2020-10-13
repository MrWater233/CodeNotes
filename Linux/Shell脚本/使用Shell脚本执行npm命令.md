1. [配置Node环境变量](https://github.com/MrWater233/CodeNotes/blob/master/FrontEnd/Node/Linux%E6%90%AD%E5%BB%BAnode%E7%8E%AF%E5%A2%83.md)

2. 假设npm项目在`/root/script`中

3. 在`package.json`中编写npm命令。假设编写的命令为`npm run script`

4. 创建shell脚本

   ```shell
   #!/bin/bash
   
   cd /root/script
   
   source /etc/profile
   
   npm run script
   ```

   

