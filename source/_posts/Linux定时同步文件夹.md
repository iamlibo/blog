---
title: Linux定时同步文件夹
date: 2016-10-20 17:08:43
tags: [linux]
categories: [技术,linux]
---

# rsync 简介

Rsync是linux/Unix文件同步和传送工具。用于替代rcp的一个工具，可通过 LAN/WAN 快速同步多台主机之间的文
件。也可以使用 rsync 同步本地硬盘中的不同目录。rsync 使用所谓的rsync算法进行数据同步，这种算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快。

本次使用rsync定时同步两台机器间的文件夹，Linux是redhat6.5，默认已经安装了rsync.
<!--more-->
# 同步文件夹

首次使用如下命令进行全量同步，并将同步的日志记录在/var/logs/rsync.log文件中。
```
rsync -ravpogt --log-file=/var/logs/rsync.log --log-file-format='%i %o %f %M %t' /source/path IP:/target/path
```
会显示同步的日志
```
sending incremental file list
....
....
....
sent 5117763 bytes  received 90 bytes  2047141.20 bytes/sec
total size is 77135632871  speedup is 15071.87
```
第一次同步完成，就需要定时去执行这个命令了，Linux的cron正好可以做这件事,将上面命令保存到一个文件中，并设置可执行权限.
```
vim syncfile.sh
```
加入下面内容：
```
rsync -ravpogt --log-file=/var/logs/rsync.log --log-file-format='%i %o %f %M %t' /source/path IP:/target/path
```
注意要修改IP目录等信息。
设置可执行权限
```
chmod 755 syncfile.sh
```
然后定时执行这个文件就可以。
使用命令编辑定时任务内容：
```
crontab -e
```
增加内容如下：
```
* * * * * sh /root/syncfile.sh
* * * * * sleep 3 && sh /root/syncfile.sh
* * * * * sleep 6 && sh /root/syncfile.sh
* * * * * sleep 9 && sh /root/syncfile.sh
* * * * * sleep 12 && sh /root/syncfile.sh
* * * * * sleep 15 && sh /root/syncfile.sh
* * * * * sleep 18 && sh /root/syncfile.sh
* * * * * sleep 21 && sh /root/syncfile.sh
* * * * * sleep 24 && sh /root/syncfile.sh
* * * * * sleep 27 && sh /root/syncfile.sh
* * * * * sleep 30 && sh /root/syncfile.sh
* * * * * sleep 33 && sh /root/syncfile.sh
* * * * * sleep 36 && sh /root/syncfile.sh
* * * * * sleep 39 && sh /root/syncfile.sh
* * * * * sleep 42 && sh /root/syncfile.sh
* * * * * sleep 45 && sh /root/syncfile.sh
* * * * * sleep 48 && sh /root/syncfile.sh
* * * * * sleep 51 && sh /root/syncfile.sh
* * * * * sleep 54 && sh /root/syncfile.sh
* * * * * sleep 57 && sh /root/syncfile.sh
```
上面的内容是指每三秒执行一次syncfile.sh。

# rsync 参数

rsync参数
```
-v, --verbose                  详细模式输出
-q, --quiet                       精简输出模式
-c, --checksum               打开校验开关，强制对文件传输进行校验
-a, --archive                   归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
-r, --recursive                 对子目录以递归模式处理
-R, --relative                  使用相对路径信息
-b, --backup                  创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
--backup-dir                  将备份文件(如~filename)存放在在目录下。
-suffix=SUFFIX             定义备份文件前缀
-u, --update                   仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件)
-l, --links                         保留软链结
-L, --copy-links              想对待常规文件一样处理软链结
--copy-unsafe-links        仅仅拷贝指向SRC路径目录树以外的链结
--safe-links                     忽略指向SRC路径目录树以外的链结
-H, --hard-links              保留硬链结
-p, --perms                    保持文件权限
-o, --owner                    保持文件属主信息
-g, --group                     保持文件属组信息
-D, --devices                 保持设备文件信息
-t, --times                      保持文件时间信息
-S, --sparse                   对稀疏文件进行特殊处理以节省DST的空间
-n, --dry-run                  现实哪些文件将被传输
-W, --whole-file             拷贝文件，不进行增量检测
-x, --one-file-system      不要跨越文件系统边界
-B, --block-size=SIZE   检验算法使用的块尺寸，默认是700字节
-e, --rsh=COMMAND 指定使用rsh、ssh方式进行数据同步
--rsync-path=PATH      指定远程服务器上的rsync命令所在路径信息
-C, --cvs-exclude          使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件
--existing                      仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件
--delete                         删除那些DST中SRC没有的文件
--delete-excluded          同样删除接收端那些被该选项指定排除的文件
--delete-after                传输结束以后再删除
--ignore-errors             及时出现IO错误也进行删除
--max-delete=NUM     最多删除NUM个文件
--partial                        保留那些因故没有完全传输的文件，以是加快随后的再次传输
--force                          强制删除目录，即使不为空
--numeric-ids                不将数字的用户和组ID匹配为用户名和组名
--timeout=TIME IP       超时时间，单位为秒
-I, --ignore-times          不跳过那些有同样的时间和长度的文件
--size-only                    当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间
--modify-window=NUM 决定文件是否时间相同时使用的时间戳窗口，默认为0
-T --temp-dir=DIR      在DIR中创建临时文件
--compare-dest=DIR   同样比较DIR中的文件来决定是否需要备份
-P 等同于 --partial
--progress                    显示备份过程
-z, --compress             对备份的文件在传输时进行压缩处理
--exclude=PATTERN  指定排除不需要传输的文件模式
--include=PATTERN   指定不排除而需要传输的文件模式
--exclude-from=FILE   排除FILE中指定模式的文件
--include-from=FILE   不排除FILE指定模式匹配的文件
--version                      打印版本信息
--address                     绑定到特定的地址
--config=FILE             指定其他的配置文件，不使用默认的rsyncd.conf文件
--port=PORT              指定其他的rsync服务端口
--blocking-io               对远程shell使用阻塞IO
-stats                           给出某些文件的传输状态
--progress                   在传输时现实传输过程
--log-format=formAT  指定日志文件格式
--password-file=FILE 从FILE中得到密码
--bwlimit=KBPS         限制I/O带宽，KBytes per second
-h, --help                    显示帮助信息
```
