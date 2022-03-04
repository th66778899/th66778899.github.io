---
title: SpringCloud
date: 2022-2-1 12:36:26
tags: SpringCloud
categories:
- Spirng
index_img: /img/springCloud.png

---

SpringCloud基础组件

<!--more-->

# 一、SpringCloud简介

- SpringCloud 和 微服务之间的关系
- SpringCloud 整体架构
- SpringCloud组件库(产地来自三个地方) Netflix Alibaba
- SpringCloud 更新很频繁，选择
- 电商系统微服务化的构想
- 开发环境和依赖组件的版本

## 1.1、微服务拆分

1. 拆分依赖项 每个pom都只引入自己使用到的依赖

2. 微服务模块拆分粒度始终
3. 公共组件拆离
4. 平台中间件剥离(注册中心，配置中心)

# 二、SpringCloudNetflix

## 2.1、服务治理(Eureka)

- 概念 技术选型

- 服务治理全链路

	服务注册，服务发现，心跳和续约，服务下线，剔除和自保

- 源码解读 - 服务注册，心跳检测和服务续约

- 注册中心高可用改造

- 实现

### 1.服务治理 - 技术选型

分布式系统CAP定理

> 一致性：各个分布式节点数据一致
>
> - 强一致性：某次更新之后，所有的请求都可以访问到更新的值
> - 弱一致性：某次更新之后，有的请求或者所有请求都拿不到更新的值
> - 最终一致性：未来的某一时刻，所有请求都可以拿到更新的值

C - 强一致性

A - 可用性

P - 分区容错性

> CAP定理：分布式系统只能三选二 且 分区容错性必须要满足，因为分布式系统不可能只有一个服务器



> 三大服务治理
>
> - Eureka 老牌劲旅(Netflix)
> - Consul 同门师弟(官方)
> - Nacos 后起之秀(Alibaba)

### 2.Eureka2.0开源计划搁置

### 3.搭建注册中心demo

- 创建Demo顶层pom 和 子项目 eureka-server
- 添加Eureka依赖
- 设置启动类
- Start

### 4.解读注册中心UI界面

### 5.服务注册

### 6.服务注册源码解读

> - 重要的注解
> - 代理模式，装饰器模式
> - 服务注册都注册什么内容

### 7.创建服务消费者

### 8.服务心跳和续约源码

### 9.启用心跳和健康度检查

### 10.注册中心高可用

## 2.2、负载均衡(Ribbon)

### 1.负载均衡介绍

> - 概念 体系架构 技术选型
> - 深入Ribbon
> 	- 负载均衡策略和原理，加载方式，IPing机制
> 	- 源码阅读 LoadBalanced 底层机制 & 可扩展性
> 	- 架构探讨 - 如何选择负载均衡策略
> 	- Demo  +  造轮子



> 理解负载均衡
>
> - 雨露均沾
> - 不能让某台机器负载过高
> - 如果请求太大，所有机器要都承担较高负载

### 2.客户端负载均衡&服务端负载均衡

- 客户端负载均衡

> 客户端从Eureka 中获取服务的机器列表，根据一定的负载均衡策略，选择访问某个服务

- 服务端负载均衡

> 客户端 借助 nginx(软件) 或者 F5(软件) 实现负载均衡，客户端



> 大型应用通常是 客户端 + 服务端 负载均衡搭配使用



> 两种负载均衡策略对比

![image-20220130171732376](/img/springCloud.assets/image-20220130171732376.png)

### 3.添加负载均衡功能

- 创建ribbon-consumer
- 添加依赖，调用eureka-client
- 启动多个eureka-client
- 将负载均衡策略应用到全局或指定服务

> Ribbon懒加载机制，Ribbon在第一次方法调用的时候才去初始化LoadBalancer，这样看来，第一个方法不仅包含HTTP连接和方法的响应时间，还包括了LoadBalancer的创建耗时，假如方法本身比较耗时的话，而且超时时间又设置地比较短，那么很大可能这第一次http调用就会失败
>
> - 通过配置关闭懒加载
>
> ```properties
> ribbon.eager-load.enabled=true
> ribbon.eager-load.clients=ribbon-consumer
> ```
>
> - 第一个参数开启了Ribbon的饥饿加载模式
> - 第二个参数指定了需要应用饥饿加载的服务名称

### 4.负载均衡配置

- 全局负载均衡配置
- 指定服务负载均衡配置

> 1. application.properties 配置文件配置 
>
> ```properties
> eureka-client.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule
> ```
>
> 2. 指定加载Bean
>
> ```java
> package com.tho.springcloud;
> 
> import com.netflix.loadbalancer.IRule;
> import com.netflix.loadbalancer.RandomRule;
> import org.springframework.cloud.netflix.ribbon.RibbonClient;
> import org.springframework.context.annotation.Bean;
> import org.springframework.context.annotation.Configuration;
> 
> /**
>  * @Author tho
>  * @Date 2022/1/31/14:08
>  * @ProjectName spring-cloud-netflix-demo
>  * @ClassName: RibbonConfiguration
>  * @Description: TODO
>  */
> @Configuration
> public class RibbonConfiguration {
> 
>     @Bean
>     public IRule defaultLBStrategy() {
>         return new RandomRule();
>     }
> }
> ```
>
> 3. 通过注解 实现
>
> ```java
> package com.tho.springcloud;
> 
> import org.springframework.cloud.netflix.ribbon.RibbonClient;
> import org.springframework.context.annotation.Configuration;
> 
> @Configuration
> @RibbonClient(name = "eureka-client", configuration = com.netflix.loadbalancer.RandomRule.class)
> public class RibbonConfiguration {
> 
> }
> ```

- 注解配置的负载均衡策略优先于配置文件配置的负载均衡 ，可能和资源文件加载顺序有关

### 5.负载均衡策略解析(源码解析)

- 熟悉7种负载均衡策略
- 自旋锁使用方式
- 防御性编程

### 6.LoadBalanced 注解

- LoadBalanced 作用原理
- 拦截器到IRule的调用链路
- IPing机制 

### 7.IPing机制解析



### 8.IRule机制可扩展研究



### 9.造轮子 - 自定义IRule

- 自定义基于一致性哈希负载均衡策略

```java
package com.tho.springcloud.rule;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.Server;
import lombok.NoArgsConstructor;
import org.springframework.util.CollectionUtils;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.List;
import java.util.SortedMap;
import java.util.TreeMap;

/**
 * @Author tho
 * @Date 2022/2/2/13:32
 * @ProjectName spring-cloud-netflix-demo
 * @ClassName: MyRule
 * @Description: TODO
 */
@NoArgsConstructor
public class MyRule extends AbstractLoadBalancerRule implements IRule {
    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {

    }

    @Override
    public Server choose(Object key) {
        HttpServletRequest request = ((ServletRequestAttributes)
                RequestContextHolder.getRequestAttributes())
                .getRequest();
        String uri = request.getServletPath() + "?" + request.getQueryString();


        return route(uri.hashCode(), getLoadBalancer().getAllServers());
    }

    public Server route(int hashId, List<Server> addressList) {

        if (CollectionUtils.isEmpty(addressList)) {
            return null;
        }
        TreeMap<Long, Server> address = new TreeMap<>();
        addressList.stream().forEach(e -> {
            // 虚化若干个服务节点，到环上
            for (int i = 0; i < 8; i++) {
                long hash = hash(e.getId() + i);
                address.put(hash, e);
            }
        });
        long hash = hash(String.valueOf(hashId));
        SortedMap<Long, Server> last = address.tailMap(hash);
        // 当request URL hash值大于任意一个服务器对应的hashKey
        // 取address中的第一个节点

        if (last.isEmpty()) {
            address.firstEntry().getValue();
        }


        return last.get(last.firstKey());
    }

    public long hash(String key) {

        MessageDigest md5;


        try {
            md5 = MessageDigest.getInstance("md5");
        }  catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }

        byte[] keyByte = null;

        try {
            keyByte = key.getBytes("UTF-8");
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException(e);
        }
        md5.update(keyByte);
        byte[] digest = md5.digest();
        long hashCode = ((long)(digest[2] & 0xFF << 16))
                | ((long)(digest[1] & 0xFF << 8))
                | ((long)(digest[0] & 0xFF));

        return hashCode & 0xffffffffL;
    }
}
```





## 2.3、服务通信&调用(Feign)

- 服务间调用介绍(什么是Feign ， 它做了什么)
- 深入Feign (体系结构，底层机制，动态代理，重试)
- Feign的项目结构
- 源码阅读 EnableFeignClient 注解的底层机制 Feign协议解析过程



### 1.现有的服务调用方式

- Eureka http://ip:port/path 借助 RestTemplate实现
- Ribbon http://serviceName/path 借助 Ribbon 实现负载均衡，借助服务名来实现

### 2.Feign基础

> Feign解决了什么问题

- 简化远程调用
- 集成Ribbon
- 集成Hystrix

> Feign调用方式

- @FignClient("service-name")

### 3.Feign应用到服务消费者中

### 4.Feign注解背后的源码

- EnableFeignClients 注解背后的加载流程
- Spring 中 Bean 加载模式的扩展点
- Feign 构造上下文对象(FeignContext) 的过程

### 5.改造项目结构

- 构建公共接口层(注意不要引入依赖)
- 改造服务提供者
- 创建基于公共接口层的服务消费者

### 6.配置Ribbon 重试 和 超时策略

配置文件配置

```yaml
feign-client:
  ribbon:
    # 每台机器最大重试次数
    MaxAutoRetries: 2
    # 可以再重试几台机器
    MaxAutoRetriesNextServer: 2
    # 连接超时
    ConnectionTimeout: 1000
    # 业务处理超时时间
    ReadTimeout: 2000
    # 在所有 HTTP Method 进行重试
    OkToRetryOnAllOpertions: true
```

### 7.Contract协议解析过程

- 一家三口：Contract，BaseContract，SpringMvcContract

> 不能存在泛型，只能继承一个接口，继承的接口不能再继承其它接口

- 子类BaseContract 校验规则是什么
- 孙类SpringMvcContract 类如何抽取元数据

### 8.Feign 项目实战



## 2.4、降级熔断(Hystrix)

- 服务容错介绍
- Hystrix服务降级
	- 核心功能，服务降级原理，常用降级方案
	- Demo Fallback，RequestCache 多级降级
- Hystrix服务熔断
	- 熔断器工作原理
	- 集成Hystrix熔断器
- 源码阅读
	- Hystrix触发方式，熔断器的参数的作用
- 架构思考
	- 降级和熔断的规划
	- 主链路规划
	- 业务与容灾策略
- 线程隔离方案
	- 线程池
	- 信号量
- Demo 
	- Turbine + Hystrix 大盘 - 聚合监控信息

### 1.服务容错解决方案

- 服务雪崩

> 生产故障三步
>
> - 延迟
> - 服务不可用
> - 造成资损



> 降低故障影响
>
> - 降低串联影响
> - 隔离异常服务
> - 减压(快速失败 熔断)
> - 备选方案(降级  最小可用性)

### 2.Feign + Hystrix 实现Fallback降级

- 创建 hystrix-consumer项目 引入依赖
- 实现 Fallback 降级逻辑
- Fallback 降级还有什么花式玩法

### 3.Hystrix 实现Timeout降级

- 配置文件配置 hystrix 超时降级

### 4.Hystrix 实现RequestCache减压

- Request Cache 只是一种减压手段

- 使用 @CacheResult 缓存值

### 5.多级降级方案

- 模拟二级降级场景
- 一直错误的情况

某个方法超时时间可以通过注解来配置

```java
@GetMapping("/timeout2")
@HystrixCommand(fallbackMethod = "timeoutFallBack",commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")})
public String timeOut2(Integer timeOut) {
    return service.retry(timeOut);
}
```

但是要注意  service.retry(timeOut);  此方法的超时时间如果短于 timeOut2 中通过注解配置的超时时间，以更短的超时时间为主

### 6.Hystrix降级触发方案(阅读源码)

- Aspect 切面编程
- RxJava 中的 Observer 模式
- RxFunction 回调函数实现异常判定

### 7.Hystrix 熔断

- 熔断器参数配置
- 验证断路器开关状态切换

```properties
# Hystrix 熔断器配置
# 重要的配置
# 熔断的前提条件,(请求的数量),在一定的时间窗口内,请求达到5个以后,才开始进行熔断判断
hystrix.command.default.circuitBreaker.requestVolumeThreshold=5
# 错误请求占总请求的百分比,超过 50% 错误请求,熔断开启
hystrix.command.default.circuitBreaker.errorThresholdPercentage=50
# 当熔断开启以后,经过 多少时间,再进入半开状态
hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds=15000
# 配置时间窗口
hystrix.command.default.metrics.rollingStats.timeInMilliseconds=20000

# 不重要的配置
hystrix.command.default.circuitBreaker.enabled=true
# 强制开启熔断开关
hystrix.command.default.circuitBreaker.forceOpen=false
# 强制关闭熔断开关
hystrix.command.default.circuitBreaker.forceClosed=false
```

### 8.Hystrix 熔断器参数作用(源码)

- HystrixCircuitBreaker

- 断路判断逻辑
- 断路器开启/关闭 触发判断
- Half - Open 下的同步控制

### 9.降级熔断规划 - 主链路

<img src="/img/springCloud.assets/image-20220205150830444.png" alt="image-20220205150830444" style="zoom:80%;" />

### 10.线程隔离 - 核心方案以及工作原理

- 每个服务设定一定的线程数量, 该服务只能使用这些线程

- 线程隔离两种实现方式
	- 线程池
	- 信号量隔离

两种实现方式对比

实现原理：

- 线程池技术：它使用Hystrix自己内建的线程池去执行方法调用，而不是使用Tomcat的线程池
- 信号量技术：它直接使用tomcat的容器线程去执行方法，不会创建新的线程，信号量只充当开关和计数器的作用。获取到信号量就执行，没有获取到就　fallback

性能对比

-　线程池技术：涉及到线程的相关操作，还有线程之间的切换，性能不如信号量机制
-　信号量机制：直接使用Tｏｍｃａｔ容器线程去访问方法，信号量只是一个计数器的作用

超时判定

- 线程池技术：相当于多了一层保护机制，可以对 “执行阶段” 超时进行判断
- 信号量技术：只能等待网络请求超时 “被动超时” 的情况

选择：官方文档指出，信号量机制仅适用超高并发的非外部接口调用上，其它场景应该使用线程池技术

> tips：线程池技术要注意ThreadLocal 的数据传递的作用，由于前后调用不在同一个线程内，也不在父子线程内，如果业务层面声明了ThreaLocal 变量 ，将无法获取正确的值

### 11.Turbine 聚合 Hystrix 信息

- 创建 hystrix-turbine 子模块，引入依赖
- 添加Turbine 配置， 指定监控服务名称
- hystrix-fallback 项目开放 actuator 服务

### 12.Turbine 集成 大盘监控

- 创建 hystrix-dashboard 项目,引入依赖
- 启动大盘监控
- 解读监控页面内容，断路器进一步了解

### 13.集成 Hystrix 和 Turbine

- 配置基础组件 Turbine
- 配置 Dashboard  +  开放微服务端点
- 基于 HystrixCommand 注解配置降级和线程池

基于注解配置

```java
@HystrixCommand(
        commandKey = "loginFail" ,// 全局唯一的标识服务, 默认函数名称
        groupKey = "password"  , // 全局服务分组, 用于组织仪表盘,统计信息 默认 类名称
        fallbackMethod = "loginFailFallBack" ,// 同一个类里面 public / private 都可以
        // ignoreExceptions = {IllegalAccessException.class}   // 在列表中的exception 不会降级
        // 线程有关属性
        // 线程组,多个服务可以共用一个线程组
        threadPoolKey = "threadPoolA",
        threadPoolProperties = {
                // 核心线程数
                @HystrixProperty(name = "coreSize", value = "8"),
                // size > 0 LinkedBlockingQueue 实现 请求等待队列
                // SynchronousQueue 不存储元素 阻塞队列(建议读源码, 学CAS 操作)
                @HystrixProperty(name = "maxQueueSize", value = "20"),
                // maxQueueSize = -1 时无效 队列没有达到 maxQueueSize 依然拒绝请求
                @HystrixProperty(name = "queueSizeRejectionThreshold", value = "15"),
                // 统计窗口持续时间 (线程池)
                @HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "1024"),
                // 统计窗口内 筒子数量 (线程池)
                @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "18")

        }
        // ,
        // commandProperties = {
        //         // 熔断降级相关属性也可以配置在这里
        // }

)
```

基于 配置文件 配置

```yml
feign:
  hystrix:
    enabled: true
  client:
    config:
      # 默认全局配置
      default:
        connectTimeout: 1000
        readTimeout: 5000
      # 优先级高于上面配置
      foodie-user-service:
        connectTimeout: 1000
        readTimeout: 5000


# 开启所有 actuator-endpoint
management:
  endpoint:
    health:
      show-details: always
  endpoints:
    web:
      exposure:
        include: '*'
        # health info xxx

hystrix:
  command:
    # 有的属性是默认值,写不写都行
    default:
      fallback:
        enabled: true
      circuitBreaker:
        enabled: true
        # 超过 50% 错误开启熔断
        errorThresholdPercentage: 50
        # 5个request 之后才进行统计
        requestVolumeThreshold: 5
        # 10 秒后进行半开启状态
        sleepWindowInMilliseconds: 10000
        # forceClosed , forceOpen 强制关闭/开启 熔断开关
      execution:
        timeout:
          enabled: true
        # 可以指定线程池方式还是 信号量方式
        isolation:
          thread:
            interruptTimeout: true
            interruptOnFutureCancel: true
            timeoutInMilliseconds: 10000
      metrics:
        rollingStats:
          # 时间窗口统计
          timeInMilliseconds: 20000
```

> Ambiguous mapping 错误

```java
/**
* 对于需要调用端指定降级业务的场景来说,由于@RequestMapping  和 xxxMapping 注解可以从原始接口上继承，因此
 * 不能配置完全一样的两个路径，否则启动报错
 *
 * 在实际应用中，ItemCommentsService 上面定义了 @RequestMapping 同时 ItemCommentsFeignClient 继承自 ItemCommentsService
 * 因此在Spring 上下文中加载了两个访问路径一样的方法，会报错 Ambiguous mapping
 *
 * 解决问题的思路，避免Spring 上下文中同时加载两个访问路径相同的方法
 *
 * 1) 在启动类扫包的时候，不要把原始的Feign接口扫描进来 也就是 ItemCommentsService
 * 具体做法:  可以使用 @EnableFeignClients 注解的clients 属性, 只加载需要的Feign接口
 * 优点： 访问提供者和调用者不需要额外的配置
 * 缺点： 需要将每个使用的类都写一遍
 * 2) 原始Feign接口不定义@RequestMapping 注解
 * 优点：启动的时候直接扫包即可，不用加载接口
 * 缺点：a.服务提供者需要额外的配置访问的注解
 *       b.任何情况下，即使不需要在调用端定义fallback类，服务调用者都需要声明一个
 * 3) 原始Feign接口不要定义 @FeignClients 注解 这样就不会被加载到上下文中
 * 优点：启动的时候直接扫包即可,不用指定加载接口,服务提供者不用额外配置
 * 缺点: 任何情况下，服务调用者都需要一个额外的 @FeignClient 接口
*/
```

-  第一种方案是最简便的

## 2.5、分布式配置中心(Config)

- 配置中心介绍，在微服务中的应用
- Config核心功能和应用
- 直连式架构模型
	- Github 准备 存储属性
	- 搭建 config-server 应用
	- 将应用方直连配置中心
- 资源文件加载方式(源码阅读)
- 参数动态刷新机制 + Demo
- 高可用性分析
	- 单中心宕机，高可用改造方向
	- 借助Eureka 实现高可用配置中心架构
- 架构思考
	- 总线式架构展望
	- 分布式配置中心的用途
- 用对称密钥对配置项加密 解密

### 1.配置中心的作用

- 配置项定义 (程序 hardcode)
- 配置文件 application.yml bootstrap.yml 保存尽量不变的属性
- 环境变量 (操作系统层面)
- 数据库存储 配置信息

> 传统配置管理的缺点
>
> - 格式不统一，json，properties，yml
> - 没有版本控制
> - 基于静态配置 
> - 分布零散 - 没有统一管理

> 配置项的静态内容
>
> - 环境配置 
> 	- 数据库连接串
> 	- Eureka 注册中心
> 	- Kafka 连接
> 	- 应用名称
> - 安全配置(进行加密)
> 	- 连接密码
> 	- 公钥私钥
> 	- Http 连接 Cert
>
> 动态内容
>
> - 功能控制
> 	- 功能开关
> 	- 人工熔断开关
> 	- 蓝绿发布
> 	- 数据源切换
> - 业务规则
> 	- 当日外汇利率
> 	- 动态文案
> 	- 规则引擎参数
> - 应用参数
> 	- 网关 黑白名单
> 	- 缓存过期时间
> 	- 日志MDC设置



> 配置管理的需求
>
> - 配置项定义
> 	- 高可用
> 	- 版本管理 (修改记录，版本控制，权限控制)
> 	- 业务需求（内容加密，动态推送变更)
> 	- 配置分离(中心化管理)

### 2.创建Github配置仓库

- 创建github仓库
- 文件命名规则(文件名不能随便起)
- 添加配置文件和属性

> 文件命名规则
>
> - Application & profile (application-dev.yml)
> - Label - 代码分支名称

### 3.搭建配置中心

- 创建  config-server 项目引入依赖
- 添加参数和启动类

```java
package com.tho.springcloud;

import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.config.server.EnableConfigServer;

/**
 * @Author tho
 * @Date 2022/2/8/14:49
 * @ProjectName spring-cloud-netflix-demo
 * @ClassName: ConfigServerApplication
 * @Description: TODO
 */
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args)  {



        new SpringApplicationBuilder(ConfigServerApplication.class)
                .web(WebApplicationType.SERVLET)
                .run(args);
    //      访问git 仓库的配置文件
    //    http://localhost:60000/{label}/{application}-{profile}.json (.yml .properties )
    //    http://localhost:60000/{application}/{profile}/{label}
    }
}
```

- 对应的配置文件

```yml
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
#          uri: git@github.com:th66778899/config-repo.git
          uri: https://github.com/th66778899/config-repo.git
          # 强制拉取资源文件
          force-pull: true
# 多个项目共用一个repo 使用文件夹来进行区分
#          search-paths: aaa, ccc, ddd
#          username:
#          password:
server:
  port: 60000
```

### 4.搭建Client 直连式配置中心

- 创建 config-client 项目引入依赖
- 配置启动项和启动类
- 注入Github 属性到测试用例

### 5.阅读源码(资源文件加载)

- Config server 加载资源文件

### 6.动态拉取参数

- 引入特殊依赖
- 改造 config-client 可以拉取动态参数
- 借助 actuator 可以让config-client 访问 config-server ,获取最新的配置信息，

访问 http://localhost:61000/actuator/refresh 会显示更新的配置信息

- 只有发生了变化的配置才会显示出来

### 7.配置中心高可用

- config-server 向eureka注册中心报道
- config-client 从注册中心获取config-server 地址

### 8.使用对称性密钥进行加解密

- JDK 中替换 JCE (jdk版本 高于8u161 不需要替换，已经集成了最新的)
- 改造 config-server 并生成加密字符串
- 修改Github 文件，启动服务拉取配置

> 密钥在哪，加解密就在哪进行

### 9.分布式配置中心的通用场景

- 环境隔离
	- 日常环境
	- 预发环境
	- 生产环境
- 业务开关 + 定向推送
	- 灰度发布，蓝绿发布，金丝雀发布
- 修改业务逻辑 
	- 网关黑名单 -> 网关层
	- 费率 - 规则引擎 （下单接口）
	- 熔断阈值   (下单接口)

### 10.集成配置中心

- 搭建配置中心
- 创建Github 配置文件
- 用户中心集成Config

## 2.6、消息总线(BUS)

BUS

- 消息总线 (BUS) 介绍
- BUS 体系结构和接入方式 (借助RabbitMQ实现)
- 将配置中心改造为总线式架构
- 源码阅读 - bus -refresh 底层机制
- Git Webhook 自动推送
- 架构思考 - 消息总线如何用于其它业务场景

### 1.BUS 简介

- BUS的标签
	- 轻量级 (依赖于 stream 组件)
	- 消息广播 (基于 发布 - 订阅)
	- 无缝集成 Config
	- 自定义消息
- BUS 的两个场景
	- 配置变更通知(仅仅是通知配置的变更)
	- 自定义消息广播

### 2.搭建总线式架构配置中心

- 创建 config-bus-server 和 config-bus-consumer
- 启动RabbitMQ 修改demo 配置属性
- 使用 actuator 服务推送Bus 变更

### 3.源码阅读 - bus-refresh底层机制

- 内置的事件结构，RefreshRemoteApplicationEvent
- 刷新事件的发送端 - RefreshBusEndpoint

### 4.WebHook 自动推送

配置Github 的 WebHook

- 设置 encrypt.key
- 将上一步中的key添加到github仓库设置中
- 配置Webhook url

### 5.BUS其它业务场景

- 自定义自己的事件
- 清空缓存：通知所有服务监听者清空某项业务的本地缓存业务，我们也可以在自定义的消息体中加业务属性，事件监听逻辑可以根据这些属性来定点清除某个特点业务对象的缓存
- 数据同步：子系统依赖实时的数据库记录变动触发响应的业务逻辑，我们这里可以将数据的binlog抓取出来，通过广播功能同步到所有监听器，起到数据同步的作用

### 6.BUS总结

<img src="/img/springCloud.assets/image-20220210152803883.png" alt="image-20220210152803883" style="zoom:80%;" />

## 2.7、服务网关(Gateway)

- 服务网关在微服务中的应用
- 第二代网关组件Gateway 介绍 (第一代网关 Zuul)
- Gateway 体系架构
- Gateway 急速落地(路由规则)
- Gateway断言功能详解
	- Demo 利用断言实现URL映射
	- 利用After 断言构建简易秒杀场景
- Gateway 过滤器原理和生命周期
	- demo 自定义过滤器
- 源码阅读Gateway 过滤器机制解析
- 权限认证 -  分布式session 的替代方案
	- 基于 jwt 的网关层鉴权服务
- 如何借助网关层对服务端各类异常做统一处理
- 网关的另一个技能 - 限流
- 还有哪些网关技术，如何选型

### 1.第二代网关Gateway简介

- SpringCloud 官方主推
- 底层基于Netty

> Gateway的业务场景

- 路由寻址 (根据url导向到后端的服务)
- 负载均衡 (ribbon)
- 限流
- 鉴权

Gateway 自动装配工厂类 GatewayAutoConfiguration

### 2.创建默认路由规则(demo)

- 创建gateway-sample 项目，引入依赖
- 连接eureka自动创建路由
- 通过 Actuator 实现动态路由功能

### 3.Path断言

- 使用Path断言转发请求(yml配置 java配置)
- 配合使用Method断言

### 4.After断言实现简易秒杀

- 创建模拟下单接口
- 通过After断言设置生效时间

### 5.自定义过滤器

- 创建 TimerFilter 实现计时功能
- 添加TimerFiler 到路由

```java
package com.tho.springcloud;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.util.StopWatch;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * @Author tho
 * @Date 2022/2/11/14:57
 * @ProjectName spring-cloud-netflix-demo
 * @ClassName: TimerFilter
 * @Description: TODO
 */
@Slf4j
@Component
// 实现 GlobalFilter 的filter 是全局filter
// 实现 GatewayFilter 的 fileter 需要进行配置,不是全局filter
public class TimerFilter implements GatewayFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        StopWatch timer = new StopWatch();
        timer.start(exchange.getRequest().getURI().getRawPath());
        // Zuul Filter before / after

        // exchange.getAttributes().put("requestTimeBegin", System.currentTimeMillis());
        return chain.filter(exchange).then(
                Mono.fromRunnable(() -> {
                    timer.stop();
                    log.info(timer.prettyPrint());
                })

        );
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

### 6.源码阅读 - Gateway过滤机制解析

- 接收请求获取路由表 - RoutePredicateHandlerMapping
- 执行过滤器 - FilteringWebHandler
- Filter执行顺序排序 - AnnotationAwareOrderComparator

### 7.实现JWT鉴权

- 创建 auth-service (登录，鉴权等服务)
- 添加JwtService 类 实现token创建和验证
- 网关层集成 auth-service (添加AuthFilter 到网关层)

### 8.其它网关技术&选型

- Nginx & lua 开源免费，软负载，性价比高
- F5 负载均衡，流量控制，高可用 99.999%，全链路监控，BIG-IP压缩流量，编程路由，拓扑路由，攻击防护，网络安全，统计和报告

> 选型
>
> - F5 贵
> - 万金油  - nginx
> - springcloud 项目 - gateway

### 9.结合项目

- 添加gateway组件
- 配置路由规则

```java
package com.tho;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author tho
 * @Date 2022/2/11/22:24
 * @ProjectName foodie-cloud
 * @ClassName: RoutesConfiguration
 * @Description: TODO
 */
@Configuration
public class RoutesConfiguration {

    @Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {

        return builder.routes()
                .route(r -> r.path("/address/**", "/passport/**", "/userInfo/**", "/center/**")
                        .uri("lb://FOODIE-USER-SERVICE"))
                .route(r -> r.path("/items/**")
                        .uri("lb://FOODIE-ITEM-SERVICE"))
                .route(r -> r.path("/shopcart/**")
                        .uri("lb://FOODIE-CART-SERVICE"))
                .route(r -> r.path("/orders/**", "/myOrders/**", "/mycommonts/**")
                        .uri("lb://FOODIE-ORDER-SERVICE"))
                .build();


    }
}
```

- 配置网关层Redis限流

```java
package com.tho;

import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.cloud.gateway.filter.ratelimit.RedisRateLimiter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import reactor.core.publisher.Mono;

/**
 * @Author tho
 * @Date 2022/2/11/22:33
 * @ProjectName foodie-cloud
 * @ClassName: RedisLimiterConfiguration
 * @Description: Redis限流
 */
@Configuration
public class RedisLimiterConfiguration {

    // ID : KEY
    @Bean
    // 多个keyResolver 采用这个为主
    @Primary
    public KeyResolver remoteAddressKeyResolver() {
        return exchange -> Mono.just(
                exchange
                        .getRequest()
                        .getRemoteAddress()
                        .getAddress()
                        .getHostAddress()
        );
    }

    //
    @Bean("redisLimiterUser")
    @Primary
    public RedisRateLimiter redisRateLimiterUser() {
        return new RedisRateLimiter(10, 20);
    }
    @Bean("redisLimiterItem")
    public RedisRateLimiter redisRateLimiterItem() {
        return new RedisRateLimiter(20, 50);
    }
}
```

### 

- 创建网关鉴权服务

- 网关层跨域Filter

- 网关层登录校验

## 2.8、服务调用链追踪(Sleuth)

- 服务调用链追踪是做什么的
- Sleuth核心功能和体系结构
	- 调用链追踪模型 - Trace Span Annotation 
	- demo
- 阅读源码 链路追踪原理 
- Zipkin 简介
	- 搭建Zipkin服务端
	- Sleuth集成Zipkin
- Sleuth集成ELK实现日志搜索
- 阿里系分布式追踪技术 - 鹰眼系统

### 1.链路追踪的作用

- 微服务之间调用关系复杂
- 得到整条链路的调用关系

> 链路追踪技术的基本功能

- 分布式环境下链路追踪
- Timing信息
- 定位链路
- 信息收集和展示

### 2.整合Sleuth追踪调用链路

- 创建sleuth-traceA 和 sleuth-traceB ， 添加sleuth依赖
- 调用请求链路，查看log中的信息
- 采样率设置

### 3.源码阅读 Sleuth原理

- Sleuth项目结构和启动类
- 以Spring WebFlux调用链举例
	-  TraceWebFilter创建Span的过程
	- 关联上下游Span的过程

### 4.Zipkin

- 搭建Zipkin服务端

```java
package com.tho.springcloud;

import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import zipkin.server.internal.EnableZipkinServer;

/**
 * ZipKin服务端: 用于收集客户端Sleuth埋点收集到的信息
 */
@SpringBootApplication
@EnableZipkinServer
public class ZipkinServerApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(ZipkinServerApplication.class)
                .web(WebApplicationType.SERVLET)
                .run(args);
    }
}
```

- Sleuth集成Zipkin实例
	- sleuth-traceA和sleuth-traceB集成Zipkin
	- 从Zipkin Dashboard 搜索调用链的时间维度数据
- Sleuth集成ELK实现日志检索
	- ELK的作用 
		- 日志收集和过滤
		- 日志信息持久化
		- 汇总分析重要数据
		- 根据关键字查找Log
	- ELK工作流程
		- Logstash Log信息的收集和过滤
		- ElasticSearch 存储log信息 提供搜索
		- Kibana  Log信息的查询,报表等
	- Docker启动ELK
		- 通过docker安装elk镜像 (耐心) `docker pull sebp/elk`
		- 创建Docker容器，取名为elk，指定ELK三个组件的端口
		- 修改logstash 接收日志的方式
	- ELK集成Sleuth
		- 引入Logstash依赖到sleuth项目中
		- 配置日志文件，输出Json格式日志到Logstash

### 5.项目集成Sleuth

- 搭建Zipkin服务端
- 项目集成sleuth 和 zipkin
- sleuth 集成 elk



## 2.9、消息驱动(Stream)

- 消息驱动在微服务中的应用
- 消息驱动三板斧(理论 - 实践 - 扩展)
	- Stream 体系结构
	- Stream快速入门 集成MQ消费
	- 商品发布销峰策略
- 源码阅读  
	- Stream Binder 的作用机制
- 发布订阅相关
	- 分布订阅模型详解
	- Demo 基于发布订阅实现广播功能
	- 利用发布订阅实现商品信息的刷新
- 消费组和消费分区详解
	- demo 基于消费组实现轮询单播功能
- 经典业务场景
	- 延迟消息介绍 + 案例 demo Stream + MQ插件实现延迟消息
- Stream中的异常消息处理 (四种方式)
- Stream 的重试机制
	- demo Stream本地重试功能
	- Stream + MQ实现 enqueue操作
- 架构思考
	- 异常情况导致消息无法被消费
		- 借助死信队列实现异常处理
		- 定制自定义异常逻辑
	- 如何根据业务场景选择合适的异常处理策略
- 项目集成消息组件

### 1.消息驱动在微服务中的应用

- 发布消息 (消息中间件 Kafka RabbitMQ)

<img src="/img/springCloud.assets/image-20220213131027914.png" alt="image-20220213131027914" style="zoom:67%;" />

> 消息驱动的应用场景

- 跨系统异步通信
- 应用解耦
- 流量削峰

### 2.Stream急速落地

- 创建stream-sample项目，引入依赖
- 创建监听器(声明和绑定信道)
- 从RabbitMQ触发消息

### 3.削峰策略

> 消息组件
>
> - 平滑输出(客户端自动拉取)
> - 不怕Timeout 不受限于API超时
> - 高吞吐量

### 4.StreamBinder作用机制



### 5.Stream实现消息广播

> 配置文件

```properties
spring.application.name=stream-sample
server.port=63000

# Rabbitmq连接字符串
spring.rabbitmq.host=192.168.198.100
spring.rabbitmq.port=5672
spring.rabbitmq.username=tho
spring.rabbitmq.password=123456

management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always

# 配置Stream自定义广播消息Topic: 绑定消费者、生产者信道到broadcastTopic
#spring.cloud.stream.bindings.broadcastTopic-consumer.destination=broadcast
#spring.cloud.stream.bindings.broadcastTopic-producer.destination=broadcast
spring.cloud.stream.bindings.broadcastTopic-consumer.destination=broadcastTopic
spring.cloud.stream.bindings.broadcastTopic-producer.destination=broadcastTopic

# 测试单播: 绑定消费者、生产者信道到groupTopic
spring.cloud.stream.bindings.groupTopic-consumer.destination=groupTopic
spring.cloud.stream.bindings.groupTopic-producer.destination=groupTopic

# 测试单播: 配置消费者分组 => 实际上是一个组一个queue, 每个queue有多个Consumer
spring.cloud.stream.bindings.groupTopic-consumer.group=GroupA

# 测试单播: 配置消息分区 => 经测试可知, 消息分区和消费组可以合起来使用, 消费组可用来实现单播(组内轮训消费), 消息分区可用来隔离消费组(只有满足条件即SpEL匹配的消费组才能消费消息)
# 打开消费者的消费分区功能
spring.cloud.stream.bindings.groupTopic-consumer.consumer.partitioned=true
# 指定当前消费者实例的总数
spring.cloud.stream.instance-count=2
# 指定当前消费者实例的索引号, 最大值为count-1, 用于测试消息分区
spring.cloud.stream.instance-index=1
# 指定生产者拥有两个消息分区
spring.cloud.stream.bindings.groupTopic-producer.producer.partition-count=2
# SpEL => Key Resolver解析, 表示只有节点为1的消费者才能消费消息, 即SpEL匹配才能消费
spring.cloud.stream.bindings.groupTopic-producer.producer.partition-key-expression=1

# 测试延迟消息: 绑定消费者、生产者信道到delayedTopic
spring.cloud.stream.bindings.delayedTopic-consumer.destination=delayedTopic
spring.cloud.stream.bindings.delayedTopic-producer.destination=delayedTopic

# 测试延迟消息: 生产者允许生成延迟交换机与延迟队列(都只有一个)
spring.cloud.stream.rabbit.bindings.delayedTopic-producer.producer.delayed-exchange=true

# 测试测试异常重试(单机版), 即在Consumer本地重试, 而不会发回给Rabbitm: 绑定消费者、生产者信道到exceptionTopic
spring.cloud.stream.bindings.exceptionTopic-consumer.destination=exceptionTopic
spring.cloud.stream.bindings.exceptionTopic-producer.destination=exceptionTopic

# 测试测试异常重试(单机版), 即在Consumer本地重试, 而不会发回给Rabbitm: 配置本机重试次数, 次数为1代表不重试
spring.cloud.stream.bindings.exceptionTopic-consumer.consumer.max-attempts=2

# 测试异常重试(联机版), 消费者会重新生成把消息投递回队列尾部: 绑定消费者、生产者信道到requeueTopic
spring.cloud.stream.bindings.requeueTopic-consumer.destination=requeueTopic
spring.cloud.stream.bindings.requeueTopic-producer.destination=requeueTopic

# 测试异常重试(联机版), 消费者会重新生成把消息投递回队列尾部: 对指定Consumer配置重新入队
#spring.cloud.stream.rabbit.bindings.requeueTopic-consumer.consumer.requeueRejected=true
# 默认全局开启Direct重新入队(不过会被Consumer重试覆盖)
#spring.rabbitmq.listener.direct.default-requeue-rejected=true
# 所以配置Consumer只能重试1次
spring.cloud.stream.bindings.requeueTopic-consumer.consumer.max-attempts=1
# 测试不同分组的消费者消费Requeue消息 => 实际上Group、Topic的名称最好都用-作为连接, 而不是驼峰标识
spring.cloud.stream.bindings.requeueTopic-consumer.group=requeue-group

# 测试死信队列Topic: 绑定消费者、生产者信道到dlqTopic
spring.cloud.stream.bindings.dlqTopic-consumer.destination=dlqTopic
spring.cloud.stream.bindings.dlqTopic-producer.destination=dlqTopic
spring.cloud.stream.bindings.dlqTopic-consumer.consumer.max-attempts=2
spring.cloud.stream.bindings.dlqTopic-consumer.group=dlq-group
# 开启死信队列(默认名称为${dlqTopic}.dlq, 复杂的需要自己指定DLK), 允许指定Consumer绑定DLQ
# => rabbitmq-plugins enable rabbitmq_shovel rabbitmq_shovel_management, 管理控制台开启重推消息其他队列功能
spring.cloud.stream.rabbit.bindings.dlqTopic-consumer.consumer.auto-bind-dlq=true

# 测试异常降级, 自定义异常逻辑 + 接口升版: 绑定消费者、生产者信道到dlqTopic
spring.cloud.stream.bindings.fallbackTopic-consumer.destination=fallback-Topic
spring.cloud.stream.bindings.fallbackTopic-producer.destination=fallback-Topic
spring.cloud.stream.bindings.fallbackTopic-consumer.consumer.max-attempts=2
spring.cloud.stream.bindings.fallbackTopic-consumer.group=fallback-group
# errors 是规定写死的
# inputChannel => fallback-Topic.fallback-group.errors
```

- 创建消息Producer服务，配置消息主题
- 启动多个Consumer节点测试消息广播
- RabbitMQ界面查看广播组(Exchanges )

### 6.基于消费组实现轮询单播功能

- 创建Producer和Consumer
- 配置消费组，启动两个节点
- RabbitMQ界面单播和广播在Exchange中的不同
- 消费分区的配置项

### 7.Stream + MQ插件实现延迟消息

- 配置RabbitMQ延迟插件，重启MQ
- 创建producer和consumer，配置 exchange-type
- 添加Message Header 传递延迟时间
- 启动查看效果

> Postman测试时注意 
>
> - @RequestParam 修饰的变量 postman传参时要传入json形式，或者在传值时多加一对双引号,直接传值会报错

### 8.Stream实现异常重试

- 创建Producer和Consumer，在Consumer中抛出异常
- 设置重试次数
- 重试成功和失败的表现

### 9.Stream实现Requeue操作进行重试

- 创建Producer和Consumer
- 开启 Re-queue功能(和retry配置又冲突)
- 测试Re-queue在不同节点的消费情况

> 如何处理重试不能被解决的异常

- 消息被拒绝 ， 重试次数达到阈值
- 消息过期(TTL)
- 队列长度已满



> 解决方式

- 死信交换机
	- DLX - 死信交换器，将异常消息路由到死信队列
	- DLK - Dead Letter Routing Key

### 10.借助死信队列实现异常处理

- 使用rabbitmq-plugins enable命令开启RabbitMQ插件
- 创建producer和consumer，配置死信队列
- 查看Rabbitmq 死信队列
- 死信队列消息重新消费

### 11.自定义异常处理逻辑

- 借助Spring-integration实现Fallback降级逻辑
- Consumer升级

## 2.10、基于RPC服务治理（Dubbo）

- 服务治理 RPC vs Http
- Dubbo介绍
	- Dubbo架构设计
	- Dubbo核心功能
- Dubbo注册中心介绍
	- 基于Zookeeper的注册中心
- RPC协议解析流程
- 构建服务消费者，发起远程调用
- Dubbo服务容错，负载均衡
- Dubbo-admin的服务治理
- 源码阅读 - Dubbo
- HSF 和 Dubbo

### 1.RPC vs Http

- RPC 远程方法调用 (Remote Procedure call)
- 服务治理 (分布式环境注册中心)
- RPC协议 (方法寻址，对象序列化/反序列化)

> 两者对比
>
> - 接口风格
>
> <img src="/img/springCloud.assets/image-20220214152855031.png" alt="image-20220214152855031" style="zoom:67%;" />
>
> <img src="/img/springCloud.assets/image-20220214154744795.png" alt="image-20220214154744795" style="zoom:67%;" />

### 2.Dubbo注册中心

> 可选的注册中心

- Multicast
- Zookeeper
- Nacos
- Redis
- Simple

> 基于Zookeeper的服务注册

<img src="/img/springCloud.assets/image-20220214154920464.png" alt="image-20220214154920464" style="zoom:80%;" />

### 	3.创建基于ZK的注册中心生产者服务

- 启动ZK作为注册中心
- 创建dubbo-api接口层
- 创建dubbo-provider，添加service层

### 4.构建服务消费者

- 创建dubbo-consumer作为服务调用方
- 添加Controller 并调用 dubbo-client中的服务
- 注意序列化/反序列化的异常

### 5.基于 Dubbo-Admin的服务治理

- 服务治理可视化
	- 条件路由
	- 标签路由
	- 黑白名单
	- 服务权重
	- 负载均衡
	- 服务测试
- 前端工程
	- Vue.js + Vuetify实现
- 后端工程
	- SpringBoot工程

> 服务治理兼容



> 环境搭建
>
> - 修改ookeeper冲突端口
> - 元数据配置
> - 启动dubbo-admin的前后端项目