---
title: learnJava
date: 2021-10-29 23:19:42
tags: Java
categories:
- Java
---

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

