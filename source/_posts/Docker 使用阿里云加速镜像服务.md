---
title: Docker 使用阿里云加速镜像服务
date: 2016-07-05 14:20:34
tags: docker
---
使用Docker默认的仓库在国内是相当的慢，还好阿里云提供了镜像服务（当然还有别家）
在CentOS7 + Docker1.10的环境下按照阿里提供的帮助文档怎么也不好用，体验不到加速度的感觉.
<!--more-->
centos7 下是用 systemd来管理docker进程的，那看一下docker.service的内容
```
[root@localhost docker]# cat /etc/systemd/system/docker.service 
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target
Wants=docker-storage-setup.service

[Service]
Type=notify
NotifyAccess=all
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
EnvironmentFile=-/etc/sysconfig/docker-network
Environment=GOTRACEBACK=crash
ExecStart=/bin/sh -c '/usr/bin/docker-current daemon \
          --exec-opt native.cgroupdriver=systemd \
          $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $ADD_REGISTRY \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY \
          2>&1 | /usr/bin/forward-journald -tag docker'
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity
TimeoutStartSec=0
MountFlags=slave
Restart=on-abnormal
StandardOutput=null
StandardError=null

[Install]
WantedBy=multi-user.target

```
可以看到环境变量是在/etc/sysconfig/docker，/etc/sysconfig/docker-storage，/etc/sysconfig/docker-network内。在/etc/sysconfig/docker内有一个ADD_REGISTRY配置项，是被注掉的，把注释去掉，改成下面内容
```
ADD_REGISTRY='--add-registry 0bzi0oqj.mirror.aliyuncs.com'
```
然后执行
```
systemctl daemon-reload
systemctl restart docker
```
再pull一个镜像试试，是不是快很多呢？
```
[root@localhost home]# docker pull sonatype/nexus
Using default tag: latest
Trying to pull repository 0bzi0oqj.mirror.aliyuncs.com/sonatype/nexus ... 
latest: Pulling from 0bzi0oqj.mirror.aliyuncs.com/sonatype/nexus
a3ed95caeb02: Pull complete 
5989106db7fb: Pull complete 
ac7d6c290ee0: Pull complete 
73011fd28201: Pull complete 
14bf1cb804f4: Pull complete 
7138b67bf6c5: Pull complete 
Digest: sha256:29e2af8c2f8af0a557d7feeef6675a9c9305ed935bd5aca8a207b9f642659e1a
Status: Downloaded newer image for 0bzi0oqj.mirror.aliyuncs.com/sonatype/nexus:latest
```
如果有Trying to pull repository 0bzi0oqj.mirror.aliyuncs.com/sonatype/nexus这个提示就证明你已经享受加速服务了！
