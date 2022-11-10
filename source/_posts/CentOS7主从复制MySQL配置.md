---
title: CentOS7主从复制MySQL5安装配置
date: 2022-11-9 20:29:02
tags: 
- MySQL
- MySQL5.6
- MySQL5.7
- CentOS7
index_img: 
banner_img: /img/article3.webp
categories:
- MySQL
comment: waline
---

# 卸载MySQL

 清理出来一个干净的环境才方便下一步计划执行

```sh
#查看已安装：
rpm -qa | grep mariadb
rpm -qa | grep mysql
#卸载查出来的数据库
rpm -e --nodeps mariadb-libs-5.5.65-1.el7.x86_64
#再次查看：
rpm -qa | grep mysql
```

# 一、安装MySQL 5.7

### 1、下载YUM库

```sh
wget  http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

[MySQL Community Downloads](https://dev.mysql.com/downloads/)可根据需要进行下载

### 2、安装YUM库

```sh
rpm -ivh mysql57-community-release-el7-10.noarch.rpm

# 或者使用以下命令
yum install mysql57-community-release-el7-10.noarch.rpm
```

### 3、安装MySQL RPM包

```sh
# -y 自动确认
# --nogpgcheck 不匹配公钥安装
yum -y install mysql-community-server  --nogpgcheck
```

### 4、测试MySQL

```sh
# 启动mysql服务
systemctl start mysqld
# 停止
systemctl stop mysqld
# 重启
systemctl restart mysqld
# 查看运行状态
systemctl status mysqld
# 开机自启动
systemctl enable mysqld
```

### 5、找出MySQL登录密码

```sh
grep "password" /var/log/mysqld.log
```

查询出来密码复制，登录备用

### 6、登录MySQL数据库

```sh
mysql -u root -p
```

### 7、修改密码策略

```sh 
# 密码安全等级
set global validate_password_policy=low;
# 密码长度
set global validate_password_length=4;
```

[MySQL密码校验策略](https://codeantenna.com/a/O3KEfh3LXY)

### 8、修改密码

```mysql
# 设置root用户密码1111
ALTER USER 'root'@'localhost' IDENTIFIED BY '1111';
```

###  9、MySQL开启远程连接

```mysql
 # 选择要使用的数据库
 use mysql;
 # 设置远程连接
 update user set host ="%" where user = "root";
 # 或者使用；password为远程登录密码
 GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
 # 刷新刚才修改的权限，使其生效。
 flush privileges;
```

# 二、MySQL主从复制

#### 1、mysql主(master)从(slave)简介

主从复制是指一台服务器充当主数据库服务器，另一台或多台服务器充当从数据库服务器，主服务器中的数据自动复制到从服务器之中。对于多级复制，数据库服务器即可充当主机，也可充当从机。MySQL主从复制的基础是主服务器对数据库修改记录二进制日志，从服务器通过主服务器的二进制日志自动执行更新。

<img src="http://s1.51cto.com/wyfs02/M00/7D/37/wKioL1bijwHhWDO_AACki5mAlXk627.jpg" style="zoom:50%;" >

主从备份要素：
       1. 开启主数据库日志功能log_bin
       2. 每个数据库需要有一个 server_id,主 server_id 值小于从server_id(标识从哪server写入的)
       3. 每个 mysql 都有一个 uuid,由于虚拟机直接进行克隆,需要修改uuid 的值(唯一识别码)

### 2、配置步骤

#### 主(master)数据库配置

1、开启日志log_bin并配置server_id

```sh
 vim /etc/my.cnf
 
# 文件中增加以下两行 
log_bin=master_log   
service_id=1
```

2、重启mysql并查看日志是否开启

```sh
# 重启mysql
systemctl restart mysqld
# 登录
mysql -u root -p
# 查看日志 
show master status;
```

#### 从(slave)数据库配置

1、配置server_id

```sh
 vim /etc/my.cnf
 
# 文件中增加以下；主 server_id 值小于从server_id
service_id=2
```

2、修改uuid

```sh
vim /var/lib/mysql/auto.cnf
# 修改为与主数据库不重复即可，本次使用的是虚拟机克隆，导致uuid一样
 server-uuid=xxxxxxx5
```

3、修改slave状态，执行同步命令

```mysql
# 登录mysql
mysql -u root -p
# 停止同步命令
stop slave;
# 执行同步命令，设置主服务器ip，同步账号密码，同步位置
change master to 
master_host='192.168.204.138',master_user='root',master_password='1111',master_log_file='master_log.000001';
# 开启同步功能
start slave;
```

4、查看slave状态

```mysql
# \G 格式化文本
show slave status \G;
```

输出内容

```mysql
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.204.138
                  Master_User: account
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master_log.000001
          Read_Master_Log_Pos: 337523
               Relay_Log_File: db2-relay-bin.000002
                Relay_Log_Pos: 337686
        Relay_Master_Log_File: master_log.000001
             Slave_IO_Running: Yes  #已开启
            Slave_SQL_Running: Yes  #已开启
              Replicate_Do_DB:
          Replicate_Ignore_DB:
          ...
```

5、验证主从关系

在主数据库中新建数据库,新建表,添加数据,观察从数据库是否同步新增



参考：

[MySQL 主从复制原理不再难](https://www.cnblogs.com/rickiyang/p/13856388.html)

[MySQL 主从复制部署配置.md](https://github.com/daydreamdev/MeetingFilm/blob/master/note/MySQL%20%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E9%83%A8%E7%BD%B2%E9%85%8D%E7%BD%AE.md)

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