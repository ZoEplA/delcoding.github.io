---
layout: post
title: "Wechall WriteUp"
date: 2017-11-23 11:47:56
categories: jekyll update
---

**这篇文章成功解决了windows下远程连接 kali 的一些问题，免去了网上一些教程中提到的卸载桌面再安装桌面的步骤。**

>本篇教程的系统环境为：
    windows 7 专业版
    kali linux 2016-2


----------


    
Step 1：安装xrdp

```shell
# apt-get install xrdp
```

Step 2：安装vnc4server

```shell
# apt-get install vnc4server
```

Step 3：编辑xrdp配置文件

```shell
# nano /etc/xrdp/xrdp.ini
```

为了防止出现诸如以下错误，需对该配置文件进行修改。
<div align="center">
    <img src="/images/posts/kali/1.png" height="300" width="500">  
</div>
 

将原来max_bpp=32改成max_bpp=16，以防止远程连接时闪退。
<div align="center">
    <img src="/images/posts/kali/2.png" height="300" width="500">  
</div>
 


Step 4：开启xrdp服务

```shell
# service xrdp start
# service xrdp-sesman start
```

Step 5：开启VNC服务

```shell
# cnvserver
```

输入连接密码
<div align="center">
    <img src="/images/posts/kali/3.png" height="300" width="500">  
</div>



Step 6：在windows下运行mstsc
<div align="center">
    <img src="/images/posts/kali/4.png" height="300" width="500">  
</div>
 
<div align="center">
    <img src="/images/posts/kali/5.png" height="300" width="500">  
</div>
 



因为远程服务器将颜色调成了32位，所以我们需要在本地上调整颜色深度。
<div align="center">
    <img src="/images/posts/kali/6.png" height="300" width="500">  
</div>
 


这里选择Xvnc，然后输入用户、密码。
<div align="center">
    <img src="/images/posts/kali/7.png" height="300" width="500">  
</div>


可以看到成功连接到kali。
<div align="center">
    <img src="/images/posts/kali/8.png" height="300" width="500">  
</div>
