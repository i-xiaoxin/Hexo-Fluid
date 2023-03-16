---
title: SpringBoot中yml占位符@*@与${*}
date: 2023-02-09 21:19:02
tags: 
- SpringBoot
index_img: 
banner_img: /img/article6.webp
categories:
- SpringBoot
comment: waline
---

# 前言

在 SpringBoot 项目yml配置中，我们经常会使用两种占位符，它们分别是：

- @*@ 
- ${*}

代码案例

```yaml
spring:
  application:
    name: @artifactId@ #占位符1
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_HOST:127.0.0.1}:${NACOS_PORT:8848} #占位符2

      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: yml
        shared-configs:
          - application-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
```

# ${*} 

使用 @Value 注解注入属性时，只能使用 ${*} 占位符解析。*

@Value 注解的处理属于 Spring 核心框架逻辑，可以参见 `PlaceholderConfigurerSupport` 这个类，最终会执行 ${*} 占位符的解析。其中的冒号后面可以写默认值。

```java
/**
* PlaceholderConfigurerSupport部分关键代码
*/
public abstract class PlaceholderConfigurerSupport extends PropertyResourceConfigurer
		implements BeanNameAware, BeanFactoryAware {

	/** Default placeholder prefix: {@value}. */
	public static final String DEFAULT_PLACEHOLDER_PREFIX = "${";

	/** Default placeholder suffix: {@value}. */
	public static final String DEFAULT_PLACEHOLDER_SUFFIX = "}";

	/** Default value separator: {@value}. */
	public static final String DEFAULT_VALUE_SEPARATOR = ":";
}
```

# maven-resources-plugin 

替换资源文件中的占位符变量

对于Maven标准的占位符变量，如${project.groupId}，${project.artifactId}，${project.version}，${project.basedir}等，只要在配置<resource>的时候，设置<filtering>true</filtering>即可，如下：

```xml
  <build>
     <resources>
        <resource>
            <directory>src/main/resources</directory>
                <includes>
                  <include>**/*.txt</include>
                </includes>
                <excludes>
                  <exclude>**/*test*.*</exclude>
                </excludes>
             <filtering>true</filtering>
         </resource>
    </resources>
  </build>
```

- 默认占位符有两种，分别是 ${*} 和 @*@
- 配置项 useDefaultDelimiters，可以控制是否使用默认占位符
- 配置项 delimiter，既可以写默认占位符，也可以自定义占位符

# 总结

- 默认占位符有两种，分别是 ${ * } 和 @ * @
- 配置项 useDefaultDelimiters，可以控制是否使用默认占位符。如果为 true，则 ${*} 和 @*@ 这两种占位符始终有效，可以同时使用
- 配置项 delimiter，既可以写默认占位符，也可以自定义占位符，比如上文中的 # * #

**注意事项：**

- 占位符必须成对使用，如果忘记写右边的，则不会被解析。
- 本文搭建的 Demo 项目，使用了 spring-boot-starter-parent 作为 parent，但有时我们可能不会使用它。此时，maven-resources-plugin 插件需要我们手动引入，道理是一样的。

**常见情况：**

- 如果项目直接或间接引入 spring-boot-starter-parent 作为 parent，且没有手动配置 maven-resources-plugin 插件。则只能使用 @*@ 这一种占位符，这是在 spring-boot-starter-parent 指定的。
- 如果项目没有引入 spring-boot-starter-parent 作为 parent，手动引入 maven-resources-plugin 插件，但没有指定任何 delimiter，也没有显式配置 useDefaultDelimiters 为 false，那么可以使用默认占位符 @*@ 或 ${*}，因为不配置 useDefaultDelimiters 的话，默认为 true。



参考：[聊聊 SpringBoot 中的两种占位符：@*@ 和 ${*} ](https://www.cnblogs.com/xiaoxi666/p/15676529.html)



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