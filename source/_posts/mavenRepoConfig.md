---
title: Maven仓库详细配置
date: 2022-11-26 16:35:53
tags: Maven
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/maven.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/maven.png
description: 介绍如何配置Maven的仓库。
---
## Maven仓库

Maven是一个软件项目的依赖管理和构建工具。

Maven可以根据需要，从一个仓库中下载依赖到本地上。Maven的仓库分为两种类型：

+ 本地仓库：本地仓库是本地机器上的一个目录，默认是`${user.home}/.m2/repository`。Maven下载依赖的时候是先去本地仓库找有没有这个依赖，没有才去互联网上下载。

可以在Maven的全局配置文件`settings.xml`中进行配置：

```xml
<settings>
...
	<localRepository>{custom_path}</localRepository>
...
</settings>
```

+ 远程仓库：Maven在本地仓库上找不到依赖时，就会去远程仓库上下载依赖并保存到本地仓库中。

互联网上有很多Maven仓库是公开给人下载依赖的，比如阿里云就提供了很多仓库的镜像地址和源地址：

![repo](https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/maven_repo.png)

但是这么多远程仓库，Maven怎么知道去哪个仓库下载依赖呢？

在Maven中，有一个“超级POM”([Super POM](https://maven.apache.org/ref/3.8.6/maven-model-builder/super-pom.html))，其它Maven项目默认都是继承这个POM的，其中有这么一个配置：

```xml
<repositories>
	<repository>
		<id>central</id>
		<name>Central Repository</name>
		<url>https://repo.maven.apache.org/maven2</url>
		<layout>default</layout>
		<snapshots>
			<enabled>false</enabled>
		</snapshots>
	</repository>
</repositories>
```

可以看到，Maven给所有项目配置了一个默认的远程仓库，id为`central`，所以这个官方的仓库也被称为`中央仓库`。

Maven默认从中央仓库下载依赖。

## 给中央仓库设置镜像

由于众所周知的原因，从中央仓库下载依赖可能下载得会很慢甚至失败，所以需要为中央仓库设置一个镜像仓库，相当于代理了中央仓库，以后会直接从镜像仓库下载中央仓库的依赖，而不会去中央仓库下载了。

在Maven的全局配置文件`settings.xml`中配置如下阿里云镜像：

```xml
<mirrors>
...
	<mirror>
		<id>aliyunmaven</id>
		<mirrorOf>central</mirrorOf>
		<name>阿里云公共仓库</name>
		<url>https://maven.aliyun.com/repository/public</url>
	</mirror>
...
</mirrors>
```

注意`<mirrorOf>`的值必须跟中央仓库的id一样为`central`，这样才能设置为中央仓库的镜像。

`<mirrorOf>`就是给一个远程仓库设置镜像的关键。当然，`<mirrorOf>`的值也可以设置得跟远程仓库的id不一样，比如设置成`*`，代表匹配任意远程仓库的id，这样一来，所有远程仓库的依赖都会从这个镜像下载。具体参考如下：

[mirror settings](https://maven.apache.org/guides/mini/guide-mirror-settings.html)

**给中央仓库或者其它一个远程仓库，即单个仓库，设置多个镜像是没有用的，Maven只会简单地找到第一个匹配这个远程仓库id的镜像。**

## 添加远程仓库

一个依赖，可能在中央仓库中没有，在其它远程仓库中才有。或者自己搭建了一个Maven仓库，专门用来存储自己项目的制品(`Artifact`)供自己下载使用且不公开。

基于种种原因，需要为Maven添加远程仓库，有两种添加方式：

+ 在项目的`pom.xml`文件中添加，只对当前项目生效：

```xml
<project>
...
	<repositories>
		<repository>
			<id>jcenter</id>
			<name>jcenter仓库</name>
			<url>http://jcenter.bintray.com/</url>
		</repository>
		<repository>
			<id>google</id>
			<name>谷歌仓库</name>
			<url>https://maven.google.com/</url>
		</repository>
	</repositories>
...
</project>
```

+ 在Maven的全局配置文件`settings.xml`中添加，对所有项目生效：

```xml
<profiles>
...
	<profile>
		<id>repo</id>
		<activation>
			<activeByDefault>true</activeByDefault> <!-- 默认激活这个profile -->
		</activation>
		<repositories>
			<repository>
				<id>jcenter</id>
				<name>jcenter仓库</name>
				<url>http://jcenter.bintray.com/</url>
				<!-- 启用发布版本 -->
				<releases>
					<enabled>true</enabled>
				</releases>
				<!-- 启用快照版本，允许下载version带-SNAPSHOT后缀的依赖 -->
				<snapshots>
					<enabled>true</enabled>
					<!-- 更新策略，每次构建项目都会去Nexus下载最新快照版本的依赖 -->
					<updatePolicy>always</updatePolicy>
				</snapshots>
			</repository>
			<repository>
				<id>google</id>
				<name>谷歌仓库</name>
				<url>https://maven.google.com/</url>
			</repository>
		</repositories>
	</profile>
...
</profiles>
```

**当Maven在一个远程仓库或者这个远程仓库的镜像中(如果这个仓库有匹配的镜像的话)找不到某个依赖时，就会去另一个远程仓库中找。如果配置的所有远程仓库都找不到，Maven就会报错。**

## 上传制品到Maven仓库

如果想把我们构建完项目后得到的制品(通常是一个jar包)，共享给其他人，让别人可以在项目中依赖并下载使用，就需要把我们的制品上传到Maven仓库中，有如下两种方式可以配置上传到哪个Maven仓库中：

+ 在项目的`pom.xml`文件中配置，只对当前项目生效：

```xml
<project>
...
	<distributionManagement>
		<repository>
			<id>maven-releases</id>
			<name>正式版本发布仓库</name>
			<url>http://repo_url/repository/releases/</url>
		</repository>
		<snapshotRepository>
			<id>maven-snapshots</id>
			<name>快照版本发布仓库</name>
			<url>http://repo_url/repository/snapshots/</url>
		</snapshotRepository>
	</distributionManagement>
...
</project>
```

+ 在Maven的全局配置文件`settings.xml`中配置，对所有项目生效且覆盖项目自身的配置：

```xml
<profiles>
...
	<profile>
		<id>upload</id>
		<activation>
			<activeByDefault>true</activeByDefault>
		</activation>
		<properties>
			<!-- 正式版本发布仓库 -->
			<altReleaseDeploymentRepository>maven-releases::default::http://repo_url/repository/maven-releases/</altReleaseDeploymentRepository>
			<!-- 快照版本发布仓库 -->
			<altSnapshotDeploymentRepository>maven-snapshots::default::http://repo_url/repository/maven-snapshots</altSnapshotDeploymentRepository>
		</properties>
	</profile>
...
</profiles>
```

>`<altReleaseDeploymentRepository>`这里的写法是：`id::layout::url`。

注意，如果要把制品上传到快照版本发布仓库，那项目的GAV坐标中的`version`必须带有`-SHAPSHOT`后缀，正式版本发布仓库倒是没有什么限制。

另外，如果Maven仓库的管理员对仓库设置了权限保护，那么上传制品的时候就需要验证，可以在Maven的全局配置文件`settings.xml`中添加如下配置：

```xml
<servers>
...
	<server>
		<id>maven-releases</id>
		<username>...</username>
		<password>...</password>
	</server>
	<server>
		<id>maven-snapshots</id>
		<username>...</username>
		<password>...</password>
	</server>
...
</servers>
```

**注意，`<server>`的`id`必须和上面两种配置方法中的仓库id一致，例如，`id::layout::url`中的id必须和`<server>`的id一致。**

## 通用做法

在企业或者大型组织中，关于如何管理Maven仓库，一种比较简单通用的做法是：

1. 以上关于仓库的配置均在`settings.xml`文件中配置，而不是`pom.xml`文件。

2. 集中管理`settings.xml`文件，并通过自动化分发它们。

具体参考：[Guide to Large Scale Centralized Deployments](https://maven.apache.org/guides/mini/guide-large-scale-centralized-deployments.html)