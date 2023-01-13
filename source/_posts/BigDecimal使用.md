---
title: BigDecimal使用
date: 2023-1-7 21:09:02
tags: 
- Enum
- for each
index_img: 
banner_img: /img/article2.webp
categories:
- Java
comment: waline
---

## **1、构建BigDecimal**

```cpp
BigDecimal BigDecimal(double d); //不允许使用,精度不能保证
BigDecimal BigDecimal(String s); //常用,推荐使用
static BigDecimal valueOf(double d); //常用,推荐使用
```

## **2、方法**

|         方法         | 描叙                                       |
| :------------------: | ------------------------------------------ |
|   add(BigDecimal)    | BigDecimal对象中的值相加，然后返回这个对象 |
| subtract(BigDecimal) | BigDecimal对象中的值相减，然后返回这个对象 |
| multiply(BigDecimal) | BigDecimal对象中的值相乘，然后返回这个对象 |
|  divide(BigDecimal)  | BigDecimal对象中的值相除，然后返回这个对象 |
|      toString()      | 将BigDecimal对象的数值转换成字符串         |
|    doubleValue()     | 将BigDecimal对象中的值以双精度数返回       |
|     floatValue()     | 将BigDecimal对象中的值以单精度数返回       |
|     longValue()      | 将BigDecimal对象中的值以长整数返回         |
|      intValue()      | 将BigDecimal对象中的值以整数返回。         |

## **3、格式化和四舍五入**

```cpp
// 格式化：保留2为小数
DecimalFormat df = new DecimalFormat("#.##");
// 四舍五入，默认五舍六入
df.setRoundingMode(RoundingMode.HALF_UP);
```

## **4、格式化**

 DecimalFormat 解析：

|   符号    |       位置 |                             描叙                             |
| :-------: | ---------: | :----------------------------------------------------------: |
|     0     |       数字 |                阿拉伯数字，如果不存在则显示0                 |
|     #     |       数字 |                阿拉伯数字，如果不存在不显示0                 |
|     .     |       数字 |                  小数分隔符或货币小数分隔符                  |
|     ,     |       数字 |                          分组分隔符                          |
|     E     |       数字 |  分隔科学计数法中的尾数和指数。*在前缀或后缀中无需加引号。*  |
|     -     |       数字 |                             负号                             |
|     ;     | 子模式边界 |                     分隔正数和负数子模式                     |
|     %     | 前缀或后缀 |                   乘以 100 并显示为百分数                    |
|  \u2030   | 前缀或后缀 |                   乘以 1000 并显示为千分数                   |
| ¤(\u00A4) | 前缀或后缀 | 货币记号，由货币符号替换。如果两个同时出现，则用国际货币符号替换。如果出现在某个模式中，则使用货币小数分隔符，而不使用小数分隔符。 |
|     '     | 前缀或后缀 | 用于在前缀或或后缀中为特殊字符加引号，例如 "'#'#"将 123 格式化为 "#123"。要创建单引号本身，请连续使用两个单引号："# o''clock"。 |

## **5、舍入模式介绍**

- `RoundingMode.CEILNG`：向正无限大方向舍入的舍入模式。如果结果为正，则舍入行为类似于 `RoundingMode.UP`；如果结果为负，则舍入行为类似于 `RoundingMode.DOWN`
   如下：

```dart
5.5  =>  6 
1.1  =>  2
-1.0  =>  -1 
-2.5  =>  -2
```

- `RoundingMode.DOWN`:向零方向舍入的舍入模式。从不对舍弃部分前面的数字加 1（即截尾）。注意，此舍入模式始终不会增加计算值的绝对值
   如下：

```dart
5.5  =>  5 
1.1  =>  1 
-1.0  =>  -1 
-1.6  =>  -1  
```

- `RoundingMode.FLOOR`：向负无限大方向舍入的舍入模式。如果结果为正，则舍入行为类似于 RoundingMode.DOWN；如果结果为负，则舍入行为类似于 RoundingMode.UP
   `注意：此舍入模式始终不会增加计算值`
   如下：

```dart
5.5  =>  5 
2.3  =>  2
1.0  =>  1 
-1.1  =>  -2 
-2.5  =>  -3 
```

- RoundingMode.HALF_DOWN ：向最接近数字方向舍入的舍入模式，如果与两个相邻数字的距离相等，则向下舍入。如果被舍弃部分 > 0.5，则舍入行为同 RoundingMode.UP；否则舍入行为同 RoundingMode.DOWN；
   `五舍六入`
   如下：

```dart
2.5  =>  2 
1.6  =>  2
-1.1  =>  -1 
-1.6  =>  -2
-2.5  =>  -2
```

- RoundingMode.HALF_EVEN ：向 最接近数字方向舍入的舍入模式，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。如果舍弃部分左边的数字为奇数，则舍入行为同 RoundingMode.HALF_UP；如果为偶数，则舍入行为同 RoundingMode.HALF_DOWN。注意，在重复进行一系列计算时，此舍入模式可以在统计上将累加错误减到最小。此舍入模式也称为“银行家舍 入法”，主要在美国使用。此舍入模式类似于 Java 中对 float 和 double 算法使用的舍入策略
   如下：

```dart
5.5  =>  6 
2.5  =>  2 
1.1  =>  1 
-1.0  =>  -1 
-1.6  =>  -2 
-2.5  =>  -2
```

- RoundingMode.HALF_UP ：向最接近数字方向舍入的舍入模式，如果与两个相邻数字的距离相等，则向上舍入。如果被舍弃部分 >= 0.5，则舍入行为同 RoundingMode.UP；否则舍入行为同 RoundingMode.DOWN
   `注意：此舍入模式就是通常学校里讲的四舍五入`(高频)
   如下：

```dart
2.5  =>  3 
1.1  =>  1 
-1.1  =>  -1 
-1.6  =>  -2 
```

- RoundingMode.UNNECESSARY ：用于断言请求的操作具有精确结果的舍入模式，因此不需要舍入。如果对生成精确结果的操作指定此舍入模式，则抛出 ArithmeticException
   如下：

```dart
1.5  =>  抛出 ArithmeticException
1.1  =>  抛出 ArithmeticException
1.0  =>  1
-1.1  =>抛出 ArithmeticException 
-1.6  =>  抛出 ArithmeticException 
```

- RoundingMode.UP ：远离零方向舍入的舍入模式。始终对非零舍弃部分前面的数字加 1。注意，此舍入模式始终不会减少计算值的绝对值
   如下：

```dart
5.5  =>  6 
1.1  =>  2 
-1.1  =>  -2 
-1.6  =>  -2 
```

## 问题

1. BigDecimal的divide方法进行除法时当不整除，出现无限循环小数时，就会抛异常的，异常如下：java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result. at java.math.BigDecimal.divide(Unknown Source)

> 解决方法：给divide设置精确的小数点


原文链接：[BigDecimal使用](https://www.jianshu.com/p/2947868d76eb)
参考阅读：[BigDecimal](https://www.liaoxuefeng.com/wiki/1252599548343744/1279768011997217)

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