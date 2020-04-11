### 概念

一种接口规范，只要按照这种接口编写的代码都可以称为servlet。

### 使用方法

1. 创建java对象，并继承HttpServlet
2. 覆写service方法
3. 在service方法中书写逻辑代码
4. 在WEB-INF下的`web.xml`中配置servlet

### 访问流程

1. 浏览器发起请求到服务器
2. 服务器接收浏览器请求，进行解析，创建request对象存储请求数据
3. 服务器调用对应servlet进行请求的处理，并将request对象作为实参传递给servlet方法
4. servlet的方法进行请求处理
   - 设置请求编码
   - 设置响应编码
   - 获取请求信息
   - 处理请求信息
     - 创建业务层对象
     - 调用业务层对象的发给发
   - 响应处理结果

### Servlet生命周期

- 如果在`web.xml`中配置了`<load-on-startup>启动顺序(1,2,3...)</load-on-startup>`则Servlet在服务器启动就调用，否则在使用的时候才调用
- `init()`在servlet第一次加载的时候被调用
- `service()`真正处理请求的时候被调用
- `destory()`在服务器关闭的时候被调用

### DoGet和DoPost方法

#### Service方法

能够处理Post和Get请求，如果servlet中有service方法，会优先调用service方法

#### DoGet方法

处理get请求方法

#### DoPost方法

处理post请求方法

#### 注意

- 如果在覆写service方法的时候调用父类的service的方法，则会在service方法执行完以后调用相应的get/post方法

- 一般不在service中调用父类的service方法， 避免发生405错误，因为调用父类的service方法就必须重写DoGet和DoPost方法

### Request对象

#### 作用

封存了当前请求的所有请求信息

#### 使用

- 获取请求行数据
- 获取请求头数据
- 获取用户数据

##### 获取请求行数据

| 方法                           | 作用         |
| ------------------------------ | ------------ |
| `String getMethod()`           | 获取请求方式 |
| `StringBuffer getRequestURL()` | 获取请求URL  |
| `String getRequestURI()`       | 获取请求URI  |
| `String getScheme()`           | 获取请求协议 |

##### 获取请求头数据

| 方法                           | 作用                  |
| ------------------------------ | --------------------- |
| `String getHeader(String s)`   | 通过指定键s来或获取值 |
| `Enumeration getHeaderNames()` | 获取所有的键的枚举    |

`enumeration`配合`hasMoreElements()`和`nextElement()`来使用。

##### 获取用户数据

| 方法                                    | 作用                             |
| --------------------------------------- | -------------------------------- |
| `String getParameter(String s)`         | 通过键s来获取一个值              |
| `String[] getParameterValues(String s)` | 获取桶一个键的不同value数组      |
| `Enumeration getParameterNames()`       | 返回所有用户请求数据键的枚举集合 |

虽然`get`和`post`的用户数据放置位置不同，但是通过这个函数都可以获取到用户的数据 。

<font color="red">如果获取的请求数据不存在，不会报错，返回null</font>

#### 注意

request对象由tomcat服务器创建并作为实参传递给处理请求的servlet的service方法。

### Response对象

#### 作用

用来响应数据到浏览器的一个对象

#### 使用

- 设置响应头
- 设置响应实体

##### 相应行数据处理

| 方法                     | 作用         |
| ------------------------ | ------------ |
| `sendError(状态码,提示)` | 设置响应状态 |

##### 响应头数据处理

| 方法                                                  | 作用                                   |
| ----------------------------------------------------- | -------------------------------------- |
| `addHeader(String key,String value)`                  | 在响应头中添加响应信息，但是不会覆盖   |
| `setHeader(String key,String value)`                  | 在响应头中添加响应信息，但是同键会覆盖 |
| `setHeader("content-type","text/html;charset=utf-8")` | 设置响应的编码方法1                    |
| `setContentType("text/html;charset=utf-8")`           | 设置响应的编码方法2                    |

在设置编码方式的时候同时告诉浏览器内容有`html`标签，浏览器就会自动解析`html`标签，如果把`html`标签换成`plain`，那么浏览器就会把返回数据当成普通数据处理。

##### 响应实体数据处理

| 方法                          | 作用         |
| ----------------------------- | ------------ |
| `getWriter().write(String s)` | 设置响应实体 |

### service请求处理代码流程

1. 设置响应编码格式
2. 获取请求数据
3. 处理请求数据（数据库操作，MVC思想）
4. 响应处理结果

### MVC设计模式

请求：View->control->model(Dao层)

响应：model->control->view

### 设置编码格式

浏览器默认编码格式为ios8859-1，而tomcat在8以前默认编码utf-8，所以会出现乱码。

这时候需要使用以下的方法转换req的编码格式：

- 使用String进行重新编码（通用）`new String(uname.getBytes("ios8859-1"),"utf-8")`
- Post设置请求编码格式`req.setCharacterEncoding("utf-8")`
- Get请求需要配置server.xml并加上↑这个编码语句

### 请求转发

#### 作用

实现多个servlet联动操作请求，避免了代码冗余，让servlet的职责更加明确

#### 使用

`req.getRequestDispatcher("需要转发的Servlet的别名").forward(req,resp);`

#### 特点

一次请求，浏览器地址栏不改变

#### 注意

请求转发以后直接`return`即可

<font color="red">因为请求转发以后url地址不变，所以刷新一次页面数据就会重新提交一次，所以请求转发应当在允许表单数据可以重复提交的时候使用</font>

### Request数据流转

#### 作用

解决了**一次请求**内的不同Servlet的数据（请求数据+其他数据）共享问题、

#### 作用域

基于请求转发，**一次请求**中的所有Servlet共享。

#### 使用方法

- `request.setAttribute(Object key,Object value)`

  在`request`中增加数据

- `(String)request.getAttribute(Object key)`

  获取request中键对应的值，用来获取Servlet转发过来增加的数据。

  因为返回值为`object`，所以注意类型转换。

### 重定向

#### 使用环境

1. 当前数据Servlet无法处理，建议使用重定向定位到可以处理的资源。
2. 如果请求中有表单数据，而数据又比较重要，不能重复提交。

#### 使用

`response.sendRedirect("路径/别名")`

本地路径为：URI

网络路径为：定向资源的URL信息

#### 特点

- 两次请求（让浏览器重新发送一次请求去访问重定向的地址），所以不能像请求转发那样进行request传递。
- 浏览器地址改变

### Cookie

#### 作用

解决了发送请求的数据共享问题

#### 使用

##### Cookie创建和存储

| 方法                                                  | 作用                             |
| ----------------------------------------------------- | -------------------------------- |
| `Cookie cookie = new Cookie(String Key,String Value)` | 使用键值对的方式创建Cookie对象   |
| `cookie.setMaxAge(int second)`                        | 设置Cookie有效期，单位为秒       |
| `cookie.setPath("/path")`                             | 设置在请求中发送Cookie的有效路径 |
| `response.addCookie(Cookie cookie)`                   | 添加Cookie信息到响应             |

##### Cookie的获取

| 方法                   | 作用                                        |
| ---------------------- | ------------------------------------------- |
| `request.getCookies()` | 返回一个<font color="red">Cookie数组</font> |
| `cookie.getName()`     | 获取Cookie的键                              |
| `cookie.getValue()`    | 获取Cookie的值                              |

#### 特点

浏览器端的数据存储技术

存储数据声明在服务器端

默认cookie信息存储好之后，每次请求都会附带，除非设置有效路径

##### 临时存储

存储在浏览器的运行内存中，浏览器关闭即失效

##### 定时存储

设置了Cookie的有效期，存储在浏览器客户端的硬盘中，在有效期内符合路径要求的请求都会附带该信息

#### 注意

一个Cookie对象存储一条数据

多条数据可以创建多个Cookie对象进行存储

<font color="red">在获取Cookie数组的时候的时候应该判断请求中是否含有Cookie</font>

### Session

#### 作用

解决一个用户不同请求的数据共享问题

#### 原理

用户在第一次访问服务器，服务器会创建一个session对象给此用户，并将session对象的JSEEIONID使用Cookie技术存储带浏览器中，保证用户的其他请求去能够获取到同一个session对象，也保证了不同请求能够获取到共享数据。

#### 特点

- 存储在服务器端

- 由服务器进行创建

- 依赖Cookie的临时存储技术
- 一次会话
- 默认存储时间为30分钟

#### 使用

| 方法                                     | 作用                  |
| ---------------------------------------- | --------------------- |
| `HttpSession hs = request.getSession()`  | 创建/获取Session对象  |
| `hs.setMaxInactiveInterval(int seconds)` | 设置Session的存储时间 |
| `hs.invalidate()`                        | 设置Session强制失效   |
| `hs.setAttribute(String s,Object o)`     | 存储Session中的键值   |
| `hs.getAttribute(String s)`              | 返回Session的值       |

<font color="red">第一个方法：</font>

- 如果请求中拥有session的JSESSIONID，则返回对应的session对象

- 如果请求中没有session的JSESSIONID，则创建新的session对象，并将其JSESSIONID作为Cookie数据存储到浏览器的内存中

- 如果session对象失效了，也会重新创建session对象，并将JSESSIONID存储在浏览器内存中

<font color="red">第二个方法：</font>

- 在指定的时间内session对象没有被使用则销毁，如果被使用了则重新计时。

#### 使用时机

一般用户在登陆web项目时，会将用户的个人信息存储到Session中，供用户的其他请求使用。

#### Session失效处理

1. 在创建session的时候做一个flag标记，对标记进行判断来获取session是否过期（<font color="red">不会实现，待处理，是否是加一个时间戳？</font>）

2. 根据`if(session.getAttribute('user')==null)`判断是否为空

3. `request.getSession(boolean)`

   - 若参数为`true`，那么如果当前request的session不可用，那么就会创建新的会话，如果存在就会返回当前会话。

   - 若参数`false`，那么在当前会话不存在的时候返回`null`

#### 注意

- JSESSIONID存储在了Cookie的临时存储空间中，浏览器关闭即失效。

- 存储和取出可以发生在不同的请求中，但是存储要先于取出执行。

### ServletContext对象

#### 作用

- 解决了不同用户的数据共享问题

- 获取web.xml文件中的全局配置

  在web.xml写入`<context-param></context-param>`并设置里面的键值对，一组标签申明一组键值对

  `<param-name></param-name>`

  `<param-value></param-value>`

#### 生命周期

服务器启动到服务器关闭

#### 特点

- 服务器创建
- 用户共享

#### 使用

##### 获取ServletContext对象

| 方法                                                         | 作用  |
| ------------------------------------------------------------ | ----- |
| `ServletContext sc = this.getServletContext()`               | 方法1 |
| `ServletContext sc = this.getServletConfig().getServletContext()` | 方法2 |
| `ServletContext sc = req.getSession().getServletContext()`   | 方法3 |

##### 使用ServletContext对象完成数据共享

| 方法                                 | 作用                       |
| ------------------------------------ | -------------------------- |
| `sc.setAttribute(String s,Object o)` | 数据存储                   |
| `sc.getAttribute(String s)`          | 数据获取，返回`Object`类型 |

##### 获取web.xml中的全局配置

| 方法                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `sc.getInitParameter(String name)` | 根据键的名字返回web.xml中配置的全局数据的值，返回String，如果不存在，返回null |
| `sc.getInitParameterNames()`       | 返回键名的枚举                                               |

###### 作用

1. 将静态代码跟数据解耦
2. 获取项目webroot下的资源的绝对路径(`sc.getRealPath(web下的具体路径)`)

#### 注意

- 不同的用户可以给ServletContext对象进行数据的存取
- 获取的数据不存在返回null

### ServletConfig

#### 作用

ServletConfig对象是Servlet的专属配置对象，每个Servlet都有一个ServletConfig对象，用来获取web.xml中的配置信息。

#### 使用

| 方法                               | 作用                       |
| ---------------------------------- | -------------------------- |
| `this.getServletConfig()`          | 获取ServletConfig对象      |
| `sc.getInitParameter(String name)` | 获取专属配置，返回`String` |

在`<servlet>`标签下配置`<init-param></init-param>`里面设置键值对

`<param-name></param-name>`

`<param-value></param-value>`

