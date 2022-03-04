---
title: ElasticSearch
date: 2021-11-15 15:35:56
tags: 
- ElasticSearch
categories:
- ElasticSearch
index_img: /img/elk.png


---



ElasticSearch基本使用

<!--more-->





设置本机为固定ip

- ipv4协议 属性

mysql远程连接配置

#  一、ElasticSearch基础

## 1.1、ES介绍

1、ES核心术语

- 索引 index           表
- 类型 type             表逻辑类型 (es 7.x版本已删去)
- 文档 document    行
- 字段 fields            列

2、ES核心概念

- 映射 mapping      表结构定义
- 近实时 NRT          Near real time 
- 节点 node            每一个服务器
- shard replica        数据分片 备份

3、ES集群架构原理

	>将数据进行分片,并行计算
	>
	>shard(主分片) 	 replica(备份分片)

## 1.2、倒排索引

> ElasticSearch 倒排索引
>
> 倒排索引源于实际应用中需要根据属性的值来查找记录。这种索引表中的每一项都包括一个属性值和包含该属性值的各个记录地址。由于不是根据记录来确定属性，而是根据属性来确定记录的位置，所以称之为倒排索引。





## 1.3、ES环境搭建

1.解压es压缩包 到 `/usr/local/elasticsearch` 下

`tar -zxvf /root/workspace/software/elasticsearch-7.15.2-linux-x86_64.tar.gz -C ./`

2.在 `elasticsearch` 下新建 data 文件夹 数据目录

​											logs 日志目录 

3.es配置文件 `elasticsearch.yml`

```yaml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: tho-elasticsearch
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: es-node1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /usr/local/elasticsearch/elasticsearch-7.15.2/data
#
# Path to log files:
#
path.logs: /usr/local/elasticsearch/elasticsearch-7.15.2/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
#
network.host: 0.0.0.0
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
#
#http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["es-node1"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true

```

4.配置文件 `jvm.options`

> 修改  根据机器配置来选择 
>
> ​	-Xms128m
> ​	-Xmx128m

5.es不允许root用户进行操作,要新创建用户

```JAVA
useradd esuser

进入 es文件夹
[root@centos_7_100 elasticsearch-7.15.2]# pwd
/usr/local/elasticsearch/elasticsearch-7.15.2 

chown -R esuser /usr/local/elasticsearch/elasticsearch-7.15.2
  
[root@centos_7_100 elasticsearch-7.15.2]# ll
总用量 636
drwxr-xr-x.  2 esuser root   4096 11月  4 22:08 bin
drwxr-xr-x.  3 esuser root    169 11月 25 23:17 config
drwxr-xr-x.  2 esuser root      6 11月 25 23:28 data
drwxr-xr-x.  9 esuser root    121 11月  4 22:08 jdk
drwxr-xr-x.  3 esuser root   4096 11月  4 22:08 lib
-rw-r--r--.  1 esuser root   3860 11月  4 22:02 LICENSE.txt
drwxr-xr-x.  2 esuser root      6 11月  4 22:06 logs
drwxr-xr-x. 60 esuser root   4096 11月  4 22:08 modules
-rw-r--r--.  1 esuser root 628969 11月  4 22:06 NOTICE.txt
drwxr-xr-x.  2 esuser root      6 11月  4 22:06 plugins
-rw-r--r--.  1 esuser root   2710 11月  4 22:02 README.asciidoc

```

6.进入 es bin目录 切换user `sudo esuser` 执行命令  `./elasticsearch`

7.报错 处理

>ERROR: [3] bootstrap checks failed. You must address the points described in the following [3] lines before starting Elasticsearch.
>bootstrap check failure [1] of [3]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
>bootstrap check failure [2] of [3]: max number of threads [3795] for user [esuser] is too low, increase to at least [4096]
>bootstrap check failure [3] of [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

8.`/etc/security` 目录下 `limits.conf` 修改该配置文件

其中加如下配置

```java
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* soft nproc 4096
```

9. `/etc` 下 修改  sysctl.conf

```JAVA
vm.max_map_count=262144
```

`su esuser` `su root` 切换用户

> bin目录下运行 `./elasticsearch`

10.开启 9200 9300 端口

11.  `./elasticsearch -d ` 后台运行
12. `jps` 命令 查看es进程信息

## 1.4、ES可视化工具

> elasticsearch-head
>
> https://github.com/mobz/elasticsearch-head

1.使用chrome 扩展程序 插件

2.使用 本地服务来启动 es-head

>#### Running with built in server
>
>- `git clone git://github.com/mobz/elasticsearch-head.git`
>- `cd elasticsearch-head`
>- `npm install`
>- `npm run start`
>
>- `open` http://localhost:9100/
>
>This will start a local webserver running on port 9100 serving elasticsearch-head

配置es跨域

elasticsearch.yml 配置文件修改

```JAVA
http.cors.enabled: true
http.cors.allow-origin: "*"
```

# 二、ElasticSearch基本使用

## 2.1、head插件基本操作

> 可以用可视化操作es ，也可以用postman 直接调用接口来操作es

## 2.2、mappings映射

```JSON
{
    "mappings": {
        "properties": {
            "realname": {
                "type": "text",
                "index": true
            },
            "username": {
                "type": "keyword",
                "index": false
            }

        }
    }
}
```

> text 类型会分词
>
> keyword 类型无需分词,是精确的,精确匹配



> mappings 查看分词效果

```JSON
text 类型
http://192.168.198.100:9200/index_mapping/_analyze
{
    "field": "realname",
    "text": "hello world"
}
返回消息体
{
    "tokens": [
        {
            "token": "hello",
            "start_offset": 0,
            "end_offset": 5,
            "type": "<ALPHANUM>",
            "position": 0
        },
        {
            "token": "world",
            "start_offset": 6,
            "end_offset": 11,
            "type": "<ALPHANUM>",
            "position": 1
        }
    ]
}
```

```JSON
keyword类型
http://192.168.198.100:9200/index_mapping/_analyze
{
    "field": "username",
    "text": "hello world"
}
返回消息体
{
    "tokens": [
        {
            "token": "hello world",
            "start_offset": 0,
            "end_offset": 11,
            "type": "word",
            "position": 0
        }
    ]
}
```

> 索引的类型一旦确定,是不能修改的

> 已存在mappings 增加新的mappings 字段

```json
http://192.168.198.100:9200/index_mapping/_mapping
{
    "properties": {
            "id": {
                "type": "long"
            },
            "age": {
                "type": "keyword"
            }

        }
}
```

> 主要数据类型
>
> - text , keyword
> - long, integer, short, byte
> - double, float
> - boolean
> - date
> - object
> - 数组不能混,类型一致

## 2.3、文档基本操作

>- _index: 文档数据所属那个索引,理解为数据库的某张表即可。
>- _type: 文档数据属于哪个类型，新版本使用   _doc
>- _id: 文档数据的唯一标识，类似数据库中某张表的主键，可以自动生成或手动指定
>- _score: 查询相关度，是否契合用户匹配，分数越高用户的搜索体验越高
>- _version: 版本号
>- _source: 文档数据，json格式

> 添加文档

```JSON
POST http://192.168.198.100:9200/my_doc/_doc/1  // 数字1代表此条记录在es索引中的id信息
{																								// 此参数不写 es 会自动生成一个id
    "id": 1001,   // 这个 “id” 信息表示在mysql等数据库中的id信息
    "name": "tho-1",
    "desc": "hello world！",
    "create_date": "2021-11-27"
}
```

> 文档的删除和修改

```JSON
DELETE http://192.168.198.100:9200/my_doc/_doc/9
// 返回值body
{
    "_index": "my_doc",
    "_type": "_doc",
    "_id": "9",
    "_version": 2,
    "result": "deleted",
    "_shards": {
        "total": 1,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
// 多次删除同一个文档记录,_version 会累加,result 会显示 not_found
// 此时删除是逻辑删除,当磁盘空间紧张时,才会真正物理上的删除
```

> 文档修改-局部修改

```JSON
POST http://192.168.198.100:9200/my_doc/_doc/1/_update // 1为es中文档记录的id
// 局部修改 只修改了传的参数
{
    "doc": {
        // 要修改的字段信息
        "name": "tho-778"       
    }
}
// 返回消息体
{
    "_index": "my_doc",
    "_type": "_doc",
    "_id": "1",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 1,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 12,
    "_primary_term": 1
}
```

> 文档修改-全部修改

```JSON
PUT http://192.168.198.100:9200/my_doc/_doc/1
{
    "id": 1066,
    "name": "thottt",
    "desc": "tho world 9",
    "create_date": "2021-11-26"
}
// 返回消息体
{
    "_index": "my_doc",
    "_type": "_doc",
    "_id": "1",
    "_version": 3,
    "result": "updated",
    "_shards": {
        "total": 1,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 13,
    "_primary_term": 1
}
```

> 文档查询

```JSON
GET http://192.168.198.100:9200/my_doc/_doc/1
// 返回消息体
{
    "_index": "my_doc",
    "_type": "_doc",
    "_id": "1",
    "_version": 3,
    "_seq_no": 13,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "id": 1066,
        "name": "thottt",
        "desc": "tho world 9",
        "create_date": "2021-11-26"
    }
}
```

> 查询所有文档数据

```JSON
GET http://192.168.198.100:9200/my_doc/_doc/_search
// 查询指定字段
GET http://192.168.198.100:9200/my_doc/_doc/1?_source=id,name 
// 判断一个记录是否存在  返回状态码是 200 存在该记录
// 																	404 不存在该记录
HEAD http://192.168.198.100:9200/my_doc/_doc/5
```

## 2.4、ES乐观锁

> 基于 _seq_no 字段 和 _primary_term 字段
>
> - _seq_no 			序列号,类似 version版本号
> - _primary_term   表示该记录所在分片的标识

```JSON
POST http://192.168.198.100:9200/my_doc/_doc/2001/_update?if_seq_no=16&if_primary_term=1
// 对应的if_seq_no if_primary_term 要正确,否则不能完成修改
```

## 2.5、ES分词器

### 1、ES分词基础概念

> 分词：把文本转换为一个个单词,分词成为analysis。es默认只对英文语句做分词，中文不支持，每个中文字都会被拆分为独立的个体

- es内置分词器

> standard：默认分词，单词会被拆分。大写会转换为小写
>
> simple：按照非字母分词，大写转换为小写
>
> whitespace：按照空格分词，忽略大小写
>
> stop：去除无意义单词，比如 the/is/an/a ...
>
> keyword：不做分词，把整个文本作为一个单独的关键字

```JSON
POST http://192.168.198.100:9200/my_doc/_analyze
// 返回消息体
{
    "tokens": [
        {
            "token": "hello",
            "start_offset": 0,
            "end_offset": 5,
            "type": "<ALPHANUM>",
            "position": 0
        },
        {
            "token": "world",
            "start_offset": 6,
            "end_offset": 11,
            "type": "<ALPHANUM>",
            "position": 1
        }
    ]
}
```



### 2、建立ik中文分词器

> elasticsearch-analysis-ik-7.15.2 对应es的版本
>
> 环境配置
>
> 1. ik_max_word 和 ik_smart 有什么区别？
>
> ik_max_word: 泡沫文本做最细粒度的分裂，中华爱国“中华人民共和国国歌”分裂为中华人民共和国“，和、国国、国歌”，会穷尽各种可能的组合，适合词条查询；
>
> ik_smart：会做最粗粒度的拆分，比如“中华人民共和国国歌”拆分为“中华人民共和国，国歌”，适合短语查询。



1.解压到elasticsearch 下 plugins 下 的 ik文件夹下

`unzip elasticsearch-analysis-ik-7.15.2.zip -d /usr/local/elasticsearch/elasticsearch-7.15.2/plugins/ik/`

2.重启es即可生效

3.测试

```JSON
POST http://192.168.198.100:9200/my_doc/_analyze
// 参数
{
    "analyzer": "ik_max_word",
    "field": "desc",
    "text": "上下班车流量很大"
}
// 返回结果体
{
    "tokens": [
        {
            "token": "上下班",
            "start_offset": 0,
            "end_offset": 3,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "上下",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 1
        },
        {
            "token": "下班",
            "start_offset": 1,
            "end_offset": 3,
            "type": "CN_WORD",
            "position": 2
        },
        {
            "token": "班车",
            "start_offset": 2,
            "end_offset": 4,
            "type": "CN_WORD",
            "position": 3
        },
        {
            "token": "车流量",
            "start_offset": 3,
            "end_offset": 6,
            "type": "CN_WORD",
            "position": 4
        },
        {
            "token": "车流",
            "start_offset": 3,
            "end_offset": 5,
            "type": "CN_WORD",
            "position": 5
        },
        {
            "token": "流量",
            "start_offset": 4,
            "end_offset": 6,
            "type": "CN_WORD",
            "position": 6
        },
        {
            "token": "很大",
            "start_offset": 6,
            "end_offset": 8,
            "type": "CN_WORD",
            "position": 7
        }
    ]
}
```

### 3、自定义中文词库

- 配置文件路径

`/usr/local/elasticsearch/elasticsearch-7.15.2/plugins/ik/config` 下的 `IKAnalyzer.cfg.xml`

1.config 目录下新建词库文件 custom.dic 内容为要配置的中文词汇

2.配置文件`IKAnalyzer.cfg.xml` 修改

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">custom.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>

```

3.重启es

> 之后再进行分词,custom.dic 中的中文词汇会被自动进行分词



# 三、DSL搜索

## 3.1、数据准备

> 词库 建立索引  手动建立 mappings

```JSON
POST http://192.168.198.100:9200/tho/_mapping
{
    "properties":{
        "id":{
            "type":"long"
        },
        "age":{
            "type":"integer"
        },
        "username":{
            "type":"keyword"
        },
        "nickname":{
            "type":"text",
            "analyzer":"ik_max_word"
        },
        "money":{
            "type":"float"
        },
        "desc":{
            "type":"text",
            "analyzer":"ik_max_word"
        },
        "sex":{
            "type":"byte"
        },
        "birthday":{
            "type":"date"
        },
        "face":{
            "type":"text",
            "index":false
        }
    }
}
```

> 录入数据记录

## 3.2、DSL入门语法

> QueryString方式

```JSON
// 基本查询
GET http://192.168.198.100:9200/tho/_search?q=desc:慕课网 
// 该查询查询条件是 desc:慕课网


// 多条件查询
GET http://192.168.198.100:9200/tho/_search?q=desc:慕课网&q=age:18
```

> DSL搜索
>
> QueryString用的很少，一旦参数复杂就难以构建，所以大多数都会使用DSL来进行查询
>
> - Domain Specific Language
> - 特定领域语言
> - 基于JSON格式的数据查询
> - 查询更灵活，有利于复杂查询

```json 
// DSL搜索
POST http://192.168.198.100:9200/tho/_doc/_search
// 请求体
{
    "query": {
        "match": {
            "desc": "慕课网"
        }
    }
}

// 查询某字段是否存在
// 请求体
{
    "query":{
        "exists": {
            "field": "username"
        }
    }
}
// 查询所有记录
{
    "query":{
        "match_all": {
            
        }
    }
} 

// 选择查询的字段 id nickname age
{
    "query":{
        "match_all": {
            
        }
    },
    "_source": [
        "id","nickname","age"
    ]
}

// 分页
{
    "query":{
        "match_all": {
            
        }
    },
    "_source": [
        "id","nickname","age"
    ],
    "from": 12,
    "size": 5
}

// term  精确搜索 查询关键字不会被分词处理
// match 分词搜索 查询关键字会被分词处理
{
    "query":{
        "match": {
            "desc":"百度"
        }
    },
    "_source": [
        "id", "nickname","desc"
    ]
}

{
    "query":{
        "term": {
            "desc":"百度搜索"
        }
    },
    "_source": [
        "id", "nickname","desc"
    ]
}

// terms 多个词语匹配检索
// 类似于标签,会进行多个词语的检索
{
    "query":{
        "terms": {
            "desc": ["百度搜索","打游戏"]
        }
    },
    "_source": [
        "id", "nickname","desc"
    ]
}

// match_phrase：分词结果必须在text字段分词中都包含，而且顺序必须相同，而且必须都是连续的。（搜索比较严格）
// 字段内容必须是 百度搜索es
{
    "query":{
        "match_phrase": {
            "desc": {
                "query": "百度搜索 es"
                "slop": 5 // slop 允许中间跳过字符的数量来匹配
            }
        }
    },
    "_source": [
        "id", "nickname","desc"
    ]
}

// and or 运算符
// minimum_should_match 匹配程度达到多少以上才会显示
//	"minimum_should_match":"60%" 10个分词 6个以上匹配才会显示
// 	"minimum_should_match":"3"    3个以上分词匹配才会显示
{
    "query":{
        "match": {
            "desc": {
                "query": "慕课网 游泳",
                "operator": "or",
                "minimum_should_match":"100%"
            }
        } 
    },
    "_source": [
        "id", "nickname","desc"
    ]
}

// DSL方式根据id查询
{
    "query":{
        "ids": {
            "type": "_doc",
            "values":["1001","1003","1009"]
        } 
    },
    "_source": [
        "id", "nickname","desc"
    ]
}

// multi_match 满足使用match 在多个字段中进行查询的需求
// 多个字段 其中有满足 query 条件的就会被查询出来
{
    "query":{
        "multi_match": {
            "query": "游泳 慕课网",
            "fields":["desc","nickname"]
        } 
    },
    "_source": [
        "id", "nickname","desc"
    ]
}
// "nickname^10" 提升某个字段的权重
{
    "query":{
        "multi_match": {
            "query": "游泳 慕课网",
            "fields":[
                "desc","nickname^10"
            ]
        } 
    },
    "_source": [
        "id", "nickname","desc"
    ]
}

// DSL 布尔查询 多条件查询
{
    "query":{
        "bool": {
          	// "must" 下面条件都是 and
          	// "should" 下面条件都是 or
          	// "must_not" 下面条件都不满足(都是非)
            "must": [
              // 条件1
                { "multi_match": {
                    "query":"百度",
                    "fields": [
                        "desc", "nickname"
                    ]
                    }
                },
              // 条件2 and 条件
                {
                    "term": {
                        "sex": 1
                    }
                }  
            ]
            
        }
    },
    "_source": [
        "id", "nickname","sex","desc"
    ]
}
// 条件组合查询
{
    "query":{
        "bool": {
            
            "should": [
                {
                    "match": {
                        "nickname": "ns"
                    }
                },{
                    "match": {
                        "desc": "百度搜索"
                    }
                }
            ],
            "must": [
                {
                    "match": {
                        "desc": "百度"
                    }
                }
            ],
            "must_not": [
                {
                    "match": {
                        "nickname": "巨硬"
                    }
                }
            ]
            
        }
    },
    "_source": [
        "id", "nickname","sex","desc"
    ]
}
```

## 3.3、DSL查询-布尔查询

> - must：查询必须匹配搜索条件，相当于 and
> - should：查询匹配满足1个以上条件，相当于or
> - must_not：不匹配搜索条件，一个都不要满足

## 3.4、DSL高级查询

> 过滤器
>
> 对搜索出来的结果进行数据过滤。不会到es库里去搜，不会去计算文档的相关度分数，所以过滤的性能会比较高，过滤器可以和全文搜索结合在一起使用。 post_filter元素是一个顶层元素，只会对搜索结果进行过滤。不会计算数据的匹配度相关性分数，不会根据分数去排序，query则相反，会计算分数，也会按照分数进行显示

```JSON
// 根据money进行过滤
{
    "query":{
        "match": {
            "desc": "百度搜索"
        }
    },
    "post_filter": {
        "range": {
            "money": {
                "gt": 60,
                "lt": 1000
            }
        }
    }
}
```

> 排序
>
> keyword 类型可以进行排序
>
> 对文本进行排序
>
> - 由于文本会被分词，所以往往要去做排序会报错，通常我们可以为这个字段增加额外的一个附属属性，类型为keyword，用于做排序。

```JSON
// 指定某个字段及其对应的排序规则
{
    "query":{
        "match": {
            "desc": "百度搜索"
        }
    },
    "sort": [
        {
            "age": "desc"
        },
        {
            "money": "desc"
        }
    ]
}

// 根据keyword来进行排序
// 某个字段的附属属性 keyword来进行排序
{
    "query":{
        "match": {
            "name": "tho"
        }
    },
    "sort": [
        {
            "name.keyword": "desc"
        }
    ]
}
```

> 搜索结果高亮

```JSON
{
    "query":{
        "match": {
            "desc": "谷歌"
        }
    },
    "highlight": {
        "pre_tags": ["<span>"], //自定义标签 <span></span>
        "post_tags": ["</span>"],// 默认标签 <em></em>
        "fields": {
            "desc": {}
        }
    }
}
```

> 进阶
>
> - prefix  根据前缀去查询
>
> - fuzzy    模糊搜索，并不是sql的模糊搜索，而是在用户进行搜索时打字错误，搜索引擎会自动更正，然后尝试匹配索引数据库中的数据
>
> - wildcard  占位符查询
>
> 	？：1个字符
>
> 	*：1个或多个字符

```JSON
// prefix
{
	"query": {
			"prefix": {
					"desc": "imo"
			 }
	}
}
// fuzzy
{
		"query": {
				"multi_match": {
						"fields": [ "desc", "nickname"],
						 "query": "imcoc supor",
						  "fuzziness": "AUTO"
			 	}
		}
}
```

# 四、ES进阶使用

## 4.1、ES深度分页

### 1.限制分页深度

> 分页查询
>
> 查询深度过深性能会很差
>
> - 限制分页页数
>
> ​    在es中获取9999条数据到10009条数据的时候，其实每个分片都会拿到10009条数据，然后集合在一起，总共是10009*3=30027条数据，针对30027条数据再次做排序，获取最后的10条数据
>
> ​	如此一来，搜索地太深，就会造成性能问题，会耗费内存和占用CPU。而且es为了性能，不支持超过一万条数据以上的分页查询。为了避免深度分页问题，要限制分页页数，每页展示100条数据，最多提供100页的展示，从101页开始就没了，用户也不会搜索需要100页，淘宝的分页限制就是100页

### 2.设置分页查询最大值

> 还可以通过设置es参数来突破分页查询数据的限制

```JSON
// 查询es的各项设置
GET http://192.168.198.100:9200/tho/_settings
{
    "tho": {
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "3",
                "provided_name": "tho",
                "creation_date": "1638000752374",
                "number_of_replicas": "1",
                "uuid": "LPZF8yR-SaGfPVOCdXOtCg",
                "version": {
                    "created": "7150299"
                }
            }
        }
    }
}

// 设置查询限制
PUT http://192.168.198.100:9200/tho/_settings
// 参数
{
    "index.max_result_window": 16666
}
// 再次查询
{
    "tho": {
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "3",
                "provided_name": "tho",
                "max_result_window": "16666", // 分页显示数量
                "creation_date": "1638000752374",
                "number_of_replicas": "1",
                "uuid": "LPZF8yR-SaGfPVOCdXOtCg",
                "version": {
                    "created": "7150299"
                }
            }
        }
    }
}
```

## 4.2、滚动搜索scroll

>   一次查询1w+数据，往往会造成性能影响，因为数据量太大了，这个时候可以使用滚动搜索，也就是 `scroll` 。滚动搜索可以先查询出一些数据，然后再紧接着依次往下查询。在第一次查询的时候会有一个滚动id，相当于一个 `锚标记`，随后再次滚动搜索会需要上一次搜索的 锚标记，根据这个进行下一次搜索请求，每次搜索都是基于一个历史的数据快照，查询数据的期间，如果有数据的变更，那么和搜索是没有关系的，搜索的内容还是快照里的
>
> - scroll=1m, 相当于是一个session会话时间，搜索保持的上下文时间为1分钟

```JSON
// 滚动搜索 第一次搜索
POST http://192.168.198.100:9200/tho/_search?scroll=1m 
// 会话有效期1分钟,要在1分钟内进行下一次查询
// 参数
{
    "query": {
        "match_all": {}
    },
    "sort": ["_doc"],
    "size": 5
}
// 返回消息体
{
    "_scroll_id": "FGluY2x1ZGVfY29udGV4dF91dWlkDnF1ZXJ5VGhlbkZldGNoAxYxZWZTaUxORVRVUzBKOVo0bzRFVENBAAAAAAAAACsWNUV4Sk91My1TR1cyRWdibmZjQmhiQRYxZWZTaUxORVRVUzBKOVo0bzRFVENBAAAAAAAAACwWNUV4Sk91My1TR1cyRWdibmZjQmhiQRYxZWZTaUxORVRVUzBKOVo0bzRFVENBAAAAAAAAAC0WNUV4Sk91My1TR1cyRWdibmZjQmhiQQ==",
    "took": 76,
    "timed_out": false,
    "_shards": {
        "total": 3,
        "successful": 3,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 16,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "tho",
                "_type": "_doc",
                "_id": "1002",
                "_score": null,
                "_source": {
                    "id": 1002,
                    "age": 20,
                    "username": "bigFace",
                    "nickname": "飞翔的巨硬",
                    "money": "6.8",
                    "desc": "去了海外",
                    "sex": 1,
                    "birthday": "2000-06-26",
                    "face": "https://www.tho.com"
                },
                "sort": [
                    0
                ]
            },
            {
                "_index": "tho",
                "_type": "_doc",
                "_id": "1001",
                "_score": null,
                "_source": {
                    "id": 1001,
                    "age": 18,
                    "username": "imoocAmazing",
                    "nickname": "百度搜索",
                    "money": "88.8",
                    "desc": "学习java和前端",
                    "sex": 0,
                    "birthday": "1999-06-26",
                    "face": "https://www.tho.com"
                },
                "sort": [
                    0
                ]
            },
            {
                "_index": "tho",
                "_type": "_doc",
                "_id": "1003",
                "_score": null,
                "_source": {
                    "id": 1003,
                    "age": 22,
                    "username": "flyfish",
                    "nickname": "鱼",
                    "money": "6.8",
                    "desc": "池塘里，游泳",
                    "sex": 0,
                    "birthday": "2000-06-26",
                    "face": "https://www.tho.com1"
                },
                "sort": [
                    1
                ]
            },
            {
                "_index": "tho",
                "_type": "_doc",
                "_id": "1004",
                "_score": null,
                "_source": {
                    "id": 1004,
                    "age": 21,
                    "username": "gotoplay",
                    "nickname": "ns",
                    "money": "6.88",
                    "desc": "打游戏",
                    "sex": 0,
                    "birthday": "2000-06-26",
                    "face": "https://www.tho.com1"
                },
                "sort": [
                    1
                ]
            },
            {
                "_index": "tho",
                "_type": "_doc",
                "_id": "1007",
                "_score": null,
                "_source": {
                    "id": 1007,
                    "age": 21,
                    "username": "gotorun",
                    "nickname": "ns",
                    "money": "6.88",
                    "desc": "跑步ask法拉盛克里夫",
                    "sex": 0,
                    "birthday": "2000-06-26",
                    "face": "https://www.tho.com1"
                },
                "sort": [
                    2
                ]
            }
        ]
    }
}


// 滚动查询 之后的查询 (要在设置的会话有效期内进行后续查询)
POST http://192.168.198.100:9200/_search/scroll
// 参数
{
    "scroll_id": "FGluY2x1ZGVfY29udGV4dF91dWlkDnF1ZXJ5VGhlbkZldGNoAxYxZWZTaUxORVRVUzBKOVo0bzRFVENBAAAAAAAAAC4WNUV4Sk91My1TR1cyRWdibmZjQmhiQRYxZWZTaUxORVRVUzBKOVo0bzRFVENBAAAAAAAAADAWNUV4Sk91My1TR1cyRWdibmZjQmhiQRYxZWZTaUxORVRVUzBKOVo0bzRFVENBAAAAAAAAAC8WNUV4Sk91My1TR1cyRWdibmZjQmhiQQ==",
    "scroll": "1m"
}
```

## 4.3、批量查询

### 1.mget批量查询

> 获取多个查询
>
> 查询不存在的记录也会显示出来

```JSON
POST http://192.168.198.100:9200/tho/_doc/_mget
// 参数
{
    "ids": ["1001", "1003"]
}
// 返回值
{
    "docs": [
        {
            "_index": "tho",
            "_type": "_doc",
            "_id": "1001",
            "_version": 1,
            "_seq_no": 0,
            "_primary_term": 1,
            "found": true,
            "_source": {
                "id": 1001,
                "age": 18,
                "username": "imoocAmazing",
                "nickname": "百度",
                "money": "88.8",
                "desc": "学习java和前端，学到了很多",
                "sex": 0,
                "birthday": "1999-06-26",
                "face": "https://www.tho.com"
            }
        },
        {
            "_index": "tho",
            "_type": "_doc",
            "_id": "1003",
            "_version": 1,
            "_seq_no": 1,
            "_primary_term": 1,
            "found": true,
            "_source": {
                "id": 1003,
                "age": 22,
                "username": "flyfish",
                "nickname": "水",
                "money": "6.8",
                "desc": "池塘里，游泳",
                "sex": 0,
                "birthday": "2000-06-26",
                "face": "https://www.tho.com1"
            }
        }
    ]
}
```

### 2.批量增删改-bulk

>bulk操作和以往的普通格式有区别，不要格式化json，不然就不在同一行了，需要注意
>
>{ action: { metadata }}\n 
>
>{ request body }\n 
>
>{ action: { metadata }}\n 
>
>{ request body }\n
>
>- { action: { metadata }} 代表批量操作的类型，可以是新增、删除或修改
>-  \n 是每行结尾必须填写的一个规范，每一行包括最后一行都要写，用于es的解析
>-  { request body } 是请求body，增加和修改操作需要，删除操作则不需要



> 批量操作的类型
>
> action必须是以下选项之一
>
> - create：如果文档不存在，创建该文档，文档存在会报错，发生异常报错不会影响其它操作
> - index：创建一个新文档或者替换一个现有的文档
> - update：部分更新一个文档
> - delete：删除一个文档
>
> metadata中需要指定要操作文档的 _index, _type 和 _id ， _index  _type 也可以在url中指定



```JSON
POST http://192.168.198.100:9200/_bulk
// 参数
{"create": {"_index": "shop2", "_type": "_doc", "_id": "2001"}}
{"id": "2001", "nickname": "name2001"}
{"create": {"_index": "shop2", "_type": "_doc", "_id": "2002"}}
{"id": "2002", "nickname": "name2002"}
{"create": {"_index": "shop2", "_type": "_doc", "_id": "2003"}}
{"id": "2003", "nickname": "name2003"}
// 返回值
{
    "took": 72,
    "errors": false,
    "items": [
        {
            "create": {
                "_index": "tho",
                "_type": "_doc",
                "_id": "2001",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 7,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "tho",
                "_type": "_doc",
                "_id": "2002",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 9,
                "_primary_term": 2,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "tho",
                "_type": "_doc",
                "_id": "2003",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 8,
                "_primary_term": 2,
                "status": 201
            }
        }
    ]
}
```

> 参数里面的 " _index " 和 " _type " 都是一样的,可以提取出来
>
> 可以放到url里

```JSON
POST http://192.168.198.100:9200/tho/_doc/_bulk
// 参数
{"create": {"_id": "2005"}}
{"id": "2005", "nickname": "name2005"}
{"create": {"_id": "2006"}}
{"id": "2006", "nickname": "name2006"}
{"create": {"_id": "2007"}}
{"id": "2007", "nickname": "name2007"}
```

 

### 3.批量操作-index

> index 不存在的尽量进行导入
>
> 存在的记录会被直接覆盖

```JSON
POST http://192.168.198.100:9200/tho/_doc/_bulk
// 参数
{"index": {"_id": "2005"}}
{"id": "2005", "nickname": "name2005"}
{"index": {"_id": "2006"}}
{"id": "2006", "nickname": "index"}
{"index": {"_id": "2007"}}
{"id": "2007", "nickname": "index"}
{"index": {"_id": "2008"}}
{"id": "2008", "nickname": "index-new"}
// 返回值
{
    "took": 44,
    "errors": false,
    "items": [
        {
            "index": {
                "_index": "tho",
                "_type": "_doc",
                "_id": "2005",
                "_version": 3,
                "result": "updated",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 2,
                "_primary_term": 2,
                "status": 200
            }
        },
        {
            "index": {
                "_index": "tho",
                "_type": "_doc",
                "_id": "2006",
                "_version": 3,
                "result": "updated",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 11,
                "_primary_term": 2,
                "status": 200
            }
        },
        {
            "index": {
                "_index": "tho",
                "_type": "_doc",
                "_id": "2007",
                "_version": 3,
                "result": "updated", // 存在 进行更新
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 12,
                "_primary_term": 2,
                "status": 200
            }
        },
        {
            "index": {
                "_index": "tho",
                "_type": "_doc",
                "_id": "2008",
                "_version": 1,
                "result": "created", // 不存在,创建新的
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 13,
                "_primary_term": 2,
                "status": 201
            }
        }
    ]
}
```

# 五、ES集群

## 5.1、ES集群基本概念

> 主节点和 备份节点不能放在同一个服务器上

> 查看配置信息

```SHELL
more elasticsearch.yml | grep ^[^#]

[esuser@centos_7_100 config]$ more elasticsearch.yml | grep ^[^#]
cluster.name: tho-elasticsearch
node.name: es-node1
path.data: /usr/local/elasticsearch/elasticsearch-7.15.2/data
path.logs: /usr/local/elasticsearch/elasticsearch-7.15.2/logs
network.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
cluster.initial_master_nodes: ["es-node1"]

```

## 5.2、ES脑裂

> 如果发生网络中断或者服务器宕机，那么集群会有可能被划分为两个部分，各自有自己的master管理，这就是脑裂



> 解决方案
>
> - master主节点要经过多个master共同选举后才能成为新的主节点，就跟班级里的班长一样，并不是一个人可以决定的，需要班里多数人的决定
> - 解决实现原理：半数以上节点同意，节点方可成为新的master

- discovery.zen.minimum_master_nodes=(N/2)+1

	N为集群的中master节点的数量，也就是那些 node.master=true 设置的那些服务器节点总数。



> 新版本ES   ES7.X9
>
> `minimum_master_node ` 这个参数已经被移除了，这一块内容完全由es自身去管理，避免了脑裂问题，选举也会非常快

## 5.3、ES集群文档读写原理

# 六、ES整合SpringBoot

## 6.1、引入依赖

```XML
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
		<!--<version>2.1.5.RELEASE</version>-->
<version>2.2.2.RELEASE</version>
</dependency>
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
</dependency>
```

> 此依赖使用的es版本是 es6.4.3 要将服务器上的es降级为对应版本

> 项目配置文件 application.yml

```YAML
spring:
	# 需要datasource的配置,不然会报错
  datasource:
    type: com.zaxxer.hikari.HikariDataSource #数据源类型:HikariCP
    driver-class-name: com.mysql.jdbc.Driver #mysql驱动
    url: jdbc:mysql://localhost:3306/foodie-shop
    username: root
    password: 123456
  data:
    elasticsearch:
      # 集群
      # cluster-nodes: 192.168.198.100:9300,192.168.198.101:9300,192.168.198.102:9300
      # 单机
      cluster-nodes: 192.168.198.100:9300
      cluster-name: es6.4.3

```



> 可能会报错

```JAVA
ERROR SpringApplication Application run failed
 org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'elasticsearchClient' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.elasticsearch.client.transport.TransportClient]: Factory method 'elasticsearchClient' threw exception; nested exception is java.lang.IllegalStateException: availableProcessors is already set to [8], rejecting [8]
```

> 解决  Netty issue fix  放到application.class 的同级目录下

```java
package com.tho;

import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;

/**
 * @Author tho
 * @Date 2021/11/28/19:23
 * @ProjectName foodstuffMall
 * @ClassName: ESConfig
 * @Description: 解决 netty issue fix
 */
@Configuration
public class ESConfig {
    /**
     * 解决netty引起的issue
     */
    @PostConstruct
    void init() {
        System.setProperty("es.set.netty.runtime.available.processors", "false");
    }
}
```

## 6.2、整合后基本使用

> pojo类

```java
package com.tho.es.pojo;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;

/**
 * @Author tho
 * @Date 2021/11/28/19:32
 * @ProjectName foodstuffMall
 * @ClassName: Student
 * @Description: ES测试 实体类 对应 es中的文档
 */
@Document(indexName = "stu", type = "_doc")
public class Student {
    @Id // 设置之后,数据的id 同 es中的id一起设置为设定值
    private Long stuId;
    @Field(store = true)
    private String name;
    @Field(store = true)
    private Integer age;

    public Long getStuId() {
        return stuId;
    }

    public void setStuId(Long stuId) {
        this.stuId = stuId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

> 测试类

```java
package com.test;

import com.tho.Application;
import com.tho.es.pojo.Student;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;
import org.springframework.data.elasticsearch.core.query.IndexQuery;
import org.springframework.data.elasticsearch.core.query.IndexQueryBuilder;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * @Author tho
 * @Date 2021/11/28/19:29
 * @ProjectName foodstuffMall
 * @ClassName: ESTest
 * @Description: ES 测试
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ESTest {
    @Autowired
    private ElasticsearchTemplate esTemplate;

    @Test
    public void createIndexStudent() {
        Student student = new Student();
        student.setStuId(1001L);
        student.setName("tho");
        student.setAge(18);
        IndexQuery query = new IndexQueryBuilder().withObject(student).build();
        esTemplate.index(query);
    }
}
```

## 6.3、索引增删

>- 不建议使用ElasticsearchTemplate 对索引进行管理(创建索引,更新映射,删除索引)
>
>- 索引就像是数据库或数据库中的表，平时是不会通过java代码频繁地去创建修改删除数据库中的表的
>
>- 只会针对数据做CRUD操作
>- 在es中也是同理,尽量使用 ElasticsearchTemplate 对文档数据进行CRUD操作



>ElasticsearchTemplate缺点
>
>1. 属性(FieldType)类型不灵活
>2. 主分片于副分片无法设置



### 1.索引创建

```java
@Test
public void createIndexStudent() {
    Student student = new Student();
    student.setStuId(1001L);
    student.setName("tho");
    student.setAge(18);
    IndexQuery query = new IndexQueryBuilder().withObject(student).build();
    esTemplate.index(query); 
}
```

### 2.索引删除

```java
@ Test
public void deleteIndexStu() {
    esTemplate.deleteIndex(Student.class);
}
```

## 6.4、文档数据CRUD

### 1.新增文档数据 

> 同创建索引操作

```java
@Test
public void createIndexStudent() {
    Student student = new Student();
    student.setStuId(1001L);
    student.setName("tho");
    student.setAge(18);
    IndexQuery query = new IndexQueryBuilder().withObject(student).build();
    esTemplate.index(query); 
}
```

### 2、更新文档数据

```java
public void updateStudentDoc() {
    Map<String, Object> sourceMap = new HashMap<>();
    sourceMap.put("sign", "xxxxxx");
    sourceMap.put("money", 666.6f);
    sourceMap.put("age", 3);
    IndexRequest request = new IndexRequest();
    request.source(sourceMap);
    UpdateQuery updateQuery = new UpdateQueryBuilder().withClass(Student.class)
            .withId("1002")
            .withIndexRequest(request)
            .build();
    // 相当于sql语句
    // update student set sign='xxxxxx', age=3, money=666.6f where id = '1002'
    esTemplate.update(updateQuery);

}
```

### 3.查询文档数据

```java
@Test
public void getStudentDoc() {

    GetQuery getQuery = new GetQuery();
    getQuery.setId("1002");
    Student student = esTemplate.queryForObject(getQuery, Student.class);
    System.out.println(student);
}
```

### 4.删除文档数据

```java
@Test
public void deleteStudentDoc() {
    String delete = esTemplate.delete(Student.class, "1002");
}
```

## 6.5、实现分页搜索

```java
/**
* @Author tho
* @Date 2021/11/28 20:18
* @param
* @Return void
* @Description: 分页
*/
@Test
public void queryForPage() {

    // 分页设置
    Pageable pageable = PageRequest.of(0, 10);
    NativeSearchQuery query = new NativeSearchQueryBuilder()
            .withQuery(QueryBuilders.matchQuery("description", "save man"))
            .withPageable(pageable)
            .build();
    AggregatedPage<Student> pagedStudentList = esTemplate.queryForPage(query, Student.class);
    int totalPages = pagedStudentList.getTotalPages();
    System.out.println("检索后总分页数为" + totalPages);
    List<Student> studentList = pagedStudentList.getContent();
    for (Student student : studentList) {
        System.out.println(student);
    }
}
```

## 6.6、高亮

```java
/**
    * @Author tho
    * @Date 2021/11/28 20:22
    * @param
    * @Return void
    * @Description: 高亮显示
    */
    @Test
    public void highLightStudentDoc() {

        String preTag = "<font color='red'>";
        String postTag = "</fond>";
        // 分页设置
        Pageable pageable = PageRequest.of(0, 10);
        NativeSearchQuery query = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchQuery("description", "save man"))
                .withHighlightFields(new HighlightBuilder.Field("description")
                .preTags(preTag)
                .postTags(postTag))
                .withPageable(pageable)
                .build();
        AggregatedPage<Student> pagedStudentList = esTemplate.queryForPage(query, Student.class, new SearchResultMapper() {
            
            @Override
            public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> clazz, Pageable pageable) {
                // 需要加入自己的映射 直接获取的数据不包括高亮的内容
                SearchHits hits = response.getHits();
                List<Student> studentListHighLight = new ArrayList<>();
                for (SearchHit hit : hits) {
                    HighlightField highLightField = hit.getHighlightFields().get("description");
                    String description  = highLightField.getFragments()[0].toString();

                    Object stuId = hit.getSourceAsMap().get("id");
                    String name =(String) hit.getSourceAsMap().get("name");
                    Integer age =(Integer) hit.getSourceAsMap().get("age");
                    String sign =(String) hit.getSourceAsMap().get("sign");
                    Object money = hit.getSourceAsMap().get("money");

                    Student highLightStudent = new Student();
                    highLightStudent.setDescription(description);
                    highLightStudent.setAge(age);
                    highLightStudent.setSign(sign);
                    highLightStudent.setMoney(Float.valueOf(money.toString()));
                    highLightStudent.setStuId(Long.valueOf(stuId.toString()));
                    highLightStudent.setName(name);

                    studentListHighLight.add(highLightStudent);
                }
                if (studentListHighLight.size() > 0) {
                    return new AggregatedPageImpl<>( (List<T>) studentListHighLight);
                }
                return null;
            }
        });
        int totalPages = pagedStudentList.getTotalPages();
        System.out.println("检索后总分页数为" + totalPages);
        List<Student> studentList = pagedStudentList.getContent();
        for (Student student : studentList) {
            System.out.println(student);
        }
    }
```

## 6.7、排序

```java
/**
* @Author tho
* @Date 2021/11/28 20:44
* @param
* @Return void
* @Description: 排序
*/
@Test
public void sortStudentDoc() {

    String preTag = "<font color='red'>";
    String postTag = "</fond>";
    // 分页设置
    Pageable pageable = PageRequest.of(0, 10);

    // 排序
    SortBuilder sortBuilder = new FieldSortBuilder("money")
            .order(SortOrder.ASC);

    NativeSearchQuery query = new NativeSearchQueryBuilder()
            .withQuery(QueryBuilders.matchQuery("description", "save man"))
            .withHighlightFields(new HighlightBuilder.Field("description")
                    .preTags(preTag)
                    .postTags(postTag))
            .withPageable(pageable)
            .withSort(sortBuilder) //排序
            .build();
    AggregatedPage<Student> pagedStudentList = esTemplate.queryForPage(query, Student.class, new SearchResultMapper() {

        @Override
        public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> clazz, Pageable pageable) {
            // 需要加入自己的映射 直接获取的数据不包括高亮的内容
            SearchHits hits = response.getHits();
            List<Student> studentListHighLight = new ArrayList<>();
            for (SearchHit hit : hits) {
                HighlightField highLightField = hit.getHighlightFields().get("description");
                String description  = highLightField.getFragments()[0].toString();

                Object stuId = hit.getSourceAsMap().get("id");
                String name =(String) hit.getSourceAsMap().get("name");
                Integer age =(Integer) hit.getSourceAsMap().get("age");
                String sign =(String) hit.getSourceAsMap().get("sign");
                Object money = hit.getSourceAsMap().get("money");

              	// 要加判断 因为字段名称问题,可能从es中搜索不到记录
              	// 有的字段可能会是空指针
             		// Long.valueOf(stuId.toString() 会报空指针异常
              
                Student highLightStudent = new Student();
                highLightStudent.setDescription(description);
                highLightStudent.setAge(age);
                highLightStudent.setSign(sign);
                highLightStudent.setMoney(Float.valueOf(money.toString()));
                highLightStudent.setStuId(Long.valueOf(stuId.toString()));
                highLightStudent.setName(name);

                studentListHighLight.add(highLightStudent);
            }
            if (studentListHighLight.size() > 0) {
                return new AggregatedPageImpl<>( (List<T>) studentListHighLight);
            }
            return null;
        }
    });
    int totalPages = pagedStudentList.getTotalPages();
    System.out.println("检索后总分页数为" + totalPages);
    List<Student> studentList = pagedStudentList.getContent();
    for (Student student : studentList) {
        System.out.println(student);
    }
}
```

# 七、Logstash

> mysql控制远程连接
>
> mysql数据库中的user表
>
> user -> host 对应关系
>
> host为 % 都可以连接
>
> host 有localhost 只有localhost可以连接



## 7.1、数据同步

> Logstash是elastic技术栈中的一个技术。它是一个数据采集引擎，可以从数据库采集数据到es中。我们可以通过设置自增id主键或者时间来控制数据的自动同步 时间就是用于给logstash进行识别的
>
> - id：假设现在有1000条数据，Logstatsh识别后会进行一次同步，同步完会记录这个id为1000，以后数据库新增数据，那么id会一直累加 h会有定时任务，发现有id大于1000了，则增量加入到es中 
> - 时间：同理，一开始同步1000条数据，每条数据都有一个字段，为time，初次同步完毕后，记录这个time，下次同步的时候进行时间比 超过这个时间的，那么就可以做同步，这里可以同步新增数据，或者修改元数据，因为同一条数据的时间更改会被识别，而id则不会。
>
> - 预先创建索引

## 7.2、环境配置

> Logstash 要与 es 版本一致
>
> - 需要本机配置jdk 环境
>
> - 插件 ： logstash-input-jdbc 
>
> 	​	本插件用于同步，es6.x起自带，这个是集成在了 logstash中的。所以直接配置同步数据库的配置文件即可
>
> - 需要logstash.tar.gz 包
>
> - 需要mysql的驱动

1. 解压logstash 放到 `/user/local/logstash` 下

2. 在 `/usr/local/logstash/logstash-6.4.3` 下创建 `sync` 文件夹

3. 在 sync 文件夹下创建 logstash配置文件 `logstash-db-sync.conf`,  查询mysql数据库的sql脚本 `food-items.sql`, mysql驱动 `mysql-connector-java-5.1.41.jar`

	>logstash-db-sync.conf

	```java
	input {
		jdbc {
			# 设置mysql/MariaDB 数据库Url 以及数据库名称
			jdbc_connection_string => "jdbc:mysql://192.168.31.207:3306/foodie-shop"
			# 用户名和密码
			jdbc_user => "root"
			jdbc_password => "123456"
			# 数据库驱动所在位置,可以是绝对路径或者相对路径
			jdbc_driver_library => "/usr/local/logstash/logstash-6.4.3/sync/mysql-connector-java-5.1.41.jar"
			# 驱动类名
			jdbc_driver_class => "com.mysql.jdbc.Driver"
			# 开启分页
			jdbc_paging_enabled => "true"
			# 分页每页数量,可以自定义
			jdbc_page_size => "10000"
			# 执行sql文件路径
			statement_filepath => "/usr/local/logstash/logstash-6.4.3/sync/food-items.sql"
			# 设置定时任务间隔  含义: 分，时，天，月，年， 全部为* 默认含义每分钟跑一次
			schedule => "* * * * *"
			# 索引类型
			type => "_doc"
			# 是否开启记录上次追踪的结果，也就是上次更新的时间，这个会记录到 last_run_metadata_path 的文件
			use_column_value => true
			# 记录上一次追踪的结果值
			last_run_metadata_path =>  "/usr/local/logstash/logstash-6.4.3/sync/track_time"
			# 如果 use_column_value 为 true ， 配置本参数 ， 追踪的column名 ， 可以是自增id 或者时间
			tracking_column => "update_time"
			# tracking_column 对应字段的类型
			tracking_column_type => "timestamp"
			# 是否清除 last_run_metadata_path 的记录, true则每次都从头开始查询所有记录
			clean_run => false
			# 数据库字段名称大写转小写
			lowercase_column_names => false
		}
	}
	output {
		elasticsearch {
			# es地址
			hosts => ["192.168.198.100:9200"]
			# 同步的索引名
			index => "food-items"
			# 设置 _docID 和 数据相同
			document_id => "%{itemId}" 
		}
		
		# 日志输出
		stdout {
			codec => json_lines
		}
	}
	```

	>food-items.sql

	```sql
	SELECT
	            i.id as itemId,
	            i.item_name as itemName,
	            i.sell_counts as sellCounts,
	            ii.url as imgUrl,
	            tempSpec.priceDiscount as price,
							i.updated_time as updated_time
	        FROM
	            items i
	        left JOIN
	            items_img ii
	        on
	            i.id = ii.item_id
	        left join
	            (
	                SELECT
	                    item_id,MIN(price_discount) as priceDiscount
	                FROM
	                    items_spec
	                GROUP BY
	                    item_id
	            )tempSpec
	        on
	            i.id = tempSpec.item_id
	
	        WHERE
	            ii.is_main = 1
							and 
							i.updated_time >= :sql_last_value
	```

4. 进入 bin 目录 执行命令 `./logstash -f /usr/local/logstash-6.4.3/sync/logstash-db-sync.conf`

5. mysql 主键字段  要与  elasticSearch 的 _id 相一致

6. logstash 的 timestamp 要与 mysql数据库中的 `updated_time` 相匹配

7. ==远程连接 mysql 失败 `mysql.user`  表中 root 用户 对应的的 host 地址是否包含要读取mysql数据库的ip== 

## 7.3、logstash数据更新

> mysql数据库中的数据有变化,可以通过 改变 update_time 字段来让logstash自动更新ES中的数据,也可以通过改变 update_time 字段来进行逻辑删除

## 7.4、logstash中文分词模板

```json
GET http://192.168.198.100:9200/_template/logstash、
// 将返回的内容进行修改,修改后如下所示
```

> 建立一个模板文件  `logstash-ik.json`

```JSON
{
    
        "order": 0,
        "version": 1,
        "index_patterns": [
            "*"
        ],
        "settings": {
            "index": {
                "refresh_interval": "5s"
            }
        },
        "mappings": {
            "_default_": {
                "dynamic_templates": [
                    {
                        "message_field": {
                            "path_match": "message",
                            "match_mapping_type": "string",
                            "mapping": {
                                "type": "text",
                                "norms": false
                            }
                        }
                    },
                    {
                        "string_fields": {
                            "match": "*",
                            "match_mapping_type": "string",
                            "mapping": {
                                "type": "text",
                                "norms": false,
                              // 这里修改
                              // String 会匹配到 text 进行 ik_max_word 分词
                              "analyzer": "ik_max_word",
                                "fields": {
                                    "keyword": {
                                        "type": "keyword",
                                        "ignore_above": 256
                                    }
                                }
                            }
                        }
                    }
                ],
                "properties": {
                    "@timestamp": {
                        "type": "date"
                    },
                    "@version": {
                        "type": "keyword"
                    },
                    "geoip": {
                        "dynamic": true,
                        "properties": {
                            "ip": {
                                "type": "ip"
                            },
                            "location": {
                                "type": "geo_point"
                            },
                            "latitude": {
                                "type": "half_float"
                            },
                            "longitude": {
                                "type": "half_float"
                            }
                        }
                    }
                }
            }
        },
        "aliases": {}
    
}
```

2. 将 json 文件 放入 `/usr/local/logstash/logstash-6.4.3/sync` 目录下

3. 修改  `logstash-db-sync.conf` 配置文件

	```java
	output {
		elasticsearch {
			# es地址
			hosts => ["192.168.198.100:9200"]
			# 同步的索引名
			index => "food-items"
			# 设置 _docID 和 数据相同
			document_id => "%{itemId}" 
			
			# 定义模板名称
			template_name => "myik"
			# 模板所在位置
			template => "/usr/local/logstash/logstash-6.4.3/sync/logstash-ik.json"
			# 重写模板
			template_overwrite => true
			# 默认为true,false关闭logstash自动管理模板功能，如果自定义模板,则设置为false
			manage_template => false
		}
		
		# 日志输出
		stdout {
			codec => json_lines
		}
	}
	```

4. 之后可以进行logstash 与 mysql 的同步

	`./logstash -f /usr/local/logstash/logstash-6.4.3/sync/logstash-db-sync.conf`

## 7.5、logstash-中文分词器配置失效

>分词配置不成功，analyzer不显示

1.先将logstash的配置文件 `logstash-db-sync.conf` 中的 

```json
# 默认为true,false关闭logstash自动管理模板功能，如果自定义模板,则设置为false
		manage_template => true // 设置为true
```

2.删除原先索引，新建相同名字索引，启动logstash同步数据的脚本

3.使用postman查看模板是否配置成功

```json
GET http://192.168.198.100:9200/_template/myik
// 有返回值,说明配置模板成功
```

4.再将logstash配置文件 中的 manage_template 设置为false

```json
# 默认为true,false关闭logstash自动管理模板功能，如果自定义模板,则设置为false
		manage_template => false // 设置为false
```

5.再次运行logstash 同步数据脚本 之后es索引应该能显示analyzer分词设置

# 八、ES整合项目

## 8.1、初始化web环境

> 配置tomcat端口号
>
> 写一个简单的用于测试的controller
>
> es模块依赖于 common 模块 pom.xml 中引入其依赖 
>
> 配置一个maven脚本 跳过test    `install -Dmaven.test.skip=true`
>
> - 鼠标右键install -> 选择 create * -> command line 填入 `install -Dmaven.test.skip=true`
> - 之后会在下面自动生成Run Configurations , 其中会有刚才写的 maven脚本
>
> 导入log4j日志依赖
>
> ```xml
>  <!--log4j 日志依赖-->
>         <dependency>
>             <groupId>org.apache.logging.log4j</groupId>
>             <artifactId>log4j-core</artifactId>
>             <version>2.11.1</version>
>         </dependency>
> ```
>
> 拷贝log4j.properties 配置文件到search 模块下 修改配置文件中的路径参数

## 8.2、高亮

使用<em> 标签 可以在前端自定义高亮样式