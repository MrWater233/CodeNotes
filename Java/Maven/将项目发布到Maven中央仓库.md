# 一.将项目推送到Github

在Github上新建一个仓库，将需要发布到项目推送到Github

# 二.注册并登录sonatype的Jira账号

## 2.1 注册

注册地址：https://issues.sonatype.org/secure/Signup!default.jspa

> 注意：username一定要是英文的，否则无法分配权限

## 2.2 登录Jira

登录地址：https://issues.sonatype.org/login.jsp

# 三.配置Issue

## 3.1 创建Issue

创建地址: https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134

这里只说明**必填**选项：

- Project：选择【Community Support - Open Source Project Repository Hosting (OSSRH)】

- Issue Type：选择【New Project】

- Summary：自己填写项目的概要，主要要使用英文

- Group Id：

  注意要与项目`pom`中的`groupId`一致，如`com.xhiteam.xauth`

  - 如果是用自己的域名作为`groupId`，那么必须要拥有该域名的所有权（官方人员会对此进行校验）

  - 如果使用的是github的page作为`groupId`，那么格式为`io.github.你的github用户名`

- Project URL：项目的地址，如：`https://github.com/xhiteam/x-auth`

- SCM url：项目的git地址，如：`https://github.com/xhiteam/x-auth`

其他的默认即可

## 3.2 确认域名

提交Issue一段时间过后就能收到官方的回复

如果使用的是自己的域名作为`groupId`，那么官方会要求DNS解析中添加一条类型为TXT的记录，内容为Issue的编号和地址

收到官方回复，确认域名完成后就可以进行发布项目了

具体的沟通过程：https://issues.sonatype.org/browse/OSSRH-62027

# 四.配置GPG

## 4.1 下载GPG

下载地址：https://www.gpg4win.org/download.html

但是下载会有一个坑，弹出一个捐赠地址，无法让你下载

你在这儿 [files.gpg4win.org](https://files.gpg4win.org/)找到相同的版本号进行下载

## 4.2 安装GPG

默认设置一直下一步就行，安装完成后桌面上就会出现一个`Kleopatra`图标

## 4.3 配置密钥对

1. 打开`Kleopatra `
2. 新建密钥对
3. 输入姓名和邮箱
4. **输入一个8位以上的密码，请记住该密码，接下去还会用到**
5. 完成创建，得到一串公钥

## 4.4 上传公钥到 GPG key-servers

打开CMD窗口，输入以下命令将公钥发送到远程服务器

```shell
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 你的公钥地址
```

# 五.配置项目

## 5.1 配置setting.xml

打开maven目录中的conf文件夹，找到sessting.xml文件

设置一个server，添加Jira的账号和密码

这里的id会在`pom.xml`的`<distributionManagement>`内进行使用

```xml
<servers>
   <server>
      <id>oss</id>
      <username>JIRA账号</username>
      <password>JIRA密码</password>
   </server>
</servers>
```

## 5.2 配置pom.xml

可以按照以下pom来进行配置

主要配置项目信息、发布仓库信息、Mavne文档插件、编译插件、资源插件、GPG验证插件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.xhiteam.xauth</groupId>
    <artifactId>xauth</artifactId>
    <version>0.1.0</version>

    <name>x-auth</name>
    <description>A lightweight back-end security framework</description>
    <url>https://github.com/xhiteam/x-auth</url>

    <packaging>pom</packaging>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven-compiler-plugin.version>3.8.1</maven-compiler-plugin.version>
    </properties>

    <modules>
        <module>xauth-core</module>
        <module>xauth-client</module>
        <module>xauth-impl</module>
        <module>xauth-server</module>
        <module>xauth-pom</module>
        <module>xauth-examples</module>
    </modules>

    <!-- 开源协议信息 -->
    <licenses>
        <license>
            <name>MIT</name>
            <url>https://github.com/xhiteam/x-auth/blob/develop/LICENSE</url>
            <distribution>repo</distribution>
        </license>
    </licenses>

    <!-- scm信息 -->
    <scm>
        <url>http://xhiteam.com</url>
        <connection>scm:git:https://github.com/xhiteam/x-auth.git</connection>
        <developerConnection>scm:git:https://github.com/xhiteam/x-auth.git</developerConnection>
    </scm>

    <!-- 组织信息 -->
    <organization>
        <name>Xhiteam</name>
        <url>https://xhiteam.com</url>
    </organization>

    <!-- 开发者信息 -->
    <developers>
        <developer>
            <name>Lambo Chen</name>
            <email>lambo.chen.2306@gmail.com</email>
            <url>https://github.com/lambochen</url>
        </developer>

        <developer>
            <name>JingMiao Wan</name>
            <email>wanjingmiao@gmail.com</email>
            <url>https://github.com/MrWater233</url>
        </developer>
    </developers>
	
    <!-- 中央仓库信息，引入setting.xml中的id -->
    <distributionManagement>
        <snapshotRepository>
            <id>oss</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
        <repository>
            <id>oss</id>
            <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
    </distributionManagement>

    <repositories>
        <repository>
            <id>group</id>
            <url>http://nexus.xhiteam.com/repository/group/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>aliyun-plugin</id>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>

    <build>
        <plugins>
            <!-- doc plugin,Maven API文档生成插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-javadoc-plugin</artifactId>
                <executions>
                    <execution>
                        <id>attach-javadocs</id>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- compiler plugin,Maven 编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>

            <!-- resources plugin,Maven 资源插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- gpg plugin,用于签名认证 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-gpg-plugin</artifactId>
                <executions>
                    <execution>
                        <id>sign-artifacts</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>sign</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            
        </plugins>
        <pluginManagement>
            <plugins>
                <plugin>
                    <artifactId>maven-javadoc-plugin</artifactId>
                    <version>3.1.0</version>
                    <configuration>
                        <additionalJOptions>
                            <additionalJOption>-Xdoclint:none</additionalJOption>
                        </additionalJOptions>
                    </configuration>
                </plugin>

                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                    <configuration>
                        <source>${java.version}</source>
                        <target>${java.version}</target>
                        <encoding>${project.build.sourceEncoding}</encoding>
                    </configuration>
                </plugin>

                <plugin>
                    <artifactId>maven-source-plugin</artifactId>
                    <version>3.2.1</version>
                </plugin>

                <plugin>
                    <artifactId>maven-gpg-plugin</artifactId>
                    <version>1.6</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>

```

# 六.发布到Mavne中央仓库

## 6.1  提交项目到OSS

使用maven命令部署到中央仓库

```shell
mvn clean deploy
```

过程中会输入之前设置的GPG密码

## 6.2 在OSS中发布

部署过程`Open阶段->Close阶段->Release阶段`

- 登录 [https://oss.sonatype.org](https://oss.sonatype.org/)。用户名、密码就是 JIRA 的用户名、密码。

- 找到左侧`Staging Repositories`，这儿就能找到你之前提交的构建，如果没问题他现在应该是`【Open】`状态

- 找到自己的构建，点击上方的`【Close】`关闭构建，过程比较慢，可能需要几分钟。

  如果有报错，你可以通过`Activity`查看他的关闭过程并对项目进行更改并重新提交。

- 关闭成功后，再次选中自己的构建，点击上方的`【Release】`进行发布构建

- 发布成功



这是手动化的发布过程，有一种自动化的发布过程，但是未作尝试

只需要在项目的`pom.xml`中加入以下插件

```xml
<!--staging puglin,用于自动执行发布阶段(免手动)-->
<plugin>
    <groupId>org.sonatype.plugins</groupId>
    <artifactId>nexus-staging-maven-plugin</artifactId>
    <version>1.6.7</version>
    <extensions>true</extensions>
    <configuration>
        <serverid>ossrh</serverid>
        <nexusurl>https://oss.sonatype.org/</nexusurl>
        <autoreleaseafterclose>true</autoreleaseafterclose>
    </configuration>
</plugin>
<!-- release plugin,用于发布到release仓库部署插件 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-release-plugin</artifactId>
    <version>2.4.2</version>
</plugin>
```

# 七.回复Issue发布完成

在之前提交的Issue地址下回复自己已经Release完成

# 八.查看自己发布的项目

发布成功后可以进入 [https://search.maven.org](https://search.maven.org/) 网址查询自己发布的 Jar，如果未查到等几个小时再此查询。

# 九.以后的提交

以后只需要**重复Maven发布，在OSS中`Close`、`Release`过程**就行了。当然还是要等待10分钟-2小时才能发布

如果发布新的项目，在使用相同的`groupId`的情况下，与上面的过程是一样的，只有使用不同的`groupId`时才需要重新提交Issue

如果未换电脑，GPG的过程也只需要一次就行

# 十. 坑

1. 问题：

   mvn clean deploy时报`Failed to execute goal org.apache.maven.plugins:maven-gpg-plugin:1.6:sign (sign-artifacts) on project xauth: Exit code: 2 -> [Help 1]`错误

   ```shell
   [ERROR] Failed to execute goal org.apache.maven.plugins:maven-gpg-plugin:1.6:sign (sign-artifacts) on project xauth: Exit code: 2 -> [Help 1]
   [ERROR]
   [ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
   [ERROR] Re-run Maven using the -X switch to enable full debug logging.
   [ERROR]
   [ERROR] For more information about the errors and possible solutions, please read the following articles:
   [ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
   ```

   解决方法：

   这是因为maven找不到gpg，这时候就直接在`setting.xml`中配置gpg的绝对路径和密码就可以了

   ```xml
   <profile>
   	<id>ossrh</id>
   	<activation>
   		<activeByDefault>true</activeByDefault>
   	</activation>
   	
   	<properties>
   		<gpg.executable>D:\Tools\GnuPG\bin\gpg</gpg.executable>
   		<gpg.passphrase>你的gpg的Passphrase</gpg.passphrase>
   	</properties>
   </profile>
   ```

# 十一.参考

- http://www.mydlq.club/article/15/
- https://www.jianshu.com/p/8c3d7fb09bce