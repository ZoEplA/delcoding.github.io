---
layout: post
title: "部署Django项目"
date: 2018-01-18 2：00
categories: jekyll update
---

## 前言
&emsp;&emsp;月初一直在忙于期末复习准备考试，导致博客处于断更状态，考完试后就忙于为“黑塔”项目增加爬虫，并将项目正式部署改项目。现在已经完成了初步的部署，所以是时候更新博客和记录部署时遇到的一些问题。其实，开个博客也是能很好的鞭策自己去学习东西，就是为了博客不处于断更状态。

### 目录
* [一：部署概述](#1)
* [二：安装及配置uwsgi](#2)
* [三：安装及配置nginx](#3)
* [四：启动服务](#4)
* [五：部署中遇到的问题及解决方案](#5)

### <a name="1"></a>部署概述
&emsp;&emsp;在本文中采用`nginx + uwsgi`的部署方案，这也是目前较为受欢迎的方案。先放一张整个系统的架构图：
<div align="center">
    <img src="/images/posts/other/12.png" >  
</div>

&emsp;&emsp;在这个方案中，nginx主要处理静态页面，而uwsgi处理动态页面，整个系统对外体现为nginx，当nginx发现请求的是动态页面时会将请求发送给uwsgi处理，这两者之间的通信桥梁可以是端口或者sock的形式，本文采用sock文件的形式，听说这种方式比端口通信更加有效。

### <a name="2"></a>安装及配置uwsgi
&emsp;&emsp;在django的python环境中执行：`pip install uwsgi`即可。
&emsp;&emsp;配置uwsgi，本文采用.ini的文件形式配置。新建一个新的文件夹，可以在任意位置。如本文中在与django项目同级的目录下新建一个名为：uwsgi的文件夹，名字可以任意取。
<div align="center">
    <img src="/images/posts/other/13.png" >  
</div>
&emsp;&emsp;其中`web`是项目文件夹，`uwsgi`文件夹用来存放uwsgi的配置文件和日志等。
&emsp;&emsp;在uwsgi中新建一个`uwsgi.ini`的文件，具体内容如下：
```shell
# uwsig使用配置文件启动
[uwsgi]
# 项目根目录，并非是app目录
chdir=/root/django/web/
# wsgi.py的路径，first是wsgi.py存在的目录名
module=first.wsgi:application
# 指定sock的文件路径，用来与nginx通信       
socket=/root/django/uwsgi/uwsgi.sock
# 进程个数       
workers=4
pidfile=/root/django/uwsgi/uwsgi.pid
# 指定IP端口，这里可以用来测试uwsgi与django项目之间是否准确连接。调试好后可以注释掉
# 如果开启了可以不用开启nginx服务而直接通过 ip:8080访问网页。       
# http=192.168.2.108:8080

# 指定静态文件，这里可以不用收集的静态文件夹，而是使用APP里的static目录
# 如web是项目根目录，security是项目里一个app
static-map=/static=/root/django/web/security/static
# 启动uwsgi的用户名和用户组
uid=junay
gid=root
# 启用主进程
master=true
# 自动移除unix Socket和pid文件当服务停止的时候
vacuum=true
# 序列化接受的内容，如果可能的话
thunder-lock=true
# 启用线程
enable-threads=true
# 设置自中断时间
harakiri=30
# 设置缓冲
post-buffering=4096
# 设置日志目录
daemonize=/root/django/uwsgi/uwsgi.log
```
&emsp;&emsp;上面配置文件的每一条都有详细的说明，请大家仔细阅读。这里需要注意的是，我为这个项目专门添加了一个`junay`的用户，并且将它添加到`root`用户组。后面我还是使用这个用户开启nginx服务。我们需要特别注意用户权限的问题，这个问题也困扰了我两天。

### <a name="3"></a>安装及配置nginx
&emsp;&emsp;nginx直接使用apt安装即可。安装完成后我们在`/etc/nginx/conf.d/`目录下为nginx与uwsgi通信建立配置文件。文件名可以任意，内容如下：
```shell
server { 
    # nginx服务开启的端口
    listen 80; 
    # 如果有域名则写上域名，否则使用IP地址
    server_name www.secsearch.top; 
    # Nginx日志配置
    access_log /var/log/nginx/access.log; 
    charset utf-8; # Nginx编码
    gzip_types text/plain application/x-javascript text/css text/javascript application/x-httpd-php application/json text/json image/jpeg image/gif image/png application/octet-stream; # 支持压缩的类型

    error_page 404 /404.html; # 错误页面
    error_page 500 502 503 504 /50x.html; # 错误页面

    # 指定项目路径uwsgi
    location / {
        # uwsgi_params在nginx文件夹下
        include /etc/nginx/uwsgi_params; 
        # 设置连接uWSGI超时时间
        uwsgi_connect_timeout 30; 
        # nginx与uwsgi的通信方式，动态请求会通过sock传递给uwsgi处理
        uwsgi_pass unix:/root/django/uwsgi/uwsgi.sock; 
    }

    # 指定静态文件路径
    location /static/ {
    alias /root/django/web/security/static/;
    index index.html index.htm;
    }
}
```

### <a name="4"></a>启动服务
&emsp;&emsp;接下来我们启动uwsgi，进入刚才新建的uwsgi文件夹，通过配置文件启动uwsgi：
```shell
uwsgi --ini uwsgi.ini
```
&emsp;&emsp;执行后会在该文件夹下生成uwsgi.log用来记录uwsgi日志，我们可以先查看一下该文件，以保证我们的uwsgi服务是正常的。
&emsp;&emsp;然后我们启动nginx：
```shell
service nginx start
```
&emsp;&emsp;现在通过域名（如果你上面配置的是域名，否则使用IP）访问网站，看是否正常运行。如果你能成功运行那么恭喜你，你可以不用往下看了，如果你出现了一些错误，那么可以借鉴我的解决思路。再附上关闭uwsgi服务和nginx的命令：
```shell
killall -9 uwsgi
service nginx stop
```

### <a name="5"></a>部署中遇到的问题及解决方案
&emsp;&emsp;这里我访问网站后发现是`502`错误，这也是我遇到的一个大坑，很多博客都没有解释和解决掉这个问题。
&emsp;&emsp;出现502后查看日志文件：`cat /var/log/nginx/access.log`，发现是权限问题，原来nginx默认是`www-data`用户运行，但该用户没有权限访问`root`下的目录文件，所以导致服务器出现错误。所以我们需要以`root`身份运行，但root实在是太敏感，所以上面专门添加的用户`junay`就起作用了。
&emsp;&emsp;首先我们修改junay的权限，通过：`vim /etc/passwd`，将juany修改成如下：
```shell
junay:x:0:0:,,,:/home/junay:/usr/bin/git-shell
```
&emsp;&emsp;这里我们将junay的默认shell设成git-shell，防止该用户登陆bash，其次，我们再修改`/etc/ssh/sshd_config`，在该文件中添加如下两行，禁止junay使用ssh。
```shell
AllowUsers root
DenyUsers junay
```
&emsp;&emsp;最后我们修改nginx的默认用户，改配置文件为`/etc/nginx/nginx.conf`，将它的user一行改为：
<div align="center">
    <img src="/images/posts/other/14.png" >  
</div>
&emsp;&emsp;修改完成后，我们重新启动nginx，使用：`service nginx restart`，现在我们就成功的完成了django的部署。如果你还是出现了问题，那么请仔细查看`uwsgi和nginx的日志文件`。