---
title: windows下MySQL常见问题解决
date: 2023-05-03 13:12:02
tags: OAuth2
categories: 技术
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/OAuth2.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/OAuth2.png
description: 介绍了一下OAuth 2.0协议
---
## 简介

`OAuth 2.0`是一种授权框架，一个协议，它可以让第三方应用获得互联网上一个HTTP服务的有限访问权限，从而可以访问受保护的资源。例如授权某个应用获得你微信的头像、用户名等基本信息。

> OAuth协议还有个OAuth 1.0协议，是一个小范围内的社区进行实践的结果，OAuth 2.0是基于1.0的实践经验以及更多实际使用案例和扩展性要求而建立的。两者可以在网络上共存，但2.0不向后兼容1.0，两者有各自的实现。

## 角色

OAuth 2.0定义了四种角色：

1. resource owner：能够授予对受保护资源的访问权限的实体。当资源所有者是一个人时，它被称为最终用户。
2. resource server：