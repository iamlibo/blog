---
title: RedHat6.5使用yum源
date: 2016-10-12 15:10:42
tags: [redhat]
categories: [技术,redhat]
---

因为RedHat使用yum 命令需要注册，所以可以使用CentOS的yum来替换掉Redhat的yum

1、删除RedHat自带的yum

```
rpm -aq|grep yum|xargs rpm -e --nodeps
```

2、下载yum的安装文件，使用的163的镜像 http://mirrors.163.com/centos/6/os/x86_64/

在RedHat6.5上，2016年10安装 下面最新的包，版本可能会有变化
```
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-3.2.29-73.el6.centos.noarch.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-metadata-parser-1.1.2-16.el6.x86_64.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.30-37.el6.noarch.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/python-iniparse-0.3.1-2.1.el6.noarch.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/python-urlgrabber-3.9.1-11.el6.noarch.rpm
```
3、安装yum命令
```
rpm -Uvh --replacepkgs *.rpm
```
需要将上面下载的rpm包放到一个单独的目录下，执行安装。如果有其他rpm包也一块安装的。

4、更新repo文件
```
cd /etc/yum.repos.d/
mv rhel-source.repo rhel-source.repo.bak
wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
```
修改 CentOS6-Base-163.repo 文件，将$releasever全部分替换成RedHat的版本号就可以了

修改后的内容如下：
```
[root@vmtemplate yum.repos.d]# cat CentOS6-Base-163.repo 
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-6 - Base - 163.com
baseurl=http://mirrors.163.com/centos/6/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=6&arch=$basearch&repo=os
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6

#released updates
[updates]
name=CentOS-6 - Updates - 163.com
baseurl=http://mirrors.163.com/centos/6/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=6&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6

#additional packages that may be useful
[extras]
name=CentOS-6 - Extras - 163.com
baseurl=http://mirrors.163.com/centos/6/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=6&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-6 - Plus - 163.com
baseurl=http://mirrors.163.com/centos/6/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=6&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6

#contrib - packages by Centos Users
[contrib]
name=CentOS-6 - Contrib - 163.com
baseurl=http://mirrors.163.com/centos/6/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=6&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-6
```

然后执行
```
yum clean all
yum makecache
```
然后就可以正常使用yum命令了