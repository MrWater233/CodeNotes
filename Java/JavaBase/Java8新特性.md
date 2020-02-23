# 1.哈希表

加载因子：默认0.75，即元素占有哈希表大小的75％时就进行扩容，表内元素重新运算位置

哈希算法通过类中的hashcode算法计算出**数组**的索引值

- 如果该位置有元素
  - 用equals方法判断
    - 相等：用新value替换旧value
    - 不相等：创建**链表**，将新值放在末尾，如果链表中碰撞个数>8并且哈希表大小>64时则将链表改为**红黑树**结构（除了添加操作，其他操作效率都比链表高）
- 如果该位置无元素
  - 直接插入

# 2.Lambda表达式

## 简介

Java8引入了`->`，该操作符称为箭头操作符或者Lambda表达式

<font color=red>Lambda表达式主要用来简化**函数式接口**（接口中只有一个抽象方法）的实现操作</font>

箭头操作符将Lambda操作符拆分成两部分：

- 左侧

  Lambda表达式的参数列表（接口方法中的参数列表）

- 右侧

  Lambda表达式中所执行的功能，即Lambda体（对抽象方法的实现）

## 语法格式

### 1）无参数&无返回值

`() -> System.out.println("Hello Lambda!");`

```java
@Test
public void test(){
    Runnable r = () -> System.out.println("Hello Lambda!");
    r.run();
}
```

### 2）有一个参数&无返回值

`(x) -> System.out.println(x);`

```java
@Test
public void test2(){
    Consumer<String> con = (x) -> System.out.println(x);
    con.accept("Hello Lambda!");
}
```

**如果只有一个参数，小括号可以省略不写**

`x -> System.out.println(x);`

### 3）有两个及以上参数&有返回值&Lambda体中有多条语句

`(x,y) -> {语句一;return 语句2};`

```java
@Test
public void test3(){
    Comparator<Integer> com = (x,y) -> {
        System.out.println("Hello Lambda!");
        return Integer.compare(x,y);
    };
}
```

<font color=red>如果Lambda体中只有一条语句，return和大括号都可以省略。</font>

```java
@Test
public void test3(){
	Comparator<Integer> com = (x, y) -> Integer.compare(x,y);
}
```

### 注意

- Lambda表达式的参数列表数据类型可以省略，因为JVM可以通过上下文推断数据类型，即”类型推断“
  如果写了数据类型，那么参数中数据类型全部要写

- Lambda只能用于函数式接口

  可以用`@FunctionalInterface`来检查是否时函数式接口

## 四大核心函数式接口

```java
consumer<T>//消费型接口
    void accept(T t);

Suppiler<T>//供给型接口
    T get();

Function<T,R>//函数型接口
    R apply(T t);

Predicate<T>//断言型接口
    boolean test(T t);
```

## 方法引用

若Lambda体中有的内容有方法已经实现了，我们可以使用”方法引用“

### 语法格式

1. 对象::实例方法名

   ```java
   @Test
   public void test1(){
       PrintStream ps = System.out;
       Consumer<String> con = ps::println;
       //等同于下面这个方式
       Consumer<String> con1 = (x) -> ps.println(x);
   }
   ```

2. 类::静态方法名

   ```java
   @Test
   public void test1(){
       Comparator<Integer> com = Integer::compare;
       //等同于下面这个方式
       Comparator<Integer> com1 = (x,y) -> Integer.compare(x,y);
   }
   ```

3. 类::实例方法名

   ```java
   @Test
   public void test1(){
       BiPredicate<String,String> bp = String::equals;
       //等同于下面这个方式
       BiPredicate<String,String> bp1 = (x,y) -> x.equals(y);
   }
   ```

   当第一个参数是实例方法调用者，第二个参数是实例方法的参数时，可以用这种方式。

   跟方法1不同，即不需要实际对象。

### 注意

**方法体的返回值和参数需要与函数式接口中方法一致**

## 构造器引用

### 语法格式

`ClassName::new`

调用构造器的参数列表要与函数式接口中抽象方法的参数列表保持一致

```java
@Test
public void test1(){
    //自动匹配无参构造器
    Supplier<String> sup = String::new;
    //等同于这个方式
    Supplier<String> sup1 = () -> new String();

    //自动匹配一个参数的构造器
    Function<String , String> fun = String::new;
    //等同于这个方式
    Function<String , String> fun1 = (x) -> new String(x);
}
```

## 数组引用

### 语法格式

`Type::new`

```java
@Test
public void test1(){
    Function<Integer,String[]> fun = String[]::new;
    //等同于以下方式
    Function<Integer,String[]> fun1 = (x) -> new String[x];
}
```

# 3.Stream API

> Stream是数据渠道，用于操作数据源产生的元素序列。
>
> Stream自己不会存储元素
>
> Stream会创造出一个新对象而不改变原对象
>
> Stream是延迟执行的

## Stream的操作步骤

![images](https://github.com/MrWater233/CodeNotes/blob/master/Java/JavaBase/images/Stream%E6%AD%A5%E9%AA%A4.png?raw=true)

### 1.创建Stream

```java
//创建Stream
@Test
public void test1(){
    //1.可以通过Collection 系列集合提供的stream()(串行流)或parallelStream()(并行流)获取流
    List<String> list = new ArrayList<>();
    Stream<String> stream1 = list.stream();

    //2.通过Arrays 中的静态方法stream()获取数组流
    String []str = new String[10];
    Stream<String> stream2 = Arrays.stream(str);

    //3.通过Stream 类中的静态方法of()
    Stream<String> stream3 = Stream.of("aa","bb","cc");

    //4.创建无限流
    //迭代，从0开始一直+2
    Stream<Integer> stream4 = Stream.iterate(0, (x) -> x + 2);
    //生成
    Stream<Double> stream5 = Stream.generate(() -> Math.random());
}
```

### 2.中间操作

<font color=red>多个中间操作可以连接起来形成一个流水线，除非流水线上触发中止操作，否则中间操作不会执行任何处理。</font>

<font color = blue>而中止操作一次性处理中间操作，称为**惰性求值**</font>

#### 筛选与切片

```java
@Data
class Employee{
    private String name;
    private int age;
    private double salary;
    public Employee(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}
List<Employee> employees = Arrays.asList(
    new Employee("张三",18,6000.99),
    new Employee("李四",30,2000.99),
    new Employee("王五",90,3000.99),
    new Employee("赵六",60,1000.99),
    new Employee("田七",50,4000.99)
);
```

- filter---接收Lambda，从流中排除这些元素

  ```java
  //内部迭代，由Stream API完成
  @Test
  public void test2(){
      //过滤年龄>35的员工
      Stream<Employee> stream = employees.stream().filter(e -> e.getAge() > 35);
      stream.forEach(System.out::println);
  }
  //外部迭代就是自己写迭代器来判断
  ```

- limit---截断流，使其元素不超过给定数量

  ```java
  @Test
  public void test3() {
      //过滤年龄>35的员工并且不超过2个，一旦找到两条数据以后迭代就不再进行
      Stream<Employee> stream = employees.stream().filter(e -> e.getSalary() > 2000).limit(2);
      stream.forEach(System.out::println);
  }
  ```

- skip(n)---跳过元素，返回一个扔掉了前n个元素的流。若流中元素不足n个，则返回一个空流。与limit(n)互补

  ```java
  @Test
  public void test3() {
      //过滤年龄>35的员工并跳过前两个
      Stream<Employee> stream = employees.stream().filter(e -> e.getSalary() > 2000).skip(2);
      stream.forEach(System.out::println);
  }
  ```

- distinct---筛选，通过流所生成的hashCode()和equals()去除重复元素

#### 映射

- map---接收Lambda，将元素转换成其他形式提取信息，接收一个函数作为参数，该函数会被应用到每一个元素上，并将其映射成新元素。

  ```java
  @Test
  public void test3() {
      List<String> list = Arrays.asList("aaa","bbb","ccc","ddd","eee");
      //将集合中的元素全部转换为大写，结果AAA BBB CCC...
      list.stream().map((str) -> str.toUpperCase()).forEach(System.out::println);
  }
  ```

- flatMap---接收一个函数作为参数，将流中的每一个值都换成另一个流，然后把所有流连成一个流

  如果不用flatMap那么解决拆分字符串用以下方法

  ```java
  @Test
  public void test3() {
      List<String> list = Arrays.asList("aaa","bbb","ccc","ddd","eee");
      //拆分字符串,Hello为类名
      Stream<Stream<Character>> stream = list.stream().map(Hello::filterCharacter);
      //{{a,a,a},{b,b,b},{c,c,c},{d,d,d},{e,e,e}}
      stream.forEach((sm) -> sm.forEach(System.out::println));
  }
  
  public static Stream<Character> filterCharacter(String str){
      List<Character> list = new ArrayList<>();
      for(Character ch:str.toCharArray()){
          list.add(ch);
      }
      return list.stream();
  }
  ```

  如果使用了flatMap，相当于把`{{a,a,a},{b,b,b},{c,c,c},{d,d,d},{e,e,e}}`整合成一个流

  ```java
  @Test
  public void test3() {
      List<String> list = Arrays.asList("aaa","bbb","ccc","ddd","eee");
      //拆分字符串
      Stream<Character> stream = list.stream().flatMap(Hello::filterCharacter);
      stream.forEach(System.out::println);
  }
  
  public static Stream<Character> filterCharacter(String str){
      List<Character> list = new ArrayList<>();
      for(Character ch:str.toCharArray()){
          list.add(ch);
      }
      return list.stream();
  }
  ```

#### 排序

- sorted()---自然排序
- sorted(Comparator com)---定制排序

### 3.中止操作（中断操作）

#### 查找与匹配

```java
@Data
class Employee{
    private String name;
    private int age;
    private double salary;
    private Status Status;

    public enum Status{
        FREE,
        BUSY,
        VOCATION;
    }
    public Employee(String name, int age, double salary, Employee.Status status) {
        this.name = name;
        this.age = age;
        this.salary = salary;
        Status = status;
    }
}

List<Employee> employees = Arrays.asList(
    new Employee("张三",18,6000.99, Employee.Status.FREE),
    new Employee("李四",30,2000.99, Employee.Status.BUSY),
    new Employee("王五",90,3000.99, Employee.Status.VOCATION),
    new Employee("赵六",60,1000.99, Employee.Status.BUSY),
    new Employee("田七",50,4000.99, Employee.Status.FREE)
);
```

- allMatch---检查是否匹配所有元素

  ```java
  @Test
  public void test1(){
      //false
      System.out.println(employees.stream().allMatch(e -> e.getStatus().equals(Employee.Status.BUSY)));
  }
  ```

- anyMatch---检查是否至少匹配一个元素

  ```java
  @Test
  public void test1(){
      //true
      System.out.println(employees.stream().anyMatch(e -> e.getStatus().equals(Employee.Status.BUSY)));
  }
  ```

- noneMatch---检查是否没有匹配所有元素

  ```java
  @Test
  public void test1(){
      //flase
      System.out.println(employees.stream().noneMatch(e -> e.getStatus().equals(Employee.Status.BUSY)));
  }
  ```

- findFirst---返回第一个元素

  ```java
  @Test
  public void test1(){
      //为了防止空指针异常用optional对象
      Optional<Employee> op = employees.stream().sorted((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())).findFirst();
      //如果为空就用参数代替,'赵六'
      System.out.println(op.orElse(null));
  }
  ```

- findAny---返回当前流中任意元素

  ```java
  @Test
  public void test1(){
      Optional<Employee> op = employees.parallelStream().filter(e -> e.getStatus().equals(Employee.Status.BUSY)).findAny();
      System.out.println(op.orElse(null));
  }
  ```

  这里应当使用并行流，不然始终返回第一个元素

- count---返回流中元素的总个数

  ```java
  @Test
  public void test1(){
      Long count = employees.stream().count();
      //5
      System.out.println(count);
  }
  ```

- max---返回流中最大值

  ```java
  @Test
  public void test1(){
      Optional<Employee> op = employees.stream().max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
      //张三
      System.out.println(op.get());
  }
  ```

- min---返回流中最小值

  ```java
  @Test
  public void test1(){
      //返回最低工资
      Optional<Double> op = employees.stream().map(Employee::getSalary).min(Double::compare);
      //1000.99
      System.out.println(op.get());
  }
  ```

#### 规约

- reduce(T identity,BinaryOperatior) / reduce(BinaryOperator)---可以将流中元素反复结合起来，得到一个值

  ```java
  @Test
  public void test1(){
      List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
      //先将0作为x，然后从流中取下一个元素作为y，算出结果再作为x，一直反复结合
      Integer sum = list.stream().reduce(0, (x, y) -> x + y);
      //55
      System.out.println(sum);
  }
  ```

  ```java
  //计算工资的总和
  @Test
  public void test1(){
      Optional<Double> op = employees.stream().map(Employee::getSalary).reduce(Double::sum);
      System.out.println(op.get());
  }
  ```

#### 收集

- collect---将流转换为其他形式。接收一个Collector接口的实现，用于给Stream中元素做汇总的方法

  Collectors给了很多集合的实现方法，如`toList()、toSet()`：

  ```java
  @Test
  public void test1(){
      List<String> list = employees.stream().map(Employee::getName).collect(Collectors.toList());
      list.forEach(System.out::println);
  }
  ```

  转换为指定的集合：

  ```java
  @Test
  public void test1(){
      HashSet<String> list = employees.stream().map(Employee::getName).collect(Collectors.toCollection(HashSet::new));
      list.forEach(System.out::println);
  }
  ```

  Collectors的其他用法：

  ```java
  @Test
  public void test1(){
      //总数 5
      Long count = employees.stream().collect(Collectors.counting());
      System.out.println(count);
  
      //平均值 3200.99
      Double avg = employees.stream().collect(Collectors.averagingDouble(Employee::getSalary));
      System.out.println(avg);
  
      //总和 16004.949999999999
      Double sum = employees.stream().collect(Collectors.summingDouble(Employee::getSalary));
      System.out.println(sum);
  }
  ```

  将sum、average、max统一操作：

  ```java
  @Test
  public void test1(){
      DoubleSummaryStatistics dss = employees.stream().collect(Collectors.summarizingDouble(Employee::getSalary));
      System.out.println(dss.getSum());
      System.out.println(dss.getAverage());
      System.out.println(dss.getMax());
  }
  ```

  分组：

  ```java
  @Test
  public void test1(){
      Map<Employee.Status, List<Employee>> map = employees.stream().collect(Collectors.groupingBy(Employee::getStatus));
      System.out.println(map);
  }
  ```

  多级分组：

  ```java
  @Test
  public void test1(){
      Map<Employee.Status, Map<String, List<Employee>>> map = employees.stream().collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy(e -> {
          if (e.getAge() <= 35) {
              return "青年";
          } else if (e.getAge() <= 50) {
              return "中年";
          } else {
              return "老年";
          }
      })));
      System.out.println(map);
  }
  ```

  分区(按照true和false分)：

  ```java
  @Test
  public void test1(){
      Map<Boolean, List<Employee>> map = employees.stream().collect(Collectors.partitioningBy(e -> e.getSalary() > 8000));
      System.out.println(map);
  }
  ```

  连接：

  ```java
  @Test
  public void test1(){
      String str = employees.stream().map(Employee::getName).collect(Collectors.joining("，"));
      //张三，李四，王五，赵六，田七
      System.out.println(str);
  }
  ```

## 并行流和串行流

### Fork&Join框架

多线程并行计算，并且是工作窃取模式（某个线程如果计算完成会从其他线程队列中获取新的任务继续计算），效率高 

计算累加：

```java
public class ForkJoinCalculate extends RecursiveTask<Long> {
    private static  final long serialVersionUID = 12345563214L;

    private long start;
    private long end;
    private static final long THRESHOLD = 10000;

    public ForkJoinCalculate(long start,long end){
        this.start = start;
        this.end = end;
    }
    @Override
    protected Long compute() {
        long length = end - start;
        if(length<=THRESHOLD){
            long sum = 0;
            for(long i = start;i<end;i++){
                sum+=i;
            }
            return sum;
        }else{
            long middle = (start+end)/2;
            ForkJoinCalculate left = new ForkJoinCalculate(start,middle);
            left.fork();//拆分子任务，同时压入线程队列
            ForkJoinCalculate right = new ForkJoinCalculate(middle+1,end);
            right.fork();
            return left.join()+right.join();
        }
    }
}
```

```java
@Test
public void test1(){
    Instant start = Instant.now();

    ForkJoinPool pool = new ForkJoinPool();
    ForkJoinTask<Long> task = new ForkJoinCalculate(0,100000000L);
    Long sum = pool.invoke(task);
    //5000000050000000
    System.out.println("结果为："+sum);

    Instant end = Instant.now();
    //计算运行时间
    System.out.println("耗费时间为："+Duration.between(start,end).toMillis()+"毫秒");
}
```

### 并行流

> Java8中使用parallel()，底层为ForkJoin，但是实现简单

通过`parallel()`与`sequential()`在并行流和顺序流之间切换

计算累加(`rangeClosed(0,100000000L)`生成0-100000000的数)：

```java
@Test
public void test2(){
    Instant start = Instant.now();
    long sum = LongStream.rangeClosed(0, 100000000L).parallel().reduce(0, Long::sum);
    System.out.println("结果为："+sum);
    Instant end = Instant.now();
    System.out.println("耗费时间为："+Duration.between(start,end).toMillis()+"毫秒");
}
```

# 4.Optional类

> 容器类，代表一个类存在或者不存在，避免空指针异常 

## 常用方法

| 方法                       | 作用                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `Optional.of(T t)`         | 创建一个Optional实例                                         |
| `Optional.empty()`         | 创建一个空的Optional实例                                     |
| `Optional.ofNullable(T t)` | 若t不为null，创建实例，否则创建空的实例                      |
| `ispresent()`              | 判断是否包含值                                               |
| `orElse(T t)`              | 如果调用对象包含值，返回该值，否则返回t                      |
| `orElseGet(Supplier s)`    | 如果调用对象包含值，返回该值，否则返回s获取的值              |
| `map(Function f)`          | 如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty() |
| `flatMap(Function mapper)` | 与map相似，要求返回值必须是Optional                          |

