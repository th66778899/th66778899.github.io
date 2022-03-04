---
title: Kafka
date: 2021-11-30 15:35:56
tags: 
- Kafka
categories:
- 消息队列
index_img: /img/kafka.png



---



Kafka基本使用

<!--more-->







- Kafka应用实战
- Kafka高吞吐量日志收集实战
- 架构思考：分布式日志，跟踪，警告，分析平台

# 一、Kafka基础

## 1.1、Kafka介绍

- Kafka是LinkedIn开源的分布式消息系统，目前归属于Apache顶级项目
- Kafka主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始目的就是用于日志收集和传输
- 0.8版本开始支持复制，不支持事务，对消息的丢失，重复，错误没有严格要去，适合产生大量数据的互联网服务的数据收集业务。

> Kafka特点
>
> - 分布式特性
> - 跨平台的特性
> - 实时性
> - 伸缩性

kafka的主要特点： 

- 同时为发布和订阅提供高吞吐量。据了解，Kafka每秒可以生产约25万消息（50 MB），每秒处理55万消息（110 MB）。
-  可进行持久化操作。将消息持久化到磁盘，因此可用于批量消费，例如ETL，以及实时应用程序。通过将数据持久化到硬盘以及r 止数据丢失。 
- 分布式系统，易于向外扩展。所有的producer、broker和consumer都会有多个，均为分布式的。无需停机即可扩展机器。
-  消息被处理的状态是在consumer端维护，而不是由server端维护。当失败时能自动平衡。
-  支持online和offline的场景。



## 1.2、Kafka高性能原因

- 顺序写，磁盘顺序写(不进行物理删除)
- Page Cache空中接力，高效读写
- 后台异步，主动Flush
- 预读策略,IO调度

## 1.3、PageCache

- 操作系统主要实现的磁盘缓存(减少对磁盘IO的操作)，磁盘内容存储到内存中作为缓存
- 用户请求磁盘上某文件,操作系统不是直接从磁盘上读取，而是先从PageCache中找有没有该文件信息，如果没有,从磁盘读取，而且先将该文件加入到PageCache中，之后才会将该文件返回给用户 
- 写操作类似
- 零拷贝(Zero copy)

## 1.4、Kafka集群模式

- 借助zookeeper实现集群

## 1.5、Kafka的架构

​	Kafka的整体架构非常简单，是显式分布式架构，producer、broker（kafka）和consumer都可以有多个。Producer，consumer实现Kafka注册的接口，数据从pr broker，broker承担一个中间缓存和分发的作用。broker分发注册到系统中的consumer。broker的作用类似于缓存，即活跃的数据和离线处理系统之间的缓存。客 器端的通信，是基于简单，高性能，且与编程语言无关的TCP协议。

> 基本概念

- Topic：特指Kafka处理的消息源（feeds of messages）的不同分类。 
- Partition：Topic物理上的分组，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分 序的id（offset）。 
- Message：消息，是通信的基本单位，每个producer可以向一个topic（主题）发布一些消息。 
- Producers：消息和数据生产者，向Kafka的一个topic发布消息的过程叫做producers。
- Consumers：消息和数据消费者，订阅topics并处理其发布的消息的过程叫做consumers。 
- Broker：缓存代理，Kafka集群中的一台或多台服务器统称为broker。

> 发送消息的流程

- Producer根据指定的partition方法（round-robin、hash等），将消息发布到指定topic的partition里面 
- kafka集群接收到Producer发过来的消息后，将其持久化到硬盘，并保留消息指定时长（可配置），而不关注消息是否被消费。
-  Consumer从kafka集群pull数据，并控制获取消息的offset

## 1.6、kafka的优秀设计

> 从kafka的吞吐量、负载均衡、消息拉取、扩展性来说一说kafka的优秀设计。

==高吞吐==是kafka需要实现的核心目标之一，为此kafka做了以下一些设计：

- 内存访问：直接使用 linux 文件系统的cache，来高效缓存数据，对数据进行读取和写入。
-  数据磁盘持久化：消息不在内存中cache，直接写入到磁盘，充分利用磁盘的顺序读写性能。 
- zero-copy：减少IO操作步骤 
	- 采用linux Zero-Copy提高发送性能。传统的数据发送需要发送4次上下文切换，采用sendfile系统调用之后，数据直接在内核 统上下文切换减少为2次。根据测试结果，可以提高60%的数据发送性能。Zero-Copy详细的技术细节可以参考：linux官方零拷贝的介绍 
- 对消息的处理： 支持数据批量发送

# 二、Kafka基本使用

## 2.1、环境配置

⾸先准备ZOOKEEPER服务

因为 Kafka 的运⾏环境依赖于 ZooKeeper ，所以⾸先得安装并运⾏ ZooKeeper 。 

### 1.准备Kafka安装包

这⾥下载的是 3.0.0 版： kafka_2.12-3.0.0.tgz ，将下载后的安装包放在了 /root/workspace/software ⽬录下

### 2.解压并安装

1. /usr/local/ 下创建 kafka ⽂件夹并进⼊

```shell
[root@centos_7_100 ~]# cd /usr/local
[root@centos_7_100 local]# mkdir kafka
[root@centos_7_100 local]# cd kafka
```

2. 将Kafka安装包解压到 /usr/local/kafka 中即可

```java
[root@localhost kafka]# tar -zxvf /root/workspace/software/kafka_2.12-3.0.0.tgz -C ./
```

3. 解压完之后， /usr/local/kafka ⽬录中会出现⼀个 kafka_2.12-3.0.0 的⽬录

### 3.创建LOGS⽬录

这⾥直接在 /usr/local/kafka/kafka_2.12-3.0.0 ⽬录中创建⼀个 logs ⽬录

等下该 logs ⽬录地址要配到Kafka的配置⽂件中。

### 4.修改配置⽂件

进⼊到 Kafka 的 config ⽬录，编辑配置⽂件 `server.properties`

修改配置⽂件，⼀是将其中的 log.dirs 修改为上⾯刚创建的 logs ⽬录，其他选项可以按需配置

另外关注⼀下连接 ZooKeeper 的相关配置，根据实际情况进⾏配置：

```java
// logs相关配置
log.dirs=/usr/local/kafka/kafka_2.12-3.0.0/logs
// zookeeper相关配置
// root directory for all kafka znodes.
zookeeper.connect=localhost:2181

// Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000
```

### 5.启动Kafka

> 启动kafka前必须要先启动zookeeper,否则会报错

执⾏如下命令即可：

```java
./bin/kafka-server-start.sh ./config/server.properties
```

如果需要后台启动，则加上 -daemon 参数即可

```java
./bin/kafka-server-start.sh  -daemon ./config/server.properties
```

### 6.实验验证

⾸先创建⼀个名为 tho 的 topic ：

```shell
./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic tho

[root@centos_7_100 ~]# cd /usr/local/kafka/kafka_2.12-3.0.0/
[root@centos_7_100 kafka_2.12-3.0.0]# ./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic tho
Created topic tho.

```

创建完成以后，可以使⽤命令来列出⽬前已有的 topic 列表

```shell
./bin/kafka-topics.sh --list --bootstrap-server localhost:9092

[root@centos_7_100 kafka_2.12-3.0.0]# ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
tho
```

查看group信息

```shell
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group group02
```



### 7.可以同时开启三个ssh连接

> 一个连接观察kafka是否启动成功
>
> 一个用来创建生产者
>
> 一个用来创建消费者
>
> 生产者发送消息的同时可以通过消费者来观察是否成功产生消息



接下来创建⼀个⽣产者，⽤于在 codesheep 这个 topic 上⽣产消息

```shell
./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic tho
```

⽽后接着创建⼀个消费者，⽤于在 codesheep 这个 topic 上获取消息：

```shell
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic tho
```

此时⽣产者发出的消息，在消费者端可以获取到

### 8.实验过程中如果出现⼀些诸如客户端不能连通或访问等问题

> 先查看zookeeper是否成功启动
>
> ps -ef | grep zookeeper

### 或者可尝试考虑关闭防⽕墙：

```shell
# 关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service
# 打开防火墙

systemctl start firewalld.service
systemctl enable firewalld.service

# 或者开启9092端口
# 打开单个端口：
firewall-cmd --zone=public --add-port=9092/tcp --permanent
# 重启防火墙    
systemctl restart firewalld.service
firewall-cmd --list-ports
```

## 2.2、SpringBoot整合-producer

### 1.maven配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tho</groupId>
    <artifactId>Kafka</artifactId>
    <packaging>pom</packaging>
    <version>1.0</version>
    <modules>
        <module>Kafka-producer</module>
        <module>Kafka-consumer</module>
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
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
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
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
    </dependencies>
</project>
```



### 2.application.properties

```properties
server.servlet.context-path=/producer
server.port=8001

## Spring 整合 kafka
spring.kafka.bootstrap-servers=192.168.198.100:9092
# kafka发送消息失败时重试的次数
spring.kafka.producer.retries=0
# kafka批量发送数据的配置
spring.kafka.producer.batch-size=16384
# 设置kafka生产者内存缓存区大小(32m)
spring.kafka.producer.buffer-memory=33554432
# kafka消息的序列化配置
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
# 这个是kafka生产端最重要的选项
# ack=0 ：生产者在成功写入消息之前不会等待任何来自服务器的响应
# ack=1 ：只要集群的首领节点收到消息，生产者就会收到一个来自服务器的成功响应
# acks=-1: 表示分区leader必须等待消息被成功写入到所有的ISR副本(同步副本)中才认为producer请求成功。这种方案提供最高的消息持久性保证，但是理论上吞吐率也是最差的。
spring.kafka.producer.acks=1
```

### 3.创建KafkaTemplate

```java
package com.tho.kafka.producer;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Component;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.util.concurrent.ListenableFutureCallback;

/**
 * @Author tho
 * @Date 2021/12/11/15:45
 * @ProjectName Kafka
 * @ClassName: KafkaProducerService
 * @Description: Kafka生产者服务类
 */
@Component
@Slf4j
public class KafkaProducerService {

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    public void sendMessage(String topic, Object object) {
        ListenableFuture<SendResult<String, Object>> future = kafkaTemplate.send(topic, object);

        future.addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {
            @Override
            public void onFailure(Throwable throwable) {
                log.info("发送消息失败:" + throwable.getMessage());
            }

            @Override
            public void onSuccess(SendResult<String, Object> result) {
                log.info("发送消息成功:" + result.toString());
            }
        });
    }

}
```

## 2.3、SpringBoot整合-cosumer

> maven pom.xml 配置同producer

### 1.application.properties

```properties
server.servlet.context-path=/consumer
server.port=8002

## Spring 整合 kafka
spring.kafka.bootstrap-servers=192.168.198.100:9092
# kafka consumer消息签收机制:手工签收
spring.kafka.consumer.enable-auto-commit=false

spring.kafka.listener.ack-mode=manual
# 该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：
# latest（默认值）在偏移量无效的情况下，消费者将从最新的记录开始读取数据（在消费者启动之后生成的记录）
# earliest ：在偏移量无效的情况下，消费者将从起始位置读取分区的记录
spring.kafka.consumer.auto-offset-reset=earliest
## 序列化配置
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer

spring.kafka.listener.concurrency=5
```

### 2.consumer实现类

```java
package com.tho.kafka.consumer;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.stereotype.Component;

/**
 * @Author tho
 * @Date 2021/12/11/15:59
 * @ProjectName Kafka
 * @ClassName: KafkaConsumerService
 * @Description: Kafka消费者服务类
 */
@Component
@Slf4j
public class KafkaConsumerService {

    @KafkaListener(groupId = "group02", topics = "topic02")
    public void onMessage(ConsumerRecord<String, Object> record,
                          Acknowledgment acknowledgment,
                          Consumer<?, ?> consumer) {

        log.info("消费端接收消息:{}", record.value());
        // 手工签收机制
        acknowledgment.acknowledge();
    }
}
```

## 2.4、测试

### 1.producer测试

> producer测试类

```java
package kafka.test;

import com.tho.kafka.Application;
import com.tho.kafka.producer.KafkaProducerService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * @Author tho
 * @Date 2021/12/11/15:50
 * @ProjectName Kafka
 * @ClassName: KafkaTest
 * @Description: Kafka 测试
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class KafkaTest {
    @Autowired
    private KafkaProducerService kafkaProducerService;
    @Test
    public void kafkaProducerTest() throws InterruptedException {

        String topic = "topic02";
        for (int i = 0; i < 6; i++) {
            kafkaProducerService.sendMessage(topic, "hello kafka:" + i);
        }
        Thread.sleep(Integer.MAX_VALUE);


    }
}
```

> 如果报错

```java
Error connecting to node node1:9092 (id: 2 rack: null) java.net.UnknownHostException
```

> 可能是kafka无法根据节点名称获得服务器ip地址
>
> 需要在windows系统中修改hosts文件
>
> ip地址对应 服务器的名称 也就是服务器的hostname

> 修改为 
>
> 192.168.198.100  centos_7_100

windows系统中,在命令行中断 cmd 中使用命令 `ipconfig/flushdns` 刷新dns缓存

### 2.consumer测试

> 启动kafka consumer 通过Application.java 
>
> 可以看到group的信息

```shell
[root@centos_7_100 kafka_2.12-3.0.0]# ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group group02

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                     HOST             CLIENT-ID
group02         topic02         0          0               12              12              consumer-2-a54662f3-5d45-45d7-bf0e-5a169c307a11 /192.168.198.110 consumer-2

```

可以打印出consumer消费的信息

# 三、Kafka收集海量日志

## ELK

==注意的点:东八区的时间 8个小时 logstash kinaba 搜索的时候要处理好时间==

==watcher告警 搜索的字段是关键字(日志级别),错误日志信息的索引要加入模板，映射为keyword==

## 3.1、架构设计

适合海量数据的官方结构设计

![image-20211211173259526](/img/Kafka.assets/image-20211211173259526.png)



![image-20211211173747917](/img/Kafka.assets/image-20211211173747917.png)



![image-20211211173915636](/img/Kafka.assets/image-20211211173915636.png)

## 3.2、日志输出

- Log4j2 (性能更好一些,占用服务器资源更多)
- Log4j

> 日志分级 		warn info error 级别的日志
>
> 日志过滤 		需要的/不需要的
>
> MDC线程变量 日志里的ThreadLocal

## 3.3、整合Springboot

### 1.maven依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!--Springboot 依赖-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath />
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>Kafka-logCollect</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- 	排除spring-boot-starter-logging -->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>

        </dependency>
        <!-- 	排除spring-boot-starter-logging -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>*</groupId>
                    <artifactId>*</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!-- log4j2 强依赖于disruptor框架-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
        <dependency>
            <groupId>com.lmax</groupId>
            <artifactId>disruptor</artifactId>
            <version>3.3.4</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.58</version>
        </dependency>

    </dependencies>
    <build>
        <finalName>collector</finalName>
        <!-- 打包时包含properties、xml -->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <!-- 是否替换资源中的属性-->
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
              	<!--springboot项目 不加会报错 没有主清单属性-->
                <configuration>
                    <mainClass>com.tho.log.collect.Application</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2.application.properties

```properties
server.servlet.context-path=/
server.port=8001

spring.application.name=kafka-logCollect
spring.http.encoding.charset=UTF-8
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
spring.jackson.default-property-inclusion=NON_NULL
```

### 3.log4j2.xml

> log4j2 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO" schema="Log4J-V2.0.xsd" monitorInterval="600" >
    <Properties>
        <Property name="LOG_HOME">logs</Property>
        <property name="FILE_NAME">Kafka-logCollect</property>
        <property name="patternLayout">[%d{yyyy-MM-dd'T'HH:mm:ss.SSSZZ}] [%level{length=5}] [%thread-%tid] [%logger] [%X{hostName}] [%X{ip}] [%X{applicationName}] [%F,%L,%C,%M] [%m] ## '%ex'%n</property>
    </Properties>
    <Appenders>
        <Console name="CONSOLE" target="SYSTEM_OUT">
            <PatternLayout pattern="${patternLayout}"/>
        </Console>  
        <RollingRandomAccessFile name="appAppender" fileName="${LOG_HOME}/app-${FILE_NAME}.log" filePattern="${LOG_HOME}/app-${FILE_NAME}-%d{yyyy-MM-dd}-%i.log" >
          <PatternLayout pattern="${patternLayout}" />
          <Policies>
              <TimeBasedTriggeringPolicy interval="1"/>
              <SizeBasedTriggeringPolicy size="500MB"/>
          </Policies>
          <DefaultRolloverStrategy max="20"/>         
        </RollingRandomAccessFile>
        <RollingRandomAccessFile name="errorAppender" fileName="${LOG_HOME}/error-${FILE_NAME}.log" filePattern="${LOG_HOME}/error-${FILE_NAME}-%d{yyyy-MM-dd}-%i.log" >
          <PatternLayout pattern="${patternLayout}" />
          <Filters>
              <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
          </Filters>              
          <Policies>
              <TimeBasedTriggeringPolicy interval="1"/>
              <SizeBasedTriggeringPolicy size="500MB"/>
          </Policies>
          <DefaultRolloverStrategy max="20"/>         
        </RollingRandomAccessFile>            
    </Appenders>
    <Loggers>
        <!-- 业务相关 异步logger -->
        <AsyncLogger name="com.tho.*" level="info" includeLocation="true">
          <AppenderRef ref="appAppender"/>
        </AsyncLogger>
        <AsyncLogger name="com.tho.*" level="info" includeLocation="true">
          <AppenderRef ref="errorAppender"/>
        </AsyncLogger>       
        <Root level="info">
            <Appender-Ref ref="CONSOLE"/>
            <Appender-Ref ref="appAppender"/>
            <AppenderRef ref="errorAppender"/>
        </Root>         
    </Loggers>
</Configuration>
```

### 4.打印的日志

```java
// 日志格式
[%d{yyyy-MM-dd'T'HH:mm:ss.SSSZZ}]   时间
[%level{length=5}]  								日志级别
[%thread-%tid] 											线程信息
[%logger] 													具体的类 
[%X{hostName}] 											host名称
[%X{ip}] 														ip地址
[%X{applicationName}] 							应用名称
[%F,%L,%C,%M]												F 当前执行的类(file文件) L 行号 C class信息 M 方法(method) 
[%m]  															日志info信息
## '%ex'%n                          ex异常信息 %n 表示换行

[2021-12-11T18:34:13.024+08:00] 
[INFO] 
[main-1] 
[org.springframework.boot.web.embedded.tomcat.TomcatWebServer]
[] 
[] 
[] [TomcatWebServer.java,204,org.springframework.boot.web.embedded.tomcat.TomcatWebServer,start] 
[Tomcat started on port(s): 8001 (http) with context path ''] 
  ## ''
```

## 3.4、filebeat日志收集

### 环境搭建

```java
filebeat安装：

cd /usr/local
// 安装包放在 /root/workspace/software
tar -zxvf filebeat-7.15.2-linux-x86_64.tar.gz -C /usr/local/filebeat
cd /usr/local
mv filebeat-7.15.2-linux-x86_64.tar.gz/ filebeat-7.15.2

## 配置filebeat，可以参考filebeat.full.yml中的配置。
vim /usr/local/filebeat-5.6.2/filebeat.yml

filebeat启动：

## 检查配置是否正确
cd /usr/local/filebeat-6.6.0
./filebeat -c filebeat.yml -configtest
## Config OK

## 启动filebeat
/usr/local/filebeat-6.6.0/filebeat &
ps -ef | grep filebeat

可以通过查看 kafka的日志文件和 工程运行生成的log日志文件进行比对,当工程生成的log更新之后,查看kafka日志是否更新来判断filebeat是否启动成功


## 启动kafka：
./bin/kafka-server-start.sh ./config/server.properties

## 查看topic列表：
./bin/kafka-topics.sh --list --bootstrap-server localhost:9092

## 创建topic
./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic app-Kafka-logCollect
  
./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic error-Kafka-logCollect





## 查看topic情况
kafka-topics.sh --zookeeper 192.168.11.111:2181 --topic app-log-test --describe


```

filebeat配置文件  filebeat.yml

```yaml


# ============================== Filebeat inputs ===============================

filebeat.prospectors:

# Each - is an input. Most options can be set at the input level, so
# you can use different inputs for various configurations.
# Below are the input specific configurations.

- input_type: log

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /root/project/logs/app-Kafka-logCollect.log
    #- c:\programdata\elasticsearch\logs\*
  document_type: "app-log"
  multiline: 
    pattern: '^\[' # 中括号开头为日志记录
    negate: true    # 是否匹配到
    match: after    # 合并到上一行的末尾
    max_lines: 2000 # 最大的行数
    timeout: 2s     # 如果在规定的时间内没有新的日志事件,就不用等待后面的日志信息
  fields:
    logbiz: Kafka-logCollect
    logtopic: app-Kafka-logCollect  # 按服务划分 用作kafka topic
    evn: dev
  
- input_type: log


  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /root/project/logs/error-Kafka-logCollect.log
  document_type: "error-log"
  multiline: 
    pattern: '^\[' # 中括号开头为日志记录
    negate: true    # 是否匹配到
    match: after    # 合并到上一行的末尾
    max_lines: 2000 # 最大的行数
    timeout: 2s     # 如果在规定的时间内没有新的日志事件,就不用等待后面的日志信息
  fields:
    logbiz: Kafka-logCollect
    logtopic: error-Kafka-logCollect  # 按服务划分 用作kafka topic
    evn: dev



# ============================== Filebeat modules ==============================
output.kafka:
    enable: true
    hosts: ["192.168.198.100:9092"]
    topic: '%{[fields.logtopic]}'
    
    partition.hash:
      reachable_only: true
    compression: gzip
    max_message_bytes: 1000000
    required_acks: 1
logging.to_files: true


```

## 3.5、logstash日志过滤

### 1.logstash配置

logstash目录下 /usr/local/logstash/logstash-6.4.3 新建文件夹 script

script文件夹下新建配置文件 logstash-script.conf

```java
input {
	kafka {
		## app-log-服务名称
		topics_pattern => "app-Kafka-.*"
		bootstrap_servers => "192.168.198.100:9092"
		codec => json
		consumer_threads => 1 ## 增加consumer并行消费线程数
		decorate_events => true
		# auto_offset_rest => "latest"
		group_id => "app-log-group"
	}
	
	kafka {
		## app-log-服务名称
		topics_pattern => "error-Kafka-.*"
		bootstrap_servers => "192.168.198.100:9092"
		codec => json
		consumer_threads => 1 ## 增加consumer并行消费线程数
		decorate_events => true
		# auto_offset_rest => "latest"
		group_id => "error-log-group"
	}
}

filter {
	
	## 时区转换
	ruby {
		code => "event.set('index_time', event.timestamp.time.localtime.strftime('%Y.%m.%d'))"
	}
	
	if	"app-Kafka" in [fields][logtopic] {
		grok {
			## 表达式
			match => ["message", "\[%{NOTSPACE:currentDateTime}\] \[%{NOTSPACE:level}\] \[%{NOTSPACE:thread-id}\] \[%{NOTSPACE:class}\] \[%{DATA:hostName}\] \[%{DATA:ip}\] \[%{DATA:applicationName}\] \[%{DATA:location}\] \[%{DATA:messageInfo}\] ## (\'\'|%{QUOTEDSTRING:throwable})"]
		}
	}
	
	if	"error-Kafka" in [fields][logtopic] {
		grok {
			## 表达式
			match => ["message", "\[%{NOTSPACE:currentDateTime}\] \[%{NOTSPACE:level}\] \[%{NOTSPACE:thread-id}\] \[%{NOTSPACE:class}\] \[%{DATA:hostName}\] \[%{DATA:ip}\] \[%{DATA:applicationName}\] \[%{DATA:location}\] \[%{DATA:messageInfo}\] ## (\'\'|%{QUOTEDSTRING:throwable})"]
		}
	}
	
	
	
}
## 测试输出到控制台
output {
	stdout {codec => rubydebug}
}

## 输出到elasticSearch
output {
	
	if	"app-Kafka" in [fields][logtopic] {
		
		## es插件
		elasticsearch {
		# es地址
		hosts => ["192.168.198.100:9200"]
		# 同步的索引名
		index => "app-log-%{[fields][logbiz]}-%{index_time}"
		# 设置 _docID 和 数据相同
		# document_id => "%{itemId}" 
		
		# 定义模板名称
		# template_name => "myik"
		# 模板所在位置
		# template => "/usr/local/logstash/logstash-6.4.3/sync/logstash-ik.json"
		# 重写模板
		# template_overwrite => true
		# 默认为true,false关闭logstash自动管理模板功能，如果自定义模板,则设置为false
		manage_template => true
		}
	}
	
	if	"error-Kafka" in [fields][logtopic] {
		
		## es插件
		elasticsearch {
		# es地址
		hosts => ["192.168.198.100:9200"]
		# 同步的索引名
		index => "app-log-%{[fields][logbiz]}-%{index_time}"
		# 设置 _docID 和 数据相同
		# document_id => "%{itemId}" 
		
		# 定义模板名称
		# template_name => "myik"
		# 模板所在位置
		# template => "/usr/local/logstash/logstash-6.4.3/sync/logstash-ik.json"
		# 重写模板
		# template_overwrite => true
		# 默认为true,false关闭logstash自动管理模板功能，如果自定义模板,则设置为false
		manage_template => true
		}
	}

	
}
```

### 2.启动logstash

> 进入logstash 的 bin目录 执行命令 `./logstash -f /usr/local/logstash/logstash-6.4.3/script/logstash-script.conf`

> 可以通过kafka来查看结果 查看group信息

```shell
# logstash-script.conf 配置文件中配置了两种日志类型各自的groupId 
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group app-log-group
```

## 3.6、日志持久化，可视化

> ElasticSearch 索引创建周期,命名规范选择
>
> Kibana控制台应用，可视化日志
>
> 监控告警 Watcher 插件基本使用 WatchApi

- 启动filebeat
- kafka (启动kafka之前需要启动zk)
- logstash

启动ElasticSearch => 将日志持久化到es上

配置好kibana控制台

### 1.配置kibana

> - 解压缩kibana安装包
> - 修改配置文件 kibana.yml

```java
server.port: 5601
server.host: "0.0.0.0"
server.name: "kibana_198.100"
elasticsearch.hosts: ["http://192.168.198.100:9200"]
#elasticsearch.username: "kibana_system"
#elasticsearch.password: "pass"
i18n.locale: "zh-CN"
```

### 2.启动kibana

#### **启动kibana**

该命令在kibana的bin目录下面

启动时间会稍微较长

`./kibana --allow-root`

### 3.kibana配置

> elasticSearch 和 kibana全部配置好 x-pack插件
>
> elasticSearch 设置账户密码 依赖于ealsticSearch 的 应用要配置好es的账户密码

> kibana配置可视化查看es数据
>
> kibana界面左侧最底部的 
>
> 1.Stack Management
>
> 2.Kibana选项下的 索引模式
>
> 3.创建索引模式
>
> 4.名称 app-log-* 查看右侧是否匹配成功 
>
> ​	时间戳字段选择 currentDateTime
>
> 5.创建索引成功,回到主页,选择Analytics下的 Discover 查看具体的es信息

## 3.7、watcher监控告警

### 1.watch配置

```java
## 创建一个watcher,比如定义一个trigger 每个10s钟看一下input里的数据
## 创建一个watcher,比如定义一个trigger 每个5s钟看一下input里的数据
PUT _xpack/watcher/watch/error_log_collector_watcher
{
  "trigger": {
    "schedule": {
      "interval": "5s"
    }
  },
  "input": {
    "search": {
      "request": {
        "indices": ["<error-log-error-collect-{now+8h/d}>"],
        "body": {
          "size": 0,
          "query": {
            "bool": {
              "must": [
                  {
                    "term": {"level": "ERROR"}
                  }
              ],
              "filter": {
                "range": {
                    "currentDateTime": {
                    "gt": "now-30s" , "lt": "now"
                  }
                }
              } 
            }
          }
        }
      }
    }
  },

  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "gt": 0
      }
    }
  },
 
  "transform": {
    "search": {
      "request": {
        "indices": ["<error-log-error-collect-{now+8h/d}>"],
        "body": {
          "size": 1,
          "query": {
            "bool": {
              "must": [
                  {
                    "term": {"level": "ERROR"}
                  }
              ],
              "filter": {
                "range": {
                    "currentDateTime": {
                    "gt": "now-30s" , "lt": "now"
                  }
                }
              } 
            }
          },
          "sort": [
            {
                "currentDateTime": {
                    "order": "desc"
                }
            }
          ]
        }
      }
    }
  },
  "actions": {
    "test_error": {
      "webhook" : {
        "method" : "POST",
        "url" : "http://192.168.198.100:8001/accurateWatch",
        "body" : "{\"title\": \"异常错误告警\", \"applicationName\": \"{{#ctx.payload.hits.hits}}{{_source.applicationName}}{{/ctx.payload.hits.hits}}\", \"level\":\"告警级别P1\", \"body\": \"{{#ctx.payload.hits.hits}}{{_source.messageInfo}}{{/ctx.payload.hits.hits}}\", \"executionTime\": \"{{#ctx.payload.hits.hits}}{{_source.currentDateTime}}{{/ctx.payload.hits.hits}}\"}"
      }
    }
 }
}

# 查看一个watcher
# 
GET _xpack/watcher/watch/error_log_collector_watcher


#删除一个watcher
DELETE _xpack/watcher/watch/error_log_collector_watcher

#执行watcher
# POST _xpack/watcher/watch/error_log_collector_watcher/_execute

#查看执行结果
GET /.watcher-history*/_search?pretty
{
  "sort" : [
    { "result.execution_time" : "desc" }
  ],
  "query": {
    "match": {
      "watch_id": "error_log_collector_watcher"
    }
  }
}

GET error-log-collector-2019.09.18/_search?size=10
{

  "query": {
    "match": {
      "level": "ERROR"
    }
  }
  ,
  "sort": [
    {
        "currentDateTime": {
            "order": "desc"
        }
    }
  ] 
}


GET error-log-collector-2019.09.18/_search?size=10
{

  "query": {
    "match": {
      "level": "ERROR"
    }
  }
  ,
  "sort": [
    {
        "currentDateTime": {
            "order": "desc"
        }
    }
  ] 
}

```

### 2.创建错误日志信息模板

> 错误的日志 error-log 索引需要的模板如下
>
> 需要该索引字段来进行watcher的监控警告

```json
PUT _template/error-log-
{
  "template": "error-log-*",
  "order": 0,
  "settings": {
    "index": {
      "refresh_interval": "5s"
    }
  },
  "mappings": {
    
      "dynamic_templates": [
        {
          "message_field": {
            "match_mapping_type": "string",
            "path_match": "message",
            "mapping": {
              "norms": false,
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word"
            }
          }
        },
        {
          "throwable_field": {
            "match_mapping_type": "string",
            "path_match": "throwable",
            "mapping": {
              "norms": false,
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word"
            }
          }
        },
        {
          "string_fields": {
            "match_mapping_type": "string",
            "match": "*",
            "mapping": {
              "norms": false,
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word",
              "fields": {
                "keyword": {
                  "type": "keyword"
                }
              }
            }
          }
        }
      ],
      "properties": {         
        "hostName": {
          "type": "keyword"
        },
        "ip": {
          "type": "ip"
        },
        "level": {
          "type": "keyword"
        },
		"currentDateTime": {
		  "type": "date"
		}
      }
    
  }
}
```

### 3.可以ack掉watcher告警信息

## 3.8、watcher相关学习