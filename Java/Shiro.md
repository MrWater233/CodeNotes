# 权限控制

## 用户认证

即用户的身份验证

## 用户授权

简单理解为访问控制，在用户通过认证后系统对用户访问资源进行控制。

# 快速开始

1. 导入依赖

2. 配置文件

3. Hello world

   ```java
   //获得当前用户对象
   Subject currentUser = SecurityUtils.getSubject();
   //通过当前用户拿到session
   Session session = currentUser.getSession();
   //判断当前用户是否被认证
   currentUser.isAuthenticated();
   //用户登陆
   currentUser.login(token);
   //获得当前用户的认证
   currentUser.getPrincipal();
   //判断用户是否拥有"schwartz"这个角色
   currentUser.hasRole("schwartz");
   //判断用户是否有"lightsaber:wield"这个权限
   currentUser.isPermitted("lightsaber:wield");
   //用户注销
   currentUser.logout();
   ```

# 关键对象

## 认证

- Subject---主题，理解为用户，系统需要对subject进行认证
- principal：身份信息，通常是唯一的，一个主体由多个身份信息，但是都有一个主身份信息（primary principal）
- credential：凭证信息，可以实密码、证书、指纹

总结：主体在身份认真需要身份和凭证信息

## 授权

过程：who对what做how操作

- who：主体，即Subject，Subject在认证通过后系统进行访问控制
- which：资源（resource），subject必须具备某些资源的权限才能访问。资源比如：系统用户的列表页面
  - 资源类型：系统用户信息
  - 资源实例：id为001的用户
- how：权限/许可（permission），针对资源的权限或许可，Subject具有permission访问资源，如何访问，操作需要定义permission权限比如哟关乎添加、修改、删除

- SecurityManager---管理所有用户
- Realm---连接数据

### 权限模型

- 用户

- 权限
- 角色
- 角色和权限的关系
- 用户和角色的关系

# 环境配置

## 导入依赖

```xml
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-spring-boot-starter</artifactId>
  <version>1.4.2</version>
</dependency>
```

# Shiro使用

## 框架

### 创建Realm

UserRealm.java：

```java
//自定义的realm
public class UserRealm extends AuthorizingRealm {
    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行了->授权doGetAuthorizationInfo");
        return null;
    }
    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行了->认证doGetAuthorizationInfo");
        return null;
    }
}
```

### 配置ShrioConfig

ShrioConfig.java：

```java
@Configuration
public class ShiroConfig {
    //3.ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        //设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);
        return bean;
    }


    //2.DefaultWebSecurityManager
    @Bean(name = "securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm){//将realm对象从ioc中获取，并指定获取的bean名
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联UserRealm
        securityManager.setRealm(userRealm);
        return securityManager;
    }


    //1.创建 realm对象，需要自定义
    @Bean
    public UserRealm userRealm(){
        return new UserRealm();
    }
}
```

## 设置拦截

| 名称  | 作用                         |
| ----- | ---------------------------- |
| anon  | 无需认证就能访问             |
| authc | 必须认证（登陆）才能访问     |
| user  | 必须拥有"记住我"功能才能用   |
| perms | 拥有对某个资源的权限才能访问 |
| role  | 拥有某个角色权限才能访问     |

```java
@Bean
public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager){
    ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
    //设置安全管理器
    bean.setSecurityManager(defaultWebSecurityManager);
    //登陆拦截
    Map<String, String> filterMap = new LinkedHashMap<>();
    //设置需要拦截的位置
    filterMap.put("/user/add","authc");
    filterMap.put("/user/update","authc");
    //如果验证不通过，设置登陆页面
    bean.setLoginUrl("/toLogin");
    //添加需要拦截的页面
    bean.setFilterChainDefinitionMap(filterMap);
    return bean;
}
```

## 设置授权

ShiroConfig：

```java
@Bean
public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager){
    ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
    //设置安全管理器
    bean.setSecurityManager(defaultWebSecurityManager);
    //登陆拦截
    Map<String, String> filterMap = new LinkedHashMap<>();
    //设置进入/user/add需要user:add这个权限，正常情况下，没有授权会跳转到401
    filterMap.put("/user/add","perms[user:add]");
    //设置需要拦截的位置
    filterMap.put("/user/*","authc");
    //添加需要拦截的页面
    bean.setFilterChainDefinitionMap(filterMap);
    //如果验证不通过，设置登陆页面
    bean.setLoginUrl("/toLogin");
    //如果没有被授权，那么跳转到固定的自定义页面
    bean.setUnauthorizedUrl("/unauth");
    return bean;
}
```

UserRealm：

```java
//自定义的realm
public class UserRealm extends AuthorizingRealm {    
	//授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行了->授权doGetAuthorizationInfo");
        //设置简单授权
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        //拿到当前登录的这个对象
        Subject subject = SecurityUtils.getSubject();
        //拿到User对象
        User cerrentUser = (User)subject.getPrincipal();
        //从用户中读取从数据库中去除的权限并设置为当前用户权限
        info.addStringPermission(cerrentUser.getPerms());
        return info;
    }
	//认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行了->认证doGetAuthorizationInfo");
        UsernamePasswordToken userToken = (UsernamePasswordToken) authenticationToken;
        //userSerice为自动注入的业务层
        User user = userSerice.queryUserByName(userToken.getUsername());
        //用户名认证
        if(user==null){
            return null;//抛出异常(UnknownAccountException)
        }
        //简单的密码认证，shiro自动执行
        //可以加密:MD5和MD5盐值加密
        return new SimpleAuthenticationInfo(user,user.getPwd,"");//将对象保存进subject中，便于授权获取
    }
}
```

## 设置用户的认证

<font color=red>Shiro自动将login的相关操作跟认证相互关联</font>

### 未集成数据库

UserRealm：

```java
//认证
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
    //设置用户名，密码
    String name = "root";
    String password = "123456";
    UsernamePasswordToken userToken = (UsernamePasswordToken) authenticationToken;
    //用户名认证
    if(!userToken.getUsername().equals(name)){
        return null;//抛出异常(UnknownAccountException)
    }
    //密码认证，shiro自动执行
    return new SimpleAuthenticationInfo("",password,"");
}
```

Controller：

```java
@GetMapping("/login")
public String login(String username,String password,Model model){
    //获取当前的用户
    Subject subject = SecurityUtils.getSubject();
    //封装用户的登陆数据
    UsernamePasswordToken token = new UsernamePasswordToken(username, password);
    //执行用户的登陆并捕获异常
    try {
        subject.login(token);
        return "index";
    }catch (UnknownAccountException e){//用户名不存在
        model.addAttribute("msg","用户名错误");
        return "login";
    }catch (IncorrectCredentialsException e){//密码不存在
        model.addAttribute("msg","密码错误");
        return "login";
    }
}
```

### 集成Mybatis数据库

```java
//认证
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
    System.out.println("执行了->认证doGetAuthorizationInfo");
    UsernamePasswordToken userToken = (UsernamePasswordToken) authenticationToken;
    //userSerice为自动注入的业务层
    User user = userSerice.queryUserByName(userToken.getUsername());
    //用户名认证
    if(user==null){
        return null;//抛出异常(UnknownAccountException)
    }
    //简单的密码认证，shiro自动执行
    //可以加密:MD5和MD5盐值加密
    return new SimpleAuthenticationInfo("",user.getPwd,"");
}
```

Controller：

```java
@GetMapping("/login")
public String login(String username,String password,Model model){
    //获取当前的用户
    Subject subject = SecurityUtils.getSubject();
    //封装用户的登陆数据
    UsernamePasswordToken token = new UsernamePasswordToken(username, password);
    //执行用户的登陆并捕获异常
    try {
        subject.login(token);
        return "index";
    }catch (UnknownAccountException e){//用户名不存在
        model.addAttribute("msg","用户名错误");
        return "login";
    }catch (IncorrectCredentialsException e){//密码不存在
        model.addAttribute("msg","密码错误");
        return "login";
    }
}
```

