# 一.单例模式

> 一个应用中只有一个实例

## 1.1饿汉式

```java
public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton(){}
    public static Singleton getInstance(){
        return instance;
    }
}
```

- 优点：线程安全

- 缺点：在类加载的时候就实例化，造成了堆内存浪费，优化点在应该实现懒加载

## 1.2懒汉式 

```java
public class SingletonLazy {
    private static SingletonLazy instance;
    private static SingletonLazy getInstance(){
        if(instance == null){
            synchronized (SingletonLazy.class){
                if(instance == null)
                    instance = new SingletonLazy();
            }
        }
        return instance;
    }
}
```

- 使用**双重锁**机制，保证了线程安全，同时**避免了堆内存的浪费**，做到了用到的时候再生成对象

# 二.工厂模式

## 2.1静态工厂模式

> 项目中的各种静态辅助类

## 2.2简单工厂模式

> 将生产类的工厂从使用中剥离出来，给工厂传入不同的参数来获取不同的对象
>
> 但是这个模式不符合开闭原则

- 创建一个形状的接口

  ```java
  public interface Shape {
      void draw();
  }
  ```

- 各种形状的实现类

  ```java
  public class CircleShape implements Shape{
      @Override
      public void draw() {
          System.out.println("Circle:draw");
      }
  }
  ```

  ```java
  public class RectShape implements Shape{
      @Override
      public void draw() {
          System.out.println("Rect:draw");
      }
  }
  ```

- 创建一个根据需要返回对应形状的工厂

  ```java
  public class ShapeFactory {
      public static Shape getShape(String type) {
          Shape shape = null;
          if (type.equalsIgnoreCase("circle")) {
              shape = new CircleShape();
          } else if(type.equalsIgnoreCase("rect")){
              shape = new RectShape();
          }
          return shape;
      }
  }
  ```

## 2.3工厂方法模式

> 工厂方法模式时简单工厂模式的进一步深化，不是所有的类使用同一个工厂，而是每个对象都有一个对应的工厂

- 创建一个形状的接口

  ```java
  public interface Shape {
      void draw();
  }
  ```

- 各种形状的实现类

  ```java
  public class CircleShape implements Shape{
      @Override
      public void draw() {
          System.out.println("Circle:draw");
      }
  }
  ```

  ```java
  public class RectShape implements Shape{
      @Override
      public void draw() {
          System.out.println("Rect:draw");
      }
  }
  ```

- 工厂的接口

  ```java
  public interface ShapeFactory {
      Shape getShape();
  }
  ```

- 各种工厂的实现类

  ```java
  public class CircleFactory implements ShapeFactory{
      @Override
      public Shape getShape() {
          return new CircleShape();
      }
  }
  ```

  ```java
  public class RectFactory implements ShapeFactory{
      @Override
      public Shape getShape() {
          return new RectShape();
      }
  }
  ```

> 工厂方法相比较简单工厂的好处：在需要扩展功能的时候不会违背开闭原则，直接创建对应的实现类和是实现工厂即可使用

## 2.4抽象方法模式

> 是工厂模式的进一步深化，但是不符合开闭原则
>
> 这个模式中工厂可以创建一组对象

比如现在每个系统有两个模块，显示和操作

- 抽象显示控制器和操作控制器

  ```java
  public interface DisplayController {
      void display();
  }
  ```

  ```java
  public interface OperationController {
      void operation();
  }
  ```

- 接下来有两个系统，系统A和系统B

  - 系统A

    ```java
    public class SystemADisplayController implements DisplayController{
        @Override
        public void display() {
            System.out.println("SystemA Display");
        }
    }
    ```

    ```java
    public class SystemAOperationController implements OperationController{
        @Override
        public void operation() {
            System.out.println("SystemA Operation");
        }
    }
    ```

  - 系统B

    ```java
    public class SystemBDisplayController implements DisplayController{
        @Override
        public void display() {
            System.out.println("SystemB Display");
        }
    }
    ```

    ```java
    public class SystemBOperationController implements OperationController{
        @Override
        public void operation() {
            System.out.println("SystemB Operation");
        }
    }
    ```

- 定义抽象工厂，完成对两种类创建的抽象

  ```java
  public interface SystemFactory {
      DisplayController createDisplayController();
      OperationController createOperationController();
  }
  ```

- 实现两种系统的工厂创建

  ```java
  public class SystemAFactory implements SystemFactory{
      @Override
      public DisplayController createDisplayController() {
          return new SystemADisplayController();
      }
  
      @Override
      public OperationController createOperationController() {
          return new SystemAOperationController();
      }
  }
  ```

  ```java
  public class SystemBFactory implements SystemFactory {
      @Override
      public DisplayController createDisplayController() {
          return new SystemBDisplayController();
      }
  
      @Override
      public OperationController createOperationController() {
          return new SystemBOperationController();
      }
  }
  ```

  

