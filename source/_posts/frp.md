---
title: 利用FRP实现内网穿透
date: 2020-10-11 19:50:36
tags:
    - 内网穿透
    - 反向代理
categories: Go
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/frp.jpeg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/frp.jpeg
description: 这篇文章讲解了如何利用内网穿透的技术，来实现在外网上访问内网中的主机，用到的工具是frp。
---
## 内网穿透

内网穿透，也称`NAT`穿透，进行NAT穿透是为了让处于不同NAT网络(局域网、内网)下的两个主机节点(Peer)之间建立直接或间接的连接，从而可以互相通信。

## 反向代理

反向代理可以简单便捷地实现内网穿透，只需要有一台具有公网IP的主机即可。

这里使用`FRP`这个开源软件来进行反向代理。

### FRP介绍

>frp是一个专注于内网穿透的高性能的反向代理应用，支持`TCP、UDP、HTTP、HTTPS`等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。

`FRP`开源、免费。

### 实现原理

完整的frp服务由客户端(frpc)和服务端(frps)组成，服务端通常部署在具有公网IP的主机上，客户端通常部署在需要穿透的内网服务所在的主机上。

提供内网服务的主机由于没有公网IP，不能被非局域网内的其他主机访问。

用户通过访问服务端的frps，由frp负责根据请求的端口或其他信息将请求路由(转发)到对应的内网主机上，从而实现通信。

+ **简单来说，就是frp实现了数据包的转发。外网上的主机首先将要发送给内网主机的数据包发给frp服务端，服务端再发送给内网主机，即frp在这通信之间充当了反向代理服务器。**

frp客户端可以配置多个代理。

### 官方

frp可以在Gihub上下载：[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases)

![FRP下载](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/frp/frpdownload.png)

注意要在相应系统上部署相应的frp服务，还要注意版本一致。

官方网站：[https://gofrp.org/](https://gofrp.org/)

上面说的那些概念都可以查看官方文档。

### 部署

以我的一个案例来进行演示。

由于我要上一个Java项目课程，要在课堂上做项目，学校机房的电脑没有相应的编程环境，我不想每次都要重新配环境，也不想带自己的电脑去上课^ _ ^

于是想到了远程桌面连接，那问题来了，宿舍电脑的网络和机房电脑的网络不是同一个局域网，互相不能`直接`访问，于是需要进行内网穿透。

部署frp客户端的是我宿舍里的电脑，安装的是macOS系统，是要进行穿透的内网主机。部署frp服务端的是我在阿里云上的一台云服务器，具有公网IP。

#### 部署客户端

下载macOS版本的frp，解压后得到的文件目录：

![FRP文件目录](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/frp/frpdir.png)

下载得到的frp是服务端程序和客户端程序都有的，但我们在客户端只需配置客户端的配置，服务端同理。

配置客户端，即配置`frpc.ini`文件：

    [common]
    server_addr = xx.xx.xx.xx   #frp服务端的公网IP
    server_port = 6666          #frp服务端接收来自frp客户端请求的端口，对应服务端配置的bind_port

    [vnc]                       #代理的别名，可以自己起
    type = tcp                  #请求方式
    local_ip = 127.0.0.1        #要暴露到公网的本机IP，默认就填这个
    local_port = 5900           #要暴露到公网的本机端口
    remote_port = 6000          #指明frp服务端用6000号端口进行代理

    [ssh]
    type = tcp
    local_ip = 127.0.0.1
    local_port = 22
    remote_port = 6001

macOS的远程桌面服务是vnc(屏幕共享)，打开方法可以自行百度，这个服务默认占用的端口是5900，所以我上面的配置作用就是：将127.0.0.1(本机)上5900号端口的服务`映射`到IP为xx.xx.xx.xx的frp服务端主机(阿里云服务器)的6000号端口。这样，在外网上访问frp服务端的6000号端口，就可以访问到内网主机上5900号端口上的服务了。

上面的配置中还有我顺便配的一个ssh代理，可以让外网上的主机远程登录我宿舍里的电脑。ssh服务的默认端口为22。

配置好后，在frp文件夹下运行命令：

``` bash
./frpc -c frpc.ini
```

没报错就说明frp客户端服务部署成功了，然后就可以将命令行窗口放一边让服务运行，记得不要关，关了服务就停止运行了。

#### 部署服务端

配置`frps.ini`文件：

    bind_port = 6666            # frp服务端服务运行占用的端口，用来监听frp客户端的请求

    dashboard_port = 7500       # frp控制面板端口
    dashboard_user = admin      # 控制面板用户名
    dashboard_pwd = admin       # 控制面板密码

+ 在浏览器上访问frp服务端的7500号端口，就可以打开frp的控制面板，查看详细的frp服务信息、代理信息、流量信息等等。

配置好后，在frp文件夹下运行命令：

``` bash
nohup ./frps -c frps.ini &
```

+ `nohup`是linux上的一个不挂断运行服务的命令，加上`&`让服务在后台运行。这样在frp服务端上frp服务可以一直运行，而frp客户端关掉命令行窗口就可以停止frp服务，当然在linux上可以直接杀掉frp服务。

最后，如果用浏览器可以访问到frp服务端的frp控制面板，并看到如下代理状态：

![FRP控制面板](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/frp/frpstat.png)

说明成功完成了内网穿透！

## 总结

实现内网穿透的技术不只端口转发这一种，还有像STUN、TURN、ICE这类复杂的`P2P`协议和技术。