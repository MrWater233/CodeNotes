# HTML

## 元素分类

- block：独占一行，能够改变宽高（div）
- inline：能够跟其他元素同一行，能够被拆分，不能改变宽高（span）
- lnline-block：能够跟其他元素同一行，不能被拆分，能改变宽高（section）

## 元素

- `label[for] `：通过for指定其他标签的`id`这样将两个标签关联在一起

# CSS

## 选择器

### 选择器分类

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200401221641.png)

#### 伪类选择器

>  表示元素某种状态时的样式

- `a:link{}`：表示普通链接（没有访问过的连接）
- `a:visited{}`：表示访问过的连接
- `:hover{}`：表示鼠标移入状态
- `:active{}`：表示被点击的状态
- `:force{}`：获取焦点的状态
- `::selection{}`：为选中的内容设置样式

#### 伪元素选择器

> 表示元素中的一些特殊位置

- `::first-letter`：第一个字母

- `::first-line`：第一行

- `::before`：表示元素最前面的部分（开始标签标签和内容开始之间）

- `::after`：表示元素最后面的部分（结束标签和内容结束之间）

  一般`::before`和`::after`需要结合`content`样式添加内容（在`::before`或者`::after`区域添加内容，这些内容无法选中）一起使用

### 选择器的解析

> 浏览器从右向左解析，这样可以加快解析效率

## 非布局样式

### 字体族

> `font-family=字体族（无引号）`

- serif：衬线字体，有笔锋，如宋体
- sans-serif：非衬线字体，笔画起收规则，如黑体
- monospace：等宽字体，如编程所用的字体
- cursive：手写体
- fantasy：花体

### 表格

- 表格布局`table-layout`
  - `automatic`：单元格的大小由td的内容宽度决定
  - `fixed`：单元格的大小由td上设置的宽度决定
- 表格边框`border-collapse`
  - `separate`：边框分离
  - `collapse`：边框合并

### 边框

-  边框风格`border-style`
  - `solid`：实线
  - `dotted`：点状
  - `dashed`：虚线
  - `double`：双线
- `border:宽度 风格 颜色`

### 内边距

- `padding:上 右 下 左`

  如果**缺少左**内边距的值，则**使用右**内边距的值。
  如果**缺少下**内边距的值，则**使用上**内边距的值。
  如果**缺少右**内边距的值，则**使用上**内边距的值。

- `padding-方位`

## 布局样式

### 绝对定位

> `position:absolute`
>
> 通过指定`left`、`top`绝对定位一个位置
>
> 绝对定位是基于最近的一个定位了的父容器

### 相对定位

> `position:relatve`
>
> 与绝对定位不同的是，相对定位不会把该元素从原文档删除掉，而是在原文档的位置的基础上，移动一定的距离
>
> 相对于其正常位置进行定位

### 浮动

> 浮动的框可以向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止
>
> `float:[left/right]`

不允许左边右浮动元素：`clear:left`

### 显示方式

- `display:none`：使得被选择的元素隐藏，并且不占用原来的位置
- `display:block`： 表示块级元素，块级元素会自动在前面和后面加上换行，并且在其上的width和height也能够生效
- `display:inline`：表示内联元素，内联元素前后没有换行，并且在其上的width和height也无效。 其大小由其中的内容决定
- `display:inline-block`：表示元素处于同一行，同时还能指定大小

### 居中

- `text-align:center`：内容水平居中
- `margin:0 auto`：元素水平居中，块级元素需要设置`width`
- `line-height`：适合单独一行垂直居中，可以使用内边距`padding: 30 0`代替

