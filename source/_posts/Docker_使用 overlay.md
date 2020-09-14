---
title: Docker_使用 overlay
date: 2016-06-07 14:43:18
tags: [docker]
---

centos7 中docker 要使用overlay存储必须升级到7.2

<!--more-->
系统升级
```
sudo yum upgrade --assumeyes --tolerant
sudo yum update --assumeyes
```

确认内核
```
  uname -r
3.10.0-327.10.1.el7.x86_64
```

启用overlay
```
$ sudo tee /etc/modules-load.d/overlay.conf <<-'EOF'
overlay
EOF
```
重启系统
```
reboot
```

确认 overlay启用

```
$ lsmod | grep overlay
overlay
```
如果没有安装过docker可以参照[原文](http://www.codexiu.cn/linux/blog/16526/)继续安装，我是已经安装过最新版的docker,所以略过安装，执行下面的操作

```
systemctl stop docker 
rm -rf /var/lib/docker
```

修改/etc/sysconfig/docker文件，将OPTIONS='--selinux-enabled' 改成 OPTIONS='--selinux-enabled=false'
```
#OPTIONS='--selinux-enabled'
OPTIONS='--selinux-enabled=false'
```

然后修改/etc/sysconfig/docker-storage这个文件
```
DOCKER_STORAGE_OPTIONS= -s overlay

```
重启docker服务
```
systemctl start docker
```
执行docker info
```
Containers: 0
Images: 0
Server Version: 1.9.1
Storage Driver: overlay
 Backing Filesystem: xfs
Execution Driver: native-0.2
Logging Driver: json-file
Kernel Version: 3.10.0-327.18.2.el7.x86_64
Operating System: CentOS Linux 7 (Core)
CPUs: 4
Total Memory: 7.536 GiB

```
至此修改overlay存储成功。

参照
http://www.codexiu.cn/linux/blog/16526/


