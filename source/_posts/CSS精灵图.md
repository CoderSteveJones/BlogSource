---
title: CSS精灵图
date: 2016-11-08 09:22:06
categories: 
	- CSS笔记
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CSS精灵图</title>
    <style>
        .box{
            width: 86px;
            height: 28px;
            background-image: url(images/weibo.png);
            background-position: -425px -200px;
        }
    </style>
</head>
<body>
<!--
1.什么是CSS精灵图
CSS精灵图是一种图像合成技术

2.CSS精灵图作用
可以减少请求的次数, 以及可以降低服务器处理压力

3.如何使用CSS精灵图
CSS的精灵图需要配合背景图片和背景定位来使用
-->
<div class="box"></div>
</body>
</html>
```

