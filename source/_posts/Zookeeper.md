---
title: 分布式锁
date: 2021-12-26 12:18:26
tags: 分布式锁
categories:
- 分布式相关
index_img: /img/zookeeper.png
---

SpringCloud基础组件















分布式锁相关

<!--more-->

# 一、分布式锁

## 2.1、超卖问题

### 1.超卖现象一

==某商品库存10件，结果卖出了15件==

> 超卖现象:解决办法
>
> - 扣减库存不在程序中进行,而是通过数据库
> - 向数据库传递库存增量，扣减1个库存，增量为 -1
> - 在数据库 update 语句计算库存，通过update 行锁解决并发

> 不使用java语句进行商品数量的扣减,而是使用sql的更新语句来进行库存的扣减
>
> - 计算好库存,更新物品库存 (会产生超卖问题)

```sql
<update id="updateProductCount">
      update product
      set count = count - #{purchaseProductNum,jdbcType=INTEGER},
      update_user = #{updateUser,jdbcType=VARCHAR},
       update_time = #{updateTime,jdbcType=TIME}
      where id = #{id,jdbcType=INTEGER}
    </update>
```

> 最终问题:商品库存可能成为负值

### 2.超卖现象二

==商品库存减为了负数==

> 产生原因：并发校验库存,造成库存充足的假想,update更新库存,导致库存成为负数



> 解决办法一：更新了库存之后,进行库存的校验,如果库存为负数,抛出异常,回滚事务,放弃此线程本次操作
>
> 解决办法二：校验库存,扣减库存统一加锁,使之成为原子性操作,并发时,线程只有获得锁才能校验,扣减库存,扣减库存后释放锁，确保库存不会扣除负数

## 2.2、使用锁解决超卖现象

### 1.使用Synchronized锁 

> 方式一：给方法加上Synchronized 关键字
>
> 方式二：使用Synchronized 代码块



#### 方式一:Synchronized 关键字

```java
@Transactional(rollbackFor = Exception.class)
public synchronized Integer createOrder() throws Exception{}
```

> 问题：库存只有1个，最后却生成了两笔订单,最终商品库存变为-1
>
> @Transactional 注解,第一个线程执行完商品的库存扣减,事务没有提交,数据库没有更新,紧接着第二个线程进来,查询商品库存,仍旧是1,还可以生成订单,最后两个库存扣减sql作为同一个事务,进行提交,商品库存减为负数

#### 方式一:解决办法

> 使用synchronized锁包裹事务,只有当事务提交之后,锁才会释放,交给下一个线程来执行方法中的内容
>
> 成功解决问题

```java
@Autowired
private PlatformTransactionManager platformTransactionManager;
@Autowired
private TransactionDefinition transactionDefinition;
public synchronized Integer createOrder() throws Exception{
  			Product product = null;
  			// 定义一个事务
        TransactionStatus transaction = platformTransactionManager.getTransaction(transactionDefinition);
        // 具体的业务流程
  			product = productMapper.selectByPrimaryKey(purchaseProductId);
            if (product==null){
              	// 事务回滚
                platformTransactionManager.rollback(transaction);
                throw new Exception("购买商品："+purchaseProductId+"不存在");
            }
  			//校验库存
        if (purchaseProductNum > currentCount){
         		 // 事务的回滚
     	       platformTransactionManager.rollback(transaction);
             throw new Exception("商品"+purchaseProductId+"仅剩"+currentCount+"件，无法购买");
        }
  			// 提交事务
        platformTransactionManager.commit(transaction);
        return order.getId();
    }
```

#### 方式二：Synchronized块

> 对象锁：每一个对象有一个其自身的对象锁
>
> 类锁：只有一个，该类实例化的所有对象共有一个类锁

```java
// 第一种写法(对象锁)
synchronized (this) {
            
}
// 放在方法内部,该类由spring创建,单例模式,获得了该对象的锁才能执行其中的方法

// 第二种写法(对象锁)
Object object = new Object();
synchronized (object) {

}

// 第三种写法(类锁)
// 避免事务的嵌套
        synchronized (this) {
          	// 创建事务
            TransactionStatus transaction1 = platformTransactionManager.getTransaction(transactionDefinition);
            product = productMapper.selectByPrimaryKey(purchaseProductId);
            if (product==null){
                platformTransactionManager.rollback(transaction1);
                throw new Exception("购买商品："+purchaseProductId+"不存在");
            }

            //商品当前库存
            Integer currentCount = product.getCount();
            System.out.println(Thread.currentThread().getName()+"库存数："+currentCount);
            //校验库存
            if (purchaseProductNum > currentCount){
                platformTransactionManager.rollback(transaction1);
                throw new Exception("商品"+purchaseProductId+"仅剩"+currentCount+"件，无法购买");
            }
            // 使用数据库的update行锁解决超卖问题
            productMapper.updateProductCount(purchaseProductNum,"xxx",new Date(),product.getId());
            platformTransactionManager.commit(transaction1);
        }
```

### 2.使用ReentrantLock锁

```java
private Lock lock = new ReentrantLock();
public Integer createOrder() throws Exception{
        Product product = null;
				// 加锁操作 , 当代码中有异常抛出时,要释放锁,否则异常抛出后,程序执行结束,但是锁还没有被释放,会导致之后的线程永远也拿不到锁
        lock.lock();
        try {
            TransactionStatus transaction1 = platformTransactionManager.getTransaction(transactionDefinition);
            product = productMapper.selectByPrimaryKey(purchaseProductId);
            if (product==null){
                platformTransactionManager.rollback(transaction1);
                throw new Exception("购买商品："+purchaseProductId+"不存在");
            }

            //商品当前库存
            Integer currentCount = product.getCount();
            System.out.println(Thread.currentThread().getName()+"库存数："+currentCount);
            //校验库存
            if (purchaseProductNum > currentCount){
                platformTransactionManager.rollback(transaction1);
                throw new Exception("商品"+purchaseProductId+"仅剩"+currentCount+"件，无法购买");
            }
            // 使用数据库的update行锁解决超卖问题
            productMapper.updateProductCount(purchaseProductNum,"xxx",new Date(),product.getId());
            platformTransactionManager.commit(transaction1);
        } finally {
          	// 解锁
            lock.unlock();
        }
}
```

## 2.3、单体应用锁局限性

通过之前的并发编程的学习，对JAVA中的锁有了深刻的理解。前面内容中讲到的锁都是有JDK官方提供的锁的解决方案，也就是说这些锁只能在一个JVM进程内有效，我们把这种锁叫做**单体应用锁**

>如上图所示，在整个系统架构中，存在两个Tomcat，每个Tomcat是一个JVM。在进行秒杀业务的时候，由于大家都在抢购秒杀商品，大量的请求同时到达系统，通过Nginx分发到两个Tomcat上。我们通过一个极端的案例场景，可以更好地理解单体应用锁的局限性。假如，秒杀商品的数量只有1个，这时，这些大量的请求当中，只有一个请求可以成功的抢到这个商品，这就需要在扣减库存的方法上加锁，扣减库存的动作只能一个一个去执行，而不能同时去执行，如果同时执行，这1个商品可能同时被多个人抢到，从而产生超卖现象。加锁之后，扣减库存的动作一个一个去执行，凡是将库存扣减为负数的，都抛出异常，提示该用户没有抢到商品。通过加锁看似解决了秒杀的问题，但是事实上真					的是这样吗？
>
>我们看到系统中存在两个Tomcat，我们加的锁是JDK提供的锁，这种锁只能在一个JVM下起作用，也就是在一个Tomcat内是没有问题的。当存在两个或两个以上的Tomcat时，大量的并发请求分散到不同的Tomcat上，在每一个Tomcat中都可以防止并发的产生，但是在多个Tomcat之间，每个Tomcat中获得锁的这个请求，又产生了并发，从而产生超卖现象。这也就是**单体应用锁的局限性，它只能在一个JVM内加锁，而不能从这个应用层面去加锁。**
>
>那么这个问题如何解决呢？这就需要使用分布式锁了，在整个应用层面去加锁。什么是分布式锁呢？我们怎么去使用分布式锁呢？

### 1.分布式锁

> 在说分布式锁之前，我们看一看单体应用锁的特点，单体应用锁是在一个JVM进程内有效，无法跨JVM、跨进程。那么分布式锁的定义就出来了，分布式锁就是可以跨越多个JVM、跨越多个进程的锁，这种锁就叫做分布式锁。

![image-20211214163728776](Zookeeper.assets/image-20211214163728776.png)

> 在上图中，由于Tomcat是由Java启动的，所以每个Tomcat可以看成一个JVM，JVM内部的锁是无法跨越多个进程的。所以，我们要实现分布式锁，我们只能在这些JVM之外去寻找，通过其他的组件来实现分布式锁。系统的架构如图所示

![image-20211214163747172](Zookeeper.assets/image-20211214163747172.png)

> 两个Tomcat通过第三方的组件实现跨JVM、跨进程的分布式锁。这就是分布式锁的解决思路，找到所有JVM可以共同访问的第三方组件，通过第三方组件实现分布式锁。

### 2.分布式锁解决方案

分布式锁都是通过第三方组件来实现的，目前比较流行的分布式锁的解决方案有：

- 数据库，通过数据库可以实现分布式锁，但是在高并发的情况下对数据库压力较大，所以很少使用。
- Redis，借助Redis也可以实现分布式锁，而且Redis的Java客户端种类很多，使用的方法也不尽相同。
- Zookeeper，Zookeeper也可以实现分布式锁，同样Zookeeper也存在多个Java客户端，使用方法也不相同。

## 2.4、基于数据库实现分布式锁

### 1.步骤

- 多个线程,多个进程访问共同组件数据库
- 通过select ... for update 访问同一条数据
- for update 锁定某一行数据,其它线程只能等待

> 两线程同时对一行数据加锁,两线程会争抢锁,抢到了锁的线程才能执行下去

### 2.优缺点

- 优点：简单方便,易于理解，易于操作
- 缺点：并发量大时，对数据库压力较大
- 建议：作为锁的数据库和业务数据库分开

## 2.5、基于Redis的Setnx实现分布式锁

### 1.实现原理

> - Redis是单线程执行,即使多线程的请求,最终还是单线程顺序执行,借助Redis这个特性,其相关语句的操作都是原子性的
> - 利用NX的原子性,多个线程并发时，只有一个线程可以设置成功
> - 设置成功即获得锁，可以执行后续的业务处理
> - 如果出现了异常，过了锁的有效期，锁自动释放

- 获取锁的Redis命令
- SET resource_name my_random_value NX PX 30000
	- resource_name ： 资源名称，可根据不同的业务区分不同的锁
	- my_random_value: 随机值,每个线程的随机值都不相同，用于释放锁时的校验
	- NX ： key不存时设置成功,key存在则设置失败
	- PX：自动失效时间,出现异常情况，锁可以过期失效(也就是自动释放锁)

### 2.释放锁

> - 使用redis的delete命令
> - 释放锁校验之前设置的随机数,相同才能释放
> - 释放锁的LUA脚本

```lua
if redis.call("get", KEYS[1]) == ARGV[1] then
  return redis.call("del", KEYS[1])
else
  return 0
end
```

![image-20211214221713182](Zookeeper.assets/image-20211214221713182.png)

### 3.代码实现

```java
@Autowired
private RedisTemplate redisTemplate;

@RequestMapping("redisLock")
public String redisLock() {
    log.info("我进入了方法");
    String key = "redisKey";
    String value = UUID.randomUUID().toString();
    RedisCallback<Boolean> redisCallback = connection -> {
        // 设置NX
        RedisStringCommands.SetOption setOption = RedisStringCommands.SetOption.ifAbsent();
        // 过期时间
        Expiration expiration = Expiration.seconds(30);
        // 序列化key
        byte[] redisKey = redisTemplate.getKeySerializer().serialize(key);
        // 序列化value
        byte[] redisValue = redisTemplate.getValueSerializer().serialize(value);
        // 执行setNx操作
        Boolean result = connection.set(redisKey, redisValue, expiration, setOption);
        return result;

    };
    Boolean lock = (Boolean) redisTemplate.execute(redisCallback);
    if (lock) {
        log.info("我获得了锁");
        try {
            // 执行业务代码
          
          
            Thread.sleep(15000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // 释放redis分布式锁
            String script = "if redis.call(\"get\", KEYS[1]) == ARGV[1] then\n" +
                    "  return redis.call(\"del\", KEYS[1])\n" +
                    "else\n" +
                    "  return 0\n" +
                    "end";
            RedisScript<Boolean> redisScript = RedisScript.of(script, Boolean.class);
            List<String> keys = Arrays.asList(key);
            Boolean result = (Boolean)redisTemplate.execute(redisScript, keys, value);
            log.info("释放锁的结果" + result);
        }
    }
    log.info("方法执行完成");
    return "方法执行完成";
}
```

## 2.6、分布式锁解决定时任务重复问题

### 1、原理

> 定时任务执行之前先获取分布式锁,获得了分布式锁的任务才会执行定时任务

### 2、Redis分布式锁封装

```java
package com.example.distributelock.lock;

import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.connection.RedisStringCommands;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.RedisScript;
import org.springframework.data.redis.core.types.Expiration;

import java.util.Arrays;
import java.util.List;
import java.util.UUID;

@Slf4j
public class RedisLock implements AutoCloseable {

    private RedisTemplate redisTemplate;
    private String key;
    private String value;
    //单位：秒
    private int expireTime;

    public RedisLock(RedisTemplate redisTemplate,String key,int expireTime){
        this.redisTemplate = redisTemplate;
        this.key = key;
        this.expireTime=expireTime;
        this.value = UUID.randomUUID().toString();
    }

    /**
     * 获取分布式锁
     * @return
     */
    public boolean getLock(){
        RedisCallback<Boolean> redisCallback = connection -> {
            //设置NX
            RedisStringCommands.SetOption setOption = RedisStringCommands.SetOption.ifAbsent();
            //设置过期时间
            Expiration expiration = Expiration.seconds(expireTime);
            //序列化key
            byte[] redisKey = redisTemplate.getKeySerializer().serialize(key);
            //序列化value
            byte[] redisValue = redisTemplate.getValueSerializer().serialize(value);
            //执行setnx操作
            Boolean result = connection.set(redisKey, redisValue, expiration, setOption);
            return result;
        };

        //获取分布式锁
        Boolean lock = (Boolean)redisTemplate.execute(redisCallback);
        return lock;
    }

    public boolean unLock() {
        String script = "if redis.call(\"get\",KEYS[1]) == ARGV[1] then\n" +
                "    return redis.call(\"del\",KEYS[1])\n" +
                "else\n" +
                "    return 0\n" +
                "end";
        RedisScript<Boolean> redisScript = RedisScript.of(script,Boolean.class);
        List<String> keys = Arrays.asList(key);

        Boolean result = (Boolean)redisTemplate.execute(redisScript, keys, value);
        log.info("释放锁的结果："+result);
        return result;
    }


    @Override
    public void close() throws Exception {
        unLock();
    }
}
```



# 二、Zookeeper分布式锁

## 1、zookeeper基本概念

> 基于 Zookeeper 的瞬时节点实现分布式锁



==数据结构==

> zookeeper数据结构
>
> 一棵树 根节点 子节点

![image-20211219171825749](Zookeeper.assets/image-20211219171825749.png)









## 2、Zookeeper环境搭建



后台端口改为了 63010



### 1.1、下载zookeeper软件包

https://downloads.apache.org/zookeeper/zookeeper-3.6.3/

- 这⾥使⽤的是 3.6.3 版，下载的是 apache-zookeeper-3.6.3-bin.tar.gz 压缩包，并将其放在 了 /root/workspace/software ⽬录下

### 1.2、解压并安装

1.  /usr/local/ 下创建 zookeeper ⽂件夹并进⼊

```shell
[root@centos_7_100 ~]# cd /usr/local
[root@centos_7_100 local]# mkdir zookeeper
[root@centos_7_100 local]# cd zookeeper

```

2. 将 ZooKeeper 安装包解压到 /usr/local/zookeeper 中即可

```shell
[root@centos_7_100 zookeeper]# tar -zxvf /root/workspace/software/apache-zookeeper-3.6.3-bin.tar.gz -C ./

```

解压完之后， /usr/local/zookeerper ⽬录中会出现⼀个 apache-zookeeper-3.6.1-bin 的⽬录

### 1.3、创建DATA⽬录

直接在 /usr/local/zookeeper/apache-zookeeper-3.6.1-bin ⽬录中创建⼀个 data ⽬录

```shell
[root@centos_7_100 zookeeper]# ll
总用量 0
drwxr-xr-x. 6 root root 133 12月  7 12:51 apache-zookeeper-3.6.3-bin
[root@centos_7_100 zookeeper]# cd apache-zookeeper-3.6.3-bin/
[root@centos_7_100 apache-zookeeper-3.6.3-bin]# ll
总用量 36
drwxr-xr-x. 2 esuser esuser  4096 4月   9 2021 bin
drwxr-xr-x. 2 esuser esuser    77 4月   9 2021 conf
drwxr-xr-x. 5 esuser esuser  4096 4月   9 2021 docs
drwxr-xr-x. 2 root   root    4096 12月  7 12:51 lib
-rw-r--r--. 1 esuser esuser 11358 4月   9 2021 LICENSE.txt
-rw-r--r--. 1 esuser esuser   432 4月   9 2021 NOTICE.txt
-rw-r--r--. 1 esuser esuser  1963 4月   9 2021 README.md
-rw-r--r--. 1 esuser esuser  3166 4月   9 2021 README_packaging.md
[root@centos_7_100 apache-zookeeper-3.6.3-bin]# mkdir data
[root@centos_7_100 apache-zookeeper-3.6.3-bin]# 

```

等下该 data ⽬录地址要配到 ZooKeeper 的配置⽂件中

### 1.4、创建配置⽂件并修改

进⼊到 zookeeper 的 conf ⽬录，复制 zoo_sample.cfg 得到 zoo.cfg ：

```shell
[root@centos_7_100 apache-zookeeper-3.6.3-bin]# cd conf/
[root@centos_7_100 conf]# ll
总用量 12
-rw-r--r--. 1 esuser esuser  535 4月   9 2021 configuration.xsl
-rw-r--r--. 1 esuser esuser 3435 4月   9 2021 log4j.properties
-rw-r--r--. 1 esuser esuser 1148 4月   9 2021 zoo_sample.cfg
[root@centos_7_100 conf]# cp zoo_sample.cfg zoo.cfg
[root@centos_7_100 conf]# ll
总用量 16
-rw-r--r--. 1 esuser esuser  535 4月   9 2021 configuration.xsl
-rw-r--r--. 1 esuser esuser 3435 4月   9 2021 log4j.properties
-rw-r--r--. 1 root   root   1148 12月  7 12:53 zoo.cfg
-rw-r--r--. 1 esuser esuser 1148 4月   9 2021 zoo_sample.cfg
```

修改配置⽂件 zoo.cfg ，将其中的 dataDir 修改为上⾯刚创建的 data ⽬录，其他选项可以按需配置

修改内容为

```java
dataDir=/usr/local/zookeeper/apache-zookeeper-3.6.3-bin/data
```

### 1.5、启动Zookeeper

回退到 /usr/local/zookeeper/apache-zookeeper-3.6.3-bin 目录 

- 执行命令

```shell
[root@centos_7_100 apache-zookeeper-3.6.3-bin]# ./bin/zkServer.sh start
```

启动后可以通过如下命令来检查启动后的状态：

```shell
[root@centos_7_100 apache-zookeeper-3.6.3-bin]# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/apache-zookeeper-3.6.3-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: standalone
```



### 1.6、启动可能会报错

/usr/local/zookeeper/apache-zookeeper-3.6.3-bin/logs目录下的

zookeeper-root-server-centos_7_100.out 可以查看zookeeper启动的详细信息

```java
Problem starting AdminServer on address 0.0.0.0, port 8080 and command URL /commands
```

上面这个报错可能是8080端口被占用了,

 可在`zoo.cfg`中配置`admin.serverPort=8088`修改端口。

### 1.7、配置环境变量

编辑配置⽂件：/etc路径下 的 profile文件

在文件尾部加入部加⼊ ZooKeeper 的 bin 路径配置即可

```shell
export ZOOKEEPER_HOME=/usr/local/zookeeper/apache-zookeeper-3.6.1-bin
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

最后执行 `source /etc/profile` 使环境变量生效即可

### 1.8、设置开机⾃启

⾸先进⼊ /etc/rc.d/init.d ⽬录，创建⼀个名为 zookeeper 的⽂件，并赋予执⾏权限

```shell
[root@centos_7_100 ~]# cd /etc/rc.d/init.d
[root@centos_7_100 init.d]# ll
总用量 56
-rwxr-xr-x. 1 root root   961 12月  3 10:17 fdfs_storaged
-rwxr-xr-x. 1 root root   963 12月  3 10:17 fdfs_trackerd
-rw-r--r--. 1 root root 18281 5月  22 2020 functions
-rwxr-xr-x. 1 root root  4569 5月  22 2020 netconsole
-rwxr-xr-x. 1 root root  7928 5月  22 2020 network
-rw-r--r--. 1 root root  1160 10月  2 2020 README
-rwxr-xr-x. 1 root root  1422 11月 14 20:38 redis_init_script
-rwxr-xr-x. 1 root root   817 11月 13 12:18 tomcat
[root@centos_7_100 init.d]# touch zookeeper
[root@centos_7_100 init.d]# chmod +x zookeeper 
```

接下来编辑 zookeeper ⽂件，并在其中加⼊如下内容：

```shell
#!/bin/bash
#chkconfig:- 20 90
#description:zookeeper
#processname:zookeeper
ZOOKEEPER_HOME=/usr/local/zookeeper/apache-zookeeper-3.6.3-bin
export JAVA_HOME=/usr/local/java/jdk1.8.0_221 # 此处根据你的实际情况更换对
应
case $1 in
 start) su root $ZOOKEEPER_HOME/bin/zkServer.sh start;;
 stop) su root $ZOOKEEPER_HOME/bin/zkServer.sh stop;;
 status) su root $ZOOKEEPER_HOME/bin/zkServer.sh status;;
 restart) su root $ZOOKEEPER_HOME/bin/zkServer.sh restart;;
 *) echo "require start|stop|status|restart" ;;
esac
```

最后加⼊开机启动即可：

```shell
[root@centos_7_100 init.d]# chkconfig --add zookeeper 
[root@centos_7_100 init.d]# chkconfig zookeeper on 
```

### 1.9、zookeeper cli客户端



##  3、Zookeeper分布式锁原理

### 3.1、实现原理

>- 利用Zookeeper的瞬时有序节点的特性
>- 多线程并发创建瞬时节点时,得到有序的序列
>- 序号最小的线程获得锁
>- 其它的线程监听自己序号的前一个序号

### 3.2、Zookeeper的观察器

> 可设置观察器的3个方法：getData(); , getChildren(); , exists();
>
> 观察器只能监控一次,再监控需重新设置

## 4、Maven依赖

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.14</version>
</dependency>
```

## 5、代码实现

```java
package com.example.distributezklock.lock;

import lombok.extern.slf4j.Slf4j;
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.io.IOException;
import java.util.Collections;
import java.util.List;

@Slf4j
/**
* @Author tho
* @Date 2021/12/21 10:54
* @param null
* @Return null
* @Description: Zookeeper 分布式锁
*/
public class ZkLock implements AutoCloseable, Watcher {

    private ZooKeeper zooKeeper;
    private String znode;

    public ZkLock() throws IOException {
        this.zooKeeper = new ZooKeeper("192.168.198.100:2181",
                10000,this);
    }

    public boolean getLock(String businessCode) {
        try {
            //创建业务 根节点
            Stat stat = zooKeeper.exists("/" + businessCode, false);
            // 创建持久节点
            if (stat == null){
                zooKeeper.create("/" + businessCode,businessCode.getBytes(),
                        ZooDefs.Ids.OPEN_ACL_UNSAFE,
                        CreateMode.PERSISTENT);
            }

            //创建瞬时有序节点  /order/order_00000001
            znode = zooKeeper.create("/" + businessCode + "/" + businessCode + "_", businessCode.getBytes(),
                    ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL);

            //获取业务节点下 所有的子节点
            List<String> childrenNodes = zooKeeper.getChildren("/" + businessCode, false);
            //子节点排序
            Collections.sort(childrenNodes);
            //获取序号最小的（第一个）子节点
            String firstNode = childrenNodes.get(0);
            //如果创建的节点是第一个子节点，则获得锁
            if (znode.endsWith(firstNode)){
                return true;
            }
            //不是第一个子节点，则监听前一个节点
            String lastNode = firstNode;
            for (String node:childrenNodes){
                if (znode.endsWith(node)){
                    zooKeeper.exists("/"+businessCode+"/"+lastNode,true);
                    break;
                }else {
                    lastNode = node;
                }
            }
            synchronized (this){
                wait();
            }

            return true;

        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }

    @Override
    public void close() throws Exception {
        zooKeeper.delete(znode,-1);
        zooKeeper.close();
        log.info("我已经释放了锁！");
    }

    @Override
    public void process(WatchedEvent event) {
        if (event.getType() == Event.EventType.NodeDeleted){
            synchronized (this){
                notify();
            }
        }
    }
}
```

## 6、单元测试

```java
@Test
public void testZkLock() throws Exception {
    ZkLock zkLock = new ZkLock();
    boolean lock = zkLock.getLock("order");
    log.info("获得锁的结果："+lock);

    zkLock.close();
}
```

# 三、Curator分布式锁

## 3.1、实现

> - 引入Curator客户端
> - Curator已经实现了分布式锁的方法
> - 直接调用即可

## 3.2、Maven依赖

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>
```

## 3.3、单元测试

```java
@Test
public void testCuratorLock(){
    RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
    CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.198.100:2181", retryPolicy);
    client.start();
    InterProcessMutex lock = new InterProcessMutex(client, "/order");
    try {
        if ( lock.acquire(30, TimeUnit.SECONDS) ) {
            try  {
                log.info("我获得了锁！！！");
            }
            finally  {
                lock.release();
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    client.close();
}
```

## 3.4、结合controller使用

生成Bean

```java
@Bean(initMethod="start",destroyMethod = "close")
public CuratorFramework getCuratorFramework() {
    RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
    CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.198.100:2181", retryPolicy);
    return client;
}
```

controller 类实现

```java
@Autowired
private CuratorFramework client;
@RequestMapping("curatorLock")
    public String curatorLock(){
        log.info("我进入了方法！");
        InterProcessMutex lock = new InterProcessMutex(client, "/order");
        try{
            if (lock.acquire(30, TimeUnit.SECONDS)){
                log.info("我获得了锁！！");
                Thread.sleep(10000);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                log.info("我释放了锁！！");
                lock.release();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        log.info("方法执行完成！");
        return "方法执行完成！";
    }
```

# 四、Redisson分布式锁

## 4.1、使用Redisson

- 通过Java Api方式引入Redisson
- Spring项目引入Redisson
- SpringBoot 项目引入Redisson

## 4.2、Maven依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.11.2</version>
</dependency>
```

## 4.3、单元测试

- Java Api 方式使用Redisson

```java
package com.example.redissonlock;

import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.concurrent.TimeUnit;

@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class RedissonLockApplicationTests {

    @Test
    public void contextLoads() {
    }

    @Test
    public void testRedissonLock() {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://192.168.198.100:6379");
        RedissonClient redisson = Redisson.create(config);

        RLock rLock = redisson.getLock("order");

        try {
            rLock.lock(30, TimeUnit.SECONDS);
            log.info("我获得了锁！！！");
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            log.info("我释放了锁！！");
            rLock.unlock();
        }
    }

}
```

## 4.4、SpringBoot使用Redisson

- maven依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.11.2</version>
</dependency>
```

- application.properties 配置文件

```properties
spring.redis.host=192.168.198.100
```

- controller

```java
package com.example.redissonlock.controller;

import lombok.extern.slf4j.Slf4j;
import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

@RestController
@Slf4j
public class RedissonLockController {
    @Autowired
    private RedissonClient redisson;

    @RequestMapping("redissonLock")
    public String redissonLock() {
        RLock rLock = redisson.getLock("order");
        log.info("我进入了方法！！");
        try {
            rLock.lock(30, TimeUnit.SECONDS);
            log.info("我获得了锁！！！");
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            log.info("我释放了锁！！");
            rLock.unlock();
        }
        log.info("方法执行完成！！");
        return "方法执行完成！！";
    }
}
```

## 4.5、Spring使用Redisson

- maven 依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.11.2</version>
</dependency>
```

- application.properties 配置文件

```properties
# spring.redis.host=192.168.198.100
```

- redisson.xml 配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:redisson="http://redisson.org/schema/redisson"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://redisson.org/schema/redisson
       http://redisson.org/schema/redisson/redisson.xsd
">

    <redisson:client>
        <redisson:single-server address="redis://192.168.198.100:6379"/>
    </redisson:client>
</beans>
```

- 启动类引入配置文件

```java
package com.example.redissonlock;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ImportResource;

@SpringBootApplication
// 引入xml文件
// @ImportResource("classpath*:redisson.xml")
public class RedissonLockApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedissonLockApplication.class, args);
    }

}
```

## 4.6、controller

```java
package com.example.redissonlock.controller;

import lombok.extern.slf4j.Slf4j;
import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

@RestController
@Slf4j
public class RedissonLockController {
    @Autowired
    private RedissonClient redisson;

    @RequestMapping("redissonLock")
    public String redissonLock() {
        RLock rLock = redisson.getLock("order");
        log.info("我进入了方法！！");
        try {
            rLock.lock(30, TimeUnit.SECONDS);
            log.info("我获得了锁！！！");
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            log.info("我释放了锁！！");
            rLock.unlock();
        }
        log.info("方法执行完成！！");
        return "方法执行完成！！";
    }
}
```

# 五、分布式锁方案选择

## 5.1、各种方案优缺点对比

![image-20211221115756739](Zookeeper.assets/image-20211221115756739.png)

## 5.2、数据库方式

- 建议将分布式锁的数据库和生产的数据库分开,不要给数据库造成太大压力

## 5.3、总结

- 不建议使用自己编写分布式锁
- 推荐 ==Redisson== 和 ==Curator== 实现的分布式锁