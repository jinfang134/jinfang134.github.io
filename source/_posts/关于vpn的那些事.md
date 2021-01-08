---
title: 关于vpn的那些事
date: 2020-12-30 19:11:15
author: zuo_ji
categories:
  - Web技术
tags:
  - vpn
  - shadowsocks
thumbnail:
blogexcerpt:
---

在外企上班，公司网络是直接vpn到新加坡的，也经常要用slack等工具进行线上交流，没个梯子还经常挺不方便的。最近抽空折腾了一下，写个文章记录一下过程。

## 服务器
服务器我用的是[virmach](https://billing.virmach.com/index.php?rp=/store/kvm-and-ssd-windows-vps),1.25刀一个月,价格还是挺实惠的，500GB的网络带宽应该是用不完的，至于网络速度，我测试了一下，youtube里的720p视频可以流畅播放。

![](/images/2020-12-31-00-28-50.png)
<!-- more -->
## 服务端
服务端我用的是的[shadowsocks-libv](https://github.com/shadowsocks/shadowsocks-libev),为了与系统环境进行隔离，方便进行迁移，我用的是docker的方式来进行部署的，在官方hub里就有相应的[docker镜像](https://hub.docker.com/r/shadowsocks/shadowsocks-libev)，使用还挺方便的。

如果没有docker的话，得先安装docker
```sh
# install common tools
sudo apt-get update
sudo apt-get install -y vim git curl zsh htop python3-pip

# install docker
curl -ssl https://get.docker.com/ | sh
sudo groupadd docker
sudo usermod -aG docker $USER

# install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

```

- docker直接运行
  ```sh
  docker run -e PASSWORD=<password> -p<server-port>:8388 -p<server-port>:8388/udp -d shadowsocks/shadowsocks-libev
  ```
- docker-compose方式，我更推荐这种方式，更加便于维护和管理，写一个`docker-compose.yml`,内容如下：
  ```yaml
  version: '2'
  services:
    shadowsocks1:
      image: shadowsocks/shadowsocks-libev
      container_name: shadowsocks1
      environment:
        PASSWORD: password
        # METHOD: CHACHA20-IETF-POLY1305
      ports:
        - "8388:8388"
        - "8388:8388/udp"
      # volumes: 
        # - $PWD/config:/etc/shadowsocks
      restart: always
  ```
  然后运行`docker-compose up -d`来启动shadowsocks服务，如果不配置method,默认是使用`aes-256-gcm`方法

## 客户端
Shadowsocks 目前的客户端基本各个平台上面的都有。本文主要介绍在 Windows、macOS、Linux、Android、iOS、OpenWRT 平台上常用的一些客户端。

需要说明的一点是，如果使用`Outline`客户端，需要一个访问密钥，那这个访问密钥是怎么生成呢？

规则如下：
```js
const method='aes-256-gcm'
const password='password'
const host='107.174.250.123'
const port='8388'
const name='myss'
console.log( "ss://" + btoa(`${method}:${password}@${host}:${port}`)+ '#' + name )
```
例如：
```properties
method: rc4-md5
password: test123
host: 283.234.123.21
port: 4000
```
拼接字符串得到：`rc4-md5:test123@283.234.123.21:4000`
进行`base64`编码以后，得到：
```
cmM0LW1kNTp0ZXN0MTIzQDI4My4yMzQuMTIzLjIxOjQwMDA=
```
最后访问密钥如下：
```
ss://cmM0LW1kNTp0ZXN0MTIzQDI4My4yMzQuMTIzLjIxOjQwMDA=#shadowsock
```
更新：　发现原来还有一个[官网](https://shadowsocks.org/en/config/quick-guide.html)，里面可以自动生成这个链接．
![picture 1](https://i.loli.net/2021/01/06/t2FZVpMEwTChcUb.png)  


### Windows

**Shadowsocks Windows**: [GitHub](https://github.com/shadowsocks/shadowsocks-windows/releases)

在 Windows 平台推荐使用，由官方出品，需要安装 [.NET Framework](https://www.microsoft.com/net/download/dotnet-framework-runtime)，请尽量使用最新版本。使用教程请参考：[在 Windows 中配置 Shadowsocks 客户端](https://www.vpnto.net/posts/windows-shadowsocks/)

**Outline Windows**: [GitHub](https://github.com/Jigsaw-Code/outline-client/)/[Direct Download](https://raw.githubusercontent.com/Jigsaw-Code/outline-releases/master/client/Outline-Client.exe)

[Outline](https://getoutline.org/) 由 [Google Jigsaw](https://jigsaw.google.com/) 出品，功能上比较简单，必须使用秘钥进行连接。

### macOS

**ShadowsocksX-NG**: [GitHub](https://github.com/shadowsocks/ShadowsocksX-NG/releases)

新一代的 ShadowsocksX，底层是基于 Shadowsocks-libev 的 ss-local 客户端。使用教程请参考：[在 macOS 中配置 Shadowsocks 客户端](https://vpnto.net/posts/macos-shadowsocks/)

**Outline macOS**: [GitHub](https://github.com/Jigsaw-Code/outline-client/)/[App Store](https://itunes.apple.com/app/outline-app/id1356178125)

[Outline](https://getoutline.org/) 由 [Google Jigsaw](https://jigsaw.google.com/) 出品，功能上比较简单，必须使用秘钥进行连接。

### Linux

**Shadowsocks Qt5**: [GitHub](https://github.com/shadowsocks/shadowsocks-qt5/wiki/Installation)

Shadowsocks 在 Linux 上唯一的 GUI，请按照 [Wiki 安装指南](https://github.com/shadowsocks/shadowsocks-qt5/wiki/Installation)进行安装, 如果你喜欢命令行的请使用 Shadowsocks-libev 客户端。使用教程请参考：[在 Linux 中配置 Shadowsocks 客户端](https://vpnto.net/posts/linux-shadowsocks/)

### Android

如果无法访问 Google play 请下载 apk 安装。

**Shadowsocks Android**: [GitHub](https://github.com/shadowsocks/shadowsocks-android/releases)/[Google Play](https://play.google.com/store/apps/details?id=com.github.shadowsocks) ([beta](https://play.google.com/apps/testing/com.github.shadowsocks))

Shadowsocks 官方开发的 Android 版本，功能强大，推荐使用。使用教程请参考：[在 Android 中使用 Shadowsocks 客户端](https://vpnto.net/posts/android-shadowsocks/)

**Outline Android**: [GitHub](https://github.com/Jigsaw-Code/outline-client/)/[Play Store](https://play.google.com/store/apps/details?id=org.outline.android.client)/[Direct Download](https://github.com/Jigsaw-Code/outline-releases/blob/master/client/Outline.apk?raw=true)

[Outline](https://getoutline.org/) 由 [Google Jigsaw](https://jigsaw.google.com/) 出品，功能上比较简单，必须使用秘钥进行连接。

### iOS

目前在 App Store 上面无法在中国区搜索到，请使用非中国大陆区的账号搜索并购买。

**Outline iOS**: [GitHub](https://github.com/Jigsaw-Code/outline-client/)/[App Store](https://itunes.apple.com/app/outline-app/id1356177741)

[Outline](https://getoutline.org/) 由 [Google Jigsaw](https://jigsaw.google.com/) 出品，功能上比较简单，必须使用秘钥进行连接。可以免费下载。

**Potatso Lite**: [App Store](https://itunes.apple.com/app/potatso-lite/id1239860606)

[Potatso](https://potatso.com/) 的简版，不过功能够用，可以免费下载。

**Shadowrocket**: [App Store](https://itunes.apple.com/app/shadowrocket/id932747118)

Shadowrocket 应该属于目前最火的 Shadowsocks iOS 客户端了，功能强大，更新快。性价比非常高，推荐购买使用，目前美区是 $2.99 的价格。使用教程请参考：[在 iOS 中使用 Shadowsocks 客户端](https://vpnto.net/posts/ios-shadowsocks/)

### OpenWrt/LEDE

**Shadowsocks-libev for OpenWrt**: [GitHub](https://github.com/shadowsocks/openwrt-shadowsocks/releases)

**OpenWrt LuCI for Shadowsocks-libev**: [GitHub](https://github.com/shadowsocks/luci-app-shadowsocks/releases)

在 OpenWrt/LEDE 的路由器中使用 Shadowsocks，配合 ChinaDNS 和 DNS-Forwarder 使用更好，在这里查看 [OpenWrt/LEDE Shadowsocks](https://www.vpnto.net/posts/shadowsocks-openwrt/) 安装教程。

## 其它方案

与shadowsocks齐名的还有[v2ray](https://toutyrater.github.io/prep/),部署要复杂得多，可以作为一个备选项.
