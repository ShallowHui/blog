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

在多线程或者多进程的情况下，对于共享资源的访问进行控制是非常有必要的，加锁（互斥量）是一种对并发程序进行同步控制的方式。