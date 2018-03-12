---
layout: post
title: "N1CTF funning eating cms writeup"
date: 2018-03-12 23：40
categories: jekyll update
---

# 前言
&emsp;&emsp;这道题get到点时已经太晚了，比赛期间的进度也可以算是达到了`80%`，略微遗憾。

&emsp;&emsp;打开题目的链接，发现只有`login`登陆表，尝试访问`register.php`，返回注册页面，先注册一个账号。用注册的账号登陆后发现是如下页面。
<div align="center">
    <img src="/images/posts/n1ctf/10.png" >  
</div>
&emsp;&emsp;开始猜测是`文件包含漏洞`，一开始是尝试读取`index.php`的源码，但发现没有返回数据。
<div align="center">
    <img src="/images/posts/n1ctf/11.png" >  
</div>
&emsp;&emsp;然后尝试只读`index`，发现这才是正确的打开方式:)
<div align="center">
    <img src="/images/posts/n1ctf/12.png" >  
</div>
&emsp;&emsp;解密后的代码如下：
```php
<?php
require_once "function.php";
if(isset($_SESSION['login'] )){
    Header("Location: user.php?page=info");
}
else{
    include "templates/index.html";
}
?>
```
&emsp;&emsp;用类似的办法将其他文件给读出来。
<b>info.php</b>
```php
<?php
if (FLAG_SIG != 1){
    die("you can not visit it directly ");
}
include "templates/info.html";
?>
```
<br><b>info.php</b>
```php
<?php
if (FLAG_SIG != 1){
    die("you can not visit it directly ");
}
include "templates/guest.html";
?>
```
<br><b>login.php</b>
```php
<?php
require_once "function.php";
if($_POST['action'] === 'login'){
    if (isset($_POST['username']) and isset($_POST['password'])){
        $user = $_POST['username'];
        $pass = $_POST['password'];
        $res = login($user,$pass);
        if(!$res){
            Header("Location: index.php");
        }else{
            Header("Location: user.php?page=info");
        }
    }
    else{
        Header("Location: error_parameter.php");
    }
}else if($_REQUEST['action'] === 'logout'){
    logout();
}else{
    Header("Location: error_parameter.php");
}
?>
```
<br><b>register.php</b>
```php
<?php
require_once "function.php";
if($_POST['action'] === 'register'){
    if (isset($_POST['username']) and isset($_POST['password'])){
        $user = $_POST['username'];
        $pass = $_POST['password'];
        $res = register($user,$pass);
        if($res){
            Header("Location: index.php");
        }else{
            $errmsg = "Username has been registered!";
        }
    }
    else{
        Header("Location: error_parameter.php");
    }
}
if (!$_SESSION['login']) {
    include "templates/register.html";
} else {
    Header("Location : user.php?page=info");
}
?>
```
<br><b>function.php</b>
```php
<?php
session_start();
require_once "config.php";
function Hacker()
{
    Header("Location: hacker.php");
    die();
}


function filter_directory()
{
    $keywords = ["flag","manage","ffffllllaaaaggg"];
    $uri = parse_url($_SERVER["REQUEST_URI"]);
    parse_str($uri['query'], $query);
//    var_dump($query);
//    die();
    foreach($keywords as $token)
    {
        foreach($query as $k => $v)
        {
            if (stristr($k, $token))
                hacker();
            if (stristr($v, $token))
                hacker();
        }
    }
}

function filter_directory_guest()
{
    $keywords = ["flag","manage","ffffllllaaaaggg","info"];
    $uri = parse_url($_SERVER["REQUEST_URI"]);
    parse_str($uri['query'], $query);
//    var_dump($query);
//    die();
    foreach($keywords as $token)
    {
        foreach($query as $k => $v)
        {
            if (stristr($k, $token))
                hacker();
            if (stristr($v, $token))
                hacker();
        }
    }
}

function Filter($string)
{
    global $mysqli;
    $blacklist = "information|benchmark|order|limit|join|file|into|execute|column|extractvalue|floor|update|insert|delete|username|password";
    $whitelist = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'(),_*`-@=+><";
    for ($i = 0; $i < strlen($string); $i++) {
        if (strpos("$whitelist", $string[$i]) === false) {
            Hacker();
        }
    }
    if (preg_match("/$blacklist/is", $string)) {
        Hacker();
    }
    if (is_string($string)) {
        return $mysqli->real_escape_string($string);
    } else {
        return "";
    }
}

function sql_query($sql_query)
{
    global $mysqli;
    $res = $mysqli->query($sql_query);
    return $res;
}

function login($user, $pass)
{
    $user = Filter($user);
    $pass = md5($pass);
    $sql = "select * from `albert_users` where `username_which_you_do_not_know`= '$user' and `password_which_you_do_not_know_too` = '$pass'";
    $res = sql_query($sql);
//    var_dump($res);
//    die();
    if ($res->num_rows) {
        $data = $res->fetch_array();
        $_SESSION['user'] = $data[username_which_you_do_not_know];
        $_SESSION['login'] = 1;
        $_SESSION['isadmin'] = $data[isadmin_which_you_do_not_know_too_too];
        return true;
    } else {
        return false;
    }
    return;
}

function updateadmin($level,$user)
{
    $user = Filter($user);
    $sql = "update `albert_users` set `isadmin_which_you_do_not_know_too_too` = '$level' where `username_which_you_do_not_know`='$user' ";
    $res = sql_query($sql);
//    var_dump($res);
//    die();
//    die($res);
    if ($res == 1) {
        return true;
    } else {
        return false;
    }
    return;
}

function register($user, $pass)
{
    global $mysqli;
    $user = Filter($user);
    $pass = md5($pass);
    $sql = "insert into `albert_users`(`username_which_you_do_not_know`,`password_which_you_do_not_know_too`,`isadmin_which_you_do_not_know_too_too`) VALUES ('$user','$pass','0')";
    $res = sql_query($sql);
    return $mysqli->insert_id;
}

function logout()
{
    session_destroy();
    Header("Location: index.php");
}
?>
```
<br><b>config.php</b>
```php
<?php
error_reporting(E_ERROR | E_WARNING | E_PARSE);
define(BASEDIR, "/var/www/html/");
define(FLAG_SIG, 1);
$OPERATE = array('userinfo','upload','search');
$OPERATE_admin = array('userinfo','upload','search','manage');
$DBHOST = "localhost";
$DBUSER = "root";
$DBPASS = "Nu1LCTF2018!@#qwe";
//$DBPASS = "";
$DBNAME = "N1CTF";
$mysqli = @new mysqli($DBHOST, $DBUSER, $DBPASS, $DBNAME);
if(mysqli_connect_errno()){
        echo "no sql connection".mysqli_connect_error();
        $mysqli=null;
        die();
}
?>
```
<br><b>user.php</b>
```php
<?php
require_once("function.php");
if( !isset( $_SESSION['user'] )){
    Header("Location: index.php");

}
if($_SESSION['isadmin'] === '1'){
    $oper_you_can_do = $OPERATE_admin;
}else{
    $oper_you_can_do = $OPERATE;
}
//die($_SESSION['isadmin']);
if($_SESSION['isadmin'] === '1'){
    if(!isset($_GET['page']) || $_GET['page'] === ''){
        $page = 'info';
    }else {
        $page = $_GET['page'];
    }
}
else{
    if(!isset($_GET['page'])|| $_GET['page'] === ''){
        $page = 'guest';
    }else {
        $page = $_GET['page'];
        if($page === 'info')
        {
//            echo("<script>alert('no premission to visit info, only admin can, you are guest')</script>");
            Header("Location: user.php?page=guest");
        }
    }
}
filter_directory();
//if(!in_array($page,$oper_you_can_do)){
//    $page = 'info';
//}
include "$page.php";
?>
```
&emsp;&emsp;以上就是一开始能读到的源码。想试图尝试读取`flag、ffffllllaaaaggg`，但发现有过滤，会被重定向到`hacker`。然后在用`zap`抓包的时候发现了hint。
<div align="center">
    <img src="/images/posts/n1ctf/15.png" >  
</div>
&emsp;&emsp;但由于过滤的原因，无法访问此文件，但这个hint也为我们指明了方向，接下来尝试了一下`敏感目录扫描`，然后就发现了`.viminfo`泄露。
<div align="center">
    <img src="/images/posts/n1ctf/13.png" >  
</div>
```php
<?php
if (FLAG_SIG != 1){
    die("you can not visit it directly ");
}
include "templates/update.html";
?>
```
&emsp;&emsp;尝试访问了一下：
<div align="center">
    <img src="/images/posts/n1ctf/14.png" >  
</div>
&emsp;&emsp;再去读取`updateadmin233333333333333.php`的源码，发现跟`updateadmin.php`的源码相同。

&emsp;&emsp;然后自己当时想到的点就是：
* **1、绕过isadmin验证**
* **2、绕过filter_directory()过滤**

&emsp;&emsp;先尝试的进行`register`注册账号时进行注入，致使`isadmin`置为`1`，尝试构造了的payload：`hahaha','123','1')-- +`，但是可以看到有白名单限制，它过滤了空格，所以这个猜想看起来行不通。

&emsp;&emsp;再进行`filter_directory`绕过时自己没有看出tips，看出来估计这里就不用停那么久了。。。后来逼得没办法了，就将它里面的关键代码拿去google了一下，这下还真有发现。
<div align="center">
    <img src="/images/posts/n1ctf/16.png" width="50%" />  
</div>
&emsp;&emsp;打开后就找到了payload：
>这里用到了 parse_url 函数在解析 url 时存在的 bug，通过：////x.php?key=value 的方式可以使其返回 False。

&emsp;&emsp;拿着这个payload，发现果真有用。
<div align="center">
    <img src="/images/posts/n1ctf/17.png" width="50%" />  
</div>
&emsp;&emsp;读它的源码：
```php
<?php
if (FLAG_SIG != 1){
    die("you can not visit it directly");
}
include "templates/upload2323233333.html";

?>
```
&emsp;&emsp;发现并没有什么有用的信息，接着访问一下。
<div align="center">
    <img src="/images/posts/n1ctf/18.png" />  
</div>
&emsp;&emsp;在读`upllloadddd.php`的源码：
```php
<?php
$allowtype = array("gif","png","jpg");
$size = 10000000;
$path = "./upload_b3bb2cfed6371dfeb2db1dbcceb124d3/";
$filename = $_FILES['file']['name'];
if(is_uploaded_file($_FILES['file']['tmp_name'])){
    if(!move_uploaded_file($_FILES['file']['tmp_name'],$path.$filename)){
        die("error:can not move");
    }
}else{
    die("error:not an upload file！");
}
$newfile = $path.$filename;
echo "file upload success<br />";
echo $filename;
$picdata = system("cat ./upload_b3bb2cfed6371dfeb2db1dbcceb124d3/".$filename." | base64 -w 0");
echo "<img src='data:image/png;base64,".$picdata."'></img>";
if($_FILES['file']['error']>0){
    unlink($newfile);
    die("Upload file error: ");
}
$ext = array_pop(explode(".",$_FILES['file']['name']));
if(!in_array($ext,$allowtype)){
    unlink($newfile);
}
?>
```
&emsp;&emsp;将关键部分的代码`Google一下`，然后又发现了几乎一样的源码。。。
<div align="center">
    <img src="/images/posts/n1ctf/19.png" width="50%" />  
</div>
&emsp;&emsp;他这里的payload是：
<div align="center">
    <img src="/images/posts/n1ctf/20.png" width="50%" />  
</div>
&emsp;&emsp;由于找到此篇文章的时候已经比赛结束了，，拿着它提供的脚本跑了一会却没有按他说预言的生成`php`文件，所以就从`ctftime`上找了一篇writeup照着复现一下。

&emsp;&emsp;因为他执行的是`system()`函数，所以这里可以造成`任意代码执行`漏洞。
<div align="center">
    <img src="/images/posts/n1ctf/21.png" />  
</div>
<div align="center">
    <img src="/images/posts/n1ctf/22.png" height="50%" />  
</div>
&emsp;&emsp;再查看上一级目录：
<div align="center">
    <img src="/images/posts/n1ctf/23.png" height="50%" />  
</div>
<div align="center">
    <img src="/images/posts/n1ctf/24.png" height="50%" />  
</div>
&emsp;&emsp;读取这个文件，发现直接`cat ../flag_233333`不行，`/`被过滤掉了，不过我们可以`cd ..`再读。
<div align="center">
    <img src="/images/posts/n1ctf/25.png" />  
</div>
<div align="center">
    <img src="/images/posts/n1ctf/26.png" />  
</div>
&emsp;&emsp;至此，成功拿到`flag`。这道题自己学到了很多，也意识到了自己的经验还是不足，以后必将勤加练习，当然还是要留意细节跟善用搜索引擎。。。

&emsp;&emsp;希望能在这条路上走远一点:)

## 参考链接
<a href="https://fireshellsecurity.team/n1ctf-funning-eating-cms/">[*]N1CTF 2018 - Funning eating cms</a><br>
<a href="http://seaii-blog.com/index.php/2017/04/26/49.html">[*]Web中的条件竞争漏洞</a><br>
<a href="https://www.anquanke.com/post/id/84837">[*]GeekPwn2016跨次元CTF Writeup</a><br>
<a href="http://sol.logdown.com/posts/2016/07/14/ais3-pre-exam-2016-part-write-up">[*]web</a>