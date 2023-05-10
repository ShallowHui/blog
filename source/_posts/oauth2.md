---
title: OAuth 2.0协议简介
date: 2023-05-03 13:12:02
tags: OAuth2
categories: 技术
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/OAuth2.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/OAuth2.png
description: 简单介绍了一下OAuth 2.0协议。
---
## 简介

`OAuth 2.0`是一种授权框架，一个协议，它可以让第三方应用获得互联网上一个HTTP服务的有限访问权限，从而可以访问受保护的资源。例如授权某个应用获得你微信的头像、用户名等基本信息。

> OAuth协议还有个OAuth 1.0协议，是一个小范围内的社区进行实践的结果，OAuth 2.0是基于1.0的实践经验以及更多实际使用案例和扩展性要求而建立的。两者可以在网络上共存，但2.0不向后兼容1.0，两者有各自的实现。

关于OAuth 2.0协议的详细文档，可以参考[RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)。

## 角色

OAuth 2.0定义了四种角色：

1. resource owner：能够授予对受保护资源的访问权限的实体。当资源所有者是一个人时，它被称为最终用户。
2. resource server：托管受保护资源的服务器，能够响应携带访问令牌的受保护资源请求。
3. client：代表资源所有者并经其授权进行受保护资源请求的应用程序。
4. authorization server：在成功验证资源所有者的身份并获得授权后向客户端发放访问令牌的服务器。

类似上面的例子，用户即是资源所有者，第三方应用即是客户端，微信则是即充当了授权服务器又充当了资源服务器。

除此之外，还有个重要的组成`user agent`，用户是通过它与客户端进行交互的，一般来说，user agent指的就是浏览器、App。

## Authorization Grant

**OAuth 2.0协议的授权许可(Authorization Grant)，指的是一种凭证，代表着资源所有者对客户端的授权，被客户端用于获取访问令牌。**

OAuth 2.0协议定义了四种授权许可的类型：

1. Authorization Code Grant：授权码。
2. Implicit：隐式授权。
3. Resource Owner Password Credentials：资源所有者密码凭证。
4. Client Credentials：客户端凭证。

由于后面三种授权类型使用得不是很广泛，而且第一种授权码类型使用得最广泛，也是最安全的，所以，下面只重点介绍一下授权码类型的授权流程。

## 授权码许可

完整的流程图：

![Authorization Code Grant](https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/authorizationcode.png)

简单来说，就两大步：

1. 获取授权码 (code)。
2. 根据授权码去获取访问令牌 (access_token)。

### 客户端注册

首先，客户端需要到授权服务器上面进行注册，一般都是登录授权服务器的网站，然后填写表单信息进行注册的。

客户端需要告诉授权服务器所需的各种信息，比如应用类型、应用名、应用的回调地址、应用logo等等，这些都需要授权服务器去进行审核以此判断该应用是否正规，就比如微信也不可能随便就允许任意一个第三方应用来请求获得自身微信用户的信息。

最后，客户端会获得授权服务器给予的一个应用id和应用密钥，这是用来对客户端进行认证的，客户端需要将这两个数据保存好，不能泄露。

### 授权请求

**当用户希望在第三方应用上，使用自己其它平台的信息时，比如想要在第三方应用上使用微信进行登录，就需要由第三方应用也就是客户端，引导用户向授权服务器发起一个授权请求，也就是去访问授权服务器。**

先是由客户端根据授权服务器的授权端点构造好一个请求地址，这个地址以`application/x-www-form-urlencoded`的方式携带以下参数：

| | 参数名 | 描述 |
| :------: | :------: | :------: |
| 必须 | response_type | 响应类型，这里必须为`code` |
| 必须 | client_id | 上面注册时获取到的客户端id |
| 可选 | redirect_uri | 回调地址，如果为空，则使用注册时填的回调地址 |
| 可选 | scope | 访问请求的范围 |
| 可选 | state | 客户端用来维护状态的一个随机值，用于防止CSRF攻击 |

再让用户通过`GET`请求去访问这个地址。

请求示例：

```text
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcode HTTP/1.1

Host: authorize_server.example.com
```

**值得注意的是，这里介绍的都是OAUth 2.0的规范步骤，至于具体的实现细节需要结合实际情况自己考虑。**

比如，如何让用户发起授权请求？具体对于一个第三方Web应用来说，要想使用微信进行登录，就要让浏览器界面跳转到微信自身的授权网站，这个网站上会显示一个二维码，用户使用手机微信扫码后就授权登录了。那一般的，可以这样实现：用户点击一个第三方登录链接，浏览器就向应用的后端接口发送一个请求，后端返回一个重定向响应，重定向的目标地址就是上面构造好的地址，这样用户的浏览器就会跳转到授权服务器自身的授权页面了。

**总之，客户端通过重定向或者其它方式，让用户代理跳转到授权服务器的授权端点。**一般的，授权服务器会先对用户进行一个认证，认证成功后，再显示是否允许对客户端的授权，以及确认授予客户端哪些权限，这个权限范围对应的就是`scope`参数。用户同意之后，授权服务器也会返回给用户代理一个重定向响应，目标地址就是`redirect_url`参数指定的地址，一般是跳转回客户端。

### 授权响应

授权服务器返回的重定向响应中，会在目标地址后面添加几个特定的参数，响应示例如下：

```text
HTTP/1.1 302 Found

Location: http://client.example.com/code?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz
```

**其中`code`参数表示授权码，就是授权服务器返回给客户端的授权许可，客户端需要根据这个授权码，再去授权服务器获取访问令牌。**

#### 为什么不直接返回访问令牌

