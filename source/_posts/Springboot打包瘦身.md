---
title: Springboot打包瘦身
date: 2020-12-10 19:11:15
author: zuo_ji
categories:
    - 编程技术
tags: 
  - java
  - springboot
---

**Springboot** 默认的打包方式会将所有的依赖都打到jar包里，导致打包以后的文件体积巨大，少则几十M，多则上百M，内网网络环境当中还不怎么影响，但是一旦部署到公网或者云服务器上时，如果网络环境不好，等待将是一个漫长的过程，极大地影响应用的快速部署和测试流程。

有需求就会有解决的办法，其实早有插件做到这件事。[Spring Boot Thin Launcher](https://github.com/dsyer/spring-boot-thin-launcher)， 你值得拥有。

<!-- more -->

使用插件以后，原来打包45M的jar包，现在体积只有 `40K`，瘦身效果非常明显，并且也不改变原有的打包方式和部署方法。打包以后的jar也是可执行的。

第一次执行时，会自动下载所有的依赖包，并且缓存到本地，默认的缓存路径是`${user.home}/.m2/`，你也可以使用如下命令在启动时修改依赖的路径
```sh
java -Dthin.root=.  -jar  app-0.0.1-SNAPSHOT.jar
```
它将会下载所有的依赖包到你当前目录的`repository` 文件夹下。

#### 使用方法：
在pom.xml添加以下插件代码，然后正常地打包即可
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <dependencies>
                <dependency>
                    <groupId>org.springframework.boot.experimental</groupId>
                    <artifactId>spring-boot-thin-layout</artifactId>
                    <version>1.0.3.RELEASE</version>
                </dependency>
            </dependencies>
            <configuration>
                <executable>true</executable>
            </configuration>
        </plugin>
    </plugins>
</build>
```

## 参考资料

- [Spring Boot Thin Launcher](https://github.com/dsyer/spring-boot-thin-launcher).
