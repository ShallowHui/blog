---
title: Docker入门
date: 2020-11-23 23:54:11
tags: Docker
categories: Go
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/docker.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/docker.png
description: Docker是现在非常流行的容器化技术，利用Docker，可以非常快速方便的配置好开发环境和部署项目~
---
## Docker简介

官网介绍：

>Docker提供了在松散隔离的环境（称为容器）中打包和运行应用程序的功能。隔离和安全性使您可以在给定主机上同时运行多个容器。容器是轻量级的，因为它们不需要管理程序的额外负载，而是直接在主机的内核中运行。这意味着与使用虚拟机相比，在给定的硬件组合上可以运行更多的容器。您甚至可以在实际上是虚拟机的主机中运行Docker容器！

传统的虚拟化技术，最经典的就是虚拟机技术了，虚拟机技术就不用我过多介绍了吧。而Docker作为一种容器化技术，对比虚拟机技术，更加的轻量，高效：

+ 虚拟机技术是通过某种技术手段，虚拟出一套硬件，再安装一个OS，然后在OS上面运行应用，开销较大。

+ Docker是将应用程序和应用所依赖的环境打包到容器中，应用在容器中运行。而每个容器之间，除了共享宿主机的内核外，是相互隔离，互不影响的。

Docker采用C/S架构，下面是它的架构图：

![Docker架构](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/docker/dockerfk.png)

+ Client：Docker客户端，是与Docker进行交互的主要方式，通过命令我们可以和Docker的守护进程进行通信。

+ Images(镜像)：可以说是Docker容器的一个只读模板，镜像规定了容器里面配置什么环境、运行什么程序。Docker基于镜像启动容器。

+ Containers(容器)：容器是镜像的一个可运行实例，在容器里面就可以运行各种应用了。

**镜像通常是基于另一个镜像，并进行一些自定义配置。比如可以基于centOS镜像，这个centOS镜像是打包配置好的一个精简的centOS系统，然后再打包进去一些应用程序，比如Tomcat，基于这些环境就可以构建出一个新的Tomcat镜像，Docker基于这个镜像启动一个容器后，你就可以在这个容器中运行你的Web应用了。**

所以，镜像可以类比于我们安装操作系统时要用到的系统镜像文件，容器可以当作是一个简易版的Linux系统。

另外，Docker还有一个重要的东西，那就是`仓库`。Docker仓库就是存放镜像的地方，你可以自己建，也可以去网上的公共仓库下载镜像，镜像都是别人配置好的，用起来特别方便。下面是一个知名Docker仓库，你可以在上面找到可以运行各种应用的镜像：

[Docker Hub](https://hub.docker.com/search?q=&type=image)

## Docker的安装

Docker支持Windows、macOS、Linux。

在官网上有相应平台的安装教程：

[https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)

注意，Linux平台安装的时候，由于apt或者yum用的软件源是国外的，可能下载安装Docker会失败，建议更改为国内源。

安装好Docker后，还需配置Docker的仓库源，不然下载镜像会很慢，具体教程可以自行百度。

## Docker常用命令

安装好Docker后，启动Docker：

``` bash
systemctl start docker #启动docker
systemctl enable docker #开机自启动docker
```

查看Docker信息:

``` bash
docker version #docker版本
docker info #显示docker的信息，包括镜像、容器数量
```

查看Docker中的镜像、容器：

``` bash
root@zunhuier:~# docker images          #查看docker中所有的镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat              9.0                 e0bd8b34b4ea        5 days ago          649MB
mysql               5.7                 1b12f2e9257b        4 weeks ago         448MB

root@zunhuier:~# docker ps              #查看正在运行中的容器
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
50c393a0c18d        mysql:5.7           "docker-entrypoint.s…"   6 days ago          Up 6 seconds        33060/tcp, 0.0.0.0:3307->3306/tcp   MySQL-2

root@zunhuier:~# docker ps -a           #查看docker中所有的容器
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                               NAMES
55811db0bf39        tomcat:9.0          "catalina.sh run"        17 hours ago        Exited (143) 17 hours ago                                       Tomcat-1
50c393a0c18d        mysql:5.7           "docker-entrypoint.s…"   6 days ago          Up 11 seconds               33060/tcp, 0.0.0.0:3307->3306/tcp   MySQL-2
```

+ `CONTAINER ID`是容器的ID，通过它可以启动、停止、删除容器，当然也可以使用容器别名。

搜索镜像：

``` bash
docker search image

root@zunhuier:~# docker search tomcat
NAME                          DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
tomcat                        Apache Tomcat is an open source implementati…   2883                [OK]                
tomee                         Apache TomEE is an all-Apache Java EE certif…   84                  [OK]                
dordoka/tomcat                Ubuntu 14.04, Oracle JDK 8 and Tomcat 8 base…   55                                      [OK]
bitnami/tomcat                Bitnami Tomcat Docker Image                     36                                      [OK]
kubeguide/tomcat-app          Tomcat image for Chapter 1                      29                                      
consol/tomcat-7.0             Tomcat 7.0.57, 8080, "admin/admin"              17                                      [OK]
cloudesire/tomcat             Tomcat server, 6/7/8                            15                                      [OK]
aallam/tomcat-mysql           Debian, Oracle JDK, Tomcat & MySQL              13                                      [OK]
arm32v7/tomcat                Apache Tomcat is an open source implementati…   10                                      
rightctrl/tomcat              CentOS , Oracle Java, tomcat application ssl…   6                                       [OK]
maluuba/tomcat7-java8         Tomcat7 with java8.                             6                                       
unidata/tomcat-docker         Security-hardened Tomcat Docker container.      5                                       [OK]
```

`OFFICIAL`带有[OK]的是官方出品的镜像。但我们一般不用这个命令搜索镜像，而是到我上面提到的叫Docker Hub的仓库中去搜索镜像。因为Docker Hub上面的镜像比较全，而且可以显示tag，即镜像的版本，以及可以查阅官方提供的Docker版应用文档，了解清楚启动容器的时候可以带什么参数之类的信息。

下载镜像：

``` bash
docker pull image:tag

#docker pull tomcat
```

**镜像名字后面加个冒号和tag，可以指明下载哪个版本的镜像，不加就会默认下载latest，即最新版镜像。**

启动容器：

``` bash
docker run image:tag

#docker run -d -p 8080:8080 --name Tomcat-1 tomcat:9.0
```

`docker run`命令的常用参数有：

+ -d：让容器在后台运行。

+ -it：让容器在前台运行，可以看到容器启动时打印的日志信息。

+ -p 主机端口:容器端口：将容器的端口映射到宿主机的端口上，这样通过访问宿主机的端口就可以访问容器中的服务了：

``` bash
root@zunhuier:~# netstat -ntlp        
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      558/sshd: /usr/sbin 
tcp6       0      0 :::3307                 :::*                    LISTEN      1542/docker-proxy   
tcp6       0      0 :::8080                 :::*                    LISTEN      2178/docker-proxy   
tcp6       0      0 :::22                   :::*                    LISTEN      558/sshd: /usr/sbin
```

上面我启动了两个Docker容器，并用主机端口代理了容器端口。

**运行了Docker的机器上，Docker会创建一个虚拟网桥，名字叫docker0，这个网桥有自己的IP地址作为Docker容器的网关，Docker会为每个容器分配一个相同网段的IP地址以进行容器间的通信（Docker容器的网段跟宿主机的网段不同），docker0相当于将两个不同网段的局域网连接了起来。**

``` bash
root@zunhuier:~# ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:b3ff:fe81:7a7c  prefixlen 64  scopeid 0x20<link>
        ether 02:42:b3:81:7a:7c  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5  bytes 526 (526.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.22.132  netmask 255.255.255.0  broadcast 172.16.22.255
        inet6 fe80::20c:29ff:fe38:ea  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:38:00:ea  txqueuelen 1000  (Ethernet)
        RX packets 1709  bytes 177317 (173.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1274  bytes 224365 (219.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

这涉及到了Docker的四种网络模式，具体可以自行查阅Docker的官方文档，这里不过多介绍。

+ --name 容器名：为容器起个名字，不加Docker就会随机起个名字。

+ **-e：添加一些自定义参数，通常是跟容器中的应用密切相关的，可以到Docker Hub上查阅镜像的官方文档，看看可以添加什么参数。**

进入容器：

``` bash
docker exec -it 容器名/ID bash

docker attach 容器名/ID

root@zunhuier:~# docker exec -it Tomcat-1 bash
root@55811db0bf39:/usr/local/tomcat# ls
BUILDING.txt  CONTRIBUTING.md  LICENSE  NOTICE  README.md  RELEASE-NOTES  RUNNING.txt  bin  conf  lib  logs  native-jni-lib  temp  webapps  webapps.dist  work
root@55811db0bf39:/usr/local/tomcat# cd /
root@55811db0bf39:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

**从上面可以看出，进入容器后，就是进入了一个Linux系统，Tomcat安装在这个系统上。所以说可以把容器当作是一个简易Linux系统。**

退出容器：

``` bash
root@55811db0bf39:/usr/local/tomcat# exit
exit
root@zunhuier:~# 
```

+ 注意如果是通过`exec`命令进入容器的话，退出容器不会使容器停止运行，但`attach`命令会。

查看容器启动日志：

``` bash
docker logs 容器名/ID
```

从宿主机上拷贝文件到容器中：

``` bash
docker cp 主机路径 容器名/ID:容器内路径 #如果想从容器中拷贝文件到主机上，交换路径顺序就行了
```

通过`docker run`命令创建并启动一个容器后，可以通过下面的几条命令操作容器：

``` bash
docker start 容器名/ID    #启动
docker restart 容器名/ID  #重启容器
docker stop 容器名/ID     #停止容器
docker kill 容器名/ID     #强制停止
```

删除镜像，容器：

``` bash
docker rmi -f 镜像ID或镜像名          #删除指定镜像
docker rmi -f $(docker images -aq)  #删除全部镜像

docker rm 容器ID或者容器名	        #删除指定容器
docker rm -f $(docker ps -aq)    #删除所有容器
```

## 挂载容器目录

我们在Docker Hub上面找到了一个Apache官方的Tomcat镜像，然后通过命令把镜像下载下来并启动，接着我们怎么在容器中部署我们的Web应用呢？

当然可以使用`docker cp`命令把Web应用拷贝到容器中，但还有更方便的方式，即让主机上的文件跟容器中的文件同步，主机上的目录挂载到容器目录上，操作主机上的文件就相当于操作容器中的文件。

事实上，整个Docker的镜像、容器之类的对象，都是保存在宿主机的文件上的，Linux下目录为`/var/lib/docker/`。

通过下面的命令可以查看容器的目录挂载情况，由`Mounts`属性记录：

``` bash
docker inspect 容器名/ID

root@zunhuier:~# docker inspect MySQL-2
...
"Mounts": [
    {
        "Type": "volume",
        "Name": "62c02e5d5e119b8727b1a6bc136f82ac0d8e30e16eb9de6426b2fa831f941d40",
        "Source": "/var/lib/docker/volumes/62c02e5d5e119b8727b1a6bc136f82ac0d8e30e16eb9de6426b2fa831f941d40/_data",
        "Destination": "/var/lib/mysql",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
...
```
+ type是指挂载类型，`bind`是用户自己指定主机上的目录挂载到容器目录中，`volume`则是将一个Volume(卷，可以自己通过docker命令创建，或者docker自己创建)挂载到容器目录中，这个Volume也是主机上的文件，不过是由docker自身进行管理，保存在主机目录`/var/lib/docker/volumes/`下。

+ Source是宿主机上的文件，Destination是容器中的文件。

在启动容器的时候，我们可以自行挂载容器目录：

``` bash
docker run -d --name Tomcat-1 -p 3306:3306 -v /home/zunhuier/tomcat_webapps:/usr/local/tomcat/webapps tomcat:9.0

# 添加 -v 主机路径:容器路径 参数即可
```

这样我们就可以直接把Web应用放到主机上相应的目录下，就相当于部署到容器中了。还可以同步Tomcat的配置文件，这样就不用进入到容器中修改配置了，Docker容器作为一个简易Linux系统自然没装Vim，还要自己装。。

## 自定义镜像

Docker提供了一个简便的方式来构建新的镜像，即编写Dockerfile文件。

下面我将演示如何将一个SpringBoot应用打包成一个Docker镜像，让其在容器中运行。

创建`Dockerfile`文件，编写如下信息：

``` dockerfile
FROM java:8
MAINTAINER zunhuier<1031762684@qq.com>
COPY springboot-mybatis-1.0.jar /usr/local/springboot.jar
ENV MYPATH /usr/local
WORKDIR $MYPATH
ENTRYPOINT ["java","-jar","/usr/local/springboot.jar"]
EXPOSE 8080
```

Dockerfile可以编写的指令有：

    FROM          #从一个基本的镜像开始构建
    MAINTAINER    #镜像构建者信息
    RUN           #构建镜像时要运行的命令
    ADD           #添加文件到镜像中
    WORKDIR       #设置镜像工作目录，即进入容器时的落脚点
    VOLUME        #挂载VOLUME到容器目录
    EXPOSE        #指定容器运行时对外开放的端口
    CMD           #容器运行时要运行的命令，只有最后一个生效
    ENTRYPOINT    #容器运行时要运行的命令，可以追加命令
    ONBUILD       #构建一个被继承DockerFile时会被触发
    COPY          #将文件拷贝到镜像中
    ENV           #设置容器中的环境变量

将已经打成jar包的SpringBoot项目放在Dockerfile文件的同目录下，然后在当前目录下执行`docker build -t 起个镜像名 .`命令开始构建镜像：

``` bash
root@zunhuier:/home/zunhuier/docker-build# ls
Dockerfile  springboot-mybatis-1.0.jar

root@zunhuier:/home/zunhuier/docker-build# docker build -t springboot .
Sending build context to Docker daemon  24.21MB
Step 1/7 : FROM java:8
8: Pulling from library/java
5040bd298390: Pull complete 
fce5728aad85: Pull complete 
76610ec20bf5: Pull complete 
60170fec2151: Pull complete 
e98f73de8f0d: Pull complete 
11f7af24ed9c: Pull complete 
49e2d6393f32: Pull complete 
bb9cdec9c7f3: Pull complete 
Digest: sha256:c1ff613e8ba25833d2e1940da0940c3824f03f802c449f3d1815a66b7f8c0e9d
Status: Downloaded newer image for java:8
 ---> d23bdf5b1b1b
Step 2/7 : MAINTAINER zunhuier<1031762684@qq.com>
 ---> Running in 395de34e3527
Removing intermediate container 395de34e3527
 ---> f11e4f66867e
Step 3/7 : COPY springboot-mybatis-1.0.jar /usr/local/springboot.jar
 ---> 93f368c3f9f3
Step 4/7 : ENV MYPATH /usr/local
 ---> Running in c8643079be39
Removing intermediate container c8643079be39
 ---> 35505863be9c
Step 5/7 : WORKDIR $MYPATH
 ---> Running in f1d8a400d460
Removing intermediate container f1d8a400d460
 ---> d06a34032e96
Step 6/7 : ENTRYPOINT ["java","-jar","/usr/local/springboot.jar"]
 ---> Running in 76256d2dedf7
Removing intermediate container 76256d2dedf7
 ---> 410e6b45ff53
Step 7/7 : EXPOSE 8080
 ---> Running in 1e6116caf9a8
Removing intermediate container 1e6116caf9a8
 ---> 2f94ca039005
Successfully built 2f94ca039005
Successfully tagged springboot:latest

root@zunhuier:/home/zunhuier/docker-build# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
springboot          latest              2f94ca039005        10 minutes ago      667MB
tomcat              9.0                 e0bd8b34b4ea        5 days ago          649MB
mysql               5.7                 1b12f2e9257b        4 weeks ago         448MB
java                8                   d23bdf5b1b1b        3 years ago         643MB
```

默认tag为latest，然后就可以启动容器，运行SpringBoot应用了~

## 总结

Docker真的太方便了，就是命令有点多......