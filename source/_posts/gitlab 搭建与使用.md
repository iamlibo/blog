---
title: gitlab 搭建与使用
date: 2016-07-04 14:50:24
tags: gitlab
---

gitlabe 可以称得上版本帝了，更新速度真是太快了！
本文档是在CentOS7 上安装8.6.6 CE版，并进行汉化。
<!-- more -->
## 安装

[官网下载地址](https://about.gitlab.com/downloads/)
[中文版网站](https://www.gitlab.cc/downloads/)
[非最新版本下载地址](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/)

安装时有清华大学的镜像和浙江大学的镜像，速度还是比较快的。

>使用清华大学的镜像安装可以参照这个[文档](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)
新建 /etc/yum.repos.d/gitlab-ce.repo，内容为
```javascript
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```
>然后再执行
```javascript
sudo yum makecache
sudo yum install gitlab-ce
```

安装指定的版本
```javascript
yum install gitlab-ce-8.6.6-ce.3.el7.x86_64
```
安装成功后执行
```javascript
gitlab-ctl reconfigure
```
就可以通过浏览器访问http://localhost/就可以显示gitlab的登录页了。
首次登录用户名密码：
```
root
5iveL!fe
```

## 配置
gitlab的所有配置项都在/etc/gitlab/gitlab.rb文件中，修改配置文件后重新执行
```
gitlab-ctl reconfigure
```即可生效。下面主要列出我使用的几个配置项（都是在gitlab.rb文件中修改）

### 修改访问的域名
```
external_url 'http://git.host.com' 
```
### 配置email
配置email发送邮件，有两处需要修改,本例使用的是QQ的企业邮。
```ruby
############################
# gitlab.yml configuration #
############################
 gitlab_rails['gitlab_email_enabled'] = true
 gitlab_rails['gitlab_email_from'] = 'service@host.cn'
 gitlab_rails['gitlab_email_display_name'] = 'gitlab'
 gitlab_rails['gitlab_email_reply_to'] = 'service@host.cn'
```
```ruby
################################
# GitLab email server settings #
################################
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "service@host.cn"
gitlab_rails['smtp_password'] = "password"
gitlab_rails['smtp_domain'] = "exmail.qq.com"
gitlab_rails['smtp_authentication'] = "plain"
gitlab_rails['smtp_enable_starttls_auto'] = true
```
### 指定备份路径
```ruby
## For setting up backups
 gitlab_rails['backup_path'] = "/data/gitbackups"
 gitlab_rails['backup_keep_time'] = 604800
```
### 修改存储路径
```
## For setting up different data storing directory
# git_data_dir "/var/opt/gitlab/git-data"
 git_data_dir "/data/git-data"
```

## 使用
### 设置自动备份计划
创建自动备份计划，在centos命令行输入
```javascript
crontab -e
0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create
```
这样就可以每天凌晨2点进行一次自动备份,备份文件就会保存到上面指定的备份路径下。

### 手动备份与恢复
执行下面命令就会在备份路径中生成一个备份
```
gitlab-rake gitlab:backup:create
```
生的文件是1467171356_gitlab_backup.tar这样的压缩包，前面一串数字是时间。

恢复备份
```javascript
# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

# 从1467171356编号备份中恢复,就是上面备份文件的前面数字部分
gitlab-rake gitlab:backup:restore BACKUP=1467171356

# 启动Gitlab
sudo gitlab-ctl start
```
在备份与恢复的时候如果版本不一致会有提示，将不能恢复，所以此方法不适合迁移升级。

### 汉化
确认版本：
```
sudo cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
```
并确认当前汉化版本的 VERSION 是否相同，当前最新的汉化版本为 8.6 。
如果安装版本小于当前汉化版本，请先升级。如果安装版本大于当前汉化版本，请在本项目中提交新的 issue。
如果版本相同，首先在本地 clone 仓库。
```
# GitLab.com 仓库
git clone https://gitlab.com/larryli/gitlab.git
```

然后比较汉化分支和原分支，导出 patch 用的 diff 文件。
```
# 8.6 版本的汉化补丁
git diff origin/8-6-stable..8-6-zh > ../8.6.diff
```
然后上传 8.6.diff 文件到服务器。
```
# 停止 gitlab
sudo gitlab-ctl stop
sudo patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < 8.6.diff
```
确定没有 .rej 文件，重启 GitLab 即可。
```
sudo gitlab-ctl start
```
