> 类选择器（`.xxx`），属性选择器（`[xxx]`），伪类选择器（`:xxx`）的权重都是10

# 属性选择器

> 可以根据元素特定的属性来选择元素，这样就不需要借助类选择器或者ID选择器了

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200721133709.png)

- 值的引号可以不加
- `E[att]`权重=1+10=11

# 结构伪类选择器

> 主要是根据文档结构来选择元素，常用于父元素选择子元素

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200721134256.png)

1. `nth-child(n)`中的`n`可以为数字，关键字和公式

   - 数字：选择第n个子元素，从1开始

   - 关键字：`even`偶数，`odd`奇数

   - 公式：用`n`来代表未知数，`n`从0开始，每次增加1

     ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200721140054.png)

2. `nth-child(n)`和`nth-of-type(n)`的区别

   - `nth-child(n)`会把**所有子盒子**排列序号，执行时先找到第n个孩子，再看看是否和E匹配，需要都匹配才生效
   - `nth-of-type(n)`会把**指定标签类型的子盒子**排列序号。先去匹配E，再找到E类型的第n个孩子

# 伪元素选择器

> 利用CSS创建新标签元素而不需要HTML标签，简化HTML结构

| 选择器     | 作用                     |
| ---------- | ------------------------ |
| `::before` | 在元素内部的前面插入内容 |
| `::after`  | 在元素内部的后面插入内容 |

- `before`和`after`会创建一个**行内元素**，但是在文档树中是找不到的
- `before`和`after`必须有`cntent`属性
- 伪元素选择器权重跟标签选择器一样是1