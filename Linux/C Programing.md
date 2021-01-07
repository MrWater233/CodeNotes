# 一.类Unix系统下C语言编程常用头文件

| 头文件        | 说明                           |
| ------------- | ------------------------------ |
| `stdio.h`     | 标准输入输出                   |
| `stdlib.h`    | 标准库头文件                   |
| `string.h`    | 字符串处理                     |
| `unistd.h`    | 类unix系统中的系统调用         |
| `sys/types.h` | 基本系统数据类型、多个派生类型 |
| `sys/stat.h`  | 方便获取文件属性               |
| `fcntl.h`     | 定义文件信息控制               |
| `sys/time.h`  | 时间、日期相关函数             |

# 二.进程

# 三.线程

## 3.1 线程的基本使用

`test.c`

```c
#include "stdio.h"
#include "stdlib.h"
#include "pthread.h"
#include "unistd.h"

void fun(void) {
	for(int i = 0;i < 10;i++) {
		printf("This is a pthread\n");
	}
}

int main() {
	pthread_t id;
	int i,ret;
	// 创建一个线程
	// 1.线程标识符 2.设置线程属性 3.线程运行函数的起始地址 4.运行函数的参数
	ret = pthread_create(&id, NULL, (void*)fun, NULL);
	// ret=0标识线程创建失败
	if (ret != 0) {
		printf("Create pthread failed\n");
		exit(-1);
	}
	for(int i = 0;i < 10;i++) {
		printf("This is a main process\n");
	}
	// 用来等待一个线程的结束，线程间的同步操作
	// 1.线程标识符 2.用户定义的指针，用来存放被等待线程的返回值
	pthread_join(id, NULL);
	return 0;
}
```

编译运行：

```shell
gcc -o test.out test.c -lpthread
./test.out
```

结果：

```c
This is a main process
This is a pthread
This is a pthread
This is a pthread
This is a pthread
This is a pthread
This is a pthread
This is a pthread
This is a pthread
This is a pthread
This is a pthread
This is a main process
This is a main process
This is a main process
This is a main process
This is a main process
This is a main process
This is a main process
This is a main process
This is a main process
```

# 四.文件

## 4.1 打开文件

`int open(const char* filename, int flags)`

`flags`的取值：

| flags        | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| `O_RDONLY`   | 只读模式                                                     |
| `O_WRONLY`   | 只写模式                                                     |
| `O_RDWR`     | 读写模式                                                     |
| `O_APPEND`   | 每次写操作都写入文件的末尾                                   |
| `O_CREAT`    | 如果指定文件不存在，则创建这个文件                           |
| `O_EXCL`     | 如果要创建的文件已存在，则返回-1，并且修改errno的值          |
| `O_TRUNC`    | 如果文件存在，并且以只写/读写方式打开，则清空文件全部内容(即将其长度截短为0) |
| `O_NOCTTY`   | 如果路径名指向终端设备，不要把这个设备用作控制终端。         |
| `O_NONBLOCK` | 如果路径名指向FIFO/块文件/字符文件，则把文件的打开和后继I/O  |

返回值：文件标识符

## 4.2 读取文件

`ssize_t read(int fd, void *buf, size_t count)`

参数：

1. fd：需要读取文件的文件标识符
2. buf：写入的缓冲区
3. count：每次读取的字节数

返回值：成功返回读取的字节数，出错返回-1并设置errno，如果在调read之前已到达文件末尾，则这次read返回0。

## 4.3 写入文件

`ssize_t write(int fd, const void *buf, size_t nbyte);`

参数：

1. fd：写入文件的文件标识符
2. buf：读取的缓冲区
3. nbyte：每次写入的字节数

返回值：写入文档的字节数（成功）；-1（出错）

## 4.4 读取并写入文件

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#define count 1

int main(int argc, char *argv[]) // argc存储的是运行时输入的参数个数。argv[]存储输入字符串（其中argv[0]存储"./copy")
{
	//输入的两个文件名（xxx yyy）加上运行指令（./copy）一共三个参数
	if (argc != 3)
	{
		printf("输入参数错误\n");
		exit(-1);
	}
	char *p, ch;
	p = (char *)malloc(sizeof(char) * count);
	//只读模式打开文件，从文件头开始读。
	int fp1 = open(argv[1], O_RDONLY);
	//打开失败
	if (fp1 == -1)
	{
		printf("打开文件1失败\n");
		exit(-1);
	}
	//权限标志：S_IRUSR 文件拥有者可读；S_IWUSR 文件拥有者可写
	//O_WRONLY 读写模式打开文件；O_CREAT 按照参数mode中给出的访问模式创建文件
	int fp2 = open(argv[2], O_WRONLY | O_CREAT | O_APPEND, S_IRUSR | S_IWUSR);
	//打开失败
	if (fp2 == -1)
	{
		printf("打开文件2失败\n");
		exit(-1);
	}
	//从文件1中每次读count个字符，写入文件2中 ；read（）和write（）运行成功返回读写字符个数
	while (read(fp1, p, count) == 1)
		write(fp2, p, count);
	//释放指针变量
	free(p);
	return 0;
}
```



