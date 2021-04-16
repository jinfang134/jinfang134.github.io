---
layout: post
title: 开源软件license管理
date: 2021-04-16 15:39:48
categories:
   - 应用技巧 
tags:
   - license
---
项目当中遇到一些问题，需要排查所有的依赖的license信息，需要将所有的依赖的license信息都列出来，由于依赖的继承和传递关系，依赖的数量非常庞大，因此手工来完成这个任务是不太现实，好在已经有很好的开源工具实现了这个功能．
<!-- more -->

## java项目
修改gradle的配置，安装插件[Gradle-License-Report](https://github.com/jk1/Gradle-License-Report)．
```gradle
plugins {
    id("com.github.jk1.dependency-license-report") version "1.16"
}

licenseReport {
    renderers = arrayOf<ReportRenderer>(InventoryHtmlReportRenderer("report.html","Backend"),com.github.jk1.license.render.CsvReportRenderer())
    filters = arrayOf<DependencyFilter>(LicenseBundleNormalizer())
}

```
这款插件会为你生成你项目所有依赖的license报表信息，可以支持多种报表格式，包括csv,html,pdf等．
安装以后，会添加如下的task
```shell
Reporting tasks
---------------
generateLicenseReport - Generates license report for all dependencies of this project and its subprojects

```
执行`./gradlew generateLicenseReport`以后，就可以在`build/reports/dependency-license`里找到新生成的report了，新生成的report如下图
![license report](https://i.loli.net/2021/04/16/S6PNVCoMnj2tyFL.png)  


## nodejs 
npm里有一个[license-checker](https://www.npmjs.com/package/license-checker)可以很方便地查看每个依赖的license信息，使用前先进行全局安装
```shell
npm install -g license-checker
npm install -g yui-lint

```
然后运行如下的命令就可以生成相应的csv报告，注意在执行前要先运行一下`yarn install`, 确保所有的依赖都已经正确安装了
```shell
license-checker --production --csv > license.csv
```
