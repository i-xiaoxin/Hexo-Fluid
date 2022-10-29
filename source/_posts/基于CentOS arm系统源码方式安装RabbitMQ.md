---
title: 基于CentOS arm系统源码方式安装RabbitMQ
date: 2022-10-19 21:29:02
tags: 
- RabbitMQ
- CentOS
- Erlang
index_img: 
banner_img: /img/article1.webp
categories:
- RabbitMQ
comment: waline
---

# 基于CentOS arm系统源码方式安装RabbitMQ

Rabbit技术公司基于AMQP标准开发的RabbitMQ1.0发布。RabbitMQ采用Erlang 语言开发。

本次环境为Mac M1虚拟机CentOS9 arm

## 源码安装Erlang

[Erlang官网地址](https://www.erlang-solutions.com/downloads/)

Erlang源码[Github Dowanload](https://github.com/erlang/otp/releases)

```sh
#yum安装erlang编译所依赖的环境 
yum -y install make gcc gcc-c++ kernel-devel m4ncurses-devel openssl-devel

# Erlang 源码下载
wget https://github.com/erlang/otp/releases/download/OTP-25.1.2/otp_src_25.1.2.tar.gz

tar -zxvf otp_src_25.1.2.tar.gz
cd otp_src_25.1.2

# 预编译到/usr/local/erlang
./configure  --prefix=/usr/local/erlang --without-javac
# 安装
make && make install
```

## 安装RabbitMQ

RabbitMQ官网[下载地址](http://www.rabbitmq.com/download.html)

源码Github[下载地址](https://github.com/rabbitmq/rabbitmq-server/releases)

RabbitMQ 和 Erlang/OTP [兼容性对照表](https://www.rabbitmq.com/which-erlang.html#compatibility-matrix)安装前确认一下！

```sh
# rabbitmq-server 源码下载
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.10.8/rabbitmq-server-generic-unix-3.10.8.tar.xz

# 解压
xz -d rabbitmq-server-generic-unix-3.10.8.tar.xz
# 注意没有z参数；下载的文件并不是“通过 gzip 过滤归档”所以添加参数z就无法正常解压
tar -xvf rabbitmq-server-generic-unix-3.10.8.tar

# 移动文件到系统目录下
mv rabbitmq-server-generic-unix-3.10.8/ /usr/local/rabbitmq
```

## 配置系统环境变量

```sh
# 打开环境变量文件
vim /etc/profile

# 添加erlang rabbitmq 到环境变量
export PATH=$PATH:/usr/local/erlang/bin
export PATH=$PATH:/usr/local/rabbitmq/sbin

# 重新加载环境变量
source /etc/profile
# 测试Erlang环境生效
erl 
或 erl -v
# 输出内容如下及成功
Erlang/OTP 25 [erts-13.1.2] [source] [64-bit] [smp:2:2] [ds:2:2:10] [async-threads:1] [jit]

Eshell V13.1.2  (abort with ^G)
1>
```

## RabbitMQ相关命令

```sh
# 开启web界面管理工具
rabbitmq-plugins enable rabbitmq_management

# 启动 可以省略start
rabbitmq-server start
# 后台启动,两个命令建议第二个
rabbitmq-server -detached
rabbitmqctl start_app
# 停止
rabbitmq-server stop
# 节点启动后使用这个停止
rabbitmqctl  stop_app
```

## 添加远端登录账户

```sh
# 创建账号
rabbitmqctl  add_user admin 1111
# 设置用户角色
rabbitmqctl  set_user_tags admin  administrator
# 设置用户权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
# 查看当前用户和角色(需要开启服务)
rabbitmqctl list_users
```

账号guest具有所有的操作权限，并且又是默认账号，出于安全因素的考虑，guest用户只能通过localhost登陆

## 测试

 浏览器输入：serverip:15672。其中serverip是RabbitMQ-Server所在主机的ip，15672是RabbitMQ-Server的端口号

## 关于rpm安装方式

使用rpm安装，yum install erlang 与 yum insatll rabbitmq-server提示匹配不到命令，关于CentOS9 arm的没有解决

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