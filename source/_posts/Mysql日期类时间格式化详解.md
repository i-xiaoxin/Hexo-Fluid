---
title: Mysql日期类时间格式化详解
date: 2022-09-27 19:22:02
tags: 
- Mysql
- Date_format
index_img: 
banner_img: /img/article4.webp
categories:
- Mysql

---

### MySQL时间格式化函数date_format()用法详解

### 语法

```sql
DATE_FORMAT(date,format)
```

### 参数

- *`date`*

    必需的。需要格式化的日期。

- *`format`*

    必需的。格式化模式字符串。

### 返回值

`DATE_FORMAT()` 函数按照指定的格式格式化日期时间并返回格式化后的字符串。

如果其中任何一个参数为 `NULL`，`DATE_FORMAT()` 函数将返回 `NULL`。

### `DATE_FORMAT()` 示例

#### 示例 1: 格式化日期

```sql
SELECT
    DATE_FORMAT('2022-02-28', '%Y'),
    DATE_FORMAT('2014-02-28', '%W'),
    DATE_FORMAT('2022-02-01', '%M %d, %Y'),
    DATE_FORMAT('2022-02-01', '%M %e %Y'),
    DATE_FORMAT('2022-02-28', '%W, %M %e, %Y'),
    
# 输出结果
 DATE_FORMAT('2022-02-28', '%Y'): 2022
 DATE_FORMAT('2014-02-28', '%W'): Friday
 DATE_FORMAT('2022-02-01', '%M %d, %Y'): February 01, 2022
 DATE_FORMAT('2022-02-01', '%M %e %Y'): February 1 2022
 DATE_FORMAT('2022-02-28', '%W, %M %e, %Y'): Monday, February 28, 2022
```

#### 示例 2: 格式化日期和时间

```sql
SELECT NOW(), DATE_FORMAT(NOW(), '%Y%m%d%H%i%S'),

# 输出结果
NOW(): 2022-04-12 03:18:38
DATE_FORMAT(NOW(), '%Y%m%d%H%i%S'): 20220412031838
```

### 参数符号

| 符号 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| `%a` | 星期的缩写 (`Sun`..`Sat`)                                    |
| `%b` | 月份的缩写 (`Jan`..`Dec`)                                    |
| `%c` | 月份数字 (`0`..`12`)                                         |
| `%D` | 带有英语前缀的月份中的每天 (`0th`, `1st`, `2nd`, `3rd`, …)   |
| `%d` | 月份中的每天的两位数字表示 (`00`..`31`)                      |
| `%e` | 月份中的每天的数字表示 (`0`..`31`)                           |
| `%f` | 微秒 (`000000`..`999999`)                                    |
| `%H` | 小时 (`00`..`23`)                                            |
| `%h` | 小时 (`01`..`12`)                                            |
| `%I` | 小时 (`01`..`12`)                                            |
| `%i` | 分钟 (`00`..`59`)                                            |
| `%j` | 一年中的每天 (`001`..`366`)                                  |
| `%k` | 小时 (`0`..`23`)                                             |
| `%l` | 小时 (`1`..`12`)                                             |
| `%M` | 月份名称 (`January`..`December`)                             |
| `%m` | 两位数字月份 (`00`..`12`)                                    |
| `%p` | `AM` 或者 `PM`                                               |
| `%r` | 十二小时制时间 (*`hh:mm:ss`* 后跟 `AM` 或 `PM`)              |
| `%S` | 秒 (`00`..`59`)                                              |
| `%s` | 秒 (`00`..`59`)                                              |
| `%T` | 二十四小时制时间 (*`hh:mm:ss`*)                              |
| `%U` | 一年中的星期 (`00`..`53`), 每周的开始是星期天; [`WEEK()`](https://www.sjkjc.com/mysql-ref/week/) 函数中的 mode 0 |
| `%u` | 一年中的星期 (`00`..`53`), 每周的开始是星期一; [`WEEK()`](https://www.sjkjc.com/mysql-ref/week/) 函数中的 mode 1 |
| `%V` | 一年中的星期 (`01`..`53`), 每周的开始是星期天; [`WEEK()`](https://www.sjkjc.com/mysql-ref/week/) 函数中的 mode 2, 用于 `%X` |
| `%v` | 一年中的星期 (`01`..`53`), 每周的开始是星期一; [`WEEK()`](https://www.sjkjc.com/mysql-ref/week/) 函数中的 mode 3, 用于 `%x` |
| `%W` | 星期的名称 (`Sunday`..`Saturday`)                            |
| `%w` | 星期中的每天 (`0`=星期天..`6`=星期六)                        |
| `%X` | 一年中的星期，每周的开始是星期天，四位数字，用于 `%V`        |
| `%x` | 一年中的星期，每周的开始是星期一，四位数字，用于 `%v`        |
| `%Y` | 四位数字年份                                                 |
| `%y` | 两位数字年份                                                 |
| `%%` | 转义 `%`                                                     |
| `%x` | *`x`*, 上面为列举的其他字符                                  |

1.[菜鸟SQL教程参考](https://www.runoob.com/sql/sql-dates.html)

2.[MySQL 教程DATE_FORMAT() 函数参考](https://www.sjkjc.com/mysql-ref/date_format/)