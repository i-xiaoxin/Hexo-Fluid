---
title: CentOS安装使用FastDFS
date: 2022-10-23 20:29:02
tags: 
- 微服务
- Spring Cloud Alibaba
- FastDFS
index_img: 
banner_img: /img/article4.webp
categories:
- 微服务
- Spring Cloud Alibaba
comment: waline
---

# CentOS安装使用单节点FastDFS

## FastDFS文件准备

```sh
# libfastcommon是从 FastDFS 和 FastDHT 中提取出来的公共 C 函数库
git clone https://github.com/happyfish100/libfastcommon.git --depth 1
# fastdfs
git clone https://github.com/happyfish100/fastdfs.git --depth 1
# fastdfs-nginx-module插件
git clone https://github.com/happyfish100/fastdfs-nginx-module.git --depth 1
# nginx
wget http://nginx.org/download/nginx-1.15.4.tar.gz #下载nginx压缩包
```

# 1. 安装libfastcommon-master

## 1.1 安装gcc

GCC用来对C语言代码进行编译运行，使用yum命令安装：

```shell
yum -y install gcc
```

## 1.2 编译安装

```sh
#进入解压完成的目录
cd libfastcommon-master

#编译并且安装：
./make.sh 
./make.sh install
```

# 2. 安装fastdfs

```sh
tar -zxvf FastDFS_v5.08.tar.gz
#进入解压完成的目录
cd FastDFS
# 编译
./make.sh 
# 安装
./make.sh install
```

如果安装成功，会看到/etc/init.d/下看到提供的脚本文件：

```shell
ll /etc/init.d/ | grep fdfs 
```

- `fdfs_trackerd` 是tracker启动脚本
- `fdfs_storaged` 是storage启动脚本

能够在 /etc/fdfs/ 目录下看到默认的配置文件模板：

```shell
ll /etc/fdfs/
```

- `tarcker.conf.sample` 是tracker的配置文件模板
- `storage.conf.sample` 是storage的配置文件模板
- `client.conf.sample` 是客户端的配置文件模板

## 2.1 配置并启动tracker服务

FastDFS的tracker和storage在刚刚的安装过程中，都已经被安装了，因此我们安装这两种角色的方式是一样的。不同的是，两种需要不同的配置文件。先配置tracker服务

```sh
# 将配置模板文件复制
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
# 修改复制后的配置文件
vim /etc/fdfs/tracker.conf 

# 存储日志和数据的根目录（生产环境中会变得臃肿）
base_path=/powershop/tracker   

# 新增配置文件
mkdir -p /powershop/tracker

# 启动tracker服务器
/etc/init.d/fdfs_trackerd start
# 停止tracker服务器
/etc/init.d/fdfs_trackerd stop
# 启动fdfs_trackerd服务，停止用stop
service fdfs_trackerd start 
```

## 2.2 配置并启动storage服务

```sh
# 将模板文件复制
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
# 修改复制后的配置文件
vim /etc/fdfs/storage.conf

# 日志文件存储根目录
base_path=/powershop/storage 
# 文件存储目录
store_path0=/powershop/storage            
# 文件存储目录
tracker_server=192.168.204.131:22122    

# 新建目录
mkdir -p /powershop/storage
# 启动storage服务器
/etc/init.d/fdfs_storaged start
# 停止storage服务器
/etc/init.d/fdfs_storaged stop
# 启动fdfs_storaged服务，停止用stop
service fdfs_storaged start  
```

检查FastDFS Tracker Server是否启动成功：

```sh
ps -ef | grep fdfs
```

# 3. nginx访问FastDFS

## 3.1 为什么需要用Nginx访问？

FastDFS通过Tracker服务器,将文件放在Storage服务器存储，但是同组存储服务器之间需要进入文件复制，有同步延迟的问题。

假设Tracker服务器将文件上传到了192.168.4.125，上传成功后文件ID已经返回给客户端。此时FastDFS存储集群机制会将这个文件同步到同组存储192.168.4.126，在文件还没有复制完成的情况下，客户端如果用这个文件ID在192.168.4.126上取文件,就会出现文件无法访问的错误。

而fastdfs-nginx-module可以重定向文件连接到文件上传时的源服务器取文件,避免客户端由于复制延迟导致的文件无法访问错误

## 3.2 安装fastdfs-nginx-module插件

```sh
cd fastdfs-nginx-module/src/
# 编辑config
vim config
# 使用以下底行命令：将所有的/usr/local替换为 /usr
:%s+/usr/local/+/usr/+g
```

## 3.3 配置fastdfs-nginx-module与FastDFS关联配置文件

```sh
cp /usr/upload/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/

vim /etc/fdfs/mod_fastdfs.conf
# 客户端访问文件连接超时时长（单位：秒）
connect_timeout=10                       
 # tracker服务IP和端口
tracker_server=192.168.204.131:22122   
# 访问链接前缀加上组名
url_have_group_name=true                
# 访问链接前缀加上组名
store_path0=/powershop/storage  


# 复制 FastDFS 的部分配置文件到/etc/fdfs 目录，否则不支持nginx
cd /usr/upload/FastDFS/conf/
cp http.conf mime.types /etc/fdfs/
```

# 4. 安装Nginx

### 4.1 如果没有安装过nginx

- 安装nginx的依赖库


```shell
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel 
```

- 配置nginx安装包，并指定fastdfs-nginx-model

```shell
cd nginx-1.10.0
# 将fastdfs-nginx-moudle源码作为模块编译进去
./configure --prefix=/usr/local/nginx --add-module=/usr/upload/fastdfs-nginx-module/src
```

- 编译并安装

```shell
make && make install
```

### 4.2 如果已经安装过nginx

1） 进入nginx目录：

```shell
cd /usr/upload/nginx-1.10.0/
```

2） 配置FastDFS 模块

```shell
./configure --prefix=/usr/local/nginx --add-module=/usr/upload/fastdfs-nginx-module/src
```

注意：这次配置时，要添加fastdfs-nginx-moudle模块 

3） 编译，注意，这次不要安装（install）

```shell
make
```

 4） 替换nginx二进制文件:

备份：

```shell
mv /usr/bin/nginx /usr/bin/nginx-bak
```

用新编译的nginx启动文件替代原来的：

```shell
cp objs/nginx /usr/bin/
```

### 4.3 启动nginx

配置nginx整合fastdfs-module模块

我们需要修改nginx配置文件：

```shell
vim  /usr/java/nginx/conf/nginx.conf
```

将文件中，原来的`server 80{ ...}` 部分代码替换为如下代码：

```nginx
    server {
        listen       80;
        server_name  image.shop.com;

    	# 监听域名中带有group的，交给FastDFS模块处理
        location ~/group([0-9])/ {
            ngx_fastdfs_module;
      #获得fastdfs中图片的存在路径  /usr/storage/group/0/atm.jpg
        }
    }
```

启动nginx：

```shell
./nginx	# 启动nginx

./nginx -s stop	# 停止nginx

./nginx -s reload	# 重新载入配置文件
```

\# 可通过`ps -ef | grep nginx`查看nginx是否已启动成功    

<div>
    <hr>
    <script src="//cdn.jsdelivr.net/npm/@waline/client"></script>
<script src="//cdn.jsdelivr.net/npm/@waline/client"></script>  
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>