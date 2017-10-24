---
title: Elasticsearch 基础入门
date: 2017-09-27 10:58:59
tags: [Elasticsearch,搜索]
categories: 后端
---
![](http://ow97db1io.bkt.clouddn.com/elasticSearch-15.jpg)

## 一、什么是 ElasticSearch
ElasticSearch是一个基于 Lucene 的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于 RESTful web 接口。Elasticsearch 是用 Java 开发的，并作为 Apache 许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

<!-- more -->

### 1.1 基础概念
索引：含有相同属性的文档集合

类型：索引可以定义一个或多个类型，文档必须属于一个类型

文档：可以被索引的基础数据单位

分片：每个索引都有多个分片，每个分片都是 Lucene 索引

备份：拷贝一份分片就完成分片的备份


**形象比喻：**

百货大楼里有各式各样的商品，例如书籍、笔、水果等。书籍可以根据内容划分成不同种类，如科技类、教育类、悬疑推理等。悬疑推理类的小说中比较有名气的有《福尔摩斯探案集》、《白夜行》等。

百货大楼 --> ElasticSearch 数据库

书籍     --> 索引

悬疑推理 --> 类型

白夜行   --> 文档

### 1.2 应用场景
* 海量数据分析引擎
* 站内搜索引擎
* 数据仓库

## 二、安装和配置
> 本次测试使用一台 ip 为 192.168.2.41 的虚拟机（Centos7），建议使用 7.x 版本，笔者之前使用 6.x 启动服务时报出各种错误

### 2.1 依赖环境

JDK 和 NodeJS

### 2.2 下载
登陆 [elasticSearch](http://elastic.co/) 官网下载文件。

### 2.3 安装

```
tar -zxvf elasticsearch-5.6.1.tar.gz -C /usr

cd elasticsearch-5.6.1

```

elasticsearch 文件目录如下图：

![](http://ow97db1io.bkt.clouddn.com/elasticSearch-1.jpg)


### 2.4 启动
***踩坑提醒 1：***

因为 Elasticsearch 可以执行脚本文件，为了安全性，默认不允许通过 root 用户启动服务。我们需要新创建用户名和用户组启动服务

```
#增加 es 组
groupadd es    

#增加 es 用户并附加到 es 组
useradd es -g es -p es       

#给目录权限
chown -R es:es elasticsearch-5.6.1    

#使用es用户
su es          
```


***踩坑提醒 2：***

默认情况下，Elasticsearch 只允许本机访问，如果需要远程访问，需要修改其配置文件 


```
vim config/elasticsearch.yml 

# 去掉 network.host 前边的注释，将它的值改成0.0.0.0
network.host: 0.0.0.0


```

***踩坑提醒 3：***

在启动过程中，Centos 环境下可能还会报错，具体解决方案请参照文章末尾提供的资料

启动服务

```
bin/elasticsearch
```


通过浏览器访问 http://192.168.2.41:9200 ，当出现如下内容说明启动成功:


```
{
  "name" : "OwUwJe-",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "vanzxnpaRumdRKiYic3f5A",
  "version" : {
    "number" : "5.6.1",
    "build_hash" : "667b497",
    "build_date" : "2017-09-14T19:22:05.189Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```


### 2.5 安装插件
访问 http://192.168.2.41:9200 查看内容显示效果不友好，因此，我们需要安装一个名为 elasticsearch-head 的插件，让内容显示效果比较舒适。

登陆 [GitHub](https://github.com) 网站，搜索 mobz/elasticsearch-head ，将其下载到本地。


```
wget https://github.com/mobz/elasticsearch-head/archive/master.zip

unzip master.zip

cd elasticsearch-head-master

npm install

npm run start
```

通过上述命令的操作，我们已经安装好 elasticsearch-head 插件。通过浏览器访问 http://192.168.2.41:9100，如下图:

![](http://ow97db1io.bkt.clouddn.com/elasticSearch-4.jpg)


图中我们发现 elasticsearch-head 插件和 Elasticsearch 服务并没有建立连接，所以我们还需要修改 Elasticsearch 的配置文件：


```
cd elasticsearch-5.6.1

vim config/elasticsearch.yml

# 在文件末尾添加 2 段配置

http.cors.enabled: true
http.cors.allow-origin: "*"

```

保存文件后，分别起来 2 个程序：

```

cd elasticsearch-5.6.1

# 后台启动 elasticSearch 服务
bin/elasticsearch -d

cd elasticsearch-head-master

npm run start
```

通过浏览器访问 http://192.168.2.41:9100，如下图:

![](http://ow97db1io.bkt.clouddn.com/elasticSearch-5.jpg)

通过插件创建索引

![](http://ow97db1io.bkt.clouddn.com/elasticSearch-13.jpg)

查看索引基本情况

![](http://ow97db1io.bkt.clouddn.com/elasticSearch-14.jpg)

**该插件能直接对 Elasticsearch 的数据进行增删改查，因此存在安全性的问题。建议生产环境下不要使用该插件！**

## 三、使用

Elasticsearch 支持 RESTFUL 风格 API，其 API 基本格式如下：


```
http://<ip>:<port>/<索引>/<类型>/<文档id>
```

### 3.1 创建/删除索引
为了方便测试，我们使用 POSTMAN 工具进行接口的请求。

创建一个非结构化的索引，需要使用 **PUT** 请求。例如创建一个名为 book 的索引。


执行：
```
[PUT] http://192.168.2.41:9200/book
```

返回结果：

```
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "book"
}
```

创建一个结构化的索引，如下图：

![](http://ow97db1io.bkt.clouddn.com/elasticSearch-07.jpg)

删除一个索引，需要使用 **DELETE** 请求。

执行：

```
[DELETE] http://192.168.2.41:9200/book
```

返回结果：

```
{
    "acknowledged": true
}
```


### 3.2 插入数据

插入指定 ID 的数据，需要使用 **PUT** 请求。如下图：

![](http://ow97db1io.bkt.clouddn.com/elasticSearch-9.jpg)

插入不指定 ID 的数据，需要使用 **POST** 请求。如下图：

![](http://ow97db1io.bkt.clouddn.com/elasticSearch-10.jpg)

### 3.3 修改数据

修改数据，需要使用 **POST** 请求，且 URL 需要添加 _update 

执行：

```
[POST] http://192.168.2.41:9200/fruit/apple/1/_update
```

请求参数（修改颜色）：

```
{
    "doc": {
        "color": "black"
    }
}
```

返回结果：

```
{
    "_index": "fruit",
    "_type": "apple",
    "_id": "1",
    "_version": 7,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    }
}
```


### 3.4 删除数据

修改数据，需要使用 **DELETE** 请求。

执行：

```
[DELETE] http://192.168.2.41:9200/fruit/apple/1
```

返回结果：

```
{
    "found": true,
    "_index": "fruit",
    "_type": "apple",
    "_id": "1",
    "_version": 8,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    }
}
```

### 3.5 查找数据

查询指定ID的数据，需要使用 **GET** 请求。

执行：

```
[GET] http://192.168.2.41:9200/fruit/apple/AV69_4DDdZbC-YBdV-U3
```
返回结果：

```
{
    "_index": "fruit",
    "_type": "apple",
    "_id": "AV69_4DDdZbC-YBdV-U3",
    "_version": 1,
    "found": true,
    "_source": {
        "color": "green",
        "weight": 1,
        "createTime": "2017-09-26 19:05:26"
    }
}
```

条件查询，需要使用 **POST** 请求。

执行：

```
[POST] http://192.168.2.41:9200/fruit/apple/_search
```

请求参数（查找 color = "green"）：

```
{
    "query": {
        "match":{
            "color": "green"
        }
    }
}
```

返回结果：

```
{
    "took": 8,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 0.2876821,
        "hits": [
            {
                "_index": "fruit",
                "_type": "apple",
                "_id": "AV69_4DDdZbC-YBdV-U3",
                "_score": 0.2876821,
                "_source": {
                    "color": "green",
                    "weight": 1,
                    "createTime": "2017-09-26 19:05:26"
                }
            }
        ]
    }
}
```


## 参考资料
* <http://blog.csdn.net/qq942477618/article/details/53414983>    解决问题方案
* <http://blog.csdn.net/laotoumo/article/details/53890279>          解决问题方案
* <http://blog.csdn.net/jiankunking/article/details/65448030>       解决问题方案
* <http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html>
* <http://www.cnblogs.com/ghj1976/p/5293250.html>
