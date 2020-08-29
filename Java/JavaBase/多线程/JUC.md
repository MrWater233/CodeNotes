# 一.JUC简介

JUC(Java Util Concurrent)主要由三个包构成：

- `java.util.concurrent`：并发实用程序类
- `java.util.concurrent.atomic`：小型工具包，支持变量上的线程安全编程
- `java.util.concurrent.locks`：Lock工具包

# 二.内存可见性和volatile关键字

## 2.1 Java内存模型

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.png)

- 所有共享变量都储存在主内存中，每个线程都有一个自己的工作内存，工作内存会存储部分共享变量的拷贝
- 线程对共享变量的所有操作都必须从自己的工作内存中读写而不能直接从主内存中读写
- 不同线程之间不能直接访问各自的工作内存，线程之间的变量传递需要通过主内存来完成

## 2.2 内存可见性

> 当线程1对某一变量修改后没有及时写回主内存，而线程2恰好从主内存中读取了该变量，导致线程1对共享变量的修改无法被线程2及时看到

代码示例：

```java
public class TestVolatile {
    public static void main(String[] args) {
        //创建并启动线程
        ThreadDemo threadDemo = new ThreadDemo();
        Thread thread = new Thread(threadDemo);
        thread.start();
        while (true) {
            //当共享变量为true时输出并结束循环
            if (threadDemo.isFlag()) {
                System.out.println("主线程读取到的flag=" + threadDemo.isFlag());
                break;
            }
        }
    }
}

class ThreadDemo implements Runnable {
    //线程共同操作的变量
    public boolean flag = false;

    @Override
    public void run() {
        //延迟1s后对flag进行修改
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        flag = true;
        System.out.println("修改flag为：" + isFlag());
    }

    public boolean isFlag() {
        return flag;
    }
}
```

副线程在延迟1秒后将`flag=true`，由于主线程此时已经读取了`flag=false`，导致主线程无法及时看见其他线程的修改，进而无限循环。

运行结果：

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200828104323.png)

## 2.3 synchronized实现内存可见性

`synchronized`的2条规定

- 线程解锁前，必须将共享变量的最新值刷新到主内存中
- 线程加锁时，将清空工作内存中的共享变量值，以便及时从主内存中读取最新的值

代码改进：

```java
synchronized (threadDemo) {
    if (threadDemo.isFlag()) {
        System.out.println("主线程读取到的flag=" + threadDemo.isFlag());
        break;
    }
}
```

加上锁以后就能让每次循环都去主存中读取数据。

这里是给主线程代码块加锁，没什么影响。但是当这个代码块被多个线程访问的时候，由于加锁会导致线程阻塞效率低下。

## 2.4 volatile关键字

> 特点：
>
> - 修改变量值后立即从工作内存刷新到主存中
> - 每次使用该变量时立即从主存刷新到工作内存中

用法：

```java
public  volatile boolean flag = false;
```



`volatile`和`synchronized`的区别：

- `volatile`不具有互斥性（一个线程持有锁，其他线程进不来）
- `volatile`不具备原子性

# 三.原子性

