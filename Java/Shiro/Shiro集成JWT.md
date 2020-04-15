1. 登陆成功后返回token

   ```java
   subject.login(token);
   //通过subject获取token并返回
   loginToken = subject.getSession().getId().toString();
   ```

2. 配置自定义的`sessionManager`来获取请求头中的token

   ```java
   public class MySessionManage extends DefaultWebSessionManager {
   
       private static final String AUTHORIZATION = "Authorization";
   
       private static final String REFERENCED_SESSION_ID_SOURCE = "Stateless request";
   
       public MySessionManage() {
           super();
       }
   
       @Override
       protected Serializable getSessionId(ServletRequest request, ServletResponse response) {
           String id = WebUtils.toHttp(request).getHeader(AUTHORIZATION);
           //如果请求头中有 Authorization 则其值为sessionId
           if (!StringUtils.isEmpty(id)) {
               request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE, REFERENCED_SESSION_ID_SOURCE);
               request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID, id);
               request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_IS_VALID, Boolean.TRUE);
               return id;
           } else {
               //否则按默认规则从cookie取sessionId
               return super.getSessionId(request, response);
           }
       }
   }
   ```

3. ajax复杂请求会先发送一个options请求，该请求请求头中没有Authorization，如果不配置会被shiro拦截，所以需要配置一个拦截器放行该请求

   ```java
   public class MyAuthenticationFilter extends FormAuthenticationFilter {
       /**
        * 放行options请求
        */
       @Override
       protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
           if (request instanceof HttpServletRequest) {
               if (((HttpServletRequest) request).getMethod().toUpperCase().equals("OPTIONS")) {
                   return true;
               }
           }
           return super.isAccessAllowed(request, response, mappedValue);
       }
       /**
        * 让Shiro在鉴权失败后不要redirect,而是返回status 401. 通过自定义AuthenticatingFilter实现.
        */
       @Override
       protected boolean onAccessDenied(ServletRequest request, ServletResponse response)
               throws Exception {
           WebUtils.toHttp(response).sendError(HttpServletResponse.SC_UNAUTHORIZED);
           return false;
       }
   }
   ```

4. 配置ShiroConfig

   ```java
   @Configuration
   public class ShiroConfig {
       /**
        * 注入自定义的Realm
        * @param matcher
        * @return
        */
       @Bean
       public RealmConfig realm(@Qualifier("hashedCredentialsMatcher") HashedCredentialsMatcher matcher){
           RealmConfig realm= new RealmConfig();
           realm.setCredentialsMatcher(matcher);
           return realm;
       }
       
       /*
        * 配置自定义的sessionManager
        * @return
        */
       @Bean
       public SessionManager sessionManager() {
           MySessionManage mySessionManager = new MySessionManage();
           return mySessionManager;
       }
   
       /**
        * 注入自定义的安全管理器
        * @param realm
        * @return
        */
       @Bean
       public DefaultWebSecurityManager securityManager(@Qualifier("realm") RealmConfig realm,@Qualifier("sessionManager") SessionManager sessionmanager){
           DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
           securityManager.setRealm(realm);
           securityManager.setSessionManager(sessionManager);
           return securityManager;
       }
   
       /**
        * 注入自定义的拦截器
        * @param securityManager
        * @return
        */
       @Bean
       public ShiroFilterFactoryBean shiroFilter(@Qualifier("securityManager") DefaultWebSecurityManager securityManager){
           ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
           shiroFilterFactoryBean.setSecurityManager(securityManager);
           Map<String,String> filterChainDefinitionMap = new LinkedHashMap<>();
           //登出
           filterChainDefinitionMap.put("/logout","logout");
           filterChainDefinitionMap.put("/**","anon");
           
           //登录成功跳转页面
           shiroFilterFactoryBean.setSuccessUrl("/index");
           //设置未授权跳转页面
           shiroFilterFactoryBean.setUnauthorizedUrl("/login");
           shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
           
           //配置自定义的拦截器
           Map<String, Filter> filterMap = shiroFilterFactoryBean.getFilters();
           filterMap.put("authc", new MyAuthenticationFilter());
           return filter;
       }
   
       /**
        * 自定义密码校验规则
        * @return
        */
       @Bean
       public HashedCredentialsMatcher hashedCredentialsMatcher(){
           HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
           //设置加密方式
           hashedCredentialsMatcher.setHashAlgorithmName("sha-256");
           //设置加密迭代次数
           hashedCredentialsMatcher.setHashIterations(UserConstant.ITER);
           //true=16进制格式 false=base64格式
           hashedCredentialsMatcher.setStoredCredentialsHexEncoded(false);
           return hashedCredentialsMatcher;
       }
   
   
       /**
        *  开启shiro aop注解支持.否则@RequiresRoles等注解无法生效
        *  使用代理方式;
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
        * @return
        */
       @Bean
       public static LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
           return new LifecycleBeanPostProcessor();
       }
   
       /**
        * 扫描上下文，寻找所有的Advistor(通知器）
        * 必须在LifecycleBeanPostProcessor之后创建
        * @return
        */
       @Bean
       public static DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator(){
           DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
           // 强制使用cglib，防止重复代理和可能引起代理出错的问题
           // https://zhuanlan.zhihu.com/p/29161098
           defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
           return defaultAdvisorAutoProxyCreator;
       }
   }
   ```

   