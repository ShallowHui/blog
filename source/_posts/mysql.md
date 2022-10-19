---
title: windows下MySQL常见问题解决
date: 2020-03-28 20:12:02
tags: windowss
categories: MySQL
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/mysql.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/mysql.jpg
description: 这里记录了在windows系统上日常使用MySQL遇到的问题。
---
*我们在日常开发中，使用MySQL总会遇到各种各样的问题，这篇博客就记录总结一下windows系统下MySQL常见的问题和解决方法。*

## Community Server安装常见问题

### 下载

[官网](https://dev.mysql.com/downloads/)下载社区版服务，也可以去[清华镜像站](https://mirrors.tuna.tsinghua.edu.cn/mysql/)下载。安装方式有两种：MSI安装和ZIP解压缩到指定目录。

### 配置

>**MySQL Community Server是MySQL的开源版本，其就是我们常用的数据库版本，已经是一个完整的数据库服务器了。MySQL Installer提供了Server及其它组件的安装，是MySQL集成的一个安装工具。推荐下载Installer，这样就不用进行下面的配置，因为在安装程序安装的时候都搞定了。这里就是要解决仅安装Server时遇到的一些问题。**

网上的资料说MySQL Server 5.7之后的版本都默认安装目录下不存在my.ini文件和data文件夹。那么需要我们自己创建。

![MySQL安装目录](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/mysql/mysqlcata.png)

**首先要把MySQL安装目录下的bin目录添加到系统环境变量中，因为后面会在cmd中用到MySQL自带的命令。**

关于my.ini文件，是MySQL的一个配置文件，无非就是配置一些参数，如端口号，最大连接数，创建数据库时的编码格式等等。这些MySQL都是有默认配置的，也可以通过命令在MySQL中自定义配置，my.ini的作用就是将命令配置持久保存下来，所以这个文件有没有都影响不大。如果你需要自己配置MySQL参数并保存，可以在安装目录下新建一个my.ini文件，填入以下参数，注意目录路径要填自己实际的路径：

    [mysql]
    # 设置mysql客户端默认字符集
    default-character-set=utf8
    [mysqld]
    #设置3306端口
    port=3306
    # 设置mysql的安装目录
    basedir=D:\Program Files\MySQL\
    # 设置mysql数据库的数据的存放目录
    datadir=D:\Program Files\MySQL\data
    # 允许最大连接数
    max_connections=200
    # 服务端使用的字符集默认为8比特编码的latin1字符集
    character-set-server=utf8
    # 创建新表时将使用的默认存储引擎
    default-storage-engine=INNODB

+ 用ANSI编码格式（记事本默认格式）保存为my.ini，存放在MySQL安装目录下。

重点是data文件夹的创建，一定不能手动创建，要通过MySQL的命令创建，有以下两种方式：

**第一种方式会为root账号随机生成一个密码：**

``` bash
> mysqld --initialize
```
等待一会，没有提示即为创建成功data文件夹，随机密码在data文件夹下的xxx.err文件中，打开这个文件找到：

    2020-03-28T10:01:52.840136Z 1 [Note] A temporary password is generated for root@localhost: i?nEs.svL4QL

其中`i?nEs.svL4QL`就是随机生成的密码。

**第二种方式不会为root账号生成随机密码，默认为空：**

``` bash
> mysqld --initialize-insecure
```

创建好data文件夹后，就添加MySQL服务到服务进程中：

``` bash
> mysqld -install
> # mysqld -remove # 移除服务
```

启动服务：

``` bash
> net start mysql #需要以管理员的身份打开cmd
> # net stop mysql # 关闭服务
```

+ MySQL的服务名（mysql）可能不一样，要到你自己的电脑服务进程中查看

**MySQL服务启动成功后，我们就可以登录MySQL，使用数据库了^_^**

### 修改密码

以root账号登录MySQL后，可以通过下面的命令修改密码：

``` bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '重设密码';
```

## 编码问题

我们在使用数据库驱动连接到数据库，进行SQL语句查询时，经常会遇到一些编码格式的问题，比如用insert语句插入一条带有中文的记录，在数据库中中文会显示为`???`，明显编码格式错误。

首先要确保数据库是utf-8编码格式的，可以用下面这条SQL语句创建数据库：

``` bash
mysql> CREATE DATABASE 数据库名 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

然后在程序中修改数据库驱动连接：

    jdbc:mysql://localhost:3306/数据库名?useUnicode=true&characterEncoding=utf8

注意，如果是在xml文件中配置数据库驱动连接的话，这句话要写成：

    jdbc:mysql://localhost:3306/数据库名?useUnicode=true&amp;characterEncoding=utf8

这是xml的语法规定的，`&amp;`是转义字符，相当于&。

## useSSL问题

我们在Web应用中连接到较高版本的MySQL数据库时，可能会遇到以下警告：

    WARN:Establishing SSL connection without server’s identity verification is not recommended. According to MySQL 5.5.45+,5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn’t set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to ‘false’. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.

原因是连接高版本的MySQL时需要指明是否建立SSL连接，实测MySQL Server 5.7.29需要指明。

解决方法是添加连接参数：

    jdbc:mysql://localhost:3306/数据库名?useSSL=false

一般填false，不建议在没有服务器身份验证的情况下建立SSL连接。

## 命令行下备份还原数据库

备份先不用登录mysql，直接在命令行下执行命令，还原才需要先登录mysql。

### 备份

备份使用的是`mysqldump`命令，命令格式如下：

``` bash
> mysqldump [远程服务器地址] -u用户名 -p密码 [-P端口号] 数据库名 [表名] > 备份路径
```

+ 这里[ ]括起来是可以省略的意思。

比如，备份一个本机的名为spring的数据库到D盘下，可以这样写命令：

``` bash
> mysqldump -uroot -p888888 spring > d:\spring.sql
```

### 还原

从上面可以知道，命令行备份数据库保存的是数据库完整的SQL脚本，还原就是要执行这个脚本，使用如下的`source`命令：

``` bash
mysql> source SQL脚本路径
```

+ 如果备份完后就把数据库删了，那么还原之前要先重新创建数据库，并use它，再执行还原命令。

比如，还原spring数据库：

``` bash
mysql> source d:\spring.sql
```

## 未完待续
