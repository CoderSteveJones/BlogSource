---
title: video标签
date: 2016-09-25 09:22:06
categories: 
	- HTML笔记
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>video标签</title>
</head>
<body>
<!--
1.什么是video标签?
作用: 播放视频

格式:
<video src="">
</video>

video标签的属性
src: 用于告诉video标签需要播放的视频地址
autoplay: 用于告诉video标签是否需要自动播放视频
controls: 用于告诉video标签是否需要显示控制条
poster: 用于告诉video标签视频没有播放之前显示的占位图片
loop: 一般用于做广告视频, 用于告诉video标签视频播放完毕之后是否需要循环播放
preload: 预加载视频, 但是需要注意preload和autoplay相冲, 如果设置了autoplay属性, 那么preload属性就会失效
muted:静音
width/height: 和img标签中的一模一样
-->

<!--<video src="images/sb1.webm" autoplay="autoplay" controls="controls"></video>-->
<!--<video src="images/sb1.webm"  controls="controls" poster="images/NJ.jpg"></video>-->
<video src="images/sb1.webm"  autoplay="autoplay" loop="loop" muted="muted" width="800px"></video>
</body>
</html>
```


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>video标签的第二种格式</title>
</head>
<body>
<!--
1.格式:
<video>
    <source src="" type=""></source>
    <source src="" type=""></source>
</video>

2.第二种格式存在的意义:
由于视频数据非常非常的重要, 所以五大浏览器厂商都不愿意支持别人的视频格式, 所以导致了没有一种视频格式是所有浏览器都支持的
这个时候W3C为了解决这个问题, 所以推出了第二个video标签的格式

video标签的第二种格式存在的意义就是为了解决浏览器适配问题
video 元素支持三种视频格式, 我们可以把这三种格式都通过source标签指定给video标签, 那么以后当浏览器播放视频时它就会从这三种中选择一种自己支持的格式来播放

3.注意点:
3.1当前通过video标签的第二种格式虽然能够指定所有浏览器都支持的视频格式, 但是想让所有浏览器都通过video标签播放视频还有一个前提条件, 就是浏览器必须支持HTML5标签, 否则同样无法播放
3.2在过去的一些浏览器是不支持HTML5标签的, 所以为了让过去的一些浏览器也能够通过video标签来播放视频, 那么我们以后可以通过一个JS的框架叫做html5media来实现
-->
<video autoplay="autoplay" controls="controls">
    <source src="images/sb1.webm" type="video/webm"></source>
    <source src="images/sb1.mp4" type="video/mp4"></source>
    <source src="images/sb1.ogg" type="video/ogg"></source>
</video>
</body>
</html>
```

