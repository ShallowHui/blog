---
title: 生成一个简洁、美观、方便的接口文档
date: 2024-03-16 12:03:29
tags: 
  - OpenAPI
  - Swagger
  - Knife4j
categories: 技术
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/swagger.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/swagger.png
description: 使用Knife4j来自动生成具有良好规范的接口文档吧~
---
## 前言

日常开发中，接口文档是至关重要的，规范、清晰明了的接口文档更是能让人赏心悦目。虽然说接口文档是比较开放的~~看开发人员心情的~~，在哪写，怎么写都可以，只要项目成员之间看得懂就行了，但还是有必要建立一套规范，来说明接口文档应该如何写。

## 规范

[Swagger](https://swagger.io)是一个用于描述`RESTful API`的规范和工具集合，它以`yaml`或者`json`格式来描述接口的结构，方便机器读取：

```yaml
swagger: "2.0"
info:
  title: Sample API
  description: API description in Markdown.
  version: 1.0.0

host: api.example.com
basePath: /v1
schemes:
  - https

paths:
  /users:
    get:
      summary: Returns a list of users.
      description: Optional extended description in Markdown.
      produces:
        - application/json
      responses:
        200:
          description: OK
```

基于这个结构，Swagger中的一个工具`swagger-ui`就可以生成一个可交互的接口文档，是一个HTML页面，可以在浏览器中访问：

![Swagger](https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/swagger-ui.png)

`Swagger规范(Swagger Specification)`在2015年左右逐渐变成了`OpenAPI规范(OpenAPI Specification)`，这一转变进一步推动了API开放行业的发展。OpenAPI是Linux基金会的一个项目，Linux选择OpenAPI作为API的标准描述语言，让OpenAPI(Swagger)规范得到了广泛的接受，成为了事实上的行业标准。

随着`OpenAPI 3.0`版本的发布，OpenAPI规范更加注重规范本身，而不关心具体实现的工具，所以与之对应的`Swagger 3`被视作为基于OpenAPI 3.0规范实现的一个工具集，之前版本的Swagger 2被称为`Swagger 2`或者`OpenAPI 2.0`规范。**如今，OpenAPI已经成为了描述RESTful API的标准规范，而Swagger则更多地被看作是实现这一规范的工具集。**

最新的`OpenAPI 3.0 Specification`如下：

```yaml
openapi: 3.0.0
info:
  title: Sample API
  description: Optional multiline or single-line description in [CommonMark](http://commonmark.org/help/) or HTML.
  version: 0.1.9

servers:
  - url: http://api.example.com/v1
    description: Optional server description, e.g. Main (production) server
  - url: http://staging-api.example.com
    description: Optional server description, e.g. Internal staging server for testing

paths:
  /users:
    get:
      summary: Returns a list of users.
      description: Optional extended description in CommonMark or HTML.
      responses:
        '200':    # status code
          description: A JSON array of user names
          content:
            application/json:
              schema: 
                type: array
                items: 
                  type: string
```

也是可以用`yaml`或者`json`格式来编写。

我们的确是可以手写这样一份规范文件，然后再用Swagger工具来生成接口文档，但显然工作量有点大，查OpenAPI的规范文档也让人头大。事实上，这可以通过在源码中使用注解来自动生成，下面会介绍。

[Knife4j](https://doc.xiaominfo.com)是一个基于Swagger的开源项目，前身是`swagger-bootstrap-ui`，本来是一个纯前端UI的皮肤项目，就是重写swagger-ui的界面，让其更好看。后面随着项目的发展，面对越来越多的个性化需求，不得不添加Java后端代码，为Swagger添加一些增强功能，更名为`Knife4j`是希望能像一把匕首一样小巧，轻量，并且功能强悍，也是希望把项目做成一个为Swagger接口文档服务的通用性解决方案，不仅仅只是专注于前端Ui。

## SpringBoot集成Knife4j

Knife4j有多个版本，每个版本有对应支持的OpenAPI规范版本和SpringBoot版本，需要进行区分。这里不用OpenAPI 2.0(Swagger 2)规范，而是直接使用OpenAPI 3.0规范，在SpringBoot 2.x版本中引入如下依赖：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-spring-boot-starter</artifactId>
    <version>4.5.0</version>
</dependency>
```

