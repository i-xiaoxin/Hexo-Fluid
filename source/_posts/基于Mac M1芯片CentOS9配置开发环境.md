---
title: 基于Mac M1芯片CentOS9配置开发环境
date: 2022-10-08 21:29:02
tags: 
- Mysql
- MacOS
- M1
index_img: 
banner_img: /img/article9.webp
categories:
- MacOS
comment: waline
---



## 基于Mac M1芯片CentOS9配置Mysql

### 1. 操作环境

> 系统：M1 芯片（arm64）
>
> 虚拟机：CentOS9

M1 支持的CentOS版本问题很多，故使用9；[CentOS Download](https://www.centos.org/download/)

### 2. 下载8.0版本的MySQL yum

```shell
wget http://repo.mysql.com/mysql80-community-release-el9-1.noarch.rpm
```

Red Hat Enterprise Linux 9 / Oracle Linux 9 (Architecture Independent), RPM Package [Mysql yum Download](https://dev.mysql.com/downloads/repo/yum/)

linux9  arm 目前没找到Mysql 5.7.*版本QAQ

### 3. 安装源

```shell
rpm -Uvh mysql80-community-release-el9-1.noarch.rpm
```

### 4. 安装MySQL服务端

```shell
yum install -y mysql-community-server
```

### 5. 启动MySQL

```shell
systemctl start mysqld.service
```

### 6. 检查是否启动成功

```shell
systemctl status mysqld.service
```

### 7. 获取临时密码

```shell
grep 'temporary password' /var/log/mysqld.log 
```

### 8. 登陆Mysql

```shell
mysql -uroot -p
```

### 9. 设置强密码

```shell
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'Root_12root';
```

### 10. Mysql 密码策略强度降低设置

```shell
#查看密码策略
SHOW VARIABLES LIKE 'validate_password%';

#密码强度改为LOW
set global validate_password.policy=0;
#密码长度改为至少4
set global validate_password.length=4;
```

关于 mysql 密码策略相关参数：

- validate_password.length  固定密码的总长度

- validate_password.dictionary_file 指定密码验证的文件路径；

- validate_password.mixed.case_count  整个密码中至少要包含大/小写字母的总个数；

- validate_password.number_count  整个密码中至少要包含阿拉伯数字的个数；

- validate_password_special_char_count 整个密码中至少要包含特殊字符的个数；

- validate_password.policy 指定密码的强度验证等级，默认为 MEDIUM；


关于 validate_password.policy 的取值：

- 0/LOW：只验证长度；
- 1/MEDIUM：验证长度、数字、大小写、特殊字符；
-   2/STRONG：验证长度、数字、大小写、特殊字符、字典文件；

### 11. 修改自定义密码

```shell
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

## 其他

### 1. 无法远程连接

```shell
#查询防火墙状态
systemctl status firewalld
# 关闭防火墙
systemctl stop firewalld
```

### 2. Mysql设置远程访问

```shell
# 修改host值（以“%”通配符，增加在主机/ip地址）
update user set host='%' where user ='root';
# 刷新mysql系统权限相关表
FLUSH PRIVILEGES;
# 授权
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
```

### 3. 防火墙开放3306端口（不关防火墙处理）

```shell
firewall-cmd --state
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

### 4. 卸载MySQL仓库（yum）

```shell
# 查询
rpm -qa | grep mysql
# 删除
yum -y remove 查出哪个删除哪个
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

  

