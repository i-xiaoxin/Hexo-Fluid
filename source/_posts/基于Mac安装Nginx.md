---
title: 基于Mac配置安装Nginx
date: 2022-10-18 21:29:02
tags: 
- Mac
- MacOS
- M1
- Nginx
index_img: 
banner_img: /img/article3.webp
categories:
- MacOS
comment: waline
---

## 前提

> Mac需要安装Homebrew

## 配置

### 安装

使用brew安装nginx

```shell
brew install nginx
```

查看nginx的配置信息

```shell
brew info nginx
```

输出内容如下

```sh
nginx: stable 1.23.1 (bottled), HEAD
HTTP(S) server and reverse proxy, and IMAP/POP3 proxy server
https://nginx.org/
# 以下是安装nginx路径
/opt/homebrew/Cellar/nginx/1.23.1 (26 files, 2.2MB) *
  Poured from bottle on 2022-10-18 at 18:00:26
# 安装源
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/nginx.rb
License: BSD-2-Clause
==> Dependencies
Required: openssl@1.1 ✔, pcre2 ✔
==> Options
--HEAD
	Install HEAD version
==> Caveats
# 根目录
Docroot is: /opt/homebrew/var/www
# 配置路径
The default port has been set in /opt/homebrew/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.
# 启动路径
nginx will load all files in /opt/homebrew/etc/nginx/servers/.

To restart nginx after an upgrade:
  brew services restart nginx
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/nginx/bin/nginx -g daemon off;
==> Analytics
install: 29,315 (30 days), 115,535 (90 days), 454,216 (365 days)
install-on-request: 29,264 (30 days), 115,323 (90 days), 453,381 (365 days)
build-error: 3 (30 days)
```

### brew相关命令

```sh
# 启动nginx服务
brew services start nginx 
# 重启命令
brew services restart nginx
# 关闭命令
brew services stop nginx

# 关闭命令提示，不推荐使用
Stopping `nginx`... (might take a while)
==> Successfully stopped `nginx` (label: homebrew.mxcl.nginx)

```

### 检验nginx配置文件

```sh
# 命令
sudo /opt/homebrew/Cellar/nginx/1.23.1/bin/nginx -t -c /opt/homebrew/etc/nginx/nginx.conf

# 输出信息
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful

```

### 通过进程关闭Nginx

```sh
# 获取到nginx的进程
ps -ef|grep nginx

# 输出内容
 501 74155     1   0  6:05下午 ??         0:00.00 nginx: master process /opt/homebrew/Cellar/nginx/1.23.1/bin/nginx
  501 74156 74155   0  6:05下午 ??         0:00.06 nginx: worker process
  501 77226 72927   0  6:41下午 ttys002    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox nginx
  
# 进程结束命令
kill -term 74155

# 拓展
kill -QUIT 72 (从容的停止，即不会立刻停止)

Kill -TERM 72 （立刻停止）

Kill -INT 72 （和上面一样，也是立刻停止）
```

## 解决异常错误

### Error: Failure while executing;

起因：

通过brew 启动nginx时报错

```sh
 # 键入命令
 brew services start nginx
```

报错内容：

```sh
Bootstrap failed: 5: Input/output error
Try re-running the command as root for richer errors.
Error: Failure while executing; `/bin/launchctl bootstrap gui/501 /Users/heminxin/Library/LaunchAgents/homebrew.mxcl.nginx.plist` exited with 5.
```

解决：

直接进入nginx启动脚本目录通过执行nginx脚本启动；也可以使用nginx命令启动

### Error：在访问被代理url部分js时提示【net::ERR_CONTENT_LENGTH_MISMATCH 200 (OK)】

错误：

当访问url链接时，发现内容不能够加载出来，空白页面，打开控制台开发者工具发现js访问200报错net::ERR_CONTENT_LENGTH_MISMATCH 200 (OK)；查看nginx日志会发现提示权限不够，给proxy_temp文件赋权即可

解决：

```sh
# 进入nginx运行目录找proxy_temp
cd /opt/homebrew/var/run/nginx
# 赋权
sudo chmod 777 proxy_temp
```

再次访问即可打开完整页面

## Nginx相关命令

```sh
nginx -s reopen #重启Nginx

nginx -s reload #重新加载Nginx配置文件，然后以优雅的方式重启Nginx

nginx -s stop #强制停止Nginx服务

killall nginx #杀死所有nginx进程  

nginx -s quit #优雅地停止Nginx服务（即处理完所有请求后再停止服务）

nginx -t #检测配置文件是否有语法错误，然后退出

nginx -v #显示版本信息并退出

nginx -V #显示版本和配置选项信息，然后退出

nginx -t #检测配置文件是否有语法错误，然后退出

nginx -T #检测配置文件是否有语法错误，转储并退出

nginx -q #在检测配置文件期间屏蔽非错误信息

nginx -?,-h #打开帮助信息  

nginx -p prefix #设置前缀路径(默认是:/usr/share/nginx/)

nginx -c filename #设置配置文件(默认是:/etc/nginx/nginx.conf)

nginx -g directives #设置配置文件外的全局指令

```

可以通过以上方式进行相关设置

推荐Nginx开源社区🔗[链接](https://www.nginx.org.cn/)

<div>
    <script src="//cdn.jsdelivr.net/npm/@waline/client"></script>
<script src="//cdn.jsdelivr.net/npm/@waline/client"></script>  
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>
