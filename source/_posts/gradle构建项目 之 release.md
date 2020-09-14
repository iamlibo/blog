---
title: gradle构建项目 之 release
date: 2016-04-16 21:42:26
tags: [gradle]
categories: [技术,gradle]
---

今天要写的内容主要是关于使用gradle进行release的。

先看一下release的流程：
1、把稳定的代码提交到git-master(当然也可以使用其他分支）
2、定义好本次release的版本，如：1.0-release
3、创建1.0-release分支
4、更改下配置文件中的下一个开发版本的代号，如：1.1-SNAPSHOT
5、提交1.1-SNAPSHOT到master

这些步骤手工做起来也能很快完成，但主要是我们要自动化啊。。。

那么在gradle中使用[`gradle-release`](https://github.com/researchgate/gradle-release)插件来搞定这件事。

<!-- more -->
IntelliJ Idea 号称是groovy最好的IDE，但在写gradle的配置时也是没有智能提示。。。

先看build.gradle的内容：
```java
buildscript {
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath "net.researchgate:gradle-release:2.3.5"
    }
}
//release plugin
plugins {
    id "net.researchgate.release" version "2.3.5"
}

group 'cn.myplus'
version '1.1-SNAPSHOT'

//引入其他文件,其他文件的内容如下
apply from: "./libraries.gradle"

apply plugin: 'java'
apply plugin: 'maven'
apply plugin: 'idea'
apply plugin: "net.researchgate.release"
description = "myplus-core"

//使用maven资源库
repositories {
    //使用本地资源库
    mavenLocal()
    //oschina的maven镜像，但经常挂，不稳定
    maven { url "http://maven.oschina.net/content/groups/public/" }
    mavenCentral()
}

// project的额外属性，这里用于定义profile属性
ext {
    if (project.hasProperty('profile')) {
        profile = project['profile']
    } else {
        profile = "dev"
    }
    println "profile:" + profile
    // java文件编码方式设置为utf-8
    compileJava.options.encoding = 'UTF-8'
    compileTestJava.options.encoding = 'UTF-8'
}

/*------------------------
----- RELEASE PLUGIN -----
https://github.com/researchgate/gradle-release
--------------------------*/
release {
    failOnCommitNeeded = false
    failOnUnversionedFiles = false

    scmAdapters = [
            net.researchgate.release.GitAdapter
    ]
    git {
        requireBranch = 'master'
        pushToRemote = 'origin'
        pushToBranchPrefix = ''
    }
}

//工程依赖
dependencies {
    compile(libraries.slf4j_api)
    compile(libraries.spring_context)
    compile(libraries.spring_test)
    compile "ch.qos.logback:logback-classic:1.1.2"
    compile "commons-codec:commons-codec:1.5"

    testCompile(libraries.testng)
}

```
首先是在文件的顶部 buildscript{} 和 plugins{} 定义gradle-release插件的下载址版本信息等，然后使用
```
apply plugin: "net.researchgate.release"
```
在工程中应此插件。
插件的配置信息在这段代码中：
```
release {
    failOnCommitNeeded = false
    failOnUnversionedFiles = false

    scmAdapters = [
            net.researchgate.release.GitAdapter
    ]
    git {
        requireBranch = 'master'
        pushToRemote = 'origin'
        pushToBranchPrefix = ''
    }
}
```
本例使用的是git的资源库，还有支持其他的资源库：
```
net.researchgate.release.GitAdapter,
net.researchgate.release.SvnAdapter,
net.researchgate.release.HgAdapter,
net.researchgate.release.BzrAdapter
```
使用哪种版本，进行相应的配置就可以了，可以参照插件的[github](https://github.com/researchgate/gradle-release "gradle-release")地址

配置好以后执行`gradle release` 命令就可以执行发版本了。

如果没有什么错误（我是遇到问题了，一会再说）就可以显示执行结果，过程需要输入This release version: 和 Enter the next version ，这个根据实现情况输入就可以了。如果是使用IntelliJ Idea 这种集成工具执行，需要在gradle.properties 文件中加入
`version=1.3-SNAPSHOT`
执行结果如下：

```
Microsoft Windows [版本 6.3.9600]
(c) 2013 Microsoft Corporation。保留所有权利。

D:\git\gradle-test>gradle release
profile:"dev"
:release
profile:"dev"
:gradle-test:createScmAdapter
:gradle-test:initScmAdapter
:gradle-test:checkCommitNeeded
!!WARNING!! You have unversioned files:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
?? .gradle/
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
!!WARNING!! You have uncommitted files:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 M build.gradle
 M libraries.gradle
D  src/main/java/cn/myplus/config/MyPlusConfigureDevImpl.java
 M src/main/java/cn/myplus/config/MyPlusConfigureImpl.java
D  src/main/java/cn/myplus/config/MyPlusConfigureProductionImpl.java
D  src/main/java/cn/myplus/config/MyPlusConfigureTestImpl.java
A  src/main/resources/dev/myplus.properties
A  src/main/resources/pro/myplus.properties
A  src/main/resources/test/myplus.properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
:gradle-test:checkUpdateNeeded
:gradle-test:unSnapshotVersion
> Building 0% > :release > :gradle-test:confirmReleaseVersion
??> This release version: [1.1] 1.2
:gradle-test:confirmReleaseVersion
:gradle-test:checkSnapshotDependencies
:gradle-test:runBuildTasks
profile:"dev"
:gradle-test:beforeReleaseBuild UP-TO-DATE
:gradle-test:compileJava
:gradle-test:processResources
:gradle-test:classes
:gradle-test:jar
:gradle-test:assemble
:gradle-test:compileTestJava
:gradle-test:processTestResources UP-TO-DATE
:gradle-test:testClasses
:gradle-test:test
:gradle-test:check
:gradle-test:build
:gradle-test:afterReleaseBuild UP-TO-DATE
:gradle-test:preTagCommit
Running [git, commit, -a, -m, [Gradle Release Plugin] - pre tag commit:  '1.2'.]
 produced an error: [warning: LF will be replaced by CRLF in build.gradle.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in libraries.gradle.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in build.gradle.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in libraries.gradle.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in libraries.gradle.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in build.gradle.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in libraries.gradle.
The file will have its original line endings in your working directory.]
:gradle-test:createReleaseTag
> Building 0% > :release > :gradle-test:updateVersion
??> Enter the next version (current one released as [1.2]): [1.3-SNAPSHOT]
:gradle-test:updateVersion
:gradle-test:commitNewVersion

BUILD SUCCESSFUL

Total time: 40.96 secs
D:\git\gradle-test>
```

下面来说说使用过程中遇到的坑：
使用的是git.oschina.net的git（其实无论是哪个都会一样的）出现下面错误：
```
Running [git, remote, update] produced an error: [bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://git.oschina.net': Invalid argument
error: Could not fetch origin]
```
直观感觉就是没有用户名，执行过程中也确定没有提示输入用户名，因为一直使用的是https的方式，所以没有配置key什么的，后来在网上找到答案解决了这个问题，

第一种解决方案，这个地址http://blog.csdn.net/liukang325/article/details/24105913写得比较具体

>在C:\Documents and Settings\Administrator\ 目录下有一个  .gitconfig 的文件，里面会有你先前配好的name 和email，只需在下面加一行
```
[credential]      
    helper = store   
```
>下次再输入用户名 和密码 时，git就会记住，从而在C:\Documents and Settings\Administrator\ 目录下形成一个  .git-credentials 文件，里面就是保存的你的用户名和密码。
```
https://username:12345678@git.oschina.net  
```
这配置如果同时在git.oschina.net有多个用户好象就不灵了。

第二种方案就是在项目的.git文件夹中的config文件中进行配置，config文件中会下面代码
```
[remote "origin"]
	url = https://git.oschina.net/xxxxx/gradle-test.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```
只需要在url中的git.oschina.net前面加上用户名和密码就可以了
```
[remote "origin"]
	url = https://username:password@git.oschina.net/xxxxx/gradle-test.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```

恭喜你，如果你的用户或密码中没有@符号，应该可以顺利的访问资源库了，可是我的用户名是用邮箱注册的，所有这个url变成这样子了：
```
https://username@qq.com:password@git.oschina.net/xxxxx/gradle-test.git
```
一运行，去访问qq.com去了，看来这个url中根据@字符进行分隔的。
后来使用sourceTree保存了一下用户名和密码，查看他的配置文件中是把用户名的@字符转换成%40了，所以这个url应该是这样子：
```
https://username%40qq.com:password@git.oschina.net/xxxxx/gradle-test.git
```

至此，gradle-release插件正常运行。

