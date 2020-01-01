# 概述

> socket起源于UNIX，在Unix一切皆文件哲学的思想下，socket是一种"打开—读/写—关闭"模式的实现，服务器和客户端各自维护一个"文件"，在建立连接打开后，可以向自己文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。 

# Socket

## 创建Socket对象

`new Socket(ip,port)`

参数为服务端的IP地址和端口号

## 获取输入输出流

`.getInputStream()`

获取一个输入流，可以用`.read(bytes[])`来将网络传输读取进一个字节数组中

`.getOutputStream()`

获取一个输出流，可以用`.write(bytes[])`来将字节数组传输出去

### 注意

该输入输出流和IO流不一样，这个用于网络传输，不能和new出来的IO流混用

# ServerSocket

> 服务端用来接收Socket的对象

## 创建对象

`new ServerSocket(port)`

暴露一个服务，服务地址：本机IP:端口号

## 监听请求

`.accept()`

返回一个`Socket`类型数据，可以用它来操作并传输的数据

# 例子

## 1.单线程传输文件

客户端：

```java
package socket;

import java.io.*;
import java.net.Socket;

public class Client {
    public static void main(String[] args) throws IOException {
        //客户端访问服务端发布的服务
        Socket socket = new Socket("127.0.0.1",9999);

        //创建网络输入流对象
        InputStream in = socket.getInputStream();
        //接收文件
        File file = new File("Path");
        //创建本地的输入输出流
        OutputStream fileOut = new FileOutputStream(file);
        //接收每次文件的大小
        byte[] bs = new byte[1000];
        int len = -1;
        while((len = in.read(bs))!=-1){
            fileOut.write(bs,0,len);
        }
        System.out.println("下载成功！");

        //关闭相关流
        fileOut.close();
        in.close();
        socket.close();
    }
}
```

服务端：

```java
package socket;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static void main(String[] args) throws IOException {
        //暴露一个服务，该服务地址：本机ip:9999
        ServerSocket server = new ServerSocket(9999);
        while(true) {
            //监听是否有客户端访问
            Socket socket = server.accept();
            System.out.println("连接成功！");
            //创建网络输出流对象
            OutputStream out = socket.getOutputStream();
            //准备要发送的文件
            File file = new File("Path");
            //将此文件从硬盘读取进内存
            InputStream fileIn = new FileInputStream(file);
            //定义每次发送文件的大小
            byte[] fileBytes = new byte[1000];
            int len = -1;
            //发送（因为文件较大，不能一次发送完毕，所以需要通过循环来分次发送）
            while ((len = fileIn.read(fileBytes)) != -1) {
                out.write(fileBytes, 0, len);
            }
            System.out.println("发送成功！");

            //关闭相关流
            fileIn.close();
            out.close();
            socket.close();
        }
    }
}
```

## 2.多线程并发下载

服务端：

```java
package socket;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static void main(String[] args) throws IOException {
        //暴露一个服务，该服务地址：本机ip:9999
        ServerSocket server = new ServerSocket(9999);
        while(true) {
            //监听是否有客户端访问
            Socket socket = server.accept();
            //开启下载的线程
            Thread th = new Thread(new MyDownload(socket));
            th.start();
        }
    }
}
```

MyDownload：

```java
package socket;

import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

//处理下载的线程
public class MyDownload implements Runnable{
    private Socket socket;

    public MyDownload(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            System.out.println("连接成功！");
            //创建网络输出流对象
            OutputStream out = socket.getOutputStream();
            //准备要发送的文件
            File file = new File("Path");
            //将此文件从硬盘读取进内存
            InputStream fileIn = new FileInputStream(file);
            //定义每次发送文件的大小
            byte[] fileBytes = new byte[1000];
            int len = -1;
            //发送（因为文件较大，不能一次发送完毕，所以需要通过循环来分次发送）
            while ((len = fileIn.read(fileBytes)) != -1) {
                out.write(fileBytes, 0, len);
            }
            System.out.println("发送成功！");

            //关闭相关流
            fileIn.close();
            out.close();
            socket.close();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```