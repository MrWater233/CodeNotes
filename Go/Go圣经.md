# 一.程序结构

## 1.1 命名

命名规则：

- 必须以一个**字母**或者**下划线**开头，后面可以跟任意的字母/数字/下划线

- 区分大小写

- 不能用关键字来命名

  Go中的关键字有25个

  ```go
  break      default       func     interface   select
  case       defer         go       map         struct
  chan       else          goto     package     switch
  const      fallthrough   if       range       type
  continue   for           import   return      var
  ```

此外还有30多个预定义的名字，比如`int`和`true`等，这些预定义的名字不是关键字，可以再定义来使用，但是还是要避免语义混乱。



名字开头的**字母大小写**决定了名字在**包外的可见性**

- 大写：是导出的，可以背外部的包访问
- 小写：包内使用的



Go推荐使用**驼峰式**命名

## 1.2 声明

Go语言四种类型的声明：

- `var`：变量
- `const`：常量
- `type`：类型
- `func`：函数实体



Go语言文件是以包的声明开始的，然后导入其他的依赖包

## 1.3 变量

变量声明的语法：

```go
var 变量名 类型 = 表达式
```

`类型`或`= 表达式`可以省略其中一个

- 如果省略了类型，那么就会通过表达式来推到类型信息

- 如果省略了表达式，那么将用零值初始化变量。各种类型对应的零值如下
  - 数值类型：`0`
  - 布尔类型：`false`
  - 字符串类型：`""`
  - 接口或者引用类型（`slice`、`map`、`chan`和函数）：`nil`
  - 数组或结构体等聚合类型：每个元素或者字段都对应该类型的零值



可以在一个声明语句中声明一组变量

```go
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```

## 1.4 类型

类型声明语句可以创造一个新的类型名称，用来创造一个新的逻辑概念

```go
type 类型名 底层类型
```

类型声明语句一般放在包一级，如果创造的类型首字母大写，那么外部包也可以使用

我们创造的类型各种行为都是它底层类型所控制的



对于每一个类型`T`，都有一个对应的类型转换操作`T(x)`，用于将`x`转为`T`类型（如果`T`是**指针**类型，需要用小括号包裹，如`(*int)(0)`）。只有两个类型的底层基础类型相同时，才允许进行转换，或者两者都是指向相同底层结构的指针类型



我们可以给我们定义的类型增加`String`方法，当使用`fmt`包打印该类型时会优先使用该类型对应的`String`方法返回的结果

## 1.5 包和文件

包的目的是为了支持模块化、封装、单独编译和代码重用

每个包对应一个独立的名字空间



每个go文件的包声明前紧跟着的是包注释。通常包注释的第一句应该是包的功能概要说明。**一个包通常只有一个源文件有包注释**。

## 1.6 作用域

声明语句的作用域指的是源代码中可以有效使用这个名字的范围



作用域和生命周期：

- 作用域：对应源代码的文本区域，是一个编译时属性
- 生命周期：程序运行时便浪存在的有效时间段，在此期间他能被其他程序的其他部分所应用，是一个运行时概念

# 二.基础数据类型

## 2.1 整型

Go提供了有符号和无符号类型的整数运算

- 有符号：`int8`、`int16`、`int32`、`int32`
- 无符号：`uint8`、`uint16`、`uint32`、`uint64`

分别对应8、16、32、64bit大小的整数

除此之外还有`int`、`uint`，他们跟CPU平台机器字大小有关。

Unicode字符`rune`类型和`int32`等价；`byte`与`int8`类型等价。

最后还有一种无符号整数类型`uintptr`，没有指定具体的bit大小但是足以容纳指针



无符号数往往只有在位运算或其它特殊的运算场景才会使用，就像bit集合、分析二进制文件格式或者是哈希和加密操作等。它们通常并不用于仅仅是表达非负数量的场合。



fmt使用技巧：

```go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)
// Output:
// 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

- %之后的`[1]`告诉`Printf`函数使用第一个操作数
- %之后的`#`告诉`Printf`在%o、%x或%X输出时生成0、0x或0X前缀

## 2.2 浮点数

Go提供了两种浮点数：`float32`和`float64`

`float32`提供大约6个十进制数的精度；`float64`提供约15个十进制数的精度



用`Printf`函数的`%g`参数打印浮点数，将采用更紧凑的表示形式打印，并提供足够的精度，但是对应表格的数据，使用`%e`（带指数）或`%f`的形式打印可能更合适。所有的这三个打印形式都可以指定打印的宽度和控制打印精度。

## 2.3 复数

Go语言提供了两种精度的复数类型：`complex64`和`complex128`，分别对应`float32`和`float64`两种浮点数精度

使用内建的`real`和`imag`函数分别返回复数的实部和虚部

```go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

如果一个**浮点数**或者一个**十进制整数**后面跟着一个`i`，它将构成一个复数的虚部，实部为0

```go
fmt.Println(1i * 1i) // "(-1+0i)", i^2 = -1
```

## 2.4 布尔型

布尔值有两种：`true`和`false`

## 2.5 字符串

一个字符串是一个不可改变的字节序列。字符串可以包含任意的数据

文本字符串通常被解释为采用UTF8编码的Unicode码点（rune）序列



内置的len函数可以返回一个字符串中的字节数目（不是rune字符数目），索引操作s[i]返回第i个字节的字节值，i必须满足0 ≤ i< len(s)条件约束。

```go
s := "hello, world"
fmt.Println(len(s))     // "12"
fmt.Println(s[0], s[7]) // "104 119" ('h' and 'w')
```

如果下标越界将会导致panic异常

```go
c := s[len(s)] // panic: index out of range
```



可以使用`:`来获取一个字串

```go
fmt.Println(s[0:5]) // "hello"
fmt.Println(s[:5]) // "hello"
fmt.Println(s[7:]) // "world"
fmt.Println(s[:])  // "hello, world"
```



`+`操作符将两个字符串链接构造一个新字符串

```go
fmt.Println("goodbye" + s[5:]) // "goodbye, world"
```



字符串可以用`==`和`<`进行比较；比较通过逐个字节比较完成的，因此比较的结果是字符串自然编码的顺序



因为字符串是不可修改的，因此尝试修改字符串内部数据的操作也是被禁止的：

```go
s[0] = 'L' // compile error: cannot assign to s[0]
```

这种不变性意味着两个字符串共享底层数据是安全的。像复制任何长度的字符串、一个字符串`s`对应字串切片`s[:7]`也能安全地共享内存，并且没有必要分配新的内存

## 2.6 常量

常量表达式的值是在**编译期**就进行计算。每种常量的潜在类型都是基础类：`boolean`、`string`或数字



一个常量的声明语句定义了常量的名字，和变量的声明语法类似，常量的值不可修改，这样可以防止在运行期被意外或恶意的修改

```go
const pi = 3.14159 // approximately; math.Pi is a better approximation
```

和变量声明一样，可以批量声明多个常量；这比较适合声明一组相关的常量：

```go
const (
    e  = 2.7182818284590452353602874713526624977572470
    pi = 3.1415926535897932384626433832795028841971693
)
```

如果是批量声明的常量，除了第一个外其它的常量右边的初始化表达式都可以省略，如果省略初始化表达式则表示使用前面常量的初始化表达式写法，对应的常量类型也一样的。例如：

```go
const (
    a = 1
    b
    c = 2
    d
)

fmt.Println(a, b, c, d) // "1 1 2 2"
```

除此之外还有iota常量生成器语法



所有常量的运算都可以在编译期完成，这样可以减少运行时的工作，也方便其他编译优化。当操作数是常量时，一些运行时的错误也可以在编译时被发现，例如整数除零、字符串索引越界、任何导致无效浮点数的操作等。

常量间的所有算术运算、逻辑运算和比较运算的结果也是常量，对常量的类型转换操作或以下函数调用都是返回常量结果：`len`、`cap`、`real`、`imag`、`complex`和`unsafe.Sizeof`

# 三.复合数据类型

## 3.1 数组

> 数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因为数组的长度是固定的，因此在Go语言中很少直接使用数组。而是使用能够动态增长和收缩的slice

- 数组的初始化：

  数组的长度必须是常量表达式，因为数组的长度需要在编译阶段就确定

  数组的每个元素都被初始化为元素类型对应的零值，对于数字类型来说就是0。我们也可以使用数组字面值语法用一组值来初始化数组

  ```go
  var q [3]int = [3]int{1, 2, 3} // 1, 2, 3
  var r [3]int = [3]int{1, 2} // 1, 2, 0
  ```

  在数组字面值中，如果在数组的长度位置出现的是`...`省略号，则表示数组的长度是根据初始化值的个数来计算

  ```go
  q := [...]int{1, 2, 3}
  fmt.Printf("%T\n", q) // "[3]int"（类型信息）
  ```

  初始化特定位置的元素，其他位置用零值初始化

  ```go
  r := [...]int{99: -1}
  ```

- 获取数组的长度

  数组的每个元素可以通过索引下标来访问，索引下标的范围是从0开始到数组长度减1的位置。内置的len函数将返回数组中元素的个数。

  ```go
  var a [3]int             // array of 3 integers
  fmt.Println(a[0])        // print the first element
  fmt.Println(a[len(a)-1]) // print the last element, a[2]
  
  // 打印索引和元素
  for i, v := range a {
      fmt.Printf("%d %d\n", i, v)
  }
  
  // 只打印元素
  for _, v := range a {
      fmt.Printf("%d\n", v)
  }
  ```

- 数组之间的比较

  只有两个数组所有元素都相等时两个数组才是相等的

## 3.2 Slice

> `Slice`（切片）代表变长的序列，序列中每个元素的类型都是相同的
>
> slice类型一般写作`[]T`，其中`T`代表slice中元素的类型
>
> slice的语法和数组的很相似，只是没有固定长度

- 底层结构

  数组和slice联系紧密。slice底层就是一个数组，它提供了访问数组子序列元素的功能。

  一个slice由三部分组成：指针、长度和容量。指针指向第一个slice元素对应底层数组元素的地址；长度对应slice中元素的数目，长度不超过容量；容量一般是从slice开始的位置到底层数据的结尾位置。内置的`len`和`cap`函数分别返回slice的长度和容量。

  多个slice之间可以共享底层的数据，并且引用的数组部分区间可能重叠。

  因为slice的值包含指向第一个slice元素的指针，因此向数组传递slice将允许在函数内部修改底层数组的元素。换句话说，复制一个slice只是对底层的数组创建了一个新的slice别名，因此如果像函数传递slice将允许函数内部修改底层数组元素。这里我们定义了一个slice的反转函数

  ```go
  // reverse reverses a slice of ints in place.
  func reverse(s []int) {
      for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
          s[i], s[j] = s[j], s[i]
      }
  }
  ```

  然后我们进行反转

  ```go
  a := [...]int{0, 1, 2, 3, 4, 5}
  reverse(a[:])
  fmt.Println(a) // "[5 4 3 2 1 0]"
  ```

- 创建

  - 方法1，显式引用数组

    ```go
    arr := [3]int{1, 2, 3}
    slice1 := arr[:]
    ```

  - 方法2，使用make方法，隐式引用数组

    使用`make`底层是创建了一个匿名的数组

    - `make([]T, len)`

      ```go
      slice1 := make([]int, 3)
      ```

      底层slice是引用的是整个数组，数组长度为`len`，dlice的长度和容量都为`len`

    - `make([]T, len, cap)`

      ```go
      slice2 := make([]int, 3, 5)
      ```

      底层数组长度为`cap`，slice只引用了数组的前`len`个元素，但是容量包含了整个数组，额外的元素可以留给未来增长使用

  这里定义一个月份数组

  ```go
  months := [...]string{1: "January", /* ... */, 12: "December"}
  ```

  因为月份是从1月份开始的，所以我们这儿声明数组的时候跳过了0，第0个元素会被自动初始化位空字符串

  我们可以使用切片操作`s[i:j]`创造一个slice，该slice引用s的从第`i`个元素开始到第`j-1`个元素的子序列，新的slice将只有`j-i`个元素。如果i位置的索引被省略将使用0代替，如果j位置的索引被省略，那么将使用`len(s)`代替。

  ```go
  Q2 := months[4:7]
  summer := months[6:9]
  fmt.Println(Q2)     // ["April" "May" "June"]
  fmt.Println(summer) // ["June" "July" "August"]
  ```

  如果切片的操作超出了`cap(s)`的上限将导致一个panic异常，但是超出len(s)就会扩展slice

- 比较

  与数组不同的是slice不能比较，不能用`==`来判断两个slice是否含有全部相等元素。

  但是标准库提供了高度优化的`bytes.Equal`函数来判断两个字节型slice（[]byte）是否相等，但是对于其他类型的slice我们需要自己展开每个元素进行比较

  ```go
  func equal(x, y []string) bool {
      if len(x) != len(y) {
          return false
      }
      for i := range x {
          if x[i] != y[i] {
              return false
          }
      }
      return true
  }
  ```

  slice唯一合法的比较是和nil比较，如

  ```go
  if summer == nil { /* ... */ }
  ```

  一个零值的slice等于nil，一个nil值的slice没有底层数组,我们可以通过`[]int(nil)`类型转换表达式来生成一个对应slice类型的nil值。此时slice的长度和容量都为0，但是也有非nil的slice长度和数组也是0，例如`[]int{}`或`make(int[],3)[3:]`。

  ```go
  var s []int    // len(s) == 0, s == nil
  s = nil        // len(s) == 0, s == nil
  s = []int(nil) // len(s) == 0, s == nil
  s = []int{}    // len(s) == 0, s != nil
  ```

  如果需要判断一个slice是否是空的，使用len(s) == 0来判断，而不应该用s == nil来判断。

## 3.3 Map

哈希表是一个无序的`key/value`对集合，其中所有的`key`都是不同的

通过给定的`key`可以在**常数时间复杂度**内**检索、更新或删除**对应的`value`



在Go语言中，一个map就是一个哈希表的引用，map类型可以写为`map[K]V`，所有的key和value类型都是相同的，key和value的类型可以不同。其中key必须是支持`==`比较运算符的数据类型，map可以通过测试key是否相等来判断是否已经存在了。value没有任何限制。



- 创建

  - 通过make函数能够创建一个map

    ```go
    ages := make(map[string]int])
    ```

  - 通过map字面值的语法创建map，同时赋予初值

    ```go
    ages := map[string]int{
        "alice":   31,
        "charlie": 34,
    }
    ```

    这相当于

    ```go
    ages := make(map[string]int)
    ages["alice"] = 31
    ages["charlie"] = 34
    ```

    因此另一种创建空map的方式是`map[string]int{}`

- 取值

  通过key来访问map的value

  ```go
  ages["alice"] = 32
  fmt.Println(ages["alice"]) // "32"
  ```

  如果map中没有个这个元素，那么会返回value的**零值**

  因为有可能元素本身就是零值所以如果想要真正知道是否存在该元素需要用到第二个返回值，他用于说明元素是否真的存在，一般命名为ok

  ```go
  age, ok := ages["bob"]
  if !ok{/*bob这个key不在map中；age == 0*/}
  ```

  我们可以将这两个结合起来使用：

  ```go
  if age, ok := ages["bob"]; !ok { /* ... */ }
  ```

  

  如果想要遍历map中全部的key/value对的话，可以使用range风格的for循环实现

  ```go
  for name, age := range ages {
      fmt.Printf("%s\t%d\n", name, age)
  }
  ```

  map的迭代顺序是不确定的，如果想要按照顺序访问key/value对需要显式地对key进行排序，可以使用sort包的`Strings`函数对字符串slice进行排序

  ```go
  import "sort"
  
  var names []string
  for name := range ages {
      names = append(names, name)
  }
  sort.Strings(names)
  for _, name := range names {
      fmt.Printf("%s\t%d\n", name, ages[name])
  }
  ```

- 删除

  通过内置的`delete`函数能够删除元素

  ```go
  delete(ages, "alice")
  ```

  删除操作是安全的，即使map中没有该元素

- 比较

  和slice一样，map之间不能进行相等的比较，唯一的例外是和nil进行比较，如果要判断两个map是否包含相同的key和value，需要通过一个循环来实现

  ```go
  func equal(x, y map[string]int) bool {
      if len(x) != len(y) {
          return false
      }
      for k, xv := range x {
          if yv, ok := y[k]; !ok || yv != xv {
              return false
          }
      }
      return true
  }
  ```

- set

  Go中没有提供set类型，但是map中的key是不相同，可以用map实现类似set的功能

  ```go
  func main() {
      seen := make(map[string]bool) // a set of strings
      input := bufio.NewScanner(os.Stdin)
      for input.Scan() {
          line := input.Text()
          if !seen[line] {
              seen[line] = true
              fmt.Println(line)
          }
      }
  
      if err := input.Err(); err != nil {
          fmt.Fprintf(os.Stderr, "dedup: %v\n", err)
          os.Exit(1)
      }
  }
  ```

## 3.4 结构体

> 结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。每个值称为结构体的成员。

- 定义

  ```go
  type struct_name struct {
    ...
  }
  ```

  通常我们一行对应一个结构体成员，但是如果相邻的成员类型相同的话可以合并成一行

  ```go
  type User struct {
    age						int
    Name, Address string
  }
  ```

  一个命名为S的结构体类型不能再包含S类型的成员，但是可以包含*S指针类型的成员，这让我们可以创建递归的数据结构，比如链表、树结构

- 实例化

  - 基本的实例化形式

    ```go
    var ins T
    
    ins.xxx = xxx
    ```

  - 创建指针类型的结构体

    ```go
    ins := new(T)
    ```

    - T为类型，可以为结构体、整型、字符串等
    - ins类型为*ins，属于指针

  - 取结构体的地址实例化

    使用该方法可以在实例化的时候就进行属性的初始化

    ```go
    ins := &T{}
    ```

    - T为结构体类型
    - ins为结构体实例，类型为*T，是指针类型

- 获取成员

  可以通过`.`来获取结构体的成员变量，也可以直接对成员进行赋值

  ```go
  user.Name = "Tom"
  ```

  或者是对成员取地址，通过指针来访问

  ```go
  name := &user.Name
  *name = "Jack"
  ```

  `.`也可以和指向结构体的指针一起工作

  ```go
  var user1 *User = &user
  
  user1.Name = "Kitty"
  // 上面的赋值语句是一个语法糖，相当于以下语句
  (*user1).Name = "kitty"
  ```

  

  下面编写一个函数用来创建我们的User对象

  ```go
  func getUser() *User {
  	var user User
  	return &user
  }
  
  func main() {
  	getUser().Name = "Kite"
  }
  ```

  如果`getUser`函数的返回值改为`User`，那么赋值语句就会报错，因为函数调用返回的是值，不是一个可取地址的变量

## 3.5 JSON

> JavaScript对象表示法（JSON）是一种用于发送和接收结构化信息的标准协议

只有导出的结构体成员才会被编码和解码

[Json自定义序列化方式](https://colobu.com/2020/03/19/Custom-JSON-Marshalling-in-Go/)

- JSON编码

  有如下结构体：

  ```go
  type Person struct {
      Name string `json:"name, omitempty"`
      Age int `json:"age"`
  }
  ```

  成员变量后用反引号包裹的是Tag，Tag可以是任意的字符串面值，通常是一系列用空格分隔的`key:"value"`值对序列。

  json开头键名对应的值用于控制encoding/json包的编码和解码的行为，并且encoding/...下面其它的包也遵循这个约定。

  `value`的第一部分指定JSON对象的名字，除此之外还可以填写其他选项

  - `omitempty`：表示当Go语言结构体成员为空或零值时不生成JSON对象（false也为零值）
  - `json:"-,"`：不被序列化

  编码的两种方式：

  - ``json.Marshal(&v)``

    - 紧凑编码

      ```go
      person := Person{"张三", 24}
      data, err := json.Marshal(&person)
      if err == nil {
        // 返回的是字节数组 []byte
        fmt.Println("json.Marshal 编码结果: ", string(data))
      }
      ```

    - 缩进编码

      ```go
      person := Person{"张三", 24}
      data, err := json.MarshalIndent(&person, "", "    ")
      if err != nil {
          log.Fatalf("JSON marshaling failed: %s", err)
      }
      // 返回的是字节数组 []byte
      fmt.Println("json.Marshal 编码结果: ", string(data))
      ```

      该函数额外两个参数表示每一行输出的前缀和每一个层级的缩进

  - `json.NewEncoder(<Writer>).encode(v)`

    ```json
    person := Person{"王五", 30}
    // 编码结果暂存到 buffer
    bytes1 := new(bytes.Buffer)
    _ = json.NewEncoder(bytes1).Encode(person)
    if err == nil {
      fmt.Print("json.NewEncoder 编码结果: ", string(bytes1.Bytes()))
    }
    ```

- JSON解码

  在解码会自动忽略没有定义的json参数

  - ``json.Unmarshal([]byte, &v)``

    ```json
    str := `{"name":"李四","age":25}`
    // json.Unmarshal 需要字节数组参数, 需要把字符串转为 []byte 类型
    bytes1 := []byte(str) // 字符串转换为字节数组
    var person Person    // 用来接收解码后的结果
    if json.Unmarshal(bytes1, &person) == nil {
      fmt.Println("json.Unmarshal 解码结果: ", person.Name, person.Age)
    }
    ```

  - `json.NewDecoder(<Reader>).decode(&v)`

    ```go
    str := `{"name":"赵六","age":28}`
    var person Person
    // 创建一个 string reader 作为参数
    err = json.NewDecoder(strings.NewReader(str)).Decode(&person)
    if err == nil {
      fmt.Println("json.NewDecoder 解码结果: ", person4.Name, person4.Age)
    }
    ```

# 四.函数

## 4.1 函数声明