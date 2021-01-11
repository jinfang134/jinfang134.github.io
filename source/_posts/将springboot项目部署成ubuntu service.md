---
layout: java
title: 将springboot项目部署成ubuntu service
date: 2020-12-24 16:58:02
tags:
    - ubuntu
    - springboot
categories:
    - 编程技术
---
在部署springboot项目时，为了运行维护的方便，可将其部署为ubuntu service,并且可随系统启动而自动启动，本文将介绍一种比较简单的方式来实现此功能。

<!-- more -->

1. 在`home` 目录下建一个 `workspace` 目录，目录里放你的springboot打包以后的jar包，如：`demo.jar`
2. 在`/etc/systemd/system/` 下添加一个新文件 `demo.service`
```properties
[Unit]
Description=My Webapp Java REST Service
[Service]
User=ubuntu
# The configuration file application.properties should be here:
#change this to your workspace
WorkingDirectory=/home/ubuntu/workspace
#path to executable. 
#executable is a bash script which calls jar file
# please change your java path
ExecStart=/usr/bin/java -jar /home/ubuntu/workspace/demo.jar
SuccessExitStatus=143
TimeoutStopSec=10
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
```
3. 依次执行以下命令
```sh
sudo systemctl daemon-reload
sudo systemctl enable demo.service
sudo systemctl start demo
sudo systemctl status demo
```
如果一切正常的话，应用已经启动好了，你可以使用以下命令查看日志
```sh
sudo journalctl -f -u demo
```

## 参考
- [Run Your Java App as a Service on Ubuntu](https://dzone.com/articles/run-your-java-application-as-a-service-on-ubuntu)
- [Springboot doc](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment-install.html)