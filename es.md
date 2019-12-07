
# 本文档的定位是一个速查手册，以解决工程上经常用到的api和概念，如果想要更深理解，请查看es权威指南的“怎么读这本书”提到的深入部分

#全文数据
------------
* 全文数据库是全文检索系统的主要构成部分。所谓全文数据库是将一个完整的信息源的全部内容转化为计算机可以识别、处理的信息单元而形成的数据集合。全文数据库不仅存储了信息，而且还有对全文数据进行词、字、段落等更深层次的编辑、加工的功能，而且所有全文数据库无一不是海量信息数据库。
#Restful
--------------
* es这里应该指的的webRestfulAPI
* Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在请求之间是无状态的。从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器回答，这十分适合云计算之类的环境。客户端可以缓存数据以改进性能。
* 在服务器端，应用程序状态和功能可以分为各种资源。资源是一个有趣的概念实体，它向客户端公开。资源的例子有：应用程序对象、数据库记录、算法等等。每个资源都使用 URI (Universal Resource Identifier) 得到一个唯一的地址。所有资源都共享统一的接口，以便在客户端和服务器之间传输状态。使用的是标准的 HTTP 方法，比如 GET、PUT、POST 和 DELETE。Hypermedia 是应用程序状态的引擎，资源表示通过超链接互联。
* RESTful的关键是定义可表示流程元素/资源的对象。在REST中，每一个对象都是通过URL来表示的，对象用户负责将状态信息打包进每一条消息内，以便对象的处理总是无状态的。
* 由于轻量级以及通过 HTTP 直接传输数据的特性，Web 服务的 RESTful 方法已经成为最常见的替代方法。可以使用各种语言（比如 Java 程序、Perl、Ruby、Python、PHP 和 Javascript[包括 Ajax]）实现客户端。RESTful Web 服务通常可以通过自动客户端或代表用户的应用程序访问。但是，这种服务的简便性让用户能够与之直接交互，使用它们的 Web 浏览器构建一个 GET URL 并读取返回的内容。
* 在 REST 样式的 Web 服务中，每个资源都有一个地址。资源本身都是方法调用的目标，方法列表对所有资源都是一样的。这些方法都是标准方法，包括 HTTP GET、POST、PUT、DELETE，还可能包括 HEAD 和 OPTIONS。
#API
------------
es提供两种API 
##1. javaAPI（官方提供）
[java以及其他编程语言的api](https://www.elastic.co/guide/en/elasticsearch/client/index.html)
其他语言的api用各自社区提供
##2. restfulAPI
通过HTTP的json传输数据，你直接通http交互数据。
在linux 中使用 curl 命令发起http请求。
```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```


 参数|解释 | 
---------|----------|
 VERB |可用的http方法。如：GET,PUT,POST,HEAD,DELETE | 
 PROTOCOL | 协议 | 
 HOST| 你任何一个，注意是任何一个es集群的节点，或者就localhost在你本地的主机上 |
 PORT|默认9200|
 PATH|你访问资源的路径|
 QUERY_STRING|Any optional query-string parameters (for example ?pretty will pretty-print the JSON response to make it easier to read.)|
 BODY|A JSON-encoded request body (if the request needs one.)|

举例
```
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'
```
你收到的json body
```json
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```
我们收到的json是包含请求头的，但是我们没有显示他们。如果你想查看，就使用 -i
```
curl -XGET 'localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'
```
在kibana里面，我们可以用更加简易的命令，省略curl命令。主机名，端口，参数列表，-d
```json
GET /_count
{
    "query": {
        "match_all": {}
    }
}
```
##索引文档
-------------------


**注意哦，索引文档的索引是个动词，就是给文档建立索引的过程，相当于关系型数据库中的插入一条新记录**
```
PUT /megacorp/_doc/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```
如果不指明id es会帮你创建。所有文档的id都是唯一的。

##检索文档

一个 CRUD 操作只对单个文档进行处理，文档的唯一性由 _index, _type, 和 routing values （通常默认是该文档的 _id ）的组合来确定。 这表示我们确切的知道集群中哪个分片含有此文档。
搜索需要一种更加复杂的执行模型因为我们不知道查询会命中哪些文档: 这些文档有可能在集群的任何分片上。 一个搜索请求必须询问我们关注的索引（index or indices）的所有分片的某个副本来确定它们是否含有任何匹配的文档。

但是找到所有的匹配文档仅仅完成事情的一半。 在 search 接口返回一个 page 结果之前，多分片中的结果必须组合成单个排序列表。 为此，搜索被执行成一个两阶段过程，我们称之为 query then fetch

------------
指明id的检索
```
GET /megacorp/_doc/1
```

##简单搜索
----------------
```
GET /megacorp/_doc/_search
```
这个命令会检索指定index中所有的文档，并且默认返回顶上的十条结果。
```
GET /megacorp/employee/_search?q=last_name:Smith


curl -X GET "localhost:9200/megacorp/employee/_search?q=last_name:Smith"

```
在这个例子中，我们增加一个query-string  q= parameter
##部分索引
```
GET /website/blog/123?_source=title,text
```
返回
```
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```
很像sql中的投影，我觉得这个是可以和DSL配合使用的
另外，一个空_source会返回这个source的所有内容，注意，原始文档就是存在_source域中的。其实就是返回这个原始文档
```
GET /website/blog/123/_source
```
```
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```
##Query DSL
query-string无疑是简单的，方便。但是它还是有局限性的。所以我使用更灵活的，丰富的查询语句叫做Query DSL
```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```
这个例子中返回的结果和‘GET /megacorp/employee/_search?q=last_name:Smith’是一样的。


Let’s make the search a little more complicated. We still want to find all employees with a last name of Smith, but we want only employees who are older than 30. Our query will change a little to accommodate a filter, which allows us to execute structured searches efficiently:
```json
GET /megacorp/employee/_search
{
    "query" : {
        "bool" : {
            "must" : {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter" : {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
```
>This portion of the query is a range filter, which will find all ages older than 30—gt stands for greater than.

##[全文检索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/full-text-search.html)
全文搜索两个最重要的方面是：

* 相关性（Relevance）

它是评价查询与其结果间的相关程度，并根据这种相关程度对结果排名的一种能力，这种计算方式可以是 TF/IDF 方法（参见 相关性的介绍）、地理位置邻近、模糊相似，或其他的某些算法。

* 分析（Analysis）

它是将文本块转换为有区别的、规范化的 token 的一个过程，（参见 分析的介绍） 目的是为了（a）创建倒排索引以及（b）查询倒排索引。
一旦谈论相关性或分析这两个方面的问题时，我们所处的语境是关于查询的而不是过滤。

```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```
结果返回
```
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, 
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016, 
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}

```
>注意看这里有分数哦

##短语查询
```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```
短语查询，返回包含所有查询短语的结果，不像全文匹配就只要一个满足即可。

##高亮我们的检索


[HILIGHT_DOC](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-request-highlighting.html)

#检索文档详解
https://www.elastic.co/guide/en/elasticsearch/guide/current/get-doc.html


##分析数据
举个例子，我们找在雇员中最流行的兴趣
```
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```
>这个数据不是预先算好的，而是在我们查询的结果中计算的。这个例子中就是在‘GET /megacorp/employee/_search’的结果中统计
```
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```
>这个例子是统计所有叫smith的人的兴趣统计。在查询所有叫Smith的人的结果中进行统计。
#####聚合也允许分层的查询。
>举个例子：
>>我们找出所有喜欢特定兴趣的雇员的的平均年龄
```
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```
>从这里看出分层聚合也是对已有的查询结果进行统计。很明显。。如果都是聚合则嵌套，如果对一般查询进行统计就是顺序。
##集群
###用到的时候再看
[LINK_TO_CLUSTER](https://www.elastic.co/guide/en/elasticsearch/guide/current/distributed-cluster.html)


##DATA IN DATA OUT
>在es中，每一个字段都会被建立一个倒排索引
###什么是文档
>在es中，在一个唯一的ID下的对象。
###文档的元数据
>_index
>>Where the document lives

>_type(7.X及以上被弃用)
>>The class of object that the
document represents

>_id
>>The unique identifier for the document

**_Tip._**
Actually, in Elasticsearch, our data is stored and indexed in shards, while an index is just a logical namespace that groups together one or more shards. However, this is an internal detail; our application shouldn’t care about shards at all. As far as our application is concerned, our documents live in an index. Elasticsearch takes care of the details

###其他的元数据
[出现在Types and Mappoing](https://www.elastic.co/guide/en/elasticsearch/guide/current/_document_metadata.html)

###为文档建立索引
>每个文档都有一个版本号（_version），每一次改变这个文档就会使这个文档的版本号增加。所以你可以通过确认版本号以确定有没有被其他的副本重写

>Autogenerated IDs are 20 character long, URL-safe, Base64-encoded GUID strings. 



##确认文档是否存在

使用head命令 但是不返回json，只返回http头

```
curl -i -XHEAD http://localhost:9200/website/blog/123
```
如果文档存在
```http
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```
如果不存在
```http
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

##更新文档
* 你可以指定一个id，然后简单的put上去，这时是有一个复写过程的，所以版本号++
* 你也可以先用delete命令删除，再put上去一个，由于这个put是一个新的文档，所以版本号是1
* 对于delete和put你可以用一个命令解决--update
>由于两个请求合并成一个请求了，所以减少了响应时间。
* 一个文档是不可变的，但是update给你外部变现为部分的修改文档，但是内部还是经历了先删再改的过程。不同的是，他这个过程就用了一个shard，一个请求就完成了

[部分更新，根据脚本更新，解决冲突](https://www.elastic.co/guide/en/elasticsearch/guide/current/partial-updates.html)


##创建一个新的文档，如果id没被占用就创建，如果id被占用就不创建。
```
PUT /website/blog/123/_create
{ ... }
```
If the request succeeds in creating a new document, Elasticsearch will return the usual metadata and an HTTP response code of 201 Created.

On the other hand, if a document with the same _index, _type, and _id already exists, Elasticsearch will respond with a 409 Conflict response code, and an error message like the following:
```
{
   "error": {
      "root_cause": [
         {
            "type": "document_already_exists_exception",
            "reason": "[blog][123]: document already exists",
            "shard": "0",
            "index": "website"
         }
      ],
      "type": "document_already_exists_exception",
      "reason": "[blog][123]: document already exists",
      "shard": "0",
      "index": "website"
   },
   "status": 409
}
```
##删除文档
```
DELETE /website/blog/123


```
If the document is found, Elasticsearch will return an HTTP response code of 200 OK and a response body like the following. 


## 删除索引
删除一个索引编辑
用以下的请求来 删除索引:

DELETE /my_index
你也可以这样删除多个索引：

DELETE /index_one,index_two
DELETE /index_*
你甚至可以这样删除 全部 索引：

DELETE /_all
DELETE /*

注意
对一些人来说，能够用单个命令来删除所有数据可能会导致可怕的后果。如果你想要避免意外的大量删除, 你可以在你的 elasticsearch.yml 做如下配置：

action.destructive_requires_name: true

这个设置使删除只限于特定名称指向的数据, 而不允许通过指定 _all 或通配符来删除指定索引库。你同样可以通过 Cluster State API 动态的更新这个设置。
**Note that the _version number has been incremented:**
```
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```
If the document isn’t found, we get a 404 Not Found response code and a body like this:
```
{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}
```
**Even though the document doesn’t exist (found is false), the _version number has still been incremented. **
This is part of the internal bookkeeping, which ensures that changes are applied in the correct order across multiple nodes.

**_Note_**
>As already mentioned in Updating a Whole Document, deleting a document doesn’t immediately remove the document from disk; it just marks it as deleted. Elasticsearch will clean up deleted documents in the background as you continue to index more data.



##并发控制
es采取应对读写矛盾的方式就是如果，资源在读写之间的状态，就取消这一次操作。
一般es作为一个搜索引擎，原始数据源存在其他数据库里。专门的数据库在处理并发矛盾更加成熟。

[link___为什么ES不适合做数据存储](https://blog.csdn.net/wzdxt/article/details/50868031)

es通过检查版本号来确定是否接受更新。
https://www.elastic.co/guide/en/elasticsearch/guide/current/optimistic-concurrency-control.html

##索引多个文档
为了节省带宽，你可以一次性检索多个文档，以此节省多次请求的开销，速度更快。
```
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```
```
{
   "docs" : [
      {
         "_index" :   "website",
         "_id" :      "2",
         "_type" :    "blog",
         "found" :    true,
         "_source" : {
            "text" :  "This is a piece of cake...",
            "title" : "My first external blog entry"
         },
         "_version" : 10
      },
      {
         "_index" :   "website",
         "_id" :      "1",
         "_type" :    "pageviews",
         "found" :    true,
         "_version" : 2,
         "_source" : {
            "views" : 2
         }
      }
   ]
}
```
>分数只在模糊类型的查询才计算，但是在这个精确查询中，并不会有分数。

>如果哪个文档并不存在，那么在回应体中也会体现出来。
```
{
  "docs" : [
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "2",
      "_version" : 10,
      "found" :    true,
      "_source" : {
        "title":   "My first external blog entry",
        "text":    "This is a piece of cake..."
      }
    },
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "1",
      "found" :    false  
    }
  ]
}
```
>**Note**
>>The HTTP status code for the preceding request is 200, even though one document wasn’t found. In fact, it would still be 200 if none of the requested documents were found—because the mget request itself completed successfully. To determine the success or failure of the individual documents, you need to check the found flag.

## Bulk API
 
 the bulk API allows us to make multiple create, index, update, or delete requests in a single step

 https://www.elastic.co/guide/en/elasticsearch/guide/current/bulk.html


# 一些工程不用太理解，但是我花了很大功夫才看懂的概念

* [consistency](https://www.elastic.co/guide/en/elasticsearch/guide/current/distrib-write.html)
>配置的副本数跟真正存活的的副本数是不一样的概念。这个参数是存活的副本达到法定数量时候才允许写操作。

* [Document-Based Replication](https://www.elastic.co/guide/en/elasticsearch/guide/current/_partial_updates_to_a_document.html)
>转发完整新文档的方式是，不管怎么样，主碎片的内容是按照命令顺序改的，由于转发到副本是异步的，所以到达的更改顺序可能不同，导致文档损坏。所以我就干脆把主碎片修改好的文档转发出去，让副本结点自己替换掉，而不是执行更新替换命令。简而言之，就是把最新后正确的文档一次转发出去替换旧的，而不是转发一条条更新指令！！


##检索要知道得三个概念
* mapping
* analysis
* query DSL
>While many searches will just work out of the box, to use Elasticsearch to its full potential, you need to understand three subjects:

Mapping
How the data in each field is interpreted
Analysis
How full text is processed to make it searchable
Query DSL
The flexible, powerful query language used by Elasticsearch
Each of these is a big subject in its own right, and we explain them in detail in Search in [Depth](https://www.elastic.co/guide/en/elasticsearch/guide/current/search-in-depth.html). The chapters in this section introduce the basic concepts of all three—just enough to help you to get an overall understanding of how search works.

```json
{
   "hits" : {
      "total" :       14,
      "hits" : [
        {
          "_index":   "us",
          "_type":    "tweet",
          "_id":      "7",
          "_score":   1,
          "_source": {
             "date":    "2014-09-17",
             "name":    "John Smith",
             "tweet":   "The Query DSL is really powerful and flexible",
             "user_id": 2
          }
       },
        ... 9 RESULTS REMOVED ...
      ],
      "max_score" :   1
   },
   "took" :           4,
   "_shards" : {
      "failed" :      0,
      "successful" :  10,
      "total" :       10
   },
   "timed_out" :      false
}


```




**hits** | 
---------|
 返回体中最重要的部分，包含了匹配结果总计的数量，并且包含了前10条匹配的文档的--匹配结果。这些匹配结果每一个都包含_index,_type,_id,_source filed。每一个都有_score，这就是相关分数，表达了匹配的效果。max_score表明所有匹配结果中的最高分。 |
 
 **took**|
 -----------|
 这个结果花了我们多少时间，以milliseconds为单位|

 
 **shards**|
 ------------|
 告诉我们有都多少个shards参与了我们的查询，他们之中有多少是成功的，多少是失败的。|

**timeout**|
----------|
告诉我们这次查询有没有超时。默认是没有超时的，如果你觉得相应时间比结果完整性更重要的话，你可以设置一个超时参数。|
```
GET /_search?timeout=10ms
```
es会在时间限制结束前，返回所有已经查询的结果。

**Warning**

It should be noted that this timeout does not halt the execution of the query; it merely tells the coordinating node to return the results collected so far and to close the connection. In the background, other shards may still be processing the query even though results have been sent.

Use the time-out because it is important to your SLA, not because you want to abort the execution of long-running queries.

## 多indeces，多typeps搜索
```
/_search
Search all types in all indices
/gb/_search
Search all types in the gb index
/gb,us/_search
Search all types in the gb and us indices
/g*,u*/_search
Search all types in any indices beginning with g or beginning with u
/gb/user/_search
Search type user in the gb index
/gb,us/user,tweet/_search
Search types user and tweet in the gb and us indices
/_all/user,tweet/_search
Search types user and tweet in all indices
```
## Pagination 页码
之前提到的hits包括十个文档，这里我们谈论怎么查看其他文档
```
size
Indicates the number of results that should be returned, defaults to 10
from
Indicates the number of initial results that should be skipped, defaults to 0
```
If you wanted to show five results per page, then pages 1 to 3 could be requested as follows:
```
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10

```
[注意不要一次性请求太深的页码，或者大量的结果](https://www.elastic.co/guide/en/elasticsearch/guide/current/pagination.html)


##Search Lite
query-string是很有用的，在运行一个临时的查询命令行的的时候。
 For instance, this query finds all documents of type tweet that contain the word elasticsearch in the tweet field:
```
 GET /_all/tweet/_search?q=tweet:elasticsearch
```
The next query looks for john in the name field and mary in the tweet field. The actual query is just
```
+name:john +tweet:mary
```
The + prefix indicates conditions that must be satisfied for our query to match. Similarly a - prefix would indicate conditions that must not match. All conditions without a + or - are optional—the more that match, the more relevant the document.

This simple search returns all documents that contain the word mary:
```
GET /_search?q=mary
```
the results from this query mention mary in three fields:

* A user whose name is Mary
* Six tweets by Mary
* One tweet directed at @mary

When you index a document, Elasticsearch takes the string values of all of its fields and concatenates them into one big string, which it indexes as the special _all field. For example, when we index this document:
```
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}

```
it’s as if we had added an extra field called _all with this value:
```
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"

```

**The query-string search uses the _all field unless another field name has been specified.**


The next query searches for tweets, using the following criteria:

* The name field contains mary or john
* The date is greater than 2014-09-10
* The _all field contains either of the words aggregations or geo

```
+name:(mary john) +date:>2014-09-10 +(aggregations geo)
```
更多query string 看这里
[Query String Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/query-dsl-query-string-query.html#query-string-syntax)

但是querystring有问题
* 难以debug   
* And it’s fragile—a slight syntax error in the query string, such as a misplaced -, :, /, or ", and it will return an error instead of results.
* Finally, the query-string search allows any user to run potentially slow, heavy queries on any field in your index, possibly exposing private information or even bringing your cluster to its knees!

**Tip：**
For these reasons, we don’t recommend exposing query-string searches directly to your users, unless they are power users who can be trusted with your data and with your cluster.


[Mapping and Analysis](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping-analysis.html)


##Mapping一些要注意的

字段类型string 默认是是包含全文的。所以它会被analysis
indexedit
The index attribute controls how the string will be indexed. It can contain one of three values:

analyzed
index this field as full text.

not_analyzed
可以被搜索，但不被analysis

no
根本就不建立这个字段的索引，所以不能被搜索

默认是analyzed，我们也可以自定义
```
{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}
```

**Note：**
The other simple types (such as long, double, date etc) also accept the index parameter, but the only relevant values are no and not_analyzed, as their values are never analyzed.

对于analyzed的字段，可以设置analyzer
>{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}

在创建index的时候创建mapping,映射
你也可以增加新的字段映射，但你不能更改已经存在的字段映射，因为你一旦改了，先前mapping的数据将不能被正确搜索。


es中的数组就是一个多个值的包，没有顺序，不是我们平常理解的数组，通过下标访问单个值，只知道里面有没有包含一个值。


##想要发挥搜索的全部威力，就用 Body SEARCH API

https://www.elastic.co/guide/en/elasticsearch/guide/current/full-body-search.html

DSL 包含两个上下文：
* filtering context
*  query context

[link——filter query](https://blog.csdn.net/laoyang360/article/details/80468757)

filtering context 的回答很简单，就是是或者不是。这是没有得分的查询
query context 是计分查询，查询结果是不缓存的

性能上filtering 更快

如何选择用哪个？
一般对关系分数的情况使用分数，其他情况用filter



几个重要的query

>match_all 简单查询所有文档，在没有任何query别定义的时候这就是默认的query
```
{ "match_all": {}}
```
>match The match query should be the standard query that you reach for whenever you want to query for a full-text or exact value in almost any field.

在你查询前，他会使用正确的analyzer analysis querystring然后再执行查询

```
{ "match": { "tweet": "About Search" }}

```

If you use it on a field containing an exact value, such as a number, a date, a Boolean, or a not_analyzed string field, then it will search for that exact value:
```
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```

**Tip**
For exact-value searches, you probably want to use a filter clause instead of a query, as a filter will be cached. We’ll see some filtering examples soon.


>The multi_match query allows to run the same match query on multiple fields:

```
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```
>The range query allows you to find numbers or dates that fall into a specified range:
```
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}

gt
Greater than
gte
Greater than or equal to
lt
Less than
lte
Less than or equal to
```


> term Queryedit
The term query is used to search by exact values, be they numbers, dates, Booleans, or not_analyzed exact-value string fields:

```
{ "term": { "age":    26           }}
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}

 ```
The term query performs no analysis on the input text, so it will look for exactly the value that is supplied.


> terms Queryedit
The terms query is the same as the term query, but allows you to specify multiple values to match. If the field contains any of the specified values, the document matches:

```
{ "terms": { "tag": [ "search", "full_text", "nosql" ] }}

```
Like the term query, no analysis is performed on the input text. It is looking for exact matches (including differences in case, accents, spaces, etc).

>The exists and missing queries are used to find documents in which the specified field either has one or more values (exists) or doesn’t have any values (missing). It is similar in nature to IS_NULL (missing) and NOT IS_NULL (exists)in SQL:
```
{
    "exists":   {
        "field":    "title"
    }
}
```

Although not used nearly as often as the bool query, the _**constant_score query**_ is still useful to have in your toolbox. The query applies a static, constant score to all matching documents. It is predominantly used when you want to execute a filter and nothing else (e.g. no scoring queries).

You can use this instead of a bool that only has filter clauses. Performance will be identical, but it may aid in query simplicity/clarity.
```
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" } 
        }
    }
}
```

A term query is placed inside the constant_score, converting it to a non-scoring filter. This method can be used in place of a bool query which only has a single filter

# 超级重要的
**在查询中**

**must**
>Clauses that must match for the document to be included.

**must_not**
>Clauses that must not match for the document to be included.

**should**
>If these clauses match, they increase the _score; otherwise, they have no effect. They are simply used to refine the relevance score for each document.


**在过滤器中**

**must**
>所有的语句都 必须（must） 匹配，与 AND 等价。

**must_not**
>所有的语句都 不能（must not） 匹配，与 NOT 等价。

**should**
至少有一个语句要匹配，与 OR 等价。


区别就在于两个 should 语句，也就是说：一个文档不必包含两个词项，但如果一旦包含，我们就认为它们 更相关 ：
```
GET /my_store/products/_search
{
   "query" : {
      "filtered" : { 
         "filter" : {
            "bool" : {
              "should" : [
                 { "term" : {"price" : 20}}, 
                 { "term" : {"productID" : "XHDK-A-1293-#fJ3"}} 
              ],
              "must_not" : {
                 "term" : {"price" : 30} 
              }
           }
         }
      }
   }
}
```


**filter**
>Clauses that must match, but are run in non-scoring, filtering mode. These clauses do not contribute to the score, instead they simply include/exclude documents based on their criteria.

[这里用结合sql语句的组合过滤器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/combining-filters.html)

range 查询可同时提供包含（inclusive）和不包含（exclusive）这两种范围表达式，可供组合的选项如下：

* gt: > 大于（greater than）
* lt: < 小于（less than）
* gte: >= 大于或等于（greater than or equal to）
* lte: <= 小于或等于（less than or equal to）

```
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "price" : {
                        "gte" : 20,
                        "lt"  : 40
                    }
                }
            }
        }
    }
}
```

**Tip**
If there are no must clauses, at least one should clause has to match. However, if there is at least one must clause, no should clauses are required to match.


If we don’t want the date of the document to affect scoring at all, we can re-arrange the previous example to use a filter clause:
```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }} 
        }
    }
}
```



[在写复杂查询的时候，有时候会出现一些问题，这时候我们希望它能告诉我们一些信息](https://www.elastic.co/guide/en/elasticsearch/guide/current/_validating_queries.html)



[除了默认的相关性排序，还有自定义的排序](https://www.elastic.co/guide/en/elasticsearch/guide/current/_sorting.html#_sorting)
[常见排序一般有如下两类：
（1）按照相似度匹配得分排序
（2）按照指定字段排序](https://blog.csdn.net/kjsoftware/article/details/76292911)

能被解析（analyzed）的字段也是多值字段，这是因为可被解析的字段被拆分，各自对term键索引，这就导致了对字段的存储也是拆分的。在排序的过程中，它不知道按哪个字段排序，这样会导致排序结果的不可确定性。


由于它是拆开存储的，所以你只要存的时候是一个简单顺序就行，不用嵌套，只要mapping规定了他们之间的关系，存入的字段放到那个关系下。很方便的。

## es中使用的关联度算法
term frequency/inverse document frequency

 TF/IDF
 
 **Term frequency**
 >term在一个字段中出现的频率，出现五次明显比只出现一次相关度高。
 
 **Inverse document frequency**
>term在全文中出现频率很高，则相关度低。这是因为有些词大家都常用，反而要尽量减少这些词的影响。
 
 **Field-length norm**
 >这个字段有多长，越长，则这个相关度越低。

 es也会考虑其他因素来计算得分。比如，the term proximity in phrase queries, or term similarity in fuzzy queries.
Relevance is not just about full-text search, though. It can equally be applied to yes/no clauses, where the more clauses that match, the higher the _score.

**When multiple query clauses are combined using a compound query like the bool query, the _score from each of these query clauses is combined to calculate the overall _score for the document.**

[整个关于控制相关度计算的章节](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html)


##doc values
doc values 是一种列式的存储结构，在字段值被建立倒排索引时，也会存入各自的doc values中。倒排索引为了快速检索，doc values为了方便以下操作。
Elasticsearch 中的 Doc Values 常被应用到以下场景：

* 对一个字段进行排序
* 对一个字段进行聚合
* 某些过滤，比如地理位置过滤
* 某些与字段相关的脚本计算
因为文档值被序列化到磁盘，我们可以依靠操作系统的帮助来快速访问。当 working set 远小于节点的可用内存，系统会自动将所有的文档值保存在内存中，使得其读写十分高速； 当其远大于可用内存，操作系统会自动把 Doc Values 加载到系统的页缓存中，从而避免了 jvm 堆内存溢出异常。

我们稍后会深入讨论 `Doc Values`。现在所有你需要知道的是排序发生在索引时建立的平行数据结构中。

##**搜索类型编辑**
缺省的搜索类型是 query_then_fetch 。 在某些情况下，你可能想明确设置 search_type 为 dfs_query_then_fetch 来改善相关性精确度：

```
GET /_search?search_type=dfs_query_then_fetch

```
搜索类型 dfs_query_then_fetch 有预查询阶段，这个阶段可以从所有相关分片获取词频来计算全局词频。 我们在 [被破坏的相关度！](https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-is-broken.html) 会再讨论它。

## [scroll search](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scroll.html)

scroll 查询 可以用来对 Elasticsearch 有效地执行大批量的文档查询，而又不用付出深度分页那种代价。

## analyzer
standard 分析器是用于全文字段的默认分析器， 对于大部分西方语系来说是一个不错的选择。 它包括了以下几点：

* standard 分词器，通过单词边界分割输入的文本。
* standard 语汇单元过滤器，目的是整理分词器触发的语汇单元（但是目前什么都没做）。
* lowercase 语汇单元过滤器，转换所有的语汇单元为小写。
* stop 语汇单元过滤器，删除停用词--对搜索相关性影响不大的常用词，如 a ， the ， and ， is 。
分析器可以从三个层面进行定义：按字段（per-field）、按索引（per-index）或全局缺省（global default）。Elasticsearch 会按照以下顺序依次处理，直到它找到能够使用的分析器。索引时的顺序如下：

字段映射里定义的 analyzer ，否则
索引设置中名为 default 的分析器，默认为
standard 标准分析器

在搜索时，顺序有些许不同：

查询自己定义的 analyzer ，否则
字段映射里定义的 analyzer ，否则
索引设置中名为 default 的分析器，默认为
standard 标准分析器


在下面的例子中，我们创建了一个新的分析器，叫做 es_std ， 并使用预定义的 西班牙语停用词列表：
```
PUT /spanish_docs
{
    "settings": {
        "analysis": {
            "analyzer": {
                "es_std": {
                    "type":      "standard",
                    "stopwords": "_spanish_"
                }
            }
        }
    }
}
```
es_std 分析器不是全局的--它仅仅存在于我们定义的 spanish_docs 索引中。 为了使用 analyze API来对它进行测试，我们必须使用特定的索引名：
[自定义分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html)
对于整个索引，映射在本质上被 扁平化 成一个单一的、全局的模式。这就是为什么两个类型不能定义冲突的字段：当映射被扁平化时，Lucene 不知道如何去处理。

[ dynamic mapping ](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-mapping.html)
[自定义动态映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-dynamic-mapping.html)
当 Elasticsearch 遇到文档中以前 未遇到的字段，它用 dynamic mapping 来确定字段的数据类型并自动把新的字段添加到类型映射。

有时这是想要的行为有时又不希望这样。通常没有人知道以后会有什么新字段加到文档，但是又希望这些字段被自动的索引。也许你只想忽略它们。如果Elasticsearch是作为重要的数据存储，可能就会期望遇到新字段就会抛出异常，这样能及时发现问题。


**tip**
做好准备：在你的应用中使用别名而不是索引名。然后你就可以在任何时候重建索引。别名的开销很小，应该广泛使用。做到不停机，更新。
https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-aliases.html
* _aliases原子化操作
* _alias


## [近实时搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/near-real-time.html)
 Elasticsearch 是 近 实时搜索: 文档的变化并不是立即对搜索可见，但会在一秒之内变为可见。

这些行为可能会对新用户造成困惑: 他们索引了一个文档然后尝试搜索它，但却没有搜到。这个问题的解决办法是用 refresh API 执行一次手动刷新:
```
POST /_refresh  
POST /blogs/_refresh 
```
刷新（Refresh）所有的索引。
只刷新（Refresh） blogs 索引。

>提示
尽管刷新是比提交轻量很多的操作，它还是会有性能开销。 当写测试的时候， 手动刷新很有用，但是不要在生产环境下每次索引一个文档都去手动刷新。 相反，你的应用需要意识到 Elasticsearch 的近实时的性质，并接受它的不足。



##深入搜索
搜索不仅仅是全文搜索：我们很大一部分数据都是结构化的，如日期和数字。 我们会以说明结构化搜索与全文搜索最高效的结合方式开始本章的内容。
https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-in-depth.html

term就是精确查询

从概念上记住非评分计算是首先执行的，这将有助于写出高效又快速的搜索请求。




##影响评分
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```
为什么将译者条件语句放入另一个独立的 bool 查询中呢？所有的四个 match 查询都是 should 语句，所以为什么不将 translator 语句与其他如 title 、 author 这样的语句放在同一层呢？

答案在于评分的计算方式。 bool 查询运行每个 match 查询，再把评分加在一起，然后将结果与所有匹配的语句数量相乘，最后除以所有的语句数量。处于同一层的每条语句具有相同的权重。在前面这个例子中，包含 translator 语句的 bool 查询，只占总评分的三分之一。如果将 translator 语句与 title 和 author 两条语句放入同一层，那么 title 和 author 语句只贡献四分之一评分。

**语句的优先级**

前例中每条语句贡献三分之一评分的这种方式可能并不是我们想要的， 我们可能对 title 和 author 两条语句更感兴趣，这样就需要调整查询，使 title 和 author 语句相对来说更重要。

在武器库中，最容易使用的就是 boost 参数。为了提升 title 和 author 字段的权重， 为它们分配的 boost 值大于 1 ：

```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { 
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { 
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { 
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
```

要获取 boost 参数 “最佳” 值，较为简单的方式就是不断试错：设定 boost 值，运行测试查询，如此反复。 boost 值比较合理的区间处于 1 到 10 之间，当然也有可能是 15 。如果为 boost 指定比这更高的值，将不会对最终的评分结果产生更大影响，因为评分是被 归一化的（normalized） 。



通过filter取消参与评分


通过should must must_not 规定一部分

dis_max只取某一个query最大的分数，完全不考虑其他query的分数
tie_breaker 优化 dis_max
使用tie_breaker将其他query的分数也考虑进去

tie_breaker参数的意义，在于说，将其他query的分数，乘以tie_breaker，然后综合与最高分数的那个query的分数，综合在一起进行计算
除了取最高分以外，还会考虑其他的query的分数
tie_breaker的值，在0~1之间，是个小数，就ok
```
GET /forum/article/_search
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "java beginner" }},
                { "match": { "body":  "java beginner" }}
            ],
            "tie_breaker": 0.3
        }
    }
}

--------------------- 
作者：chenshiying007 
来源：CSDN 
原文：https://blog.csdn.net/qq_27384769/article/details/79645487 
版权声明：本文为博主原创文章，转载请附上博文链接！
```

[查询语句提升权重](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_boosting_query_clauses.html)

[有关模糊字段名查询的权重](https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-match-query.html)


[利用子列改变分数](https://www.elastic.co/guide/cn/elasticsearch/guide/current/most-fields.html)

子列是为了方便同一键值对发生不同的行为，比如同一键值对建立两次不同的索引。


[fullname的意义](https://www.elastic.co/guide/cn/elasticsearch/guide/current/field-centric.html)
其中由于每个查询的权重一样，所以可能出现重复同一字段值得比都命中的权重高，因为都命中的字段只算了一份权重。_all 字段的索引方式是将所有其他字段的值作为一个大字符串索引的也是这个道理、。但是这样做并不在灵活，所以我们需要自定义

[索引时 自定义_all 字段](https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-all.html#custom-all)

[搜索时自定义_all字段](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_cross_fields_queries.html)
跨字段查询是词(term)中心查询，我不管你字段怎样，所以就把所有字段合并成一个字段，只关注这个文档中整体的内容。


[短语匹配中如何提高灵活性](https://www.elastic.co/guide/cn/elasticsearch/guide/current/slop.html)


[搜索term的一部分](https://www.elastic.co/guide/cn/elasticsearch/guide/current/partial-matching.html)


[控制相关度的实际例子](https://www.elastic.co/guide/cn/elasticsearch/guide/current/controlling-relevance.html)

