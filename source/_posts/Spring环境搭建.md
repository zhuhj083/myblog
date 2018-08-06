---
title: Spring环境搭建
date: 2018-08-01 11:08:39
tags: [spring]
categories: [java]
---
Spring的源码在Github上，基于Gradle的构建来构建项目。所以要构建Spring源码首先要安装Github以及Gradle。
github的安装不再赘述。

# 安装gradle
Gradle是一个基于Groovy的构建工具，它使用Groovy来编写构建脚本，支持依赖管理和多项目构建，类似于maven。

下载地址为：http://gradle.org/downloads

下载后解压到指定目录，我放在了`C:\Program Files\gradle-1.6`

然后配置环境变量
1. 根据对应目录创建GRADLE_HOME系统变量
2. 将系统变量加入到path中
3. 测试，打开命令窗口输入`gradle -version`,如果安装成功会出现Gradle对应的版本信息。

# 下载Spring
例如要将下载的源码存储到g:\spring下，进入这个目录，输入一下命令
```bash
git clone git@github.com:spring-projects/spring-framework.git
cd spring-framework
```
等待一段时间后，完成下载。
```bash
$ git clone git@github.com:spring-projects/spring-framework.git
Cloning into 'spring-framework'...
remote: Counting objects: 438994, done.
remote: Compressing objects: 100% (66/66), done.
remote: Total 438994 (delta 6), reused 39 (delta 0), pack-reused 438914
Receiving objects: 100% (438994/438994), 111.82 MiB | 2.83 MiB/s, done.
Resolving deltas: 100% (211139/211139), done.
Checking out files: 100% (8092/8092), done.
```
进入这个目录，会看到已经存在了相应的源码信息
```bash
$ cd Spring-framework/
$ ll
total 53
-rw-r--r-- 1 zhuhaijun 1049089 11072 8月   1 12:06 build.gradle
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 buildSrc/
-rw-r--r-- 1 zhuhaijun 1049089  2395 8月   1 12:06 CODE_OF_CONDUCT.adoc
-rw-r--r-- 1 zhuhaijun 1049089  6401 8月   1 12:06 CONTRIBUTING.md
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 gradle/
-rw-r--r-- 1 zhuhaijun 1049089    30 8月   1 12:06 gradle.properties
-rwxr-xr-x 1 zhuhaijun 1049089  5533 8月   1 12:06 gradlew*
-rw-r--r-- 1 zhuhaijun 1049089  2348 8月   1 12:06 gradlew.bat
-rw-r--r-- 1 zhuhaijun 1049089  2486 8月   1 12:06 import-into-eclipse.md
-rw-r--r-- 1 zhuhaijun 1049089  1868 8月   1 12:06 import-into-idea.md
-rw-r--r-- 1 zhuhaijun 1049089  2290 8月   1 12:06 README.md
-rw-r--r-- 1 zhuhaijun 1049089   831 8月   1 12:06 settings.gradle
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-aop/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-aspects/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-beans/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-context/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-context-indexer/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-context-support/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-core/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-expression/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-framework-bom/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-instrument/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-jcl/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-jdbc/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-jms/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-messaging/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-orm/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-oxm/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-test/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-tx/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-web/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-webflux/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-webmvc/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 spring-websocket/
drwxr-xr-x 1 zhuhaijun 1049089     0 8月   1 12:06 src/
```


# 构建spring
当前的源码并不可以直接导入到Eclipse中，我们还需要将源码转换为Eclipse可以读取的形式。

在每个目录下，一个个地执行 gradle cleanidea eclipse
```bash
$ gradle cleanidea eclipse
Starting a Gradle Daemon, 1 incompatible Daemon could not be reused, use --statu                                                                                                                                                                                               s for details
Generating JAR file 'gradle-api-4.9.jar'
> Task :buildSrc:compileJava NO-SOURCE
> Task :buildSrc:compileGroovy
> Task :buildSrc:processResources
> Task :buildSrc:classes
> Task :buildSrc:jar
> Task :buildSrc:assemble
> Task :buildSrc:compileTestJava NO-SOURCE
> Task :buildSrc:compileTestGroovy NO-SOURCE
> Task :buildSrc:processTestResources NO-SOURCE
> Task :buildSrc:testClasses UP-TO-DATE
> Task :buildSrc:test NO-SOURCE
> Task :buildSrc:check UP-TO-DATE
> Task :buildSrc:build
> Task :spring-tx:cleanIdeaModule UP-TO-DATE
> Task :spring-tx:cleanIdea UP-TO-DATE
> Task :spring-tx:eclipseClasspath
> Task :spring-tx:eclipseJdtPrepare
> Task :spring-tx:eclipseJdt
> Task :spring-tx:eclipseProject
> Task :spring-tx:eclipseSettings
> Task :spring-tx:eclipseWstComponent
> Task :spring-tx:eclipse

BUILD SUCCESSFUL in 36s
8 actionable tasks: 6 executed, 2 up-to-date
```
