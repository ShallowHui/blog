---
title: Maven基础
date: 2020-09-29 10:35:53
tags: Maven
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/maven.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/maven.png
description: 掌握maven有助于你更快地构建、开发项目。
---
## 项目构建

一般而言，我们开发一个项目时，需要经历的构建步骤有：编码、编译、测试、打包、发布，当然这中间还有一些其它步骤。

但其实在如今各种智能IDE大行其道的背景下，我们对这些构建流程几乎没什么感觉。比如我们开发一个Java Web项目，我们写完代码后点击运行按钮，就可以在浏览器上看到相应的效果，这背后其实是IDE帮我们完成了从编译到把项目发布到服务器的工作，如果你的环境在IDE配置好了的话。

那在以前没有IDE，或者说IDE不够智能的情况下，完成这些工作不难，但一件件去完成就会很繁琐，每个项目都还要各自去配置。

**所以就有了Maven这个专门用于Java项目的管理和构建工具，它提供了一套标准的项目目录结构、一套标准化的项目构建流程，以及最重要的一套标准化的依赖管理机制，这样我们就不用到处去找项目依赖的框架、组件的jar包，还要自己导入到项目中去。**

## Maven

### 安装配置

maven官网下载：[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

下载好后，解压文件放到一个目录下，接着配置maven环境变量：

    M2_HOME=/maven存放目录/maven-版本号
    PATH=$PATH:$M2_HOME/bin

这是linux下的配置方式，windows下配置方式参考jdk的配置方式。

注意，maven是依赖于jdk的，所以要先确保你的系统里有`JAVA_HOME`环境变量。最后在命令行输入`mvn -version`，出现maven相关信息说明配置成功。

### 全局配置

安装好maven后，进入maven的文件夹，打开conf文件夹，里面有一个`settings.xml`文件，它是maven的全局配置文件，打开它并做如下修改：

在`<mirrors>`标签下添加阿里的镜像源，这样maven去官方中央仓库下载依赖的时候能快很多：

``` xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>central</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

## 约定大于配置

一个用maven管理的项目是有其约定俗成的项目目录结构的，不要随意更改其目录结构，直接使用就行。

一个标准的maven项目的目录结构如下：

    MavenProject
    ├── pom.xml
    ├── src
    │   ├── main
    │   │   ├── java
    │   │   └── resources
    │   └── test
    │       ├── java
    │       └── resources
    └── target

+ pom.xml：这个maven项目的管理配置文件

+ src/main/java：存放项目的源代码

+ src/main/resource：存放项目的资源文件，比如项目的配置文件

+ src/test/java：存放项目的测试源代码

+ src/test/resource：存放项目的测试资源文件

+ target：存放项目所有编译、打包后的文件

## 依赖管理

最重要的东西先讲~

### 仓库

平时我们开发要用到框架、项目依赖组件，我们是不是要到各个官网上去寻找、下载相应的jar包？这样不仅费时费力，还不一定在一些下载入口很深的官网上找到——没错，说的就是你，Oracle。

Mvaen就帮我们解决了这个问题，Maven几乎把你能找到的jar包都放到了互联网上的一个中央仓库中。

你只需要在项目的pom.xml文件中进行配置，Maven就可以帮你自动地把项目依赖的jar包从中央仓库下载下来并导入到项目中。

当然，Maven不是每次都到中央仓库中寻找并下载jar包，它会在你的系统上建立一个本地仓库，位置在`/用户目录/.m2`目录下。

这样Maven从中央仓库下载的jar包，会放到本地仓库中进行缓存，当你在下一个项目中要用到同一个依赖时，Maven就会先去本地仓库查找，没有才去中央仓库。

### 坐标

在Maven的仓库中，唯一确定一个项目的是一个三维坐标：

+ groupId：公司、组织的名称，通常为域名的形式

+ artifactId：项目、模块的名称

+ version：项目的版本

在这个网站中，可以查找知名jar包的坐标：[https://mvnrepository.com/](https://mvnrepository.com/)

在pom.xml配置文件中，我们可以定义自己项目的坐标，以及通过坐标引入依赖：

``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- 定义自己的项目的坐标 -->
    <groupId>club.zunhuier</groupId>
    <artifactId>myproject</artifactId>
    <version>1.0</version>

    <!-- 指明项目的打包方式，是打成jar包还是war包 -->
    <packaging>jar</packaging>

    <!-- 引入依赖，比如引入Spring框架所需的jar包 -->
    <dependencies>
	<dependency>
        <groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>5.2.8.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

+ **Maven的依赖管理还有一个非常强的地方，就是它会帮我们自动判断我们导入的依赖是否还依赖于其它的jar包，然后帮我们自动导入。比如上面我们只引入了Spring框架的context模块，那maven就会自动帮我们导入context模块依赖的core模块以及其它模块，非常方便。**

### 依赖关系

在`<dependency>`标签下，我们还可以通过`<scope>`标签指定我们导入的每一个依赖在项目的构建流程中的作用域。

有如下几种依赖关系：

+ **compile**：项目编译时需要用到该jar包，是每一个依赖默认的关系

+ **test**：测试项目时，编译test文件夹下的源码需要用到，典型的如JUnit：

``` xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.3.2</version>
    <scope>test</scope>
</dependency>
```

+ **runtime**：项目编译时不需要，但运行时需要，典型的如JDBC驱动，比如MySQL的驱动：

``` xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.46</version>
    <scope>runtime</scope>
</dependency>
```

+ **provided**：项目编译时需要，但运行时不需要，典型的如servlet的API：

``` xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.0</version>
    <scope>provided</scope>
</dependency>
```

## 构建流程

### 生命周期(Lifecycle)

Maven提供了一套标准化的构建流程，并用流水线上一个个的阶段(phase)来表示一个maven项目的完整生命周期，比如Maven默认的生命周期：

+ validate
+ initialize
+ generate-sources
+ process-sources
+ generate-resources
+ process-resources
+ compile
+ process-classes
+ generate-test-sources
+ process-test-sources
+ generate-test-resources
+ process-test-resources
+ test-compile
+ process-test-classes
+ test
+ prepare-package
+ package
+ pre-integration-test
+ integration-test
+ post-integration-test
+ verify
+ install
+ deploy

有点多哈，其实也就大概对应于这篇文章开头说的，一个项目的大致构建流程，编译(compile)、测试(test)、打包(package)、发布(deploy)。

进入项目文件夹，在命令行下，使用命令`mvn phase`就可以自动化地将项目构建到某个阶段，常用的命令有：

+ mvn compile：将项目源代码编译，即构建到默认生命周期的compile阶段

+ mvn package：将项目打包，注意，打包之前要先编译，生命周期里的每一个阶段是依次执行的

Maven还有一个clean生命周期，是用来清理项目编译、打包后产生的文件的，即清理target文件夹：

+ mvn clean

### 插件

上面说的生命周期、阶段是概念上的东西，实际上完成这些构建工作的是maven里的插件，`mvn phase`命令就是去调用这些插件的。常用的命令，maven里都有默认的标准插件。

但这些标准的插件有时不能满足需求。比如3.6.3版本的maven里，对应于package阶段的插件，用于将项目打包。但在将项目打成jar包时，是不会将项目引入的依赖包也打进包里去的。

这时，我们可以在pom.xml文件中通过`<build>`标签对项目的构建流程进行配置：

``` xml
<project>
    ...
    <build>
        <plugins>
            <plugin>
                <!-- 指定构建要用的插件的坐标，maven会去自动下载 -->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.1</version>
                <executions>
                    <execution>
                        <!-- 指定哪个阶段使用 -->
                        <phase>package</phase>
                        <goals>
                            <!-- 插件里的方法 -->
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer>
                                    <!-- 指明打包成的jar包的执行入口，即main方法所在的类 -->
                                    <mainClass>club.zunhuier.main</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

+ `maven-shade-plugin`插件会把我们的项目打成两个jar包，一个带original标记的jar包，是不含依赖包的，另一个是不带original标记，包含依赖包。

## Maven的其它作用

Maven还有一些更高级的用途，可以根据需要去了解使用。

比如，将一个大型项目拆分为一个个的子模块，通过maven去管理它们之间的依赖关系。

比如，通过maven将项目发布。

还有值得说的一点是，我们当然不是就简简单单地新建一个文件夹，作为项目根目录，然后用记事本写代码，保存源代码文件，然后用maven在这个文件夹下通过命令行对整个项目进行构建、管理。

这种方式可以是可以，但在如今IDE越来越智能化的情况下，我们还是要用IDE来提高我们的开发效率^ _ ^

最好就是在IDE中使用maven。像Eclipse、IDEA这些IDE都支持并内置maven，具体的配置过程可以自行查找。

## 后记

像maven这样的项目构建、管理工具，其实有过以下几代的发展过程：

**make  ——>  ant ——>  maven  ——>  gradle**

gradle是比较新的工具，但目前主流的还是用maven。

像在linux系统上，要对一些软件进行编译，可能就会用到make了。