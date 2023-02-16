---
title: 搭建远程Jupyter Notebook服务
date: 2020-11-14 16:13:11
tags:
    - Jupyter
    - Nginx
categories: Python
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/jupyter.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/jupyter.png
description: 快来搭建属于你自己的远程编程环境吧~
---
## Jupyter简介

百度百科：

>Jupyter Notebook（此前被称为 IPython notebook）是一个交互式笔记本，支持运行40多种编程语言。
>
>Jupyter Notebook的本质是一个Web应用程序，便于创建和共享文学化程序文档，支持实时代码，数学方程，可视化和markdown。用途包括：数据清理和转换，数值模拟，统计建模，机器学习等等.

想必学习Python的人都了解，或使用过Jupyter Notebook。它让我们简单地在浏览器上就可以获得类似于IDE的编程体验，不得不说这个工具很棒。

既然Jupyter是基于Web应用的，那这篇文章就讲解如何在远程服务器上搭建Jupyter服务，以此获得一个个人专属的远程编程环境^ _ ^

## 远程服务器上安装Jupyter

Jupyter依赖于Python，所以你的服务器上必须要安装有Python，推荐Python3。

然后安装Jupyter，通过Python自带的pip：

``` bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple jupyter
```

Jupyter下载安装好后，试一下jupyter命令，如果终端提示说找不到命令，就将Python程序所在目录下的`bin`目录添加到系统Path中，具体教程自行百度。

+ 如果连Python命令、pip命令都无效，那就......

接着输入命令：

``` bash
jupyter notebook --generate-config
```

这条命令会在`用户主目录/.jupyter/`下生成Jupyter的配置文件`jupyter_notebook_config.py`。

编辑这个配置文件：

``` python
c.NotebookApp.allow_remote_access = True #允许远程访问
c.NotebookApp.ip = '0.0.0.0' #允许任何主机访问

c.NotebookApp.notebook_dir = '自定义路径' #指定Jupyter的工作目录，也就是存放代码文件的位置
```

+ 这些配置选项都是事先写好的，只不过被注释掉了，你可以找到这几条语句后去掉注释再编写，或者你嫌找得麻烦就自己直接写出这几条语句。

保存配置文件，然后再输入命令：

``` bash
jupyter notebook password
```

它会提示你输入新密码并确认，这个密码以后就是你远程登录Jupyter的时候需要输入的密码了。

最后启动Jupyter:

``` bash
nohup jupyter notebook --allow-root &
```

+ 添加--allow-root参数使得linux下的root用户可以启动Jupyter服务。nohup命令自行查阅资料。

记住启动信息里的端口号，默认应该是8888。

## 配置远程访问Jupyter

其实完成上面的步骤，就可以远程访问Jupyter了，通过下面的地址:

    http://zunhuier.club:8888 #当然你的服务器要开放这个端口

+ 我的域名`zunhuier.club`是解析到我的阿里云服务器的，指向我的服务器IP。*（23年换域名了~）*

但浏览器链接的默认端口是80，每次访问Jupyter都要输入端口号有点麻烦。

当然，你自己可以去配置Jupyter使用80端口，但我的博客服务是已经占用了80端口的，所以我得换个方法。

### 通过Nginx实现对访问Jupyter的代理

首先我去阿里云将下面的这个域名解析到服务器：

    jupyter.zunhuier.club ——> 8.129.78.12

然后在服务器上配置Nginx，在Nginx的配置文件中编辑：

``` code
server {
    listen 80; #监听80端口
    server_name jupyter.zunhuier.club; #指定要访问的主机名

    location /{

        #转发到Jupyter的服务端口上
        proxy_pass http://localhost:8888;

        tcp_nodelay on;
        proxy_set_header Host            $host;
        proxy_set_header X-Real-IP       $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        #下面三行让Nginx提供WebSocket服务
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

保存配置文件，重启Nginx服务器。

最后用下面的这个地址就可以远程访问Jupyter了：

    http://jupyter.zunhuier.club

+ 可以试试访问我的链接，我是设置了密码的^ _ ^自己完成上面的配置也一定要设置密码哦，保证安全~