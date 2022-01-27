1. 下载镜像

   ```shell
   docker pull mongo
   ```

2. 创建运行Mongo容器

   ```shell
   docker run -p 27017:27017 -v /home/mongo:/data/db --name mongo -d mongo --auth
   ```

3. 进入容器内部

   ```shell
   docker exec -it mongo mongo admin
   ```

4. 创建用户并分配权限

   - 建立超级用户

     ```shell
     db.createUser(
       {
         user: "root",
         pwd: "123456",
         roles:
         [ "root"]
       }
     )
     ```

     