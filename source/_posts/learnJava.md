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

> 将密码通过md5加密存入数据库
>
> 登陆时,将用户输入的密码进行MD5加密,与数据库中的值进行比较

### 4.mybatis分页插件

> 原生分页
>
> 按价格排序
>
> 第一页按价格升序,之后第二页价格排序的结果和之前那一页商品价格没有完全对应上
>
> 前一页和后一页的商品价格没有对应上

### 5.模糊查询

```sql
/*mapper.xml文件中要这样写*/

<if test="paramsMap.keywords != null">
                and i.item_name like '%${paramsMap.keywords}$%'
</if>
```

mapper.xml 多条件查询

```XML
		order by
        <!--k: 默认,代表默认排序,根据name 
        c: 根据销量排序
        p: 根据价格排序
        -->
	&quot; &quot; 
	< 转义'' 符号 , 有可能报错/>
		<choose>
                
          <when test="paramsMap.sort == &quot;c&quot;">
              i.sell_counts desc
          </when>
             <when test="paramsMap.sort == &quot;p&quot;">
                tempSpec.priceDiscount asc
           </when>
           <otherwise>
               i.item_name asc
           </otherwise>
     </choose>
	<!--
      <choose>
                
          <when test="paramsMap.sort == 'c'">
              i.sell_counts desc
          </when>
             <when test="paramsMap.sort == 'p'">
                tempSpec.priceDiscount asc
           </when>
           <otherwise>
               i.item_name asc
           </otherwise>
     </choose> -->
```



### 6. xml文件 " 号的转义

> xml文件中 "" 转义

```XML
<when test="paramsMap.sort == &quot;c&quot;">
    i.sell_counts desc
</when>
```

> 这样写会报错
>
> java.lang.NumberFormatException: For input string: "k"



```XML
<when test="paramsMap.sort == 'c' ">
    i.sell_counts desc
</when>
```

> 模糊查询时的书写 用$符号取值 而不是 # 号

```XML
<if test="paramsMap.keywords != null and paramsMap.keywords != '' ">
   and i.item_name like '%${paramsMap.keywords}%'
</if>
```





> mybatis-pagehelper

1. 引入分页插件依赖

	```XML
	<!--mybatis-pagehelper 实现分页 -->
	<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper-spring-boot-starter</artifactId>
	<version>1.2.12</version>
	</dependency>
	```

2. 配置application.yml配置文件

	```YML
	pagehelper:
		helperDialect: mysql
		supportMethodsArguments: true
	```

3. 用分页插件，在查询前使用分页插件，原理：统一拦截sql，为其提供分页功能

	```JAVA
	/**
	* page: 第几页
	* pageSize: 每页显示条数
	*/
	PageHelper.startPage(page, pageSize);
	```

4. 页数据封装到 PagedGridResult.java 传给前端

	```JAVA
	PageInfo<?> pageList = new PageInfo<>(list);
	PagedGridResult grid = new PagedGridResult();
	grid.setPage(page);
	grid.setRows(list);
	grid.setTotal(pageList.getPages());
	grid.setRecords(pageList.getTotal());
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



## BaseController

> 定义分页中每页显示数据的大小
>
> 其它Controller继承BaseController即可









## 涉及到金额

> 使用分为单位 9.98元   998分
>
> int类型









## sql语句

```sql
`` 和 '' 使用
varchar类型变量的使用
```





## Git使用





## 购物车功能

1. 购物车存储形式 - Cookie
	- 无需登录,无需查库,保存在浏览器端
	- 优点：性能好,访问快,没有和数据库交互
	- 缺点1：换电脑购物车数据会丢失
	- 缺点2：电脑被其他人登录,不安全
2. 购物车存储形式-Session
	- 用户登录后,购物车数据放入用户会话
	- 优点：初期性能较好,访问快
	- 缺点1：session基于内存,用户量庞大影响服务器性能
	- 缺点2：只能存在当前会话,不适用集群与分布式系统
3. 购物车存储形式-数据库
	- 用户登录后,购物车数据存入数据库
	- 优点：数据持久化，可在任何地点任何时间访问
	- 缺点：频繁读写数据库,造成数据库压力
4. 购物车存储形式 - Redis
	- 用户登录后，购物车数据存入redis缓存
	- 优点1：数据持久化，可在任何地点任何时间访问
	- 优点2：频繁读写基于缓存,不会造成数据库压力
	- 优点3：适合使用集群和分布式系统,可扩展性强 

