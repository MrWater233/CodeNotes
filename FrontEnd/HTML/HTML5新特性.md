# 语义化标签

- `<header>`：头部标签
- `<nav>`：导航标签
- `<article>`：内容标签
- `<section>`：定义文本某个区域（大号的`<div>`）
- `<aside>`：侧边栏标签
- `<footer>`：尾部标签

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200721111933.png)

# 多媒体标签

chrome将音频和视频的自动播放给禁止了

## 音频`<audio>`

> 一般使用mp3格式的音频，兼容性较好

### 语法

一般写法：

```html
<audio src="文件地址" controls="controls"></audio>
```

兼容性写法（如果不支持mp3就会播放ogg格式的音频，如果再不支持就会显示文字）：

```css
<audio controls="controls">
	<source src="xxx.mp3" type="audio/mpeg"/>
	<source src="xxx.ogg" type="audio/ogg"/>
	您的浏览器不支持audio标签
</audio>
```

### 属性

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200721114444.png)

## 视频`<video>`

> 一般使用mp4格式的视频，兼容性较好

### 语法

一般写法：

```html
<vedio src="文件地址" controls="controls"></vedio>
```

兼容性写法（如果不支持mp4就会播放ogg格式的视频，如果再不支持就会显示文字）：

```css
<vedio controls="controls">
	<source src="xxx.mp4" type="vedio/mp4"/>
	<source src="xxx.ogg" type="vedio/ogg"/>
	您的浏览器不支持vedio标签
</vedio>
```

### 属性

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200721113725.png)

# input表单

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200721115105.png)

- 验证的时候必须添加`<form>`表单域，在提交的时候进行验证

# 表单属性

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200721125308.png)

可以通过以下CSS设置`placeholder`里面的字体

```css
input::placeholder {
    color: pink;
}
```

