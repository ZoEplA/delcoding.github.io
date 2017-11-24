---
layout: post
title: "Linux Xampp安装配置Xdebug"
date: 2017-11-24 09:50:0
categories: jekyll update
---

## 介绍
&emsp;`Xdebug`是一个开源的PHP程序调试工具，可以使用它来调试、跟踪及分析程序运行状态，`XAMPP`（Apache+MySQL+PHP+PERL）是一个功能强大的建站集成软件包，很多PHP初学者都会使用XAMPP+Xdebug的组合。下面介绍怎样在`Linux xmapp`下安装xdebug。

### Step 1：安装Xdebug
&emsp;前往xdebug官网下载相应的版本，如果不知道下载什么版本，可通过官网上自动判断程序。只需新建一个php文件，内容如下：

```php
<?php
    phpinfo();
?>
```

在浏览器中访问并把整个网页的源代码复制下来，提交到如下链接中
<div align="center">
    <img src="/images/posts/other/1.png" height="300" width="500">  
</div>
<br>
<div align="center">
    <img src="/images/posts/other/2.png" height="300" width="500">  
</div>

点击判断。即可得到适合的版本。

### Step 2：解压、安装文件

解压文件：
```shell
# tar –xzvf xdebug-2.5.4.tgz
# cd xdebug-2.5.4/
```

安装前检索，执行phpize，这里由于每个人的路径不一，建议使用find命令查找执行。
```shell
# find / -name phpize
```

得到路径后执行，如：
```shell
# /opt/lampp/bin/phpize
```

执行时如遇如下错误需要安装autoconf解决。
<div align="center">
    <img src="/images/posts/other/3.png">  
</div>
```shell
# apt-get install autoconf
```

安装完成后，重新执行 
```shell
# /opt/lampp/bin/phpize 
```
<div align="center">
    <img src="/images/posts/other/4.png" height="300" width="500">  
</div>

可见成功运行。

### 安装
```shell
# ./configure –enable-xdebug –with-php-config=/opt/lampp/bin/php-config
# make
```
<div align="center">
    <img src="/images/posts/other/5.png">  
</div>

按提示执行
```shell
# make  test
```
<div align="center">
    <img src="/images/posts/other/6.png">  
</div>

接下来安装xdebug
```shell
#make install
```
<div align="center">
    <img src="/images/posts/other/7.png" height="300" width="500">  
</div>

成功安装.

### Step 3：配置php.ini
&emsp;首先找到php.ini文件，如果不知道位置可通过find找到。在xampp下是/opt/lampp/etc/php.ini
<p>
&emsp;然后将xdebug.so路径添加进配置文件，不知道路径同样使用find查找。Xampp下是在 /opt/xdebug-2.5.4/modules/xdebug.so。在文尾添加：

    zend_extension=” /opt/xdebug-2.5.4/modules/xdebug.so”

再将如下配置添加到末尾：
```shell
;显示错误信息
xdebug.default_enable = 1 
;函数调试 
xdebug.auto_trace=on 
xdebug.trace_output_dir 
xdebug.trace_output_name 
;Type: string, Default value: trace.%c 
; (参数长度，参数值，参数=值)
xdebug.collect_params = 1|3|4
; 显示内存
xdebug.show_mem_delta=1 
显示返回值
xdebug.collect_return=1 
; 追加日志
xdebug.trace_options =1 
xdebug.collect_params=1 
xdebug.collect_vars = 1 
;开启性能分析 
xdebug.profiler_enable=1 
;性能分析日志保存目录 
xdebug.profiler_output_dir = /data/logs/xdebug/ 
;性能分析日志文件名称 
xdebug.profiler_output_name = cachegrind.out.log 
;默认是如下格式,t时间,p进程id 
;xdebug.profiler_output_name = cachegrind.out.%t.%p 
;代码覆盖率 
xdebug.coverage_enable = 1 
;以下是远程调试配置 
xdebug.remote_host= 127.0.0.1 
xdebug.remote_connect_back = 1 
xdebug.remote_port = 9000 xdebug.remote_log="/data/logs/xdebug/xdebug.log"
```

### step 4：重启xampp
&emsp;此时在phpinfo页面上可以看到xdebug的信息

<div align="center">
    <img src="/images/posts/other/8.png">  
</div>

