---
layout: post
title: 使用nginx搭建日志服务器
date: 2021-01-07 14:16:55
categories:
    - 编程技术
tags:
    - nginx
    - 日志
thumbnail: https://i.loli.net/2021/01/07/ndV2skv9QUWKq1A.png
---

网站在运行时会产生很多日志文件，需要查询日志一般需要登录到服务器上，再down下来进行搜索检索，非常不方便，业内有一些开源的方案可以将日志文件进行聚合，然后导入到`elasticsearch`当中，再用kibana进行前端日志展示与检索．即所谓的ELK(elasticsearch+Logstash+Kibana)．

这个方案当然好是肯定好，logstash可以聚合多个应用的日志文件，同时借助elasticsearch的搜索引擎能力，可以快速地通过关键字查询日志，进行日志分析．

但是对于一个小型应用来说，这么做未免就有点大动干戈了，对于日志量不大的单体应用来说，我们可以将日志目录用ngnix来挂载到一个contextpath上，这样就可以在线对日志进行一些简单的查询，此即是本文要讨论的问题．
<!-- more -->

## 基本需求
假设我们有一个站点`http://www.foobar.com`,他的日志存在`/home/ubuntu/log/`这个目录下面，当我们访问`http://www.foobar.com/log` 时就可以看到日志的内容

1. 日志按日期分成多个文件，需要列出文件列表
```sh
    ll /home/ubuntu/log
    total 19M
    -rw-r--r-- 1 ubuntu ubuntu 470K Dec 22 23:55 2020-12-22.log
    -rw-r--r-- 1 ubuntu ubuntu 644K Dec 23 23:45 2020-12-23.log
    -rw-r--r-- 1 ubuntu ubuntu 950K Dec 24 23:45 2020-12-24.log
    -rw-r--r-- 1 ubuntu ubuntu 260K Dec 25 23:45 2020-12-25.log
```
2. 当访问`http://www.foobar.com/log`需要输入用户名密码
3. 可以直接在网页显示日志内容，而不是点击下载
4. 文件编码为`utf-8`

## 具体实现
先来看一眼完整的配置文件
```nginx
server {
    listen 80 ;
    server_name www.foobar.com;
    add_header Content-Security-Policy 'upgrade-insecure-requests';
    proxy_set_header HOST $HOST;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Request-Url $request_uri;

    location /log {
        autoindex on;
        charset utf-8;
        auth_basic           "Administrator's Area";
        auth_basic_user_file /etc/apache2/.htpasswd; 	
        root /home/ubuntu/log;
        rewrite "^/log/(.*)$" /$1 break;
        
        types {
            text/plain log;
        }
    } 
    ...
}

```

**说明：**
1. `autoindex on;` 这行是为了列出文件目录
2. `types`是为了告诉nginx哪些后缀文件可以作为`text/plain`返回
    ```
    types {
        text/plain log;
    }
    ```
3. 为了保护文件内容，加了auth_basic认证，用以下命令生成用户名和密码
    ```sh
    sudo htpasswd -c /etc/apache2/.htpasswd username
    ```
    然后在nginx里配置如下信息就可以了．
    ```nginx
    auth_basic           "Administrator's Area";
    auth_basic_user_file /etc/apache2/.htpasswd; 
    ```


## 效果

浏览器输入http://www.foobar.com/log以后，会弹出如下对话框，提示输入用户名和密码
![picture 7](https://i.loli.net/2021/01/07/ndV2skv9QUWKq1A.png)  
输入完以后，就会显示如下的文件列表
![picture 8](https://i.loli.net/2021/01/07/MsqSrOPNtXV1JWp.png)  
点击单个链接就可以直接查看具体的日志文件内容了，而不需要再下载．