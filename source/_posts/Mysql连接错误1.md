---
title: Mysql异常错误处理-Establishing SSL connection without server's identity verification is not recommended. 
date: 2022-10-15 23:29:02
tags: 
- Mysql
index_img: /img/article8.webp
banner_img: /img/post_banner.webp
categories:
- Mysql
---

## 异常

Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.

翻译：

不建议在没有服务器身份验证的情况下建立 SSL 连接。 根据 MySQL 5.5.45+、5.6.26+ 和 5.7.6+ 的要求，如果未设置显式选项，则默认情况下必须建立 SSL 连接。 为了符合不使用 SSL 的现有应用程序，verifyServerCertificate 属性设置为“false”。 您需要通过设置 useSSL=false 来显式禁用 SSL，或者设置 useSSL=true 并为服务器证书验证提供信任库。

## 原因

MySQL在高版本需要指明是否进行SSL连接

## 解决

在mysql连接字符串url中加入ssl=true或者false即可

url=jdbc:mysql://127.0.0.1:3306/framework?characterEncoding=utf8`&useSSL=false`

## 附加

由于使用阿里云服务器，导致雪崩引发问题`MySql Host is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'`

原因：同一个ip在短时间内产生太多（超过mysql数据库max_connection_errors的最大值）中断的数据库连接而导致的阻塞；

解决：

1、提高允许的max_connection_errors数量（治标不治本）：

　　① 进入Mysql数据库查看max_connection_errors： **show variables like '%max_connection_errors%';**

　  ② 修改max_connection_errors的数量为1000： **set global max_connect_errors = 1000;**

　　③ 查看是否修改成功：**show variables like '%max_connection_errors%';**

2、使用**mysqladmin flush-hosts** 命令清理一下hosts文件（不知道mysqladmin在哪个目录下可以使用命令查找：**whereis mysqladmin**）；

　　① 在查找到的目录下使用命令修改：/usr/bin/**mysqladmin flush-hosts -h192.168.1.1** -P3308 **-uroot** -prootpwd;

　　备注：

　　　　其中端口号，用户名，密码都可以根据需要来添加和修改；

　　　　配置有master/slave主从数据库的要把主库和从库都修改一遍的（我就吃了这个亏明明很容易的几条命令结果折腾了大半天）；

　　　　第二步也可以在数据库中进行，命令如下：**flush hosts;**

**参考黄聪博客园博客，原文链接：http://www.cnblogs.com/huangcong/p/5072915.html**