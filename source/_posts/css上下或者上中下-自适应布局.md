---
title: css上下或者上中下 自适应布局
date: 2018-10-10 19:10:23
tags: css

---



方法一：定位

```html
<div class="header">头部</div>
<div class="content">内容</div>
<div class="footer">底部</div>
```

```css
*{
	margin: 0;
	padding: 0;
}
div{
	text-align: center;
	font-size: 30px;
}
.header,.footer{
	width: 100%;
	height: 100px;
	line-height: 100px;
	background-color: red;
}
.content{
	width: 100%;
	position: absolute;
	top: 100px;
	bottom:100px;
	background-color: yellow;
}
.footer{
	position: absolute;
	bottom: 0px;
}
```

方法二：flex

```html
<div class="content">
  <div class="header">header</div>
  <div class="center">center</div>
  <div class="footer">footer</div>
</div>
```

```css
html,body{
  height:100%;
}
.content{
  display: flex;
  flex-direction: column;
  height: 100%;
}

.header{
  background:red;
  height:100px;
  width:100%;
}

.center{
  background: blue;
  flex: 1;
  height: 100%;
}

.footer{
  background:green;
  height:100px;
  width:100%;
}
```

