# 一.Shiro的三大组件

- Subject：直接用用户使用，使用简单，底层操作SecurityManager
- SecurityManager：核心，执行Shiro的各种操作（除了加密）
- Realm：类似Dao层，由核心调用，提供用户的认证和授权信息

# 二.权限规则

| 符号 | 说明                                      |
| ---- | ----------------------------------------- |
| `:`  | 分隔符，用于分隔资源和操作【`资源:操作`】 |
| `,`  | 用于分隔多个权限【`权限1,权限2,权限3`】   |
| `*`  | 通配符，代表所有操作、资源【`资源:*`】    |

## 细节

- `user:*`可以匹配`user:xx:xxx`

  `*:query`不能匹配 `xxx:xx:query`

- `user:update,user:insert`可以简写为 `"user:update,insert"`（<font color=red>注意引号</font>）

## 实例级别的权限

> `资源:操作:实例`

如`user:delete:1`：删除用户1

# 三.过滤规则

过滤规则=路径规则+权限规则

<font color=blue>URL匹配是从上到下寻找过滤规则，找到就停止，所以应当遵循范围由小到大从上到下配置</font>

## 路径规则

- `/user/*`：代表`/user`后面还有一级任意路径
- `/user/**`： 代表`/user`后面还有任意多级任意路径
- `/user/hello?`：代表hello后面还有任意一个字符，如`/user/helloa`

## 权限规则

| 名称         | 作用                                                   |
| ------------ | ------------------------------------------------------ |
| anon（匿名） | 无需认证就能访问                                       |
| authc        | 必须认证（登陆）才能访问                               |
| user         | 必须拥有"记住我"功能才能用                             |
| perms        | 拥有对某个资源的权限才能访问，如`perms["user:update"]` |
| roles        | 拥有某个角色权限才能访问，如`roles["admin"]`           |
| logout       | 登出，不用定义handler                                  |

# 四.Shiro标签

> 通过Shiro完成对前端页面元素的访问控制

## 配置

1. 依赖

   ```xml
   <dependency>
       <groupId>com.github.theborakompanioni</groupId>
       <artifactId>thymeleaf-extras-shiro</artifactId>
       <version>2.0.0</version>
   </dependency>
   ```

2. 配置Bean

   ```java
   @Configuration
   public class ShrioConfig {
       @Bean
       public ShiroDialect shiroDialect() {
           return new ShiroDialect();
       }
   }
   ```

3. 导入Shiro标签

   ```html
   <!DOCTYPE html>
   <html xmlns:th="http://www.thymeleaf.org" 
         xmlns:shiro="http://www.pollix.at/thymeleaf/shiro" >
   ```

## 使用

### 身份认证

| 标签                       | 作用                       |
| -------------------------- | -------------------------- |
| `<shrio:authenticated>`    | 已登录                     |
| `<shiro:user>`             | 已登录 或 记住我           |
| `<shiro:guest>`            | 游客（未登录 且 未记住我） |
| `<shiro:notAuthenticated>` | 未登录                     |
| `<shiro:principal/>`       | 获取用户身份信息           |

### 角色校验

| 标签                                       | 作用               |
| ------------------------------------------ | ------------------ |
| `<shiro:hasAnyRoles name="admin,manager">` | 是其中任何一个角色 |
| `<shiro:hasRole name="admin">`             | 是指定角色         |
| `<shiro:lacksRole name="admin">`           | 不是指定角色       |

### 权限校验

| 标签                                         | 作用         |
| -------------------------------------------- | ------------ |
| `<shiro:hasPermission name="user:delete">`   | 有指定权限   |
| `<shiro:lacksPermission name="user:delete">` | 没有指定权限 |

# 五.自定义Realm

## 表的创建

一共需要五张表来建立**用户---角色---权限**的关系

![images](https://raw.githubusercontent.com/MrWater233/PictureHost/master/%E7%94%A8%E6%88%B7%E8%A7%92%E8%89%B2%E6%9D%83%E9%99%90%E8%A1%A8.png)

## 授权和认证框架

> 自定义Realm需要继承父类
>
> 有两个父类可以选择：
>
> 1. 如果只做身份验证：AuthenticatingRealm
> 2. 身份验证+权限校验：AuthorizingRealm

```java
public class RealmConfig extends AuthorizingRealm {
    /**
     * 作用：查询权限信息
     * 触发时刻：1.过滤URL时  2.使用shiro标签时
     * 查询方式：通过用户名查询用户权限角色信息
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //获取用户名
        String username = (String)principalCollection.getPrimaryPrincipal();
        //查询角色和权限信息 RoleService PermissionService
        return null;
    }

    /**
     * 作用：查询身份信息，将身份信息返回即可
     * 查询方式：通过用户名（token）查询用户信息
     *         （UsernamePasswordToken是AuthenticationToken子类）
     * 触发时刻：调用subject.login(token)时，底层由securitymanage调用
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //获取用户登陆时的用户名
        String username = (String)authenticationToken.getPrincipal();
        //查询用户信息 UserService
        return null;
    }
}
```

## Dao层的处理

因为用户---角色---权限表相互关联，所以需要用到[内连接](com.mysql.cj.jdbc.Driver)的多表查询

如下为最复杂的一个权限查询sql

<font color=green>PS：注意内连接可能产生重复（因为角色之间的权限可能重复），需要`DISTINCT`去重</font>

```sql
SELECT DISTINCT t_permission.permission_name FROM t_user
JOIN t_user_role ON t_user.id = t_user_role.user_id
JOIN t_role ON t_role.id = t_user_role.role_id
JOIN t_role_permission ON t_role_permission.role_id = t_role.id
JOIN t_permission ON t_role_permission.permission_id = t_permission.id
WHERE t_user.username = #{username}
```

## Realm的完整配置

```java
public class RealmConfig extends AuthorizingRealm {
    @Autowired
    UserService userService;
    @Autowired
    RoleService roleService;
    @Autowired
    PermissionService permissionService;
    /**
     * 作用：查询权限信息
     * 触发时刻：1.过滤URL时  2.使用shiro标签时
     * 查询方式：通过用户名查询用户权限角色信息
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //获取用户名
        String username = (String)principalCollection.getPrimaryPrincipal();
        //查询角色信息，并转为set
        List<String> roles_list = roleService.queryAllRolenameByUsername(username);
        Set<String> roles = new HashSet<>(roles_list);
        //查询权限信息，并转为set
        List<String> perms_list = permissionService.queryAllPermissionByUsername(username);
        Set<String> perms = new HashSet<>(perms_list);
        //封装对象
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo(roles);
        simpleAuthorizationInfo.setStringPermissions(perms);
        return simpleAuthorizationInfo;
    }

    /**
     * 作用：查询身份信息，将身份信息返回即可
     * 查询方式：通过用户名（token）查询用户信息
     *         （UsernamePasswordToken是AuthenticationToken子类）
     * 触发时刻：调用subject.login(token)时，底层由securitymanage调用
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //获取用户登陆时的用户名
        String username = (String)authenticationToken.getPrincipal();
        //查询用户信息
        User user = userService.queryUserByUsername(username);
        //判断用户信息是否为空
        if(user==null){
            //在后续流程中会抛出异常 UnknownAccountException
            return null;
        }
        /**
         * 将用户信息封装，后续会将token和返回值进行比对
         * 参数：1.数据库中用户名 2.数据库中密码 3.当前Realm的标识（类名+_+内部封装的某一个值）
         */
        return new SimpleAuthenticationInfo(user.getUsername(), user.getPassword(), getName());
    }
}
```

# 六.ShrioConfig完整配置

```java
@Configuration
public class ShrioConfig {
    /**
     * 配置Shrio的拦截器
     * @param securityManager
     * @return
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilter(@Qualifier("securityManager") DefaultWebSecurityManager securityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean  = new ShiroFilterFactoryBean();
        // 必须设置 SecurityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        //拦截器，配置拦截链，请求会逐条判断
        Map<String,String> filterChainDefinitionMap = new LinkedHashMap<String,String>();
        //配置退出过滤器,其中的具体的退出代码Shiro已经替我们实现了，只需要访问/logout即可退出登录
        filterChainDefinitionMap.put("/logout", "logout");
        //过滤链定义，从上向下顺序执行，一般将 /**放在最为下边;
        filterChainDefinitionMap.put("/**", "anon");
        //配置登录的URL
        shiroFilterFactoryBean.setLoginUrl("/user/login");
        //登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/user/index");
        //没有授权跳转的URL
        shiroFilterFactoryBean.setUnauthorizedUrl("/user/login");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    /**
     * 注入Shiro的安全管理器
     * @param realmConfig
     * @return
     */
    @Bean
    public DefaultWebSecurityManager securityManager(@Qualifier("realm") RealmConfig realmConfig){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(realmConfig);
        return securityManager;
    }

    /**
     * 注入自定义的Realm
     * @return
     */
    @Bean
    public RealmConfig realm(){
        return new RealmConfig();
    }

    /**
     * 注入Shiro标签需要的配置
     * @return
     */
    @Bean
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }
}
```

# 七.加密

> 为了安全考虑，一般数据库中存入用户**不可逆**的加密密码

## 加密方式

1. 基本加密：

   直接用md5、sha加密

2. 加盐加密：

   密码+盐值进行md5、sha加密

3. 加盐多次迭代：

   反复进行加盐和md5、sha加密

建议使用加盐迭代加密，迭代次数1000+

## 加密代码

```java
/**
 * password:密码
 * salt:盐值
 * iter:迭代次数
 * toString默认转换为16进制返回
 * toBase64压缩后更短
 */
String pwd = new Md5Hash(password,salt,iter).toString;//md5加密
String pwd = new Md5Hash(password,salt,iter).toBase64;//加密转base64

String pwd = new Sha256Hash(password,salt,iter).toString;//sha256加密
String pwd = new Sha256Hash(password,salt,iter).toBase64;//加密转base64

String pwd = new Sha512Hash(password,salt,iter).toString;//sha512加密
String pwd = new Sha512Hash(password,salt,iter).toBase64;//加密转base64
```

## SpringBoot应用

在用户表中增加salt字段来存储随机生成的盐

### 注册

在Service层实现加盐的迭代加密处理

```java
public Integer insertUser(User user) {
    //获取随机盐
    String salt = UUID.randomUUID().toString();
    //加密
    //参数：1.密码 2.盐值 3.迭代次数
    String pwd = new Sha256Hash(user.getPassword(), salt, 1000).toBase64();
    //设置密文
    user.setPassword(pwd);
    //设置盐
    user.setSalt(salt);
    return userDao.insertUser(user);
}
```

### 登录

1. Realm的认证中增加盐值返回

   ```java
   /**
    * 作用：查询身份信息，将身份信息返回即可
    * 查询方式：通过用户名（token）查询用户信息
    * （UsernamePasswordToken是AuthenticationToken子类）
    * 触发时刻：调用subject.login(token)时，底层由securitymanage调用
    * @param authenticationToken
    * @return
    * @throws AuthenticationException
    */
   @Override
   protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
       System.out.println("认证");
       //获取用户登陆时的用户名
       String username = (String) authenticationToken.getPrincipal();
       //查询用户信息
       User user = userService.queryUserByUsername(username);
       //判断用户信息是否为空
       if (user == null) {
           //在后续流程中会抛出异常 UnknownAccountException
           return null;
       }
       /**
        * 将用户信息封装，后续会将token和返回值进行比对
        * 参数：1.数据库中用户名 2.数据库中密码  3.盐值(转换为ByteSource类型) 4.当前Realm的标识（类名+_+内部封装的某一个值）
        */
       return new SimpleAuthenticationInfo(user.getUsername(), user.getPassword(), ByteSource.Util.bytes(user.getSalt()), getName());
   }
   ```

2. ShiroConfig中设置密码校验规则并交给Realm

   ```java
   /**
    * 注入自定义的Realm
    * @return
    */
   @Bean
   public RealmConfig realm(@Qualifier("hashedCredentialsMatcher") HashedCredentialsMatcher matcher){
       RealmConfig myRealm = new RealmConfig();
       //设置密码校验方式，这里选择我们设置的哈希加密
       myRealm.setCredentialsMatcher(matcher);
       return myRealm;
   }
   
   /**
    * 密码校验规则HashedCredentialsMatcher
    * 防止密码在数据库里明码保存 , 当然在登陆认证的时候 ,
    * 这个类也负责对form里输入的密码进行编码
    * 处理认证匹配处理器：如果自定义需要实现继承HashedCredentialsMatcher
    * 需要将这个类注册给Realm，间接关联给SecurityManager
    */
   @Bean("hashedCredentialsMatcher")
   public HashedCredentialsMatcher hashedCredentialsMatcher() {
       //声明使用哈希密码校验器
       HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
       //指定加密方式为SHA-256
       credentialsMatcher.setHashAlgorithmName("sha-256");
       //指定迭代次数
       credentialsMatcher.setHashIterations(1000);
       //true=16进制格式(toString) false=base64格式
       credentialsMatcher.setStoredCredentialsHexEncoded(false);
       return credentialsMatcher;
   }
   ```

#  八.记住我

> 登陆后将用户信息保存在Cookie中，下次访问不用登陆就可以识别身份

## 开启

> 在登陆之前开启即可，默认记住一年

```java
//获取subject，调用login
Subject subject = SecurityUtils.getSubject();
//创建登陆令牌
UsernamePasswordToken token = new UsernamePasswordToken(user.getUsername(),user.getPassword());
//开启记住我
token.setRememberMe(true);
//登陆，失败会抛出异常，交给异常解析器处理
subject.login(token);
```

## 自定义配置

[记住我的配置](https://juejin.im/post/5d3aa1a151882545be4b096a)

ShrioConfig配置如下（SimpleCookie、CookieRememberMeManager可以提取到其他类中注入，简化ShrioConfig的配置 ）：

```java
/**
 * 设置记住我Cookie的基本属性
 * @return
 */
@Bean
public SimpleCookie rememberMeCookie() {
    // 设置cookie名称，对应login.html页面的<input type="checkbox" name="rememberMe"/>
    SimpleCookie cookie = new SimpleCookie("rememberMe");
    // 设置cookie的过期时间，单位为秒，这里为60秒
    cookie.setMaxAge(60);
    return cookie;
}

/**
 * 记住我的管理器，用于注册给SecurityManager
 * @return
 */
@Bean
public CookieRememberMeManager rememberMeManager(@Qualifier("rememberMeCookie") SimpleCookie simpleCookie) {
    CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
    cookieRememberMeManager.setCookie(simpleCookie);
    // rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法回自动生成 密钥长度(128 256 512 位)
    //cookieRememberMeManager.setCipherKey(Base64.decode("3AvVhmFLUs0KTA3Kprsdag=="));
    return cookieRememberMeManager;
}

/**
 * 注入Shiro的安全管理器
 * @param realmConfig
 * @return
 */
@Bean
public DefaultWebSecurityManager securityManager(@Qualifier("realm") RealmConfig realmConfig,
                                                 @Qualifier("rememberMeManager") CookieRememberMeManager rememberMeManager){
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    //注册自定义的Realm
    securityManager.setRealm(realmConfig);
    //注册记住我管理器
    securityManager.setRememberMeManager(rememberMeManager);
    return securityManager;
}
```

# 九.Session管理

> 1.Shiro的Session方案与容器无关（如Servlet容器）
>
> 2.Shiro的Session可以定制存储位置（内存，缓存，数据库等）
>
> 3.用户认证也是通过Session来管理用户登陆状态
>
> 4.当配置了session以后，项目将默认使用shiro的session管理

## 自定义Session配置

```java
/**
 * 设置SessionCookie的基本属性
 * @return
 */
@Bean
public SimpleCookie sessionIdCookie() {
    // 设置cookie名称
    SimpleCookie cookie = new SimpleCookie("JSESSIONID");
    // 设置cookie的过期时间，-1为存活一个会话（浏览器关闭就消失），默认为-1
    cookie.setMaxAge(-1);
    return cookie;
}

/**
 * sessionmanager的基本配置
 */
@Bean
public DefaultWebSessionManager sessionManager(@Qualifier("sessionIdCookie") SimpleCookie simpleCookie){
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    //设置Cookie信息
    sessionManager.setSessionIdCookie(simpleCookie);
    //session全局超时时间，默认为30分钟，即1800000ms
    sessionManager.setGlobalSessionTimeout(1800000);
    return sessionManager;
}

/**
 * 注入Shiro的安全管理器
 * @param realmConfig
 * @return
 */
@Bean
public DefaultWebSecurityManager securityManager(@Qualifier("realm") RealmConfig realmConfig,
                                                 @Qualifier("sessionManager") DefaultWebSessionManager sessionManager){
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    //注册自定义的Realm
    securityManager.setRealm(realmConfig);
    //注册session管理器
    securityManager.setSessionManager(sessionManager);
    return securityManager;
}
```

## 检测

> 如果用户没有主动退出登陆，只是关闭浏览器，session是否过期无法获知，服务器也就不能停止session
>
> 只有等用户下次访问，服务器才会去停止session
>
> Shiro提供session的检测机制，识别session过期，并停止

```java
/**
 * sessionmanager的基本配置
 */
@Bean
public DefaultWebSessionManager sessionManager(@Qualifier("rememberMeCookie") SimpleCookie simpleCookie){
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    //开启session检测
    sessionManager.setSessionValidationSchedulerEnabled(true);
    //设置检测器运行间隔，默认1小时
    sessionManager.setSessionValidationInterval(3600000);
    return sessionManager;
}
```

# 十.Shiro注解开发

## 配置

在ShiroConfig中添加以下配置

```java
/**
 *  开启shiro aop注解支持.否则@RequiresRoles等注解无法生效
 *  在此Bean构建过程中初始化了一些额外功能和pointcut
 * @param securityManager
 * @return
 */
@Bean
public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(@Qualifier("securityManager") DefaultWebSecurityManager securityManager){
    AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
    authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
    return authorizationAttributeSourceAdvisor;
}

/**
 * Shiro生命周期处理器,管理Shrio的bean
 * 调用工厂中Initializable类型组件的init方法
 * @return
 */
@Bean
public static LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
    return new LifecycleBeanPostProcessor();
}

/**
 * Spring的一个bean , 由Advisor决定对哪些类的方法进行AOP代理
 * 扫描上下文，寻找所有的Advistor(通知器）
 * 必须在LifecycleBeanPostProcessor之后创建
 * @return
 */
@Bean
public static DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator(){
    return new DefaultAdvisorAutoProxyCreator();
}
```

## 使用

> 这些注解可以在类上也可以在方法上
>

| 注解                      | 作用                           |
| ------------------------- | ------------------------------ |
| `@RequiresAuthentication` | 类中方法需要身份验证           |
| `@RequiresRoles()`        | 类中方法需要角色，在参数中定制 |
| `@RequiresUser`           | 记住我 或 身份已经验证         |
| `@RequiresGuest`          | 游客身份                       |
| `@RequiresPermissions()`  | 需要权限，默认是且             |

