---
layout: post
title: "kali下安装mongodb遇到的问题及解决方案"
date: 2018-01-23 20：40
categories: jekyll update
---

&emsp;&emsp;此文仅以记录Kali下安装mongodb遇到的缺少libssl.so.1.0.0问题的解决方案，如果你安装时出现如下报错信息，那么这篇文章会帮助你解决问题。
```shell
mongod: error while loading shared libssl.so.1.0.0: cannot open shared object file: No such file or directory
```

&emsp;&emsp;这段话无非是告诉你缺少`libssl.so.1.0.0`库，一开始把整个报错信息复制到百度搜索，但没有找到有价值的信息。无奈只能自己想办法解决，而问题的根源就是缺少这个文件，那么我们将从网上下载回这个库文件，所以我将直接在Google上搜索`libssl.so.1.0.0`，出现的第一条就是我们要的信息。
<div align="center">
    <img src="/images/posts/other/15.png" >  
</div>
&emsp;&emsp;选择自己系统对应版本：
<div align="center">
    <img src="/images/posts/other/16.png" >  
</div>

&emsp;&emsp;按照页面所显示的添加它的更新源：
<div align="center">
    <img src="/images/posts/other/17.png" >  
</div>

&emsp;&emsp;这里直接放出它的源：
```shell
deb http://security.debian.org/debian-security wheezy/updates main 
```

&emsp;&emsp;然后执行：
```shell
apt update
apt-get install libssl1.0.0 libssl-dev
```

&emsp;&emsp;如上，即可解决问题。