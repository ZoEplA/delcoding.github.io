---
layout: post
title: Selenium下自动识别验证码登陆
date: 2017-11-29 0:0:0
categories: jekyll update
---

## 介绍
&emsp;在自动化或者爬虫登陆网站时经常遇到验证码模块，这也算是反爬的一种手段。这篇文章将介绍如何在selenium框架下自动识别验证码登陆。

### 一：安装Tesseract-ocr
&emsp;Tesseract-ocr是文字识别系统，能识别英文、数字，如果需要识别汉字，则需要导入汉字语言包。<br>
* **在Windows下安装**<br>
&emsp;&emsp;下载地址：<a href="https://digi.bib.uni-mannheim.de/tesseract/">Tesseract-ocr</a>，这里可以选择版本，本机中选择<a href="https://digi.bib.uni-mannheim.de/tesseract/tesseract-ocr-setup-4.0.0-alpha.20170804.exe">4.0.0版</a>。下载后默认安装，这里可以选择修改原安装路径，但修改后在后续进行环境变量的配置时记得更改。
&emsp;&emsp;安装完毕后，配置系统环境变量。在Path一项中新增Tesseract-ocr的安装路径。
<div align="center">
    <img src="/images/posts/other/9.png" >  
</div>

&emsp;&emsp;接着在系统变量中新建如下系统变量，告诉Tesseract-ocr数据集的位置。
<div align="center">
    <img src="/images/posts/other/10.png" >  
</div>

&emsp;&emsp;然后验证是否安装成功。
<div align="center">
    <img src="/images/posts/other/11.png" >  
</div>

* ** 在Linux下安装**<br>
&emsp;&emsp;直接执行：apt install tesseract-ocr。

### 二：python脚本

```python

from selenium import webdriver
from PIL import Image

browser = webdriver.Chrome()

host = "https://www.baidu.com"
browser.get(host)
# 获取整个网页的截图
browser.save_screenshot("temp.png")
# 获取元素位置
element = browser.find_element_by_css_selector("img")
location = element.location
size = element.size
# # 计算出元素位置图像坐标
img = Image.open("temp.png")
left = location['x'] + 145
top = location['y'] + 90
right = location['x'] + size['width'] + 155
bottom = location['y'] + size['height'] + 100
img = img.crop((int(left), int(top), int(right), int(bottom)))
img.save('screenshot.png')  # 是否保存图像

log = os.system('tesseract screenshot.png png -l eng')
keyword = open('png.txt', 'r').readline().rstrip('\n')
rex = re.compile(r'\D')
if re.search(rex, keyword) != None:
    print('错误的验证码：' + keyword)
else:
    print('keyword is ' + keyword)
```

&emsp;代码说明：selenium自带有截图功能，如果是选择`非PhantomJS`作为浏览器的话，当使用`browser.save_screenshot("temp.png")`时截的只是当前浏览器窗口显示的图片而`不是整个网页`，但使用PhantomJS时，截取的`是整个网页`。
&emsp;selenium也可以选择区域或网页元素进行截图，不过需要`PIL`辅助，先从网页中获取该元素的位置，然后使用PIL Image里的`crop`截取。这里需要`注意`，如果使用例如`Chrome`等浏览器而非`PhantomJS`的话，会因为浏览器弹出的位置不同，导致无法准确的截到该元素的位置，需要自己反复调节，但使用PhantomJS时不会有这个问题。

&emsp;可以注意到，我这里是使用系统调用的方法进行识别，但有另一种方法是安装pyocr。直接执行`pip install pyocr`即可。由于作者不熟悉，加之有急需，故不再演示，只提出方法。
