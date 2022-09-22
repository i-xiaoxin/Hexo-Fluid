---
title: dubbo微服务基础知识
date: 2022-09-21 18:12:02
tags: 
- dubbo
- dubbox
- zookeeper
index_img: /img/article3.jpg
banner_img: /img/post_banner.jpg
categories:
- 微服务
---



<div align="center"><a href="https://i-xiaoxin.github.io/"><img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=25&pause=1000&width=435&lines=%F0%9F%8D%8E%E8%B7%AF%E6%BC%AB%E6%BC%AB%E5%85%B6%E4%BF%AE%E8%BF%9C%E5%85%AE%2C%E5%90%BE%E5%B0%86%E4%B8%8A%E4%B8%8B%E8%80%8C%E6%B1%82%E7%B4%A2;The+road+ahead+will+be+long;+Our+climb+will+be+steep" alt="Typing SVG" /></a></div>

### 🚩微服务出现背景简述

2008年以后，国内互联网行业飞速发展，我们对软件系统的需求已经不再是过去”能用就行”这种需求了，像**抢红包、双十一**这样的活动不断逼迫我们去突破软件系统的性能上限，传统的IT企业”能用就行”的开发思想已经不能满足互联网**高并发、大流量**的性能要求。系统架构走向**分布式**已经是服务器开发领域解决该问题唯一的出路，然而分布式系统由于天生的复杂度，并不像开发单体应用一样把框架一堆就能搞定，因此各大互联网公司都在投入技术力量研发自己的基础设施。这里面比较有名的如阿里的开源项目dubbo, Netflix开发的一系列服务框架。

### 🚩系统架构演变

- #### 单体架构

    单体架构也称之为单体系统或者是单体应用。就是一种把系统中所有的功能、模块耦合在一个应用中的架构方式

    ![单体架构](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20220922204728268.png)

    > 存在以下问题：
    >
    > ​	[🏀](https://emojipedia.org/basketball/) 代码耦合：模块的边界模糊、依赖关系不清晰，整个项目非常复杂，每次修改代码都心惊胆战
    >
    > ​	[🏀](https://emojipedia.org/basketball/)  迭代困难：每次功能的变更或bug的修复都会导致重新部署整个应用，随着代码的增多，构建、测试和部署的时间也会增加
    >
    > ​	[🏀](https://emojipedia.org/basketball/)  扩展受限：单体应用只能作为一个整体进行扩展，无法根据业务模块的需要进行伸缩
    >
    > ​	[🏀](https://emojipedia.org/basketball/)  技术债务：随着时间推移、需求变更和人员更迭，会逐渐形成应用程序的技术债务，并且越积越多不坏不修
    >
    > ​	[🏀](https://emojipedia.org/basketball/)  阻碍创新：单体应用往往使用统一的技术平台或方案解决所有的问题，要想引入新技术平台会非常困难

- #### 分布式架构

    分布式是需要按照功能点把系统拆分，拆分成独立的功能，单独为某一个节点添加服务器，需要系统之间配合才能完成整个业务逻辑。

    ![分布式架构](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20220922205059287.png)

    > 分布式架构优点：
    >
    > ​	[🧩](https://emojipedia.org/puzzle-piece/) 不同的团队负责不同的子项目
    >
    > ​	[🧩](https://emojipedia.org/puzzle-piece/) 可以灵活的进行分布式部署
    >
    > ​	[🧩](https://emojipedia.org/puzzle-piece/) 可以为某一模块单独加集群
    >
    > 分布式架构缺点：
    >
    > ​	[🎬](https://emojipedia.org/clapper-board/)  模块之间有一些通用的业务逻辑无法共用。

- #### SOA架构

    SOA：Service Oriented Architecture（面向服务的架构）。也就是把工程拆分成**服务层，表现层**两个工程。服务层中包含业务逻辑，只需要对外提供服务即可。表现层只需要处理和页面的交互，业务逻辑都是调用服务层的服务来实现，使用ESB（Enterparise Servce Bus企业服务总线，代表技术：Mule、WSO2）提供表现层和服务层之间的交互。

![SOA架构](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20220922205438294.png)

> 存在的问题：
>
> [🏐](https://emojipedia.org/volleyball/)  不支持集群、臃肿

### Dubbo框架

#### Dubbo简介

因为需求的增长导致架构创新，dubbo的出现就是为了实现SOA架构的框架，Dubbo(读音[ˈdʌbəʊ])是阿里巴巴公司开源的一个基于Java的高性能**RPC**（Remote Procedure Call）框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring框架无缝集成。后期阿里巴巴停止了该项目的维护，于是当当网在这之上推出了自己的Dubbox，项目 [Apache Dubbo](https://dubbo.apache.org/zh/)



#### Dubbo架构

![Dubbo架构](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20220922205824470.png)

### 注册中心 zookeeper

[Zookeeper](https://zookeeper.apache.org/)是Apacahe Hadoop的子项目，可以为分布式应用程序协调服务，适合作为Dubbo服务的注册中心，负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互。

![zookeeper](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20220922210506808.png)

### zookeeper的安装

<p class="note note-success">
	(1) 安装jdk
</p>

<p class="note note-success">
	(2) 上传并解压缩zookeeper压缩包
</p>

<p class="note note-success">
    (3) 进入 bin 目录，启动服务输入命令
</p>

### Dubbo案例Demo

#### [🏅](https://emojipedia.org/sports-medal/)父工程的pom.xml中添加依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
    </parent>
    
    <groupId>com.bjpowernode</groupId>
    <artifactId>dubbox_parent</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- Dubbo Spring Boot Starter -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.1.0</version>
        </dependency>
        <!-- 由于使⽤了zookeeper作为注册中⼼，则需要加⼊zookeeper的客户端jar包： -->
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
        </dependency>
    </dependencies>
</project>

```

#### [🏅](https://emojipedia.org/sports-medal/)创建公共接口

```java
public interface HelloService{
    String hello();
}

```

#### [🏅](https://emojipedia.org/sports-medal/)定义服务提供方并配置

```java
//创建service
import com.alibaba.dubbo.config.annotation.Service;
//import org.springframework.stereotype.Service;
@Service
public class HelloServiceImpl implements HelloService{
    @Override
    public String hello() {
        return "hello,Dubbox.......";
    }
}

```

#### [🏅](https://emojipedia.org/sports-medal/)定义服务消费方

```java
//定义controller
import com.alibaba.dubbo.config.annotation.Reference;
import com.bjpowernode.service.HelloService;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {

    @Reference
    //@Autowired
   private HelloService helloService;

    @RequestMapping("/hello")
    @ResponseBody
    public String hello() {
        return helloService.hello();
    }
}

```

### 推荐IDEA安装zookeeper插件

