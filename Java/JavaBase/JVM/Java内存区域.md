# 一.运行时数据区域

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200828184643.png)

## 1.1 程序计数器

> 程序计数器是一块较小的内存空间，每个线程都有一个独立的程序计数器

作用：**字节码解释器**通过改变**计数器的值**来选取下一跳需要执行的字节码指令。分支、循环、跳转、异常处理、线程恢复等功能都依靠计数器来完成

## 1.2 Java虚拟机栈

> 它用来描述Java方法执行的线程内存模型。虚拟机栈是线程私有的，他的生命周期与线程相同。

### 1.2.1 栈帧

> 每个方法执行的时候Java虚拟机都会同步创建一个栈帧。栈帧都存储虚拟机栈中

栈帧存储的数据：

- 局部变量表

  - 存储编译期可知的基本数据类型（`boolean byte char short int float long double`）
  - 存储对象引用
  - 存储`returnAddress`类型（指向了一条字节码指令的地址）

  这些数据类型再变量表中以**局部变量槽（Slot）**来表示，其中`long double`会占用两个变量槽，其他占用一个

- 操作数栈

- 动态连接

- 方法出口

每一个方法被调用到执行完毕的过程就对应一个栈帧再虚拟机栈中从出栈到入栈的过程

## 1.3 本地方法栈

> 为虚拟机提供使用本地方法（其他语言的的接口）的服务

## 1.4 Java堆

> 所有线程共享的内存区域，用于存放对象实例。

Java堆可以处于物理上不连续的内存空间，但是逻辑上应该是独立的

## 1.5 方法区

> 各个线程共享的内存区域

作用：存储被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存

### 1.5.1 运行时常量池

> 它是方法区的一部分

作用：存放编译器生成的各种字面量与符号引用

## 1.6 直接内存

> 在堆外直接分配的内存，如果使用了直接内存能够在一些场景下能够提高代码性能。

# 二.HotSpot虚拟机对象

> 通过HotSpot虚拟机剖析虚拟机对象的创建

## 2.1 对象的创建

1. 遇到一条字节码`new`指令后，检查这个指令的参数是否能在常量池中定位到一个符号引用（类的基本信息），并检查这个符号是否已经被加载、解析，初始化过。如果没有执行相应的**类加载过程**。

2. 为新生对象分配内存，所需要的内存大小在类加载完成后就可以确定。

   - 分配内存方法：
     - 当堆中内存是绝对规整的时候。通过一个指针来划分可用和以用的内存空间。分配内存只需要让指针计数器向可用内存空间移动对象大小的距离即可。这种分配方式称为**指针碰撞**
     - 当堆中内存大小不规整时，虚拟机需要维护一个列表，记录哪些内存块可用，分配给对象一个足够大的空间并更新列表记录。这种分配方式称为**空闲列表**

   - 但是修改指针位置在并发情况下线程并不安全（A分配的途中指针没来得及改B使用原来的指针分配内存）。

     解决方法：

     - 分配内存空间的动作进行同步处理（CAS+失败重试）
     - 将内存分配动作按照线程划分在不同空间上进行，即每个线程在Java堆中预先分配一小块内存，称为**本地线程分配缓冲（TLAB）**
   
3. 将对象分配的内存空间初始化为0值。如果使用TLAB可以提前到TLAB分配时进行。

   这步操作保证了对象实例字段在Java代码中不赋初值就能够使用，程序能够访问这些字段数据类型对应的零值

4. 对对象进行必要的设置，如对象是哪个类的实例、如何才能找到对象的元数据信息、对象的哈希码、对象的GC分代年龄信息。这些信息都存储在**对象头**中

5. 执行对象的构造函数（即Class文件中`<init>()`方法的执行）

## 2.2 对象的内存布局

> HotSpot虚拟机中对象在堆中分布可以划分成三部分：对象头、实例数据、对齐填充

### 2.2.1 对象头

对象头部分包括两类信息：

- 存储对象自身运行时数据：哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有锁、偏向线程ID、偏向时间戳。

  这些数据在32位和64位虚拟机中分别存储32bit和64bit

- 类型指针：指向他的类型元数据的指针。Java通过这个指针来确定该对象是哪个类的实例，但是查找元数据不一定要通过对象本身，还能通过句柄。

- 如果对象是一个Java数组，还需要有一块用于记录数组长度的数据。

### 2.2.2 实例数据

存储对象真正有效的信息，即程序代码中所定义的各种类型的字段内容（父类和自己的）

存储顺序：

- 相同大小的字段分配在一起存放
- 父类定义的变量出现在子类之前

### 2.2.3 对齐填充

对齐填充并不是必然存在的。

HotSpot规定对象的起始地址必须是8字节的整数倍，换句话说就是任何对象的大小都必须是8字节的整数倍。

由于对象头已经被设计成8字节的倍数，所以对齐填充主要用来填充实例数据的。

## 2.3 对象的访问控制

> 程序通过栈上的`reference`来操作堆上的具体对象
>
> 这种引用访问方式有两种：句柄和直接指针

- 句柄

  Java堆中会划分一块内存作为句柄池，`reference`存储的就是句柄的地址，句柄包含了对象和对象所属类的地址

  ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200828194054.png)

- 直接指针

  `reference`直接存储对象地址，对象地址的地址头中存储对象所属类的地址

  ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200828194122.png)

HotSpot采用第二种方式进行对象访问，因为他节省了一次指针定位的开销时间，速度更快