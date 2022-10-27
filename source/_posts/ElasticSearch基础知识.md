---
title: Elasticsearch基础入门知识
date: 2022-10-24 22:09:02
tags: 
- Elasticsearch
- Kibana
- Elasticsearch-head
index_img: 
banner_img: /img/article8.webp
categories:
- Elasticsearch
comment: waline
---

# ES快速入门

​      ES作为一个索引及搜索服务，对外提供丰富的REST接口，快速入门部分的实例使用kibana来测试，目的是对ES的使用方法及流程有个初步的认识。

## 1. index管理

### 1.1. 创建index

索引库。包含若干相似结构的 Document 数据，相当于数据库的database。

语法：`PUT /index_name`

如：

```json
PUT /java
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  }
}
```

**number_of_shards** -  表示一个索引库将拆分成多片分别存储不同的结点，提高了ES的处理能力

**number_of_replicas** - 是为每个 primary shard分配的replica shard数，提高了ES的可用性，<font color=red>如果只有一台机器，设置为0</font>

### 1.2. 修改index

注意：索引一旦创建，primary shard 数量不可变化，可以改变replica shard 数量。

语法：`PUT /index_name/_settings`

如：

```json
PUT /java/_settings
{
  "number_of_replicas" : 1
}
```

ES 中对 shard 的分布是有要求的，有其内置的特殊算法：

​       Replica shard 会保证不和他的那个 primary shard 分配在同一个节点上；如过只有一个节点，则此案例执行后索引的状态一定是yellow。

### 1.3. 删除index

```
DELETE /java[, other_index]
```

## 2. mapping管理

映射，创建映射就是向索引库中创建field（类型、是否索引、是否存储等特性）的过程，下边是document和field与关系数据库的概念的类比：

| elasticsearch  | 关系数据库       |
| -------------- | ---------------- |
| index(索引库)  | database(数据库) |
| type(类型)     | table(表)        |
| document(文档) | row(记录)        |
| field(域)      | column(字段)     |

注意：6.0之前的版本有type（类型）概念，type相当于关系数据库的表，ES6.x 版本之后，type概念被弱化ES官方将在ES7.0版本中彻底删除type。

### 2.1   创建mapping

语法：`POST /index_name/type_name/_mapping`

如：

```json
POST /java/course/_mapping
{
  "properties": {
     "name": {
        "type": "text"
     },
     "description": {
        "type": "text"
     },
     "studymodel": {
        "type": "keyword"
     }
  }
}
```

### 2.2.查询mapping

查询所有索引的映射：

```json
GET /java/course/_mapping
```

### 2.3.更新mapping

映射创建成功可以添加新字段，已有字段不允许更新。

### 2.4.删除mapping

通过删除索引来删除映射。		

## 3. document管理

### 3.1. 创建document

ES中的文档相当于MySQL数据库表中的记录。

#### 3.1.1. POST语法

此操作为 ES 自动生成 id 的新增 Document 方式。

语法：`POST /index_name/type_name/id`

如：

```json
POST /java/course/1
{
  "name":"python从入门到放弃",
  "description":"人生苦短，我用Python",
  "studymodel":"201002"
}

POST /java/course
{
  "name":".net从入门到放弃",
  "description":".net程序员谁都不服",
  "studymodel":"201003"
}
```

#### 3.1.2. PUT语法

此操作为手工指定 id 的 Document 新增方式。

语法：PUT/index_name/type_name/id{field_name:field_value}

如：

```json
PUT /java/course/2
{
  "name":"php从入门到放弃",
  "description":"php是世界上最好的语言",
  "studymodel":"201001"
}
```

结果：

```json
{
  "_index": "test_index", 新增的 document 在什么 index 中，
  "_type": "my_type", 新增的 document 在 index 中的哪一个 type 中。
  "_id": "1", 指定的 id 是多少
  "_version": 1, document 的版本是多少，版本从 1 开始递增，每次写操作都会+1
  "result": "created", 本次操作的结果，created 创建，updated 修改，deleted 删除
  "_shards": { 分片信息
      "total": 2, 分片数量只提示 primary shard
      "successful": 1, 数据 document 一定只存放在 index 中的某一个 primary shard 中
      "failed": 0
  },
  "_seq_no": 0, 
  "_primary_term": 1
}
```

通过head查询数据

### 3.2. 查询document

语法：

​       `GET /index_name/type_name/id` 

或

​       `GET /index_name/type_name/_search?q=field_name:field_value`

如：根据课程id查询文档

```
GET /java/course/1
```

如：查询所有记录

```
GET /java/course/_search
```

如：查询名称中包括php 关键字的的记录

```
GET /java/course/_search?q=name:门
```

结果：

```json
{
  "took": 1, # 执行的时长。单位毫秒
  "timed_out": false, # 是否超时
  "_shards": { # shard 相关数据
    "total": 1, # 总计多少个 shard
    "successful": 1, # 成功返回结果的 shard 数量
    "skipped": 0,
    "failed": 0
  },
  "hits": { # 搜索结果相关数据
    "total": 3, # 总计多少数据，符合搜索条件的数据数量
    "max_score": 1, # 最大相关度分数，和搜索条件的匹配度
    "hits": [# 具体的搜索结果
      {
        "_index": "java",# 索引名称
        "_type": "course", # 类型名称
        "_id": "1",# id 值
        "_score": 1, # 匹配度分数，本条数据匹配度分数
        "_source": { # 具体的数据内容
          "name": "php从入门到放弃",
          "description": "php是世界上最好的语言",
          "studymodel": "201001"
        }, {
			"_index": "java",
			"_type": "course",
			"_id": "2",
			"_score": 0.13353139,
			"_source": {
				"name": "php从入门到放弃",
				"description": "php是世界上最好的语言",
				"studymodel": "201001"
			}
		}, {
			"_index": "java",
			"_type": "course",
			"_id": "6ljFCnIBp91f7uS8FkjS",
			"_score": 0.13353139,
			"_source": {
				"name": ".net从入门到放弃",
				"description": ".net程序员谁都不服",
				"studymodel": "201003"
			}
		}
	 ]
  }
}
```

### 3.3. 删除Document

​       ES 中执行删除操作时，ES先标记Document为deleted状态，而不是直接物理删除。当ES 存储空间不足或工作空闲时，才会执行物理删除操作，标记为deleted状态的数据不会被查询搜索到（ES 中删除 index ，也是标记。后续才会执行物理删除。所有的标记动作都是为了NRT（近实时）实现）

语法：`DELETE /index_name/type_name/id`

如：

```json
DELETE /java/course/3
```

结果：

```json
{
  "_index": "java",
  "_type": "course",
  "_id": "2",
  "_version": 2,
  "result": "deleted",
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 3,
  "_primary_term": 1
}
```

## 4. ES读写过程

### 4.1. documnet routing（数据路由）

当客户端创建document的时候，es需要确定这个document放在该index哪个shard上，这个过程就是document routing。

路由过程：

　　　　路由算法：shard = hash(id) %number_of_primary_shards

　　　　id：document的_id，可能是手动指定，也可能是自动生成，决定一个document在哪个shard上 

　　　　number_of_primary_shards*：*主分片數量。

### 4.4. 为什么primary shard数量不可变？

​       原因：假如我们的集群在初始化的时候有5个primary shard，我们往里边加入一个document    id=5，假如hash(5)=23,这时该document 将被加入 (shard=23%5=3)P3这个分片上。如果随后我们给es集群添加一个primary shard ，此时就有6个primary shard，当我们GET id=5 ，这条数据的时候，es会计算该请求的路由信息找到存储他的 primary shard（shard=23%6=5） ，根据计算结果定位到P5分片上。而我们的数据在P3上。所以es集群无法添加primary shard，但是可以扩展replicas shard。

## 5. luke查看ES的逻辑结构

[luke](https://github.com/DmitryKey/luke/releases)使用[指南](https://blog.csdn.net/fly910905/article/details/81190382)

# IK分词器

## 1. 测试分词器

在添加文档时会进行分词，索引中存放的就是一个一个的词（term），当你去搜索时就是拿关键字去匹配词，最终找到词关联的文档。

测试当前索引库使用的分词器：

```json
POST /_analyze
{
  "text":"测试分词器，后边是测试内容：spring cloud实战"
}
```

结果如下：

```json
{
  "tokens": [
    {
      "token": "测",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<IDEOGRAPHIC>",
      "position": 0
    },
    {
      "token": "试",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<IDEOGRAPHIC>",
      "position": 1
    },
    {
      "token": "分",
      "start_offset": 2,
      "end_offset": 3,
      "type": "<IDEOGRAPHIC>",
      "position": 2
    },
    {
      "token": "词",
      "start_offset": 3,
      "end_offset": 4,
      "type": "<IDEOGRAPHIC>",
      "position": 3
    },
    {
      "token": "器",
      "start_offset": 4,
      "end_offset": 5,
      "type": "<IDEOGRAPHIC>",
      "position": 4
    },
    {
      "token": "后",
      "start_offset": 6,
      "end_offset": 7,
      "type": "<IDEOGRAPHIC>",
      "position": 5
    },
    {
      "token": "边",
      "start_offset": 7,
      "end_offset": 8,
      "type": "<IDEOGRAPHIC>",
      "position": 6
    },
    {
      "token": "是",
      "start_offset": 8,
      "end_offset": 9,
      "type": "<IDEOGRAPHIC>",
      "position": 7
    },
    {
      "token": "测",
      "start_offset": 9,
      "end_offset": 10,
      "type": "<IDEOGRAPHIC>",
      "position": 8
    },
    {
      "token": "试",
      "start_offset": 10,
      "end_offset": 11,
      "type": "<IDEOGRAPHIC>",
      "position": 9
    },
    {
      "token": "内",
      "start_offset": 11,
      "end_offset": 12,
      "type": "<IDEOGRAPHIC>",
      "position": 10
    },
    {
      "token": "容",
      "start_offset": 12,
      "end_offset": 13,
      "type": "<IDEOGRAPHIC>",
      "position": 11
    },
    {
      "token": "spring",
      "start_offset": 14,
      "end_offset": 20,
      "type": "<ALPHANUM>",
      "position": 12
    },
    {
      "token": "cloud",
      "start_offset": 21,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 13
    }
  ]
}
```

会发现分词的效果将“测试”这个词拆分成两个单字“测”和“试”，这是因为当前索引库使用的分词器对中文就是单字分词。

## 2. 中文分词器

### 2.1.Lucene自带中文分词器

> StandardAnalyzer：

单字分词：就是按照中文一个字一个字地进行分词。如：“我爱中国”，
效果：“我”、“爱”、“中”、“国”。

> CJKAnalyzer

二分法分词：按两个字进行切分。如：“我是中国人”，效果：“我是”、“是中”、“中国”“国人”。



上边两个分词器无法满足需求。

> SmartChineseAnalyzer

对中文支持较好，但扩展性差，扩展词库和禁用词库等不好处理

### 2.2.第三方中文分析器

 **paoding**： [庖丁解牛](https://code.google.com/p/paoding/) 中最多支持Lucene 3.0，已经过时，不予考虑。

**IK-analyzer**：[IK](https://code.google.com/p/ik-analyzer/)支持Lucene 4.10从20年12月推出1.0版开始， IKAnalyzer已经推出了4个大版本。最初，它是以开源项目Luence为应用主体的，结合词典分词和文法分析算法的中文分词组件。从3.0版本开 始，IK发展为面向Java的公用分词组件，独立于Lucene项目，同时提供了对Lucene的默认优化实现。在2012版本中，IK实现了简单的分词 歧义排除算法，标志着IK分词器从单纯的词典分词向模拟语义分词衍化。 但是也就是2012年12月后没有在更新。

## 3. 安装IK分词器

使用IK分词器可以实现对中文分词的效果。

下载IK分词器：（[Github地址](https://github.com/medcl/elasticsearch-analysis-ik)）

1、下载zip：

2、解压，并将解压的文件拷贝到ES安装目录的plugins下的ik(重命名)目录下，重启es	

3、测试分词效果：

```json
POST /_analyze
{
  "text":"中华人民共和国人民大会堂",
  "analyzer":"ik_smart"
}
```

## 4. 两种分词模式

ik分词器有两种分词模式：ik_max_word和ik_smart模式。

1、ik_max_word

​       会将文本做最细粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为“中华人民共和国、中华人民、中华、华人、人民大会堂、人民、共和国、大会堂、大会、会堂等词语。

2、ik_smart

​       会做最粗粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为中华人民共和国、人民大会堂。

## 5. 自定义词库

如果要让分词器支持一些专有词语，可以自定义词库。

iK分词器自带的main.dic的文件为扩展词典，stopword.dic为停用词典。	

也可以上边的目录中新建一个my.dic文件（<font color=red>注意文件格式为utf-8（不要选择utf-8 BOM）</font>）

可以在其中自定义词汇：

比如定义：

配置文件中 配置my.dic，

```xml
?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict">main.dic</entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords">stopword.dic</entry>
        <!--用户可以在这里配置远程扩展字典 -->
        <!-- <entry key="remote_ext_dict">words_location</entry> -->
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

​	

#  field详细介绍

上边安装了ik分词器，如何在索引和搜索时去使用ik分词器呢？如何指定field的类型？比如日期类型、数值类型等。

[ES字段类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)

## 1. field的属性介绍

### 1.1. type：

通过type属性指定field的类型。

```json
"name":{	
       "type":"text"
}
```

### 1.2. analyzer：

通过analyzer属性指定分词模式。

```json
 "name": {
                  "type": "text",
                  "analyzer":"ik_max_word"
   }
```

上边指定了analyzer是指在索引和搜索都使用ik_max_word，如果单独想定义搜索时使用的分词器则可以通过search_analyzer属性。
对于ik分词器建议是索引时使用ik_max_word将搜索内容进行细粒度分词，搜索时使用ik_smart提高搜索精确性。

```json
"name": {
                  "type": "text",
                  "analyzer":"ik_max_word",#生成索引目录时
                  "search_analyzer":"ik_smart"#检索时
 }
```

### 1.3. index：

通过index属性指定是否<font color=red>索引</font>。
默认为index=true，即要进行索引，只有进行索引才可以从索引库搜索到。
但是也有一些内容不需要索引，比如：商品图片地址只被用来展示图片，不进行搜索图片，此时可以将index设置
为false。	
删除索引，重新创建映射，将pic的index设置为false，尝试根据pic去搜索，结果搜索不到数据

```json
"pic": {
  	   "type":"text",           
       "index":false
}
```

### 1.4. source：

​	如果某个字段内容非常多，业务里面只需要能对该字段进行搜索，比如：商品描述。查看文档内容会再次到mysql或者hbase中取数据，把大字段的内容存在Elasticsearch中只会增大索引，这一点文档数量越大结果越明显，如果一条文档节省几KB，放大到亿万级的量结果也是非常可观的。 

如果只想存储某几个字段的原始值到Elasticsearch，可以通过incudes参数来设置，在mapping中的设置如下:

```json
POST /java/course/_mapping
{
  "_source": {
    "includes":["description"]
  }
}
```

同样，可以通过excludes参数排除某些字段：

```json
POST /java/course/_mapping
{
  "_source": {
    "excludes":["description"]
  }
}
```

## 2. 常用field类型

### 2.1. text文本字段

例如：
1、创建新映射：	

```json
POST /java/course/_mapping
{
  "_source": {
    "includes":["description"]
  }  
  "properties": {   
       "name": {
           "type": "text",
           "analyzer":"ik_max_word",
           "search_analyzer":"ik_smart"
       },         
      "description": {
          "type": "text",
          "analyzer":"ik_max_word",
          "search_analyzer":"ik_smart"
      },
      "pic":{
          "type":"text",
          "index":false
      }
  }   
}
```

2、插入文档：

```json
POST /java/course/1
{
  "name":"python从入门到放弃",
  "description":"人生苦短，我用Python",
  "pic":"250.jpg"
}
```

3、查询测试：

```json
GET /java/course/_search?q=name:放弃
GET /java/course/_search?q=description:人生
GET /java/course/_search?q=pic:250.jpg
```

结果：name和description都支持全文检索，pic不可作为查询条件

### 2.2. keyword关键字字段

上边介绍的text文本字段在映射时要设置分词器，keyword字段为关键字字段，通常搜索keyword是按照整体搜索，所以创建keyword字段<font color=red>往索引目录写时是不进行分词</font>的，比如：邮政编码、手机号码、身份证等。keyword字段通常用于过虑、排序、聚合等。

例如：
1、更改映射：

```json
POST /java/course/_mapping
{
 	"properties": {
       "studymodel":{
          "type":"keyword"
       }
 	}
}
```

2、插入文档：

```json
PUT /java/course/2
{
 "name": "java编程基础",
 "description": "java语言是世界第一编程语言",
 "pic":"250.jpg",
 "studymodel": "2010年01月"
}
```

3、根据name查询文档：

```
GET /java/course/_search?q=studymodel:2010年01月
```

name是keyword类型，所以查询方式是精确查询。

### 2.3. date日期类型	

日期类型不用设置分词器，通常日期类型的字段用于排序。
1)format
通过format设置日期格式，多个格式使用双竖线||分隔, 每个格式都会被依次尝试, 直到找到匹配的

例如：
1、设置允许date字段存储年月日时分秒、年月日及毫秒三种格式。

```json
POST /java/course/_mapping
{
	"properties": {
       "timestamp": {
         "type":   "date",
         "format": "yyyy-MM-dd"
       }
     }
}
```

2、插入文档：	

```json
PUT /java/course/3
{
"name": "spring开发基础",
"description": "spring 在java领域非常流行，java程序员都在用。",
"studymodel": "201001",
 "pic":"250.jpg",
 "timestamp":"2018-07-04 18:28:58"
}
```

### 2.4. Numeric类型

es中的数字类型经过分词(特殊)后支持排序和区间搜索		

例如：
1、更新已有映射：

```json
POST /java/course/_mapping
{
	"properties": {
	"price": {
        "type": "float"
     }
  }
} 
```

2、插入文档

```json
PUT /java/course/3
{
 "name": "spring开发基础",
 "description": "spring 在java领域非常流行，java程序员都在用。",
 "studymodel": "201001",
 "pic":"250.jpg",
 "price":38.6
}
```

## 3. field属性的设置标准

| 属性   | 标准       |
| ------ | ---------- |
| type   | 是否有意义 |
| index  | 是否搜索   |
| source | 是否展示   |



# Spring Boot整合ElasticSearch

## 1. ES客户端

ES提供多种不同的客户端：

1、TransportClient

​       ES提供的传统客户端，官方计划8.0版本删除此客户端。

2、RestClient

​       RestClient是官方推荐使用的，它包括两种：REST Low Level Client和 REST High Level  Client。ES在6.0之后提供REST High Level  Client， 两种客户端官方更推荐使用 REST High Level  Client，不过当前它还处于完善中，有些功能还没有。

## 2. 搭建工程

### 2.1. pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>springboot_elasticsearch</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <!-- 修改elasticsearch的版本 -->
    <properties>
        <elasticsearch.version>6.2.3</elasticsearch.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>${elasticsearch.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>
</project>

```

### 2.2. application.yml

```yaml
spring:
  elasticsearch:
    rest:
      uris:
        - http://192.168.204.132:9200
```

### 2.3. app

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ElasticsearchApp {

	public static void main(String[] args) {
		SpringApplication.run(ElasticsearchApp.class, args);
	}
}
```

## 3. 索引管理

### 3.1. 创建索引库

#### 3.1.1.api

创建索引库：

```json
PUT /java
{
  "settings":{
       "number_of_shards" : 2,
       "number_of_replicas" : 0
  }
}
```

创建映射：

```json
POST /java/course/_mapping
{
  "_source": {
    "excludes":["description"]
  }, 
 	"properties": {
      "name": {
          "type": "text",
          "analyzer":"ik_max_word",
          "search_analyzer":"ik_smart"
      },
      "description": {
          "type": "text",
          "analyzer":"ik_max_word",
          "search_analyzer":"ik_smart"
       },
       "studymodel": {
          "type": "keyword"
       },
       "price": {
          "type": "float"
       },
       "pic":{
		   "type":"text",
		   "index":false
	    }
  }
}
```

#### 3.1.2.Java Client

```java
package com.example.test;

import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import org.elasticsearch.action.DocWriteResponse;
import org.elasticsearch.action.admin.indices.create.CreateIndexRequest;
import org.elasticsearch.action.admin.indices.create.CreateIndexResponse;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.client.IndicesClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = {ElasticsearchApp.class})
public class IndexWriterTest {
	@Autowired
    private RestHighLevelClient restHighLevelClient;

   //创建索引库
    @Test
    public void testCreateIndex() throws IOException {
        //创建“创建索引请求”对象，并设置索引名称
        CreateIndexRequest createIndexRequest = new CreateIndexRequest("java");
        //设置索引参数
        createIndexRequest.settings("{\n" +
                "       \"number_of_shards\" : 2,\n" +
                "       \"number_of_replicas\" : 0\n" +
                "  }", XContentType.JSON);
        createIndexRequest.mapping("course", "{\r\n" + 
        		"  \"_source\": {\r\n" + 
        		"    \"excludes\":[\"description\"]\r\n" + 
        		"  }, \r\n" + 
        		" 	\"properties\": {\r\n" + 
        		"           \"name\": {\r\n" + 
        		"              \"type\": \"text\",\r\n" + 
        		"              \"analyzer\":\"ik_max_word\",\r\n" + 
        		"              \"search_analyzer\":\"ik_smart\"\r\n" + 
        		"           },\r\n" + 
        		"           \"description\": {\r\n" + 
        		"              \"type\": \"text\",\r\n" + 
        		"              \"analyzer\":\"ik_max_word\",\r\n" + 
        		"              \"search_analyzer\":\"ik_smart\"\r\n" + 
        		"           },\r\n" + 
        		"           \"studymodel\": {\r\n" + 
        		"              \"type\": \"keyword\"\r\n" + 
        		"           },\r\n" + 
        		"           \"price\": {\r\n" + 
        		"              \"type\": \"float\"\r\n" + 
        		"           },\r\n" + 
        		"  }\r\n" + 
        		"}", XContentType.JSON);
        //创建索引操作客户端
        IndicesClient indices = restHighLevelClient.indices();

        //创建响应对象
        CreateIndexResponse createIndexResponse = 
            indices.create(createIndexRequest);
        //得到响应结果
        boolean acknowledged = createIndexResponse.isAcknowledged();
        System.out.println(acknowledged);
    } 
  }
```



### 3.2. 删除索引库

#### 3.2.1.api

```json
DELETE /java
```

#### 3.2.2.java client

```java
	//删除索引库
	@Test
	public void testDeleteIndex() throws IOException {
		//创建“删除索引请求”对象
		DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("java");
		//创建索引操作客户端
		IndicesClient indices = restHighLevelClient.indices();
		//创建响应对象
		DeleteIndexResponse deleteIndexResponse = 
            indices.delete(deleteIndexRequest);
		//得到响应结果
		boolean acknowledged = deleteIndexResponse.isAcknowledged();
		System.out.println(acknowledged);
	}
```

### 3.2. 添加文档

#### 3.2.1.api

```json
POST /java/course/1
{
 "name":"spring cloud实战",
 "description":"本课程主要从四个章节进行讲解： 1.微服务架构入门 2.spring cloud 基础入门 3.实战Spring Boot 4.注册中心eureka。",
 "studymodel":"201001",
 "price":5.6
}
```

#### 3.2.2.java client

```java
	//添加文档
	@Test
	public void testAddDocument() throws IOException {
		//创建“索引请求”对象：索引当动词
		IndexRequest indexRequest = new IndexRequest("java", "course", "1");
		indexRequest.source("{\n" +
				" \"name\":\"spring cloud实战\",\n" +
				" \"description\":\"本课程主要从四个章节进行讲解： 1.微服务架构入门 " +
				"2.spring cloud 基础入门 3.实战Spring Boot 4.注册中心nacos。\",\n" +
				" \"studymodel\":\"201001\",\n" +
				" \"price\":5.6\n" +
				"}", XContentType.JSON);
		IndexResponse indexResponse = 
            restHighLevelClient.index(indexRequest);
		System.out.println(indexResponse.toString());
	}
```

### 3.3. 批量添加文档

支持在一次API调用中，对不同的索引进行操作。支持四种类型的操作：index、create、update、delete。

- 语法：

```
POST /_bulk
{ action: { metadata }} 
{ requestbody }\n
{ action: { metadata }} 
{ requestbody }\n
...
```

#### 3.3.1.api

```json
POST /_bulk
{"index":{"_index":"java","_type":"course"}}
{"name":"php实战","description":"php谁都不服","studymodel":"201001","price":"5.6"}
{"index":{"_index":"java","_type":"course"}}
{"name":"net实战","description":"net从入门到放弃","studymodel":"201001","price":"7.6"}
```

#### 3.3.2.java client

```java
@Test
public void testBulkAddDocument() throws IOException {
    BulkRequest bulkRequest = new BulkRequest();
    bulkRequest.add(new IndexRequest("java", "course").source("{...}",
                                                                  XContentType.JSON));
    bulkRequest.add(new IndexRequest("java", "course").source("{...}",
                                                                  XContentType.JSON));
    BulkResponse bulkResponse = 
                   restHighLevelClient.bulk(bulkRequest);
    System.out.println(bulkResponse.hasFailures());
}
```

### 3.4. 修改文档

#### 3.4.1.api

```json
PUT /java/course/1
{
 "price":66.6
}
```

#### 3.4.2.java client

```java
//更新文档
@Test
public void testUpdateDocument() throws IOException {
    UpdateRequest updateRequest = new UpdateRequest("java", "course", "1");
    updateRequest.doc("{\n" +
            "  \"price\":7.6\n" +
            "}", XContentType.JSON);
    UpdateResponse updateResponse = 
                   restHighLevelClient.update(updateRequest);
    System.out.println(updateResponse.getResult());
}
```

### 3.5. 删除文档

#### 3.5.1.api

```
DELETE /java/coures/1
```

#### 3.5.2.java client

```java
    //根据id删除文档
    @Test
    public void testDelDocument() throws IOException {
        //删除请求对象
        DeleteRequest deleteRequest = new DeleteRequest("java","course","1");
        //响应对象
        DeleteResponse deleteResponse = 
            restHighLevelClient.delete(deleteRequest);
        System.out.println(deleteResponse.getResult());
    }
```

## 4. 文档搜索

### 4.1.准备环境

向索引库中插入以下数据：

```json
PUT /java/course/1
{
  "name": "Bootstrap开发",
  "description": "Bootstrap是由Twitter推出的一个前台页面开发css框架，是一个非常流行的开发框架，此框架集成了多种页面效果。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长css页面开发的程序人员）轻松的实现一个css，不受浏览器限制的精美界面css效果。",
  "studymodel": "201002",
  "price":38.6,
  "pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}

PUT /java/course/2
{
  "name": "java编程基础",
  "description": "java语言是世界第一编程语言，在软件开发领域使用人数最多。",
  "studymodel": "201001",
  "price":68.6,
  "pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}

PUT /java/course/3
{
  "name": "spring开发基础",
  "description": "spring 在java领域非常流行，java程序员都在用。",
  "studymodel": "201001",
  "price":88.6,
  "pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}
```

### 4.2. 简单搜索

简单搜索就是通过url进行查询，以get方式请求ES。
语法：

> GET /index_name/type_name/doc_id
> GET [/index_name/type_name/]_search[?parameter_name=parameter_value&...]

例如：

```json
GET /java/course/_search?q=name:spring&sort=price:desc
```

注意：
	如果查询条件复杂，很难构建搜索条件 ，生产环境中很少使用。
	例如：要求搜索条件为商品名称包含手机，价格在 1000~5000之间，销量在每月 500 以上，根据价格升序排列，分页查询第二页，每页 40 条数据：?q=xxxx:xxx&range=xxx:xxx:xxx&aggs&sort&from&size

#### 4.2.1.api

```
GET /java/course/1
```

#### 4.2.2.java client

```java
    //查询文档
    @Test
    public void getDoc() throws IOException {
        GetRequest getRequest = new GetRequest("java","course","1");
        GetResponse getResponse = restHighLevelClient.get(getRequest);
        boolean exists = getResponse.isExists();
        System.out.println(exists);
		String source = getResponse.getSourceAsString();
		System.out.println(source);
    }
```

### 4.3. DSL搜索

DSL(Domain Specific Language)是ES提出的基于json的搜索方式，在搜索时传入特定的json格式的数据来完成不同的搜索需求，DSL比URI搜索方式功能强大，在项目中建议使用DSL方式来完成搜索。
语法：

> ​	GET /index_name/type_name/_search
> ​	{
> ​		"commond":{
> ​			"parameter_name" : "parameter_value"
> ​		}
> ​	}

#### 4.3.1.match_all查询

##### 4.3.1.1.api

```json
GET /java/course/_search
{
  "query" : { 
    "match_all" : {}
  }
}
```

##### 4.3.1.2.java client

```java
package com.example.test;

import com.example.ElasticsearchApp;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.sort.SortOrder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.io.IOException;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = {ElasticsearchApp.class})
public class IndexReaderTest {
    @Autowired
    private RestHighLevelClient restHighLevelClient;
    private SearchRequest searchRequest;
    private SearchResponse searchResponse;

    @Before
    public void init(){
        searchRequest = new SearchRequest();
        searchRequest.indices("java");
        searchRequest.types("course");
    }

    @Test
    public void testMatchAll() throws IOException {
        //2、创建 search请求对象
        SearchRequest searchRequest = new SearchRequest();
        searchRequest.indices("java");
        searchRequest.types("course");

        //3、创建 参数构造器
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());

        //4、设置请求参数
        searchRequest.source(searchSourceBuilder);

        //1、调用search方法
        SearchResponse searchResponse = restHighLevelClient.search(searchRequest);

        SearchHits searchHits = searchResponse.getHits();
        
        long totalHits = searchHits.getTotalHits();
        System.out.println("共搜索到"+totalHits+"条文档");

        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
        }
    }

    @After
    public void show(){
        SearchHits searchHits = searchResponse.getHits();
        long totalHits = searchHits.getTotalHits();
        System.out.println("共搜索到"+totalHits+"条文档");

        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
        }
    }
}
```

#### 3.3.2.分页查询

##### 3.3.2.1.api

```json
GET /java/course/_search
{
  "query" : { "match_all" : {} },
  "from" : 1, # 从第几条数据开始查询，从0开始计数
  "size" : 3, # 查询多少数据
  "sort" : [
    { "price" : "asc" }
  ]
}
```

##### 3.3.2.2.java client

```java
//分页查询
@Test
public void testSearchPage() throws Exception {
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.matchAllQuery());
    searchSourceBuilder.from(1);
    searchSourceBuilder.size(5);
    searchSourceBuilder.sort("price", SortOrder.ASC);

    // 设置搜索源
    searchRequest.source(searchSourceBuilder);
    // 执行搜索
    searchResponse = restHighLevelClient.search(searchRequest);
}
```

#### 3.3.4.match查询

match Query即全文检索，它的搜索方式是先将搜索字符串分词，再使用各各词条从索引中搜索。

##### 3.3.4.1.api

query：搜索的关键字
operator：or 表示 只要有一个词在文档中出现则就符合条件，and表示每个词都在文档中出现则才符合条件。

1、基本使用：

```json
GET /java/course/_search
{
  "query" : {
    "match" : {
      "name": {
        "query": "spring开发"
      }
    }
  }
}
```

2、operator：

```json
GET /java/course/_search
{
  "query" : {
    "match" : {
      "name": {
        "query": "spring开发",
        "operator": "and"
      }
    }
  }
}
```

上边的搜索的执行过程是：
	1、将“spring开发”分词，分为spring、开发两个词
	2、再使用spring和开发两个词去匹配索引中搜索。
	3、由于设置了operator为and，必须匹配两个词成功时才返回该文档。

##### 3.3.4.2	java client

```java
@Test
public void testMatchQuery() throws Exception {
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.matchQuery("name", "spring开
                                                       发").operator(Operator.AND));
		
    // 设置搜索源
    searchRequest.source(searchSourceBuilder);
    // 执行搜索
    searchResponse = restHighLevelClient.search(searchRequest);
 }
```



#### 3.3.5.multi_match查询

matchQuery是在一个field中去匹配，multiQuery是拿关键字去多个Field中匹配。

##### 3.3.5.1.api

1、基本使用
例子：关键字 “开发”去匹配name 和description字段

```json
GET /java/course/_search
{
  "query": {
    "multi_match": {
      "query": "开发",
      "fields": ["name","description"]
    }
  }
}
```

注意：此搜索操作适合构建复杂查询条件，生产环境常用。

##### 3.3.5.2.java client

```java
@Test
public void testMultiMatchQuery() throws Exception {
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.multiMatchQuery("开发","name","description"));
		
    // 设置搜索源
    searchRequest.source(searchSourceBuilder);
    // 执行搜索
    searchResponse = restHighLevelClient.search(searchRequest);
}
```



#### 3.3.6.bool查询

布尔查询对应于Lucene的BooleanQuery查询，实现将多个查询组合起来。
参数：
	must：表示必须，多个查询条件必须都满足。（通常使用must）
	should：表示或者，多个查询条件只要有一个满足即可。
	must_not：表示非。

##### 3.3.6.1.api

例如：查询name包括“开发”并且价格区间是1-100的文档

```json
GET /java/course/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "开发"
          }
        },
        {
          "range": {
            "price": {
              "gte": 50,
              "lte": 100
            }
          }
        }
      ]
    }
  }
}
```

##### 3.3.6.2.java client

```java
    @Test
    public void testBooleanMatch() throws IOException {
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        //json条件
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(QueryBuilders.matchQuery("name","开发"));
        boolQueryBuilder.must(QueryBuilders.rangeQuery("price").gte("50").lte(100));
        searchSourceBuilder.query(boolQueryBuilder);

        searchRequest.source(searchSourceBuilder);
        SearchResponse searchResponse = restHighLevelClient.search(searchRequest);
    }
```

#### 3.3.7.filter查询

过滤查询。此操作实际上就是 query DSL 的补充语法。过滤的时候，不进行任何的匹配分数计算，相对于 query 来说，filter 相对效率较高。Query 要计算搜索匹配相关度分数。Query更加适合复杂的条件搜索。

##### 3.3.7.1.api

如：使用bool查询，搜索 name中包含 "开发"的数据，且price在 10~100 之间
1、不使用 filter， name和price需要计算相关度分数：

```json
GET /java/course/_search
{
  "query": {
     "bool" : {
        "must":[
            {
               "match": {
                 "name": "开发"
               }
            },
            {
              "range": {# 范围， 字段的数据必须满足某范围才有结果。
                "price": {
                  "gte": 10, # 比较符号 lt gt lte gte
                  "lte": 100
                }
              }
            }
        ]
     }
  }
}
```

2、使用 filter， price不需要计算相关度分数：

```json
GET /java/course/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "开发"
          }
        }
      ],
      "filter": {# 过滤，在已有的搜索结果中进行过滤，满足条件的返回。
        "range": {
          "price": {
            "gte": 1,
            "lte": 100
          }
        }
      }
    }
  }
}
```

##### 3.3.7.2.java client

```java
@Test
public void testFilterQuery() throws IOException {
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
    boolQueryBuilder.must(QueryBuilders.matchQuery("name","开发"));
    boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gte(10).lte(100))
    searchSourceBuilder.query(boolQueryBuilder);
    searchRequest.source(searchSourceBuilder);
    searchResponse = restHighLevelClient.search(searchRequest);
}
```

#### 3.3.8.highlight查询

高亮显示：高亮不是搜索条件，是显示逻辑，在搜索的时候，经常需要对搜索关键字实现高亮显示。

##### 3.3.8.1.api

例如：

```json
GET /java/course/_search
{
  "query": {
    "match": {
      "name": "开发"
    }
  },
  "highlight": {
      "pre_tags": ["<font color='red'>"],
      "post_tags": ["</font>"],
      "fields": {"name": {}}
  }
}
```

##### 3.3.8.2.java clent

1、查询：

```java
  @Test
  public void testHighLightQuery() throws Exception {
      SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
      searchSourceBuilder.query(QueryBuilders.matchQuery("name", "spring"));
      //设置高亮
      HighlightBuilder highlightBuilder = new HighlightBuilder();
      highlightBuilder.preTags("<font color='red'>");
      highlightBuilder.postTags("</font>");
      highlightBuilder.fields().add(new HighlightBuilder.Field("name"));
      searchSourceBuilder.highlighter(highlightBuilder);

      searchRequest.source(searchSourceBuilder);
      searchResponse = restHighLevelClient.search(searchRequest);
}
```

2、遍历：

```java
 @After
public void displayDoc() {
    SearchHits searchHits = searchResponse.getHits();
    long totalHits = searchHits.getTotalHits();
    System.out.println("共搜索到" + totalHits + "条文档");

    SearchHit[] hits = searchHits.getHits();
    for (int i = 0; i < hits.length; i++) {
        SearchHit hit = hits[i];
        String id = hit.getId();
        System.out.println("id：" + id);
        String source = hit.getSourceAsString();
        System.out.println(source);

        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        if (highlightFields != null) {
            HighlightField highlightField = highlightFields.get("name");
            Text[] fragments = highlightField.getFragments();
            System.out.println("高亮字段：" + fragments[0].toString());
        }
    }

}
```

# 集群管理

## 1.集群结构

ES通常以集群方式工作，这样做不仅能够提高 ES的<font color=red>搜索能力</font>还可以处理大数据搜索的能力，同时也增加了系统的<font color=red>容错能力</font>及高可用。

[ES集群结构的示意图](https://www.yijiyong.com/es/basic/05-colonyprin.html)	

此处的设置为：每个主分片有两个副本， 如果某个节点挂了也不怕，比如节点1挂了，我们可以查询位于节点3和节点3上的副本0

添加文档过程：

（1）假设用户把请求发给了节点1

（2）系统通过余数算法得知这个’文档’应该属于主分片2，于是请求被转发到保存该主分片的节点3

（3）系统把文档保存在节点3的主分片2中，然后将请求转发至其他两个保存副本的节点。

查询文档过程：

（1） 请求被发给了节点1

（2）节点1计算出该数据属于主分片2，这时候，有三个选择，分别是位于节点1的副本2， 节点2的副本2，节点3的主分片2， 假设节点1负载均衡，采用轮询的方式，选中了节点2，把请求转发。

（3） 节点2把数据返回给节点1， 节点1 最后返回给客户端。

## 2. 创建结点2

1、拷贝节点elasticsearch-1	

2、修改elasticsearch.yml内容如下：

```yaml
node.name: power_shop_node_2
discovery.zen.ping.unicast.hosts: ["192.168.204.132:9300", "192.168.204.133:9300"]
```

3、<font color='red'>删除节点2的data目录</font>

## 3. 查看集群健康状态

1、查询当前集群的健康信息：

```json
GET /_cluster/health
```

2、结果：

```json
{
  "cluster_name": "power_shop",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 2,
  "number_of_data_nodes": 2,
  "active_primary_shards": 2,
  "active_shards": 4,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}
```

**status**：用三种颜色来展示健康状态

​       green：索引库的每个 primary shard 和 replica shard 都是 active 的

​       yellow：索引库的每个 primary shard 都是 active 的，但部分的 replica shard 不是 active 的，如单节点创建

​                      备份分配

​       red：不是所有的 primary shard 都是 active 状态的。

## 4. 测试

1、启动两个节点 ，测试集群健康状况和分片情况

2、关闭节点2，测试集群状态

3、创建备份分配，关闭节点2，再测试集群状态

### 参考

[♥ElasticSearch知识体系详解♥](https://pdai.tech/md/db/nosql-es/elasticsearch.html)

[ES-入门介绍](https://www.yijiyong.com/es/basic/01-intro.html)

<div>
<hr>
<script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script> 
<link
  rel="stylesheet"
  href="https://unpkg.com/@waline/client@v2/dist/waline.css"
/>
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>