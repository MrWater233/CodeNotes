# 一.SpringCloudAlibaba依赖导入

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

# 二.Nacos注册中心

[官方教程](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example/readme-zh.md)

1. 下载并启动[Nacos Server](https://github.com/alibaba/nacos/releases)

2. 修改pom，导入注册中心Starter

   ```xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
   ```

3. 修改配置文件，配上注册中心地址并配上当前微服务的名称

   ```properties
   spring:
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
     application:
       name: mall-member
   ```

4. 在微服务启动类上加上`@EnableDiscoveryClient`注解开启服务注册与发现

5. 登陆`127.0.0.1:8848/nacos`账号密码默认为`nacos`

   查看微服务注册情况

# 三.Nacos配置中心

[官方教程](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/nacos-example/nacos-config-example/readme-zh.md)

1. 下载并启动[Nacos Server](https://github.com/alibaba/nacos/releases)

2. 修改pom，导入配置中心Starter

   ```xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
   ```

3. 在应用的 `/src/main/resources/bootstrap.properties` 配置文件中配置 Nacos Config 元数据

   该配置文件会优先于SpringBoot配置文件启动

   ```properties
    spring.application.name=nacos-config-example
    spring.cloud.nacos.config.server-addr=127.0.0.1:8848
   ```

4. 在nacos配置中心添加一个数据集

   规则：应用名.properties

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200409184903.png)

5. 在应用中动态获取配置

   ```java
    @RefreshScope
    class SampleController {
   
    	@Value("${coupon.user.name}")
    	String userName;
   
    	@Value("${coupon.user.age}")
    	int age;
    }
   ```