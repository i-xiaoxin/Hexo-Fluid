---
title: Feign声明式服务
date: 2022-10-13 21:09:02
tags: 
- 微服务
- Spring Cloud Alibaba
- Feign
- HTTPClient
- OpenFeign
index_img: 
banner_img: /img/article3.webp
categories:
- 微服务
- Spring Cloud Alibaba
- Feign
- HTTPClient
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

## 五、Feign原理

### 1. 扫描Feign接口并注入到Spring容器

`@EnableFeignClients`开启Feign接口扫描，调用FeignClientsRegistrar.registerFeignClients()方法扫描含有`@FeignClient`注解的接口，在这些接口调用时生成`代理类`注入到Spring IOC容器中，方便后续被调用。

### 2. RequestTemplate封装请求信息

当Controller调用Feign代理类时，通过JDK的代理方式为Feign接口生成的一个动态代理类，代理类调用`SynchronousMethodHandler.invoke()`创建一个RequestTemplate对象，该对象封装了HTTP请求需要的全部信息，如请url、参数，请求方式等信息

```java
public Object invoke(Object[] argv) throws Throwable {
    //创建一个RequestTemplate
    RequestTemplate template = this.buildTemplateFromArgs.create(argv);
    Retryer retryer = this.retryer.clone();

    while(true) {
        try {
            //发出请求
            return this.executeAndDecode(template);
        } catch (RetryableException var8) {
            ... ... ...
        }
    }
}
```

### 3. 发起请求

`SynchronousMethodHandler.executeAndDecode()`通过RequestTemplate生成Request，然后把Request交给Client去处理，Client可以是JDK原生的URLConnection，Apache的HttpClient，也可以时OKhttp，最后Client结合Ribbon负载均衡发起服务调用请求。

```java
 Object executeAndDecode(RequestTemplate template) throws Throwable {
        //生成请求对象
        Request request = this.targetRequest(template);
        if (this.logLevel != Level.NONE) {
            this.logger.logRequest(this.metadata.configKey(), this.logLevel, request);
        }

        long start = System.nanoTime();

        Response response;
        try {
            //发起请求
            response = this.client.execute(request, this.options);
        } catch (IOException var15) {
            ... ... ...

            throw FeignException.errorExecuting(request, var15);
        }
 }       
```



## 六、Fegin接口三种传参方式

-  当参数比较复杂时，Feign即使声明为get请求也会强行使用post请求
- 不支持@GetMapping类似注解声明请求，需使用@RequestMapping(value = "url",method = RequestMethod.GET)

### 1. ？拼接方式传参

```java
/**
* feign_provider
*/
@RestController
@RequestMapping("/provider")
public class UserController {
    @Autowired
    private UserService userService;
    
    @RequestMapping("/delUserById")
    public User delUserById(Integer id){
        return userService.delUserById(id);
    }  
    // 数组形式
   	 @RequestMapping("/delUserByIdS")
    public Integer[] deletedIds(Integer[] ids){
        return userService.delUserByIds(ids);
    }
}

```

```java
/**
* feign_consumer
*/
@RestController
@RequestMapping("/provider")
public class UserController {
    @Autowired
    private UserFeign userFeign;
    
    @RequestMapping("/delUserById")
    public User delUserById(Integer id){
        return userFeign.delUserById(id);
    }
    
   	// 数组
    @RequestMapping("/delUserByIdS")
    public Integer[] delUserByIdS(Integer[] ids){
        return userFeign.delUserByIds(ids);
    }
}
```

```java
/**
* UserFeign 接口
*/
@FeignClient("feign-provider")
@RequestMapping("/provider")
public interface UserFeign {
    /**
    * 参数处 @RequestParam("id")
    */
    @RequestMapping("/delUserById")
    User delUserById(@RequestParam("id") Integer id);
    // RequestParam("id") 作用 拼接参数->  http://127.0.0.1/consumer/delUserById?id=18
    
    @RequestMapping("/delUserByIdS")
    Integer[] delUserByIds(@RequestParam("ids")  Integer[] ids);
    //@RequestParam("ids") 作用 拼接参数-> http://127.0.0.1/consumer/delUserByIdS?ids=1&ids=2
}

```

接口注意使用`@RequestParam("value")`接收拼接参数

### 2. RESTful方式传参

```java
/**
* feign_provider
*/
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

```java
/**
* feign_consumer
*/
@RestController
@RequestMapping("/provider")
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

```java
/**
* UserFeign 接口
*/
@FeignClient("feign-provider")
@RequestMapping("/provider")
public interface UserFeign {
    // 参数拼接@PathVariable("id") 
    @RequestMapping("/getUserById/{id}")
    User getUserById(@PathVariable("id") Integer id);
    // @PathVariable("id") 作用 拼接参数-> http://127.0.0.1/consumer/getUserById/1
}

```

接口注意使用` @PathVariable("value")`接收拼接参数

### 3. POJO传参方式

```java
/**
* feign_provider
*/
@RestController
@RequestMapping("/provider")
public class UserController {
    @Autowired
    private UserService userService;
    
  	 @RequestMapping("/addUser")
    public User addUser(@RequestBody User user){
        return userService.addUser(user);
    }
    // 集合POJO
    @RequestMapping("/addUsers")
    public List<User> addUsers(@RequestBody List<User> userList){
        return userService.addUsers(userList);
    }
}
```

```java
/**
* feign_consumer
*/
@RestController
@RequestMapping("/provider")
public class UserController {
    @Autowired
    private UserFeign userFeign;
    
    @RequestMapping("/addUser")
    public User addUser(User user){
        return userFeign.addUser(user);
    }
    
    @RequestMapping("/addUsers")
    public List<User> addUser(){
        List<User> userList = new ArrayList<>();
        userList.add(new User(1,"user",18));
        userList.add(new User(2,"user2",18));
        userList.add(new User(3,"user3",18));
        userList.add(new User(4,"user4",18));
        return userFeign.addUsers(userList);
    }
}
```

```java
/**
* UserFeign 接口
*/
@FeignClient("feign-provider")
@RequestMapping("/provider")
public interface UserFeign {
    
    @RequestMapping("/addUser")
    User addUser(@RequestBody User user);
    // @RequestBody  作用 拼接参数-> http://127.0.0.1/consumer/addUser
    
    @RequestMapping("/addUsers")
    List<User> addUsers(@RequestBody List<User> userList);
    // @RequestBody 作用 拼接参数->  http://127.0.0.1/consumer/addUsers
}
```

接口注意使用` @RequestBody`接收拼接参数

## 七、Feign优化

### 1. Feign日志开启

一个顶呱呱的框架怎么能没有日志？

```yaml
logging.level.project.user.UserClient: DEBUG

```

#### 1.1 Feign 日志级别

- `NONE`  不记录日志，Feign默认配置

- `BASIC` 只记录请求方法和 URL 以及响应状态码和执行时间。

- `HEADERS` 记录基本信息以及请求和响应标头。

- `FULL` 记录请求和响应的标头、正文和元数据；级别最高

### 1.2 Feign开启配置

 1. Java配置Bean方式

    ```java
    @Configuration
    public class FooConfiguration {
        @Bean
        Logger.Level feignLoggerLevel() {
            return Logger.Level.FULL;
        }
    }
    ```

 2. yaml方式配置（推荐）

    feign_consumer中的application.yml

    ```yaml
    feign:
      client:
        config:
          default: #设置default(推荐)或者是服务注册名
            logger-level: full #同时需要设置日志输出级别
    # 同时需要开启log4j日志级别
    logging:
      level:
        com.bjpowernode.fegin: debug
    ```

### 1.3 控制台日志输出

```shell
2022-10-13 16:23:09.873 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] ---> GET http://feign-provider/provider/getUserById/2 HTTP/1.1
2022-10-13 16:23:09.873 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] Accept-Encoding: gzip
2022-10-13 16:23:09.873 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] Accept-Encoding: deflate
2022-10-13 16:23:09.873 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] ---> END HTTP (0-byte body)
2022-10-13 16:23:09.963  INFO 19580 --- [p-nio-80-exec-1] c.netflix.config.ChainedDynamicProperty  : Flipping property: feign-provider.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
2022-10-13 16:23:09.984  INFO 19580 --- [p-nio-80-exec-1] c.netflix.loadbalancer.BaseLoadBalancer  : Client: feign-provider instantiated a LoadBalancer: DynamicServerListLoadBalancer:{NFLoadBalancer:name=feign-provider,current list of Servers=[],Load balancer stats=Zone stats: {},Server stats: []}ServerList:null
2022-10-13 16:23:09.988  INFO 19580 --- [p-nio-80-exec-1] c.n.l.DynamicServerListLoadBalancer      : Using serverListUpdater PollingServerListUpdater
2022-10-13 16:23:10.043  INFO 19580 --- [p-nio-80-exec-1] c.netflix.config.ChainedDynamicProperty  : Flipping property: feign-provider.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
2022-10-13 16:23:10.044  INFO 19580 --- [p-nio-80-exec-1] c.n.l.DynamicServerListLoadBalancer      : DynamicServerListLoadBalancer for client feign-provider initialized: DynamicServerListLoadBalancer:{NFLoadBalancer:name=feign-provider,current list of Servers=[192.168.168.1:8090],Load balancer stats=Zone stats: {unknown=[Zone:unknown;	Instance count:1;	Active connections count: 0;	Circuit breaker tripped count: 0;	Active connections per server: 0.0;]
},Server stats: [[Server:192.168.168.1:8090;	Zone:UNKNOWN;	Total Requests:0;	Successive connection failure:0;	Total blackout seconds:0;	Last connection made:Thu Jan 01 08:00:00 CST 1970;	First connection made: Thu Jan 01 08:00:00 CST 1970;	Active Connections:0;	total failure count in last (1000) msecs:0;	average resp time:0.0;	90 percentile resp time:0.0;	95 percentile resp time:0.0;	min resp time:0.0;	max resp time:0.0;	stddev resp time:0.0]
]}ServerList:com.alibaba.cloud.nacos.ribbon.NacosServerList@3352a852
2022-10-13 16:23:10.991  INFO 19580 --- [erListUpdater-0] c.netflix.config.ChainedDynamicProperty  : Flipping property: feign-provider.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
2022-10-13 16:23:12.148 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] <--- HTTP/1.1 200  (2273ms)
2022-10-13 16:23:12.148 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] connection: keep-alive
2022-10-13 16:23:12.148 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] content-type: application/json
2022-10-13 16:23:12.148 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] date: Thu, 13 Oct 2022 08:23:12 GMT
2022-10-13 16:23:12.148 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] keep-alive: timeout=60
2022-10-13 16:23:12.148 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] transfer-encoding: chunked
2022-10-13 16:23:12.148 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] 
2022-10-13 16:23:12.149 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] {"id":2,"name":"admin-1","age":18}
2022-10-13 16:23:12.149 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign          : [UserFeign#getUserById] <--- END HTTP (34-byte body)
```

### 2. GZIP压缩

<p class="note note-primary">HTTP 协议中的数据压缩</p>

[数据压缩](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Compression)是提高 Web 站点性能的一种重要手段。对于有些文件来说，高达 70% 的压缩比率可以大大减低对于带宽的需求。随着时间的推移，压缩算法的效率也越来越高，同时也有新的压缩算法被发明出来，应用在客户端与服务器端。

在实际应用时，web 开发者不需要亲手实现压缩机制，浏览器及服务器都已经将其实现了，不过他们需要确保在服务器端进行了合理的配置。数据压缩会在三个不同的层面发挥作用：

- 首先某些格式的文件会采用特定的优化算法进行压缩，（GZIP压缩算法[deflate](https://blog.csdn.net/qq_42139383/article/details/115064562?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166565721916781432959945%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166565721916781432959945&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-115064562-null-null.142^v56^pc_rank_34_queryrelevant25,201^v3^control&utm_term=deflate&spm=1018.2226.3001.4187)）
- 其次在 HTTP 协议层面会进行通用数据加密，即数据资源会以压缩的形式进行端到端传输，
- 最后数据压缩还会发生在网络连接层面，即发生在 HTTP 连接的两个节点之间。

<p class="note note-primary">Feign GZIP压缩配置</p>

feign_consumer中的application.yml

```yaml
server:
  port: 80
  compression:
    enabled: true #开启浏览器到consumer层的gzip压缩
feign:
  compression:
    request:		# 请求开启
      enabled: true # 开启feign到provide层gzip压缩
    response:		# 响应开启
      enabled: true
```

<p class="note note-primary">Feign GZIP压缩验证</p>

```shell
2022-10-13 16:23:09.873 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign: [UserFeign#getUserById] Accept-Encoding: gzip
2022-10-13 16:23:09.873 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign: [UserFeign#getUserById] Accept-Encoding: deflate
2022-10-13 16:23:09.873 DEBUG 19580 --- [p-nio-80-exec-1] com.bjpowernode.fegin.UserFeign: [UserFeign#getUserById] ---> END HTTP (0-byte body)
```

### 3. HTTP池连接开启

HTTP连接需要的 3 次握手 4 次分手开销大，Feign 在默认情况下使用的是 JDK 原生的 URLConnection 发送HTTP请求，并不支持连接池；因为优化方案选择[Apache HttpClient](https://hc.apache.org/httpcomponents-client-5.2.x/index.html) 

- 灵活的连接管理和池化。
- 支持 HTTP 响应缓存。

feign_consumer pom.xml

```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

HTTPClient在Feign中默认开启并且配置默认参数，故只需引入依赖即可

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221013185707720.png)

### 4. Feign超时处理

#### 1. 模拟超时

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

#### 2. 测试

```shell
 ERROR 20032 --- [p-nio-80-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is feign.RetryableException: Read timed out executing GET http://feign-provider/provider/getUserById/11111] with root cause

java.net.SocketTimeoutException: Read timed out
```

#### 3.设置Feign超时处理

##### 方式一：Feign超时处理（推荐）

在feign_consumer的application.yml中加入

```yaml
feign:
  client:
    config:
      default: #设置default 或者是服务注册名
        connect-timeout: 5000 # 请求链接超时时间；防止由于服务器处理时间长而阻塞调用者。
        read-timeout: 5000 	  #请求处理超时时间；从连接建立时开始应用，在返回响应时间过长时触发。
```

[Feign Timeout Handling参考](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#timeout-handling)

##### 方式二：Ribbon中超时处理

```yaml
ribbon:
  ConnectTimeout: 5000 #请求连接的超时时间
  ReadTimeout: 5000 #请求处理的超时时间
```

[Ribbon Timeout Handling参考](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/1.4.0.RELEASE/single/spring-cloud-netflix.html)

两者同时配置，方式一生效；[参考](https://github.com/spring-cloud/spring-cloud-netflix/issues/696)

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

