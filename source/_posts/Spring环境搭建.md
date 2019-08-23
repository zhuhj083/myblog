---
title: Spring环境搭建
date: 2018-08-01 11:08:39
tags: [spring]
categories: [java]
---
Spring的源码在Github上，基于Gradle的构建来构建项目。所以要构建Spring源码首先要安装Github以及Gradle。
github的安装不再赘述。

# 一.安装gradle
Gradle是一个基于Groovy的构建工具，它使用Groovy来编写构建脚本，支持依赖管理和多项目构建，类似于maven。

下载地址为：http://gradle.org/downloads

## windows下安装gradle

下载后解压到指定目录，我放在了`C:\Program Files\gradle-1.6`

然后配置环境变量
1. 根据对应目录创建GRADLE_HOME系统变量
2. 将系统变量加入到path中
3. 测试，打开命令窗口输入`gradle -version`,如果安装成功会出现Gradle对应的版本信息。

## Mac下brew安装Gradle

1.打开终端，使用`brew install gradle`完成安装

```sh
$ brew info gradle
```

输出：

```sh
...省略
==> Downloading https://services.gradle.org/distributions/gradle-5.6-all.zip
==> Downloading from https://downloads.gradle-dn.com/distributions/gradle-5.6-all.zip
######################################################################## 100.0%
🍺  /usr/local/Cellar/gradle/5.6: 14,317 files, 245.0MB, built in 45 seconds
```



2.安装成功，默认的安装位置为：
`/usr/local/Cellar/gradle/5.6`

3.验证是否安装成功，`gradle -v`,打印下面信息说明安装成功

```sh
$ gradle -v

------------------------------------------------------------
Gradle 5.6
------------------------------------------------------------

Build time:   2019-08-14 21:05:25 UTC
Revision:     f0b9d60906c7b8c42cd6c61a39ae7b74767bb012

Kotlin:       1.3.41
Groovy:       2.5.4
Ant:          Apache Ant(TM) version 1.9.14 compiled on March 12 2019
JVM:          1.8.0_191 (Oracle Corporation 25.191-b12)
OS:           Mac OS X 10.14.6 x86_64
```



# 二.下载Spring

例如要将下载的源码存储到g:\spring下，进入这个目录，输入一下命令
```bash
$ git clone git@github.com:spring-projects/spring-framework.git spring-framework
```
等待一段时间后，完成下载。

进入这个目录，会看到已经存在了相应的源码信息
```bash
$ cd spring-framework/
$ ll
```


# 三.构建spring并导入IDEA
当前的源码并不可以直接导入到IDEA中，我们还需要将源码转换为IDEA可以读取的形式。

参考：https://github.com/spring-projects/spring-framework/blob/master/import-into-idea.md

## 步骤：

### 1.Precompile spring-oxm with ./gradlew :spring-oxm:compileTestJava

```bash
$ ./gradlew :spring-oxm:compileTestJava
```

输出：

```bash
Downloading https://services.gradle.org/distributions/gradle-5.6-bin.zip
.........................................................................................

Welcome to Gradle 5.6!

Here are the highlights of this release:
 - Incremental Groovy compilation
 - Groovy compile avoidance
 - Test fixtures for Java projects
 - Manage plugin versions via settings script

For more details see https://docs.gradle.org/5.6/release-notes.html

Starting a Gradle Daemon (subsequent builds will be faster)

> Task :spring-core:compileTestJava
注: 某些输入文件使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。

> Task :spring-beans:compileTestJava
注: 某些输入文件使用了未经检查或不安全的操作。
注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。

> Task :spring-context:compileTestJava
注: 某些输入文件使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
注: 某些输入文件使用了未经检查或不安全的操作。
注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。

> Task :spring-oxm:genJaxb
[ant:javac] : warning: 'includeantruntime' was not set, defaulting to build.sysclasspath=last; set to false for repeatable builds

BUILD SUCCESSFUL in 4m 50s
49 actionable tasks: 49 executed
```



### 2.导入IntelliJ

步骤：

(File -> New -> Project from Existing Sources -> 打开项目根目录 -> 选择 build.gradle)

### 3编译相关模块

1.因为其他项目需要依赖spring-core和spring-oxm，所以我们导入后需要先编译这两个



![spring-core](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/2BF6BF68-CC3B-4F87-B6EB-B549D7023CD9.png)



![spring-oxm](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/F3CC7F89-FCFC-4077-B2F3-73F0EF852C58.png)

2.执行完了后，接着编译spring-context，spring-bean,同样的方法

3.最后编译spring-aspects和spring-aop



### 4.CoroutinesUtils类 找不到的处理

在spring-core项目的kotlin-coroutines的build目录下找到`kotlin-coroutines-5.2.0.BUILD-SNAPSHOT.jar`.再引入spring-core项目中即可。

### 5.完成

到了这里，spring的源码基本编译完成，然后就可以debug了

