---
title: learnJava
date: 2021-10-29 23:19:42
tags: Java
categories:
- Java
index_img: /img/spring-boot.jpg
---

Java

<!--more-->

## mybatis

### 1.mybatis逆向生成工具

mybatis通用生成工具

maven依赖

```XML
<!-- 通用mapper逆向工具 -->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>2.1.5</version>
        </dependency>
```

可以借助mybatis逆序生成工具 生成pojo类 mapper文件

### 2.使用包下的类来简化查询

```JAVA
Example userExample = new Example(Users.class);
Example.Criteria userCriteria = userExample.createCriteria();
userCriteria.andEqualTo("username", username);
Users result = usersMapper.selectOneByExample(userExample);
```

### 3.数据脱敏

```java
MD5Utils.getMD5(password);
```

## 跨域

![image-20211029230023380](learnJava.assets/image-20211029230023380.png)

前端端口 8080 到 后端端口8088 不一致 , 会产生跨域问题 ,

写一个配置类 注册到spring

```java
package com.tho.config;


import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

// 配置跨域问题
@Configuration
public class CorsConfig {

    public CorsConfig() {

    }
    /**
    * @Author tho
    * @Date 2021/10/29 23:28
    * @param
    * @Return CorsFilter
    * @Description: 配置跨域
    */
    @Bean
    public CorsFilter corsFilter() {
        // 1.添加cors配置信息
        CorsConfiguration config = new CorsConfiguration();

        config.addAllowedOrigin("http://localhost:8080");

        // 设置是否发送 cookie信息
        config.setAllowCredentials(true);

        // 设置允许请求的方式
        config.addAllowedMethod("*");

        // 设置允许的header
        config.addAllowedHeader("*");

        // 2.为url添加映射路径
        UrlBasedCorsConfigurationSource corsSource = new UrlBasedCorsConfigurationSource();
        corsSource.registerCorsConfiguration ("/**", config);

        // 3.返回重新定义好的corsSource
        return new CorsFilter(corsSource);

    }
}
```

## cookie&session

> cookie 
>
> - 以键值对的形式存储信息在浏览器
>
> - cookie不能跨域,当前及其父级域名可以取值
> - cookie可以设置有效期
> - cookie可以设置path

> session
>
> - 基于服务器内存的缓存(非持久化),可保持请求会话
> - 每个session通过sessionid来区分不同请求
> - session可以设置过期时间
> - session也是以键值对存在的

## springboot配置日志

依赖

- springboot-starter 和 springboot-web都要屏蔽自带的日志框架

```XML
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <!--使用自定义日志框架 屏蔽自带的日志框架-->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!--使用自定义日志框架 屏蔽自带的日志框架-->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
            <!-- 打包war [2] 移除自带内置tomcat -->
            <!--<exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>-->
        </dependency>
```

- 为slf4j添加配置文件

log4j.properties

```properties
log4j.rootLogger=DEBUG,stdout,file
log4j.additivity.org.apache=true
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.threshold=INFO
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%-5p %c{1}:%L - %m%n
log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.DatePattern='.'yyyy-MM-dd-HH-mm
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
log4j.appender.file.Threshold=INFO
log4j.appender.file.append=true
log4j.appender.file.File=/workspaces/logs/foodie-api/mylog.log
```



## sql语句

```sql
`` 和 '' 使用
varchar类型变量的使用
```





## Git使用

