---
title: Git的基础命令
date: 2020-03-16 19:50:26
tags: Git基础
categories: Git
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/gitpage.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/gitpage.jpg
description: 这篇文章简单地入门Git这个工具，介绍了版本控制的概念。文章后面用来记录Git的常用命令。
---
## Git——分布式版本控制系统

### 简介

Linus大佬花了两周的时间用C语言写出了Git。Git是套命令行工具，去[官网](https://git-scm.com/)直接下载安装即可(windows下需要配环境变量，mac下不用(git会自动将一个命令替身安装在/usr/bin/目录下))，linux下直接终端输入sudo apt-get install git)。安装好后用命令配置个人用户信息(git配置按优先级从低到高分为：全局配置，适用于全体电脑用户→用户配置，适用于当前电脑用户→项目(特定仓库)配置)：

``` bash
$ git config --global user.name "输入你的名字"
$ git config --global user.email "输入你的邮箱"
```
这个配置起识别用户的作用。

### 推荐

 我这篇博客只是简单总结了一下git的常用简单命令，若想系统地了解git，分支，标签，分布式等等，这里推荐一个[教程](https://www.liaoxuefeng.com/wiki/896043488029600)。

## 简单命令的使用

### 一些基本概念

+ 工作区：指本地仓库(git初始化后的本地文件夹)，你进行操作的地方。
+ 暂存区：修改必须先送到暂存区保存起来。
+ 版本库：从暂存区将修改送到版本库即完成一次提交。
+ 修改：在git中，增、删、改操作都可以叫做修改。
+ 提交：对修改进行提交，完成一次版本记录。

### Git进行版本控制的精髓

**git通过一系列底层数据对象和结构，对文件内容和修改提交进行追踪并引用。git有HEAD引用，指向当前分支，分支指向最新提交，提交指向当时的顶层tree对象。解决合并冲突需要手动修改文件然后再提交。**

上面一段话看不懂没关系，可以去Git的官方文档查看Git的底层原理那一节，讲得非常详细。

![Branch](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/git/git.jpg)

### 初始化一个本地仓库

新建一个文件夹当做本地仓库准备初始化，然后在这个文件夹下进行git命令操作，windows下控制台cd进入文件夹，mac下打开当前文件夹的终端。

``` bash
$ git init #初始化一个本地git仓库，目录下会多出一个隐藏文件夹(.git)，暂存区、版本库、项目配置等都在里面
```
### 建立远程连接

+ 这一步可以不做。

``` bash
$ git remote add 自定义远程仓库名 远程仓库地址 #可以将本地仓库与一个远程仓库关联
```
可以是Gihthub上的一个仓库，也可以是自己搭建的Git服务器。如果要与Github上的仓库连接，事先要进行本机的.ssh_key配置，具体教程百度，很简单，当然要有Github账户。

### 完成一次提交

提交要按顺序不能颠倒命令。

``` bash
$ git add 文件名.扩展名 #将修改送到暂存区，可以一次性add多份文件，注意后面的命令都要加上扩展名
$ git add . #将工作区里的所有修改送到暂存区
```
``` bash
$ git commit -m "提交备注" #将暂存区里的所有修改提交到版本库
```
### 常用命令

``` bash
$ git push 自定义的远程仓库名 本地仓库分支名 #将本地仓库的一个指定分支推送到远程仓库
```
+ 注意：不能推送远程仓库已有的同名分支，git会认为这两个分支冲突(除非远程仓库的那个同名分支最初是由这个本地仓库推送过去的)，这种情况下要先从远程仓库拉取该分支过来进行合并。

``` bash
$ git fetch 自定义的远程仓库名 远程仓库分支名 #将远程仓库的一个指定分支拉取到本地仓库
```
``` bash
$ git merge 指定分支名 #将指定分支合并到当前分支
```
``` bash
$ git pull 自定义的远程仓库名 远程仓库分支名 # = git fetch + git merge
```
``` bash
$ git status #查看当前分支的工作区的状态
```
``` bash
$ git diff #查看工作区与当前分支的版本库的区别
```
``` bash
$ git log #查看当前分支的提交历史
```
``` bash
$ git checkout -- 文件名 #将指定文件回溯到当前分支最近一次add或commit之后的状态，即撤销修改
```
``` bash
$ git reset HEAD 文件名 #将指定文件已经在暂存区的修改撤销掉
```
``` bash
$ git rm 文件名 #删除工作区里的指定文件，并且已经将这个修改给add了
```
``` bash
$ git clone 远程仓库地址 #将远程仓库里的项目克隆到本地
```
``` bash
$ git branch #查看分支情况
```
``` bash
$ git checkout -b 新建分支名 #新建一个分支并切换过去
$ git switch -c 新建分支名 #新版git的创建切换分支的新命令
```
``` bash
$ git checkout 分支名 #切换到指定分支
$ git switch 分支名 #新命令
```
``` bash
$ git branch -d 分支名 #删除指定分支，删除当前分支要先切换到其它分支
$ git branch -D 分支名 #强制删除分支
```
``` bash
$ git tag "标签名称" #给当前分支的最新提交打上一个标签，进行版本控制
```
``` bash
$ git tag #查看当前分支的标签情况
```
``` bash
$ git reset --hard 提交历史id #将当前分支的版本回退到指定的一次提交，即版本回退
```
## 未完待续
