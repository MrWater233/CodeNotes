# 一.SpringCloud Alibaba

## 1.SpringCloudAlibaba依赖导入

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

## 2.Nacos注册中心

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

## 3.Nacos配置中心

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
    spring.application.name=mall-coupon
    spring.cloud.nacos.config.server-addr=127.0.0.1:8848
   ```

4. 在nacos配置中心添加一个数据集

   规则：应用名.properties

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200409184903.png)

5. 在应用中动态获取配置

   ```java
   //启动热加载
   @RefreshScope
   class SampleController {
       @Value("${coupon.user.name}")
       private String name;
       
       @Value("${coupon.user.age}")
       private String age;
   }
   ```

<font color=red>即使不配置配置集，启动默认也会读取`服务名.properties`文件</font>

### 命名空间 

> 配置隔离
>
> 应用场景
>
> 1. 环境隔离（也可以用`配置分组`来区分环境）
> 2. 每一个微服务互相隔离配置

默认：public（保留空间），默认新增的所有配置都在public空间

1. 创建命名空间

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200424165045.png)

2. 在配置列表的新建命名空间中添加配置

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200424165233.png)

3. 修改`bootstrap.properties`添加以下配置指定命名空间

   ```properties
   spring.cloud.nacos.config.namespace=ea4f7ea9-877c-439d-8a2f-78a6ef6146d1
   ```

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200424165416.png)

### 配置集

> 一组配置的集合，就是一个一个配置文件

一个配置集：

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200426160805.png)

### 配置集ID

> 类似于配置文件名

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200424172140.png)

### 配置分组

> 默认所有的配置集都属于`DEFAULT_GROUP`
>
> 可用于区分不用的生产环境

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200424172140.png)

修改`bootstrap.properties`添加以下来配置指定配置分组

```properties
spring.cloud.nacos.config.group=配置的组名
```

### 多配置集的加载

先创建多个配置集

在`bootstrap.properties`中输入该命令配置配置集信息

```properties
spring.cloud.nacos.config.extension-configs[0].data-id=配置集ID
# 不指定组名默认为DEFAULT_GROUP
spring.cloud.nacos.config.extension-configs[0].group=组名
spring.cloud.nacos.config.extension-configs[0].refresh=是否动态刷新

# 接着用[1]、[2]...指定其他配置集
```

# 二.SpringCloud

## 1.Feign声明式远程调用

1. 导入pom依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 编写生产者的接口方法，表明这是一个申明式的远程调用

   `com.mall.coupon.controller.CouponController`

   ```java
   @RestController
   @RequestMapping("coupon/coupon")
   public class CouponController {
   ...
       @RequestMapping("/member/list")
       public R memberCoupons(){
           CouponEntity couponEntity = new CouponEntity();
           couponEntity.setCouponName("满100-10");
           return R.ok().put("coupons",Arrays.asList(couponEntity));
       }
   ...
   }
   ```

3. 消费者编写一个接口，告诉SpringCloud这个接口需要调用远程服务

   `com.mall.member.Feign.CouponFeignService`

   ```java
   //参数为注册中心注册的微服务名
   @FeignClient("mall-coupon")
   public interface CouponFeignService {
       //需要指定全路径
       @RequestMapping("coupon/coupon/member/list")
       public R memberCoupons();
   }
   ```

4. 消费者在启动类上开启远程调用功能

   ```java
   //指定执行远程调用方法所在的包
   @EnableFeignClients(basePackages = "com.mall.member.Feign")
   @EnableDiscoveryClient
   @SpringBootApplication
   public class MallMemberApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(MallMemberApplication.class, args);
       }
   
   }
   ```

5. 消费者调用

   ```java
   @Autowired
   CouponFeignService couponFeignService;
   
   @GetMapping("/coupons")
   public R test(){
       MemberEntity memberEntity = new MemberEntity();
       memberEntity.setNickname("张三");
       R memberCoupons = couponFeignService.memberCoupons();
       return R.ok().put("member",memberEntity).put("coupons",memberCoupons);
   }
   ```


## 2.Gateway

> 请求通过网关，经过`断言`的判断选择合适的`路由`，再经过`过滤器`到达微服务

### 配置初始化

1. 创建Gateway项目

2. 导入nacos注册中心和配置中心依赖

3. 加入服务发现

   ```java
   @EnableDiscoveryClient
   @SpringBootApplication
   public class MallGatewayApplication {
       public static void main(String[] args) {
           SpringApplication.run(MallGatewayApplication.class, args);
       }
   }
   ```

4. 配置`application.yaml`

   ```yaml
   spring:
     application:
       name: mall-gateway
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
           
   server:
     port: 88
   ```

5. 配置`bootstrap.propreties`

   ```properties
   spring.application.name=mall-gateway
   spring.cloud.nacos.config.server-addr=127.0.0.1:8848
   spring.cloud.nacos.config.namespace=73921d85-7521-4b02-99c3-0e550ea37cbc
   ```

   

