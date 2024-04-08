---
title: 生成一个简洁、美观、方便的接口文档
date: 2024-03-16 12:03:29
tags: 
  - OpenAPI
  - Swagger
categories: 技术
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/swagger.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/swagger.png
description: 介绍如何使用Swagger来自动生成具有良好规范的接口文档~
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

`Swagger规范(Swagger Specification)`在2015年左右逐渐演变成了`OpenAPI规范(OpenAPI Specification)`，这一转变进一步推动了API开放行业的发展。OpenAPI是Linux基金会的一个项目，Linux选择OpenAPI作为API的标准描述语言，让OpenAPI(Swagger)规范得到了广泛的接受，成为了事实上的行业标准。

随着`OpenAPI 3.0`版本的发布，OpenAPI规范更加注重规范本身，而不关心具体实现的工具，所以与之对应的`Swagger 3`被视作为基于OpenAPI 3.0规范实现的一个工具集，之前版本的Swagger 2被称为`Swagger 2`或者`OpenAPI 2.0`规范。**如今，OpenAPI已经成为了描述RESTful API的标准规范，而Swagger则更多地被看作是实现这一规范的工具集。**

[OpenAPI 3.0 Specification](https://swagger.io/docs/specification/about)的结构如下：

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

我们可以手写这样一份规范文件，然后再用Swagger工具来生成接口文档，但这显然工作量有点大，查OpenAPI的规范文档也让人头大。事实上，这可以通过在项目代码中使用注解来标记接口，让框架解析并自动生成，下面会介绍。

## SpringBoot集成springdoc-openapi

[springdoc-openapi](https://github.com/springdoc/springdoc-openapi)是一个可以将SpringBoot和SwaggerUI集成到一起的库。SpringBoot 2.x版本可以引入springdoc-openapi v1的版本，如下：

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.8.0</version>
</dependency>
```

引入之后就可以直接访问接口文档了，SwaggerUI生成的接口文档默认地址是：`http://localhost:8080/swagger-ui.html`，生成的OpenAPI Specification可以通过`http://localhost:8080/v3/api-docs`访问。

下面是一些基本配置：

```yaml
springdoc:
  swagger-ui:
    path: /swagger-ui-custom.html # 自定义接口文档的地址
  api-docs:
    path: /v3/api-docs
  # 接口分组，可以通过扫描包路径来分组，也可以通过匹配URL前缀来分组
  group-configs:
    - group: module1
      # 指定要包含的接口路径前缀
      pathsToMatch:
        - /api/**
    - group: module2
      pathsToMatch:
        - /**
      pathsToExclude:
        - /api/**
```

详细的配置可以编写一个配置类，对接口文档的基本信息进行一下配置：

```java
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenAPIConfig {
    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("接口文档的标题")
                        .description("接口文档的介绍")
                        .version("接口文档的版本 V1")
                        .license(new License().name("接口文档的许可协议 License").url("https://zunhuier.top"))
                        .contact(new Contact().name("联系人 zunhuier").email("联系人邮箱")));
    }
}
```

在SwaggerUI中效果如图所示：

![OpenAPI Config](https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/swaggerui-info.png)

最后，就是使用OpenAPI规范的注解，来注释各个Restful接口了，下面介绍这些注解的作用。

## 注解

### @Tag

这个注解作用于类上，也可以标记到方法上，起一个标记的作用。一般加在Controller类上，表示这个Controller下的所有接口是某一类的接口，被归类到同一标记下，可以起到接口分组的作用。

```java
import org.springframework.web.bind.annotation.*;
import io.swagger.v3.oas.annotations.tags.Tag;

@RestController
@Tag(name = "测试接口", description = "这个controller里的都是测试接口")
public class TestController {

}
```

### @Operation

一般加在接口方法上，表示这是一个操作，用来对接口进行详细的描述。后面讲的@Parameters、@ApiResponses注解的作用其实都可以直接用@Operation注解中的属性来说明实现，一个注解就能干很多事。

```java
import org.springframework.web.bind.annotation.*;
import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.Operation;

@RestController
@Tag(name = "测试接口", description = "这个controller里的都是测试接口")
public class TestController {
  
    @Operation(summary = "修改用户信息", description = "上传用户id和用户信息")
    @PutMapping("/api/user/{id}")
    public CommonResponse<User> user(@RequestBody User user, @PathVariable("id") int id) {
        System.out.println(user);
        user.setId(id);
        // 修改用户信息...
        return CommonResponse.result(ResultEnum.SUCCESS, user);
    }

}
```

### @Parameters和@Parameter

这两个注解用于描述接口中的显式参数，显式参数是指`header、query、path、cookie`上的参数，不是请求体中的参数，请求体有专门的注解说明。@Parameters中可以有多个@Parameter。

```java
import org.springframework.web.bind.annotation.*;
import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.Parameters;

@RestController
@Tag(name = "测试接口", description = "这个controller里的都是测试接口")
public class TestController {
  
    @Operation(summary = "修改用户id", description = "上传用户id和用户信息")
    @Parameters({
            @Parameter(name = "id", description = "要修改的用户id", in = ParameterIn.PATH)
    })
    @PutMapping("/api/user/{id}")
    public CommonResponse<User> user(@RequestBody User user, @PathVariable("id") int id) {
        System.out.println(user);
        user.setId(id);
        // 修改用户信息...
        return CommonResponse.result(ResultEnum.SUCCESS, user);
    }

}
```

**springdoc会自动扫描接口方法中的参数，推断参数的数据类型。如果要手动给接口方法中的参数添加描述，那@Parameter注解的`name`属性要设置成跟方法参数名一样。注意，`name`属性始终表示的是HTTP请求中的参数名。**

### @RequestBody

