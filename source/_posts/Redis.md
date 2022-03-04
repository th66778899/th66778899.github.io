---
title: Redis
date: 2021-11-20 12:24:50
tags: Redis
categories:
- Redis
index_img: /img/redis.png

---

Redis

<!--more-->

# 一、Redis基础

Nosql常见分类

- 键值对数据库  Redis,Memcache
- 列存储数据库  Hbase,Cassandra
- 文档型数据库  MongoDB,CouchDB
- 图形数据库     Neo4j,FlockDB

> 分布式缓存 
>
> - 提示读取速度性能
> - 分布式计算领域
> - 为数据库降低查询压力
> - 跨服务器缓存
> - 内存式缓存

> Redis 
>
> - 分布式缓存中间件
> - key-value 存储
> - 提供海量数据存储访问,读取更快,非关系型,分布式,开源,水平扩展

## 1.1、缓存方案对比

> Ehcache 
>
> - 基于java开发,基于jvm缓存 简单，轻巧，方便
> - 集群,分布式不支持
>
> Memcache
>
> - 简单的 key-value存储
> - 内存使用率较高
> - 多核，多线程
> - 无法容灾,无法持久化
>
> Redis
>
> - 丰富的数据结构
> - 持久化
> - 主从同步,故障转移,内存数据库
> - 单线程

## 1.2、Redis环境配置

（Linux环境)

1. 官网下载redis安装包 `redis-6.2.6.tar.gz`

2.  建立 `/usr/local/redis` 文件夹并进入

	`tar zxvf /root/workspace/software/redis-6.2.6.tar.gz -C ./`

3. 进入 `redis-6.2.6` 文件夹

	执行命令 `make && make install`

5. 将配置文件由  /usr/local/redis/redis-6.2.6  拷贝一份到  `/usr/local/redis` 下

6. 修改配置文件

	```JAVA
	bind 127.0.0.1 改为 bind 0.0.0.0 								# 保证外网能够访问本地redis
	#requirepass foobared 改为   requirepass 123456 # redis访问时密码
	daemonize no 改为 daemonize yes 								# 保证redis后台运行 
	dir ./ 改为  dir /usr/local/redis/workspace 		# 指定Redis工作目录
	```

7. redis开机自启动

	拷贝 `/usr/local/redis/redis-6.2.6/utils`目录下  `redis_init_script` 到 /etc/init.d

	脚本内容为

	```JAVA
	#!/bin/sh
	#
	# Simple Redis init.d script conceived to work on Linux systems
	# as it does use of the /proc filesystem.
	
	### BEGIN INIT INFO
	# Provides:     redis_6379
	# Default-Start:        2 3 4 5
	# Default-Stop:         0 1 6
	# Short-Description:    Redis data structure server
	# Description:          Redis data structure server. See https://redis.io
	### END INIT INFO
	
	#chkconfig 22345 10 90
	#description: Start and Stop redis
	
	REDISPORT=6379
	EXEC=/usr/local/bin/redis-server
	CLIEXEC=/usr/local/bin/redis-cli
	
	PIDFILE=/var/run/redis_${REDISPORT}.pid
	CONF="/usr/local/redis/redis.conf"  # 要改为配置文件所在位置
	
	case "$1" in
	    start)
	        if [ -f $PIDFILE ]
	        then
	                echo "$PIDFILE exists, process is already running or crashed"
	        else
	                echo "Starting Redis server..."
	                $EXEC $CONF
	        fi
	        ;;
	    stop)
	        if [ ! -f $PIDFILE ]
	        then
	                echo "$PIDFILE does not exist, process is not running"
	        else
	                PID=$(cat $PIDFILE)
	                echo "Stopping ..."
	                $CLIEXEC -p $REDISPORT shutdown
	                while [ -x /proc/${PID} ]
	                do
	                    echo "Waiting for Redis to shutdown ..."
	                    sleep 1
	                done
	                echo "Redis stopped"
	        fi
	        ;;
	    *)
	        echo "Please use start or stop as first argument"
	        ;;
	esac
	
	```

	8.`ps -ef | grep redis` 查看redis启动情况

	9.redis_init_script 启动脚本中加入密码 41行代码

	```JAVA
	#!/bin/sh
	#
	# Simple Redis init.d script conceived to work on Linux systems
	# as it does use of the /proc filesystem.
	
	### BEGIN INIT INFO
	# Provides:     redis_6379
	# Default-Start:        2 3 4 5
	# Default-Stop:         0 1 6
	# Short-Description:    Redis data structure server
	# Description:          Redis data structure server. See https://redis.io
	### END INIT INFO
	
	#chkconfig 22345 10 90
	#description: Start and Stop redis
	
	REDISPORT=6379
	EXEC=/usr/local/bin/redis-server
	CLIEXEC=/usr/local/bin/redis-cli
	
	PIDFILE=/var/run/redis_${REDISPORT}.pid
	CONF="/usr/local/redis/redis.conf"
	
	case "$1" in
	    start)
	        if [ -f $PIDFILE ]
	        then
	                echo "$PIDFILE exists, process is already running or crashed"
	        else
	                echo "Starting Redis server..."
	                $EXEC $CONF
	        fi
	        ;;
	    stop)
	        if [ ! -f $PIDFILE ]
	        then
	                echo "$PIDFILE does not exist, process is not running"
	        else
	                PID=$(cat $PIDFILE)
	                echo "Stopping ..."
	                $CLIEXEC -a "123456" -p $REDISPORT shutdown # 加入redis密码 -a "123465"
	                while [ -x /proc/${PID} ]
	                do
	                    echo "Waiting for Redis to shutdown ..."
	                    sleep 1
	                done
	                echo "Redis stopped"
	        fi
	        ;;
	    *)
	        echo "Please use start or stop as first argument"
	        ;;
	esac
	
	```

	> 在`/etc/init.d` 目录下 通过   可以停止redis服务  
	>
	> `./redis_init_script start` 启动redis
	>
	> `./redis_init_script stop` 关闭redis

## 1.3、Redis基本数据类型

### 1、String

> String 键值对
>
> Redis 默认16个库，默认使用0号库

```java
get/set/del  查/改/删
set key value // 会覆盖存在的值
setnx key value // 不会覆盖存在的值
set key value ex time // 设置过期时间数据
expire age 30 // 30s过期时间
ttl - time to leave // 查看过期时间  
  
append key : 合并/追加字符串
strlen key：字符串长度


incr // 累加 incrby
decr // 累减 decrby
getrange // 截取数据 end = -1 表示截取到最后
setrange // 从start位置开始替换数据
mset  // 连续设值
mget  // 连续取值
msetnx  // 连续设值,如果存在则不设置
  
select index // 选择数据库
flushdb // 清除本库内容
flushall // 清除所有库内容 
```

### 2、Hash

> hash

```JAVA
127.0.0.1:6379> hset user name tho
(integer) 1
127.0.0.1:6379> hget user name
"tho"
127.0.0.1:6379> hmset user age 19 sex "man"
OK
127.0.0.1:6379> hmget user name age
1) "tho"
2) "19"
127.0.0.1:6379> hgetall user
1) "name"
2) "tho"
3) "age"
4) "19"
5) "sex"
6) "man"
127.0.0.1:6379> hlen user
(integer) 3
127.0.0.1:6379> hkeys user
1) "name"
2) "age"
3) "sex"
127.0.0.1:6379> hvals user
1) "tho"
2) "19"
3) "man"
127.0.0.1:6379> hincrby user age 6
(integer) 25
127.0.0.1:6379> hincrby user age 6
(integer) 31
127.0.0.1:6379> hincrby user age 6
(integer) 37
127.0.0.1:6379> hincrbyfloat user age 0.6
"37.6"
127.0.0.1:6379> hincrbyfloat user age 0.6
"38.2"
127.0.0.1:6379> hincrbyfloat user age 0.6
"38.8"
127.0.0.1:6379> hexists user age
(integer) 1
127.0.0.1:6379> hdel user age
(integer) 1
127.0.0.1:6379> hdel user name
(integer) 1
127.0.0.1:6379> hgetall user
1) "sex"
2) "man"
127.0.0.1:6379> hdel user sex
(integer) 1
127.0.0.1:6379> hgetall user
(empty array)

```

### 3、List

> list

```JAVA
127.0.0.1:6379> lpush list1 pig pow sheep duck
(integer) 4
127.0.0.1:6379> lrange list1 0 -1
1) "duck"
2) "sheep"
3) "pow"
4) "pig"
127.0.0.1:6379> rpush list2 pig pow sheep duck
(integer) 4
127.0.0.1:6379> lrange list2 0 -1
1) "pig"
2) "pow"
3) "sheep"
4) "duck"
127.0.0.1:6379> lpop list1
"duck"
127.0.0.1:6379> lrange list1 0 -1
1) "sheep"
2) "pow"
3) "pig"
127.0.0.1:6379> rpop list1
"pig"
127.0.0.1:6379> lrange list1 0 -1
1) "sheep"
2) "pow"
127.0.0.1:6379> llen list1
(integer) 2
127.0.0.1:6379> llen list2
(integer) 4
127.0.0.1:6379> rpop list2
"duck"
127.0.0.1:6379> lindex list1 0
"sheep"
127.0.0.1:6379> lindex list1 1
"pow"
127.0.0.1:6379> lset list1 0 123556
OK
127.0.0.1:6379> lrange list1 0 -1
1) "123556"
2) "pow"
127.0.0.1:6379> linsert list1 before pow 999
(integer) 3
127.0.0.1:6379> lrange list1 0 -1
1) "123556"
2) "999"
3) "pow"
127.0.0.1:6379> linsert list1 after pow 999
(integer) 4
127.0.0.1:6379> lrange list1 0 -1
1) "123556"
2) "999"
3) "pow"
4) "999"
127.0.0.1:6379> lrem list1 2 999
(integer) 2
127.0.0.1:6379> lrange list1 0 -1
1) "123556"
2) "pow"
127.0.0.1:6379> ltrim list1 1 -1
OK
127.0.0.1:6379> lrange list1 0 -1
1) "pow"
127.0.0.1:6379> del list2
(integer) 1
127.0.0.1:6379> lrange list2 0 -1
(empty array)
127.0.0.1:6379> keys *
1) "list1"
```

### 4、Set

> set不重复  集合

```JAVA
127.0.0.1:6379> sadd set duck pig cow sheep pig cow
(integer) 4
127.0.0.1:6379> smembers set
1) "sheep"
2) "cow"
3) "duck"
4) "pig"
127.0.0.1:6379> scard set
(integer) 4
127.0.0.1:6379> sismember set pig
(integer) 1
127.0.0.1:6379> sismember set pi
(integer) 0
127.0.0.1:6379> srem set duck
(integer) 1
127.0.0.1:6379> smembers set
1) "cow"
2) "pig"
3) "sheep"
127.0.0.1:6379> spop set 
"cow"
127.0.0.1:6379> spop set 
"pig"
127.0.0.1:6379> sadd set cow 
(integer) 1
127.0.0.1:6379> sadd set pig
(integer) 1
127.0.0.1:6379> sadd set1 1 2 3 4 5 6 7 8 9
(integer) 9
127.0.0.1:6379> smembers set1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "8"
9) "9"
127.0.0.1:6379> srandmember set1 3
1) "5"
2) "6"
3) "8"
127.0.0.1:6379> srandmember set1 3
1) "5"
2) "9"
3) "4"
127.0.0.1:6379> srandmember set1 3
1) "9"
2) "3"
3) "1"
127.0.0.1:6379> srandmember set1 3
1) "7"
2) "4"
3) "8"
127.0.0.1:6379> srandmember set1 3
1) "2"
2) "1"
3) "4"
127.0.0.1:6379>  sadd set2 2 1 3 7 9 11 13 15  
(integer) 8
127.0.0.1:6379> smove set1 set2 6
(integer) 1
127.0.0.1:6379> smembers set1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "7"
7) "8"
8) "9"
127.0.0.1:6379> smembers set2
1) "1"
2) "2"
3) "3"
4) "6"
5) "7"
6) "9"
7) "11"
8) "13"
9) "15"
127.0.0.1:6379> sdiff set1 set2
1) "4"
2) "5"
3) "8"
127.0.0.1:6379> sinter set1 set2
1) "1"
2) "2"
3) "3"
4) "7"
5) "9"
127.0.0.1:6379> sunion set1 set2
 1) "1"
 2) "2"
 3) "3"
 4) "4"
 5) "5"
 6) "6"
 7) "7"
 8) "8"
 9) "9"
10) "11"
11) "13"
12) "15"
```

### 5、Zset

> Zset 有序的set sorted set

```java
127.0.0.1:6379> zadd zset 10 duck 20 pig 30 chicken 40 beef 50 sheep
(integer) 5
127.0.0.1:6379> zrange zset 0 -1
1) "duck"
2) "pig"
3) "chicken"
4) "beef"
5) "sheep"
127.0.0.1:6379> zrange zset 0 -1 withscores
 1) "duck"
 2) "10"
 3) "pig"
 4) "20"
 5) "chicken"
 6) "30"
 7) "beef"
 8) "40"
 9) "sheep"
10) "50"
127.0.0.1:6379> zadd zset 25 abc 35 xyz
(integer) 2
127.0.0.1:6379> zrange zset 0 -1 withscores
 1) "duck"
 2) "10"
 3) "pig"
 4) "20"
 5) "abc"
 6) "25"
 7) "chicken"
 8) "30"
 9) "xyz"
10) "35"
11) "beef"
12) "40"
13) "sheep"
14) "50"
127.0.0.1:6379> rank zset beef
(error) ERR unknown command `rank`, with args beginning with: `zset`, `beef`, 
127.0.0.1:6379> zrank zset beef
(integer) 5
127.0.0.1:6379> zrange zset 0 -1 withscores
 1) "duck"
 2) "10"
 3) "pig"
 4) "20"
 5) "abc"
 6) "25"
 7) "chicken"
 8) "30"
 9) "xyz"
10) "35"
11) "beef"
12) "40"
13) "sheep"
14) "50"
127.0.0.1:6379> zrange zset 0 -1 
1) "duck"
2) "pig"
3) "abc"
4) "chicken"
5) "xyz"
6) "beef"
7) "sheep"
127.0.0.1:6379> zscore zset beef
"40"
127.0.0.1:6379> zcard zset
(integer) 7
127.0.0.1:6379> zcount zset 20 40
(integer) 5
127.0.0.1:6379>  
127.0.0.1:6379> zrangebyscore 20 40
(error) ERR wrong number of arguments for 'zrangebyscore' command
127.0.0.1:6379> zrangebyscore zset 20 40
1) "pig"
2) "abc"
3) "chicken"
4) "xyz"
5) "beef"
127.0.0.1:6379> zrangebyscore zset 20 40 withscores
 1) "pig"
 2) "20"
 3) "abc"
 4) "25"
 5) "chicken"
 6) "30"
 7) "xyz"
 8) "35"
 9) "beef"
10) "40"
127.0.0.1:6379> zrangebyscore zset 20 (40 withscores
1) "pig"
2) "20"
3) "abc"
4) "25"
5) "chicken"
6) "30"
7) "xyz"
8) "35"
127.0.0.1:6379> zrangebyscore zset (20 (40 withscores
1) "abc"
2) "25"
3) "chicken"
4) "30"
5) "xyz"
6) "35"
127.0.0.1:6379> zrangebyscore zset 20 40 limit 1 2 
1) "abc"
2) "chicken"
127.0.0.1:6379> zrangebyscore zset 20 40 limit 2 2 
1) "chicken"
2) "xyz"
127.0.0.1:6379> zrangebyscore zset 20 40 limit 3 2 
1) "xyz"
2) "beef"
127.0.0.1:6379> zrem zset pig
(integer) 1
127.0.0.1:6379> zrange zset 0 -1 
1) "duck"
2) "abc"
3) "chicken"
4) "xyz"
5) "beef"
6) "sheep"
```

# 二、Redis使用

## 2.1、Springboot整合Redis

pom.xml 依赖

```xml
  <!--redis依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

> maven - 要使用install命令进行安装

> redisTemplate 简单使用

```java
package com.tho.controller;


import com.tho.utils.RedisOperator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import springfox.documentation.annotations.ApiIgnore;

@ApiIgnore
@RestController
@RequestMapping("redis")
public class RedisController {

    // 配置显示日志
    // final static Logger logger = LoggerFactory.getLogger(RedisController.class);

    /*@Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private StringRedisTemplate stringRedisTemplate;*/
  
		// RedisOperator 是一个redis工具类
    @Autowired
    private RedisOperator redisOperator;
  	
    @PostMapping("/set")
    public Object set(String key, String value){

        // redisTemplate.opsForValue().set(key, value);
        redisOperator.set(key, value);

        return "yes";
    }

    @GetMapping("/get")
    public Object get(String key){
        /* Object o = redisTemplate.opsForValue().get(key); */
        String s = redisOperator.get(key);
        return s;
    }

    @PostMapping("/del")
    public Object del(String key){
        // Boolean delete = redisTemplate.delete(key);
        Boolean del = redisOperator.del(key);

        return del;
    }
}

```

## 2.2、RedisTemplate

> 问题

- 直接使用redisTemplate 进行操作redis 查看其在redis所设置的值可以看到redis里面的值都有前缀,是因为默认采用的序列化方式是 `org.springframework.data.redis.serializer.JdkSerializationRedisSerializer`

- 将序列化的方式改为 `org.springframework.data.redis.serializer.StringRedisSerializer` 会自动去掉`\xac\xed\x00\x05t\x00`前缀

> 解决

方案一：使用 `StringRedisTemplate`

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;
```

方案二：修改默认的序列化方式：

```JAVA
@Autowired
private RedisTemplate redisTemplate;
@Autowired(required = false)
public void setRedisTemplate(RedisTemplate redisTemplate) {
    RedisSerializer stringSerializer = new StringRedisSerializer();
    redisTemplate.setKeySerializer(stringSerializer);
    redisTemplate.setValueSerializer(stringSerializer);
    redisTemplate.setHashKeySerializer(stringSerializer);
    redisTemplate.setHashValueSerializer(stringSerializer);
    this.redisTemplate = redisTemplate;
}
```

方案三：使用封装了`redisTemplate` 的 RedisOperator 工具类 (存储的键值对没有了前缀)

```JAVA
package com.tho.utils;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * @Title: Redis 工具类
 * @author 慕课网
 */
@Component
public class RedisOperator {
	
//	@Autowired
//    private RedisTemplate<String, Object> redisTemplate;
	
	@Autowired
	private StringRedisTemplate redisTemplate;
	
	// Key（键），简单的key-value操作

	/**
	 * 实现命令：TTL key，以秒为单位，返回给定 key的剩余生存时间(TTL, time to live)。
	 * 
	 * @param key
	 * @return
	 */
	public long ttl(String key) {
		return redisTemplate.getExpire(key);
	}
	
	/**
	 * 实现命令：expire 设置过期时间，单位秒
	 * 
	 * @param key
	 * @return
	 */
	public void expire(String key, long timeout) {
		redisTemplate.expire(key, timeout, TimeUnit.SECONDS);
	}
	
	/**
	 * 实现命令：INCR key，增加key一次
	 * 
	 * @param key
	 * @return
	 */
	public long incr(String key, long delta) {
		return redisTemplate.opsForValue().increment(key, delta);
	}

	/**
	 * 实现命令：KEYS pattern，查找所有符合给定模式 pattern的 key
	 */
	public Set<String> keys(String pattern) {
		return redisTemplate.keys(pattern);
	}

	/**
	 * 实现命令：DEL key，删除一个key
	 * 
	 * @param key
	 */
	public Boolean del(String key) {
		Boolean delete = redisTemplate.delete(key);
		return delete;
	}

	// String（字符串）

	/**
	 * 实现命令：SET key value，设置一个key-value（将字符串值 value关联到 key）
	 * 
	 * @param key
	 * @param value
	 */
	public void set(String key, String value) {
		redisTemplate.opsForValue().set(key, value);
	}

	/**
	 * 实现命令：SET key value EX seconds，设置key-value和超时时间（秒）
	 * 
	 * @param key
	 * @param value
	 * @param timeout
	 *            （以秒为单位）
	 */
	public void set(String key, String value, long timeout) {
		redisTemplate.opsForValue().set(key, value, timeout, TimeUnit.SECONDS);
	}

	/**
	 * 实现命令：GET key，返回 key所关联的字符串值。
	 * 
	 * @param key
	 * @return value
	 */
	public String get(String key) {
		return (String)redisTemplate.opsForValue().get(key);
	}

	// Hash（哈希表）

	/**
	 * 实现命令：HSET key field value，将哈希表 key中的域 field的值设为 value
	 * 
	 * @param key
	 * @param field
	 * @param value
	 */
	public void hset(String key, String field, Object value) {
		redisTemplate.opsForHash().put(key, field, value);
	}

	/**
	 * 实现命令：HGET key field，返回哈希表 key中给定域 field的值
	 * 
	 * @param key
	 * @param field
	 * @return
	 */
	public String hget(String key, String field) {
		return (String) redisTemplate.opsForHash().get(key, field);
	}

	/**
	 * 实现命令：HDEL key field [field ...]，删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。
	 * 
	 * @param key
	 * @param fields
	 */
	public void hdel(String key, Object... fields) {
		redisTemplate.opsForHash().delete(key, fields);
	}

	/**
	 * 实现命令：HGETALL key，返回哈希表 key中，所有的域和值。
	 * 
	 * @param key
	 * @return
	 */
	public Map<Object, Object> hgetall(String key) {
		return redisTemplate.opsForHash().entries(key);
	}

	// List（列表）

	/**
	 * 实现命令：LPUSH key value，将一个值 value插入到列表 key的表头
	 * 
	 * @param key
	 * @param value
	 * @return 执行 LPUSH命令后，列表的长度。
	 */
	public long lpush(String key, String value) {
		return redisTemplate.opsForList().leftPush(key, value);
	}

	/**
	 * 实现命令：LPOP key，移除并返回列表 key的头元素。
	 * 
	 * @param key
	 * @return 列表key的头元素。
	 */
	public String lpop(String key) {
		return (String)redisTemplate.opsForList().leftPop(key);
	}

	/**
	 * 实现命令：RPUSH key value，将一个值 value插入到列表 key的表尾(最右边)。
	 * 
	 * @param key
	 * @param value
	 * @return 执行 LPUSH命令后，列表的长度。
	 */
	public long rpush(String key, String value) {
		return redisTemplate.opsForList().rightPush(key, value);
	}

}
```

## 2.3、使用Redis保存购物车

> cookie中的购物车和redis中的购物车
>
> 两者内保存信息的不同要合理处理,以redis中的购物车信息为主

## 2.4、发布于订阅

publisher -> subscriber

```JAVA
订阅者：
  SUBSCRIBE food tho-bigdata tho-backend tho -frontend
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "food"
3) (integer) 1
1) "subscribe"
2) "tho-bigdata"
3) (integer) 2
1) "subscribe"
2) "tho-backend"
3) (integer) 3
1) "subscribe"
2) "tho"
3) (integer) 4
1) "subscribe"
2) "-frontend"
3) (integer) 5
1) "message"
2) "tho-bigdata"
3) "javaversion"
1) "message"
2) "food"
3) "chaobing"
发布者：
  127.0.0.1:6379> PUBLISH tho-bigdata javaversion
	(integer) 2
	127.0.0.1:6379> PUBLISH food chaobing
	(integer) 2
	127.0.0.1:6379> 

```

> 批量订阅

```JAVA
127.0.0.1:6379> PSUBSCRIBE tho*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "tho*"
3) (integer) 1
1) "pmessage"
2) "tho*"
3) "thox"
4) "chaobing"

```

## 2.5、Redis持久化

### 1、RDB

>redis.conf 配置文件
>
>rdb适合使用大数据量的数据库备份
>
>最后一次保存出现问题会导致数据的完整性出现问题
>
>全量备份模式

### 2、AOF

>以日志形式进行追加
>
>flushall 误操作可以删除aof文件中的相关语句
>
>重启redis

## 2.6、Vmware 克隆虚拟机

> 要修改克隆机的静态ip,使得克隆机可以通过Xshell 访问

https://www.jianshu.com/p/29e3f4f3cbe7

## 2.7、Redis主从复制

> 单机redis 并发量 10w +

> 读写分离 - 主从复制
>
> 
>
> master节点将数据写入自己磁盘,将磁盘内数据传输到slave节点,slave节点读取数据库数据,进行复制
>
> 
>
> 第一次同步 全量复制 之后 增量复制
>
> Redis Master
>
> (数据复制 RDB)
>
> Redis Slave



- 搭建redis主从复制

> info replication 展示redis信息

```shell
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:4a8892cdb96c411b16e19891fb40bb658d8bd2db
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

```

> 只需要修改从节点的配置文件 redis.conf
>
> 重启redis

```java
# redis 主从复制配置
# replicaof <masterip> <masterport>
replicaof 192.168.198.100 6379
# redis暴露在外网有风险,最好设置密码
masterauth 123456
# 从节点只能读,不能写
replica-read-only yes
```

> 主节点的数据以及数据变更都会同步到从节点
>
> 且从节点不能写入

```JAVA
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.198.102,port=6379,state=online,offset=596,lag=1
slave1:ip=192.168.198.101,port=6379,state=online,offset=596,lag=0
master_failover_state:no-failover
master_replid:5957efff30070c8a728c5a86ff9067353bad417e
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:596
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:596

```

> 主节点 从节点 关闭 重启 其主从关系不会改变,主节点会自动成为主节点

## 2.8、无磁盘化复制

> 不需要使用磁盘,利用socket 来完成复制,磁盘速度低,网络带宽较高

## 2.9、Redis缓存过期机制

- (主动) 定期删除

	定时检查key,expire已符合key过期的要去,删去key

- (被动) 惰性删除

	key被访问到,该key过期的话,删除该key,内存会被一直占用

> 内存淘汰管理机制

- MEMORY MANAGEMENT
- maxmemory 当内存使用率已经到达设定值,则开始清理缓存

> 缓存更新策略
>
> - noeviction：旧缓存永不过期,新缓存设置不了,返回错误
> - allkeys-lru：清除最少用的旧缓存,然后保存新的缓存(推荐使用)
> - allkeys-random：在所有的缓存中随机删除(不推荐)
> - volatile-lru：在那些设置了expire过期时间的缓存中,清除最少使用的旧缓存,然后保存新的缓存
> - volatile-random：在那些设置了expire过期时间的缓存中,随机删除缓存
> - volatile-ttl：在那些设置了expire过期时间的缓存中,删除即将过期的

# 三、Redis集群

## 3.1、Redis哨兵

> Sentinel(哨兵) 是用于监控Redis集群中Master状态的工具,是Redis高可用解决方案,哨兵可以监视一个或者多个Redis master服务,以及这些Master服务的所有从服务;当某个master服务器宕机后,会把这个master下的某个从服务升级为master来代替已宕机的master继续工作

1.配置Redis哨兵,sentinel.conf 配置文件修改

```java
# 关闭保护模式
< protected-mode no
> # protected-mode no
# 开启守护进程,后台运行
< daemonize yes
---
> daemonize no
# sentinel机制的日志目录
< logfile /usr/local/redis/sentinel/redis-sentinel.log
---
> logfile ""
# sentinel 工作目录
< dir /usr/local/redis/sentinel
---
> dir /tmp
# sentine机制设定主节点 最后的参数2代表启动几个哨兵
< sentinel monitor tho_master 192.168.198.100 6379 2
---
> sentinel monitor mymaster 127.0.0.1 6379 2
# 设置sentinel机制的密码 
< sentinel auth-pass tho_master 123456
---
> 

< 
# sentinel机制下 主节点失效的判断时间
< sentinel down-after-milliseconds tho_master 10000
---
> sentinel down-after-milliseconds mymaster 30000

< # 主从节点并行同步的数量
< sentinel parallel-syncs tho_master 1
---
> sentinel parallel-syncs mymaster 1

< # 主节点宕机,主从切换的时间
< sentinel failover-timeout tho_master 180000
---
> sentinel failover-timeout mymaster 180000

```

使用 `scp sentinel.conf root@192.168.198.101:/usr/local/redis` 将主节点配置好的sentinel配置文件传输给从节点

2.创建redis-sentinel日志文件夹

3.使用命令 `redis-sentinel sentinel.conf` 启动redis哨兵



## 3.2、Redis原master节点恢复

> 原master节点恢复后,会自动变成slave,但是其同步状态有问题  `master_link_status:down`
>
> 可能是因为只设置了 两个从节点中redis.conf中的 `masterauth` ,这是用于同步master中的数据,但是一开始原master节点是不受影响的,当原master转换为slave后,由于其没有设置masterauth,所以不能从新的master同步数据,随之导致`info replication` 的时候,同步状态为down,所以只需要修改`redis.conf` 中的`masterauth` 为 `123456` 及redis设置的密码即可



> 一般master数据无法同步给slave的检查方案如下
>
> 1. 网络通信问题,要保证互相ping通,内网互通
> 2. 关闭防火墙,对应的端口打开(虚拟机建议永久关闭防火墙,云服务器的话要保证内网互通)
> 3. 统一所有的密码,不要漏了某个节点没有设置

## 3.3、Redis哨兵信息查看

> 查看Redis哨兵详细信息
>
> `[root@centos_7_100 init.d]# redis-cli -p 26379
> 127.0.0.1:26379> sentinel master tho_master`
>
> 查看redis哨兵 从节点信息
>
> `127.0.0.1:26379> sentinel slaves tho_master`
>
> 查看 redis 哨兵 哨兵的信息
>
> `127.0.0.1:26379> sentinel sentinels tho_master`



> 哨兵部署约定
>
> - 哨兵节点至少要有至少三个或者奇数个节点
> - 哨兵分布式部署在不同的计算机节点(部署在一台服务器上的不同端口,如果该服务器宕机,所有的哨兵全部下线)
> - 一组哨兵只监听一组主从

## 3.4、Springboot集成Redis哨兵

> 修改配置文件

```yaml
# redis 配置
  redis:
      # redis单机单实例
#    host: 192.168.198.100
#    password: 123456
#    port: 6379
#    database: 0
    # redis 哨兵配置
    database: 0
    password: 123456
    sentinel:
      master: tho_master
      nodes: 192.168.198.100:26379,192.168.198.101:26379,192.168.198.102:26379
```

## 3.5、Redis集群

> 主从复制以及哨兵,它们可以提高读的并发,但是单个master节点容量有限,数据达到一定程度会有瓶颈,这个时候通过水平扩展为多master-slave成为集群
> 

### 1、Redis-cluster

> 可以支撑多个master-slave,支持海量数据,实现高可用与高并发
>
> 哨兵模式也是一种集群,它能够提高读请求的并发,但是容错方面可能会有一些问题,比如master同步数据给slave的时候,其实是异步复制,这个时候master宕机了,那么slave上的数据就没有master上的新,数据同步需要时间,1-2s左右的数据会丢失。master节点恢复转换成slave后,新数据则丢失

- 集群特点

	> 1. 每个节点知道彼此之间的关系,也会知道自己的角色,当然它们也会知道自己处于一个集群当中,它们彼此之间可以交互和通信,比如ping,pong。这些关系会保存到某个配置文件中,每个节点都有,这个在搭建的时候会进行配置,
	> 2. 客户端要和集群建立连接的话,只需要和其中一个建立关系就行
	> 3. 某个节点挂了,也是通过超过半数的节点来进行检测，客观下线后主从切换,和之前的哨兵模式是一样的道理
	> 4. Redis中存在很多的插槽,又可以称之为槽节点,用于存储数据

- 集群容错

	> 构建redis集群，至少需要3个节点作为master，以此组成一个高可用的集群,此外每个master都需要配置6个节点,这也是最经典的redis集群，也可以称之为三主三从,容错性更佳

### 2、集群搭建

 1. 修改各个服务器redis配置文件

	redis.conf

	```JAVA
	# 开启集群模式
	cluster-enabled yes
	# 每一个节点需要一个配置文件,需要6份,每个处于集群的节点角色需要告知其它所有节点,彼此知道,这个文件用于存储集群模式下的集群状态等信息,这个文件是由redis自己维护的,重新创建集群 删掉该文件即可
	cluster-config-file nodes-100.conf
	# 超时时间,超时则认为master宕机,随后主备切换
	cluster-node-timeout 5000
	# 开启AOF
	appendonly yes
	```

	2. 启动6个redis实例

	> 如果启动过程出错，把rdb等文件删除清空

	3. 创建集群

	> redis3.x版本 需要使用redis-trib.rb 来构建集群
	>
	> 新版redis试域redis-cli 来构建集群

	```JAVA
	# 创建集群 主从节点比例为 1 : 1  1-3节点为主 4-6节点为从
	redis-cli -a 123456 --cluster create ip1:port1 ip2:port2 ip3:port3 ip4:port4 ip5:port5 ip6:port6 --cluster-replicates 1
	# 要进行密码验证
	-a <password>
	```

	- slots: 槽 用于存储数据,主节点有,从节点没有

- 检查集群信息

	`redis-cli -a 123456 --cluster check 192.168.198.100:6379` 

## 3.6、Slot槽节点

> 将槽节点平分给master节点    槽总数:16384
>
> 内存槽 - Slot槽节点 

# 四、Redis缓存穿透

## 4.1、Redis缓存穿透

> 数据库中不存在的记录,大量请求数据库会造成数据库很大压力
>
> 即使是不存在数据库中的记录,也保存到redis中,保护数据库
>
> 问题：某个key 在未来是可能用到的 ,redis也有该key的数据，不修改代码的情况下,该key的值不会更新(缓存一致性)



> 要查询的key在redis中不存在,在数据库中也不存在,此时非法用户进行攻击,大量请求会打在数据库上，造成宕机，影响整个系统的运行，这种现象成为缓存穿透
>
> 解决方案：把空的数据也缓存起来，比如空字符串，空对象，空数组或者 list

## 4.2、布隆过滤器

> - 误判率 <-> 数组的大小 两者取舍
>
> - 无法删除数据 

## 4.3、缓存雪崩

> 大面积的缓存超过过期时间,全部失效



==雪崩预防==

- 设置key永不过期
- 过期时间错开
- 多缓存结合
- 采购第三方redis数据库服务器

# 五、批量查询优化

## 5.1、multiget优化

> 循环批量查找

```JAVA
		@Test
    public void testRedis() {
        ArrayList<String> result = new ArrayList<>();
        ArrayList<String> params = new ArrayList<String>(){{
            add("a");
            add("b");
            add("c");
        }};
        for (String param : params) {
            result.add(redisOperator.get(param));
        }
        for (String s : result) {
            System.out.println(s);
        }


    }
```

>批量查询 同 mget

```java
		@Test
    public void testRedis() {
        // ArrayList<String> result = new ArrayList<>();
        ArrayList<String> params = new ArrayList<String>(){{
            add("a");
            add("b");
            add("c");
        }};
        /*for (String param : params) {
            result.add(redisOperator.get(param));
        }*/
        List<String> strings = redisOperator.mGet(params);
        for (String s : strings) {
            System.out.println(s);
        }


    }
```



## 5.2、pipeline批量查询优化

> 无需因为循环多次请求连接
>
> pipeline 可以获取更多的键值类型
>
> pipeline可以做更多的操作

```JAVA
// 工具类内的方法
/**
	* @Author tho
	* @Date 2021/11/21 13:52
	* @param keys
	* @Return List<String>
	* @Description: 通过pipeline管道批量查询key
	*/
	public List<Object> pipelineBatchGet(List<String> keys) {

		List<Object> result = redisTemplate.executePipelined(new RedisCallback<String>() {
			@Override
			public String doInRedis(RedisConnection connection) throws DataAccessException {
				StringRedisConnection src = (StringRedisConnection)connection;
				for (String key : keys) {
					src.get(key);
				}

				return null;
			}
		});
		return result;
	}
```

