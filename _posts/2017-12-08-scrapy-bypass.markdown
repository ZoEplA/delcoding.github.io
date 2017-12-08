---
layout: post
title: "一种爬虫绕过百度云加速检测的方法"
date: 2017-12-08 11：30：00
categories: jekyll update
---

## 介绍
&emsp;&emsp;最近在做爬虫的时候遇到了百度云加速的浏览器安全检查，经过搜索后得到一种解决方法，现记录。

### 正文
&emsp;&emsp;遇到的检测网页是类似：xxx.com浏览器安全检查中...，如下：
<div align="center">
    <img src="/images/posts/other/12.png" />  
</div>

&emsp;&emsp;这个网页检测的方法是让浏览器执行一段JS代码，将执行结果作为表单的值，跟其他隐藏值一起提交。这个网页的类似代码如下：
```html

<!DOCTYPE HTML>
<html lang="en-US">
<head>
  <meta charset="UTF-8" />
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1" />
  <meta name="robots" content="noindex, nofollow" />
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1" />
  <title>安全检查中...</title>
  <style type="text/css">
    html, body {width: 100%; height: 100%; margin: 0; padding: 0;}
    body {background-color: #ffffff; font-family: Helvetica, Arial, sans-serif; font-size: 100%;}
    h1 {font-size: 1.5em; color: #404040; text-align: center;}
    p {font-size: 1em; color: #404040; text-align: center; margin: 10px 0 0 0;}
    #spinner {margin: 0 auto 30px auto; display: block;}
    .attribution {margin-top: 20px;}
  </style>

    <script type="text/javascript">
  //<![CDATA[
  (function(){
    var a = function() {try{return !!window.addEventListener} catch(e) {return !1} },
    b = function(b, c) {a() ? document.addEventListener("DOMContentLoaded", b, c) : document.attachEvent("onreadystatechange", b)};
    b(function(){
      var a = document.getElementById('yjs-content');a.style.display = 'block';
      setTimeout(function(){
        var s,t,o,p,b,r,e,a,k,i,n,g,f, rmFZnZw={"zIAsbP":+((+!![]+[])+(+[]))};
        t = document.createElement('div');
        t.innerHTML="<a href='/'>x</a>";
        t = t.firstChild.href;r = t.match(/https?:\/\//)[0];
        t = t.substr(r.length); t = t.substr(0,t.length-1);
        a = document.getElementById('jschl-answer');
        f = document.getElementById('challenge-form');
        ;rmFZnZw.zIAsbP+=+((!+[]+!![]+!![]+[])+(!+[]+!![]+!![]+!![]));rmFZnZw.zIAsbP*=+((+!![]+[])+(!+[]+!![]+!![]+!![]+!![]+!![]+!![]+!![]));rmFZnZw.zIAsbP*=+((+!![]+[])+(+!![]));rmFZnZw.zIAsbP+=+((!+[]+!![]+[])+(!+[]+!![]+!![]+!![]+!![]+!![]));rmFZnZw.zIAsbP+=+((!+[]+!![]+!![]+[])+(!+[]+!![]+!![]+!![]+!![]+!![]+!![]+!![]+!![]));rmFZnZw.zIAsbP-=+((!+[]+!![]+!![]+[])+(+!![]));rmFZnZw.zIAsbP+=+((!+[]+!![]+!![]+[])+(+!![]));rmFZnZw.zIAsbP+=+((!+[]+!![]+[])+(+!![]));rmFZnZw.zIAsbP-=+((!+[]+!![]+!![]+!![]+[])+(+[]));rmFZnZw.zIAsbP*=+((!+[]+!![]+!![]+!![]+[])+(!+[]+!![]+!![]+!![]+!![]+!![]));a.value = parseInt(rmFZnZw.zIAsbP, 10) + t.length; '; 121'
        f.submit();
      }, 4000);
    }, false);
  })();
  //]]>
</script>


</head>
<body>
  <table width="100%" height="100%" cellpadding="20">
    <tr>
      <td align="center" valign="middle">
          <div class="yjs-browser-verification yjs-im-under-attack">
  <noscript><h1 data-translate="turn_on_js" style="color:#bd2426;">请打开浏览器的javascript，然后刷新浏览器</h1></noscript>
  <div id="yjs-content" style="display:none">
    <div>
      <div class="bubbles"></div>
      <div class="bubbles"></div>
      <div class="bubbles"></div>
    </div>
    <h1>webbaozi.com <span data-translate="checking_browser">浏览器安全检查中...</span></h1>
    <p data-translate="process_is_automatic"></p>
    <p data-translate="allow_5_secs">还剩 5 秒&hellip;</p>
  </div>
  <form id="challenge-form" action="/cdn-cgi/l/chk_jschl" method="get">
    <input type="hidden" name="jschl_vc" value="3e6d314d3209433cc8d475dce0d7f73a"/>
    <input type="hidden" name="pass" value="1512703287.646-mblyHaO5bp"/>
    <input type="hidden" id="jschl-answer" name="jschl_answer"/>
  </form>
</div>


          <div class="attribution"><a href="http://su.baidu.com/" target="_blank" style="font-size: 12px;"></a></div>
      </td>
    </tr>
  </table>
</body>
</html>

```

&emsp;&emsp;其中一个解决办法就是用python将这段JS代码执行后得到`jschl-answer`的值，然后构造get请求提交，从而获得`cookies`。但是笔者在用requests处理的时候发现，就算将结果运行出来后，使用`requests.get()`的时候还是会重定向到检测页面，没有办法验证通过。一个可能的原因是requests并不会记录`会话`，当你提交结果的时候，检测系统会把你当作第一次访问，而不会认为你已经访问过一次，现在是提交结果。这是`requests`的工作方式。所以只能另想办法。

&emsp;&emsp;我们需要能像浏览器一样能自动解析JS代码的爬虫库，而`selenium`，这个自动化测试工具就能实现这个目标。所以一个有效的解决方案就是使用`selenium`，接下来就用代码解释：

```python
from selenium import webdriver

browser = webdriver.PhantomJS()
host = 'http://www.xxxx.com'
browser.get(host)
sleep(7)    #这里注意需要睡眠5s以上，因为检测页面需要5s左右的时间通过，通过后才能获得真正的网页。
soup = BeautifulSoup(browser.page_source,'lxml')
# 下面就可以做你需要的工作了
```

### 关于python运行JavaScript
&emsp;&emsp;这里顺带记录如何在python下运行JavaScript。代码：
```python
import execjs
fct = """
function test(){
    var a = 10;
    var b = 20;
    return a+b;
}
"""
ej = execjs.compile(fct)
value = ej.call("test") # 调用JS函数并取得返回值
print(value)
```