# File类

## 构造方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/File%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.png?raw=true)

## 常用方法

### 访问文件信息方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/File%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.png?raw=true)

### 获得文件属性方法

| 方法                  | 作用                           |
| --------------------- | ------------------------------ |
| `long length()`       | 返回指定文件长度               |
| `boolean exists()`    | 测试指定文件是否存在           |
| `long lastModified()` | 返回直嘀咕文件最后被修改的时间 |

### 文件操作方法

| 方法                          | 作用                               |
| ----------------------------- | ---------------------------------- |
| `boolean renameTo(File dest)` | 文件重命名                         |
| `boolean delete()`            | 删除空目录                         |
| `boolean createNewFile()`     | 文件不存在时创建文件对象代表的文件 |

### 对目录操作

| 方法                                      | 作用                                 |
| ----------------------------------------- | ------------------------------------ |
| `boolean mkdir()`                         | 创建指定目录                         |
| `boolean mkdirs()`                        | 创建指定目录，包括所有不存在的父目录 |
| `String []list`                           | 返回目录中所有文件名字符串           |
| `File[] listFiles()`                      | 返回目录中所有文件对象               |
| `File[] listFiles(FilenameFilter filter)` | 返回目录中满足过滤器的文件和目录     |

## 文件过滤器

- 提供了`FileFilter`和`FilenameFilter`两个接口

- 两个接口都有`accept`方法，实现以后用来过滤文件。
- 当方法`accept()`方法返回值为`true`时筛选出符合要求的文件

### FileFilter接口

```java
public interface FileFilter{
    public boolean accept(File pathname);
}
```

### FilenameFilter接口

```java
public interface FilenameFilter{
    public boolean accept(File dir,String name);
}
```

### 两个接口的区别

- `File`的`list()`方法只能接受`FilenameFilter`接口对象作为参数
- `listFiles()`两个接口都可接受作为参数

# 输入输出流

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/IO.jpg?raw=true)

## 字节流

### InputStream

#### 常用方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/InputStream%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.png?raw=true)



#### FileInputStream

> 以字节对象为单位进行读取出数据

- 因为该类以字节为单位读取文件内容，因此文件中有汉字读取输出是乱码

##### 构造方法

1. 以path作为参数
2. 以File作为参数

##### 常用方法

- 实现了父类中的常用方法，但是**不支持**`mark()`和`reset()`
- 当调用该方法进行读取操作的时候，可以调用`available()`方法判断文件时是否还有可以读的字节，再调用`read()`
- 读取操作均抛出`IOException`异常，但是创建对象的时候文件不存在可能抛出`FileNotFoundException异常

###### read()方法

| 方法                                 | 作用                                |
| ------------------------------------ | ----------------------------------- |
| `int read()`                         | 读取一个字节                        |
| `int read(byte []b)`                 | 读取`b.length`个字节到数组b中       |
| `int read(byte []b,int off,int len)` | 读取len个字节存入从off开始的数组b中 |

- 如果`b==null`抛出`NullPointerException`异常
- 如果b数组越界抛出`IndexOutOfBoundException`异常



#### BufferedInputStream

> 提高数据的读能力，带缓冲功能的流类

- 支持`mark()`和`reset()`方法

##### 构造方法

| 方法                                           | 作用                                         |
| ---------------------------------------------- | -------------------------------------------- |
| `BufferedInputStream(InputStream in)`          | 以输入流对象in创建缓冲输入对象               |
| `BufferedInputStream(InputStream in,int size)` | 创建指定缓冲大小为size个字节的缓冲输入流对象 |

##### 常用方法

继承并实现了父类的方法



#### ByteArrayInputStream

> 从缓冲即字节数组中读取数据 

- 继承并实现了`InputStream`类中的方法，支持`reset()`、`mark()`、`skip()`等方法，但是`close()`**无效**。

##### 构造方法

| 方法                                                     | 作用                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| `ByteArrayInputStream(byte []buf)`                       | 常见一个字节数组输入流对象，使用buf作为缓冲区数组            |
| `ByteArrayInputStream(byte []bufmint offset,int length)` | 创建一个字节数组输入流对象，从offset位置开始读取了length个字节数据 |



#### DataInputStream

> 字节数据输入流，可以对基本数据类型进行输入

> 支持`skip()`但是不支持`mark()`和`reset()`

> 可以用`available()`方法判断输入源中有没有数据可以用来读取

##### 构造方法

`DataInputStream(InputStream in)`

以输入流对象`in`来创建输入流对象

##### 常用方法

> `DataInputStream`类继承了`InputStream`类中的方法用于读取字节数据，同时增加了读取基本数据的方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/DataInputStream%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.png?raw=true)



### OutputStream

#### 常用方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/OutputStream%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.png?raw=true)



#### FileOutputStream

##### 构造方法

1. 以path为参数
2. 以File为参数

- 但是存在第二个`boolean append`参数，可以指定是否以追加方式写入

##### 常用方法

- 实现了父类的所有方法吗，增加了`getFD()`、`getChannel()`、`finalize()`三个方法
- 常见异常和`fileInputStream`类似



#### BufferedOutputStream

> 提高数据的写能力，带缓冲功能的流类

##### 构造方法

| 方法                                              | 作用                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| `BufferedOutputStream(OutputStream out)`          | 以字节输出流对象创建缓冲输出流对象                           |
| `BufferedOutputStream(OutputStream out,int size)` | 以字节输出流对象创建指定缓存大小为size个字节的缓冲输出流对象 |

##### 常用方法

继承了父类中的所有方法，`flush()`将缓冲中所有字节内容输出。



#### ByteArrayOutputStream

> 提供了缓冲区来暂存输出的字节数据

- 缓冲区会随着字节数据的不断增长不断写入而自动增长，输出完成以后可以从中提取数据。因此常用于存储数据以用于一次输出

- 用户可以通过`toByteArray()`和`toString`获取缓冲区中的数据
- `close()`无效

##### 构造方法

| 方法                              | 作用                                             |
| --------------------------------- | ------------------------------------------------ |
| `ByteArrayOutputStream()`         | 创建一个字节数组输出流对象                       |
| `ByteArrayOutputStream(int size)` | 创建一个指定缓冲区大小为size的字节数组输出流对象 |

##### 常用方法

继承了父类的方法，同时新增部分方法和重载`write()`。

| 方法                                   | 作用                                            |
| -------------------------------------- | ----------------------------------------------- |
| `int size()`                           | 返回缓冲区当前大小                              |
| `void reset()`                         | 将缓冲区重置为0                                 |
| `byte[] toByteArray()`                 | 创建一个新分配的byte数组                        |
| `String toString()`                    | 将缓冲区的字节数据转换成字符串                  |
| `String toString(String charsetName)`  | 将缓冲区的字节数据转换成指定字符集编码的字符串  |
| `void write(byte[] b,int off,int len)` | 将字节数组b从off位置开始的len个数据输出到缓冲区 |
| `void write(int b)`                    | 将字节数据b输出到缓冲区                         |
| `void writeTo(OutputStream out)`       | 将缓冲区中的字节数据输出到out中                 |



#### DataOutputStream

> 将Java的基本数据类型输出到目标中

##### 构造方法

`DataOutputStream(OutputStream out)`

以`OutputSream`对象为输出目标创建数据输出流

##### 常用方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/DataOutpueStream%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.png?raw=true)



#### PrintStream

> 打印流，以字节为单位输出数据，能方便输出各种数值标识形式，使用平台默认编码

> 可以使用`FileInputStream`或者`FileReader`类来以字节、字符读取输出文件内容

##### 构造方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/PrintStream%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.jpg?raw=true)

如果参数为`System.out`那么向标准输出设备输出字节数据

##### 常用方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/PringStream%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.jpg?raw=true)

增加了`print()`、`println()`、`printf()`方法将数据**原样**输出到输出目标中

## 字符流

### Reader

#### 常用方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/Reader%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.png?raw=true)



#### InputStreamReader

> 他是字节流通向字符流的桥梁

> 不支持`mark()`和`reset()`操作

> 可以使用`ready()`来判断此流是否准备好用于读取

##### 构造方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/InputStreamReader%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.png?raw=true)

##### 注意

- 此时的`read()`方法读取一个<font color=red>字符</font>，具体读取几个字节通过编码格式确定

- 常用的中文字符集是GB2312和GBK



#### FileReader

> 对指定文件以字符为单位进行读取

> 用`ready()`来判断输入源中是否准备好课进行读操作

> 不支持`mark()`和`reset()`操作，可以使用`skip()`

##### 构造方法

| 方法                            | 作用                   |
| ------------------------------- | ---------------------- |
| `FileReader(File fileName)`     | 以`File`对象创建新对象 |
| `FileReader(String fileName)`   | 以`FileName`创建新对象 |
| `FileReader(FileDescriptor fd)` | 以文件描述符创建新对象 |



#### BufferedReader

> 以字符为单位进行读写操作的缓冲流

> 支持`mark()`、`reset()`和`skip()`方法

##### 构造函数

| 方法                                 | 作用                                   |
| ------------------------------------ | -------------------------------------- |
| `BufferedReader(Reader in)`          | 创建缓冲字符输入流对象，缓冲大小默认   |
| `BufferedReader(Reader in,int size)` | 创建缓冲字符输入流，指定缓冲大小为size |

##### 常用方法

增加`readLine()`方法来读一行文本内容



#### CharArrayReader

> 字符数组输入流

##### 构造方法

| 方法                                                | 作用                                                      |
| --------------------------------------------------- | --------------------------------------------------------- |
| `CharArrayReader(char []buf)`                       | 指定数据源buf来创建对象                                   |
| `charArrayReader(char []buf,int offset,int length)` | 从字符数组buf的offset开始取length个字符作为输入源创建对象 |

### Writer

#### 常用方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/Writer%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.jpg?raw=true)



#### OutputStreamWriter

> 他是字符流转向字节流的桥梁

> 每次调用`write()`方法都会在字符上调用编码转换器

> 在写入底层输出流之前，字节会在缓冲区存放，所以<font color=red>在`close()`之前应该使用`flush()`</font>

##### 构造方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/OutputStreamWriter%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.png?raw=true)

创建向标准输出设备的输出

`OutputStreamWriter osw = new OutputStream(System.out)`



#### FileWriter

> 以字符为单位输出数据

> 可以使用`append()`方法向输出目标追加数据

> 因为带有缓冲区，所以`close()`之前使用`flush()`刷新缓冲区

##### 构造方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/FileWriter%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.png?raw=true)



#### BufferedWriter

> 以字符为单位向输出目标进行输出数据，提供单个字符，多个字符或者字符串的高效写入

> 在`close()`之前记得`flush()`

##### 构造方法

| 方法                                  | 作用                                   |
| ------------------------------------- | -------------------------------------- |
| `BufferedWriter(Writer out)`          | 创建字符缓冲输出流对象，缓冲大小默认值 |
| `BufferedWriter(Writer out,int size)` | 创建字符缓冲输出流，缓冲大小指定为size |

##### 常用方法

- 继承了Writer类中的所有方法

- 增加了`newLine`来向目标对象写入一个换行符

- 向标准输出设备输出：`new BufferedWriter(new OutputStreamWriter(System.out))`



#### CharArrayWriter

> 字符数组输出流，将字符数据写入字符缓冲区，缓冲区会随着写入数据自动增长

##### 构造方法

| 方法                        | 作用                       |
| --------------------------- | -------------------------- |
| `CharArrayWriter()`         | 创建缓冲区大小为默认的对象 |
| `CharArrayWriter(int size)` | 创建缓冲区大小为size的对象 |

##### 常用方法

继承了writer的所有方法，增加toCharArray()`和`toString()`来从缓冲区获取数据



#### PrinterWriter

> 以字符为单位向输出单位输出数据

##### 构造方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/PrinterWriter%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95.png?raw=true)

##### 常用方法

![image]()

- 新增了`print()`、`println()`方法向输出目**原样**打印数据，所以可以用字符流来读取文件内容
- 新增了`printf()`和`format()`方法向输出目标输出格式数据

### tips

- 字节流一般以`stream`结尾，字符流以`Reader/Writer`结尾
- 判断字节输入流结束一般用`available()`来判断，字符输入流结束用`ready()`来判断（<font color=red>对象输入输出流除外！</font>）

## 随机存取流

> 既能够读取又能够写入

### 构造方法

| 方法                                        | 作用                                       |
| ------------------------------------------- | ------------------------------------------ |
| `RandomAccessFile(String name,String mode)` | 创建向指定路径文件中随机读取和写入的新对象 |
| `RandomAccessFile(File file,String mode)`   | 创建向指定的文件中随机读取和写入的新对象   |

| mode  | 解释                                                         |
| ----- | ------------------------------------------------------------ |
| "r"   | 只读模式                                                     |
| "rw"  | 读/写模式                                                    |
| "rws" | 每次更新时，都对数据和元数据的写磁盘操作同时进行同步读/写模式 |
| "rwd" | 每次更新时，只对数据的写磁盘操作进行同步的读/写模式          |

