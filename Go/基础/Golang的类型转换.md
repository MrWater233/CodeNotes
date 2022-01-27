> Golang类型转换分为三种：
>
> - 断言
> - 强制
> - 显式

# 一.断言

## 1.1 类型断言

1. ```go
   s := x.(T)
   ```

   - 如果`x`不是`nil`，且`x`可以转换成`T`类型，就会断言成功，返回`T`类型的变量`s`。如果断言失败就会触发`panic`

   - 如果`T`不是接口类型，则要求`x`的类型就是`T`，如果`T`是一个接口，要求`x`实现了`T`接口

2. ```go
   s, ok := x.(T)
   ```

   这种写法如果断言失败不会出现`panic`，ok就代表了是否会成功

## 1.2 类型switch

```go
switch i := x.(type) {
case nil:
    printString("x is nil")                // type of i is type of x (interface{})
case int:
    printInt(i)                            // type of i is int
case float64:
    printFloat64(i)                        // type of i is float64
case func(int) float64:
    printFunction(i)                       // type of i is func(int) float64
case bool, string:
    printString("type is bool or string")  // type of i is type of x (interface{})
default:
    printString("don't know the type")     // type of i is type of x (interface{})
}
```

`x`断言成了`type`类型，`type`类型具体值就是`switch case`的值

此时`i := x.(type)`的`i`在`case`中就已经是对应类型的变量了

# 二.强制类型转换

## 2.1 unsafe

```go
var f float64
bits = *(*uint64)(unsafe.Pointer(&f))

type ptr unsafe.Pointer
bits = *(*uint64)(ptr(&f))

var p ptr = nil
```

此时`float64`就强制转为了`int64`类型

## 2.2 接口类型检测

```go
var _ Context = (*ContextBase)(nil)
```

可以用来判断`*ContextBase`是否实现了`Context接口`

# 三.显式类型转换

一个显式转换的表达式`T(x)`，其中`T`是一种类型并且`x`是可转换为类型的表达式`T`，例如：`uint(666)`

在以下任何一种情况下，变量 x 都可以转换成 T 类型：

- `x`可以分配成`T`类型。
- 忽略`struct`标记`x`的类型和`T`具有相同的基础类型。
- 忽略`struct`标记`x`的类型和`T`是未定义类型的指针类型，并且它们的指针基类型具有相同的基础类型。
- `x`的类型和`T`都是整数或浮点类型。
- `x`的类型和`T`都是复数类型。
- `x`的类型是整数或`[] byte`或`[] rune`，并且`T`是字符串类型。
- `x`的类型是字符串，`T 类型是 [] byte 或 [] rune。

```go
int64(222)
[]byte("ssss")

type A int
A(2)
```

