# 一.fork()

作用：创建一个子进程

`fork()`函数返回一个`pid_t`类型的返回值，假设该变量为`pid`

- $$pid < 0$$：子进程创建失败
- $$pid == 0$$：正在执行子进程
- $$pid > 0$$：正在执行父进程

# 二.execvp()

作用：在程序中运行另一个程序

参数：

1. 需要运行的文件位置
2. 传入参数

`test.c`

将该文件编译为`test.out`

```c
#include<stdio.h>

int main(int argc, char *argv[]) {
    printf("%d个参数:%s、 %s\n", argc, argv[0],argv[1]);
    return 0;
}
```

`parent.c`

```c
#include<stdio.h>
#include<unistd.h>

int main() {
    pid_t pid = fork();
    if(pid < 0) {
        printf("failed!\n");
    }
    if (pid == 0) {
        // 用0作为最后一个参数结束
        char* argv[] = {"test.out", "Hello",0};
        execvp("/root/test.out", argv);
        // 结束子进程
        exit(0);
    } else {
        printf("parent: hello!\n");
    }
    return 0;
}
```

结果：

![image-20201123202513112](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201123202513.png)

# 三.wait()

作用：等待子进程退出