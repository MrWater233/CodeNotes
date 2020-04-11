# 一.Feign声明式远程调用

1. 导入pom依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 编写生产者的接口方法，表明这是一个申明式的远程调用

   ```java
   @RequestMapping("/member/list")
   public R memberCoupons(){
       CouponEntity couponEntity = new CouponEntity();
       couponEntity.setCouponName("满100-10");
       return R.ok().put("coupons",Arrays.asList(couponEntity));
   }
   ```

3. 消费者编写一个接口，告诉SpringCloud这个接口需要调用远程服务

   `com.mall.member.Feign.CouponFeignService`

   ```java
   //参数为注册中心注册的微服务名
   @FeignClient("mall-coupon")
   public interface CouponFeignService {
       @RequestMapping("coupon/coupon/member/list")
       public R memberCoupons();
   }
   ```

4. 消费者在启动类上开启远程调用功能

   ```java
   //指定执行远程调用的包
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

   

