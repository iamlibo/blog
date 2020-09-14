---
title: Docker_使用杂记
date: 2016-05-30 10:28:03
tags: [docker]
---

<!--more-->

1、执行 docker version 时提示
```
Docker can't connect to docker daemon
```
是因为docker server 没有在后台运行，使用下面命令运行
```
docker daemon &

```

2、杀死所有正在运行的容器
```
docker kill $(docker ps -a -q)
```
3、删除所有已经停止的容器
```
docker rm $(docker ps -a -q)
```

4、删除所有未打 dangling 标签的镜像
```
docker rmi $(docker images -q -f dangling=true)
```
5、除所有镜像
```
docker rmi $(docker images -q)
```

为这些命令创建别名
```
# ~/.bash_aliases
# 杀死所有正在运行的容器.
alias dockerkill='docker kill $(docker ps -a -q)'
# 删除所有已经停止的容器.
alias dockercleanc='docker rm $(docker ps -a -q)'
# 删除所有未打标签的镜像.
alias dockercleani='docker rmi $(docker images -q -f dangling=true)'
# 删除所有已经停止的容器和未打标签的镜像.
alias dockerclean='dockercleanc || true && dockercleani'
```