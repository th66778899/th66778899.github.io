---
title: RabbitMQ
date: 2021-12-5 18:35:56
tags: 
- RabbitMQ
categories:
- 消息队列
index_img: /img/rabbitmq.png




---



RabbitMQ基本使用

<!--more-->

- 分布式消息队列(MQ)认知与提升

- RabbitMQ实战
- RabbitMQ可靠性投递基础组件封装

# 一、MQ基础

## 1.1、MQ应用场景

>JMS（Java Message Service）规 ava消息服务，它定义了Java中访问消息中间件的接口的规范。在这里注意哦，JMS只是接口，并没有给予实现，实现JMS接口的消息中间件称为 “JMS Provide 的开源 MOM （Message Oriented Middleware，也就是消息中间件）系统包括Apache的ActiveMQ、RocketMQ、Kafka，以及RabbitMQ，可以说他们都 “基本遵 考” JMS规范，都有自己的特点和优势。 
>
>- 专业术语 
>	- JMS（Java Message Service）：实现JMS 接口的消息中间件； 
>	- Provider（MessageProvider）：消息的生产者； 
>	- Consumer（MessageConsumer）：消息的消费者； 
>	- PTP（Point to Point）：即点对点的消息模型，这也是非常经典的模型； 
>	- Pub / Sub（Publish/Subscribe）：，即发布/订阅的消息模型； 
>	- Queue：队列目标，也就是我们常说的消息队列，一般都是会真正的进行物理存储；
>	-  Topic：主题目标； 
>	- ConnectionFactory：连接工厂，JMS 用它创建连接；
>	-  Connection：JMS 客户端到JMS Provider 的连接；
>	-  Destination：消息的目的地； 
>	- Session：会话，一个发送或接收消息的线程（这里Session可以类比Mybatis的Session
>- JMS消息格式定义：
>	-  StreamMessage 原始值的数据流 
>	- MapMessage 一套名称/值对 
>	- TextMessage 一个字符串对象
>	-  BytesMessage 一个未解释字节的数据流 
>	- ObjectMessage 一个序列化的Java对象

- 服务解耦
- 销峰填谷
- 异步化缓冲

> 思考点
>
> - 生产端可靠性投递
> - 消费端幂等性
> - 高可用
> - 低延迟
> - 可靠性
> - 堆积能力
> - 可扩展性

## 1.2、主流MQ

- ActiveMQ 阿帕奇   (性能较差)
- RabbitMQ (重点)   (扩展性较差)
- RocketMQ (阿里 -> 阿帕奇)
- Kafka (高吞吐量,数据转储)

> 如何进行技术选型
>
> - 各个MQ的性能,优缺点，相应的业务场景
> - 集群架构模式，分布式，可扩展性，高可用性，可维护性
> - 综合成本问题，集群规模，人员成本
> - 未来的方向，规划，思考

## 1.3、RabbitMQ集群架构模型与原理

1.主备模式(master -> slave)

2.远程模式(配置复杂)

3.镜像模式(使用最为广泛)

4.多活模型(类似远程模式)

### 1、主备模式

- warren(兔子窝),主备方案 ,主节点挂了,备份节点启用

### 2、远程模式

- 远距离通信复制，实现双活的一种模式，让两个跨地域的MQ构成集群
- 配置复杂

### 3、镜像模式

- 保证100%数据不丢失
- 在实际工作中用的最多，并且实现集群非常简单，一般互联网大厂都会构建这种集群模式

> 高可用  数据同步  高可靠(奇数个节点)
>
> 无法横向扩展
>
> 多个服务器同步相同数据

### 4、多活模式

- 依赖RabbitMQ的 federation插件
- RabbitMQ部署架构采用双中心(多中心),那么在两套(或者多套)数据中心部署一套RabbitMQ集群，各中心的RabbitMQ服务除了需要为业务提供正常的消息服务外，中心之间还需要实现部分队列消息共享

# 二、RabbitMQ

## 2.1、基础

> RabbitMQ是一个开源的消息代理和队列服务器，用来通过普通协议在完全不同的应用之间共享数据，RabbitMQ是使用Erlang语言来编写的，并且RabbitMQ是基 议的。各个互联网大厂都在使用RabbitMQ作为消息中间件，为什么呢，下面我们来一起看看，“她” 都有哪些优点！ 
>
> - 采用Erlang语言作为底层实现：Erlang有着和原生Socket一样的延迟 
> - 开源、性能优秀，稳定性保障 
> - 提供可靠性消息投递模式（confirm）、返回模式（ return ） 
> - 与SpringAMQP完美的整合、API丰富 
> - 集群模式丰富，表达式配置，HA模式，镜像队列模型
> -  保证数据不丢失的前提做到高可靠性、可用性

- AMQP核心概念
- RabbitMQ安装与入门
- RabbitMQ核心API
- RabbitMQ高级特性
- RabbitMQ集群架构实操（镜像集群）
- SpringBoot整合
- MQ基础组件封装和实战

## 2.2、RabbitMQ环境搭建

### 1.首先安装erlang环境

因为RabbitMQ需要erlang环境的支持，所以必须安装erlang

要安装的是 erlang-22.3.3-1.el7.x86_64.rpm 先执行以下命令来安装其对应的yum repo

`curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash`

执行以下命令来正式安装 erlang 环境

`yum install erlang-22.3.3-1.el7.x86_64`

(如果有提示 重新执行该事务) 执行命令

`yum load-transaction /tmp/yum_save_tx.2020-05-14.22-21.n0cwzm.yumtx`

测试erlang是否安装成功

`erl`

出现

> [root@centos_7_100 ~]# erl
> Erlang/OTP 22 [erts-10.7.1] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:1] [hipe]
>
> Eshell V10.7.1  (abort with ^G)
>
> 说明安装成功

### 2、安装RabbitMQ

首先安装其对应的yum repo

`curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash`

执行命令正式安装RabbitMQ

`yum install rabbitmq-server-3.8.3-1.el7.noarch`

### 3、设置RabbitMQ开机启动

(关闭防火墙或者打开 15672端口)

`chkconfig rabbitmq-server on`

启动RabbitMQ服务

`systemctl start rabbitmq-server.service`

开启WEB可视化管理插件

`rabbitmq-plugins enable rabbitmq_management`

访问可视化管理界⾯

http://192.168.198.100:15672

能看到网页登录入口

我们可以在后台添加一个用户/密码对

`rabbitmqctl add_user tho 123456 `

`rabbitmqctl set_user_tags tho administrator`

```java
[root@centos_7_100 ~]# rabbitmqctl add_user tho 123456
Adding user "tho" ...
[root@centos_7_100 ~]# 
[root@centos_7_100 ~]# rabbitmqctl set_user_tags tho administrator
Setting tags for user "tho" to [administrator] ...
[root@centos_7_100 8~]# 
```

之后在网页登录即可

## 2.3、RabbitMQ高性能

- Erlang语言用于交换机领域的架构模式
- Erlang有着和原生Socket一样的延迟

> AMQP高级消息队列协议
>
> - AMQP协议模型

AMQP核心概念

> - Server：Broker，接收客户端连接，实现AMQP实体服务
> - Connection：连接，应用程序与Broker的网络连接
> - Channel：网络信道，所以的操作几乎都在Channel中进行，客户端可以建立多个Channel，每个Channel代表一个会话任务
> - Message：消息，服务器和应用程序之间传送的数据，由Properties和Body组成。Properties可以对消息进行修饰，比如消息的优先级，延迟等高级特性，Body则是消息体内容
> - Virtual Host：虚拟地址，进行逻辑隔离，最上层的消息路由，一个Virtual host里可以有多干个Exchange和Queue，同一个Virtual Host 不能有相同名称的Exchange和Queue
> - Exchange：交换机，接收消息，根据路由键转发消息到绑定的队列
> - Binding：Exchange和Queue之间的虚拟连接，binding中包括 routing key
> - Routing key ： 一个路由规则，虚拟机 可用它来确定如何路由一个特定消息
> - Queue：也称为Message Queue：消息队列，保存消息并把它们转发给消费者



> RabbitMQ整体架构
>
> producer -> exchange -> queue -> consumer

## 2.4、如何保证100%投递成功

> 生产端可靠性投递
>
> - 保证消息的成功发出
> - 保证MQ节点成功接收
> - 发送端接收到MQ节点(Broker)确认应答
> - 完善的消息进行补偿机制



> 解决方案
>
> - 消息落库，对消息状态进行打标(高并发场景下不太合适)
> - 消息延迟投递，做二次确认，回调检查(较为常用，减少数据库操作)



## 2.5、幂等性

> - 借鉴数据库的乐观锁机制、
> - 执行一条更新库存的sql语句
> - update t_reps set count = count - 1, version = version + 1 where version = 1



> 海量订单产生的业务高峰期，如何避免消息的重复消费问题
>
> - 消费端实现幂等性，就意味着，我们的消息永远不会消费多次，即使我们收到了多条一样的消息



> 消费端-幂等性保障
>
> - 唯一ID + 指纹码 机制，利用数据库主键去重
> - 利用Redis的原子性去实现

### 1.唯一id + 指纹码 机制

- 唯一ID + 指纹码机制，利用数据库主键去重
- Select count(1) from t_order where id = 唯一id + 枝纹码
- 好处 ： 实现简单
- 坏处： 高并发下有数据库写入的性能瓶颈
- 解决方案：跟进ID进行分库分表进行算法路由

### 2.利用Redis原子特性实现

- 利用Redis进行幂等性操作

> 需要考虑的问题
>
> - 是否要进行数据库落库，如果落库的话，数据库和缓存如何做到原子性(一致性)
> - 如果不进行落库，都存储到缓存中，如何设置定时同步策略

## 2.6、整合SpringBoot

1.@RabbitListener注解

> 消费端监听 @RabbitMQListener
>
> @QueueBinding @Queue @Exchange

### 1.生产者

#### 相关依赖配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tho</groupId>
    <artifactId>RabbitMQ</artifactId>
    <packaging>pom</packaging>
    <version>1.0</version>
    <modules>
        <module>RabbitMQ-producer</module>
    </modules>
    <!--Springboot 依赖-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath />
    </parent>
    <!--相关配置-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!--使用自定义日志框架 屏蔽自带的日志框架-->
            <!--<exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>-->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>

        </dependency>
        <!--Springboot 整合RabbitMQ-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>


    </dependencies>

</project>
```

#### 配置文件

```properties
server.port=8001
server.servlet.context-path=/

spring.rabbitmq.host=192.168.198.100
spring.rabbitmq.username=tho
spring.rabbitmq.password=123456
spring.rabbitmq.virtual-host=/
spring.rabbitmq.connection-timeout=15000

## 启用消息确认模式
spring.rabbitmq.publisher-confirms=true

## 设置return消息模式,注意要和mandatory一起去配合使用
## spring.rabbitmq.publisher-returns=true
## spring.rabbitmq.template.mandatory=true

spring.http.encoding.charset=UTF-8
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
spring.jackson.default-property-inclusion=non_null
```

#### 组件

```java
package com.tho.component;


import org.springframework.amqp.AmqpException;
import org.springframework.amqp.core.MessagePostProcessor;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

import java.util.Map;
import java.util.UUID;

/**
 * @Author tho
 * @Date 2021/12/4/20:01
 * @ProjectName RabbitMQ
 * @ClassName: RabbitMQSender
 * @Description: RabbitMQ消息发送者
 */
@Component
public class RabbitMQSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 确认消息的回调消息监听，用于确认消息是否被Broker收到
     */
    final RabbitTemplate.ConfirmCallback confirmCallback= new RabbitTemplate.ConfirmCallback(){

        /**
         *
         * @param correlationData 作为一个唯一的标识,标记消息是不是对应的返回
         * @param ack              消息是否落盘成功
         * @param cause             失败的一些异常信息
         */
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {

        }
    };

    /**
    * @Author tho
    * @Date 2021/12/4 20:08
    * @param message  具体的消息内容
    * @param properties 额外的附加属性
    * @Return void
    * @Description: 对外发送消息的方法
    */
    public void send(Object message, Map<String, Object> properties) throws Exception {

        // 携带附加属性,需符合AMQP协议
        MessageHeaders mhs = new MessageHeaders(properties);
        // 构造消息
        Message<?> msg = MessageBuilder.createMessage(message, mhs);
        // 采用 spring.rabbitmq.publisher-confirms=true 模式
        // 消息确认模式 需要设定回调事件
        rabbitTemplate.setConfirmCallback(confirmCallback);
        // 指定了业务唯一id
        CorrelationData cd = new CorrelationData(UUID.randomUUID().toString());
        MessagePostProcessor mpp = new MessagePostProcessor() {
            @Override
            public org.springframework.amqp.core.Message postProcessMessage(org.springframework.amqp.core.Message message) throws AmqpException {
                System.err.println("post to do: +  " + message);
                return message;
            }
        };

        // 发送消息
        rabbitTemplate.convertAndSend("exchange-1", "springboot.rabbit",
                msg,
                mpp,
                cd);

    }
}
```

### 2.消费者

#### 配置文件

```properties
server.port=8002
server.servlet.context-path=/

spring.rabbitmq.host=192.168.198.100
spring.rabbitmq.username=tho
spring.rabbitmq.password=123456
spring.rabbitmq.virtual-host=/
spring.rabbitmq.connection-timeout=15000

## 消费者消费成功后需要手工进行签收(ack),默认为auto
spring.rabbitmq.listener.simple.acknowledge-mode=manual
spring.rabbitmq.listener.simple.concurrency=5
spring.rabbitmq.listener.simple.prefetch=1
spring.rabbitmq.listener.simple.max-concurrency=10

## 自定义配置
## 最好不要在代码里将配置固定,尽量使用配置文件使用${}来获取参数
spring.rabbitmq.listener.order.exchange.name=exchange-1

spring.http.encoding.charset=UTF-8
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
spring.jackson.default-property-inclusion=non_null
```

#### 组件

```java
package com.tho.consumer.component;

import com.rabbitmq.client.Channel;

import org.springframework.amqp.rabbit.annotation.*;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

/**
 * @Author tho
 * @Date 2021/12/4/20:27
 * @ProjectName RabbitMQ
 * @ClassName: RabbitMQReceiver
 * @Description: RabbitMQ消费者
 */
@Component
public class RabbitMQReceiver {

    /**
     * 组合使用监听
     * @RabbitListener  @QueueBinding @Queue @Exchange
     * @param message
     * @param channel
     * @throws Exception
     */
    @RabbitListener(
            bindings = @QueueBinding(
                    value = @Queue(value = "queue-1", durable = "true"),
                    exchange = @Exchange(name = "${spring.rabbitmq.listener.order.exchange.name}",
                            durable = "true",
                            type = "topic",
                            ignoreDeclarationExceptions = "true"),
                    key = "springboot.*"
            )
    )
    @RabbitHandler
    public void onMessage(Message message, Channel channel) throws Exception {
        // 1.收到消息后进行业务端消费处理
        System.err.println("----------------------");
        System.err.println("消费消息" + message.getPayload());

        // 2.处理成功之后,获取 deliveryTag 并进行手工的Ack操作，因为配置文件里配置了手工签收
        // ## 消费者消费成功后需要手工进行签收(ack),默认为auto
        // spring.rabbitmq.listener.simple.acknowledge-mode=manual
        Long deliveryTag = (Long) message.getHeaders().get(AmqpHeaders.DELIVERY_TAG);
        channel.basicAck(deliveryTag, false);

    }
}
```

### 3.可能会报错

> RabbitMQ默认端口是5672
>
> 配置文件输入host时仅需要输入ip地址即可
>
> 配置文件中有关于端口的配置
>
> ```java
> spring.rabbitmq.port=5672
> ```



RabbitMQ设置了自己的用户,需要进入控制面板给自己的用户相应的权限

> Admin选项
>
> 点击自己设定的用户
>
> 点击 Set permission 设定权限

### 4.测试整合是否成功

#### 启动消费者项目

- 进入RabbitMQ,可以看到设定了相应的exchange和queue
- Overview 和 Connections 界面也发生了一些变化

#### 启动生产者的组件,进行测试

消费者的测试类

```java
package com.test.producer;

import com.tho.Application;
import com.tho.producer.component.RabbitMQSender;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;

/**
 * @Author tho
 * @Date 2021/12/4/20:58
 * @ProjectName RabbitMQ
 * @ClassName: ApplicationTests
 * @Description: RabbitMQ生产者的测试类
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ApplicationTests {

    @Autowired
    private RabbitMQSender rabbitMQSender;

    @Test
    public void SenderTest() throws Exception{


        Map<String, Object> properties = new HashMap<>();
        properties.put("attr1", "12");
        properties.put("attr2", "abc");
        rabbitMQSender.send("hello rabbitMQ", properties);

        Thread.sleep(10000);
    }
}
```

## 2.7、RabbitMQ集群架构模式

### 1.主备模式

> 主从模式 主节点可以读写 从节点只能读
>
> 主备模式 主节点正常提供服务,从节点不提供任何服务(主节点宕机,备用节点提供新的服务)

- 实现RabbitMQ的高可用集群,一般在并发和数据量也不高的情况下,这种模型非常好用且简单。主备模式也称之为Warrent模式

- 一般在并发和数据量不高的情况下使用主备模式,配置简单,维护简单 中小型公司常用 
- 借助Haproxy实现主备模式

### 2.远程模式

- 远程模式可以实现双活的一种模式，简称Shovel模式,Shovel就是我们可以把消息进行不同数据中心的复制工作，我们可以跨地域地让两个mq集群互联
- 借助 rabbitmq_shovel 插件实现

### 3.镜像模式

- 集群最经典的就是Mirror镜像模式,保证100%数据不丢失，在实际工作中也是用的最多的。并且实现集群非常简单
- 为了保证RabbitMQ数据的高可靠性方案，对于100%数据可靠性一般是三个节点

# 三、RabbitMQ基础组件封装

## 3.1、基础组件关键点

- 一线大厂MQ组件实现思路和架构设计方案
- 基础组件封装设计-迅速消息发送
- 基础组件封装设计-确认消息发送
- 基础组件封装设计-延迟消息发送

> 基础组件实现功能点
>
> - 迅速，延迟，可靠
> - 消息异步化，序列化
> - 连接池化，高性能
> - 完备的补偿机制

## 3.2、基础组件模块

父工程 pom.xml

 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tho</groupId>
    <artifactId>RabbitMQ</artifactId>
    <packaging>pom</packaging>
    <version>1.0</version>
    <modules>
        <module>RabbitMQ-producer</module>

        <module>RabbitMQ-consumer</module>
    </modules>
    <!--Springboot 依赖-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath />
    </parent>
    <!--相关配置-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <fasterxml.uuid.version>3.1.4</fasterxml.uuid.version>
        <org.codehaus.jackson.version>2.1.4</org.codehaus.jackson.version>
        <druid.version>1.0.24</druid.version>
        <elastic-job.version>2.1.4</elastic-job.version>
        <guava.version>20.0</guava.version>
        <commons-lang3.version>3.3.1</commons-lang3.version>
        <commons-io.version>2.4</commons-io.version>
        <commons-collections.version>3.2.2</commons-collections.version>
        <curator.version>2.11.0</curator.version>
        <fastjson.version>1.1.26</fastjson.version>



    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!--使用自定义日志框架 屏蔽自带的日志框架-->
            <!--<exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>-->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--Springboot 整合RabbitMQ-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>${guava.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>${commons-io.version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
        <!--对json格式的支持-->
        <dependency>
            <groupId>org.codehaus.jackson</groupId>
            <artifactId>jackson-mapper-asl</artifactId>
            <version>1.9.13</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.uuid</groupId>
            <artifactId>java-uuid-generator</artifactId>
            <version>${fasterxml.uuid.version}</version>
        </dependency>
    </dependencies>

</project>
```

- RabbitMQ-common ：一些通用模块
- RabbitMQ-api：对外提供接口模块
- RabbitMQ-task：一些定时任务
- RabbitMQ-core-producer：延迟发送等业务实现

## 3.3、API模块封装

> serialVersionUID IDea自动生成需要在设置里进行设置
>
> https://blog.csdn.net/hetongun/article/details/81904393

## 3.4、消息的可靠性投递

### 1.集成数据源

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>RabbitMQ</artifactId>
        <groupId>com.tho</groupId>
        <version>1.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>RabbitMQ-common</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.tho</groupId>
            <artifactId>RabbitMQ-api</artifactId>
            <version>1.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
        </dependency>
        <!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.41</version>
        </dependency>
    </dependencies>
</project>
```

> 需要的 broker_message 表 消息投递日志表

```sql
-- 表 rabbitmq.broker_message 结构
CREATE TABLE IF NOT EXISTS `broker_message` (
	`message_id` VARCHAR(128) NOT NULL,
	`message` VARCHAR(3600),
	`try_count` int(4) DEFAULT 0,
	`status` VARCHAR(10) DEFAULT '',
	`next_retry` timestamp NOT NULL DEFAULT '2019-07-01 00:00:00',
	`create_time` timestamp NOT NULL DEFAULT '1970-07-01 12:26:55',
	`update_time` timestamp NOT NULL DEFAULT '2019-07-01 06:06:55',
	PRIMARY KEY (`message_id`)

)ENGINE=INNODB DEFAULT CHARSET=utf8;
```

## 3.5、消息补偿

> elastic job 当当网
>
> 分布式定时任务

# 四、分布式定时任务

## 4.1、Elastic-Job

- 需要借助zookeeper实现,需要配置好zookeeper环境



> 可以通过zookeeper来修改elastic-job的相关参数
>
> 利用ealstic-job的后台管理程序进行修改,可以更方便地修改elastic-job
>
> 后台还可以查看定时任务的执行日志记录

## 4.2、SimpleJob

- 最简单的job



## 4.3、DataFlowJob

- 可以实时抓取数据的Job