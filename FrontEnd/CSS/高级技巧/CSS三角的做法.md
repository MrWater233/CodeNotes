# 基本用法

```css
.box {
    width: 0;
    height: 0;
    /* 以下两行代码是为了兼容性考虑，并不是必须的 */
    line-height: 0;
    font-size: 0;
    border: 50px solid transparent;
    border-left-color: pink;
}
```

以上代码就可以做出一个向右的三角了

原理：四个边框给了宽度以后本质上就是四个三角形，只要显示其中一个边框，便可得到一个三角形

应用：

- 在大盒子中放入一个小三角，再通过定位移动三角到大盒子边缘就可以做出类似消息框的样式

# 高级用法

做出一个直角三角形

```css
.box {
    width: 0;
    height: 0;
    /* 以下两行代码是为了兼容性考虑 */
    line-height: 0;
    font-size: 0;
    /* 上边框边框调大 */
    border-top: 100px solid transparent;
    /* 下面两个属性也可以省略不写 */
    border-bottom: 0px solid blue;
    border-left: 0px solid purple;
    border-right: 50px solid black;
}
```

也可以采用以下写法

```css
.box {
    width: 0;
    height: 0;
    /* 以下两行代码是为了兼容性考虑 */
    line-height: 0;
    font-size: 0;
    /* 只保留右遍边框有颜色 */
    border-color: transparent black transparent transparent;
    /* 样式都是solid */
    border-width: 100px 50px 0 0;
    /* 上边框大，右边框稍小，让上边框拉高有边框，其余边框为0 */
    border-style: solid;
}
```

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200720120829.png)

## 例子

京东秒杀框

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200720164334.png)

```css
* {
    margin: 0;
    padding: 0;
}
.box {
    width: 195px;
    height: 28px;
    margin: 100px auto;
    border: 1px solid #e1251b;
    line-height: 28px;
}
.miaosha {
    position: relative;
    float: left;
    width: 115px;
    height: 100%;
    color: #fff;
    background-color: #e1251b;
    text-align: center;
    font-weight: 700;
    margin-right: 10px;
}
.miaosha i {
    position: absolute;
    right: 0;
    top: 0;
    width: 0;
    height: 0;
    border-color: transparent #fff transparent transparent;
    border-style: solid;
    border-width: 28px 10px 0 0;
}
```

```html
<body>
  <div class="box">
    <span class="miaosha">
      ￥500
      <i></i>
    </span>
    <span>￥1000</span>
  </div>
</body>
```

