---
title: Feign声明式服务
date: 2022-10-11 21:09:02
tags: 
- 微服务
- Spring Cloud Alibaba
- Feign
index_img: /img/article5.webp
banner_img: /img/post_banner.webp
categories:
- 微服务
- Spring Cloud Alibaba
- Feign
comment: waline
---

## 一、背景

当我们通过RestTemplate调用其它服务的API时，所需要的参数须在请求的URL中进行拼接，如果参数少的话或许我们还可以忍受，一旦有多个参数的话，这时拼接请求字符串就会效率低下

那么有没有更好的解决方案呢？答案是确定的有，Netflix已经为我们提供了一个框架：Feign。

## 二、Fegin简介

[Feign](https://github.com/Netflix/feign)是一个声明式 Web 服务客户端。它使编写 Web 服务客户端更容易。要使用 Feign，只需要创建一个接口并添加`@FeignClient`注解即可。它具有可插入的注释支持，包括 Feign 注释和 JAX-RS 注释。

Feign 还支持可插拔的编码器和解码器。

Spring Cloud 添加了对 Spring MVC 注释的支持，并支持使用`HttpMessageConverters`Spring Web 中默认使用的注释。

Spring Cloud 集成 Ribbon 和 Feign，在使用 Feign 时提供负载均衡的 http 客户端。

## 三、Fegin入门案例

### 1. 创建feign_provider

#### 1.1 controller

```java
import com.example.User;
import com.example.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/provider")
public class UserController {
    @Autowired
    private UserService userService;

    @RequestMapping("/getUserById/{id}")
    public User getUserById(@PathVariable Integer id){
        return userService.getUserById(id);
    }
}
```

#### 1.2 service

```java
import com.example.User;

public interface UserService {
    User getUserById(Integer id);
}
```

```java
import com.example.User;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService {
    @Override
    public User getUserById(Integer id){
        return new User(id,"admin-1",18);
    }
}
```

#### 1.3 SpringBootApp

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class FeignProviderApp {
    public static void main(String[] args){
        SpringApplication.run(FeignProviderApp.class,args);
    }
}
```

#### 1.4 application.yml

```yaml
server:
  port: 8090
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 47.98.105.36:8848
  application:
    name: feign-provider
```

### 2. 创建feign_interface

#### 2.1 pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

JDK动态代理

#### 2.2 UserFeign

```java
import com.example.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@FeignClient("feign-provider")
@RequestMapping("/provider")
public interface UserFeign {

    @RequestMapping("/getUserById/{id}")
    public User getUserById(@PathVariable("id") Integer id);//必须增加("id")

}
```

注意：

- 在 @FeignClient 注解中，value 属性的取值为：服务提供者的服务名，即服务提供者配置文件application.yml中 spring.application.name 的取值
- 接口中定义的每个方法都与服务提供者中 Controller 定义的服务方法对应（类似mapper与xml）

### 3. 创建feign_consumer

#### 3.1 controller

```java
import com.example.User;
import com.example.fegin.UserFeign;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.List;
import java.util.Random;

@RestController
@RequestMapping("/consumer")
public class UserController {

    @Autowired
    private UserFeign userFeign;

    @RequestMapping("/getUserById/{id}")
    public User getUserById(@PathVariable Integer id){
        System.out.println(userFeign.getClass());
        return userFeign.getUserById(id);
    }
}
```

#### 3.2 SpringBootApp

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients // 开启feign
public class FeignConsumerApp {
    public static void main(String[] args){
        SpringApplication.run(FeignConsumerApp.class,args);
    }
}
```

`@EnableFeignClients `注解开启Feign扫描，先调用FeignClientsRegistrar.registerFeignClients()方法扫描@FeignClient注解的接口，再将这些接口注入到Spring IOC容器中，方便后续被调用。

#### 3.3 application.yml

```yaml
server:
  port: 80
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 47.98.105.36:8848
  application:
    name: ribbon-consumer
```

## 四、OpenFeign

OpenFeign 全称 Spring Cloud OpenFeign，它是 Spring 官方推出的一种声明式服务调用与负载均衡组件，它的出现就是为了替代进入停更维护状态的 Feign。

OpenFeign 是 Spring Cloud 对 Feign 的二次封装，它具有 Feign 的所有功能，并在 Feign 的基础上增加了对 Spring MVC 注解的支持，例如 @RequestMapping、@GetMapping 和 @PostMapping 等。

OpenFeign 常用注解：

| 注解                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| @FeignClient        | 该注解用于通知 OpenFeign 组件对 @RequestMapping 注解下的接口进行解析，并通过动态代理的方式产生实现类，实现负载均衡和服务调用。 |
| @EnableFeignClients | 该注解用于开启 OpenFeign 功能，当 Spring Cloud 应用启动时，OpenFeign 会扫描标有 @FeignClient 注解的接口，生成代理并注册到 Spring 容器中。 |

## 五、Feign请求超时

### 5.1 模拟超时

修改feign_provider：

```java
import com.example.User;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService {
    @Override
    public User getUserById(Integer id){
            //模拟网络延迟
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        return new User(id,"admin-1",18);
    }
}
```

### 5.2 测试

```shell
 ERROR 20032 --- [p-nio-80-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is feign.RetryableException: Read timed out executing GET http://feign-provider/provider/getUserById/11111] with root cause

java.net.SocketTimeoutException: Read timed out
```

### 5.3.设置feign超时时间

在feign_consumer的application.yml中加入

```yaml
ribbon:
  ConnectTimeout: 5000 #请求连接的超时时间
  ReadTimeout: 5000 #请求处理的超时时间
```

> 注意：
>
> IDEA无提词并警告
>
> Cannot resolve configuration property 'ribbon.ReadTimeout'
>
> Cannot resolve configuration property 'ribbon.ConnectTimeout'
>
> 运行程序测试发现Read timed out异常问题解决，配置生效
>
> IDEA报错原因分析：
>
> 由于 OpenFeign 集成了 Ribbon ，其服务调用以及负载均衡在底层都是依靠 Ribbon 实现的，因此 OpenFeign 超时控制也是通过 Ribbon 来实现的。
>
> 这是IDEA的智能提示，但是它的智能是一定的逻辑支持的，因为ribbon的配置比较复杂，IDEA怀疑这个配置可能是多余的。[issues参考](https://github.com/dyc87112/blog-comments/issues/118)
>
> 解决方案：
>
> [解决IDEA警告可进入此链接](https://stackoverflow.com/questions/48954087/intellij-idea-complains-cannot-resolve-spring-boot-properties-but-they-work-fine)



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