---
layout: post
title: "bugku ctf writeup1"
date: 2018-03-07 22：40
categories: jekyll update
---

## 前言
&emsp;&emsp;此篇文章记录bugku中作者觉得比较`有价值`的writeup。

## MISC
### linux基础问题
将<a href="http://120.24.86.145:8002/misc/1.tar.gz">压缩包</a>下载后可以发现一个flag文件，拿到`010Editor`查找flag却没有任何发现，后来使用`winhex`查找flag，发现了flag.txt，但这并没有什么用，所以再次查找key，此时就能找到flag。
flag：key{feb81d3834e2423c9903f4755464060b}
<div align="center">
    <img src="/images/posts/bugku/1.png" >  
</div>



