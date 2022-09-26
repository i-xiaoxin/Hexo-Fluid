---
title: Redis数据结构
date: 2022-09-26 18:12:02
tags: 
- Redis
index_img: /img/article7.jpg
banner_img: /img/post_banner.jpg
categories:
- Redis
---

### Reids数据结构

<p class="note note-success">
    数据库掌握的就是数据结构，Redis数据结构就是“键（key）”；“键”道不过是赋值//取值/删除，因此要想入道，就从基础开始学习
</p>

| 数据类型 |               赋值语法               |             取值语法             |           删除语法            |   备注   |
| :------: | :----------------------------------: | :------------------------------: | :---------------------------: | :------: |
|  String  |          SET key_name value          |           GET key_name           |         DEL key_name          |  String  |
|   Hash   |      HSET key_name field value       |       HGET key_name field        |      HDEL key_name field      |   Map    |
|   List   | LPUSH/RPUSH key_name value1...valueN |      LRANGE key_name  0 -1       |   LREM key_name count value   |   List   |
|   Set    |    SADD key_name value1...valueN     |        SMEMBERS key_name         | SREM key_name value1...valueN |   Set    |
|   Zset   |     ZADD key_name   score value      | ZRANGE key_name 0 -1[WITHSCORES] |      ZRANK key [member]       | Sort Set |

***注意：**DEL key命令可以删除任何key

### String命令示例

```sql
# 对不存在的键进行设置
 
redis 127.0.0.1:6379> SET key "value"
OK
 
redis 127.0.0.1:6379> GET key
"value"
 
 
# 对已存在的键进行设置
 
redis 127.0.0.1:6379> SET key "new-value"
OK
 
redis 127.0.0.1:6379> GET key
"new-value"
```

### Hset命令示例

```sql
redis 127.0.0.1:6379> HSET myhash field1 "foo"
OK
redis 127.0.0.1:6379> HGET myhash field1
"foo"
 
redis 127.0.0.1:6379> HSET website google "www.g.cn"       # 设置一个新域
(integer) 1
 
redis 127.0.0.1:6379>HSET website google "www.google.com" # 覆盖一个旧域
(integer) 0
```



### List命令示例

```sql
#LPUSH KEY_NAME VALUE1.. VALUEN
redis 127.0.0.1:6379> LPUSH list1 "foo"
(integer) 1
redis 127.0.0.1:6379> LPUSH list1 "bar"
(integer) 2
redis 127.0.0.1:6379> LRANGE list1 0 -1
1) "foo"
2) "bar
```



```sql
#LRANGE KEY_NAME START END
redis 127.0.0.1:6379> LPUSH list1 "foo"
(integer) 1
redis 127.0.0.1:6379> LPUSH list1 "bar"
(integer) 2
redis 127.0.0.1:6379> LPUSHX list1 "bar"
(integer) 0
redis 127.0.0.1:6379> LRANGE list1 0 -1
1) "foo"
2) "bar"
3) "bar"
```

### Set命令示例

```sql
# SADD KEY_NAME VALUE1..VALUEN
redis 127.0.0.1:6379> SADD myset "hello"
(integer) 1
redis 127.0.0.1:6379> SADD myset "foo"
(integer) 1
redis 127.0.0.1:6379> SADD myset "hello"
(integer) 0
redis 127.0.0.1:6379> SMEMBERS myset
1) "hello"
2) "foo"
```

```sql
#SMEMBERS KEY VALUE
redis 127.0.0.1:6379> SADD myset1 "hello"
(integer) 1
redis 127.0.0.1:6379> SADD myset1 "world"
(integer) 1
redis 127.0.0.1:6379> SMEMBERS myset1
1) "World"
2) "Hello"
```



### Zset命令示例

```sql
#ZADD KEY_NAME SCORE1 VALUE1.. SCOREN VALUEN
redis 127.0.0.1:6379> ZADD myset 1 "hello"
(integer) 1
redis 127.0.0.1:6379> ZADD myset 1 "foo"
(integer) 1
redis 127.0.0.1:6379> ZADD myset 2 "world" 3 "bar"
(integer) 2
redis 127.0.0.1:6379> ZRANGE myzset 0 -1 WITHSCORES
1) "hello"
2) "1"
3) "foo"
4) "1"
5) "world"
6) "2"
7) "bar"
8) "3"
```

```sql
# ZRANGE key start stop [WITHSCORES]
redis 127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES             # 显示整个有序集成员
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"
 
redis 127.0.0.1:6379> ZRANGE salary 1 2 WITHSCORES              # 显示有序集下标区间 1 至 2 的成员
1) "tom"
2) "5000"
3) "boss"
4) "10086"
 
redis 127.0.0.1:6379> ZRANGE salary 0 200000 WITHSCORES         # 测试 end 下标超出最大下标时的情况
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"
 
redis > ZRANGE salary 200000 3000000 WITHSCORES                  # 测试当给定区间不存在于有序集时的情况
(empty list or set)
```

### 其他常用命令

<p class="note note-success">
    Redis在实际使用过程中更多的用作缓存，然而缓存的数据一般都是需要设置生存时间的，即：到期后数据销毁。
</p>

| keys命令                 | 描述                                                 |
| ------------------------ | ---------------------------------------------------- |
| EXPIRE key seconds       | 设置key的生存时间（单位：秒）key在多少秒后会自动删除 |
| TTL key                  | 查看key的生存时间                                    |
| PERSIST key              | 清除生存时间                                         |
| PEXPIRE key milliseconds | 生存时间设置单位为：毫秒                             |
| KEYS *                   | 查询数据库中素有key键                                |
| SDIFF setB setA          | 集合的差集运算A-B（difference）                      |
| SINTER setA setB         | 集合的交集运算A ∩ B（intersection）                  |
| SUNION setA setB         | 集合的并集运算A ∪B（union）                          |

[redis命令示例查询](https://www.redis.net.cn/order/)



