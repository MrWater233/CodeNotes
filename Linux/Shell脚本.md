# 一.查看Shell

Linux可以用的Shell都记录在`/etc/shells`中，它是一个纯文本文件，可以用以下命令查看他的内容

```shell
cat /etc/shells

/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
```

# 二.执行Shell脚本

创建一个脚本，扩展名为`sh`（扩展名不影响脚本的执行，只是为了认识方便）

输入以下代码

```shell
#! /bin/bash
echo "Hello World!"
```

- `#!`只是一个约定标记，告诉系统脚本需要用什么解释器来执行

## 2.1 作为可执行程序

```shell
# 给文件可执行的权限
chmod +x ./test.sh
# 执行脚本
./test.sh
```

注意：路径一定要写成`./test.sh`而不是`test.sh`，后者是在linux的PATH中寻找

## 2.2 作为解释器参数

直接运行解释器，参数就是Shell脚本文件名

```shell
/bin/sh test.sh
```

- 这种方式执行脚本就不需要在脚本第一行指定解释器的信息了

# 三.Shell变量

## 3.1 定义变量

Shell支持以下定义变量的方式：

```shell
var=value
var='value'
var="value"
```

- `var`为变量名，`value`为值。

- 如果`value`不包含任何空白字符（空格，Tab缩进），那么可以不使用引号。如果包含了空白字符，那么需要用引号包裹。
- 单引号和双引号的区别：
  - 单引号：输出时单引号里面有什么就输出什么，及时有变量和命令也会鸳鸯输出
  - 双引号：会先解析变量再进行输出

<font color=red>注意：赋值号周围不能有空格</font>



例子：

```shell
url=https://www.baidu.com
name='小明'
sex="女"
```

## 3.2 使用变量

使用变量只需要在变量之前加入`$`即可，例如：

```shell
name="小明"
echo $name
echo ${name}
```

变量外部的`{}`是可选的，为了帮助解释器识别变量边界，例如：

```shell
hello="hello"
echo "${hello}jack"
```

上面这个例子如果不加`{}`那么解释器就会把`hellojack`识别为一个变量

<font color=red>建议使用变量都加上花括号</font>

## 3.3 修改变量

```shell
name="小明"
# 修改变量
name="小红"
```

## 3.4 将命令结果赋值给变量

Shell支持将命令的执行结果赋值给变量

```shell
# 推荐使用
var=${command}
var=`command`
```

例子：

```shell
# 将log.txt中的内容读出并赋值给变量并输出变量的值
log=${cat log.txt}
echo ${log}
```

## 3.5 只读变量

使用`readonly`命令将变量定义为只读变量，它的值不能被修改。例如：

```shell
name="小明"
readonly name
```

## 3.6 删除变量

使用`unset`命令可以删除变量

```shell
# 删除name变量
unset name
```

- 变量删除以后就不能再使用了，除非重新声明变量
- `unset`命令不能删除只读变量

# 四.Shell特殊变量

`$ `表示当前Shell进程的ID，即pid

```shell
# echo $$
3227
```

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| `$0` | 当前脚本的文件名                                             |
| `$n` | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。 |
| `$#` | 传递给脚本或函数的参数个数。                                 |
| `$*` | 传递给脚本或函数的所有参数。                                 |
| `$@` | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。 |
| `$?` | 上个命令的退出状态，或函数的返回值。                         |
| `$$` | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。 |

## 4.1 命令行参数

运行脚本时传递给脚本的参数称为命令行参数。命令行参数用`$n`表示（`n`为数字，表示第几个参数，从1开始）

如以下脚本：

```shell
#! /bin/bash

# 输出文件名
echo "File Name:$0"
# 输出传入的第一个参数
echo "First Parameter:$1"
# 输出传入的第二个参数
echo "Second Parameter:$2"
# 输出传入的所有参数
echo "All Parameter:$@"
# 输出传入的所有参数
echo "All Parameter:$*"
# 输出传入的参数个数
echo "Total Number of Paramters:$#" 
```

结果：

```shell
# ./test.sh p1 p2

File Name:./test.sh
First Parameter:p1
Second Parameter:p2
All Parameter:p1 p2
All Parameter:p1 p2
Total Number of Paramters:2
```

## 4.2 `$@`和`$*`的区别

- 当参数不被`""`包裹时都是以`"$1" "$2"...`的形式输出
- 当参数被`""`包裹时
  - `$*`会将参数作为一个整体，以`"$1 $2..."`的形式输出
  - `$@`会将参数分开，以`"$1" "$2"...`的形式输出

## 4.3 退出状态

`$?`可以查看上一个命令执行的返回结果

一般情况下，成功会返回0，失败会返回1

```shell
# ./test.sh p1 p2
File Name:./test.sh
First Parameter:p1
Second Parameter:p2
All Parameter:p1 p2
All Parameter:p1 p2
Total Number of Paramters:2

# echo $?
0
```

