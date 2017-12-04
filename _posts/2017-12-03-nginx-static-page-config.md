---
layout: post
title: 在Nginx中配置代理服务
date: 2017-12-03 21:30:00
tags: [Nginx, Html, 代理]
---

有时候我们会使用Nginx来做负载均衡服务器, 该文记录对于配置静态网页界面、后台服务转发服务的配置以及如果需要上下文根对应的时候应该如何配置。

### 一、Nginx用于静态网址配置

	user  nginx nginx;
	worker_processes  8;
	worker_rlimit_nofile 65536;
	#error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;

	#pid        logs/nginx.pid;


	events {
	    worker_connections  65536;
	}


	http {
	    include       mime.types;
	    default_type  application/octet-stream;

	    log_format  main  '$remote_addr [$time_local] $upstream_addr $upstream_status $upstream_response_time "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
	    log_format cachelog '$time_local - $upstream_cache_status - Cache-Control:$upstream_http_cache_control - $request($status) - ';

	    access_log  logs/access.log  main;

	    sendfile        on;
	    tcp_nopush     on;
	    proxy_ignore_client_abort on;

	    keepalive_timeout  65;
	    #keepalive_timeout  1000;
	    charset utf-8;
	    gzip on;
	    gzip_min_length 10k;
	    gzip_buffers 4 16k;
	    gzip_comp_level 2;
	    gzip_types text/plain text/javascript application/javascript application/x-javascript text/css  application/xml application/octet-stream;
	    gzip_vary on;
	    
	    server {
			#被监听的端口号和网址
	        listen       80;
	        server_name  xxxxx;

	        #charset koi8-r;

	        access_log  logs/test_access.log  main;

	        location /functions {
	        	#这个地方指定被访问的文件夹位置, servless目录下面有一个functions目录存放的资源
	            root   /data/servless;
	            index  index.html index.htm;
	        }

	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	        }

	        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {
	        	# 对于一些资源的访问, 因为有时候由于配置的问题, js文件需要访问nginx主机下的, 所以使用该配置, 将资源访问配置正确, 否则会出现404错误造成的界面无法正常显示
	        	root /data/servless/functions;
	        	expires 30d;
	        }
	    }
	}

### 二、对于动态Restful接口的代理配置

	user  nginx nginx;
	worker_processes  8;
	worker_rlimit_nofile 65536;
	#error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;

	#pid        logs/nginx.pid;


	events {
	    worker_connections  65536;
	}


	http {
	    include       mime.types;
	    default_type  application/octet-stream;

	    log_format  main  '$remote_addr [$time_local] $upstream_addr $upstream_status $upstream_response_time "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
	    log_format cachelog '$time_local - $upstream_cache_status - Cache-Control:$upstream_http_cache_control - $request($status) - ';

	    access_log  logs/access.log  main;

	    sendfile        on;
	    tcp_nopush     on;
	    proxy_ignore_client_abort on;

	    keepalive_timeout  65;
	    #keepalive_timeout  1000;
	    charset utf-8;
	    gzip on;
	    gzip_min_length 10k;
	    gzip_buffers 4 16k;
	    gzip_comp_level 2;
	    gzip_types text/plain text/javascript application/javascript application/x-javascript text/css  application/xml application/octet-stream;
	    gzip_vary on;
	    
	    server {
			#被监听的端口号和网址
	        listen       80;
	        server_name  localhost;

	        #charset koi8-r;

	        access_log  logs/test_access.log  main;

	        location /servless {
	        	#该配置只的是 restful的contextPath=servless, 同时需要在外面访问的时候也可以使用servless的上下文根来访问
	            proxy_read_timeout 300;
	            proxy_connect_timeout 300;
	            proxy_redirect off;
	            proxy_set_header X-Forwarded-Proto $scheme;
	            proxy_set_header X-Read-IP         $remote_addr;
	            proxy_pass http://localhost:8080/servless/;
	            client_max_body_size  1000m;
	        }

	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	        }

	        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {
	        	# 对于一些资源的访问, 因为有时候由于配置的问题, js文件需要访问nginx主机下的, 所以使用该配置, 将资源访问配置正确, 否则会出现404错误造成的界面无法正常显示
	        	root /data/servless/functions;
	        	expires 30d;
	        }
	    }
	}
