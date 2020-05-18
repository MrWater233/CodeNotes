# 一.依赖

```xml
<!--SpringSecurity-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>        
<!--JWT(Json Web Token)登录支持-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
<!--工具类，定义常见工具-->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>4.6.3</version>
</dependency>
<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<!--数据库-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1.tmp</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.22</version>
</dependency>
```



# 二.配置文件

```yaml
spring:
  datasource:
    username: #数据库用户名
    password: #数据库密码
    url: jdbc:mysql://localhost:3306/room?useUnicode=true&charactorEncoding=utf8&useSSL=false&zeroDateTimeBehavior=convertToNull&serverTimezone=GMT%2B8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

mybatis-plus:
  global-config:
    FieldStrategy: 2 #NULL和空字符串都不会更新
    db-config:
      id-type: auto
      logic-delete-field: deleted
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)

# 自定义jwt key
jwt:
  tokenHeader: Authorization #JWT存储的请求头
  secret: mySecret #JWT加解密使用的密钥
  expiration: 604800 #JWT的超期限时间(60*60*24)
  tokenHead: Bearer  #JWT负载中拿到开头
```

# 三.JWT Token工具类

`com.weixin.room.util.JwtTokenUtil`

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 19:42
 * @Version
 */
@Component
public class JwtTokenUtil {
    private static final Logger LOGGER = LoggerFactory.getLogger(JwtTokenUtil.class);
    private static final String CLAIM_KEY_USERNAME = "sub";
    private static final String CLAIM_KEY_CREATED = "created";
    @Value("${jwt.secret}")
    private String secret;
    @Value("${jwt.expiration}")
    private Long expiration;

    /**
     * 根据负责生成JWT的token
     */
    private String generateToken(Map<String, Object> claims) {
        return Jwts.builder()
                .setClaims(claims)
                .setExpiration(generateExpirationDate())
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }

    /**
     * 从token中获取JWT中的负载
     */
    private Claims getClaimsFromToken(String token) {
        Claims claims = null;
        try {
            claims = Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(token)
                    .getBody();
        } catch (Exception e) {
            LOGGER.info("JWT格式验证失败:{}",token);
        }
        return claims;
    }

    /**
     * 生成token的过期时间
     */
    private Date generateExpirationDate() {
        return new Date(System.currentTimeMillis() + expiration * 1000);
    }

    /**
     * 从token中获取登录用户名
     */
    public String getUserNameFromToken(String token) {
        String username;
        try {
            Claims claims = getClaimsFromToken(token);
            username =  claims.getSubject();
        } catch (Exception e) {
            username = null;
        }
        return username;
    }

    /**
     * 验证token是否还有效
     *
     * @param token       客户端传入的token
     * @param userDetails 从数据库中查询出来的用户信息
     */
    public boolean validateToken(String token, UserDetails userDetails) {
        String username = getUserNameFromToken(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    /**
     * 判断token是否已经失效
     */
    private boolean isTokenExpired(String token) {
        Date expiredDate = getExpiredDateFromToken(token);
        return expiredDate.before(new Date());
    }

    /**
     * 从token中获取过期时间
     */
    private Date getExpiredDateFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getExpiration();
    }

    /**
     * 根据用户信息生成token
     */
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put(CLAIM_KEY_USERNAME, userDetails.getUsername());
        claims.put(CLAIM_KEY_CREATED, new Date());
        return generateToken(claims);
    }

    /**
     * 判断token是否可以被刷新
     */
    public boolean canRefresh(String token) {
        return !isTokenExpired(token);
    }

    /**
     * 刷新token
     */
    public String refreshToken(String token) {
        Claims claims = getClaimsFromToken(token);
        claims.put(CLAIM_KEY_CREATED, new Date());
        return generateToken(claims);
    }
}
```

# 四.用户实体类

`com.weixin.room.entity.Base`

```java
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableLogic;
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import java.io.Serializable;
import java.util.Date;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 16:23
 * @Version
 */
@Data
public class Base implements Serializable {
    private static final long serialVersionUID = -7246930759870694809L;
    /**
     * ID
     */
    @TableId
    Long id;

    /**
     * 创建时间
     */
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    Date createTime;

    /**
     * 修改时间
     */
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    Date modifyTime;

    /**
     * 是否删除
     */
    @TableLogic
    Integer deleted;
}

```

`com.weixin.room.entity.User`

```java
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import java.util.Date;
import java.util.List;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 15:08
 * @Version
 */
@Data
@TableName("user")
public class User extends Base{
    private static final long serialVersionUID = 690239627648565115L;
    /**
     * 用户名
     */
    String username;
    /**
     * 密码
     */
    String password;
    /**
     * 角色
     */
    @TableField(exist = false)
    List<Role> roles;
}

```

# 五.SpringSecurity配置类

`com.weixin.room.config.WebSecurityConfig`

```java
import com.weixin.room.component.RestAuthenticationEntryPoint;
import com.weixin.room.component.RestfulAccessDeniedHandler;
import com.weixin.room.dto.AuthUserDetails;
import com.weixin.room.entity.Role;
import com.weixin.room.entity.User;
import com.weixin.room.filter.JwtAuthenticationTokenFilter;
import com.weixin.room.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import java.util.List;

/**
 * @author WanJingmiao
 * @Description SpringSecurity配置类
 * @date 2020/5/8 15:09
 * @Version
 */
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    UserService userService;
    @Autowired
    RestAuthenticationEntryPoint restAuthenticationEntryPoint;
    @Autowired
    RestfulAccessDeniedHandler restfulAccessDeniedHandler;

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.csrf()
                // 由于使用的是JWT，我们这里不需要csrf
                .disable()
                // 基于token，所以不需要session
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 允许对于网站静态资源的无授权访问
                .antMatchers(HttpMethod.GET,
                        "/",
                        "/*.html",
                        "/favicon.ico",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        "/swagger-ui.html/**",
                        "/swagger-resources/**",
                        "/v2/api-docs/**"
                )
                .permitAll()
                // 对登录注册要允许匿名访问
                .antMatchers("/user/login", "/user/register")
                .permitAll()
                //跨域请求会先进行一次options请求
                .antMatchers(HttpMethod.OPTIONS)
                .permitAll()
//                .antMatchers("/**")//测试时全部运行访问
//                .permitAll()
                .anyRequest()// 除上面外的所有请求全部需要鉴权认证
                .authenticated();
        // 禁用缓存
        httpSecurity.headers().cacheControl();
        // 在UsernamePasswordAuthenticationFilter前添加JWT filter
        httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        //添加自定义未授权和未登录结果返回
        httpSecurity.exceptionHandling()
                .accessDeniedHandler(restfulAccessDeniedHandler)
                .authenticationEntryPoint(restAuthenticationEntryPoint);
    }

    /**
     * 配置UserDetailsService及PasswordEncoder
     *
     * @param auth
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService())
                .passwordEncoder(passwordEncoder());
    }

    /**
     * 定义的用于对密码进行编码及比对的接口
     *
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 用于根据用户名获取用户信息
     *
     * @return
     */
    @Bean
    @Override
    public UserDetailsService userDetailsService() {
        //获取登录用户信息
        return username -> {
            User user = userService.getUserByUsername(username);
            if (user != null) {
                List<Role> roleList = userService.getPermissions(user.getId());
                return new AuthUserDetails(user, roleList);
            }
            throw new UsernameNotFoundException("用户名或密码错误");
        };
    }

    /**
     * 在用户名和密码校验前添加的过滤器，如果有jwt的token，会自行根据token信息进行登录
     *
     * @return
     */
    @Bean
    public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter() {
        return new JwtAuthenticationTokenFilter();
    }

}
```

# 六.授权和登录的异常解析类

`com.weixin.room.component.RestfulAccessDeniedHandler`

```java
import cn.hutool.json.JSONUtil;
import com.weixin.room.enums.ResponseEnum;
import com.weixin.room.util.ResultUtil;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author WanJingmiao
 * @Description 自定义权限不足的返回结果
 * @date 2020/5/10 10:56
 * @Version
 */
@Component
public class RestfulAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request,
                       HttpServletResponse response,
                       AccessDeniedException e) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json");
        response.getWriter().println(JSONUtil.parse(ResultUtil.error(ResponseEnum.ACCESS_ERROR)));
        response.getWriter().flush();
    }
}
```

`com.weixin.room.component.RestAuthenticationEntryPoint`

```java
import cn.hutool.json.JSONUtil;
import com.weixin.room.enums.ResponseEnum;
import com.weixin.room.util.ResultUtil;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author WanJingmiao
 * @Description 当未登录或者token失效访问接口时，自定义的返回结果
 * @date 2020/5/10 10:57
 * @Version
 */
@Component
public class RestAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json");
        response.getWriter().println(JSONUtil.parse(ResultUtil.error(ResponseEnum.UNAUTH_ERROR)));
        response.getWriter().flush();
    }
}
```

# 七.自定义的用户Details

`com.weixin.room.dto.AuthUserDetails`

```java
@Data
public class AuthUserDetails implements UserDetails {
    private static final long serialVersionUID = 7117432422333250549L;

    private User user;
    private List<Role> roleList;

    public AuthUserDetails(User user, List<Role> roleList) {
        this.user = user;
        this.roleList = roleList;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        //返回当前用户的权限
        return roleList.stream()
                .filter(permission -> permission.getName() != null)
                .map(permission -> new SimpleGrantedAuthority(permission.getName()))
                .collect(Collectors.toList());
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        //return user.getStatus().equals(1);
        return true;
    }
}

```

# 八.自定义JWT filter

`com.weixin.room.filter.JwtAuthenticationTokenFilter`

```java
import com.weixin.room.util.JwtTokenUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author WanJingmiao
 * @Description JWT登录过滤器，OncePerRequestFilter能够确保在一次请求只通过一次filter，而不需要重复执行
 * @date 2020/5/8 19:24
 * @Version
 */
@Slf4j
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
    @Autowired
    private UserDetailsService userDetailsService;
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    @Value("${jwt.tokenHeader}")
    private String tokenHeader;
    @Value("${jwt.tokenHead}")
    private String tokenHead;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String authHeader = request.getHeader(this.tokenHeader);
        if (null != authHeader && authHeader.startsWith(this.tokenHead)) {
            //解析token
            String authToken = authHeader.substring(this.tokenHead.length());
            String username = jwtTokenUtil.getUserNameFromToken(authToken);
            log.info("checking username:{}", username);
            //如果得到用户名并且当前用户未授权
            if(null!=username&& SecurityContextHolder.getContext().getAuthentication()==null){
                //根据用户名从数据库中获取用户信息(userDetailsService，UserDetails都是在config中自定义的)
                UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);
                if(jwtTokenUtil.validateToken(authToken,userDetails)){
                    UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(userDetails,null,userDetails.getAuthorities());
                    authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    log.info("authenticated user:{}", username);
                    //将认证信息存入上下文中
                    SecurityContextHolder.getContext().setAuthentication(authenticationToken);
                }
            }
        }
        filterChain.doFilter(request,response);
    }
}

```

# 九.Dao层设计

`com.weixin.room.dao.UserDao`

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.weixin.room.entity.Role;
import com.weixin.room.entity.User;
import org.apache.ibatis.annotations.Select;

import java.util.List;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 15:08
 * @Version
 */
public interface UserDao extends BaseMapper<User> {
    /**
     * 根据id获取当前用户的权限信息
     * @param id
     * @return
     */
    @Select("SELECT DISTINCT role.name FROM user " +
            "JOIN user_role ON user.id = user_role.user_id AND user_role.deleted=0 " +
            "JOIN role ON role.id = user_role.role_id "+
            "WHERE user.id=#{id}")
    List<Role> selectPermissions(Long id);
}

```

# 十.Service层设计

`com.weixin.room.service.UserService`

```java
import com.weixin.room.entity.Role;
import com.weixin.room.entity.User;

import java.util.List;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 14:59
 * @Version
 */
public interface UserService {
    /**
     * 根据用户名寻找用户
     *
     * @param username
     * @return
     */
    User getUserByUsername(String username);

    /**
     * 根据用户ID查找权限
     *
     * @param userId
     */
    List<Role> getPermissions(Long userId);

    /**
     * 注册功能
     */
    Boolean register(User userParam);

    /**
     * 登录功能
     * @param username 用户名
     * @param password 密码
     * @return 生成的JWT的token
     */
    String login(String username, String password);
}
```

`com.weixin.room.service.impl.UserServiceImpl`

```java
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.weixin.room.dao.UserDao;
import com.weixin.room.entity.Role;
import com.weixin.room.entity.User;
import com.weixin.room.service.UserService;
import com.weixin.room.util.JwtTokenUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 14:59
 * @Version
 */
@Service
@Slf4j
public class UserServiceImpl implements UserService {
    @Autowired
    UserDao userDao;
    @Autowired
    private PasswordEncoder passwordEncoder;
    @Autowired
    private UserDetailsService userDetailsService;
    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Override
    public User getUserByUsername(String username) {
        QueryWrapper<User> wrapper = new QueryWrapper<User>();
        wrapper.eq("username", username);
        return userDao.selectOne(wrapper);
    }

    @Override
    public List<Role> getPermissions(Long userId) {
        return userDao.selectPermissions(userId);
    }

    @Override
    public Boolean register(User userParam) {
        User user = new User();
        BeanUtils.copyProperties(userParam, user);
        //判断用户名是否重复
        QueryWrapper<User> wrapper = new QueryWrapper<User>();
        wrapper.eq("username", user.getUsername());
        if (userDao.selectOne(wrapper) != null) {
            return false;
        }
        //对密码进行加密
        String encodePassword = passwordEncoder.encode(user.getPassword());
        user.setPassword(encodePassword);
        userDao.insert(user);
        return true;
    }

    @Override
    public String login(String username, String password) {
        String token = null;
        try{
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            //比较密码
            if(!passwordEncoder.matches(password,userDetails.getPassword())){
                throw new BadCredentialsException("密码错误");
            }
            //组装认证信息并存入上下文
            UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails,null,userDetails.getAuthorities());
            SecurityContextHolder.getContext().setAuthentication(authentication);
            token = jwtTokenUtil.generateToken(userDetails);
        }catch (AuthenticationException e) {
            log.warn("登录异常:{}", e.getMessage());
        }
        return token;
    }
}
```

# 十一.Controller层设计

`com.weixin.room.controller.UserController.java`

```java
import com.weixin.room.entity.User;
import com.weixin.room.enums.ResponseEnum;
import com.weixin.room.service.UserService;
import com.weixin.room.util.ResultUtil;
import com.weixin.room.vo.ResultVo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 15:09
 * @Version
 */
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    UserService userService;
    @Value("${jwt.tokenHeader}")
    private String tokenHeader;
    @Value("${jwt.tokenHead}")
    private String tokenHead;

    /**
     * 测试接口，需要admin权限才能访问
     * @return
     */
    @GetMapping("/hello")
    @PreAuthorize("hasAuthority('admin')")
    public String test(){
        return "hello";
    }

    /**
     * 注册接口
     * @param user
     * @return
     */
    @PostMapping("/register")
    public ResultVo register(User user){
        Boolean isSuccess = userService.register(user);
        if(isSuccess){
            return ResultUtil.success();
        }
        return ResultUtil.error(ResponseEnum.REGISTER_ERROR);
    }

    /**
     * 登录接口
     * @param username
     * @param password
     * @return
     */
    @PostMapping("/login")
    public ResultVo login(String username,String password){
        String token = userService.login(username,password);
        if (token==null){
            return ResultUtil.error(ResponseEnum.LOGIN_ERROR);
        }
        Map<String, String> tokenMap = new HashMap<>();
        tokenMap.put("token", token);
        tokenMap.put("tokenHead", tokenHead);
        return ResultUtil.success(tokenMap);
    }

    /**
     * 获取当前用户所有的权限
     * @return
     */
    @GetMapping("/permission")
    public ResultVo getPermission(){
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        return ResultUtil.success(authentication.getAuthorities());
    }
}
```

# 十二.返回类设计

`com.weixin.room.vo.ResultVo`

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 15:10
 * @Version
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ResultVo<T> implements Serializable {

    private static final long serialVersionUID = -2224766970951876976L;

    /**
     * 错误码
     */
    private String code;

    /**
     * 提示信息
     */
    private String msg;

    /**
     * 数据
     */
    private T data;
}
```

`com.weixin.room.enums.ResponseEnum`

```java
import lombok.Getter;

/**
 * @author WanJingmiao
 * @Description 相应参数
 * @date 2020/5/8 15:14
 * @Version
 */
@Getter
public enum ResponseEnum {
    SUCCESS("0000","请求成功"),

    /**
     * 系统异常
     */
    NOTFOUND("1000","接口不存在"),
    ACCESS_ERROR("1001","拒绝访问"),
    REQUEST_ERROR("1002","请求方式错误"),
    PARAM_ERROR("1003","参数异常"),
    FILE_UPDATE_ERROR("1004","文件过大"),
    CLIENT_ERROR("1005","客户端错误"),
    SERVICE_ERROR("1006","服务异常 请联系管理员"),
    UNAUTH_ERROR("1007","未登录或者token失效"),
    LOGIN_ERROR("1008","登录失败"),
    REGISTER_ERROR("1009","注册失败");

    /**
     * 响应码
     */
    private String code;
    /**
     * 相应信息
     */
    private String msg;

    ResponseEnum(String code, String msg) {
        this.code = code;
        this.msg = msg;
    }
}

```

`com.weixin.room.util.ResultUtil`

```java
import com.weixin.room.enums.ResponseEnum;
import com.weixin.room.vo.ResultVo;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 14:59
 * @Version
 */
public final class ResultUtil {

    public static ResultVo success(Object data) {
        ResultVo result = new ResultVo();
        result.setCode(ResponseEnum.SUCCESS.getCode());
        result.setMsg(ResponseEnum.SUCCESS.getMsg());
        result.setData(data);
        return result;
    }

    public static ResultVo success() {
        return success(null);
    }

    public static ResultVo error(String code, String msg) {
        ResultVo result = new ResultVo();
        result.setCode(code);
        result.setMsg(msg);
        result.setData(null);
        return result;
    }

    public static ResultVo error(ResponseEnum ResponseEnum) {
        ResultVo result = new ResultVo();
        result.setCode(ResponseEnum.getCode());
        result.setMsg(ResponseEnum.getMsg());
        result.setData(null);
        return result;
    }
}
```

# 十三.异常处理

`com.weixin.room.exception.GlobalExceptionHandler`

```java
import com.weixin.room.enums.ResponseEnum;
import com.weixin.room.util.ResultUtil;
import com.weixin.room.vo.ResultVo;
import org.apache.catalina.connector.ClientAbortException;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.firewall.RequestRejectedException;
import org.springframework.web.HttpRequestMethodNotSupportedException;
import org.springframework.web.bind.MissingServletRequestParameterException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.method.annotation.MethodArgumentTypeMismatchException;
import org.springframework.web.multipart.MaxUploadSizeExceededException;
import org.springframework.web.servlet.NoHandlerFoundException;


/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/8 15:33
 * @Version
 */
@RestControllerAdvice
public class GlobalExceptionHandler {
    /**
     * 找不到资源 -> com.weixin.room.config.ErrorConfig
     * 未找到处理器 异常
     *
     * @param e
     * @return
     */
    @ExceptionHandler(NoHandlerFoundException.class)
    public ResultVo noHandlerFoundExceptionHandler(Exception e) {
        return ResultUtil.error(ResponseEnum.NOTFOUND);
    }


    /**
     * 权限不足
     *
     * @param e
     * @return
     */
    @ExceptionHandler(AccessDeniedException.class)
    public ResultVo accessDeniedExceptionHandler(Exception e) {
        return ResultUtil.error(ResponseEnum.ACCESS_ERROR);
    }

    /**
     * 请求方式错误
     *
     * @param e
     * @return
     */
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    public ResultVo httpRequestMethodNotSupportedExceptionHandler(Exception e) {
        return ResultUtil.error(ResponseEnum.REQUEST_ERROR);
    }


    /**
     * controller参数异常/缺少
     *
     * @param e
     * @return
     */
    @ExceptionHandler({
            MissingServletRequestParameterException.class,
            MethodArgumentTypeMismatchException.class,
            RequestRejectedException.class}
    )
    public ResultVo missingServletRequestParameterException(Exception e) {
        return ResultUtil.error(ResponseEnum.PARAM_ERROR);

    }

    /**
     * 单次上传文件过大
     *
     * @param e
     * @return
     */
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ResultVo maxUploadSizeExceededException(Exception e) {
        return ResultUtil.error(ResponseEnum.FILE_UPDATE_ERROR);
    }

    /**
     * 客户端错误
     *
     * @param e
     * @return
     */
    @ExceptionHandler(ClientAbortException.class)
    public ResultVo clientAbortExceptionException(Exception e) {
        return ResultUtil.error(ResponseEnum.CLIENT_ERROR);
    }


    /**
     * 其他异常
     *
     * @param e
     * @return
     */
    @ExceptionHandler(Exception.class)
    public ResultVo exceptionHander(Exception e) {
        e.printStackTrace();
        return ResultUtil.error(ResponseEnum.SERVICE_ERROR);
    }
}
```

# 十四.跨域配置

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * @author WanJingmiao
 * @Description 跨域配置
 * @date 2020/5/8 17:10
 * @Version
 */
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    /**
     * 跨域配置
     * @param registry
     */
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                .maxAge(3600)
                .allowedHeaders("*");
    }
}
```

# 十五 .常用工具类

`com.weixin.room.util.AuthUtil`

```java
import com.weixin.room.dto.AuthUserDetails;
import com.weixin.room.entity.User;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/16 17:10
 * @Version
 */
@Component
public class AuthUtil {
    /**
     * 判断当前登陆用户是否拥有某项权限
     *
     * @param permission
     * @return
     */
    public Boolean hasAuthority(String permission) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        for (GrantedAuthority auth : authentication.getAuthorities()) {
            if (auth.getAuthority().equals(permission)) {
                return true;
            }
        }
        return false;
    }

    /**
     * 获取当前登陆用户的全部信息
     *
     * @return
     */
    public User getUserInfo() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication.getPrincipal() instanceof AuthUserDetails) {
            return ((AuthUserDetails) authentication.getPrincipal()).getUser();
        }
        return null;
    }
}
```

