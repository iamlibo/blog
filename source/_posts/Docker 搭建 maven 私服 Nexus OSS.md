---
title: Docker 搭建maven私服Nexus OSS
date: 2016-07-08 14:12:59
tags: [docker,nexus,maven]
---
本文使用Docker容器提供Nexus服务，并且前端使用Nginx进行代理。
<!--more-->

[Nexus](http://www.sonatype.org/nexus/)是搭建Maven私服的开源软件，[下载地址](http://www.sonatype.com/download-oss-sonatype)，看来3.X版本已经可以做为Docker的仓库了，但这不是本文要说的内容，以后再研究。
本文已经安装好Docker的环境，CentOS7+Docker1.10
```
[root@localhost ~]# docker version
Client:
 Version:         1.10.3
 API version:     1.22
 Package version: docker-common-1.10.3-44.el7.centos.x86_64
 Go version:      go1.4.2
 Git commit:      9419b24-unsupported
 Built:           Fri Jun 24 12:09:49 2016
 OS/Arch:         linux/amd64

Server:
 Version:         1.10.3
 API version:     1.22
 Package version: docker-common-1.10.3-44.el7.centos.x86_64
 Go version:      go1.4.2
 Git commit:      9419b24-unsupported
 Built:           Fri Jun 24 12:09:49 2016
 OS/Arch:         linux/amd64

```
Docker hub 上nexus的镜像地址是[https://hub.docker.com/r/sonatype/nexus/](https://hub.docker.com/r/sonatype/nexus/),但下载速度挺慢的。使用daocloud.io会快很多，具体方法见daocloud.io的[官方文档](https://dashboard.daocloud.io/nodes/new)，或者使用阿里云的镜像，使用方法见[这里](http://www.51liveup.cn/2016/07/05/Docker%20%E4%BD%BF%E7%94%A8%E9%98%BF%E9%87%8C%E4%BA%91%E5%8A%A0%E9%80%9F%E9%95%9C%E5%83%8F%E6%9C%8D%E5%8A%A1/)。

使用daoclound的镜像下载nexus镜像
```
[root@localhost ~]# dao pull sonatype/nexus:oss

Pulling repository sonatype/nexus:oss 

ac7d6c290ee0: Download complete                                                                                   
a3ed95caeb02: Download complete                                                                                   
73011fd28201: Download complete                                                                                   
14bf1cb804f4: Download complete                                                                                   
7138b67bf6c5: Download complete                                                                                   
Pull sonatype/nexus:oss complete, you can find it with 'docker images'
```
使用docker images 命令可以看到刚下载的镜像
```
[root@localhost ~]# docker images
REPOSITORY                                    TAG                 IMAGE ID            CREATED             SIZE
sonatype/nexus                                oss                 1175f465c39f        12 weeks ago        455.4 MB
```
使用docker run命令启动镜像
```
docker run -d --name nexus -p 8081:8081 sonatype/nexus:oss
```
这样就可以通过宿主机的8081端口访问nexus oss了.
因为容器是可以随时删除或更新的，这样用户数据就不能存到容器中，可以通过 -v 这个参数将宿主机的一个目录挂载到容器的工作目录.
```
docker run -d --name nexus -p 8081:8081 -v /data/nexus-data:/sonatype-work:rw --privileged=true sonatype/nexus:oss
```
挂载时如果启动不成功可能是目录的权限问题
nexus 使用UID 200 来启动程序的，所以只要将挂载的目录给UID=200相应的权限即可
```
chmod -R 200 /data/nexus-data
```
然后再重新启动一个容器就可以了。可以通过docker logs -f neuxs 查看日志。
