1. 打开我的电脑，在地址栏输入`%APPDATA%`回车进行跳转

2. 创建`pip`文件夹

3. 在`pip`文件夹内创建`pip.ini`

4. 输入以下内容

   ```ini
   [global]
   timeout = 6000
   index-url = https://mirrors.aliyun.com/pypi/simple/
   trusted-host = mirrors.aliyun.com
   ```



国内pip源地址：

- 阿里云：https://mirrors.aliyun.com/pypi/simple/
- 清华：https://pypi.tuna.tsinghua.edu.cn/simple
- 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
- 华中理工大学：http://pypi.hustunique.com/
- 山东理工大学：http://pypi.sdutlinux.org/
- 豆瓣：http://pypi.douban.com/simple/