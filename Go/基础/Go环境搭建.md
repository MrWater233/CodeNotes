1. 下载

   - Windows：https://golang.google.cn/dl/

   - Mac：`brew install go`

2. 配置开发的工作空间：

   - Windows：

     设置用户级的环境变量

     ```shell
     GOPATH = D:\Code\Go
     PATH = %GOPATH%\bin
     ```

   - Mac：

     ```shell
     go env -w GOPATH=/Users/wjm/Documents/Code/Go
     ```

3. 设置代理并设置go modules模式

   ```shell
   go env -w GOPROXY=https://goproxy.cn,direct
   
   go env -w GO111MODULE=on
   ```

4. 打开VSCode并进入刚才设置的工作空间目录下

   创建目录`src/golang.org/x/`并进入，输入以下命令进行工具的下载

   ```shell
   git clone https://github.com/golang/tools.git
   git clone https://github.com/golang/lint.git
   ```

5. 安装VSCode插件：`Go`、`Code Runner`

6. 随便打开一个go文件，点击右下角弹出的`Install All`或者`Update`

7. 在`./src`下开始你的Go之旅吧

