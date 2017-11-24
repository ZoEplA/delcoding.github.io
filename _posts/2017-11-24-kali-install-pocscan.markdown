---
layout: post
title: "Kali安装pocscan"
date: 2017-11-24 09:26:0
categories: jekyll update
---

## 介绍
&emsp;简单记录在`Kali下安装Pocscan`，Pocscan是一款开源 Poc 调用框架,可轻松调用Pocsuite,Tangscan,Beebeeto,Knowsec老版本POC 按照官方规范编写的 Poc对目标域名进行扫描，并可以通过 Docker 一键部署。


* Step 1：apt-get install docker

* Step 2: 更改更新源列表（/etc/apt/source.list），往里面增加docker更新源。解决：
没有可用的软件包 docker.io，但是它被其它的软件包引用了。
E: 软件包 docker.io 没有可安装候选
```
deb http://http.debian.net/debian jessie-backports main
```

* step 3：apt-get install docker.io

* step 4：service docker start

* step 5: docker pull daocloud.io/aber/pocscan:latest

* step 6: cd /

* step 7: git clone https://github.com/erevus-cn/pocscan.git

* step 8: chmod -R 0777 pocscan

* step 9: docker run -d -v /pocscan:/www -p 8090:8000 -p 8088:8088 daocloud.io/aber/pocscan:latest

* step 10: 访问本地8090端口，后台和终端的帐号是root,密码是password.

* Step 11：安装google chrome，并把主目录下pocsuite中的pocscan.crx安装到chrome。
