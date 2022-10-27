---
title: 基于CentOS系统Redis两种安装方式
date: 2022-10-17 22:29:02
tags: 
- Redis
- CentOS
- Yum
index_img: 
banner_img: /img/article6.webp
categories:
- Redis
comment: waline
---

## 方式一（yum安装）

- 检查是否有redis yum 源

```shell
yum install redis
```

- 下载fedora的epel仓库

```shell
yum install epel-release
```

- 安装redis数据库

```shell
yum install redis
```

- 安装完毕后，使用下面的命令启动redis服务

```shell
# 启动redis
service redis start
# 停止redis
service redis stop
# 查看redis运行状态
service redis status
# 查看redis进程
ps -ef | grep redis
```

- 设置redis为开机自动启动

```shell
chkconfig redis on
```

- 进入redis服务

~~~shell
# 进入本机redis
redis-cli
# 列出所有key
keys *
``
- 防火墙开放相应端口
```bash
# 开启6379
/sbin/iptables -I INPUT -p tcp --dport 6379 -j ACCEPT
# 开启6380
/sbin/iptables -I INPUT -p tcp --dport 6380 -j ACCEPT
# 保存
/etc/rc.d/init.d/iptables save
# centos 7下执行
service iptables save
``

### 修改redis默认端口和密码
- 打开配置文件
```bash
vi /etc/redis.conf
~~~

- 修改默认端口，查找 port 6379 修改为相应端口即可

- 修改默认密码，查找 requirepass foobared 将 foobared 修改为你的密码 

-  使用配置文件启动 redis 

```shell
redis-server /etc/redis.conf
```

- 使用端口登录

```shell
redis-cli -h 127.0.0.1 -p 6179
```

-  此时再输入命令则会报错 

-  输入刚才输入的密码 

```shell
auth 111
```

- 关闭redis

```shell
redis-cli -h 127.0.0.1 -p 6179
shutdown
```

进程号杀掉redis

```shell
ps -ef | grep redis
kill -9 XXX
```

以上参考链接：[centos7安装Redis教程](https://www.jianshu.com/p/b93c7f261637)

## 方式二（源码编译安装）

- 源码下载[GitHub Download](https://github.com/redis/redis/archive/refs/tags/7.0.5.tar.gz)

```shell
wget https://github.com/redis/redis/archive/refs/tags/7.0.5.tar.gz
```

- 解压

```shell
tar -zxvf redis-6.0.16.tar.gz
```
- 安装c编译

```shell
#因为Redis底层是C语言写的，需要安装gcc进行编译
yum -y install gcc-c++
```
- 预编译

```shell
cd redis-6.0.16/
make
```


- 安装指定路径

```shell
make prefix=/usr/local/redis/ install
```

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
