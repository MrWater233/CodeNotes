# 一、Spring Boot入门

## 1.Spring Boot部署

### 1.创建一个Maven工程

勾选自动下载依赖

### 2.导入依赖Spring Boot相关依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 3.编写一个主程序

```java
/*
    @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        //Spring应用启动
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

### 4.编写相关Controller

```java
@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello Word!";
    }
}
```

### 5.运行主程序测试

### 6.部署程序

```xml
<!--这个插件可以将应用打包成jar包-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

在Maven中的Lifecycle中运行package打成jar包

接下来就可以直接运行jar包运行项目

## 2.部署探究

### 1.POM文件

#### 1.父项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>

他的父项目是
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>1.5.9.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
他来真正管理Spring Boot应用里面的所有依赖版本
```

Spring Boot的版本仲裁中心

是一个由很多依赖的POM文件，这样就可以对所有项目的依赖进行统一的管理，不需要再去改所有项目的依赖。

以后我们导入依赖默认是不需要写版本的（没有在dependencies里面管理的依赖需要声明版本号）

#### 2.启动器

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

##### spring-boot-starter-web

Spring Boot场景启动器，导入了web模块正常运行需要的组件

Spring Boot将所有的功能场景抽取出来，做成启动器，只需要在项目中引入starter，相关场景的依赖就会导入。

### 2.主程序类

`@SpringBootApplication`：标注在某个类上，说明是SpringBoot的主配置类，SpringBoot应该运行这个类的Main方法来启动应用

`@SpringBootConfiguration`：SpringBoot的配置类，表明这个类是个配置类，配置类也是容器的组件`(@Component)`

`@EnableAutoConfiguration`：开启自动配置功能