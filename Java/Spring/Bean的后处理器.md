# 介绍

> Bean后处理器能够为容器中的其他Bean进行处理，如为目标Bean生成代理，他在容器中Bean初始化完成之后执行。
>
> Bean的后处理器不需要id属性，他作为为其他Bean服务而存在。
>
> 他也可以实现AOP的功能，为容器的Bean进行动态代理。

如果要定义一个后处理器Bean，需要实现`BeanPostProcessor`接口，接口需要实现的方法：

- 在容器为每个Bean初始化方法之前调用：

  ```java
  @Override
  public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      return bean;
  }
  ```

- 在容器为每个Bean初始化方法之后调用：

  ```java
  @Override
  public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      return bean;
  }
  ```

- 方法参数：

  - bean：容器中的Bean实例
  - beanName：容器中Bean的id

# 实例

- 服务层接口

  `service.TestService`

  ```java
  public interface TestService {
      public String doFirst();
  
      public void doSecond();
  }
  ```

- 服务层实现类

  ```java
  public class TestServiceImpl implements TestService {
      public TestServiceImpl() {
          System.out.println("无参构造函数执行");
      }
  
      @Override
      public String doFirst() {
          System.out.println("执行doFirst");
          return "hello";
      }
  
      @Override
      public void doSecond() {
          System.out.println("执行doSecond");
      }
  }
  ```

- 后处理器，进行动态代理增强

  `bean.MyBeanPostProcessor`

  ```java
  public class MyBeanPostProcessor implements BeanPostProcessor {
      @Override
      public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
          System.out.println("ProcessBefore执行");
          return bean;
      }
  
      /**
       * 将方法增强为大写返回
       *
       * @param bean
       * @param beanName
       * @return
       * @throws BeansException
       */
      @Override
      public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
          Object proxy = Proxy.newProxyInstance(bean.getClass().getClassLoader(),
                  bean.getClass().getInterfaces(),
                  new InvocationHandler() {
                      @Override
                      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                          Object result = method.invoke(bean, args);
                          if (result != null) {
                              result = ((String) result).toUpperCase();
                          }
                          return result;
                      }
                  });
          System.out.println("ProcessAfter执行");
          return proxy;
      }
  }
  ```

- bean.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="testService" class="service.impl.TestServiceImpl"/>
      <!-- 注册bean后处理器 -->
      <bean class="bean.MyBeanPostProcessor"/>
  </beans>
  ```

- 测试

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(locations = "classpath:bean.xml")
  public class SpringTest {
  
      @Autowired
      TestService testService;
  
      @Test
      public void test(){
          String result = testService.doFirst();
          testService.doSecond();
          System.out.println("结果:"+result);
      }
  }
  ```

- 结果

  ```txt
  无参构造函数执行
  ProcessBefore执行
  ProcessAfter执行
  执行doFirst
  执行doSecond
  结果:HELLO
  ```

  



