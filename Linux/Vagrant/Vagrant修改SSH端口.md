1. 打开vagrant目录下的`Vagrantfile`文件

2. 修改参数为如下

   ```shell
   config.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: "true"
   config.vm.network "forwarded_port", guest: 22, host: 3333
   ```

   - 第一条disabled置为`true`
   - 第一条规则必须有 `id: "ssh"` ，否则会报错
   - 第二条的host改为想要的端口

3. 重启虚拟机，使用3333端口进行连接

   ```shell
   ssh vagrant@127.0.0.1 -p 3333 -i D:/Linux/Ubuntu16/.vagrant/machines/default/virtualbox/private_key
   ```

   

