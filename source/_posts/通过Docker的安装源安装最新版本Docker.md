---
title: 通过Docker的安装源安装最新版本Docker
date: 2016-05-26 13:32:50
tags: [docker]
categories: [技术,docker]
---

使用ubuntu、centos安装docker都不是最新的版本。

在ubuntu中使用下面安装到最新版本:

要安装最新的 Docker 版本，首先需要安装 apt-transport-https 支持，之后通过添加源来安装。

```
$ sudo apt-get install apt-transport-https

$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9

$ sudo bash -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"

$ sudo apt-get update

$ sudo apt-get install lxc-docker

```
centos 7 直接执行yum可以安装到当前最新版1.9.1