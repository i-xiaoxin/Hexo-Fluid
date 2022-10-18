---
title: Nginx基础知识
date: 2022-10-17 20:29:02
tags: 
- Nginx
- 负载均衡
index_img: 
banner_img: /img/article3.webp
categories:
- Nginx
comment: waline
---

## Nginx介绍

### 1. 什么是Nginx？

[Nginx](https://nginx.org/) [engine x] 是` Web服务器 `和`反向代理`服务器、邮件代理服务器和通用 TCP/UDP 代理服务器，最初由[Igor Sysoev](http://sysoev.ru/en/)编写。很长一段时间以来，它一直在许多负载重的俄罗斯网站上运行，包括 [Yandex](http://www.yandex.ru/)、 [Mail.Ru](http://mail.ru/)、 [VK](http://vk.com/)和 [Rambler](http://www.rambler.ru/)。基于C语言开发

### 2. 反向代理

正向代理（forward proxy）：是一个位于客户端和目标服务器之间的服务器(代理服务器)，为了从目标服务器取得内容，客户端向代理服务器发送一个请求并指定目标，然后代理服务器向目标服务器转交请求并将获得的内容返回给客户端。用于代理内部网络对internet的连接请求(如VPN/NAT),`客户端`指定代理服务器,并将本来要直接发送给目标Web服务器的HTTP请求先发送到代理服务器上, 然后由代理服务器去访问Web服务器, 并将Web服务器的Response回传给客户端。

所谓的正向代理就是代理服务器**替代访问方【用户】**去访问目标服务器【服务器】

反向代理（Reverse Proxy）方式是指以代理`服务器`来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

所谓的反向代理就是**代替服务器接受用户的请求**，从目标服务器中取得用户的需求资源，然后发送给用户

### 3. 负载均衡

> 数据流量分摊到多个服务器上执行，减轻每台服务器的压力，多台服务器共同完成工作任务，从而提高了数据的吞吐量。

### 4. 动静分离

> 将静态的资源放到反向代理服务器,节省用户的访问时间

### 5. Web服务器

- web应用服务器，如：
  - tomcat 
  - resin
  - jetty
- web服务器，如：
  - Apache 服务器 
  - Nginx
  - IIS  

> 区分：web服务器不能解析jsp等页面，只能处理js、css、html等静态资源。

> 并发：web服务器的并发能力远高于web应用服务器。

## 安装Nginx

### 1. 安装nginx的依赖库

```shell
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

### 2. 解压安装包

```shell
tar -zxvf nginx-1.10.0.tar.gz
```

### 3. 配置nginx安装包

```shell
cd nginx-1.10.0

./configure --prefix=/usr/local/nginx
#  ./configure配置nginx安装到/usr/java/nginx目录下
```

### 4. 编译并安装

```shell
make && make install
```

编译报错1:

```shell
src/core/ngx_murmurhash.c: 在函数‘ngx_murmur_hash2’中:
src/core/ngx_murmurhash.c:37:11: 错误：``this` `statement may fall through [-Werror=implicit-fallthrough=]
```

解决：

```shell
# 进入到nginx-1.10.0目录下(解压目录)
vim objs/Makefile

#找到数据，del去掉-Werror
CFLAGS = -pipe -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g　
```

编译报错2:

```shell
src/os/unix/ngx_user.c: In function ‘ngx_libc_crypt’:
src/os/unix/ngx_user.c:36:7: error: ‘struct crypt_data’ has no member named ‘current_salt’
cd.current_salt[0] = ~salt[0];
```

解决：

```shell
vim src/os/unix/ngx_user.c
# 用/* */注释掉此行
cd.current_salt[0] = ~salt[0];
```

### 5. Nginx的启动及关闭

进入nginx目录下有一个sbin目录，sbin目录下有一个nginx可执行程序，启动即可

```shell
#启动
./nginx
#关闭
./nginx -s stop
#可以不关闭nginx的情况下更新配置文件
./nginx -s reload
```

### 6. Nginx.conf 配置文件介绍

```shell
#user  nobody;
#工作进程
worker_processes  1;

events {
    #连接池连接数
    worker_connections  1024;
}
#请求方式
http {
    #媒体类型
    include       mime.types;
    #默认媒体类型 二进制
    default_type  application/octet-stream;
    #上传文件
    sendfile        on;
    #超时时间
    keepalive_timeout  65;

    #gzip  on;
    #服务器配置
	server {
        #监听端口
        listen       80;
        #监听域名
        server_name  localhost;
        #请求头信息
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #请求映射规则，/代表所有请求路径
        location / {
             #请求转发地址
             #root html;
			 proxy_pass http://manage.powershop.com:8080;
             #欢迎页
             #index  index.html index.htm;
             #转发连接超时时间
			proxy_connect_timeout 600;
             #转发读取超时时间
			proxy_read_timeout 600;
        }
    }
}
```

## 反向代理配置

### 1. 安装tomcat修改端口

先到安装目录(或者解压目录)下找到conf文件夹，在里面找到server.xml的文件

```xml
<!-- 在一台计算机上配置2个tomcat，关键是tomcat里的server.xml文件中三个端口必须不同。需要修改conf/server.xml -->
<!-- 第1⃣️处 port-->
<Connector port="8090" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

<!-- 第2⃣️处 port-->
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

<!-- 第3⃣️处 port-->
<Server port="8005" shutdown="SHUTDOWN">

```

### 2. 修改nginx/conf/nginx.conf文件

```shell
    server{
        listen 80;
        server_name localhost;
        #location代表映射规则，/代表任意url
        location / {
            proxy_pass http://127.0.0.1:8080; 
            # 代理通道只写ip和port，因为url会拼接到ip和port
            proxy_connect_timeout 600;
						proxy_read_timeout 600;
        }

    }
```

location映射配置`proxy_pass`变量

## 负载均衡配置

### 1. 配置两台以上tomcat

### 2. 修改nginx/conf/nginx.conf文件

- 在http节点上添加一个upstream

- 修改location /下的反向代理

```shell
 upstream myTomcats{
        server 127.0.0.1:8080;
        server 127.0.0.1:8090;
    }
    server{
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://myTomcats;
        }
    }
```

### 负载均衡策略

| 负载均衡策略    | 说明           |
| --------------- | -------------- |
| 轮询            | 默认           |
| weight 加权轮询 | 权重方式       |
| ip_hash         | 依据ip分配方式 |
| least_conn      | 按连接数       |
| fair （第三方） | 按响应时间     |
| url_hash        | 依据URL分配    |

##### 轮询

```shell
upstream bck_testing_01 {
  # 默认所有服务器权重为 1
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

##### weight 加权轮询

```shell
upstream bck_testing_01 {
  server 192.168.250.220:8080   weight=3
  server 192.168.250.221:8080              # default weight=1
  server 192.168.250.222:8080              # default weight=1
}
```

##### least_conn

```shell
upstream bck_testing_01 {
  least_conn;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

##### ip_hash

```shell
upstream bck_testing_01 {

  ip_hash;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080

}
```

##### url_hash

```shell
upstream bck_testing_01 {

  hash $request_uri;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080

}
```

具体请参考：[Nginx知多少系列之(七)负载均衡策略](https://www.cnblogs.com/jayjiang/p/12714000.html)

[Nginx 极简教程](https://github.com/dunwu/nginx-tutorial#%E8%BD%AE%E8%AF%A2)

## 动静分离

### 1. 创建静态资源

在服务器上创建静态资源路径放置img

```shell
mkdir /usr/upload/images
```

### 2. 配置nginx.conf

```shell
server{
        listen 80;
        server_name localhost;

        location ~* \.(gif|jpg|png|jpeg)$ {
            root /usr/upload/images;
            #root：转发到目录里
        }
    }
```

### 3. 测试

访问测试地址`http://10.211.55.7/11.jpg`返回静态资源图片

