# 一.SpringMVC简介

> Spring实现web模块，简化web开发

![image-20200625115306417](https://raw.githubusercontent.com/MrWater233/PictureHost/master/image-20200625115306417.png)

SpringMVC将参数封装等行为抽取出来成为**前端控制器**

本质上**前端控制器**就是一个Servlet，他接收所有路径的请求，处理完成后再分发给不同的控制器(POJO)

配置步骤：

1. 导入SpringMVC依赖
2. 在`web.xml`中配置前端控制器DispatcherServlet（普通Servlet）
3. 编写Controller
4. 将Controller通过注解配置到容器中
5. 配置spring-mvc.xml配置组件扫描（扫描Controller，其他由Spring扫描）

# 二.环境配置

[基础IDEA环境搭建](https://yq.aliyun.com/articles/693866)

pom依赖：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<!--mvc模块+spring核心容器模块-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.4.RELEASE</version>
</dependency>
```

# 三.HelloWorld

## 3.1 编码

> SpringMVC思想是有一个**前端控制器**能拦截所有请求，智能派发
>
> 这个前端容器是一个Servlet，在web.xml中配置去拦截所有请求

1. `WEB-INF/web.xml`

   ```xml
   <!DOCTYPE web-app PUBLIC
    "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
    "http://java.sun.com/dtd/web-app_2_3.dtd" >
   
   <web-app>
     <display-name>Archetype Created Web Application</display-name>
       <!--解决POST乱码问题,一般都在其他filter之前-->
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
     
       <!--SpringWeb前端控制器-->
       <servlet>
           <servlet-name>app</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <!--Servlet初始化参数-->
           <init-param>
               <!--指定SpringMVC配置文件位置-->
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:springmvc.xml</param-value>
           </init-param>
           <!--Servlet服务器启动就加载-->
           <load-on-startup>1</load-on-startup>
       </servlet>
       <!--前端控制器的映射-->
       <servlet-mapping>
           <servlet-name>app</servlet-name>
           <!--/*和/都是拦截所有请求
           /*的范围更大，还会拦截到*.jsp这些请求-->
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   </web-app>
   ```

2. SpringMVC配置文件配置,配置文件本质为Spring配置文件

   `resources/springmvc.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
       <!--扫描controller组件-->
       <context:component-scan base-package="com.mvc.controller"></context:component-scan>
       
       <!--配置MVC注解驱动，启用处理器映射器和处理器适配器-->
       <mvc:annotation-driven/>
   
       <!--配置一个视图解析器，能够帮助我们拼接页面地址，简化Controller配置-->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <!--前缀-->
           <property name="prefix" value="/WEB-INF/pages/"></property>
           <!--后缀-->
           <property name="suffix" value=".jsp"></property>
       </bean>
       
       <!--开放静态资源访问权限-->
       <mvc:default-servlet-handler></mvc:default-servlet-handler>
   </beans>
   ```

3. 在`WEB-INF/pages`下定义`success.jsp`

4. 编写Controller

   `com.mvc.controller.MyFirstController`

   ```java
   @Controller
   public class MyFirstController {
       @GetMapping("/hello")
       public String myFirstRequest(){
           System.out.println("请求收到了...正在处理中");
           //不编辑视图解析器的情况：
           //return "/WEB-INF/pages/success.jsp";
           //使用视图解析器实现自动拼接:
           return "success";
       }
   }
   ```

## 3.2 运行流程

1. 客户端发送hello请求
2. tomcat服务器接受并交给SpringMVC的前端控制器
3. 前端处理器将请求和Controller中的注解进行匹配并执行目标方法
4. SpringMVC认为返回值是要去的页面地址
5. 用视图解析器进行拼串拿到完整的页面地址
6. 前端控制器进行**转发**到相应的页面

## 3.3 细节

### /和/*

1. 所有项目的小web-xml都是继承于大web.xml

   服务器的大web.xml中有一个DeafultServlet是url-pattern=/,他是处理静态资源的

2. 我们配置的前端控制器url-pattern=/

3. 我们前端控制器禁用了大web.xml的DeafultServlet

4. /*直接拦截所有,而/没有覆盖大web.xml中的JspServlet

# 四.注解

## @RequestMapping

> 1. 标注在类上指定改类所有url的根url
>
> 2. 标注在方法上指定方法的url

### 属性

1. value:请求路径

2. method:限定请求方式,默认什么都接受

3. params:规定请求参数,为一个数组

   例子 :

   ```java
   //规定一定要username和pwd参数
   @RequestMapping(parms = {"username","pwd"})
   //规定一定不要username参数
   @RequestMapping(parms = {"!username"})
   ```

4. headers:规定请求头

   例子:

   ```java
   //让火狐可以访问,谷歌不能访问(User-Agent)
   @RequestMapping(headers = {"User-Agent=火狐的User-Agent"})
   ```

5. consumes:规定几首类型,即规定请求头中的Content-Type

6. 告诉浏览器返回的内容是什么,即给响应头加上Content-Type

### Ant风格URL

> URL地址可以写模糊通配符

- `?`:能替代任意一个字符
- `*`:能替代任意多个字符和一层路径
- `**`:能替代多层路径

## @PathVariable

> 标注在方法参数上,获取路径占位符`{}`的值

## @RequestParam

> 标注在方法参数上,获取请求参数,默认参数必须有

### 属性

- value:获取的请求参数
- required:参数是否必须
- defaultValue:没带参数的默认值

## @RequestHeader

> 标注在方法参数上,获取请求头中的指定key中的value值
>
> 如果请求中没有相应的key就会报错

### 属性

- value:获取的请求参数
- required:key是否必须
- defaultValue:没带参数的默认值

## @CookieValue

> 标注在方法参数上,获取某个Cookie的值
>
> 如果请求中没有相应的Cookie值就会报错

### 属性

- value:获取的Cookie值
- required:Cookie值是否必须
- defaultValue:没带Cookie的默认值

# 五.REST风格

> 对资源的增删改查用请求方式来区分

- 格式:

  `/资源名/资源表示符`

- 请求方式:

  - GET---查询
  - PUT---更新
  - DELETE---删除
  - POST---添加

## Spring对Rest风格的支持

> 因为表单没有PUT和DELETE，所以需要对Spring进行设置间接使用这种请求风格

高版本tomcat:将`iErrorPage="true"`加进jsp中

1. 配置filter,记得更改位置

   `web.xml`

   ```xml
   <filter>
       <filter-name>HiddenHttpMethodFilter</filter-name>
       <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>HiddenHttpMethodFilter</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

2. 创建一个post类型的表单

   表单项中携带name为_method

   value为DELETE/PUT

   ```html
   <input name="_method" value="delete"/>
   ```

# 六.SpringMVC将数据带给页面

1. 在方法参数传入Map、Model或者ModelMap

   通过`addAttribute()`加入数据

   三种类型的关系:最终都是使用BindingAwareModelMap

   可以在页面可以在**请求域**中获取

2. 方法返回值定义为ModelAndView类型（可以自己new也可以在方法形参上让SpringMVC传递）

   在这个类型中设置转发的视图(构造方法或者`setViewName()`)

   使用`addObject()`方法在ModelAndView中加入`k-v`

   可以在页面可以在**请求域**中获取

3. 使用SpringMVC提供的临时给Session域中保存数据的方式

   在**类上**标注`@SessionAttributes`

   属性:

   - value:指定一个key值,

     当给BindingAwareModelMap/ModelAndView绑定数据一个`key=设置的key`时,在session中也会保存一份

   - types:指定一个类型

     只要在BindingAwareModelMap/ModelAndView保存一个**value为指定类型**的数据时session中也会保存一份

4. `@ModelAttribute`

   方法位置：提前于目标方法运行，在参数上设置map来保存数据

   参数位置：获取map中的数据，或者不用该注解直接用model也能实现数据交互

   作用:

   controller先从数据库中取出对象,而不是让SpringMVC去创建对象

   再接收前端传入属性去覆盖对象属性,保证了传入数据库对象不会为null

# 七.转发和重定向

- 转发：

  - 可以使用`forward:/hello.jsp`转发到项目名的jsp页面

  - 也可以使用`forward:/hello`转发到对应的地址

- 重定向：

  - 可以使用`redirect:/hello.jsp`重定向到项目名下的jsp页面

  - 也可以使用`redirect:/hello`重定向到对应的地址

# 八.自定义类型转换器

1. 实现`Converter<S,T>`接口（由S向T转化），定义自定义的转换器
2. 在IOC中配置`ConversionServiceFactoryBean`，添加我们自定义的类型转换器
3. 让MVC使用我们自定义的转换器：`<mvc:annotation-driven conversion-service="">`

# 九.`<mvc:annotation-driven/>`

- ```xml
  <mvc:default-servlet-handler/>
  <mvc:annotation-driven/>
  ```

  - 都没配：静态资源不能访问
    - 原因：只有DefaultAnnotationHandlerMapping映射，而其中的handlerMap中没有保存静态资源映射的请求，只有所有的动态资源的映射
  - 只加`<mvc:default-servlet-handler/>`，动态资源不能访问
    - 动态不能访问原因：DefaultAnnotationHandlerMapping没了，所以动态不能访问
    - 静态能够访问原因：DefaultAnnotationHandlerMapping被SimpleUrlHandlerMapping替换，它将所有请求交给Tomcat处理，所以静态能够访问
  - 只加`<mvc:annotation-driven/>`，静态资源不能访问
    - 没有RequestMappingHandlerMapping
  - 都加，静态动态都能访问
    - 原因：handlerMappings中有RequestMappingHandlerMapping和SimpleUrlHandlerMapping

# 十.数据校验

## 日期格式化

`@DataTimeFormat(pattern="yyyy-MM-dd hh:mm:ss")`：标注在实体类的属性上，可以指定输入的日期格式

## 数据校验

### 校验注解

`hibernate-validator`

| 限制                      | 说明                                                         |
| :------------------------ | :----------------------------------------------------------- |
| @Null                     | 限制只能为null                                               |
| @NotNull                  | 限制必须不为null                                             |
| @AssertFalse              | 限制必须为false                                              |
| @AssertTrue               | 限制必须为true                                               |
| @DecimalMax(value)        | 限制必须为一个不大于指定值的数字                             |
| @DecimalMin(value)        | 限制必须为一个不小于指定值的数字                             |
| @Digits(integer,fraction) | 限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction |
| @Future                   | 限制必须是一个将来的日期                                     |
| @Max(value)               | 限制必须为一个不大于指定值的数字                             |
| @Min(value)               | 限制必须为一个不小于指定值的数字                             |
| @Past                     | 限制必须是一个过去的日期                                     |
| @Pattern(value)           | 限制必须符合指定的正则表达式                                 |
| @Size(max,min)            | 限制字符长度必须在min到max之间                               |
| @Past                     | 验证注解的元素值（日期类型）比当前时间早                     |
| @NotEmpty                 | 验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0） |
| @NotBlank                 | 验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的空格 |
| @Email                    | 验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式 |

### 获取校验结果

在需要校验的对象前加上`@Valid`注解，后面**紧跟**上`BindingResult`类型获取校验结果

# 十一.AJAX

## 依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.0</version>
</dependency>
```

## 相关注解和类

- `@JsonIgnore`：标注在对象属性上，当前属性不会作为Json返回
- `@JsonFormat`：标注在对象属性上，将数据Json格式化以后返回
- `@RequesBody`：标注在方法参数上，获取请求体的内容，如果请求体是JSON格式会自动转为对象
- `@ResponseBody`：标注在方法上，将数据放在响应体中返回，如果返回的是对象，将返回Json数据
- `HttpEntity<>`：标注在方法参数上，获取请求体和请求头
- `ReponseEntity<>`：可以自定义一个响应，可以定义响应状态码、响应头、响应体的内容

## 例子

- ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Title</title>
      <script src="../../js/jquery.js"></script>
      <script>
          function responseJSON() {
              $.ajax({
                  url: '/testReturnJson',
                  type: 'post',
                  dataType: 'json',
                  success: function (data) {
                      alert(data);
                  }
              });
          }
      </script>
  </head>
  <body>
  <input type="button" value="响应JSON" onclick="responseJSON()">
  </body>
  </html>
  ```

- ```java
  @PostMapping("/testReturnJson")
  @ResponseBody
  public User testReturnJson(){
      User user = new User();
      user.setUsername("张三");
      user.setPassword("123456");
      return user;
  }
  ```

# 十二.文件相关

## 下载

```java
@GetMapping("/download")
public ResponseEntity<byte[]> download(HttpServletRequest request) {
    /**
     * 获取文件的真实路径
     */
    ServletContext context = request.getServletContext();
    String realPath = context.getRealPath("/js/jquery.js");
    //创建保存文件的字节数组
    byte[] temp = null;
    try {
        //创建文件输入流
        FileInputStream fileInputStream = new FileInputStream(realPath);
        //新建和文件输出流大小一样大的字节数组
        temp = new byte[fileInputStream.available()];
        //将文件读取进自己数组中
        fileInputStream.read(temp);
        fileInputStream.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    /**
     * 返回将要下载的文件流
     */
    HttpHeaders httpHeaders = new HttpHeaders();
    //控制用户请求所得的内容存为一个文件的时候提供一个默认的文件名
    httpHeaders.set("Content-Disposition", "attachment;filename=" + "jquery.js");
    return new ResponseEntity<byte[]>(temp, httpHeaders, HttpStatus.OK);
}
```

## 上传

1. 依赖

   ```xml
   <dependency>
       <groupId>commons-io</groupId>
       <artifactId>commons-io</artifactId>
       <version>2.6</version>
   </dependency>
   <dependency>
       <groupId>commons-fileupload</groupId>
       <artifactId>commons-fileupload</artifactId>
       <version>1.4</version>
   </dependency>
   ```

2. 配置文件上传解析器

   `springmvc.xml`

   ```xml
   <!--配置文件上传解析器-->
   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
       <!--设置最大上传大小-->
       <property name="maxUploadSize" value="#{1024*1024*1024}"></property>
       <!--设置字符编码-->
       <property name="defaultEncoding" value="utf-8"></property>
   </bean>
   ```

3. `index.jsp`

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>upload</title>
       <%--开启JSP EL表达式--%>
       <%@ page isELIgnored="false" %>
       <%--获取路径-->
       <% pageContext.setAttribute("ctp",request.getContextPath()); %>
   </head>
   <body>
   <form action="${ctp}/upload" method="post" enctype="multipart/form-data">
       用户头像：<input type="file" name="img"/></br>
       <input type="submit"/></br>
       ${msg}
   </form>
   </body>
   </html>
   
   ```

4. ```java
   @PostMapping("/upload")
   public String upload(@RequestParam(value = "img") MultipartFile file,
                        Model model) {
   
       try {
           //设置文件保存路径并保存
           file.transferTo(new File("C:\\Users\\JM Wan\\Desktop\\" + file.getOriginalFilename()));
           model.addAttribute("msg", "文件上传成功");
       } catch (IOException e) {
           model.addAttribute("msg", "文件上传失败");
           e.printStackTrace();
       }
       return "forward:/index.jsp";
   }
   ```

    其中

   ```java
   file.getName();//文件项的名字，该处为img
   file.getOriginalFilename();//文件原始名字
   ```

## 多文件上传

1. 依赖

2. 配置文件上传解析器

3. `index.jsp`

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>upload</title>
       <%--开启JSP EL表达式--%>
       <%@ page isELIgnored="false" %>
       <% pageContext.setAttribute("ctp", request.getContextPath()); %>
   </head>
   <body>
   <form action="${ctp}/uploadfiles" method="post" enctype="multipart/form-data">
       用户图片1：<input type="file" name="img"/></br>
       用户图片2：<input type="file" name="img"/></br>
       用户头像3：<input type="file" name="img"/></br>
       <input type="submit"/></br>
       ${msg}
   </form>
   </body>
   </html>
   ```

4. ```java
   @PostMapping("/uploadfiles")
   public String uploadFiles(@RequestParam(value = "img") MultipartFile[] files,
                             Model model) {
       for (MultipartFile file : files) {
           //文件非空执行
           if(!file.isEmpty()){
               try {
                   file.transferTo(new File("C:\\Users\\JM Wan\\Desktop\\" + file.getOriginalFilename()));
                   model.addAttribute("msg", "文件上传成功");
               } catch (IOException e) {
                   model.addAttribute("msg", "文件上传失败");
                   e.printStackTrace();
               }
           }
       }
       return "forward:/index.jsp";
   }
   ```

# 十三.拦截器

> 拦截器是一个接口，需要自定义实现

`HandlerInterceptor`接口

接口方法：

- preHandle()：在目标方法运行之前调用，返回true放行
- postHandle()：在目标方法运行之后调用（目标方法调用之后）
- afterCompletion()：在请求整个完成之后（来到目标页面之后）

细节流程：

- 只要preHandle()不放行，就没有后续流程
- 只要放行了afterCompletion()就会执行

## 拦截器配置

1. `index.jsp`

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>upload</title>
       <%--开启JSP EL表达式--%>
       <%@ page isELIgnored="false" %>
       <% pageContext.setAttribute("ctp", request.getContextPath()); %>
   </head>
   <body>
   <a href="${ctp}/test01">test01</a>
   </body>
   </html>
   ```

2. 实现拦截器接口

   ```java
   public class MyInterceptor implements HandlerInterceptor {
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           System.out.println("preHandle运行");
           return true;
       }
   
       @Override
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
           System.out.println("postHandle运行");
       }
   
       @Override
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
           System.out.println("afterCompletion运行");
       }
   }
   ```

3. 配置`springmvc.xml`注册拦截器

   ```xml
   <!--配置拦截器-->
   <mvc:interceptors>
       <!--配置某个拦截器，默认是拦截所有请求-->
       <bean class="com.mvc.Interceptor.MyInterceptor"></bean>
       <!--具体配置拦截器，只拦截test01请求-->
       <mvc:interceptor>
           <mvc:mapping path="/test01"/>
           <bean class="com.mvc.Interceptor.MyInterceptor"></bean>
       </mvc:interceptor>
   </mvc:interceptors>
   ```

4. 配置Controller

   ```java
   @GetMapping("/test01")
   public String test01(){
       System.out.println("方法运行");
       return "success";
   }
   ```

# 十四.异常处理

## 异常解析器

- ExceptionHandlerExceptionResolver：处理`@ExceptionHandler`
- ResponseStatusExceptionResovler：处理`@ResponseStatus`
- DefaultHandlerExceptionResolver：处理SpringMVC自定义的异常

## @ExceptionHandler

- 全局异常处理放在`@ControllerAdvice`注解的类下
- 局部异常处理则将异常处理放在对应Controller下
- **当局部和全局异常处理同时存在的时候，局部优先**

```java
/**
 * 异常类的全局处理需要加到ioc容器中
 * @ControllerAdvice标注的类专门用来处理异常
 */
@ControllerAdvice
public class ExceptionController {
    /**
     * 告诉SpringMVC用这个来处理数学和空指针异常
     * 用exception接收异常信息
     *
     * @param exception
     * @return
     */
    @ExceptionHandler(value = {ArithmeticException.class, NullPointerException.class})
    public String handleException(Exception exception) {
        System.out.println(exception.getMessage());
        //视图解析拼串
        return "error";
    }

    /**
     * 处理所有异常，SpringMVC异常处理会以精确优先
     *
     * @param exception
     * @return
     */
    @ExceptionHandler(value = Exception.class)
    public String globalException(Exception exception) {
        System.out.println(exception.getMessage());
        return "error";
    }
}
```

## @ResponseStatus

> 标注在自定义异常类上

```java
@ResponseStatus(reason = "用户拒绝登陆",value = HttpStatus.NOT_ACCEPTABLE)
public class UserNameNotFoundException extends RuntimeException{
}
```

直接抛出`UserNameNotFoundException`异常即可显示tomcat风格异常界面

# 十五.SpringMVC运行流程

1. 所有请求，前端控制器（DispatcherServlet）收到请求，调用doDispatch()处理
2. 根据HandlerMapping中保存的映射信息找到处理当前请求的处理器执行链（拦截器和处理类的方法）
3. 根据当前处理器找到他的HandlerAdaper（适配器）
4. 拦截器的perHandle先执行
5. 适配器执行目标方法，返回ModelAndView
   1. ModelAttribute注解标注方法提前先运行
   2. 执行目标方法的时候确认目标方法的参数
      1. 有注解
      2. 没注解
         1. 是否有Model、Map
         2. 如果是自定义类型
            1. 从隐含模型中看
               1. 如果有从模型中拿
               2. 如果没有，再看SessionAttributes标注的属性，如果从Session中拿，拿不到就会抛出异常
               3. 都不是就利用反射创建
6. 拦截器的postHandle执行
7. 处理结果
   1. 如果有异常，用异常解析器处理异常，处理完返回ModelAndView
   2. 调用render进行页面渲染
      1. 视图解析器根据视图名得到视图对象
      2. 视图对象调用render方法渲染
   3. 执行拦截器的afterComplate

# 十六.整合Spring

> Spring管理业务逻辑组件
>
> SpringMVC管理控制器组件

- 方法一

  在`SpringMVC.xml`中使用

  ```xml
  <import resource="spring.xml"/>
  ```

  合并Spring配置文件

- 方法二

  Spring和SpringMVC分容器

  1. 在`web.xml`中加入

     ```xml
     <context-parm>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:spring.xml</param-value>
     </context-parm>
     ```

     指定Spring配置文件

  2. SpringMVC配置文件

     ```xml
     <!--只扫描Controller和ControllerAdvice组件-->
     <context:component-scan base-package="com.mvc" use-default-filters="false">
         <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
         <context:include-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
     </context:component-scan>
     ```

  3. Spring配置文件

     ```xml
     <!--除了Controller和ControllerAdvice组件其他都扫描，use-default-filters这里不需要更改了-->
     <context:component-scan base-package="com.mvc">
         <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
         <context:exclude-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
     </context:component-scan>
     ```

     

  

