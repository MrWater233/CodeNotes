# 一.继承Thread类创建线程

- 通过重写父类`Thread`的`run()`方法来定义线程执行的操作

```java
public class Main {
    public static void main(String[] args) {
        new ThreadTest1().start();
        new ThreadTest1().start();
    }
}

class ThreadTest1 extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(getName()+"\ttest1");
        }
    }
}
```

# 二.通过Runnable接口创建线程类

```java
public class Main {
    public static void main(String[] args) {
        Runnable ra = new ThreadTest2();
        new Thread(ra,"线程A").start();
        new Thread(ra,"线程B").start();
    }
}

class ThreadTest2 implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName()+"\ttest1");
        }
    }
}
```

