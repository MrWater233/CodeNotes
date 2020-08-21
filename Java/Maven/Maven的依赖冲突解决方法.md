> 因为maven存在依赖传递导致两个依赖可能同时传递了相同的jar包，这时候就会产生冲突

# 一.依赖冲突调节原则

> Maven自身解决依赖冲突的方法

## 1.第一声明者优先原则

> 以坐标的导入顺序来确定最终使用哪个传递过来的依赖

## 2.路径近者优先原则

> 如果两个依赖同时依赖一个jar包，而此时在pom中直接引入那个jar包，那么maven不会使用传递过来的jar包，因为直接定义的路径更近

# 二.排除依赖

通过`<exclusion>`标签将传递的依赖排除出去

# <font color="red">三.版本锁定</font>

> **常用**
>
> 采用直接锁定版本的方法确定依赖jar包的版本，版本锁定后则不考虑依赖的声明顺序或路径，以锁定版本为主。

1. 在`<dependencyManagement>`标签中锁定依赖的版本（**只是进行版本锁定，并没有导入**）。通常是提取版本常量放在外部，通过`ognl`表达式引入
2. 在`<dependencies>`标签中声明需要导入的maven坐标（指定或者不指定版本都不影响）

例子：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <modules>
        <module>blog-web</module>
        <module>blog-service</module>
        <module>blog-repository</module>
        <module>blog-common</module>
        <module>blog-model</module>
    </modules>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.mrwater</groupId>
    <artifactId>blog-back</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <!--版本信息-->
    <properties>
        <blog-version>1.0-SNAPSHOT</blog-version>
        <mybatis-plus-version>3.3.1.tmp</mybatis-plus-version>
    </properties>

    <!--版本锁定-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.mrwater</groupId>
                <artifactId>blog-web</artifactId>
                <version>${blog-version}</version>
            </dependency>
            <dependency>
                <groupId>com.mrwater</groupId>
                <artifactId>blog-service</artifactId>
                <version>${blog-version}</version>
            </dependency>
            <dependency>
                <groupId>com.mrwater</groupId>
                <artifactId>blog-repository</artifactId>
                <version>${blog-version}</version>
            </dependency>
            <dependency>
                <groupId>com.mrwater</groupId>
                <artifactId>blog-model</artifactId>
                <version>${blog-version}</version>
            </dependency>
            <dependency>
                <groupId>com.mrwater</groupId>
                <artifactId>blog-common</artifactId>
                <version>${blog-version}</version>
            </dependency>
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>${mybatis-plus-version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!--SpringBoot-web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--SpringSecurity-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <!--MybatisPlus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <!--Lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

