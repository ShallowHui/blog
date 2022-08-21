---
title: Spring入门
date: 2020-06-21 13:29:29
tags: Spring
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/spring.png
description: Spirng一统Java后端开发领域，是每一个Java后端开发程序员必学的东西。这里大致介绍了下Spring框架的体系结构，顺便提及了如何去下载Spring，在项目中引入Spring框架。
---
## Spring简介

### 什么是Spring框架

Spring是由Rod Johnson组织和开发的一个用在Java SE/EE开发中的轻量级开源框架，Spring以IoC(Inversion of Control，控制反转)和AOP(Aspect Oriented Programming，面向切面编程)的特性为核心，取代了以前EJB臃肿、低效的开发模式。

Spring具有简单、可测试和松耦合的特点，不仅适用于Web服务器端开发，也可以应用于任何Java应用的开发，并且可以方便地集成其它各种优秀框架。使用Spring，可以将所有对象(JavaBean)的创建和依赖关系的维护工作都交给Spring容器管理，大大地降低了组件之间的耦合性。

### Spring的体系结构

Spring框架采用的是分层架构，它的一系列功能被分为20多个模块，大体上分为以下几大类：

1. **Core Container(核心容器)：**
    Spring的核心容器是Spring其它模块运行工作的基础，核心容器主要由Beans模块、Core模块、Context模块和Expression(SpEL)模块组成。其中Core是核心模块，它实现了Spring的IoC和DI核心功能。

2. **Data Access/Integration(数据访问/集成)：**
    数据访问/集成层包括JDBC、ORM、OXM、JMS等模块。主要功能是提供了JDBC的抽象层，并可以良好地与一些数据持久化开源框架，如Hibernate、Mybatis等进行集成，大大地减少了在开发过程中对数据库进行操作的代码。

3. **Web：**
    Web层包括WebSocket、Servlet、Web和Portlet模块。对于Web服务器端开发而言，主要提供了对MVC模型的实现。

4. **其它模块：**
    Spring的其它模块还有AOP、Aspects等模块。可以提供面向切面编程的实现，以及一些其它功能。

![Spring框架的体系结构](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/spring/spring_overview.png)

## Spring的下载和使用

### 下载

Spring的第一个版本是在2004年发布的，经过10多年的发展，截止我写这篇博客的时候，Spring的最新发行版本已经是5.2.7了。

下载可以到[Spring版本仓库](https://repo.spring.io/webapp/#/artifacts/browse/simple/General/libs-release-local/org/springframework/spring/5.2.7.RELEASE)下载。

由于Spring在Github上开源，所以可以在Github上下载Spring的源码，仓库项目名为：`spring-framework`。

### 使用

将下载下来的Spring框架压缩包解压后打开，在lib目录下可以找到Spring每个模块对应的jar包，以及相应的源代码jar包和API文档jar包，也就是说每个模块都有对应的三个jar包，但我们在使用时，只需要在项目中导入以`RELEASE.jar`结尾的jar包，这是Spring框架编译好后的class文件jar包。

在根目录下还有一个schema文件夹，里面包含了开发所需的schema文件，这些文件定义了Sping相关XML配置文件的约束。

在众多jar包中，有四个Spring的基础包，分别对应Spring核心容器的四个模块：

+ spring-core-版本号.RELEASE.jar

+ spring-beans-版本号.RELEASE.jar

+ spring-context-版本号.RELEASE.jar

+ spring-expression-版本号.RELEASE.jar

实际使用Spring开发时，除了要用到Spring自带的jar包，还需要导入一个Spring的依赖包，`commons-logging.jar`。这个依赖包可以通过下面的网址进行下载：

[http://commons.apache.org/proper/commons-logging/download_logging.cgi](http://commons.apache.org/proper/commons-logging/download_logging.cgi)

初学者学习Spring框架的时候，只需将上面四个基础包和依赖包导入到项目中，就可以使用Spring的基本功能了。


