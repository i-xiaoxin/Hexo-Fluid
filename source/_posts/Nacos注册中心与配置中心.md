---
title: Nacos注册中心与配置中心
date: 2022-10-08 21:29:02
tags: 
- 微服务
- Spring Cloud Alibaba
- Nacos
- Nginx
index_img: 
banner_img: /img/article8.webp
categories:
- 微服务
- Spring Cloud Alibaba
comment: waline
---

### 一、什么是 Nacos

<p class="note note-success">Nacos /nɑ:kəʊs/ 是 Dynamic Naming and Configuration Service的首字母简称，一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。（配置中心和注册中心）</p>

### 二、配置中心基础概述

#### 1.配置

在系统开发过程中，开发者通常会将一些需要变更的参数、变量等从代码中分离出来独立管理，以独立的配置文件的形式存在。目的是让静态的系统工件或者交付物（如 WAR，JAR 包等）更好地和实际的物理运行环境进行适配。配置管理一般包含在系统部署的过程中，由系统管理员或者运维人员完成。配置变更是调整系统运行时的行为的有效手段。

#### 2.为什么要用配置中心？

- **配置实时生效**：传统的静态配置方式要想修改某个配置只能修改之后重新发布应用，要实现动态性，可以选择使用数据库，通过定时轮询访问数据库来感知配置的变化。轮询频率低感知配置变化的延时就长，轮询频率高，感知配置变化的延时就短，但比较损耗性能，需要在实时性和性能之间做折中。**配置中心专门针对这个业务场景，兼顾实时性和一致性来管理动态配置**；
- **配置管理流程**：配置的权限管控、灰度发布、版本管理、格式检验和安全配置等一系列的配置管理相关的特性，也是配置中心不可获取的一部分；
- **分布式场景**：随着采用分布式的开发模式，项目之间的相互引用随着服务的不断增多，相互之间的调用复杂度成指数升高，每次投产或者上线新的项目时苦不堪言，需要引用配置中心治理

#### 3.配置中心支持功能

- **灰度发布**：配置的灰度发布是配置中心比较重要的功能，当配置的变更影响比较大的时候，需要先在部分应用实例中验证配置的变更是否符合预期，然后再推送到所有应用实例。
- **权限管理**：配置的变更和代码变更都是对应用运行逻辑的改变，重要的配置变更常常会带来核弹的效果，对于配置变更的权限管控和审计能力同样是配置中心重要的功能。
- **版本管理&回滚**：当配置变更不符合预期的时候，需要根据配置的发布版本进行回滚。
- **配置格式校验**：应用的配置数据存储在配置中心一般都会以一种配置格式存储，比如Properties、Json、Yaml等，如果配置格式错误，会导致客户端解析配置失败引起生产故障，配置中心对配置的格式校验能够有效防止人为错误操作的发生，是配置中心核心功能中的刚需。
- **监听查询**：当排查问题或者进行统计的时候，需要知道一个配置被哪些应用实例使用到，以及一个实例使用到了哪些配置。
- **多环境**：在实际生产中，配置中心常常需要涉及多环境或者多集群，业务在开发的时候可以将开发环境和生产环境分开，或者根据不同的业务线存在多个生产环境。如果各个环境之间的相互影响比较小（开发环境影响到生产环境稳定性），配置中心可以通过逻辑隔离的方式支持多环境。
- **多集群**：当对稳定性要求比较高，不允许各个环境相互影响的时候，需要将多个环境通过多集群的方式进行物理隔离。

### 三、Nacos安装和启动

#### <div>
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

> 从github上选择需要的版本进行[下载](https://github.com/alibaba/nacos/releases)

![image-20221009162640058](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221009162640058.png)

#### 2.解压

```shell
cd /usr/upload
tar -zxvf nacos-server-1.4.1.tar.gz -C /usr/local
```

#### 3.启动和关闭

```shell
cd /usr/local/nacos/bin
#启动
./startup.sh -m standalone #非集群模式启动

#关闭
./shutdown.sh
```

#### 4.测试

浏览器访问：http://host:8848/nacos，出现登录界面即启动成功；默认用户名/密码为： nacos/nacos

### 四、Nacos注册中心

### 1.什么是注册中心

![image-20221009163625094](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221009163625094.png)

![image-20221009163709444](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221009163709444.png)

注册中心主要有三部分组成：

- **Nacos-Server**：注册中心

​    提供服务的注册和发现。

- **Nacos-Provider**：服务提供方

​    把自身的服务实例注册到 Nacos Server 中

- **Nacos-Consumer**：服务调用方

​    通过 Nacos Server 获取服务列表，消费服务。

### 2.Nacos注册中心使用

#### 2.1创建服务提供者nacos_provider

##### 	<1>pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud_parent</artifactId>
        <groupId>com.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>nacos_provider</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>springcloud_common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--nacos客户端-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
</project>
```

##### <2>application.yml

```yaml
server:
  port: 8090
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.211.131:8848 #注册中心的地址
  application:
    name: nacos-provider #注册到nacos的服务名
```

##### <3>java

conrtoller

```java
import com.example.pojo.User;
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

service

```java
import com.example.pojo.User;

public interface UserService {
    User getUserById(Integer id);
}

```

```java
import com.example.pojo.User;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService{

    @Override
    public User getUserById(Integer id){
        return new User(id, "admin", 18);
    }
}

```

SpringBootApplication

```java

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient //注册自己并发现其他服务
public class NacosProviderApp {
    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApp.class, args);
    }
}
```

#### 2.3创建服务消费者nacos_consumer

##### <1>pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud_parent</artifactId>
        <groupId>com.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>nacos_consumer</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>springcloud_common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
</project>
```

##### <2>application.yml

```yaml
server:
  port: 80
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.211.131:8848 #注册中心的地址
  application:
    name: nacos-consumer #注册到nacos的服务名
```

##### <3>java

config

```java

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class BeanConfig {

    //RestTemplate：是spring提供的一个工具类，作用是发送restful请求
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```



controller

```java
import com.example.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.List;

@RestController
@RequestMapping("/consumer")
public class UserController {

    @Autowired
    private RestTemplate restTemplate;
    //springcloud提供的工具类，作用：发现服务
    @Autowired
    private DiscoveryClient discoveryClient;

    @RequestMapping("/getUserById/{id}")
    public User getUserById(@PathVariable Integer id){
        //获得所有的服务名
        List<String> serviceList = discoveryClient.getServices();
        for (String serviceName : serviceList) {
            System.out.println(serviceName);
        }

        //调用nacos_provider服务
        //缺点：1、ip和port硬编码    2、不能实现负载均衡
        //String url = "http://127.0.0.1:8090/provider/getUserById/"+id;

        //缺点：2、不能实现负载均衡
        ServiceInstance service = discoveryClient.getInstances("nacos-provider").get(0);
        String url = "http://"+ service.getHost() +":"+ service.getPort() +"/provider/getUserById/"+id;
        return restTemplate.getForObject(url, User.class);
    }
}

```

### 3.Nacos配置中心使用

#### 3.1创建配置中心nacos_config

##### 	<1>pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud_parent</artifactId>
        <groupId>com.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>nacos_config</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
</project>
```

##### <2>bootstrap.yml

> 客户端配置文件的名称必须为`bootstrap.yml`
>
> - `bootstrap.yml`比 `applicaton.yml` 优先加载，应用于系统级别参数配置，一般不会变动；
> - `application.yml`应用于SpringBoot项目的自动化配置；

```yaml
spring:
  cloud:
    nacos:
      config:
        server-addr: 192.168.211.131:8848 #注册中心的地址#配置文件的前缀
        prefix: nacos-config #配置文件的前缀，默认是spring.application.name
        file-extension: yaml #配置文件的后缀，默认是properties
#Data ID的语法：${spring.cloud.nacos.config.prefix}.${spring.cloud.nacos.config.file-extension}
```

##### <3>java

controller

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope //重新从BeanFactory获取一个新的实例（该实例使用新的配置）
public class ConfigController {

    @Value("${spring.datasource.driver-class-name}")
    private String driverClassName;
    @Value("${spring.datasource.url}")
    private String url;
    @Value("${spring.datasource.username}")
    private String username;
    @Value("${spring.datasource.password}")
    private String password;
    @Value("${spring.datasource.type}")
    private String type;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        System.out.println(this);
        String configInfo = driverClassName+"<br>"+url+"<br>"+username+"<br>"
                +password+"<br>"+type;
        return configInfo;
    }
}
```

SpringBootApplication

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class NacosConfigApp {

    public static void main(String[] args) {
        SpringApplication.run(NacosConfigApp.class,args);
    }
}
```

#### 3.2在nacos中新建配置文件

```yaml
server:
  port: 80
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.211.131:8848
  application:
    name: nacos-config
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.211.131:3306/health?characterEncoding=UTF-8
    username: root
    password: 1111
    type: com.alibaba.druid.pool.DruidDataSource
```

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221009212733188.png)

#### 3.3测试

1.启动时加载配置文件

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221009213211549.png)

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221009213226665.png)

2.修改配置文件后nacos监听到MD5有变化则推送消息给客户端，客户端收到消息后会拉取最新配置（参考
`配置管理->监听查询`菜单）

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221009213243110.png)

3.浏览器访问：http://127.0.0.1/config/info

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221009213256584.png)

#### 3.4Nacos配置管理模型

Nacos配置管理，通过Namespace、group、Data ID能够定位到一个配置集。

Namespace Group DataId介绍：

- Namespace: 代表不同的环境的配置隔离, 如: 开发、测试， 生产等
- Group: 可以代表某个项目, 如XX医疗项目, XX电商项目
- DataId: 每个项目下往往有若干个工程, 每个配置集(DataId)是一个工程的主配置文件

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221010165507276.png)

获取配置集需要指定：

1. nacos服务地址，必须指定
2. namespace，如不指定默认public
3. group，如不指定默认 DEFAULT_GROUP
4. dataId，必须指定

> Nacos配置隔离先在配置中对以上参数进行设置，通过对配置集指定参数进行配置隔离

#### 3.5服务隔离

nacos_provider：

```yaml
server:
  port: 8090
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 47.98.105.36:8848
        namespace: test #开发环境
        group: nacos_group #项目组  provide与consumer同项目保持一致
  application:
    name: nacos-provider
```

nacos_consumer：

```yaml
server:
  port: 80
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 47.98.105.36:8848
        namespace: test #开发环境
        group: nacos_group #项目组  provide与consumer同项目保持一致
  application:
    name: nacos-consumer
```

### 五、Nacos持久化

#### 1.为什么要持久化？

​	Nacos默认有自带嵌入式数据库derby，但是如果做集群模式的话，就不能使用自己的数据库不然每个节点一个数据库，那么数据就不统一了，需要使用外部的mysql

#### 2.持久化配置步骤

##### 2.1Nacos application.properties

进入application.properties文件vim

```properties
#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
 db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=10000&socketTimeout=30000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
 db.user.0=root
 db.password.0=12345678

### Connection pool configuration: hikariCP
db.pool.config.connectionTimeout=30000
db.pool.config.validationTimeout=10000
db.pool.config.maximumPoolSize=20
db.pool.config.minimumIdle=2

```

##### 2.2Mysql数据库中配置

Mysql中创建nacos数据库并将自带的`nacos-mysql.sql`导入即可

##### 2.3测试

启动Nacos创建配置，Mysql表config_info中数据同步增加即配置成功

### 六、Nacos集群搭建

#### 1.集群部署架构图

<img src="https://raw.githubusercontent.com/i-xiaoxin/image/master/20221010193110.png" style="zoom:25%;" />

#### 2.节点规划

|     节点     | 端口 |
| :----------: | :--: |
| 47.98.105.36 | 8848 |
| 47.98.105.36 | 8849 |
| 47.98.105.36 | 8847 |

#### 3.集群搭建

##### 3.1配置cluster.conf文件

```shell
cd /usr/local/nacos/conf/
cp cluster.conf.example cluster.conf
vim cluster.conf

#it is ip
#example
47.98.105.36:8847
47.98.105.36:8848
47.98.105.36:8849
                
```

##### 3.2复制3个Nacos

```shell
cd /usr/local
mkdir nacos_cluster
cp -r nacos nacos_cluster/nacos_8847
cp -r nacos nacos_cluster/nacos_8848
cp -r nacos nacos_cluster/nacos_8849
```

##### 3.3分别配置port

```shell
cd nacos_8847/conf/
vim application.properties
server.port=8847


cd nacos_8849/conf/
vim application.properties
server.port=8849

```

##### 3.4启动

```shell
cd /usr/local/nacos_cluster/nacos_8849/bin
./startup.sh
```

#### 4.搭建nginx

##### 4.1安装C语言环境

```shell	
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

##### 4.2下载nginx

```shell
wget -c https://nginx.org/download/nginx-1.12.0.tar.gz
```

##### 4.3解压

```sheell
tar -zxvf nginx-1.12.0.tar.gz
```

##### 4.4配置安装路径

```shell
./configure --prefix=/usr/local/nginx
```

##### 4.5编译并安装

```shell
make && make install
```

##### 4.6启动和关闭

```shell
 启动：./nginx
 关闭：./nginx -s stop
```

#### 5.配置nginx代理nacos集群

```shell
 vim /usr/local/nginx/conf/nginx.conf
 
 #gzip  on;
    upstream nacosList {
                server 47.98.105.36:8847;
                server 47.98.105.36:8848;
                server 47.98.105.36:8849;
                }
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass http://nacosList;
        }

#注意nacosList变量命名规范
```

启动nginx服务，访问http://47.98.105.36/nacos/；nginx会自动分配其中一个进行调用

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221010192857504.png)

```yanml
server:
  port: 8090
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 47.98.105.36 	
        #47.98.105.36:8848 配置Nginx后即可省略端口号由Nginx进行代理分配
        namespace: test #开发环境
        group: nacos_group #项目组
  application:
    name: nacos-provider
```

Nacos部署集群后服务地址省略端口号由Nginx进行代理分配

### 七、Nacos开机自启动配置*

#### 1.编写开机启动文件

- 添加nacos.service文件

```sh
vim /lib/systemd/system/nacos.service
```

- 文件内容如下：

```shell
[Unit]
Description=nacos
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nacos/bin/startup.sh -m standalone
ExecReload=/usr/local/nacos/bin/shutdown.sh
ExecStop=/usr/local/nacos/bin/shutdown.sh
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

#### 2.修改nacos的startup.sh

- 修改JAVA_HOME路径并注销之后的3行配置，如下：

```sh
vim /usr/local/nacos/bin/start.sh

#找到 JAVA_HOME 设置绝对路径
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/local/jdk1.8.0_191 
#[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
#[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/opt/taobao/java
#[ ! -e "$JAVA_HOME/bin/java" ] && unset JAVA_HOME
```

#### 3.设置开机启动

```sh
systemctl daemon-reload        #重新加载服务配置
systemctl enable nacos.service   #设置为开机启动
systemctl start nacos.service    #启动nacos服务
systemctl stop nacos.service    #停止nacos服务
systemctl status nacos.service   #查询nacos服务状态
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

