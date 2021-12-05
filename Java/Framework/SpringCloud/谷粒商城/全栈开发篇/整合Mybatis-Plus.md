1. 导入依赖

   ```xml
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus-boot-starter</artifactId>
       <version>3.3.1</version>
   </dependency>
   ```

2. 配置数据源

   1. 导入数据库驱动，[驱动和MySQL版本的对应关系](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-versions.html)，以下为MySQL5.7为例

      ```xml
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>8.0.19</version>
      </dependency>
      ```

   2. 配置数据源并整合Mybatis-Plus

      1. 配置数据库基本配置

         ```yaml
         spring:
           datasource:
             url: jdbc:mysql://xxx:3306/mall_pms?useUnicode=true&characterEncoding=UTF-8&useSSL=false
             username: root
             password: 123456
             driver-class-name: com.mysql.cj.jdbc.Driver
         ```

      2. 配置`@MapperScan`

3. 配置Mybatis-Plus相关信息

   ```yaml
   mybatis-plus:
     # 第一个*表示不止项目类路径，依赖jar包类路径也要扫描
     mapper-locations: classpath*:/mapper/**/*.xml
     # 主键自增
     global-config:
       db-config:
         id-type: auto
   ```

   