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

# 五.Shell替换

## 5.1 转义替换

```shell
#!/bin/bash
a=10
echo -e "Value of a is $a \n"
```

运行结果：

```shell
value of a is 10

```

这里的`-e`就是对转义字符进行替换，如果不使用`-e`将会原样输出：

```shell
value of a is 10\n
```



转义字符表：

| 转义字符 | 含义                             |
| -------- | -------------------------------- |
| \\\      | 反斜杠                           |
| \a       | 警报，响铃                       |
| \b       | 退格（删除键）                   |
| \f       | 换页(FF)，将当前位置移到下页开头 |
| \n       | 换行                             |
| \r       | 回车                             |
| \t       | 水平制表符（tab键）              |
| \v       | 垂直制表符                       |

## 5.2 命令替换

> 命令替换指Shell可以先执行命令，将结果暂时保存在变量中，在适当的时候进行输出

语法格式：

```shell
# 用反引号包裹命令
`command`
```

例如：

```shell
#!/bin/bash

DATE=`date`
echo "$DATE"
```

运行结果：

```shell
Thu Sep 10 12:02:33 UTC 2020
```

## 5.3 变量替换

> 变量替换可以根据变量的状态（是否为空，是否被定义等）来改变它的值

变量替换形式：

| 形式            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| ${var}          | 变量本来的值                                                 |
| ${var:-word}    | 如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值。 |
| ${var:=word}    | 如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word。 |
| ${var:?message} | 如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 var 是否可以被正常赋值。 若此替换出现在Shell脚本中，那么脚本将停止运行。 |
| ${var:+word}    | 如果变量 var 被定义，那么返回 word，但不改变 var 的值。      |

# 六.Shell运算符

## 6.1 算数运算符

> 原生Bash不支持简单的数学运算，但是可以通过其他命令来实现，如`expr`

例子：

```shell
#!/bin/sh

a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
   echo "a is equal to b"
fi

if [ $a != $b ]
then
   echo "a is not equal to b"
fi
```

输出：

```shell
a + b : 30
a - b : -10
a * b : 200
b / a : 2
b % a : 0
a is not equal to b
```

注意：

- 运算符与操作数之间有空格，如`2+2`是错误的，正确的应为`2 + 2`
- 条件表达式要放在方括号之间，并且要有空格，例如 `[$a==$b]` 是错误的，必须写成 `[ $a == $b ]`。



常用的运算符：

| 运算符 | 说明                                          | 举例                          |
| ------ | --------------------------------------------- | ----------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 30。    |
| -      | 减法                                          | `expr $a - $b` 结果为 10。    |
| *      | 乘法                                          | `expr $a \* $b` 结果为  200。 |
| /      | 除法                                          | `expr $b / $a` 结果为 2。     |
| %      | 取余                                          | `expr $b % $a` 结果为 0。     |
| =      | 赋值                                          | a=$b 将把变量 b 的值赋给 a。  |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | [ $a == $b ] 返回 false。     |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | [ $a != $b ] 返回 true。      |

## 6.2 关系运算符

> 关系运算符只支持数字，不支持字符串，除非字符串的值是数字

例如：

```shell
#!/bin/bash

a=10
b=20

if [ $a -eq $b ]
then
    echo "a is equal to b"
else
    echo "a is not equal to b"
fi
```

输出：

```shell
a is not equal to b
```



关系运算符列表

| 运算符 | 说明                                                  | 举例                       |
| ------ | ----------------------------------------------------- | -------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 true。  |
| -ne    | 检测两个数是否相等，不相等返回 true。                 | [ $a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大等于右边的，如果是，则返回 true。   | [ $a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |

## 6.3 布尔运算符

例子：

```shell
#!/bin/bash

a=10
b=20

if [ $a != $b ]
then
   echo "$a != $b : a is not equal to b"
else
   echo "$a != $b: a is equal to b"
fi

if [ $a -lt 100 -a $b -gt 15 ]
then
    echo "$a -lt 100 -a $b -gt 15 : returns true"
else
    echo "$a -lt 100 -a $b -gt 15 : returns false"
fi

if [ $a -lt 100 -o $b -gt 100 ]
then
    echo "$a -lt 100 -o $b -gt 100 : returns true"
else
    echo "$a -lt 100 -o $b -gt 100 : returns false"
fi
```

结果：

```shell
10 != 20 : a is not equal to b
10 -lt 100 -a 20 -gt 15 : returns true
10 -lt 100 -o 20 -gt 100 : returns true
```



布尔运算符表

| 运算符 | 说明                                                | 举例                                     |
| ------ | --------------------------------------------------- | ---------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [ $a -lt 20 -o $b -gt 100 ] 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |

## 6.4 字符串运算符

字符串运算符表：

设`a="abc" b="bcd"`

| 运算符 | 说明                                      | 举例                     |
| ------ | ----------------------------------------- | ------------------------ |
| =      | 检测两个字符串是否相等，相等返回 true。   | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否相等，不相等返回 true。 | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。     | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否为0，不为0返回 true。   | [ -z $a ] 返回 true。    |
|        | 检测字符串是否为空，不为空返回 true。     | [ $a ] 返回 true。       |

## 6.5 文件测试运算符

> 文件测试运算符用来检测Unix文件的各个属性

文件测试运算符表：

设`file="/var/www/tutorialspoint/unix/test.sh"`，它的大小为100字节，具有 `rwx` 权限。

| 操作符  | 说明                                                         | 举例                      |
| ------- | ------------------------------------------------------------ | ------------------------- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -b $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是具名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

# 七.Shell注释

以`#`开头的就是注释

Shell没有多行注释，只能一行一行注释

但是也可以将代码定义为函数，在需要的地方调用，不调用就等于注释起来了