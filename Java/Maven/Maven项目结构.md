```text
my-project
├── .mvn
│   └── wrapper
│       ├── MavenWrapperDownloader.java
│       ├── maven-wrapper.jar
│       └── maven-wrapper.properties
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        ├── java
        └── resources
```

以上为一个Maven项目的结构

- `src、pom.xml`为**基础**的Maven结构

- `.mvn/wrapper、mvnw、mvnw.cmd`为**Maven Wrapper**需要的文件，它的作用是**统一项目的Mavne版本**

  - 原理：在`maven-wrapper.properties`中记录使用的Maven版本。当用户执行`mvnw clean`时发现当前用户的Maven版本和期望的版本不一致时就下载期望的版本，然后用期望的版本来执行`mvn`命令，比如`mvn clean`

  - 安装

    命令皆在`pom.xml`目录下执行

    - 最新版本的Maven

      ```shell
      mvn -N io.takari:maven:0.7.6:wrapper
      ```

      他会自动使用最新版本的Maven版本，`0.7.6`表示Maven Wrapper（[查看最新的Maven Wrapper版本](https://github.com/takari/maven-wrapper)）

    - 指定版本的Maven

      ```shell
      mvn -N io.takari:maven:0.7.6:wrapper -Dmaven=3.3.3
      ```

      指定Maven版本为`3.3.3`

