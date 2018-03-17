---
layout: post
title: "JWT token破解绕过"
date: 2018-03-18 00：40
categories: jekyll update
---

# 前言
&emsp;&emsp;这是印度举办的CTF中遇到的一道JWT破解绕过题，觉得还是挺有价值的，mark一下。

## JWT伪造
&emsp;&emsp;这是一道`b00t2root`的一道web题，觉得很有意思，并且结合了加密的知识，所以记录一下。

&emsp;&emsp;首先了解下JWT：
>JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。JWT常被用于前后端分离，可以和Restful API配合使用，常用于构建身份认证机制。

&emsp;&emsp;JWT的数据格式分为三个部分： headers , payloads，signature(签名)，它们使用`.`点号分割。拿道题后看了一下cookie，发现是如下格式：
<div align="center">
    <img src="/images/posts/other/18.png" >  
</div>
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6ImZhbHNlIn0.oe4qhTxvJB8nNAsFWJc7_m3UylVZzO3FwhkYuESAyUM
```
&emsp;&emsp;进行base64解密后发现：
<div align="center">
    <img src="/images/posts/other/19.png" >  
</div>
&emsp;&emsp;所以，我们的目的就是把`false`改成`true`，而且要通过`服务器的验证`，这点很重要，并不是直接把`false`改成`true`就万事大吉了。因为服务器收到token后会对token的`有效性`进行验证。

&emsp;&emsp;验证方法：首先服务端会产生一个`key`，然后以这个`key`作为密钥，使用第一部分选择的加密方式（这里就是`HS256`），对第一部分和第二部分`拼接的结果`进行加密，然后把加密结果放到`第三部分`。

```
服务器每次收到信息都会对它的前两部分进行加密，然后比对加密后的结果是否跟客户端传送过来的第三部分相同，如果相同则验证通过，否则失败。
```
&emsp;&emsp;因为加密算法我们已经知道了，如果我们只要再得到加密的`key`，我们就能伪造数据，并且通过服务器的检查。

&emsp;&emsp;这里我使用了这个工具进行破解：<a href="https://github.com/brendan-rius/c-jwt-cracker">C语言版JWT破解工具</a>，下载安装完毕后，直接进行破解，如图：
<div align="center">
    <img src="/images/posts/other/20.png" >  
</div>
&emsp;&emsp;所以，我的加密密钥就是：54l7y。然后我们去验证一下，这个网站可以提供验证服务：<a href="https://jwt.io/" target="_blank">https://jwt.io/</a>。当我们使用破解出来的key时，我们能完美还原出原始数据，这证明我们的key是正确的。
<div align="center">
    <img src="/images/posts/other/21.png" width="70%" />  
</div>
&emsp;&emsp;最后我们把`false`改成`true`，然后使用key进行加密，可以得到如下：
<div align="center">
    <img src="/images/posts/other/22.png" width="70%" />  
</div>
&emsp;&emsp;然后我们拿着这个token刷新一下：
<div align="center">
    <img src="/images/posts/other/22.png" width="60%" />  
</div>
&emsp;&emsp;可以看到flag已经出来了。

## 参考链接
<a href="https://github.com/brendan-rius/c-jwt-cracker">https://github.com/brendan-rius/c-jwt-cracker</a><br>
<a href="https://auth0.com/blog/brute-forcing-hs256-is-possible-the-importance-of-using-strong-keys-to-sign-jwts/">https://auth0.com/blog/brute-forcing-hs256-is-possible-the-importance-of-using-strong-keys-to-sign-jwts/</a><br>
<a href="https://jwt.io/">https://jwt.io/</a><br>
<a href="http://www.cnblogs.com/dliv3/p/7450057.html">http://www.cnblogs.com/dliv3/p/7450057.html</a><br>
