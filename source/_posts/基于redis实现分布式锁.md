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
    StringRedisTemplate redisTemplate;

    private static final String GOODS = "GOODS";

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