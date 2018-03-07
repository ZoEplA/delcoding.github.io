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
&emsp;&emsp;将<a href="http://120.24.86.145:8002/misc/1.tar.gz">压缩包</a>下载后可以发现一个flag文件，拿到`010Editor`查找flag却没有任何发现，后来使用`winhex`查找flag，发现了flag.txt，但这并没有什么用，所以再次查找key，此时就能找到flag。
flag：key{feb81d3834e2423c9903f4755464060b}
<div align="center">
    <img src="/images/posts/bugku/1.png" >  
</div>

### 中国菜刀
&emsp;&emsp;下载压缩文件后可以发现是`wireshark的捕获包`，使用wireshark分析，追踪TCP流可以发现有一段流比较奇怪。
<div align="center">
    <img src="/images/posts/bugku/2.png" >  
</div>
&emsp;&emsp;将传输的数据拿去进行base64解密可以得知这个流是读取flag的数据流。
<div align="center">
    <img src="/images/posts/bugku/3.png" >  
</div>
&emsp;&emsp;所以将这个蓝色部分的数据提取出来
<div align="center">
    <img src="/images/posts/bugku/4.png" >  
</div>
&emsp;&emsp;调整数据流后再选择解码类型。
<div align="center">
    <img src="/images/posts/bugku/5.png" >  
</div>
&emsp;&emsp;所以flag：key{8769fe393f2b998fa6a11afe2bfcd65e}

### 这么多数据包
&emsp;&emsp;下载后用wireshark打开，将进度条下拉到`灰色（传输稳定）`的状态，选择一条，然后追踪TCP流，
<div align="center">
    <img src="/images/posts/bugku/6.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/7.png" width="70%" />  
</div>
&emsp;&emsp;调节流，在1735就有发现，将那串字符进行base64解码后可以发现。
<div align="center">
    <img src="/images/posts/bugku/8.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/9.png" >  
</div>
&emsp;&emsp;所以flag：CCTF{do_you_like_sniffer}

### Linux2
&emsp;&emsp;题目描述：
```
给你点提示吧：key的格式是KEY{}
题目地址：链接: http://pan.baidu.com/s/1skJ6t7R 密码: s7jy
```
&emsp;&emsp;将文件下载后使用binwalk、foremost可以分离出一个看似是flag的图片，但提交却是错误。无奈只能换种思路，自己想了挺久没想出来，后来查了下writeup才发现正确的解题方式：`使用strings brave分析文件`。
<div align="center">
    <img src="/images/posts/bugku/10.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/11.png" >  
</div>
&emsp;&emsp;所以flag：KEY{24f3627a86fc740a7f36ee2c7a1c124a}

## WEB
### sql注入
&emsp;&emsp;题目描述：
<div align="center">
    <img src="/images/posts/bugku/12.png" >  
</div>
&emsp;&emsp;在测试的时候发现不管'还是"都无法判断是否存在注入，查看源代码发现页面使用`gb2312`，所以考虑`宽字节注入`，。
<div align="center">
    <img src="/images/posts/bugku/13.png" >  
</div>
```
POC: id=1%df%27 and 1=1 -- +
```
<div align="center">
    <img src="/images/posts/bugku/14.png" >  
</div>
&emsp;&emsp;但在查询string的值时发生如下错误
<div align="center">
    <img src="/images/posts/bugku/15.png" >  
</div>
&emsp;&emsp;尝试将`key`使用反引号包围，即\`key\`。最终得到flag。
<div align="center">
    <img src="/images/posts/bugku/16.png" >  
</div>
&emsp;&emsp;所以flag：KEY{54f3320dc261f313ba712eb3f13a1f6d}

&emsp;&emsp;后来查阅了资源，`sqlmap进行宽字节注入`的payload如下：
```shell
python2 sqlmap.py -u http://103.238.227.13:10083/?id=1%df%27
```

### 域名解析
&emsp;&emsp;题目描述：
>听说把 flag.bugku.com 解析到120.24.86.145 就能拿到flag

&emsp;&emsp;windows下修改本地hosts解析的方法是修改`C:\Windows\System32\drivers\etc`下的hosts文件，注意要用管理员身份修改。
<div align="center">
    <img src="/images/posts/bugku/17.png" >  
</div>
&emsp;&emsp;在里面增加一条记录即可，然后访问flag.bugku.com，即可得到flag。
<div align="center">
    <img src="/images/posts/bugku/18.png" >  
</div>
