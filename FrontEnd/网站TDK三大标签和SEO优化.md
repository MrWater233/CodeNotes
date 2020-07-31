> `TDK`顾名思义就是`titile`，`description`，`keyword`
>
> `SEO`就是**搜索引擎优化**，利用搜索引擎规则提高网站排名

# 一.title网站标题

> titile具有不可替代性，是搜索引擎了解网页入口和对网页主题归属的判断点

建议：网站名（产品名）-网站介绍

例如：

```html
<title>小米商城 - 小米10 Pro、Redmi 10X、小米MIX Alpha，小米电视官方网站</title>
```

# 二.description网站说明

> 简要说明网站是做什么的

description作为网站的总体业务和主题概括，多采用"我们是..."，"我们提供..."，"xxx网作为..."，"电话：010..."之类的语句

例如：

```html
<meta name="description" content="京东JD.COM-专业的综合网上购物商城,销售家电、数码通讯、电脑、家居百货、服装服饰、母婴、图书、食品等数万个品牌优质商品.便捷、诚信的服务，为您提供愉悦的网上购物体验!">
```

# 三.keyword关键字

> keyword是页面关键字，最好限制6-8个关键字，关键字用**英文逗号**隔开

例如：

```html
<meta name="Keywords" content="网上购物,网上商城,手机,笔记本,电脑,MP3,CD,VCD,DV,相机,数码,配件,手表,存储卡,京东">
```

# 四.LOGO的SEO优化

1. logo`div`里面先放一个`<h1>`标签，目的是提权
2. `<h1>`里面再放一个链接，可以返回首页，将logo设为链接的背景即可
3. 为了搜索引擎的收录，链接里要放文字（网站名称），但是文字不显示
   - 方法1：用`text-indent`将文字移到盒子外部（`text-indent:-999ppx`），然后设置`overflow:hidden`
   - 方法2：直接给`font-size:0`
4. 给链接一个`title`属性，这样鼠标放到logo上就能看到文字

```html
<div class="logo">
    <h1><a href="index.html" title="品优购商城">品优购商城</a></h1>
</div>
```

