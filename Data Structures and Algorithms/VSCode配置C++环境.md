1. 下载vscode插件`C++`、`Code Runner`

2. 下载`MinGW`并解压到对应目录

3. 添加`MinGW`的环境变量

   如`D:\Environment\MinGW-w64\mingw64\bin`

4. 创建项目文件夹并添加到工作区，并在工作区中创建`.vscode`文件夹

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200911205229.png)

5. 在`.vscode`文件夹中创建以下文件

   `c_cpp_properties.json`

   ```json
   {
       "configurations": [
           {
               "name": "Win32",
               "includePath": [
                   "${workspaceFolder}"
               ],
               "defines": [
                   "_DEBUG",
                   "UNICODE",
                   "_UNICODE"
               ],
               "compilerPath": "D:\\Environment\\MinGW-w64\\mingw64\\bin\\g++.exe",
               "cStandard": "c11",
               "cppStandard": "c++17",
               "intelliSenseMode": "gcc-x64"
           }
       ],
       "version": 4
   }
   ```

   `launch.json`

   ```json
   {
       // 使用 IntelliSense 了解相关属性。 
       // 悬停以查看现有属性的描述。
       // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
       "version": "0.2.0",
       "configurations": [
           {
               "name": "(gdb) Launch",
               "type": "cppdbg",
               "request": "launch",
               "program": " ${file}.exe",
               "args": [],
               "stopAtEntry": false,
               "cwd": "${workspaceFolder}",
               "environment": [],
               "externalConsole": true,
               "preLaunchTask": "build",
               "MIMode": "gdb",
               "miDebuggerPath": "D:\\Environment\\MinGW-w64\\mingw64\\bin\\gdb.exe",
               "setupCommands": [
                   {
                       "description": "Enable pretty-printing for gdb",
                       "text": "-enable-pretty-printing",
                       "ignoreFailures": true
                   }
               ]
           }
       ]
   }
   ```

   `tasks.json`

   ```json
   {
       // See https://go.microsoft.com/fwlink/?LinkId=733558
       // for the documentation about the tasks.json format
       "version": "2.0.0",
       "tasks": [
           {
               "label": "build",
               "type": "shell",
               "command": "g++",
   
               "args": [
                   "-g",
                   "${file}",
                   "-o",
                   "${file}.exe"
               ],
           }
       ]
   }
   ```

   <font color=red>将路径换为MinGW具体的位置</font>

6. 解决控制台输出的中文乱码

   1. 打开控制面板

   2. 选择时钟和区域

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200911205026.png)

   3. 选择区域

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200911205111.png)

   4. 选中管理

   5. 更改系统区域设置

   6. 勾选设置全球语言支持选项

      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200911204940.png)
      
   7. 改变大括号习惯
   
      设置中搜索`clang_format`，更改为`Google`的格式化方式即可
   
      ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200915105120.png)

