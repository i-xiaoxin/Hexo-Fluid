---
title: Elasticsearch安装&kibana插件安装
date: 2022-10-24 21:09:02
tags: 
- Elasticsearch
- Kibana
- Elasticsearch-head
index_img: 
banner_img: /img/article3.webp
categories:
- Elasticsearch
comment: waline
---

# 1. ElasticSearch 介绍

 ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个基于RESTful web接口的分布式全文搜索引擎。ElasticSearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。ElasticSearch用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

类似产品：solr

# 2. 倒排索引

倒排索引（Inverted index）:也常被称为反向索引，倒排索引是**从关键字到文档**的映射（已知关键字求文档）。

逻辑结构部分是一个倒排索引表，由三部分组成：

1、将搜索的文档最终以<font color=red>Document</font>方式存储起来。

2、将要搜索的文档内容分词，所有不重复的词组成<font color=red>分词</font>列表。

3、每个分词和docment都有<font color=red>关联</font>。

推荐官方：[倒排索引](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html)

# 3. 安装 ElasticSearch 

## 环境要求

- jdk必须是jdk1.8.0_131以上版本
- ElasticSearch 需要至少4096 的线程池和 262144字节以上空间的虚拟内存才能正常启动，所以需要为虚拟机分配至少1.5G以上的内存
- 从5.0开始，ElasticSearch 安全级别提高了，不允许采用root帐号启动
- Elasticsearch的插件要求至少centos的内核要3.5以上版本

## 1. 创建用户

从5.0开始，ElasticSearch 安全级别提高了，不允许采用root帐号启动，所以我们要添加一个用户

```sh
# 创建elk 用户组
groupadd elk
# 创建用户admin
useradd admin
passwd admin
# 将admin用户添加到elk组
usermod -G elk admin
# 为用户分配权限
#chown将指定文件的拥有者改为指定的用户或组 -R处理指定目录以及其子目录下的所有文件
chown -R admin:elk /usr/upload
chown -R admin:elk /usr/local
```

## 2. 安装

Elasticsearch多版本 [下载地址](https://www.elastic.co/downloads/past-releases#)

```sh
# 下载安装包
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.3-linux-x86_64.tar.gz
# ES是Java开发的应用，解压即安装；已经安装jdk
tar -zxvf elasticsearch-6.2.3.tar.gz -C /usr/local
```

## 3. 配置文件

ES安装目录config中配置文件如下：

- elasticsearch.yml：用于配置Elasticsearch运行参数 
- jvm.options：用于配置Elasticsearch JVM设置 
- log4j2.properties：用于配置Elasticsearch日志

### 3.1. elasticsearch.yml

本项目配置如下：

```sh 
# 配置elasticsearch的集群名称，默认是elasticsearch。建议修改成一个有意义的名称。  
cluster.name: example_name
# 节点名，通常一台物理服务器就是一个节点，es会默认随机指定一个名字，建议指定一个有意义的名称，方便管理一个或多个节点组成一个cluster集群，集群是一个逻辑的概念，节点是物理概念
node.name: power_shop_node_1
# 设置绑定主机的ip地址，设置为0.0.0.0表示绑定任何ip，允许外网访问，生产环境建议设置为具体的ip
network.host: 0.0.0.0
# 设置对外服务的http端口，默认为9200
http.port: 9200
# 集群结点之间通信端口	
transport.tcp.port: 9300
# 设置集群中master节点的初始列表
discovery.zen.ping.unicast.hosts: ["0.0.0.0:9300", "0.0.0.0:9301"]
# 设置ES自动发现节点连接超时的时间，默认为3秒，如果网络延迟高可设置大些
discovery.zen.ping.timeout: 3s  
# 设置索引数据的存储路径，默认是es_home下的data文件夹，可以设置多个存储路径，用逗号隔开
path.data: /usr/local/elasticsearch-6.2.3/data
# 设置日志文件的存储路径，默认是es_home下的logs文件夹  
path.logs: /usr/local/elasticsearch-6.2.3/logs
# 是否支持跨域，默认为false
http.cors.enabled: true
# 当设置允许跨域，默认为*,表示支持所有域名
http.cors.allow-origin: /.*/
```

### 3.2.  jvm.options

设置最小及最大的JVM堆内存大小：

在jvm.options中设置 -Xms和-Xmx：

1） 两个值设置为相等

2） 将`Xmx` 设置为不超过物理内存的一半。

默认内存占用太多了，可以调小一些

```properties
-Xms512m
-Xmx512m
```

### 3.3. log4j2.properties

日志文件设置，ES使用log4j，注意日志级别的配置。

# 4. 启动ElasticSearch

## 4.1. 启动和关闭

```sh
# 进入bin目录
./elasticsearch
#或
./elasticsearch -d  

# 关闭
ps-ef|grep elasticsearch
kill -9 pid
```

# 5. 异常问题解决

## 5.1 内核问题

问题：error: requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled in when start elasticsearch [Github issues](https://github.com/elastic/elasticsearch/issues/28859)

```java
java.lang.UnsupportedOperationException: seccomp unavailable: requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled in
at org.elasticsearch.bootstrap.SystemCallFilter.linuxImpl(SystemCallFilter.java:328) ~[elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.SystemCallFilter.init(SystemCallFilter.java:616) ~[elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.JNANatives.tryInstallSystemCallFilter(JNANatives.java:258) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.Natives.tryInstallSystemCallFilter(Natives.java:113) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:110) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:172) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:323) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:121) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:112) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) [elasticsearch-cli-6.2.2.jar:6.2.2]
at org.elasticsearch.cli.Command.main(Command.java:90) [elasticsearch-cli-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92) [elasticsearch-6.2.2.jar:6.2.2]
at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:85) [elasticsearch-6.2.2.jar:6.2.2]
```

使用centos6，其linux内核版本为2.6。而Elasticsearch的插件要求至少3.5以上版本禁用这个插件即可。

修改elasticsearch.yml文件，在最下面添加如下配置：

```yaml
bootstrap.system_call_filter: false
```

## 5.2 文件创建权限问题

问题：

```java
ERROR: [1] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
```

解决：

Linux 默认来说，一般限制应用最多创建的文件是 4096个。但是 ES 至少需要 65536 的文件创建权限。我们用的是admin用户，而不是root，所以文件权限不足。

使用<font color=red>root</font>用户修改配置文件:

```
vim /etc/security/limits.conf
```

追加下面的内容：

```nginx
* soft nofile 65536
* hard nofile 65536
```

## 5.3 线程开启限制问题

问题：

```java
[2]: max number of threads [1024] for user [admin] is too low, increase to at least [4096]
```

解决：

默认的 Linux 限制 root 用户开启的进程可以开启任意数量的线程，其他用户开启的进程可以开启1024 个线程。必须修改限制数为4096+。因为 ES 至少需要 4096 的线程池预备。

​	如果虚拟机的内存是 1G，最多只能开启 3000+个线程数。至少为虚拟机分配 1.5G 以上的内存。

使用<font color=red>root</font>用户修改配置：

```sh
vim /etc/security/limits.d/90-nproc.conf
```

修改下面的内容：

```nginx
* soft nproc 1024
```

改为：

```nginx
* soft nproc 4096
```

## 5.4 虚拟内存问题

问题：

```java
[3]: max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
```

解决：

ES 需要开辟一个 262144字节以上空间的虚拟内存。Linux 默认不允许任何用户和应用直接开辟虚拟内存。

```sh
vim /etc/sysctl.conf
```

追加下面内容：

```sh
#限制一个进程可以拥有的VMA(虚拟内存区域)的数量
vm.max_map_count=655360 
```

然后执行命令，让sysctl.conf配置生效：

```sh
sysctl -p
```

# 6. 测试

​	ES 中只要启动了任意一个 ES 应用就是启动了一个 ES的 cluster 集群。默认的 ES集群命名为 elasticsearch。如果启动了多个应用（可以在多个节点或单一节点上启动多个应用），默认的ES 会自动找集群做加入集群的过程。

浏览器访问：host地址:9200

测试ok返回：

```json
{ 
  // node name 结点名称。随机分配的结点名称
  "name" : "power_shop_node_1",
  // cluster name 集群名称。 默认的集群名称
  "cluster_name" : "power_shop", 
  // 集群唯一 ID
  "cluster_uuid" : "RqHaIiYjSoOyrTGq3ggCOA", 
  "version" : {
    // 版本号
    "number" : "6.2.3", 
    "build_hash" : "c59ff00", 
    // 发布日期
    "build_date" : "2018-03-13T10:06:29.741383Z",
    // 是否快照版本
    "build_snapshot" : false,
    // 是否快照版本
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

# 7. 安装Kibana

Kibana是ES提供的一个基于Node.js的管理控制台, 可以很容易实现高级的数据分析和可视化，以图标的形式展现出来。

ElasticSearch官网 [下载Kibana](https://www.elastic.co/cn/downloads/kibana)

## 1. 安装

安装Kibana很方便，解压即安装

## 2. 修改config/kibana.yml配置

```yaml
server.port: 5601
server.host: "0.0.0.0" #允许来自远程用户的连接
elasticsearch.url: http://192.168.204.132:9200 #Elasticsearch实例的URL 
```

## 3. 启动

执行bin目录下kibana

## 4. 测试

浏览器访问：IP地址:5601

# 8. 安装head

head插件是ES的一个可视化管理插件，用来监视ES的状态，并通过head客户端和ES服务进行交互，比如创建映射、创建索引等。从ES6.0开始，head插件支持使得node.js运行。

elasticsearch-head[下载地址](https://github.com/mobz/elasticsearch-head)

### 安装运行测试

- `git clone git://github.com/mobz/elasticsearch-head.git`
- `cd elasticsearch-head`
- `npm install`
- `npm run start`

- `open` localhost:9100/





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