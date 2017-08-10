---
title: a标签的伪类选择器
date: 2016-11-21 09:22:06
categories: 
	- CSS笔记
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>a标签的伪类选择器</title>
    <style>
        /*
        a:link{
            color: tomato;
        }
        a:visited{
            color: green;
        }
        a:hover{
            color: orange;
        }
        a:active{
            color: pink;
        }
        */
        a{
            // 简写格式
            color: green;
        }
        /*a:link{*/
            /*color: green;*/
        /*}*/
        /*a:visited{*/
            /*color: green;*/
        /*}*/
        a:hover{
            color: orange;
        }
        a:active{
            color: pink;
        }
    </style>
</head>
<body>
<!--
1.通过我们的观察发现a标签存在一定的状态
1.1默认状态, 从未被访问过
1.2被访问过的状态
1.3鼠标长按状态
1.4鼠标悬停在a标签上状态

2.什么是a标签的伪类选择器?
a标签的伪类选择器是专门用来修改a标签不同状态的样式的

3.格式
:link 修改从未被访问过状态下的样式
:visited 修改被访问过的状态下的样式
:hover 修改鼠标悬停在a标签上状态下的样式
:active 修改鼠标长按状态下的样式

4.注意点
4.1a标签的伪类选择器可以单独出现也可以一起出现
4.2a标签的伪类选择器如果一起出现, 那么有严格的顺序要求
编写的顺序必须要个的遵守爱恨原则 love hate
4.3如果默认状态的样式和被访问过状态的样式一样, 那么可以缩写
-->
<a href="http://www.taobao.com">taobao</a>
<a href="http://www.jd.com">jd</a>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>a标签的伪类选择器练习</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        ul{
            width: 960px;
            height: 40px;
            background-color: green;
            margin: 100px auto;
        }
        ul li{
            list-style: none;
            width: 120px;
            height: 40px;
            float: left;
            background-color: purple;
            text-align: center;
            line-height: 40px;
        }
        /*
        1.在企业开发中编写a标签的伪类选择器最好写在标签选择器的后面
        2.在企业开发中和a标签盒子相关的属性都写在标签选择器中(显示模式/宽度/高度/padding/margin)
        3.在企业开发中和a标签文字/背景相关的都写在伪类选择器中
        */
        ul li a{
            width: 120px;
            height: 40px;
            display: inline-block;
        }
        ul li a:link{
            background-color: pink;
            color: white;
            text-decoration: none;
        }
        ul li a:hover{
            color: red;
            background-color: #ccc;
        }
        ul li a:active{
            color: yellow;
        }
    </style>
</head>
<body>
<ul>
    <li><a href="#">我是导航</a></li>
    <li><a href="#">我是导航</a></li>
    <li><a href="#">我是导航</a></li>
    <li><a href="#">我是导航</a></li>
    <li><a href="#">我是导航</a></li>
    <li><a href="#">我是导航</a></li>
    <li><a href="#">我是导航</a></li>
    <li><a href="#">我是导航</a></li>
</ul>
</body>
</html>
```

