---
layout: post
title: "FCKeditor 上传漏洞-截断上传"
date: 2017-11-24 09:0:0
categories: jekyll update
---

## 介绍
&emsp;`FCKeditor编辑器`还是使用比较广泛的网站后台编辑器，在此次实战中应用了截断上传漏洞，不过有些不同的是，截断的不是文件名，而是上传路径。适用版本：`<=2.6.4`。

**涉及手段： %00截断、文件覆盖**

&emsp;一开始，尝试在文件名后截断，但全都无果，Google后发现篇paper，使用之，成功upload。`主要思路文件夹截断，并把文件夹名当作上传文件名，从而绕过对文件名的验证`。

&emsp;Step 1：在本地编辑好PHP一句话，并且保存为txt格式。（不免杀）
<div align="center">
    <img src="/images/posts/shizhan/1.png" >  
</div>

&emsp;Step 2：选择PHP上传，并选中编辑好的txt文件。
<div align="center">
    <img src="/images/posts/shizhan/2.png" >  
</div>

&emsp;Step 3：用burpsuite抓包，在文件夹路径下写好上传后的问价名，并在最后用%00截断，如下图：
<div align="center">
    <img src="/images/posts/shizhan/3.png" >  
</div>

<div align="center">
    <img src="/images/posts/shizhan/4.png" >  
</div>

&emsp;Step 4：改好后放行，可以看到文件成功上传，然后访问返回的command.php。
<div align="center">
    <img src="/images/posts/shizhan/5.png" >  
</div>

&emsp;Step 5：在这执行系统命令，但可以会受到权限限制。
<div align="center">
    <img src="/images/posts/shizhan/6.png" >  
</div>

&emsp;Step 6：同样的方法可用于一句话上传，然后使用菜刀连接。