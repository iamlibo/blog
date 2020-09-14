---
title: docker使用过程记录_nginx
date: 2016-05-26 13:52:44
tags: [docker]
categories: [技术,docker]
---

容器运行后，进入容器的方法有两种，一种是在宿主主机通过 Docker exec -it $containerid /bin/bash 的方法，另一种方式是通过ssh远程登陆。


使用nginx的[官方镜像](https://hub.docker.com/_/nginx/).

使用下面命令安装nginx镜像
```
docker pull nginx

```
使用下面命令启动并映射80端口
```
docker run --name myngx1 -d -p 80:80 nginx

```
这时通过本机的80商品就可以访问nginx，看到欢迎界面了。

使用下面命令可以配置root目录。
```
docker run --name myngx2 -v /home/docker/www:/usr/share/nginx/html:ro -d -p 80:80 nginx
```
这时访问本机就可以看到部署在本机/home/docker/www目录的站点内容。

下面是nginx镜像默认的配置文件内容

```
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
最后一句引入/etc/nginx/conf.d/目录下的*.conf，在这个目录下只有default.conf这个文件，内容如下：
```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

如果想使用自定义配置文件使用下面命令启动容器
```
docker run --name myngx3 -v /home/docker/nginx.conf:/etc/nginx/nginx.conf:ro -d -p 80:80 nginx
```
