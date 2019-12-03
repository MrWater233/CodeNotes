# File类

## 构造函数

![image]()

## 常用方法

### 访问文件信息方法

![image]()

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



## 字符流

