---
title: Mysql之count()、count(1)、count(列名)区别
date: 2022-09-27 18:12:02
tags: 
- Mysql
index_img: /img/article9.webp
banner_img: /img/post_banner.webp
categories:
- Mysql
---

### 作用

count(*)包括了所有的列，相当于行数，在统计结果的时候，`不会忽略列值为NULL`

count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候，`不会忽略列值为NULL`

count(列名)只包括列名那一列，在统计结果的时候，`会忽略列值为空`（空字符串/0/null）的计数，即某个字段值为NULL时，不统计。

> count(1)，其实就是计算一共有多少符合条件的行。
>
> 1并不是表示第一个字段，而是表示一个固定值。其实就可以想成表中有这么一个字段，这个字段就是固定值1，count(1)，就是计算一共有多少个1。

### 效率

<p class="note note-success">
    <b>count(column)</b>也是会遍历整张表，但是不同的是它会拿到 column 的值以后判断是否为空，然后再进行累加，那么如果针对主键需要解析内容，如果是二级索引需要再次根据主键获取内容，则要多一次 IO 操作，所以 <b>count(column)</b>的性能肯定不如前两者，如果按照效率比较的话：<b>count(*)=count(1)>count(primary key)>count(非主键column)</b>。
</p>

### count(1) and count(*)

当表的数据量大些时，对表作分析之后，使用count(1)还要比使用count()用时多了！从执行计划来看，count(1)和count()的效果是一样的。但是在表做过分析之后，count(1)会比count(*)的用时少些（1w以内数据量），不过差不了多少。
如果count(1)是聚索引,id,那肯定是count(1)快。但是差的很小的。因为count(),自动会优化指定到那一个字段。所以没必要去count(1)，用count()，sql会帮你完成优化的 因此：count(1)和count(*)基本没有差别！

### 实例分析

```sql
mysql> create table counttest(name char(1), age char(2));
Query OK, 0 rows affected (0.03 sec)

mysql> insert into counttest values
    -> ('a', '14'),('a', '15'), ('a', '15'),
    -> ('b', NULL), ('b', '16'),
    -> ('c', '17'),
    -> ('d', null),
    ->('e', '');
Query OK, 8 rows affected (0.01 sec)
Records: 8  Duplicates: 0  Warnings: 0
mysql> select * from counttest;
+------+------+
| name | age  |
+------+------+
| a    | 14   |
| a    | 15   |
| a    | 15   |
| b    | NULL |
| b    | 16   |
| c    | 17   |
| d    | NULL |
| e    |      |
+------+------+
8 rows in set (0.00 sec)
mysql> select name, count(name), count(1), count(*), count(age), count(distinct(age))
    -> from counttest
    -> group by name;
+------+-------------+----------+----------+------------+----------------------+
| name | count(name) | count(1) | count(*) | count(age) | count(distinct(age)) |
+------+-------------+----------+----------+------------+----------------------+
| a    |           3 |        3 |        3 |          3 |                    2 |
| b    |           2 |        2 |        2 |          1 |                    1 |
| c    |           1 |        1 |        1 |          1 |                    1 |
| d    |           1 |        1 |        1 |          0 |                    0 |
| e    |           1 |        1 |        1 |          1 |                    1 |
+------+-------------+----------+----------+------------+----------------------+
5 rows in set (0.00 sec)COPY

```



1[参考1](https://blog.csdn.net/iFuMI/article/details/77920767)

2[参考2](https://zhidao.baidu.com/question/1244677039964272939)

3[参考3](https://www.ikeguang.com/article/713)