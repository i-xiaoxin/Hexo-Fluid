---
title: MyCat中间件MySQL分库分表、读写分离
date: 2022-11-10 20:19:02
tags: 
- MySQL
- MyCat
- CentOS7
index_img: 
banner_img: /img/article1.webp
categories:
- MyCat
comment: waline
---

# MyCat分库分表、读写分离

### 1、MyCat简介

[MyCat](http://www.mycat.org.cn/)是Java语言编写的MySQL数据库网络协议的开源中间件,前身是cobar项目。

● 性能有瓶颈了，MyCat可以读写分离

● 数据库容量有瓶颈了，MyCat可以分库分表

[Mycat-Server GitHub](https://github.com/MyCATApache/Mycat-Server)

### 2、MyCAT架构

MyCAT使用Mysql的通讯协议模拟成了一个Mysql服务器，所有能使用Mysql的客户端以及编程语言都能将MyCAT当成是Mysql Server来使用，不必开发新的客户端协议。

<img src="https://img-blog.csdn.net/20180826092536171?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25hbmdlYWxp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" style="zoom:80%;" >

### 3、MyCat分库分表

**数据库分布式核心内容无非就是数据切分（Sharding）**，以及切分后对数据的定位、整合。数据切分就是将数据分散存储到多个数据库中，使得单一数据库中的数据量变小，通过扩充主机的数量缓解单一数据库的性能问题，从而达到提升数据库操作性能的目的。

数据切分根据其切分类型，可以分为两种方式：`垂直（纵向）切分`和`水平（横向）切分`

**垂直切分**一般是按照业务维度进行数据库表的切分;把相同类型的表放在一个数据库，另一些表放在另一个数据库;也就是在不同库建不同表，把表分散到各个数据库;

**水平拆分**不是将表做分类，而是按照某个字段的某种规则来分散到多个库之中，每个表中包含一部分数据。简单来说，我们可以将数据的水平切分理解为是按照数据行的切分，就是将表中的某些行切分到一个数据库，而另外的某些行又切分到其他的数据库中。

#### 垂直切分的优缺点

优点：

- 解决业务系统层面的耦合，业务清晰
- 与微服务的治理类似，也能对不同业务的数据进行分级管理、维护、监控、扩展等
- 高并发场景下，垂直切分一定程度的提升 IO、数据库连接数、单机硬件资源的瓶颈

缺点：

- 部分表无法 join，只能通过接口聚合方式解决，提升了开发的复杂度
- 分布式事务处理复杂
- 依然存在单表数据量过大的问题（需要水平切分）

#### 水平切分的优缺点

优点：

- 不存在单库数据量过大、高并发的性能瓶颈，提升系统稳定性和负载能力
- 应用端改造较小，不需要拆分业务模块

缺点：

- 跨分片的事务一致性难以保证
- 跨库的 join 关联查询性能较差
- 数据多次扩展难度和维护量极大

### 4、MyCat的安装和启动

推荐[MyCat 1.67](https://github.com/MyCATApache/Mycat-Server/releases/tag/1.6.76-release-2020-11-2)下载的文件直接解压即可

```sh
./mycat start #启动

./mycat stop #停止

./mycat console #前台运行

./mycat install #添加到系统自动启动（暂未实现）

./mycat remove #取消随系统自动启动（暂未实现）

./mycat restart #重启服务

./mycat pause #暂停

./mycat status #查看启动状态
```

### 5、MyCat关键术语

- 逻辑库（schema）：一个包含了所有数据库的逻辑上的数据库

- 逻辑表（table）：一个包含了所有表的逻辑上的表
- 数据主机（dataHost）：数据库软件安装到哪个服务器上
- 数据节点（dataNode）：服务器上的mysql
- 分片规则（rule）：拆分规则

### 6、MyCat配置

--bin 启动目录

--conf 配置目录存放配置文件：

```
  --server.xml：是Mycat服务器参数调整和用户授权的配置文件。

  --schema.xml：是逻辑库定义和表以及分片定义的配置文件。

  --rule.xml：  是分片规则的配置文件，分片规则的具体一些参数信息单独存放为文件，也在这个目录下，配置文件修改需要重启MyCAT。

  --log4j.xml： 日志存放在logs/log中，每天一个文件，日志的配置是在conf/log4j.xml中，根据自己的需要可以调整输出级别为debug                           debug级别下，会输出更多的信息，方便排查问题。

  --autopartition-long.txt,partition-hash-int.txt,sequence_conf.properties， sequence_db_conf.properties 分片相关的id分片规则配置文件

  --lib	    MyCAT自身的jar包或依赖的jar包的存放目录。

  --logs        MyCAT日志的存放目录。日志存放在logs/log中，每天一个文件
```

#### 配置schema.xml

Schema.xml作为MyCat中重要的配置文件之一，管理着MyCat的逻辑库、表、分片规则、DataNode以及DataHost。

`schema `是实际逻辑库的配置，user，pay分别对应两个逻辑库，多个schema代表多个逻辑库。

`dataNode`是逻辑库对应的分片，如果配置多个分片只需要多个dataNode即可。

`dataHost`是实际的物理库配置地址，可以配置多主主从等其他配置，多个dataHost代表分片对应的物理库地址，下面的writeHost、readHost代表该分片是否配置多写，主从，读写分离等高级特性。

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
  <schema name="user" checkSQLschema="false" sqlMaxLimit="100" dataNode="user" />
  <schema name="pay"  checkSQLschema="false" sqlMaxLimit="100" dataNode="pay" >
     <table name="order" dataNode="pay1,pay2,pay3" rule="crc32slot"/>
     <table name="shipping" dataNode="pay1,pay2,pay3" rule="crc32slot1"/>
     <table name="item" dataNode="pay1,pay2,pay3" rule="crc32slot2"/>
  </schema>

  <dataNode name="user" dataHost="host" database="user" />
  <dataNode name="pay1" dataHost="host" database="pay1" />
  <dataNode name="pay2" dataHost="host" database="pay2" />
 	<dataNode name="pay3" dataHost="host" database="pay3" />
  
  <dataHost name="host" maxCon="1000" minCon="10" balance="0"
     writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
     <heartbeat>select 1</heartbeat>
     <!-- can have multi write hosts -->
     <writeHost host="hostM1" url="192.168.0.2:3306" user="root" password="root" >
   		<readHost host="hostS2" url="192.168.0.3:3306" user="root" password="root" />
    </writeHost>
  </dataHost>
</mycat:schema>
```

####  配置server.xml

server.xml几乎保存了所有mycat需要的系统配置信息。最常用的是在此配置用户名、密码及权限。

添加两个mycat逻辑库：user,pay
`system` 参数是所有的mycat参数配置，比如添加解析器：defaultSqlParser，其他类推; 默认
`user `是用户参数。

```xml
<!--默认配置即可-->
<system>
	<property name="defaultSqlParser">druidparser</property>
</system>

<!--root用户-->
<user name="root">
	<property name="password">DBpassword</property>
	<property name="schemas">user,pay</property>
</user>

<!--user用户只有读权限-->
<user name="user">
	<property name="password">DBpassword</property>
	<property name="schemas">user,pay</property>
  <property name="readOnly">true</property>
</user>
```

#### 配置rule 分片规则

##### auto-sharding-long 规则

```
以 500 万为单位,实现分片规则：
1-500 万保存在 db1 中, 500 万零 1 到 1000 万保存在 db2 中,1000 万零 1 到 1500 万保存在 db3 中.
```

##### crc32slot  规则

```
在 CRUD 操作时,根据具体数据的 crc32 算法计算,数据应该保存在哪一个dataNode 中
```

#### 配置rule.xml

```
1）<columns>id</columns>中推荐配置主键列

2）所有的 tableRule 只能使用一次。如果需要为多个表配置分片规则，那么需要在此重新定义该规则。三张表配置三个

3) 要分片的数据库节点数量，必须指定，否则没法分片
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:rule SYSTEM "rule.dtd">

<mycat:rule  xmlns:mycat="http://org.opencloudb/">
  <!-- tableRule 只能使用一次 三张表配置三个-->
  <tableRule name="crc32slot">
    <rule>
      <columns>orderID</columns>
      <algorithm>crc32slot12</algorithm>
    </rule>
  </tableRule>
  <tableRule name="crc32slot1">
    <rule>
      <columns>shippingID</columns>
      <algorithm>crc32slot12</algorithm>
    </rule>
  </tableRule>
  <tableRule name="crc32slot2">
    <rule>
      <columns>itemID</columns>
      <algorithm>crc32slot12</algorithm>
    </rule>
  </tableRule>
  
  <function name="crc32slot" class="io.mycat.route.function.PartitionByCRC32PreSlot">
    <property name="count">3</property><!-- 要分片的数据库数量，必须指定，否则没法分片 -->
	</function>
   
</mycat:rule >
```

### Mycat测试

测试mycat与测试mysql完全一致，mysql怎么连接，mycat就怎么连接。端口8066

命令行测试mycat连接：

mysql -uroot -proot -P8066 -h127.0.0.1

采用SQL Client(Navicat)工具连接,端口填写8066即可；

以上配置设置完成后，需要配置主从MySQL数据库;连接mycat后将配置的数据表导入mycat，同时主从数据库可查看已经分库分表

[mycat分库分表初体验](https://segmentfault.com/a/1190000024523321)

参考：

[那些年非常火的MyCAT是什么？](https://segmentfault.com/a/1190000021987297)

[分库分表方式方法](https://github.com/daydreamdev/MeetingFilm/blob/master/note/%E5%88%86%E5%BA%93%E5%88%86%E8%A1%A8.md)

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