- `final`修饰类：这个类不能被继承，如`String、StringBuffer、System`类

- `final`修饰方法：这个方法不能被重写，如`Object`类的`getClass()`方法

- `final`修饰属性：变为常量，一旦确定就不能修改

- 用`static final`来修饰，即为全局变量且确定后不能修改

- `final`修饰参数，保证参数不能被`new`