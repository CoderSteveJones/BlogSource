---
title: CSS元素的显示模式
date: 2016-11-01 09:22:06
categories: 
	- CSS笔记
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CSS元素的显示模式</title>
    <style>
        div{
            background: red;
            width: 200px;
            height: 200px;
        }
        span{
            background: blue;
            width: 200px;
            height: 200px;
        }
        img{
            width: 50px;
            height: 50px;
        }
    </style>
</head>
<body>
<!--
在HTML中HTML将所有的标签分为两类, 分别是容器级和文本级
在CSS中CSS也将所有的标签分为两类, 分别是块级元素和行内元素

1.什么是块级元素, 什么是行内元素?
块级元素会独占一行
行内元素不会独占一行

容器级的标签
div h ul ol dl li dt dd ...
文本级的标签
span p buis stong em ins del ...

块级元素
p div h ul ol dl li dt dd
行内元素
span buis strong em ins del

2.块级元素和行内元素的区别?
2.1块级元素
独占一行
如果没有设置宽度, 那么默认和父元素一样宽
如果设置了宽高, 那么就按照设置的来显示

2.2行内元素
不会独占一行
如果没有设置宽度, 那么默认和内容一样宽
行内元素是不可以设置宽度和高度的

2.3行内块级元素
为了能够让元素既能够不独占一行, 又可以设置宽度和高度, 那么就出现了行内块级元素
-->

<div>我是div</div>
<p>我是段落</p>
<h1>我是标题</h1>
<hr>
<span>我是span</span>
<b>我是加粗</b>
<strong>我是强调</strong>
<hr>
<img src="images/picture.jpg" alt="">
<img src="images/picture.jpg" alt="">
</body>
</html>
```


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CSS元素显示模式转换</title>
    <style>
        div{
            display: inline;
            background: red;
            width: 200px;
            height: 200px;
        }
        span{
            display: block;
            background: green;
            width: 200px;
            height: 200px;
        }
        .cc{
            background: blue;
            width: 200px;
            height: 200px;
            display: inline-block;
        }
    </style>
</head>
<body>
<!--
1.如何转换CSS元素的显示模式?
设置元素的display属性

2.display取值
block 块级
inline 行内
inline-block 行内块级

3.快捷键
di display: inline;
db display: block;
dib display: inline-block;
-->

<div>我是div</div>
<div>我是div</div>

<span>我是span</span>
<span>我是span</span>

<p class="cc">我是段落</p>
<b class="cc">我是加粗</b>

</body>
</html>
```


