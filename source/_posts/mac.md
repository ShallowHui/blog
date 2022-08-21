---
title: MacOS下用zsh配置环境变量
date: 2020-04-15 15:03:29
tags: macOS
categories: 开发环境配置
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/mac.jpg
description: 网上很多在MacOS上配置环境变量的教程都过时了，这里介绍了在zsh这个Shell上，如何用Vim配置环境变量。
---
## 前言

这篇博客简单记录下如何在MacOS下用zsh这个Shell配置环境变量，因为在mac配置环境变量容易被网上一些比较旧的教程带入坑，比Window要麻烦一些。同时也简单介绍一下Shell和Vim。

## Shell

首先我们简单了解一下，操作系统可以大致分为内核和外壳，Shell就是指外壳，英文意思就是“外壳、贝壳“。Shell就是操作系统内核与用户之间进行交互的”桥梁“，可以把它看成是一种解释命令的软件。比如Window下我们看到的桌面（explrer.exe），就是一层Shell，一种图形化的Shell，cmd（cmd.exe）是一种命令行式的Shell。

那么就有多种Shell，就好像人不止一件衣服。你可以用下面这条命令看看MacOS下有多少个Shell：

``` bash
$ cat /etc/shells
```

我们可以看到：

![mac下的Shell](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/mac/shell.png)

本来，Linux和MacOS都是以bash作为默认的Shell，不过Apple官方宣布，MacOS从`Catalina`这个大版本之后，把zsh作为默认的Shell。

![Catalina](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/mac/catalina.png)

这样一来，网上很多以bash作为默认Shell的配置环境变量的教程就过时了。

## 以zsh配置环境变量

网上很多教程说的mac有多个不同优先级的保存环境变量的文件。我试了下用zsh在.bash_profile文件保存环境变量，根本不能长久保存下来，每次打开zsh，上次保存的环境变量的路径就会消失。

网上有关于这个问题的解释和解决办法，但我也不想继续麻烦下去了，直接用paths文件保存环境变量。paths文件的路径为：`/etc/paths`。这个路径是隐藏起来的，我们可以用`Command+Shift+.`组合键来显示隐藏目录。

paths文件是系统级别的全局文件，对所有用户可见，系统启动时会自动加载。我们需要用sudo权限来编辑它。

### Vim

Vim是一个远古时代的编辑器。

我们用sudo命令编辑paths文件：

``` bash
sudo vim /etc/paths
```

输入电脑密码后：

![vim](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/mac/vim.png)

注意，这时我们要在英文输入法的状态下按一次`i`，这样Vim就会从命令模式转变为输入模式，即insert。

![insert](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/mac/vim_insert.png)

现在我们就可以输入我们要保存的环境变量了，注意一行一条路径。

输入完后，我们按下ESC退出输入模式，然后还是在英文输入法状态下按下`Shift` + `;`键，会发现光标移动到了Shell最后一行的冒号后面，接着我们在后面输入命令：`wq`，意思是退出Vim并保存修改，最后按下Enter键就OK了。如果不能保存，试一下`!wq`强制保存命令。

最后一步用下面的命令使paths文件的修改生效：

``` bash
$ source /etc/paths
```

我们可以用下面的命令查看系统环境变量是否添加成功：

``` bash
$ echo $PATH
```

## 小结

由于Linux和MacOS都是类Unix系统，Vim和Shell都是Linux的基础，这次配置环境的过程算是我开始接触Linux。