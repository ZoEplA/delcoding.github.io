---
layout: post
title: "解决Scrapy爬虫卡(停)顿问题"
date: 2017-12-21 22：20
categories: jekyll update
---

## 介绍
&emsp;&emsp;最近在做爬虫的时候经常遇到`爬虫卡顿（停顿）`的情况，让人很是苦恼，稍不注意就进程就卡住，在搜索了方法后，最后采用`自动代理切换+超时下载件`的方法解决。

### 编写自动代理中间件
&emsp;&emsp;在项目的`middlewares.py`中新建一个自动代理中间件的类，如下：
```python
class MyproxyMiddleware(object):
    """docstring for ProxyMiddleWare"""

    def process_request(self, request, spider):
        '''对request对象加上proxy'''
        # proxy = self.get_random_proxy()
        haha = random.randint(0,10)
        if haha >= 5:
            print('\n随机{}正在使用代理\n'.format(haha))
            proxy = 'http://127.0.0.1:8087'
            request.meta['proxy'] = proxy

    def process_response(self, request, response, spider):
        '''对返回的response处理'''
        # 如果返回的response状态不是200，重新生成当前request对象
        if response.status != 200:
            proxy = 'http://127.0.0.1:8087'
            # print("this is response ip:" + proxy)
            # 对当前reque加上代理
            request.meta['proxy'] = proxy
            return request
        return response

    def process_exception(self, request, exception, spider):
        # 出现异常时（超时）使用代理
        print("\n出现异常，正在使用代理重试....\n")
        proxy = 'http://127.0.0.1:8087'
        request.meta['proxy'] = proxy
        return request
```

&emsp;&emsp;`process_request()`方法是发起请求时调用的，所以如果你希望一开始请求就使用代理就可在这里写上代理地址，这个函数可以返回三个值：`None`、`request`、`response`。
如果是返回None，说明不对请求头做任何处理；如果返回request，则按照用户定制的请求头请求网页，如果我们要使用代理则需返回request，不写明写返回也是返回request；response表示不再使用其他下载中间件，直接返回响应结果。

&emsp;&emsp;这里我使用随机的方法使用代理，毕竟代理流量也是有限度的。我使用的是`XX-NET`免费代理，它的主要用途是被国人拿来扶墙的，它提供自动IP切换服务，可以很好的为爬虫服务。另一种免费躲避IP限制的方法是使用`tor`，不过在国内的话需要先科学上网。

&emsp;&emsp;`process_exception()`是对爬虫请求是出现的异常情况进行处理的方法，它会捕捉`超时、503`等异常，当我们出现卡顿或停顿时很有可能就是超时，所以我们要编写这个函数。当返回`request`时，爬虫会重新使用使用代理进行请求，我们需要的就是这个功能。


### 添加下载中间件
&emsp;&emsp;将上面的中间件添加到`setting.py`中，
```python
DOWNLOADER_MIDDLEWARES = {
    'scrapy_first.middlewares.MyproxyMiddleware':200,
    'scrapy_first.middlewares.RandomUserAgent':158,
    'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware':500,
}
```

&emsp;&emsp;`scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware`这个中间件可以定义超时时间，配合`DOWNLOAD_TIMEOUT = 200`使用。这也是防止爬虫停顿的方法。另一个跟这个功能差不多的是：`scrapy.contrib.downloadermiddleware.retry.RetryMiddleware`，有需要的也可以添加进去，它会把超时或者 HTTP 500 错误导致失败的页面记录下来，当爬虫爬取完正常的页面后再统一重新请求这些异常也页面。它有三个属性：

* RETRY_ENABLED
* RETRY_TIMES
* RETRY_HTTP_CODES

&emsp;&emsp;各自的属性：
```
RETRY_ENABLED

新版功能。

默认： True

Retry Middleware 是否启用。

RETRY_TIMES

默认：2

包括第一次下载，最多的重试次数

RETRY_HTTP_CODES

默认： [500, 502, 503, 504, 400, 408]

重试的 response 返回值(code)。其他错误(DNS 查找问题、连接失败及其他)则一定会进行重试。
```

&emsp;&emsp;通过合理配置这些下载中间件就能很好的避免爬虫卡死的情况。
