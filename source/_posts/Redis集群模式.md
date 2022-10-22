---
title: Rredis集群模式测试
date: 2022-10-21 22:09:02
tags: 
- Redis Cluster
- Redis
index_img: 
banner_img: /img/article2.webp
categories:
- Redis
comment: waline
---

## 主从复制

### 1.什么是主从复制

​         持久化保证了即使redis服务重启也会丢失数据，因为redis服务重启后会将硬盘上持久化的数据恢复到内存中，但是当redis服务器的硬盘损坏了可能会导致数据丢失，如果通过redis的主从复制机制就可以避免这种单点故障，	

说明：

- 主redis中的数据有两个副本（replication）即从redis1和从redis2，即使一台redis服务器宕机其它两台redis服务也可以继续提供服务。

- 主redis中的数据和从redis上的数据保持实时同步，当主redis写入数据时通过主从复制机制会复制到两个从redis服务上。

- 只有一个主redis，可以有多个从redis。

- 主从复制不会阻塞master，在同步数据时，master可以继续处理client 请求

- 一个redis可以即是主又是从，	


### 2.主从配置

#### 2.1.主redis配置

无需特殊配置。

#### 2.2.从redis配置

修改从redis服务器上的redis.conf文件，添加**slaveof** **主redis ip**  **主redis**端口    	                    

配置当前该从redis服务器所对应的主redis是192.168.101.3，端口是6379

### 3.主从复制过程	

复制过程说明：

1、  slave 服务启动，slave 会建立和master 的连接，发送sync 命令。

2、master启动一个后台进程将数据库快照保存到RDB文件中

3、master 就发送RDB文件给slave

4、slave 将文件保存到磁盘上，然后加载到内存恢复

5、master把缓存的命令转发给slave

注意：主死了，从只能读 

### 4.配置主从赋值

1、拷贝redis

```sh
cd /usr/local
cp -r redis redis-6380
```

2、修改redis-6380的redis.conf

3、分别开启主从redis，并在主redis存入数据，测试效果

​         效果：主从数据库数据一致

4、关闭主redis，使用从redis存入数据		

# redis集群的搭建

## 1.redis-cluster架构图

 <div>
   <src="https://segmentfault.com/img/remote/1460000022808584"></src>
 </div>                         	

架构细节:

(1)<font color=red>所有的redis节点彼此互联(PING-PONG机制)</font>，节点的fail是通过集群中超过半数的节点检测失效时才生效. 

(2)<font color=red>存取数据时连接任一节点都可以</font>，但集群中有一个节点fail整个集群都会fail

```
Redis 集群中内置了 16384 个哈希槽，当需要在Redis 集群中放置一个 key-value 时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点	
```

## 2.redis集群的搭建

> Redis集群中至少应该有三个节点。要保证集群的高可用，需要每个节点有一个备份机。
>
> Redis集群至少需要6台服务器。
>
> 搭建伪分布式。可以使用一台虚拟机运行6个redis实例。需要修改redis的端口号7001-7006

### 2.1.集群搭建环境

使用ruby脚本搭建集群,需要安装ruby。

```sh
[root@upload ~]# yum install ruby
[root@upload ~]# yum install rubygems
[root@upload ~]# gem install redis-3.0.0.gem 

Successfully installed redis-3.0.0

1 gem installed

Installing ri documentation for redis-3.0.0...

Installing RDoc documentation for redis-3.0.0...

[root@localhost ~]# cd redis-3.0.0/src
[root@localhost src]# ll *.rb
-rwxrwxr-x. 1 root root 48141 Apr  1  2015 redis-trib.rb
```

### 2.2.搭建步骤

<font color=red>注意：必须删除dump.rdb和appendonly.aof文件</font>

> 搭建伪分布式，需要6个redis实例放到/usr/local/redis-cluster目录下，并且运行在不同的端口7001-7006
>
> ```sh
> cp -r  /usr/local/redis  /usr/local/redis-cluster/redis-7001
> ```

#### 2.2.1.创建6个redis实例

每个实例运行在不同的端口。还需要修改redis.conf配置文件。配置文件中还需要把cluster-enabled yes前的注释去掉。	

#### 2.2.2.启动每个redis实例

创建启动集群的脚本：start-all.sh 放在/usr/java/redis-cluster目录下。

```sh
cd redis-7001
./bin/redis-server bin/redis.conf
cd ..
cd redis-7002
./bin/redis-server bin/redis.conf
cd ..
cd redis-7003
./bin/redis-server bin/redis.conf
cd ..
cd redis-7004
./bin/redis-server bin/redis.conf
cd ..
cd redis-7005
./bin/redis-server bin/redis.conf
cd ..
cd redis-7006
./bin/redis-server bin/redis.conf
cd .. 
[root@localhost redis-cluster]# chmod 777 start-all.sh
```

创建关闭集群的脚本：shutdown-all.sh，放在/usr/local/redis-cluster目录下。

```
cd redis-7001
./redis7001/ bin/redis-cli -p 7001 shutdown
./redis7001/ bin/redis-cli -p 7002 shutdown
./redis7001/ bin/redis-cli -p 7003 shutdown
./redis7001/ bin/redis-cli -p 7004 shutdown
./redis7001/ bin/redis-cli -p 7005 shutdown
./redis7001/ bin/redis-cli -p 7006 shutdown
[root@localhost redis-cluster]# chmod 777shutdown-all.sh
```

#### 2.2.3.使用ruby搭建集群

**切换到\*.rb目录**

```sh
[root@localhost src]# ./redis-trib.rb create --replicas 1 192.168.25.153:7001 192.168.25.153:7002 192.168.25.153:7003 192.168.25.153:7004 192.168.25.153:7005  192.168.25.153:7006
>>> Creating cluster
Connecting to node 192.168.25.153:7001: OK
Connecting to node 192.168.25.153:7002: OK
Connecting to node 192.168.25.153:7003: OK
Connecting to node 192.168.25.153:7004: OK
Connecting to node 192.168.25.153:7005: OK
Connecting to node 192.168.25.153:7006: OK
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.25.153:7001
192.168.25.153:7002
192.168.25.153:7003
Adding replica 192.168.25.153:7004 to 192.168.25.153:7001
Adding replica 192.168.25.153:7005 to 192.168.25.153:7002
Adding replica 192.168.25.153:7006 to 192.168.25.153:7003
M: 2e48ae301e9c32b04a7d4d92e15e98e78de8c1f3 192.168.25.153:7001
   slots:0-5460 (5461 slots) master
M: 8cd93a9a943b4ef851af6a03edd699a6061ace01 192.168.25.153:7002
   slots:5461-10922 (5462 slots) master
M: 2935007902d83f20b1253d7f43dae32aab9744e6 192.168.25.153:7003
   slots:10923-16383 (5461 slots) master
S: 74f9d9706f848471583929fc8bbde3c8e99e211b 192.168.25.153:7004
   replicates 2e48ae301e9c32b04a7d4d92e15e98e78de8c1f3
S: 42cc9e25ebb19dda92591364c1df4b3a518b795b 192.168.25.153:7005
   replicates 8cd93a9a943b4ef851af6a03edd699a6061ace01
S: 8b1b11d509d29659c2831e7a9f6469c060dfcd39 192.168.25.153:7006
   replicates 2935007902d83f20b1253d7f43dae32aab9744e6
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.....
>>> Performing Cluster Check (using node 192.168.25.153:7001)
M: 2e48ae301e9c32b04a7d4d92e15e98e78de8c1f3 192.168.25.153:7001
   slots:0-5460 (5461 slots) master
M: 8cd93a9a943b4ef851af6a03edd699a6061ace01 192.168.25.153:7002
   slots:5461-10922 (5462 slots) master
M: 2935007902d83f20b1253d7f43dae32aab9744e6 192.168.25.153:7003
   slots:10923-16383 (5461 slots) master
M: 74f9d9706f848471583929fc8bbde3c8e99e211b 192.168.25.153:7004
   slots: (0 slots) master
   replicates 2e48ae301e9c32b04a7d4d92e15e98e78de8c1f3
M: 42cc9e25ebb19dda92591364c1df4b3a518b795b 192.168.25.153:7005
   slots: (0 slots) master
   replicates 8cd93a9a943b4ef851af6a03edd699a6061ace01
M: 8b1b11d509d29659c2831e7a9f6469c060dfcd39 192.168.25.153:7006
   slots: (0 slots) master
   replicates 2935007902d83f20b1253d7f43dae32aab9744e6
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@localhost redis-cluster]# 
```

#### 2.2.4.测试

启动时使用-c参数来启动集群模式，命令如下：

```sh
./redis-cli -c -p 7001	
```

#### 2.2.5. redis cluster命令

```sh
cluster info   #打印集群的信息

cluster nodes  #列出集群当前已知的所有节点(node)，以及这些节点的相关信息  
```

# Redis cluster 官方工具搭建

## 1.配置Ruby环境

```sh
yum install ruby
yum install rubygems
gem install redis-3.0.0.gem #不推荐安装包
gem install redis # 推荐在线安装
```

## 2.创建集群

```sh
[root@localhost upload]# cd /usr/upload/redis-3.0.0/utils/
[root@localhost utils]# ll
总用量 52
-rw-rw-r--. 1 root root  593  4月  1  2015 build-static-symbols.tcl
-rw-rw-r--. 1 root root 1303  4月  1  2015 cluster_fail_time.tcl
drwxrwxr-x. 2 root root  162 10月 21 23:08 create-cluster
-rwxrwxr-x. 1 root root 2115  4月  1  2015 generate-command-help.rb
drwxrwxr-x. 2 root root   70  4月  1  2015 hyperloglog
-rwxrwxr-x. 1 root root 8545  4月  1  2015 install_server.sh
drwxrwxr-x. 2 root root   39  4月  1  2015 lru
-rwxrwxr-x. 1 root root  298  4月  1  2015 mkrelease.sh
-rw-rw-r--. 1 root root 1277  4月  1  2015 redis-copy.rb
-rwxrwxr-x. 1 root root 1098  4月  1  2015 redis_init_script
-rwxrwxr-x. 1 root root 1047  4月  1  2015 redis_init_script.tpl
-rw-rw-r--. 1 root root 1762  4月  1  2015 redis-sha1.rb
-rwxrwxr-x. 1 root root 3787  4月  1  2015 speed-regression.tcl
-rwxrwxr-x. 1 root root  693  4月  1  2015 whatisdoing.sh
[root@localhost utils]# cd create-cluster/
[root@localhost create-cluster]# ll
总用量 32
-rwxrwxr-x. 1 root root 2206  4月  1  2015 create-cluster
-rw-rw-r--. 1 root root 1301  4月  1  2015 README
```

### utils中create-cluster命令

```sh
[root@localhost create-cluster]# ./create-cluster
Usage: ./create-cluster [start|create|stop|watch|tail|clean]
start       -- Launch Redis Cluster instances.
create      -- Create a cluster using redis-trib create.
stop        -- Stop Redis Cluster instances.
watch       -- Show CLUSTER NODES output (first 30 lines) of first node.
tail <id>   -- Run tail -f of instance at base port + ID.
clean       -- Remove all instances data, logs, configs.
```

```sh
# 开始6个不同端口redis
./create-cluster start
# 创建redis cluste；分配hash slot；中途需要确认yes
./create-cluster create
# 停止6个redis
./create-cluster stop
# 清楚测试实例
./create-cluster clean
```



### 2.开始-创建6个本机redis

```sh
[root@localhost create-cluster]# ./create-cluster start
Starting 30001
Starting 30002
Starting 30003
Starting 30004
Starting 30005
Starting 30006
```

### 3.创建集群

```sh
[root@localhost create-cluster]# ./create-cluster create
>>> Creating cluster
Connecting to node 127.0.0.1:30001: OK
/usr/local/share/gems/gems/redis-3.0.0/lib/redis.rb:182:in `[]': wrong element type nil at 0 (expected array) (ArgumentError)
	from /usr/local/share/gems/gems/redis-3.0.0/lib/redis.rb:182:in `block (2 levels) in info'
	from /usr/local/share/gems/gems/redis-3.0.0/lib/redis/client.rb:82:in `call'
	from /usr/local/share/gems/gems/redis-3.0.0/lib/redis.rb:180:in `block in info'
	from /usr/local/share/gems/gems/redis-3.0.0/lib/redis.rb:36:in `block in synchronize'
	from /usr/share/ruby/monitor.rb:202:in `synchronize'
	from /usr/share/ruby/monitor.rb:202:in `mon_synchronize'
	from /usr/local/share/gems/gems/redis-3.0.0/lib/redis.rb:36:in `synchronize'
	from /usr/local/share/gems/gems/redis-3.0.0/lib/redis.rb:179:in `info'
	from ../../src/redis-trib.rb:103:in `assert_cluster'
	from ../../src/redis-trib.rb:987:in `block in create_cluster_cmd'
	from ../../src/redis-trib.rb:984:in `each'
	from ../../src/redis-trib.rb:984:in `create_cluster_cmd'
	from ../../src/redis-trib.rb:1373:in `<main>'
[root@localhost create-cluster]# gem install redis
Fetching connection_pool-2.3.0.gem
Successfully installed connection_pool-2.3.0
Fetching redis-client-0.10.0.gem
Successfully installed redis-client-0.10.0
Fetching redis-5.0.5.gem
Successfully installed redis-5.0.5
Parsing documentation for connection_pool-2.3.0
Installing ri documentation for connection_pool-2.3.0
Parsing documentation for redis-client-0.10.0
Installing ri documentation for redis-client-0.10.0
Parsing documentation for redis-5.0.5
Installing ri documentation for redis-5.0.5
Done installing documentation for connection_pool, redis-client, redis after 0 seconds
3 gems installed
[root@localhost create-cluster]# ./create-cluster create
>>> Creating cluster
Connecting to node 127.0.0.1:30001: OK
Connecting to node 127.0.0.1:30002: OK
Connecting to node 127.0.0.1:30003: OK
Connecting to node 127.0.0.1:30004: OK
Connecting to node 127.0.0.1:30005: OK
Connecting to node 127.0.0.1:30006: OK
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:30001
127.0.0.1:30002
127.0.0.1:30003
Adding replica 127.0.0.1:30004 to 127.0.0.1:30001
Adding replica 127.0.0.1:30005 to 127.0.0.1:30002
Adding replica 127.0.0.1:30006 to 127.0.0.1:30003
M: 65de76d343c777ea70b2627711e20c5bff745450 127.0.0.1:30001
   slots:0-5460 (5461 slots) master
M: 04c7bc0e56ba74c795a3b8ceae72a5306df0b72b 127.0.0.1:30002
   slots:5461-10922 (5462 slots) master
M: d9e983be581f1d07ef6d742c69e2a13d66e9536a 127.0.0.1:30003
   slots:10923-16383 (5461 slots) master
S: 8b12876f0328e4e49b260a3c6029b0f84739481e 127.0.0.1:30004
   replicates 65de76d343c777ea70b2627711e20c5bff745450
S: 2f37b926d15460fcbce84bdb75987b1f4fcb2354 127.0.0.1:30005
   replicates 04c7bc0e56ba74c795a3b8ceae72a5306df0b72b
S: 11a8b9487fc5bc3af3f0e5f5737680b0de2f5ea8 127.0.0.1:30006
   replicates d9e983be581f1d07ef6d742c69e2a13d66e9536a
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.
>>> Performing Cluster Check (using node 127.0.0.1:30001)
M: 65de76d343c777ea70b2627711e20c5bff745450 127.0.0.1:30001
   slots:0-5460 (5461 slots) master
M: 04c7bc0e56ba74c795a3b8ceae72a5306df0b72b 127.0.0.1:30002
   slots:5461-10922 (5462 slots) master
M: d9e983be581f1d07ef6d742c69e2a13d66e9536a 127.0.0.1:30003
   slots:10923-16383 (5461 slots) master
M: 8b12876f0328e4e49b260a3c6029b0f84739481e 127.0.0.1:30004
   slots: (0 slots) master
   replicates 65de76d343c777ea70b2627711e20c5bff745450
M: 2f37b926d15460fcbce84bdb75987b1f4fcb2354 127.0.0.1:30005
   slots: (0 slots) master
   replicates 04c7bc0e56ba74c795a3b8ceae72a5306df0b72b
M: 11a8b9487fc5bc3af3f0e5f5737680b0de2f5ea8 127.0.0.1:30006
   slots: (0 slots) master
   replicates d9e983be581f1d07ef6d742c69e2a13d66e9536a
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### 4.测试

同上

参考：[你了解 Redis 的三种集群模式吗？](https://segmentfault.com/a/1190000022808576)

[redis 集群方案的介绍（主从模式、哨兵模式、Redis Cluster模式）](https://blog.51cto.com/u_14032829/3644819)

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

