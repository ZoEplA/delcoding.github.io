---
layout: post
title: "tamu ctf writeup"
date: 2018-02-26 23：40
categories: jekyll update
---

## 前言
&emsp;&emsp;好像很久没更新blog了，昨天接到任务要做tamu的ctf，所以做完记录一下。

### 1、介绍题
&emsp;&emsp;Flag很明显（gigem{Howdy!}），主要是熟悉flag形式。
<div align="center">
    <img src="/images/posts/tamu/1.png" >  
</div>

### 2、杂项1
<div align="center">
    <img src="/images/posts/tamu/2.png" >  
</div>
&emsp;&emsp;将附件下载后使用binwalk分析文件，可以看到有隐藏文件，再使用foremost分离文件，解压后可以在document.xml里找到flag。这里另一种解题方法就是将这些附件`还原`成doc或docx。
<div align="center">
    <img src="/images/posts/tamu/3.png" >  
</div>
<div align="center">
    <img src="/images/posts/tamu/4.png" >  
</div>
&emsp;&emsp;所以flag就是：Flag=ICanRead!（不要自己脑补格式…）

### 3、杂项2
<div align="center">
    <img src="/images/posts/tamu/5.png" >  
</div>
&emsp;&emsp;按要求登陆，登陆后。
<div align="center">
    <img src="/images/posts/tamu/6.png" >  
</div>
&emsp;&emsp;Tips: 使用ls –a，简单粗暴点。

### 4、web3
<div align="center">
    <img src="/images/posts/tamu/7.png" >  
</div>
&emsp;&emsp;看到robots就预感跟robots.txt有关，打开网页，google bot。。。
<div align="center">
    <img src="/images/posts/tamu/8.png" >  
</div>
<div align="center">
    <img src="/images/posts/tamu/9.png" >  
</div>
<div align="center">
    <img src="/images/posts/tamu/10.png" >  
</div>
&emsp;&emsp;Flag：gigem{craw1ing_bot$!} 

### 5、web4
<div align="center">
    <img src="/images/posts/tamu/11.png" >  
</div>
<div align="center">
    <img src="/images/posts/tamu/12.png" >  
</div>
&emsp;&emsp;注意标题“SQLi”，明显是一道sql注入题，sqlmap三连后可以得到账号密码。登陆后即可得到flag：gigem{ScRuB7h3InpU7}。
<div align="center">
    <img src="/images/posts/tamu/13.png" >  
</div>
<div align="center">
    <img src="/images/posts/tamu/14.png" >  
</div>

### 6、Scenario – MCCU：01_logs
&emsp;&emsp;这道题连蒙带猜。。。一看题目，先fuzz一波。。
<div align="center">
    <img src="/images/posts/tamu/15.png" >  
</div>
&emsp;&emsp;得到：1：web；2：access.log…

&emsp;&emsp;因为一般入侵都不太可能一次试探就成功，所以打算在error.log里找IP，这里看完后也就两个IP：172.16.20.134跟172.16.20.24。

得到：3：172.16.20.24。

&emsp;&emsp;时间有点难找，所以我先跳过4，先做5。在access.log里把172.16.20.24所访问的页面全找出来，然后在`/wp-content/plugins/wpshop/includes/ajax.php`发现异常，提交后提示正确，所以时间也就是这次访问的时间。
<div align="center">
    <img src="/images/posts/tamu/16.png" >  
</div>

&emsp;&emsp;所以flag：
```
* 1：web；
* 2：access.log；
* 3：172.16.20.24；
* 4： Nov/08-20:32:49；
* 5：/wp-content/plugins/wpshop/includes/ajax.php。
```
&emsp;&emsp;注意4的时间格式，这里我交完就忘了要不要将Nov转成11，，，，再次提交又不再验证，，，反正时间是这个。。。

&emsp;&emsp;后来google了一下，发现是WordPress WPshop - 电子商务1.3.9.5，任意文件上传漏洞，这里mark一下。

<b>Payload:</b>
* <a href="https://github.com/XiphosResearch/exploits/tree/master/wpsh0pwn">https://github.com/XiphosResearch/exploits/tree/master/wpsh0pwn</a><br>
* <a href="http://www.3xploi7.com/2015/12/wordpress-wpshop-ecommerce-1395.html">http://www.3xploi7.com/2015/12/wordpress-wpshop-ecommerce-1395.html</a>

### 7、Scenario – MCCU：00_intrusion
&emsp;&emsp;Fuzz即可。。。。
<div align="center">
    <img src="/images/posts/tamu/17.png" >  
</div>

### 8、Scenario – ClandestineEnforced：01_Phishing
&emsp;&emsp;题目的意思是找出钓鱼邮件，打开下面三个邮件，1，2很明显就是钓鱼邮件，所以flag：1,2，直接提交即可。
<div align="center">
    <img src="/images/posts/tamu/18.png" >  
</div>

### 9、web1
&emsp;&emsp;这道题为自己蠢哭。。。。都说reading了，，，就是脑子抽风了不知道咋整，还好解了其他题，脑子回来了。。。
<div align="center">
    <img src="/images/posts/tamu/19.png" >  
</div>
&emsp;&emsp;慢慢reading，，，终于在某一行找到了flag。。。。
<div align="center">
    <img src="/images/posts/tamu/20.png" >  
</div>
&emsp;&emsp;所以flag：gigem{F!nD_a_F!AG!}

### 10、web2
<div align="center">
    <img src="/images/posts/tamu/21.png" >  
</div>
<div align="center">
    <img src="/images/posts/tamu/22.png" >  
</div>
&emsp;&emsp;打开burp suite，把两个按钮的包抓一下，
<div align="center">
    <img src="/images/posts/tamu/23.png" >  
</div>
&emsp;&emsp;把veggie base64解码一下，可以发现有东西。
<div align="center">
    <img src="/images/posts/tamu/24.png" >  
</div>
&emsp;&emsp;再看看第二个包，
<div align="center">
    <img src="/images/posts/tamu/25.png" >  
</div>
&emsp;&emsp;再把cookie的值base64解码一下。
<div align="center">
    <img src="/images/posts/tamu/26.png" >  
</div>
&emsp;&emsp;所以flag：gigem{CrAzzYY_4_CO0k!es}

### 11、Scenario – NotSoAwesomeInc：00_intrusion
&emsp;&emsp;将附件下载后认真找就能找到答案。
<div align="center">
    <img src="/images/posts/tamu/27.png" >  
</div>



