- Java大部分情况下是通过new关键词新建一个对象，跟C++类似，也是将标识符当作一个指针，指向堆中的实际空间。如果直接像C++一样在栈中给对象分配空间，是不行的，没有实际空间。

- Java的new关键词会给类中变量赋初始值（类似C++中的全局变量），而C++不会。

- Java有构造函数，但他没有析构函数。

- 可以将多个类定义在一个.java文件中，但只能有一个类是public。

- this指针可以代替类的构造函数。

- Java只能单根继承，C++可以多重继承。

- 不写extends，默认继承java.lang.Object类，所以Java的类存在一些默认方法。

- super()是指向父类的应用，类似于this指针。如果子类没有调用父类构造函数，在子类构造函数中会默认调用super()，即调用父类默认构造函数。

- Java抽象类需要关键字abstract，他没有办法被new，他的子类必须实现**父亲们的所有**抽象方法，不然也要变成抽象类。

- 如果一个类中都为抽象函数，那么这个类定义为接口(interface|<font color=red>没有class关键字</font>)，一个类可以实现多个接口(implement)，接口可以继承多个接口，继承和实现不冲突，接口的方法默认public，可以不写。

- 子类可以进行类型转换成父类，父类在一定条件下也可以转换为子类（父类本身由子类得来）。

- Java中多态不用virtual关键字。不用像C++实现多态用基类指针或者是用函数中的基类引用调用，直接用基类名字就可以实现多态。

- 静态变量和方法可以直接通过类名调用，静态方法不能引用非静态方法，不能使用非静态变量。

- 静态方法和变量不能被继承。

- 类中方法执行顺序：~~static代码块（new之前，只执行一次） > 匿名代码块（new之后） >~~构造函数（new之后）

- 单例模式：一个类有且只有一个对象

  在Java中通过private隐藏构造函数（只用来new内部的那个static对象），用一个静态方法返回一个自身的static对象来实现单例模式。
  
- final类不能被继承

  final方法子类不可修改

  final变量不能更改他的值

  final对象不能修改他的指针，但是可以修改对象里面的值

- import放在package后面

  package用来给java文件打包，一般用路径作为包名申明在开头

  import用来引用包

- Java的bool值默认为false，C++中永远不要让编译器帮你决定。除此之外int的默认值为0，String的默认值为null

- 可以用`instanceof`方法来判断一个对象是不是某个类的子类对象。

- `interface`可以定义静态字段，但是必须为`final`类型，因为是必须的，所以可以省略，编译器会自动加上。

- 字段或者方法没有`private`等修饰词，那么他就是包作用域的字段和方法，修饰词为`default`

- 字符串相比较应该用`equals()`方法，不能用`==`

- 重写父类方法不能降低访问权限，可以提高访问权限。

- String类中`==`比较的是两个变量的引用是否相同，比较要用`.equals()`

- 字符字节输入输出流：

  读取文件创建`FileInputStream(File)`对象，通过`FileInputStream.read([]Byte)`读取字节文件，再通过`new String()`的转换来输出
  
  写入文件创建`FileOutputStream(File,Boolean append)`对象，通过`FileOutputStrea.write([]Byte)`来写入文件

- 字符输入输出流

  `FileReader` `FileWriter` 

- 缓冲流

  `BufferedReader(FileReader)`

  `BufferedWriter(FileWriter)`

  可以用缓冲流读取一行数据`.readLine()`原理是创建了底层的`FileReader`流不断将一行数据写进缓冲区，上层`BufferedReader`流再在缓冲区读取数据，`BufferedWriter`流同理

- 随机流

  `RandomAccessFile`既可以读取数据，也可以写入数据

  `seek(long a)`定位读写位

  `getFilePointer()`获取当前的读写位置

  `.writexxx()`用来写入一个`xxx`型数据

  `.readxxx()`用来读取一个`xxx`型数据

  `.readLine()`需要使用`.getBytes("ios-8859-1")`重新编码再使用`new String`才能正常使用

- 数组流

  - 字节数组流

- 文件锁

  `创建RandomAccessFile()对象->获取文件信道.getChannel()->信道加锁.tryLock()`

- `throws`
  
  放在方法首部，调用异常类

  `throw`

  放在方法中，抛出异常对象