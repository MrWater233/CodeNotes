> - `panic`：抛出一个异常
> - `recover`：捕获异常
> - `defer`：延迟到函数`return`前执行

执行顺序：执行`panic()`后语句就会立刻中止，然后执行`defer`语句，然后报告异常信息，退出程序。如果`defer`中使用了`recover()`函数，则会捕获错误信息，`panic`抛出的异常信息不会报告，且程序不会终止。

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("a")
	f()
	fmt.Println("e")
}

func f() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("b")
		}
	}()
	fmt.Println("c")
	panic("error")
	fmt.Println("d")
}
```

结果：

```go
a
c
b
e
```

## defer

在函数中，我们经常需要创建资源（如，数据库连接 / 文件句柄 / 锁等），为了在函数执行完毕后，可以及时释放资源，Go框架设计了defer。

- 当go执行到一个defer时，不会立即执行defer后面的语句，而是将defer后面的语句压进栈中(可以理解为defer栈)
- 当函数执行结束后，defer 语句出栈，遵循先入后出
- 在defer将语句放入栈中时，也会将相关的值Copy，同时入栈
- defer最主要的价值在于，当函数执行完毕后，可以及时地释放函数创建的资源，如`defer file.close()`  / `defer connect.close()`

```go
package main
 
import "fmt"
 
func sum(num1 int, num2 int) int {
	defer fmt.Println("defer1 num1=", num1) //暂时不执行，压进defer栈中
	defer fmt.Println("defer2 num2=", num2)
	res := num1 + num2
	fmt.Println("resFun=", res)
	return res
}
 
func main() {
	res := sum(10,20)
	// 当函数执行完毕后，defer按照先进后出的方式出栈
	fmt.Println("res=",res)
}
```

结果

```go
resFun= 30
defer2 num2= 20
defer1 num1= 10
res= 30
```

