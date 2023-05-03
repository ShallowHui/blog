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

根据上面的例子，用户即是资源所有者，第三方应用即是客户端，微信则是即充当了授权服务器又充当了资源服务器。

除此之外，还有个重要的组成`user agent`，用户是通过它与客户端进行交互的，一般来说，user agent指的就是浏览器、App。

## Authorization Grant

**OAuth 2.0协议的授权许可(Authorization Grant)，指的是一种凭证，代表着资源所有者对客户端的授权，被客户端用于获取访问令牌。**

OAuth 2.0协议定义了四种授权类型：

1. Authorization Code Grant：授权码。
2. Implicit：隐式授权。
3. Resource Owner Password Credentials：资源所有者密码凭证。
4. Client Credentials：客户端凭证。

由于后面三种授权类型使用得不是很广泛，而且第一种授权码类型使用得最广泛，也是最安全的，所以，下面重点只介绍一下授权码类型的授权流程。

## 授权码许可

这是一张流程图：

![Authorization Code Grant](https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/authorizationcode.png)

### 客户端注册

首先，客户端需要到授权服务器上面进行注册，一般都是登录授权服务器的网站，然后填写表单信息进行注册的。

客户端需要告诉授权服务器所需的各种信息，比如应用类型、应用名、应用的回调地址、应用logo等等，这些都需要授权服务器去进行审核以此判断该应用是否正规，就好像微信也不可能随便就允许任意一个第三方应用来请求获得自身微信用户的信息。

最后，客户端会获得授权服务器给予的一个应用id和应用密钥，这是用来对客户端进行认证的，客户端需要将这两个数据保存好，不能泄露。
