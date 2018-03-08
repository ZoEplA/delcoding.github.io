---
layout: post
title: "bugku ctf writeup2"
date: 2018-03-08 22：40
categories: jekyll update
---

## 前言
&emsp;&emsp;这篇开始bugku高级篇。。。

## Web
### XSS注入测试
&emsp;&emsp;题目描述：
>1、请注入一段XSS代码，获取Flag值
>2、必须包含alert(_key_)，_key_会自动被替换

&emsp;&emsp;随便测试了一下，发现会对字符进行实体编码。
<div align="center">
    <img src="/images/posts/bugku/29.png" >  
</div>
&emsp;&emsp;注意到页面是utf-8编码，id传入代码会在s中运行，考虑将<>进行unicode编码，这样当代码被替换进去运行时，utf-8编码又会将其变回来
&emsp;&emsp;所以payload：`?id=\u003cscript\u003ealert(_key_)\u003c/script\u003e`，访问后查看源代码即可得到flag。

### never give up
&emsp;&emsp;查看题目的源代码如下：
<div align="center">
    <img src="/images/posts/bugku/30.png" >  
</div>
&emsp;&emsp;访问`1p.html`，注意是在查看源代码的地方访问，然后可以发现加密了的字符串。
<div align="center">
    <img src="/images/posts/bugku/31.png" >  
</div>
&emsp;&emsp;拿去解密，这里不累赘。直接放出源代码：
```php
if(!$_GET['id'])
{
    header('Location: hello.php?id=1');
    exit();
}
$id=$_GET['id'];
$a=$_GET['a'];
$b=$_GET['b'];
if(stripos($a,'.'))
{
    echo 'no no no no no no no';
    return ;
}
$data = @file_get_contents($a,'r');
if($data=="bugku is a nice plateform!" and $id==0 and strlen($b)>5 and eregi("111".substr($b,0,1),"1114") and substr($b,0,1)!=4)
{
    require("f4l2a3g.txt");
}
else
{
    print "never never never give up !!!";
}
?>
```
&emsp;&emsp;可以看到flag文件已经暴露出来，直接访问也可以拿到flag。但这里介绍绕过检测拿到flag的方法。可以看到满足拿flag的条件有三个：
* **一：id**

&emsp;&emsp;&emsp;&emsp;id既要不等于0（`if(!$_GET['id'])`），又要等于0（`$id==0`）。所以这里我们要利用php的松散性，`字符串跟0比较（==）是成立的`，所以payload：`"aaa" == 0`
* **二：php伪协议**

&emsp;&emsp;&emsp;&emsp;`$data = @file_get_contents($a,'r');`的存在可以使用`php://input`在绕过，所以：`a=php://input`，然后在`post`bugku is a nice plateform!。
* **三：字符截断**

* &emsp;&emsp;&emsp;&emsp;假设：
```php
$b = "%0012345"
substr($b,0,1)  --> 将返回空（null）
strlen($b)>5  --> 是成立的
```

&emsp;&emsp;所以最终的payload：`?id=aaa&a=php://input&b=%00abcde`。
<div align="center">
    <img src="/images/posts/bugku/32.png" >  
</div>
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