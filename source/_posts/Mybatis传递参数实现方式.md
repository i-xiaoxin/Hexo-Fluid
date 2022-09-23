---
title: Mybatis传递参数实现方式
date: 2022-09-21 18:12:02
tags: 
- Mybatis
- DAO
index_img: /img/article4.jpg
banner_img: /img/post_banner.jpg
categories:
- Mybatis
---

### 前言

### 单个参数

传参的基本方式有两种：`#{param}` `${param}`

<p class="note note-success">
    #{}为参数占位符?，即SQL预编译 <B>推荐使用</B></br>
    ${}为字符串替换，即SQL拼接
</p>

### 多个参数

#### 实体传入

service层：

```java
@Override
public User getUserInfo(User user) {
    User userInfo = userMapper.getUserInfo(user);
    return userInfo;
}
```

#### mapper层：

```java
@Mapper
public interface UserMapper{
    User getUserInfo(User User);
}
```

#### mapper.xml：

```xml

<select id="getUserInfo"  parameterType="User"  resultType="com.pojo.User">
    select userId
    from users
    where userId=#{userId} and sex=#{sex};
</select>
```

### Map传入

#### service层：

```java
@Override
public User getUserInfo(Map map) {
    User user = userMapper.getUserInfo(map);
    return user;
}
```

#### mapper层：

```java
@Mapper
public interface UserMapper{
    User getUserInfo(Map map);
}
```

#### mapper.xml层：

```xml
<!--查询-->
<select id="getUserInfo"  parameterType="Map"  resultType="com.pojo.User">
    select userId
    from users
    where userId=#{userId} and sex=#{sex};
</select>
```

### 注解@Param传入

#### service层：

```java
@Override
public User getUserInfo(User user,Integer age) {
    User userResult = userMapper.getUserInfo(user,age);
    return userResult;
}
```

#### mapper层： 

```java
@Mapper
public interface UserMapper{
    User getUserInfo(@Param("userInfo") User user,@Param("age") Integer age);
}
```

#### mapper.xml：

```xml
<!--查询-->
<select id="getUserInfo"   resultType="com.pojo.User">
    select userId
    from users
    where userId=#{userInfo.userId} and sex=#{userInfo.sex} and age=#{age};
</select>
```

参考：

[^1]:[mybatis 传递参数的7种方法 ](https://www.cnblogs.com/cangqinglang/p/14237361.html)
[^2]: [SpringBoot 使用MyBatis的几种传参规范示例](http://www.codebaoku.com/it-java/it-java-238108.html)

