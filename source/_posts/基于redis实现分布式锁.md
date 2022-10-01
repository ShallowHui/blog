---
title: 基于redis实现分布式锁
date: 2022-8-29 16:29:11
tags: Redis
categories: C/C++
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/redis.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/redis.png
description: 简单介绍一下如何基于redis的setnx命令实现一个分布式锁。
---
## 前言

在多线程或者多进程的情况下，对于共享资源的访问进行控制是非常有必要的，加锁（互斥量）是一种对并发程序进行同步控制的有效方式。

## 单机锁和分布式锁

在日常业务开发中，经常会出现多个请求同时对同一共享资源进行访问的场景，比如下面的购买商品的场景：

```java
@RestController
public class GoodsController {

    @Autowired
    StringRedisTemplate redisTemplate; // 使用 spring-boot-starter-data-redis

    private static final String GOODS = "GOODS"; // 商品

    @GetMapping("/buygoods")
    public String buyGoods() {
        try {
            String s = redisTemplate.opsForValue().get(GOODS);
            int n = s == null ? 0 : Integer.parseInt(s);
            if (n > 0) {
                redisTemplate.opsForValue().set(GOODS, String.valueOf(n-1));
                return "成功购买到商品！";
            } else {
                return "购买商品失败！";
            }
        } finally {
            // ...
        }
    }

}
```

这里简单起见，没有写什么Dao、Service层之类的，直接在Controller处理业务。业务比较简单，就是从redis中取出商品的剩余数量，如果大于0还有剩，就返回购买成功的结果并将redis中的商品数量减一。考虑到程序的健壮性，通过try-catch块捕获异常，而且后面需要在finally块中释放锁。

商品是共享资源，显然如果不对其加以限制，那么在多个请求的多个线程同时进行访问时，很容易就会出现超买超卖的现象。在传统的单机环境下，即这个服务只部署一个实例，对于多线程的同步控制，我们一般可以通过`synchronized`或者`ReentrantLock`进行加锁。

通过synchronized关键字：

```java
@RestController
public class GoodsController {

    @Autowired
    StringRedisTemplate redisTemplate;

    private static final String GOODS = "GOODS";

    @GetMapping("/buygoods")
    public String buyGoods() {
        synchronized(this) {
            try {
                String s = redisTemplate.opsForValue().get(GOODS);
                int n = s == null ? 0 : Integer.parseInt(s);
                if (n > 0) {
                    redisTemplate.opsForValue().set(GOODS, String.valueOf(n-1));
                    return "成功购买到商品！";
                } else {
                    return "购买商品失败！";
                }
            } finally {
                // ...
            }
        }
    }

}
```

使用ReentrantLock：

```java
@RestController
public class GoodsController {

    @Autowired
    StringRedisTemplate redisTemplate;

    private static final String GOODS = "GOODS";

    private final ReentrantLock lock = new ReentrantLock();

    @GetMapping("/buygoods")
    public String buyGoods() {
        lock.lock();
        try {
            String s = redisTemplate.opsForValue().get(GOODS);
            int n = s == null ? 0 : Integer.parseInt(s);
            if (n > 0) {
                redisTemplate.opsForValue().set(GOODS, String.valueOf(n-1));
                return "成功购买到商品！";
            } else {
                return "购买商品失败！";
            }
        } finally {
            lock.unlock(); // 不管业务处理结果如何，最后一定要解锁
        }
    }

}
```

但是，如果是在分布式环境下，上面这两种方法就无效了。在如今微服务架构大行其道的情况下，一个服务可能同时部署多份实例，那么一个JVM实例进程中的锁，显然管不到其它JVM实例、其它进程的运行了。**所以，需要使用更进一步的分布式锁，来保证不同进程对共享资源的互斥访问。**

## 分布式锁的特性

保证一个分布式锁的有效性，至少需要满足以下三个条件：

+ 互斥：在任何时刻，锁只能被一个客户端持有。

redis提供一个`setnx`命令，让用户在redis中不存在这个key时，才可以创建该key。一个用户先创建了key，其它用户就无法再创建同一个key了。这就是redis实现分布式锁的关键，这个key就相当于锁，只能被一个用户持有，可以有效保证锁的互斥使用。

+ 无死锁：需要保证即使当前持有锁的客户端发生崩溃，其它用户还可以继续获得锁，整个系统还能继续运行下去。

正常来说，谁持有锁，那么在完成任务后就要负责解锁。但可能这个客户端在执行任务的时候，机器崩溃了，那之后一直没人去解锁（删除redis中的key）导致其它客户端永远也无法获得锁了，整个系统就陷入了死锁的状态。所以需要给锁设置一个过期时间（redis支持设置key的过期时间），这样即使锁的持有者崩溃了，一定时间后锁过期被redis自动删除，其它客户端就可以继续去获取锁了。但这样会引入一个新的问题，这在下文会详细解释。

+ 高可用。

显然，不仅客户端可能发生崩溃，redis自己也可能发生崩溃，只使用一个redis实例支撑一个分布式系统的风险较大，所以需要使用redis集群来保证锁的高可用性，只要集群中的大多数redis节点存活，客户端就能加锁解锁。

## 原子性加锁、解锁

**redis自身保证命令执行的原子性，所以我们只需在编写操作redis的代码时保证操作的原子性即可：**

```java
@RestController
public class GoodsController {

    @Autowired
    StringRedisTemplate redisTemplate; // 使用 spring-boot-starter-data-redis

    private static final String GOODS = "GOODS";

    private static final String LOCK_KEY = "LOCK"; // 分布式锁在redis中的key

    @GetMapping("/buygoods")
    public String buyGoods() {
        String value = UUID.randomUUID().toString(); // 生成随机值，作为key的value
        Boolean isLock = redisTemplate.opsForValue().setIfAbsent(LOCK_KEY, value); // 加锁
        if (!isLock) {
            return "抢锁失败！";
        }
        try {
            String s = redisTemplate.opsForValue().get(GOODS);
            int n = s == null ? 0 : Integer.parseInt(s);
            if (n > 0) {
                redisTemplate.opsForValue().set(GOODS, String.valueOf(n-1));
                return "成功购买到商品！";
            } else {
                return "购买商品失败！";
            }
        } finally {
            redisTemplate.delete(LOCK_KEY); // 解锁
        }
    }

}
```

这里抢锁失败就简单地直接返回了，实际业务中可能要不断地去尝试加锁。

## 设置锁过期时间

`redisTemplate`提供这样的设置过期时间的方式：

```java
// test
```