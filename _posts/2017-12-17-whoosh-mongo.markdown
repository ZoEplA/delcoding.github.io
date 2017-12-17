---
layout: post
title: "Whoosh使用及为Mongodb建立索引"
date: 2017-12-17 17：57：00
categories: jekyll update
---

## 介绍
&emsp;&emsp;由于作者的项目中需要为数据库建立索引以便以搜索，经过考察后还是决定使用`Whoosh+MongoDB`的方案。所以这篇文章是介绍怎样使用whoosh为mongodb建立索引，网上关于这方面的文章还是相对比较少的，所以有必要记录一下。

&emsp;&emsp;whoosh是原生唯一的python写的`全文搜索引擎`，虽然说whoosh性能不一定比得上lucene,sphinx,xapian等,但由于是基于python，所以扩展性极好，非常容易集成到`django`或其他框架中，而且whoosh运行消耗的资源也比lucene等引擎少，加之作者的服务器性能较弱，数据量也还不大，就选择了`whoosh作为搜索引擎`。

### 架构方案
&emsp;&emsp;作者使用的方案是：使用`jieba`为需要索引的whoosh的字段进行`中文分词`，搜索时，whoosh只返回`_id`字段的值，然后在mongodb中查找。当然，这个whoosh也可以把所有的字段都储存起来，这样就可以直接返回搜索结果，而省去在mongodb上查找的时间，但由此引发的一个问题就是索引文件过大，是空间利用问题，而且这样做就把whoosh当成了数据库角色。所以作者放弃了这一方案。

### 建立索引
&emsp;&emsp;这里不介绍whoosh,mongodb的安装方法，请自行搜索。

```python
from whoosh.fields import Schema, TEXT, ID, DATETIME
from jieba.analyse import ChineseAnalyzer
from whoosh.index import create_in, open_dir
import jieba
jieba.load_userdict("dict.txt")

analyzer = ChineseAnalyzer()

schema = Schema(
            nid=ID(unique=True, stored=True),
            url=ID(unique=True, stored=False),
            date=DATETIME(unique=False, stored=False, sortable=True),
            title=TEXT(stored=False, analyzer=analyzer),
            content=TEXT(stored=False, analyzer=analyzer)
        )

if not os.path.exists('D:/pythonCode/whoosh/index/'):
    os.mkdir("D:/pythonCode/whoosh/index/")
    create_in("D:/pythonCode/whoosh/index/", schema)    # 创建索引文件，需指定Schema

ix = open_dir("D:/pythonCode/whoosh/index/")    #打开索引文件
```
&emsp;&emsp;因为whoosh是python实现的，所以很方便的就能跟结巴结合。而结巴允许的用户自定义字典，这对专业领域的分词很有帮助，我们只需要在`dict.txt`中存放专业名词就行。

&emsp;&emsp;索引首先需要定义`Schema`，`stored`代表是否存储这个字段的值，`sortable`为是否可以排序，这里因为是时间，所以我选择True，`analyzer`分词器，这里我们使用结巴中文分词器。这里再啰嗦几句，ID的值为不可分割，所以适合url、id等一类不用分割查找的项；DATETIME为时间，不用解释；这里的`TEXT`才是我们需要`分词、建立索引`的项目。

&emsp;&emsp;PPS：按照whoosh官方文档，对于简单的字段即使没有指定排序，但仍可以排序，只是结果可能不是那么完美。还有一种情况就是：如果你已经创建了Schema，并且写入数据，但是你想为某个字段排序，那么你可以使用：
```python
from whoosh import sorting
from whoosh.index import open_dir

ix = open_dir("D:/pythonCode/whoosh/index/")

with ix.writer() as w:
    sorting.add_sortable(w, "date", sorting.FieldFacet("date"))
```

### 索引数据

```python
from whoosh.index import create_in, open_dir

client = pymongo.MongoClient("xxx")
db = client['xxx']
collections = db['xxx']

ix = open_dir("D:/pythonCode/whoosh/index/")    #打开索引文件
_id = collections.insert(dict(item))

with ix.writer() as writer:
    writer.update_document(
        nid=_id.__str__(),
        url=item['url'],
        title=item['title'],
        date=datetime.strptime(item['date'], '%Y-%m-%d'),   # 字符串化成时间格式
        content=item['index_content']
    )
            
```
&emsp;&emsp;在需要建立索引的地方使用如上代码。这里注意的一点是，mongodb插入数据返回的是ObjectId，如ObjectId("5a2e2652c0bae92df4dd2372")，而whoosh不支持存储ObjectId类型，所以我们需要的是括号里面的字符串，所以需要使用`_id.__str__()`提取。


### 搜索
&emsp;&emsp;搜索代码如下：
```python
from whoosh.index import create_in, open_dir
from whoosh.qparser import MultifieldParser, QueryParser
from whoosh import scoring, sorting
from bson.objectid import ObjectId

client = pymongo.MongoClient("xxx")
db = client['xxx']
collections = db['xxx']

ix = open_dir("D:/pythonCode/whoosh/index/")    #打开索引文件
with ix.searcher() as searcher:
    query = MultifieldParser(["title", "content"], ix.schema).parse("xss")
    #query = QueryParser("content", ix.schema).parse("xss")

    # 排序
    mf = sorting.MultiFacet()
    mf.add_field("date", reverse=True)
    results = searcher.search(query, limit=10, sortedby=mf)
    print(len(results))

    for one in results:
        _id = ObjectId(one['nid'])      # 将字符串构造成 ObjectId
        res = collections.find({'_id':_id})[0]
        print(res['date'] + res['title'])
        print('-----------------------\n')
        count += 1
```
&emsp;&emsp;`MultifieldParser`是搜索多个field，也就是多个字段，而`QueryParser`只能搜索一个字段。`sorting.MultiFacet()`是whoosh中实现排序的一种方法，`reverse=True`是反向排序，这里实现的是时间降序。`searcher.search(sortedby=mf)`指定排序方式。其实whoosh里提供多种排序方法，功能还挺全面。

&emsp;&emsp;这里提供了mongodb如何使用`_id`查找的方法。

&emsp;&emsp;如果需要将`_id`中的值提取出来，则使用：
```python
_id = collections.insert(dict(item))
nid=_id.__str__()
```
&emsp;&emsp;如果需要使用字符串查找，则使用：
```python
from bson.objectid import ObjectId

_id = ObjectId(one['nid'])      # 将字符串构造成 ObjectId
res = collections.find({'_id':_id})[0]
```

&emsp;&emsp;完整的事例可查看<a href="https://github.com/DelCoding/MyCodes/blob/master/whoosh.py">此页面</a>

