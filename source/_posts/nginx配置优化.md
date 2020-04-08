---
title: Nginx 配置优化
date: 2020-04-03 09:48:38
categories:
- 笔记
tags:
- nginx
---

nginx.conf 配置优化

<!-- more -->

## nginx.conf基本配置

```bash
#定义Nginx运行的用户和用户组
user  root;
#nginx进程数，建议设置为等于CPU总核心数。
worker_processes  1;
#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#进程文件
pid        logs/nginx.pid;

#工作模式与连接数上限
events {
#参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，
#就用kqueue模型。
    use epoll;  
　　#单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections  65535;
}

#设定http服务器
http {
    include       mime.types;#文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型

    # 日志格式化
		log_format main '$host - $remote_addr - [$time_local] "$request" '
                '$status $upstream_response_time $request_time "$http_referer"'
                '"$http_user_agent" "$http_x_forwarded_for" $body_bytes_sent ';
		access_log /var/log/nginx/access.log main;


    sendfile        off;
    tcp_nopush     on;#防止网络阻塞
    tcp_nodelay on; #防止网络阻塞

    #keepalive_timeout  0;
    keepalive_timeout  65;#长连接超时时间，单位是秒


    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k; #最小压缩文件大小
    gzip_buffers 4 16k; #压缩缓冲区
    #gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2; #压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml;
    #压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;
    #limit_zone crawler $binary_remote_addr 10m; #开启限制IP连接数的时候需要使用
 

    server {
        #监听的端口
        listen       80;
        server_name  localhost;
        #编码格式
        charset utf-8;

        #access_log  logs/host.access.log  main;
   
        location /
       {
        proxy_pass http://127.0.0.1;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #以下是一些反向代理的配置，可选。
        proxy_set_header Host $host;
        client_max_body_size 10m; #允许客户端请求的最大单文件字节数
        client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数，
        proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
        proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
        proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
        proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
        proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的设置
        proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
        proxy_temp_file_write_size 64k;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```

## 文件服务器配置

```bash
server {
        listen  80;
        server_name  localhost;
        charset utf-8;
        #root 指令用来指定文件在服务器上的基路径
        root /data/statics;
        #location指令用来映射请求到本地文件系统
        location / {
           autoindex on; # 索引
           autoindex_exact_size on; # 显示文件大小
           autoindex_localtime on; # 显示文件时间
					 # 设置密码
					 auth_basic "请输入用户名密码";
           auth_basic_user_file /usr/local/nginx/passwd.db;
        }
   }
```
创建用户文件
```bash
$ htpasswd -c passwd.db
```
新增用户
```bash
$ htpasswd -b passwd.db admin 123456
```