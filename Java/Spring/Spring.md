# 一.Spring框架体系结构

![images](https://raw.githubusercontent.com/MrWater233/PictureHost/master/spring-overview.png)

# 二.IOC

## 2.1 程序的耦合和解耦

> 耦合：程序间的依赖关系

耦合包括：

1. 类之间的依赖
2. 方法间的依赖

解耦：

​	降低程序间的依赖关系

开发解耦：

1. 编译器不依赖
2. 运行时才依赖

解耦思路：

1. 用反射来创建对象，而避免使用`new`关键字
2. 读取配置文件来获取要创建的对象全限定类名

## 2.2 工厂模式解耦

> Bean：可重用组件
>
> JavaBean：用Java语言编写的可重用组件（≠实体类，＞实体类）

### 2.2.1 解耦流程

1. 使用配置文件配置需要解耦的类

   配置内容：唯一标识=全类名

2. 通过读取配置文件中的配置内容，反射创建对象，存入Bean容器中

#### 实例代码

1. `bean.properties`

   ```properties
   userService=com.spring.jdbc.service.UserService
   ```

2. `com.spring.jdbc.service.UserService`

   ```java
   public class UserService {
       public void useService(){
           System.out.println("hello");
       }
   }
   ```

3. **创建Bean工厂**，加载`bean.properties`中的配置，将需要解耦的对象加载进**Bean容器**，从而实现了对象的<font color=red>单例模式</font>

   `com.spring.jdbc.factory.BeanFactory`

   ```java
   public class BeanFactory {
       /**
        * 定义一个properties对象
        */
       private static Properties properties;
       /**
        * 定义一个Map，作为存储Bean的容器
        */
       private static Map<String, Object> beans;
   
       /**
        * 使用静态代码块为Properties对象赋值并初始化Bean容器
        */
       static {
           try {
               /**
                * 1.实例化配置对象
                */
               properties = new Properties();
               //加载ClassPath下的配置文件，获取properties的流对象
               InputStream in = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
               //将配置文件的内容进行装载
               properties.load(in);
               /**
                * 2.实例化Bean的容器
                */
               beans = new HashMap<String, Object>();
               /**
                * 3.加载配置文件并将反射的对象存入容器中
                */
               //获取key的枚举
               Enumeration<Object> keys = properties.keys();
               while (keys.hasMoreElements()) {
                   //取出每个key
                   String key = keys.nextElement().toString();
                   //根据key获取value
                   String beanPath = properties.getProperty(key);
                   //反射创建对象
                   Object value = Class.forName(beanPath).newInstance();
                   //把key和value存入容器中
                   beans.put(key, value);
               }
           } catch (Exception e) {
               e.printStackTrace();
               throw new ExceptionInInitializerError("初始化properties失败");
           }
       }
   
       /**
        * 根据Bean名称获取Bean对象
        *
        * @param beanName
        * @return
        */
       public static Object getBean(String beanName) {
           return beans.get(beanName);
       }
   }
   ```

4. 使用

   ```java
   public class Main {
       public static void main(String[] args) {
           UserService userService = (UserService) BeanFactory.getBean("userService");
           userService.useService();
       }
   }
   ```

## 2.3 Spring中的IOC（XML）

> 将创建类权力交给了工厂（框架），放弃了自己对类的创建控制
>
> 削减了程序的耦合

### 2.3.1 使用

1.  导入Spring框架依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>5.2.4.RELEASE</version>
   </dependency>
   ```

2. 创建Spring的Bean配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   </beans>
   ```

3. 将对象的创建交给Spring去管理

   id：Bean的名称，class：反射对象的全类名

   ```xml
   <bean id="userService" class="com.spring.jdbc.service.UserService"></bean>
   ```

4. 使用

   ```java
   public class Jdbc {
       /**
        * 获取IOC的核心容器，并根据id获取对象
        * @param args
        */
       public static void main(String[] args) {
           //1.获取容器对象
           ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
           //2.1 根据id获取Object类对象
           UserService userService = (UserService)ac.getBean("userService");
           //2.2 根据id和类对象直接获取对象
           //UserService userService1 = ac.getBean("userService",UserService.class);
   
           userService.useService();
       }
   }
   ```

### 2.3.2 ApplicationContext的实现类

> 用于根据参数构建bean工厂

#### ClassPathXmlApplicationContext

> 可以加载**类路径**下的配置文件

#### FileSystemXmlApplicationContext

> 加载磁盘任意路径下的配置文件（必须有访问权限）

#### AnnotationConfigApplicationContext

> 用于读取注解创建容器

### 2.3.3 核心容器的两大接口

#### ApplicationContext

*单例对象适用*

> 在构建核心容器时，创建对象的策略是立即加载
>
>  只要一读取完配置文件，就立刻创建对象

#### BeanFactory

*多例对象适用*

> 在构造核心容器是，采用延迟加载的方式加载
>
> 只有在通过容器获取对象的时候才创建对象

### 2.3.4 Spring对Bean的管理细节

#### 创建Bean的三种方式

1. 使用默认构造函数创建

   在Spring的配置文件中使用bean标签，除了id和class没有其他标签时使用

   **如果该类没有默认构造函数那么该对象无法创建**

   ```xml
   <bean id="userService" class="com.spring.jdbc.service.UserService"></bean>
   ```

2. 使用某个类中的方法返回值创建对象并存入容器

   `com.spring.jdbc.factory.InstanceFactory`

   ```java
   public class InstanceFactory {
       public UserService getUserService(){
           return new UserService();
       }
   }
   ```

   `bean.xml`

   ```xml
   <!--将工厂存入容器-->
   <bean id="instanceFactory" class="com.spring.jdbc.factory.InstanceFactory"></bean>
   <!--从工厂中的方法中获取bean-->
   <bean id="userService" factory-bean="instanceFactory" factory-method="getUserService"></bean>
   ```

3. 使用某个类中的静态方法创建对象并存入容器

   `com.spring.jdbc.factory.StaticFactory`

   ```java
   public class StaticFactory {
       public static UserService getUserService(){
           return new UserService();
       }
   }
   ```

   `bean.xml`

   ```xml
   <bean id="userService" class="com.spring.jdbc.factory.StaticFactory" factory-method="getUserService"></bean>
   ```

#### bean对象的作用范围

> 改变Bean标签的scope属性

1. **singleton（默认值）：单例**
2. **prototype：多例**
3. request：作用于web应用的请求范围
4. session：作用于web应用的会话范围
5. global-session：作用于集群环境的会话范围，当不是集群时为session

#### bean对象的生命周期

> 可以在Bean标签中加入
>
> `init-method`、`destory-method`指创建和销毁对象时运行的函数
>
> 要调用`destory-method`需要调用容器`close`方法，不然容器会直接销毁，无法查看效果

##### 单例对象

- 出生：容器创建
- 运行：容器还在，对象一直在
- 死亡：容器销毁
- **总结：与容器生命周期相同**

##### 多例对象

- 出生：通过容器获取对象时
- 运行：对象使用过程中一直活着
- 死亡：当对象长时间不使用，并且没有其他对象应用时，由Java的垃圾回收器回收

### 2.3.5 Spring的依赖注入

> 依赖注入就是**依赖关系的维护**
>
> 能注入的类型：
>
> 1. 基本类型和String
> 2. 其他Bean类型（在配置文件中或者注解标识的）
> 3. 复杂类型/集合类型
>
> 注入方式：
>
> 1. 构造函数提供
> 2. 使用set方法提供
> 3. 使用注解提供

#### 构造函数注入

> 使用constructor-arg标签给构造函数的参数赋值
>
> 缺点：必须在xml中配置所有的构造函数才能注入，不常用

`com.spring.jdbc.service.UserService`

```java
public class UserService {
    /**
     * 如果是经常变化的数据不适用于注入
     */
    private String name;
    private Integer age;
    private Date birthday;

    public UserService(String name, Integer age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }

    public void useService(){
        System.out.println(name+" "+age+" "+birthday);
    }
}
```

`bean.xml`

```xml
<!--构造函数的注入，使用constructor-arg标签指定构造函数-->
<bean id="userService" class="com.spring.jdbc.service.UserService">
    <constructor-arg name="name" value="test"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
    <constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
<!--注入一个当前时间的日期对象-->
<bean id="now" class="java.util.Date"></bean>
<!--注入一个自定义的日期对象-->
<!--<bean id="now" class="java.util.Date">-->
<!--    <constructor-arg name="year" value="100"></constructor-arg>-->
<!--    <constructor-arg name="month" value="1"></constructor-arg>-->
<!--    <constructor-arg name="date" value="1"></constructor-arg>-->
<!--</bean>-->
```

##### constructor-arg标签的属性

1. type：指定要注入的数据类型，该数据类型是构造函数中某个或者**某些**参数的类型

   再使用`value`将指定的值注入给指定的数据类型

2. index：指定赋值的索引位置，起始为0

3. **name：指定赋值的参数名字**

   --------------------------------------------------------------------------------------------------------------------------------

4. value：将基本数据类型和String类型赋给上面三种方式指定的参数的值

5. ref：指定其他的Bean类型（在SpringIOC容器中出现的Bean对象）给上面三种方式指定的参数赋值

#### Set方法注入

> 使用property配置需要赋值的参数
>
> 优势：更加灵活选择赋值，更常用

`com.spring.jdbc.service.UserService`

```java
public class UserService {

    private String name;
    private Integer age;
    private Date birthday;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public void useService(){
        System.out.println(name+" "+age+" "+birthday);
    }
}
```

`bean.xml`

```xml
<bean id="userService" class="com.spring.jdbc.service.UserService">
    <property name="name" value="test"></property>
    <property name="age" value="18"></property>
    <property name="birthday" ref="now"></property>
</bean>

<bean id="now" class="java.util.Date">
    <constructor-arg name="year" value="100"></constructor-arg>
    <constructor-arg name="month" value="1"></constructor-arg>
    <constructor-arg name="date" value="1"></constructor-arg>
</bean>
```

##### property标签的属性

1. name：指定注入时调用的**set方法名称**，不根据属性本身的名称
2. value：将基本数据类型和String类型赋给指定的参数
3. ref：指定其他的Bean类型（在SpringIOC容器中出现的Bean对象）给指定的参数

#### 集合类型的注入

`com.spring.jdbc.service.UserService`

```java
public class UserService {
    private String[] myStrs;
    private List<String> myList;
    private Set<String> mySet;
    private Map<String,Object> myMap;
    private Properties myPro;

    public void setMyStrs(String[] myStrs) {
        this.myStrs = myStrs;
    }

    public void setMyList(List<String> myList) {
        this.myList = myList;
    }

    public void setMySet(Set<String> mySet) {
        this.mySet = mySet;
    }

    public void setMyMap(Map<String, Object> myMap) {
        this.myMap = myMap;
    }

    public void setMyPro(Properties myPro) {
        this.myPro = myPro;
    }

    @Override
    public String toString() {
        return "UserService{" +
                "myStrs=" + Arrays.toString(myStrs) +
                ", myList=" + myList +
                ", mySet=" + mySet +
                ", myMap=" + myMap +
                ", properties=" + myPro +
                '}';
    }
}
```

`Bean.xml`

```xml
<bean id="userService" class="com.spring.jdbc.service.UserService">
    <property name="myStrs">
        <array>
            <value>A</value>
            <value>B</value>
            <value>C</value>
        </array>
    </property>
    <property name="myList">
        <list>
            <value>A</value>
            <value>B</value>
            <value>C</value>
        </list>
    </property>
    <property name="mySet">
        <set>
            <value>A</value>
            <value>B</value>
            <value>C</value>
        </set>
    </property>
    <property name="myMap">
        <map>
            <!--方式一-->
            <entry key="test1" value="aaa"></entry>
            <!--方式二-->
            <entry key="test2">
                <value>bbb</value>
            </entry>
        </map>
    </property>
    <property name="myPro">
        <props>
            <prop key="test1">aaa</prop>
            <prop key="test2">bbb</prop>
        </props>
    </property>
</bean>
```

##### 注意

- 用于给List结构集合注入值的标签：`list` `array` `set`

- 用于给Map结构集合注入值的标签：`map` `props`

<font color=red>结构相同，标签可以互换</font>

## 2.4 Spring中的IOC（注解）

> 注解需要的context命名空间和约束
>
> 如果想要去除XML，需要配置配置类

`bean.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <!--告知Spring在创建容器时需要扫描的包，配置所需要的标签不是在beans的约束中，而是在context名称空间和约束中-->
    <context:component-scan base-package="com.spring.jdbc"></context:component-scan>
</beans>
```

### 创建对象的注解

> 和<bean>标签实现的作用一样
>
> 以下注解的作用和属性相同，只是Spring为了让三层架构更加清晰

#### @Component

- 作用：用于将当前类存入Spring容器中
- 属性：
  - value：指定bean的id，默认为当前类名的首字母小写

#### @Controller

- 用处：表现层

#### @Service

- 用处：业务层

#### @Repository

- 用处：持久层

### 注入的注解

> 和<property>标签作用一样

<font color=red>在使用注解注入时，Set方法不是必须</font>

#### @Autowired

- 作用：按照类型注入
  - 只要容器中有**唯一**的bean对象**类型**匹配就可以注入成功
  - 当有多个类型匹配时，如果**变量名**和**同类型的某个bean的id**相同也能注入成功

#### @Qualifier

- 作用：在按照类型注入的基础上再按照bean的名称注入
  - 不能单独给类属性注入（<font color=red>需要配合@AutoWired</font>），但是可以单独给方法参数注入
- 属性：
  - value：指定注入Bean的id

#### @Resource

- 作用：直接按照Bean的id注入，可以独立使用
- 属性：
  - name：指定注入Bean的id

> 以上三个注解都只能注入Bean，基本类型和String的注入如下，但是集合类型的注入只能通过xml

#### @PropertySource

- 作用：指定properties文件的位置，再用`@Value`注入配置
- 属性：
  - value：指定文件的目录和名称

#### @Value

- 作用：注入基本类型和String类型
- 属性：
  - value：用于指定数值的值，可以使用SPEL表达式
    -  关键字：`classpath:`表示类路径

### 改变作用范围

> 在<Bean>标签中使用`scope`属性作用一样
>
> 标注在类上

#### @Scope

- 作用：用于指定Bean的作用范围
- 属性：
  - value：指定范围的取值，默认单例
    - singleton：单例
    - prototype：多例

### 生命周期相关

> 在<Bean>标签中使用`init-method` `destory-method`标签作用一样
>
> 标注在方法上

#### @PreDestory

- 作用：用于指定销毁方法

#### @PostConstruct

- 作用：用于指定初始化方法	

### 完全去除XML文件

#### @Configuration

- 作用：指明当前类为一个配置类
- 细节：当配置类作为`AnnotationConfigApplicationContext`（如下使用所示）的参数时可以不写，否则配置无效，Spring不会将Bean扫描进去

#### @ComponentScan

- 作用：指定Spring在创建容器时要扫描的包

  **取代XML中的扫描配置**

- 位置：配置类上

- 属性：

  - value/basePackages：指定扫描的包

#### @Bean

- 作用：把当前方法的返回值作为Bean对象存入容器中
- 位置：配置类中
- 属性：
  - name：用于指定Bean的id，默认为当前方法名
- 细节：如果方法有参数，Spring框架会从容器中查找是否有可用的Bean对象，查找和Autowired一样

#### @Import

- 作用：用于导入其他配置类到主配置类中

  当使用该注解后，有Imprt注解的类就是父配置类，而导入的都是子配置，子配置可以不需要`@Configuration`

- 位置：配置类上

- 属性：

  - value：指定其他配置类的字节码

#### 使用

```java
public class Main {
    public static void main(String[] args) {
        //此时需要用到注解Context实现类去扫描配置类
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfigruation.class);
        UserService userService = (UserService) applicationContext.getBean("userService");
        userService.test();
    }
}
```

## 2.5 Spring整合Junit

> 原生Junit无法在执行自己Runner类（main方法）时进行Spring容器的创建
>
> 所以必须在测试时进行手动创建容器，不然无法完成Bean的注入使用，比较麻烦

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-test</artifactId>
       <version>5.2.4.RELEASE</version>
       <scope>test</scope>
   </dependency>
   
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.12</version>
       <!-- 表示开发的时候引入，发布的时候不会加载此包 -->
       <scope>test</scope>
   </dependency>
   ```

2. 使用Junit提供的替换原有Runner（运行器：main方法）的注解，替换成Spring提供的

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   public class Test(){}
   ```

3. 告知Spring运行器SpringIOC容器是基于注解还是XML

   `@ContextConfiguration`

   属性：

   - locations：指定xml文件位置，加上`classpath:`指定在类路径下
   - classes：指定注解类所在的位置（`.class`）

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfigruation.class)
public class test {}
```

# 三.AOP

## 3.1 动态代理

> 特点：字节码随用随创建，随用随加载
>
> 作用：不修改原方法源码给方法增强
>
> 分类：
>
> 1. 基于接口的动态代理
>    - 涉及的类：Proxy
>    - 提供者：JDK官方
>    - 创建代理对象的要求：**被代理类至少实现一个接口**，如果没有不能使用
> 2. 基于子类的动态代理
>    - 涉及的类：Enhancer
>    - 提供者：第三方cglib库
>    - 创建代理对象的要求：被代理的类不能是最终类
>

### 3.1.1 基于接口的动态代理

#### 创建代理对象

> 使用Proxy类中的`newProxyInstance`方法

- `newProxyInstance`
  - 参数：
    - ClassLoader：类加载器，用于加载代理对象的字节码，和被代理对象使用相同的类加载器
    - Class[]：字节码数组，让代理对象和被代理对象有相同的方法，固定写法
    - InvocationHandler：用于提供增强的代码，写如何代理，一般写该接口实现了
      - invoke()方法
        - 作用：执行被代理对象的任何接口方法都会经过该方法
        - 参数：
          - proxy：代理对象的引用
          - method：当前执行的方法
          - args：当前执行方法需要的参数
          - 返回值：和被代理对象方法有相同的返回值

#### 具体代码

1. 生厂商接口

   `com.spring.jdbc.proxy.IProducer`

   ```java
   public interface IProducer {
       /**
        * 销售
        *
        * @param money
        */
       public void saleProduct(float money);
   
       /**
        * 售后
        *
        * @param money
        */
       public void afterService(float money);
   }
   ```

2. 生产商实现类

   `com.spring.jdbc.proxy.Producer`

   ```java
   public class Producer implements IProducer{
       /**
        * 销售
        *
        * @param money
        */
       @Override
       public void saleProduct(float money) {
           System.out.println("销售并拿到钱:" + money);
       }
   
       /**
        * 售后
        *
        * @param money
        */
       @Override
       public void afterService(float money) {
           System.out.println("提供售后服务比拿到钱:" + money);
       }
   }
   ```

3. 代理经销商

   `com.spring.jdbc.proxy.Client`

   ```java
   public class Client {
       public static void main(String[] args) {
           //匿名内部类访问外部成员需要final类型
           final Producer producer = new Producer();
           //创建一个经销商，代理厂家的销售方法
           IProducer proxyProducer = (IProducer) Proxy.newProxyInstance(producer.getClass().getClassLoader(),
                   producer.getClass().getInterfaces(),
                   new InvocationHandler() {
                       /**
                        * 作用：执行被代理对象的任何接口方法都会经过该方法
                        * @param proxy 代理对象的引用
                        * @param method 当前执行的方法
                        * @param args 当前执行方法需要的参数
                        * @return 和被代理对象方法有相同的返回值
                        * @throws Throwable
                        */
                       @Override
                       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                           /**
                            * 提供增强代码
                            */
                           //判断当前方法是不是销售
                           if ("saleProduct".equals(method.getName())) {
                               //获取原方法参数
                               Float money = (Float) args[0];
                               //修改传进原方法的参数新方法返回值
                               //参数：1.被代理对象 2.执行方法的参数
                               return method.invoke(producer, money * 0.8f);
                           }
                           return method.invoke(producer, args);
                       }
                   });
           proxyProducer.saleProduct(1000);
       }
   }
   ```


### 3.1.2 基于子类的动态代理

#### 创建代理对象

> 使用Ehancer类中的`create`方法

- `create`
  - 参数：
    - Class：指定被代理对象的字节码
    - callback：用于提供增强代码，一般用子接口实现类`MethodInterceptor`
      - intercept()方法
        - 作用：执行被代理对象的任何接口方法都会经过该方法
        - 参数：
          - o：代理对象的引用
          - method：当前执行的方法
          - objects[]：当前执行方法需要的参数
          - methodProxy：当前执行方法的代理对象
          - 返回值：和被代理对象方法有相同的返回值

#### 具体代码

1. 导入依赖

   ```xml
   <dependency>
       <groupId>cglib</groupId>
       <artifactId>cglib</artifactId>
       <version>2.1_3</version>
   </dependency>
   ```

2. 生产厂商

   `com.spring.jdbc.cglib.Producer`

   ```java
   public class Producer{
       /**
        * 销售
        *
        * @param money
        */
       public void saleProduct(float money) {
           System.out.println("销售并拿到钱:" + money);
       }
   
       /**
        * 售后
        *
        * @param money
        */
       public void afterService(float money) {
           System.out.println("提供售后服务比拿到钱:" + money);
       }
   }
   ```

3. 经销商

   `com.spring.jdbc.cglib.Client`

   ```java
   public class Client {
       public static void main(String[] args) {
           //匿名内部类访问外部成员需要final类型
           final Producer producer = new Producer();
           //创建一个经销商，代理厂家的销售方法
           Producer cglibProducer = (Producer) Enhancer.create(producer.getClass(), new MethodInterceptor() {
               /**
                * 作用：执行被代理对象的任何方法都会经过该方法
                * @param o 代理对象的引用
                * @param method 当前执行的方法
                * @param objects 当前执行方法需要的参数
                * 以上三个参数和基于接口的动态代理相同
                * @param methodProxy 当前执行方法的代理对象
                * @return 和被代理对象方法有相同的返回值
                * @throws Throwable
                */
               @Override
               public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                   /**
                    * 提供增强代码
                    */
                   //判断当前方法是不是销售
                   if ("saleProduct".equals(method.getName())) {
                       //获取原方法参数
                       Float money = (Float) objects[0];
                       //修改传进原方法的参数新方法返回值
                       //参数：1.被代理对象 2.执行方法的参数
                       return method.invoke(producer, money * 0.8f);
                   }
                   return method.invoke(producer, objects);
               }
           });
           cglibProducer.saleProduct(1000);
       }
   }
   ```

## 3.2 Spring中的AOP

### 3.2.1 AOP的术语

- JoinPoint（连接点）：

  指被拦截到的点，在Spring中这些点指方法

  通俗来说就是原来需要增强的所有方法

- PointCut（切入点）

  指我们对哪些JointPoint进行拦截定义

  即被增强的方法

**切入点一定是连接点，连接点不一定是切入点**

- advice（通知/增强）

  拦截到JoinPoint之后需要做的事

  - 通知类型：
    - 前置通知：在执行原方法（切入点方法）之前执行的通知
    - 后置通知：在执行原方法（切入点方法）之后、最终通知之前执行的通知
    - 异常通知：在织入过程中抛出异常时执行的通知，即在catch中执行的通知
    - 最终通知：最后执行的通知，即在finally中执行的通知
    - 环绕通知：整个invoke方法就是环绕通知

- Introduction（引介）

  特殊的通知，在运行期动态添加一些方法或者Field

- Target（目标对象）

  代理的目标对象

- Weaving（织入）

  把增强应用到目标对象来创建新对象的过程

  即加入增强的过程

- Proxy（代理）

  被AOP增强后返回产生的代理类

- Aspect（切面）

  切入点和通知之间的关系

### 3.2.2 基于XML的AOP配置

1. 依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>5.2.4.RELEASE</version>
   </dependency>
   <!--切入点表达式-->
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.9.5</version>
   </dependency>
   ```

2. 账户业务层接口

   `com.spring.service.IAccountService`

   ```java
   public interface IAccountService {
       /**
        * 模拟保存账户
        */
       void saveAccount();
   
       /**
        * 模拟更新账户
        * @param i
        */
       void updateAccount(int i);
   
       /**
        * 删除账户
        * @return
        */
       int deleteAccount();
   }
   ```

3. 账户实现类

   `com.spring.service.impl.AccountServiceImpl`

   ```java
   public class AccountServiceImpl implements IAccountService {
       @Override
       public void saveAccount() {
           System.out.println("执行了保存");
       }
   
       @Override
       public void updateAccount(int i) {
           System.out.println("执行了更新"+i);
       }
   
       @Override
       public int deleteAccount() {
           System.out.println("执行了删除");
           return 0;
       }
   }
   ```

4. 日志工具类，提供公共方法，作为切面通知

   `com.spring.utils.Logger`

   ```java
   public class Logger {
       /**
        * 用于打印日志，计划让其在切入点方法执行之前执行
        */
       public void printLog(){
           System.out.println("Logger中的printLog方法开始记录日志");
       }
   }
   ```

5. AOP和IOC的配置文件

   `bean.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd">
   </beans>
   ```

6. AOP的XML配置步骤

   1. 将需要增强的Bean交给Spring

      ```xml
      <bean id="accountService" class="com.spring.service.impl.AccountServiceImpl"></bean>
      ```

   2. 把通知的Bean也交给Spring

      ```xml
      <bean id="logger" class="com.spring.utils.Logger"></bean>
      ```

   3.  使用`<aop:config>`标签表明开始AOP的配置

      1. 在config标签内，使用`<aop:aspect>`标签表明配置切面

         - 属性：
           - id：切面的唯一标志
           - ref：指定通知类Bean的id

      2. 在aspect标签内配置[通知的类型](#advice)，建立通知方法和切入点之间的关系

         如`<aop:before>`前置通知

         - 属性：

           - method：指定前置通知的方法

           - pointcut：用于指定切入点表达式，指定业务层需要增强的方法

             - 写法：

               - 关键字：execution（表达式）

                 - 表达式：`访问修饰符 返回值 包名.包名...类名.方法名(参数列表)`

                   - **访问修饰符**可以省略

                   - 返回值：可以使用通配符（`*`），表示任意返回值

                     `* com.spring.service.impl.AccountServiceImpl.saveAccount()`

                   - 包名：可以使用通配符，表示任意包，但是有几级包就要有几个`*`；

                     也可以使用`..`表示当前包及其子包

                     `* *..AccountServiceImpl.saveAccount()`

                   - 类名和方法名：都可以使用`*`来实现通配

                     `* *..*.*()`

                   - 参数列表：

                     - 可以直接写数据类型
                       - 基本类型直接写名称，如`int`
                       - 引用类型写包名.类名的方式，如`java.lang.String`
                     - 可以用`*`表示任意类型，但是必须有参数
                     - 可以用`..`表示有无参数均可

                 - 全通配写法：`* *..*.*(..)`

               - 实际开发中的写法：

                 切到业务层实现类下的所有方法

                 如`execution(* com.spring.service.impl.*.*(..))`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd">
       <!--配置IOC-->
       <bean id="accountService" class="com.spring.service.impl.AccountServiceImpl"></bean>
   
       <!--Spring中基于XML的AOP配置步骤-->
       <!--1.配置通知类-->
       <bean id="logger" class="com.spring.utils.Logger"></bean>
       <!--2.配置AOP-->
       <aop:config>
           <!--3.配置切面-->
           <aop:aspect id="logAdvice" ref="logger">
               <!--4.配置通知的类型且建立通知方法和切入点方法的关联-->
               <aop:before method="printLog" pointcut="execution(public void com.spring.service.impl.AccountServiceImpl.saveAccount())"></aop:before>
           </aop:aspect>
       </aop:config>
   </beans>
   ```

7. 使用

   ```java
   public class test {
       public static void main(String[] args) {
           //获取容器
           ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
           //获取对象
           IAccountService as = (IAccountService)ac.getBean("accountService");
           //执行方法
           as.saveAccount();
       }
   }
   ```

#### <span id="advice">四种常用通知类型</span>

1. 前置通知：`<aop:before>`
2. 后置通知：`<aop:after-returning>`
3. 异常通知：`<aop:after-throwing>`
4. 最终通知：`<aop:after>`

#### 通用化切入点表达式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--配置IOC-->
    <bean id="accountService" class="com.spring.service.impl.AccountServiceImpl"></bean>

    <!--Spring中基于XML的AOP配置步骤-->
    <!--1.配置通知类-->
    <bean id="logger" class="com.spring.utils.Logger"></bean>
    <!--2.配置AOP-->
    <aop:config>
        <!--3.配置切面-->
        <aop:aspect id="logAdvice" ref="logger">
            <!--4.1 配置前置通知-->
            <aop:before method="beforePrintLog" pointcut-ref="pt1"></aop:before>
            <!--4.2 配置后置通知-->
            <aop:after-returning method="afterReturningPrintLog" pointcut-ref="pt1"></aop:after-returning>
            <!--4.3 配置异常通知-->
            <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>
            <!--4.4 配置最终通知-->
            <aop:after method="afterPrintLog" pointcut-ref="pt1"></aop:after>
            <!--配置切入点表达式
            id属性指定表达式唯一属性,expression属性指定表达式内容
            1.写在aop:aspect内部只能当前标签使用
            2.写在aop:aspect外部（必须写在引用切面之前），让所有切面可用
            -->
            <aop:pointcut id="pt1" expression="execution(* com.spring.service.impl.*.*(..))"/>
        </aop:aspect>
    </aop:config>
</beans>
```

#### 环绕通知

> 实质：他是Spring提供在代码中控制增强方法何时执行的方法
>
> 相当于动态代理的接口实现方法

1. `com.spring.utils.Logger`

   ```java
   public class Logger {
       /**
        * 环绕通知
        * Spring为我们提供一个接口ProceedingJoinPoint，接口中有一个proceed()，此方法相当于切入点调用方法。
        * 该接口可以作为环绕通知的方法参数，在程序执行时，Spring为我们提供接口的实现类使用
        */
       public Object aroundPrintLog(ProceedingJoinPoint pjp){
           Object returnValue = null;
           try {
               //得到切入点方法执行的参数
               Object[] args = pjp.getArgs();
               //前置通知
               System.out.println("环绕通知中的前置通知");
               //调用切入点方法
               returnValue = pjp.proceed(args);
               //后置通知
               System.out.println("环绕通知中的后置通知");
           } catch (Throwable throwable) {
               //异常通知
               System.out.println("环绕通知中的异常通知");
               throwable.printStackTrace();
           }finally {
               //最终通知
               System.out.println("环绕通知中的最终通知");
               return returnValue;
           }
       }
   }
   ```

2. `beam.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd">
       <!--配置IOC-->
       <bean id="accountService" class="com.spring.service.impl.AccountServiceImpl"></bean>
   
       <!--Spring中基于XML的AOP配置步骤-->
       <!--1.配置通知类-->
       <bean id="logger" class="com.spring.utils.Logger"></bean>
       <!--2.配置AOP-->
       <aop:config>
           <!--通用切入表达式-->
           <aop:pointcut id="pt1" expression="execution(* com.spring.service.impl.*.*(..))"/>
           <!--3.配置切面-->
           <aop:aspect id="logAdvice" ref="logger">
               <aop:around method="aroundPrintLog" pointcut-ref="pt1"></aop:around>
           </aop:aspect>
       </aop:config>
   </beans>
   ```

### 3.2.3 基于注解的AOP配置

> Spring注解的后置通知和异常通知会出现顺序问题
>
> 环绕通知不会有问题

加入了`xmlns:context="http://www.springframework.org/schema/aop"`命名空间

`bean.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <!--配置Spring创建容器时要扫描的包-->
    <context:component-scan base-package="com.spring"></context:component-scan>
    <!--配置Spring开启注解AOP支持-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

#### @Aspect

- 作用：表明当前类时切面类

#### @Pointcut

- 作用：指定切入点表达式
- 参数：切入点表达式

#### @Before

- 作用：表明当前通知是前置通知
- 参数：切入表达式的方法

#### @AfterReturning

- 作用：表明当前通知是后置通知
- 参数：切入表达式的方法

#### @AfterThrowing

- 作用：表明当前通知是异常通知
- 参数：切入表达式的方法

#### @After

- 作用：表明当前通知时最终通知
- 参数：切入表达式的方法

#### @Around

- 作用：表明当前通知时环绕通知
- 参数：切入表达式的方法

```java
@Component("logger")
@Aspect
public class Logger {
    /**
    * 切入表达式
    */
    @Pointcut("execution(* com.spring.service.impl.*.*(..))")
    private void pt1(){}
    
    /**
     * 前置通知
     */
    @Before("pt1()")
    public void beforePrintLog() {
        System.out.println("前置通知：Logger中的beforePrintLog方法开始记录日志");
    }

    /**
     * 后置通知
     */
    @AfterReturning("pt1()")
    public void afterReturningPrintLog() {
        System.out.println("后置通知：Logger中的afterPrintLog方法开始记录日志");
    }

    /**
     * 异常通知
     */
    @AfterThrowing("pt1()")
    public void afterThrowingPrintLog() {
        System.out.println("异常通知：Logger中的afterThrowingPrintLog方法开始记录日志");
    }

    /**
     * 最终通知
     */
    @After("pt1()")
    public void afterPrintLog() {
        System.out.println("最终通知：Logger中的finallyPrintLog方法开始记录日志");
    }

    /**
     * 环绕通知
     * Spring为我们提供一个接口ProceedingJoinPoint，接口中有一个proceed()，此方法相当于切入点调用方法。
     * 该接口可以作为环绕通知的方法参数，在程序执行时，Spring为我们提供接口的实现类使用
     */
    @Around("pt1()")
    public Object aroundPrintLog(ProceedingJoinPoint pjp){
        Object returnValue = null;
        try {
            //得到切入点方法执行的参数
            Object[] args = pjp.getArgs();
            //前置通知
            System.out.println("环绕通知中的前置通知");
            //调用切入点方法
            returnValue = pjp.proceed(args);
            //后置通知
            System.out.println("环绕通知中的后置通知");
        } catch (Throwable throwable) {
            //异常通知
            System.out.println("环绕通知中的异常通知");
            throwable.printStackTrace();
        }finally {
            //最终通知
            System.out.println("环绕通知中的最终通知");
            return returnValue;
        }
    }
}
```

#### 不使用XML的配置方式

在配置类上加上`@EnableAspectJAutoProxy`，再用注解context加载配置类即可

#### 补充

- 通知注解上的value和pointcut作用相同，都是用来指定确如表达式，当有pointcut时value会被覆盖

- 在普通通知中获取切入点方法

  `JoinPoint`是`ProceedingJoinPoint`父类，没有执行切点的`proceed`方法

  ```java
  @AfterReturning(value = "execution(* com.spring.service.impl.*.*(..))")
  public void afterReturningPrintLog(JoinPoint jp) {
      //获取切入点参数
      Object[] args = jp.getArgs();
  }
  ```

- 在普通通知里面获取切入点异常

  ```java
  @AfterThrowing(value = "execution(* com.spring.service.impl.*.*(..))",throwing = "exception")
  public void afterThrowingPrintLog(Exception exception) {
      System.out.println("异常为:"+exception);
  }
  ```

- 在普通通知里面获取切入点返回值

  ```java
  @AfterReturning(value = "execution(* com.spring.service.impl.*.*(..))", returning = "result")
  public void afterReturningPrintLog(Object result) {
      System.out.println("切入点返回值为：" + result);
  }
  ```

- [多切面问题](https://blog.csdn.net/single_wolf_wolf/article/details/81778879)

# 四.事务

## 4.1 依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.2.4.RELEASE</version>
</dependency>
```

## 4.2 Spring基于XML的事务控制

1. <span id="account">`com.spring.domain.Account`</span>

   ```java
   @Data
   public class Account implements Serializable {
       private Integer id;
       private String name;
       private Float money;
   }
   ```

2. `com.spring.dao.impl.AccountDaoImpl`

   ```java
   public class AccountDaoImpl  extends JdbcDaoSupport implements IAccountDao{
   
       @Override
       public Account findAccountById(Integer id) {
           List<Account> accounts = getJdbcTemplate().query("select *from account where id=?", new BeanPropertyRowMapper<Account>(Account.class), id);
           return accounts.isEmpty() ? null : accounts.get(0);
       }
   
       @Override
       public Account findAccountByName(String accountName) {
           List<Account> accounts = getJdbcTemplate().query("select *from account where name=?", new BeanPropertyRowMapper<Account>(Account.class), accountName);
           if (accounts.isEmpty()) {
               return null;
           }
           if (accounts.size() > 1) {
               throw new RuntimeException("结果集不唯一");
           }
           return accounts.get(0);
       }
   
       @Override
       public void updateAccount(Account account) {
           getJdbcTemplate().update("update account set name=?,money=? where id=?", account.getName(), account.getMoney(), account.getId());
       }
   }
   ```

3. `com.spring.service.impl.AccountServiceImpl`

   ```java
   public class AccountServiceImpl implements IAccountService {
       private IAccountDao accountDao;
       
       public void setAccountDao(IAccountDao accountDao) {
           this.accountDao = accountDao;
       }
   
       @Override
       public Account findAccountById(Integer id) {
           return accountDao.findAccountById(id);
       }
   
       @Override
       public void transfer(String sourceName, String targetName, Float money) {
           //1.根据名称查询转出账户
           Account source = accountDao.findAccountByName(sourceName);
           //2.根据名称查询转入账户
           Account target = accountDao.findAccountByName(targetName);
           //3.转出账户减钱
           source.setMoney(source.getMoney()-money);
           //4.转入账户加钱
           target.setMoney(target.getMoney()+money);
           //5.更新转出账户
           accountDao.updateAccount(source);
           //人为发生异常
           int i = 1/0;
           //5.更新转入账户
           accountDao.updateAccount(target);
       }
   }
   ```

4. XML配置

   1. XML的命名空间

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:aop="http://www.springframework.org/schema/aop"
             xmlns:tx="http://www.springframework.org/schema/tx"
             xsi:schemaLocation="
              http://www.springframework.org/schema/beans
              https://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/tx
              https://www.springframework.org/schema/tx/spring-tx.xsd
              http://www.springframework.org/schema/aop
              https://www.springframework.org/schema/aop/spring-aop.xsd">
      </beans>
      ```

   2. 配置事务管理器，需要指定数据源

      ```xml
      <!--数据源配置-->
      <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
          <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
          <property name="url" value="jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8"/>
          <property name="username" value="root"/>
          <property name="password" value="123456"/>
      </bean>
      <!--配置事务管理器-->
      <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource"/>
      </bean>
      ```

   3. 配置事务通知器

      - 此时需要导入事务的约束

      - 使用`<tx:advice>`标签配置事务通知

        - 属性：

          - id：给事务通知器一个唯一标识
          - transaction-manager：给事务通知器一个事务管理器的引用

        - 配置事务的属性，`<tx:attributes>`中的`<tx:method>`标签

          - <span id="transactional">属性：</span>

            - name：需要添加事务的方法名，可以使用通配符`*`

              如：`find*`表示所有以find开头的方法，或者`*`表示所有方法。优先范围小的

            - isolation：指定事务的隔离级别，默认时DEFAULT，表示使用数据库的默认隔离级别

            - propagation：指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改选；

              查询可选SUPPORTS

            - read-only：用于指定事务是否只读，默认为false（读写）；只有查询才设置为true

            - timeout：用于指定事务的超时时间，默认为-1，表示永不超时，如果指定数值以秒为单位

            - rollback-for：用于指定一个异常，当产生该异常时事务回滚，其他异常事务不回滚

              默认值表示任何异常都回滚

            - no-rollback-for：用于指定一个异常，当产生该异常时事务不会滚，其他异常事务回滚

              默认值表示任何异常都回滚

      ```xml
      <!--配置事务的通知，此时需要导入事务的约束，tx和aop的都需要-->
      <tx:advice id="txAdvice" transaction-manager="transactionManager">
          <!--配置事务的属性-->
          <tx:attributes>
              <tx:method name="*" propagation="REQUIRED" read-only="false"/>
              <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
          </tx:attributes>
      </tx:advice>
      ```

   4. 配置AOP中的通用切入点表达式，指定所有业务层方法

      建立事务通知和切入表达式的对应关系

      ```xml
      <!--配置AOP-->
      <aop:config>
          <!--配置切入点表达式-->
          <aop:pointcut id="pt1" expression="execution(* com.spring.service.impl.*.*(..))"/>
          <!--建立事务通知和切入表达式的对应关系-->
          <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
      </aop:config>
      ```

   完整配置：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xmlns:tx="http://www.springframework.org/schema/tx"
          xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/tx
           https://www.springframework.org/schema/tx/spring-tx.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd">
       <!--业务层-->
       <bean id="accountService" class="com.spring.service.impl.AccountServiceImpl">
           <property name="accountDao" ref="accountDao"/>
       </bean>
   
       <!--Dao层-->
       <bean id="accountDao" class="com.spring.dao.impl.AccountDaoImpl">
           <property name="dataSource" ref="dataSource"/>
       </bean>
   
       <!--数据源配置-->
       <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
           <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
           <property name="url" value="jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8"/>
           <property name="username" value="root"/>
           <property name="password" value="w327191248"/>
       </bean>
   
       <!--Spring中基于XML的声明式事务控制-->
       <!--1.配置事务管理器-->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <property name="dataSource" ref="dataSource"/>
       </bean>
       <!--2.配置事务的通知，此时需要导入事务的约束，tx和aop的都需要-->
       <tx:advice id="txAdvice" transaction-manager="transactionManager">
           <!--配置事务的属性-->
           <tx:attributes>
               <tx:method name="*" propagation="REQUIRED" read-only="false"/>
               <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
           </tx:attributes>
       </tx:advice>
       <!--3.配置AOP-->
       <aop:config>
           <!--配置切入点表达式-->
           <aop:pointcut id="pt1" expression="execution(* com.spring.service.impl.*.*(..))"/>
           <!--4.建立事务通知和切入表达式的对应关系-->
           <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
       </aop:config>
   </beans>
   ```

## 4.3 Spring基于注解的事务控制

1. `com.spring.domain.Account`[同上](#account)

2. `com.spring.dao.impl.AccountDaoImpl`

   ```java
   @Repository("accountDao")
   public class AccountDaoImpl implements IAccountDao{
       @Autowired
       private JdbcTemplate jdbcTemplate;
   
       @Override
       public Account findAccountById(Integer id) {
           List<Account> accounts = jdbcTemplate.query("select *from account where id=?", new BeanPropertyRowMapper<Account>(Account.class), id);
           return accounts.isEmpty() ? null : accounts.get(0);
       }
   
       @Override
       public Account findAccountByName(String accountName) {
           List<Account> accounts = jdbcTemplate.query("select *from account where name=?", new BeanPropertyRowMapper<Account>(Account.class), accountName);
           if (accounts.isEmpty()) {
               return null;
           }
           if (accounts.size() > 1) {
               throw new RuntimeException("结果集不唯一");
           }
           return accounts.get(0);
       }
   
       @Override
       public void updateAccount(Account account) {
           jdbcTemplate.update("update account set name=?,money=? where id=?", account.getName(), account.getMoney(), account.getId());
       }
   }
   ```

3. `com.spring.service.impl.AccountServiceImpl`

   使用`@Transactional`可以配置在类上或者方法上开启事务，[具体参数](#transactional)

   ```java
   @Service("accountService")
   @Transactional(propagation = Propagation.SUPPORTS,readOnly = true)
   public class AccountServiceImpl implements IAccountService {
       @Autowired
       private IAccountDao accountDao;
   
       @Override
       public Account findAccountById(Integer id) {
           return accountDao.findAccountById(id);
       }
   
       @Override
       @Transactional(propagation = Propagation.REQUIRED,readOnly = false)
       public void transfer(String sourceName, String targetName, Float money) {
           //1.根据名称查询转出账户
           Account source = accountDao.findAccountByName(sourceName);
           //2.根据名称查询转入账户
           Account target = accountDao.findAccountByName(targetName);
           //3.转出账户减钱
           source.setMoney(source.getMoney()-money);
           //4.转入账户加钱
           target.setMoney(target.getMoney()+money);
           //5.更新转出账户
           accountDao.updateAccount(source);
           //人为发生异常
           int i = 1/0;
           //5.更新转入账户
           accountDao.updateAccount(target);
       }
   }
   ```

4. XML命名空间

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xmlns:tx="http://www.springframework.org/schema/tx"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/tx
           https://www.springframework.org/schema/tx/spring-tx.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd
           http://www.springframework.org/schema/context
           https://www.springframework.org/schema/context/spring-context.xsd">
   </beans>
   ```

   配置：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xmlns:tx="http://www.springframework.org/schema/tx"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/tx
           https://www.springframework.org/schema/tx/spring-tx.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd
           http://www.springframework.org/schema/context
           https://www.springframework.org/schema/context/spring-context.xsd">
       <!--配置Spring创建容器时要扫描的包-->
       <context:component-scan base-package="com.spring"></context:component-scan>
   	
       <!--配置JdbcTemplate使用-->
       <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
           <property name="dataSource" ref="dataSource"></property>
       </bean>
   
       <!--数据源配置-->
       <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
           <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
           <property name="url" value="jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8"/>
           <property name="username" value="root"/>
           <property name="password" value="w327191248"/>
       </bean>
   
       <!--1.配置事务管理器-->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <property name="dataSource" ref="dataSource"/>
       </bean>
   
       <!--2.开启Spring对注解事务的支持-->
       <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
   
   </beans>
   ```

### 4.3.1 无XML的注解事务控制

> 使用代码代替bean.xml

- 主配置类

  `com.spring.config.SpringConfiguration`

  ```java
  /**
   * 1.声明配置类
   * 2.配置Spring创建容器时要扫描的包
   * 3.声明子配置类
   * 4.声明配置文件
   * 5.开启注解事务支持
   */
  @Configuration
  @ComponentScan("com.spring")
  @Import({JdbcConfig.class,TransactionConfig.class})
  @PropertySource(value = "jdbcConfig.properties")
  @EnableTransactionManagement
  public class SpringConfiguration {
  }
  ```

- 事务相关配置类

  `com.spring.config.TransactionConfig`

  ```java
  public class TransactionConfig {
      /**
       * 用于创建事务管理器对象
       * @param dataSource
       * @return
       */
      @Bean(name="transactionManager")
      public PlatformTransactionManager createPlatformTransactionManager(DataSource dataSource){
          return new DataSourceTransactionManager(dataSource);
      }
  }
  ```

- 连接数据库相关配置类

  `com.spring.config.JdbcConfig`

  ```java
  public class JdbcConfig {
      @Value("${jdbc.driver}")
      private String driver;
      @Value("${jdbc.url}")
      private String url;
      @Value("${jdbc.username}")
      private String username;
      @Value("${jdbc.password}")
      private String password;
  
      /**
       * 创建JdbcTemplate对象
       * @param dataSource
       * @return
       */
      @Bean(name = "jdbcTemplate")
      public JdbcTemplate createJdbcTemplate(DataSource dataSource){
          return new JdbcTemplate(dataSource);
      }
  
      /**
       * 配置数据源
       * @return
       */
      @Bean(name = "dataSource")
      public DataSource createDataSource(){
          DriverManagerDataSource dataSource = new DriverManagerDataSource();
          dataSource.setDriverClassName(driver);
          dataSource.setUrl(url);
          dataSource.setUsername(username);
          dataSource.setPassword(password);
          return dataSource;
      }
  }
  ```

  `resources.jdbcConfig.properties`

  ```properties
  jdbc.driver=com.mysql.cj.jdbc.Driver
  jdbc.url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8
  jdbc.username=root
  jdbc.password=123456
  ```

  

