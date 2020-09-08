# 一.类图

## 1.1 类

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908115122.png)

- 类图有三部分组成：
  1. 类名
  2. 属性，描述类的成员变量及返回类型
  3. 方法，描述类的方法及返回类型

- 属性或方法前的修饰词：
  - 无修饰词：默认访问权限，表示属性或者方法只允许被同一个包中的类访问
  - `+`（公开）：表示所有对象都能访问这个属性或者方法，可跨类访问，跨包访问
  - `-`（私有）：被修饰的属性或者方法只能被该类的对象访问，子类不能访问，跨包不能访问
  - `#`（保护）：只有该类本身以及子类可以访问这个类的属性或者方法，即使子类在不同的包也能访问

## 1.2 接口

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908120813.png)

- 接口图由三部分组成
  1. 接口名，必须用斜体表示，而且顶端需要用`<<interface>>`修饰
  2. 常量，描述接口中的常量及类型
  3. 方法，描述类的方法及返回类型

## 1.3 注释

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908121646.jpg)

用一个右上角带卷角的长方形表示注释，并用虚线与实体类连接

# 二.类与类之间的关系

## 2.1 泛化关系（继承）

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908194653.png)

泛化关系就是常说的继承关系，也称为`is a kind of`，用来描述父子关系

泛化关系是对象间耦合度最高的一种关系，子类继承父类所有细节并添加自己的属性

泛化关系连接两个类用一根空心三角的实线连接，起始端是子类，终点端是父类

代码：

```java
public class A{...}
```

```java
public class B extends A{...}
```

## 2.2 实现关系

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908194508.png)

类B实现接口A的功能，那么类B和接口A的关系就是实现关系

用带空心的三角形的虚线连接类和接口，起始段是类，终端是接口

代码：

```java
public interface A{
    public void method(){...}
}
```

```java
public class B implements A {
    public void method(){...}
}
```

## 2.3 依赖关系

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908195303.png)

如果B类作为参数被A类在某个方法中使用，那么A类与B类具有依赖关系，即A依赖于B

代码：

```java
public class Person{
    public void method(Bus bus){
        bus.drive();
    }
}
```

```java
public class Bus{
    public void drive(){...}
}
```

## 2.4 关联关系

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908200533.png)

如果类B中的成员变量是用A类声明的变量，即A类作为属性在B类中存在，则A，B具有关联关系，称B关联与A

关联关系通常有三种关系：

- 单向关联
- 双向关联
- 自关联

代码：

```java
public class A {
    public void methodA(){...}
}
```

```java
public class B {
    private A a;
    //get set省略
    public void methodB(){...}
}
```

## 2.5 聚合关系

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908201043.png)

聚合关系是关联关系的一种（弱关联），体现整体和个体的关系（整体和个体可分离，个体可以独立于整体存在）

他是一种`has a`关系

代码：

```java
public class Station {
    private Train train;
    //set get方法省略
    public void start() {
        train.run();
    }
}
```

```java
public class Train {
    public void run() {...}
}
```

## 2.6 组合关系

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200908201520.png)

组合关系是关联关系的一种（强关联），与聚合类似体现整体和个体关系（这里的整体和个体不可分离，个体是整体的一部分）

代码：

```java
public class Car {
    private Engine engine;
    //get set省略
    public void start() {
        engine.start();
    }
}
```

```java
public class Engine {
    public void start(){...}
}
```