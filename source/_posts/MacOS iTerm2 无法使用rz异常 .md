---
title: MacOS iTerm2 无法使用rz异常 
date: 2022-10-15 21:29:02
tags: 
- Mac
- iTerm2
index_img: 
banner_img: /img/article8.webp
categories:
- Mac
comment: waline
---

## 问题

运行rz会报类似错：rz会出现?z waiting to receive.**B0100000023be50

## 原因

- Mac OS未安装lrzsz
- mac上的终端不支持rz和sz，需要在iterm2配置一下

## 解决

### 1. Mac安装rz

注意：需要提前安装brew，此处不在赘述

```sh
brew install lrzsz
```

### 2. 配置iTerm2 sh脚本

#### 2.1 在Mac目录/usr/local/bin下安装两个脚本

官方提供[脚本](https://github.com/Keystion/iterm2-zmodem)

- iterm2-send-zmodem.sh
- iterm2-recv-zmodem.sh

```sh
sudo wget https://raw.github.com/Keystion/iterm2-zmodem/master/iterm2-send-zmodem.sh
sudo wget https://raw.github.com/Keystion/iterm2-zmodem/master/iterm2-recv-zmodem.sh
sudo chmod 777 /usr/local/bin/iterm2-*	
```

注意网络影响

#### 2.2 iTerm2配置

在iTerm2中配置选项

点击preferences → profiles，选择某个profile，如Default，之后继续选择advanced → triggers，添加编辑添加如下triggers

需要添加两个内容如下：

```shell
#triggers1：
Regular expression: /*/*B0100
Action: Run Silent Coprocess
Parameters:/usr/local/bin/iterm2-send-zmodem.sh
 
#triggers2：
Regular expression: /*/*B00000000000000
Action: Run Silent Coprocess
Parameters:/usr/local/bin/iterm2-recv-zmodem.sh
```

配置完成最好重启一下Mac💻



参考：[macOS iTerm2无法使用rz并提示waiting-to-receive. **B0100000023be50. _](https://webclown.net/2020/06/19/macOS-iTerm2%E6%97%A0%E6%B3%95%E4%BD%BF%E7%94%A8rz%E5%B9%B6%E6%8F%90%E7%A4%BAwaiting-to-receive%E3%80%90%E8%BD%AC%E3%80%91/)

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