1. 配置Node环境变量（修改`/etc/profile`文件）

2. 假设npm项目在`/root/script`中

3. 在`package.json`中编写npm命令。假设编写的命令为`npm run script`

4. 创建shell脚本

   ```shell
   #!/bin/bash
   
   cd /root/script
   
   source /etc/profile
   
   npm run script
   ```

   

