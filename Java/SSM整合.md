# 一.项目创建

![image-20200626095836328](https://raw.githubusercontent.com/MrWater233/PictureHost/master/image-20200626095836328.png)

# 二.项目目录

![image-20200626183711861](https://raw.githubusercontent.com/MrWater233/PictureHost/master/image-20200626183711861.png)

# 三.依赖

```xml
<!--Spring相关-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.5</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.2.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.7.RELEASE</version>
</dependency>

<!--Servlet和jsp-->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.0</version>
</dependency>
<dependency>
    <groupId>jstl</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>

<!--mybatis相关-->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.5</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
</dependency>
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>

<!--测试-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>

<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.12</version>
</dependency>
```

# 四.web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--Spring配置-->
    <!--全局配置文件，配置Spring配置文件位置，用于Spring监听器初始化bean工厂-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <!--Spring监听器配置-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!--SpringMVC前端控制器-->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--编码过滤器-->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!--解决请求乱码-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <!--解决响应乱码-->
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

# 五.Mybatis配置

`mybatisConfig.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--定义别名-->
    <typeAliases>
        <!--方法一.定义具体的别名-->
        <!--<typeAlias type="com.mrwater.ssm.entity.User" alias="user"></typeAlias>-->
        <!--方法二.扫包，别名为类名首字母小写/原名称-->
        <package name="com.mrwater.ssm.entity"/>
    </typeAliases>

    <!--不整合时的原始配置-->
    <!--    &lt;!&ndash;加载配置文件&ndash;&gt;-->
    <!--    <properties resource="classpath:jdbc.properties"></properties>-->
    <!--    &lt;!&ndash;环境&ndash;&gt;-->
    <!--    <environments default="development">-->
    <!--        <environment id="development">-->
    <!--            <transactionManager type="JDBC"></transactionManager>-->
    <!--            <dataSource type="POOLED">-->
    <!--                <property name="driver" value="${jdbc.driver}"/>-->
    <!--                <property name="url" value="${jdbc.url}"/>-->
    <!--                <property name="username" value="${jdbc.username}"/>-->
    <!--                <property name="password" value="${jdbc.password}"/>-->
    <!--            </dataSource>-->
    <!--        </environment>-->
    <!--    </environments>-->

    <!--    &lt;!&ndash;加载映射&ndash;&gt;-->
    <!--    <mappers>-->
    <!--        &lt;!&ndash;方法一.具体加载&ndash;&gt;-->
    <!--        &lt;!&ndash; <mapper resource="mapper/UserMapper.xml"></mapper>&ndash;&gt;-->
    <!--        &lt;!&ndash;方法二.扫包&ndash;&gt;-->
    <!--        <package name="mapper"/>-->
    <!--    </mappers>-->
</configuration>
```

# 六.JDBC参数配置文件

`jdbc.properties`

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?useUnicode=true&charactorEncoding=utf8&useSSL=false&zeroDateTimeBehavior=convertToNull&serverTimezone=GMT%2B8
jdbc.username=root
jdbc.password=123456
```

# 七.Spring配置文件

`appliactionContext.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--组件扫描，排除Controller-->
    <context:component-scan base-package="com.mrwater.ssm">
        <context:exclude-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"></context:exclude-filter>
    </context:component-scan>

    <!--加载properties文件-->
    <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"></property>
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="user" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
    </bean>

    <!--配置sessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--数据源-->
        <property name="dataSource" ref="dataSource"></property>
        <!--配置Mybatis配置文件-->
        <property name="configLocation" value="classpath:mybatisConfig.xml"></property>
        <!--Mapper映射文件位置-->
        <property name="mapperLocations" value="classpath:mapper/*.xml"></property>
    </bean>

    <!--扫描Dao层-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--Dao层路径-->
        <property name="basePackage" value="com.mrwater.ssm.repository"></property>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>

    <!--声明式事务控制-->
    <!--平台事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--数据源-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--配置事务增强-->
    <tx:advice id="txAdvice">
        <tx:attributes>
            <!--匹配所有方法-->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
    <!--事务的AOP织入-->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.mrwater.ssm.service.impl.*.*(..))"></aop:advisor>
    </aop:config>
</beans>
```

# 八.SpringMVC配置文件

`spring-mvc.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!--组件扫描(Controller)-->
    <context:component-scan base-package="com.mrwater.ssm.controller"></context:component-scan>
    <!--mvc注解驱动-->
    <mvc:annotation-driven></mvc:annotation-driven>
    <!--内部资源视图解析器-->
    <bean id="resourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--视图资源路径前缀-->
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <!--视图资源后缀-->
        <property name="suffix" value=".jsp"></property>
    </bean>
    <!--开放静态资源访问权限-->
    <mvc:default-servlet-handler></mvc:default-servlet-handler>
</beans>
```

# 九.三层架构

`UserMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mrwater.ssm.repository.UserRepository">
    <select id="getAll" resultType="user">
        SELECT * FROM user
    </select>
</mapper>
```

`UserRepository`

```java
import com.mrwater.ssm.entity.User;

import java.util.List;

public interface UserRepository {
    public List<User> getAll();
}
```

`UserService`

```java
import com.mrwater.ssm.entity.User;

import java.util.List;

public interface UserService {
    public List<User> getAll();
}

```

`UserServiceImpl`

```java
import com.mrwater.ssm.entity.User;
import com.mrwater.ssm.repository.UserRepository;
import com.mrwater.ssm.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserServiceImpl implements UserService {
    @Autowired
    UserRepository userRepository;

    @Override
    public List<User> getAll() {
        return userRepository.getAll();
    }
}

```

`UserController`

```java
import com.mrwater.ssm.entity.User;
import com.mrwater.ssm.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import java.util.List;

@Controller
@RequestMapping("/user")
public class UserController {
    @Autowired
    UserService userService;

    @GetMapping
    public ModelAndView getAll(ModelAndView modelAndView){
        List<User> userList = userService.getAll();
        modelAndView.addObject("userList",userList);
        modelAndView.setViewName("userList");
        return modelAndView;
    }
}

```

# 十.页面

`userList.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>展示用户数据</title>
</head>
<body>
    <table>
        <tr>
            <th>用户ID</th>
            <th>用户名</th>
            <th>用户密码</th>
        </tr>
        <c:forEach items="${userList}" var="user">
            <tr>
                <td>${user.id}</td>
                <td>${user.username}</td>
                <td>${user.password}</td>
            </tr>
        </c:forEach>
    </table>
</body>
</html>
```

# 十一.日志

`log4j.properties`

```properties
log4j.rootLogger = debug,console

### log file ###
log4j.appender.debug = org.apache.log4j.DailyRollingFileAppender
log4j.appender.debug.Append = true
log4j.appender.debug.Threshold = DEBUG
log4j.appender.debug.layout = org.apache.log4j.PatternLayout
log4j.appender.debug.layout.ConversionPattern = %-d{yyyy-MM-dd HH\:mm\:ss} [%p]-[%c] %m%n

## console##
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern =  %d{ABSOLUTE} %5p %c{ 1 }:%L - %m%n
```

