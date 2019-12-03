# File类

## 构造函数

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

##### 构造函数

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



## 字符流

### Reader

#### 常用方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/Reader%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.png?raw=true)

### Writer

#### 常用方法

![image](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/Writer%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.jpg?raw=true)

