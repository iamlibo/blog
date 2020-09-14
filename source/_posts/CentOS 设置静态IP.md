---
title: CentOS 设置静态IP
date: 2015-09-21 00:00:00
tags: [CentOS]
categories: [技术,linux]
---

本篇内容最初发表于我的 [另一个博客](http://my.oschina.net/thinker4self/blog/509218)
<!-- more -->

## 1.修改网卡配置　编辑：vi /etc/sysconfig/network-scripts/ifcfg-eth0

　DEVICE=eth0 #描述网卡对应的设备别名，例如ifcfg-eth0的文件中它为eth0

　BOOTPROTO=static #设置网卡获得ip地址的方式，可能的选项为static，dhcp或bootp，分别对应静态指定的 ip地址，通过dhcp协议获得的ip地址，通过bootp协议获得的ip地址
　BROADCAST=192.168.0.255 #对应的子网广播地址
　HWADDR=00:07:E9:05:E8:B4 #对应的网卡物理地址

　IPADDR=12.168.0.33 #如果设置网卡获得 ip地址的方式为静态指定，此字段就指定了网卡对应的ip地址
　NETMASK=255.255.255.0 #网卡对应的网络掩码
　NETWORK=192.168.0.0 #网卡对应的网络地址

## 2.修改网关配置

　编辑：vi /etc/sysconfig/network　修改后如下：　
```
NETWORKING=yes
HOSTNAME=centos
GATEWAY=192.168.0.1

```
　NETWORKING=yes(表示系统是否使用网络，一般设置为yes。如果设为no，则不能使用网络，而且很多系统服务程序将无法启动)
　HOSTNAME=centos(设置本机的主机名，这里设置的主机名要和/etc/hosts中设置的主机名对应)
　GATEWAY=192.168.0.1(设置本机连接的网关的IP地址。)

　我在修改这里打开编辑时前三项已经默认有了所以只增加了GATEWAY

3.修改DNS 配置

  编辑：vi /etc/resolv.conf　修改后如下：　
```
nameserver 202.98.0.68
nameserver 202.98.5.68

```
  nameserver　即是DNS服务器IP地址，第一个是首选，第二个是备用。

4.重启网络服务

　　执行命令：
```
　　service network restart
```
 　或
 ```
 /etc/init.d/network restart
 ```
 