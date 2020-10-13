1. 创建`.vimrc`配置文件

   ```shell
   vim ~/.vimrc
   ```

2. 修改配置文件

   ```shell
   set number
   set tabstop=4
   set softtabstop=4
   set shiftwidth=4
   set noexpandtab
   set autoindent
   ```

   配置说明：

   set number：表示打开文件自动显示行号

   set tabstop=4：表示一个Tab键显示出来多少个空格的长度，默认是8，这里设置为4

   set softtabstop=4：表示在编辑模式下按退格键时候退回缩进的长度，设置为4

   set shiftwidth=4：表示每一级缩进的长度，一般设置成和softtabstop长度一样

   set noexpandtab：当设置成expantab时表示缩进用空格来表示，noexpandtab则用制表符表示一个缩进

   set autoindent：表示自动缩进

3. 保存退出

