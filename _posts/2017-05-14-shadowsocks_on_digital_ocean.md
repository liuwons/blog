---
layout: post
title: 使用Digital Ocean搭建Shadowsocks代理服务器
date: 2017-05-14 23:45:09
categories: code
tags: [Shadowsocks]
---


由于 **GFW** 的限制，国内对一些海外网站的访问受到限制，使用代理是一种常见的突破方式，但是可用的免费代理很少并且服务不稳定，收费代理比较昂贵并且高峰时段偶尔带宽很低甚至不可用。如果希望有高可用并且带宽稳定的代理服务器，使用海外云主机自己搭建是一种比较好的途径。

笔者使用过不同的云主机商(阿里云香港、美国节点；腾讯云香港、美国节点；**Digital Ocean** 美国节点)的云主机，以及不同的代理服务器( **PPTP**、**L2TP**、**OpenVPN**、**Shadowsocks**)。最终从价格、性能、带宽、稳定性方面综合考虑，选择使用 **Digital Ocean** 的海外云主机搭建 **Shadowsocks** 代理服务器。

### 1. Digital Ocean

**Digital Ocean** 最低配置的主机只需要每月5美元，该机型配置为：512M内存，20G SSD硬盘容量， 1TB的网络流量(实际上目前不对超标流量收费)。与国内的云主机相比，具有以下优势：1.价格更低 2.不限制带宽 3.硬盘为SSD 。使用此配置的云主机搭建一个代理服务器完全够用了，实际上还可以搭建一些其他服务，例如搭建web服务器、git服务器等。

另外新用户使用推荐链接可以立即得到10美元的代金券，可以免费使用两个月 `^_^` 。


### 2. 创建Digital Ocean主机实例

1. 注册账户

    从我的推荐链接进入注册可以得到10美元： [我的推荐链接](http://www.digitalocean.com/?refcode=c9b4d15f5243)

2. 创建 **Droplet** 实例

    **Droplet** 就是云主机实例。这里可以选择自己熟悉的 **Linux** 发行版版本。我比较熟悉 **Ubuntu** ，这里以 **Ubuntu** 为例。

3. 登陆云主机

4. 安装shadowsocks

    ```
sudo apt-get update
sudo apt-get install python-pip python-m2crypto
sudo pip install shadowsocks
```

5. 启动shadowsocks

    ```
    sudo ssserver -p 5656 -k mypassword -m rc4-md5 --user nobody -d start
    ```
    `-k` 是设置密码的， `-p` 是设置端口的， `-m` 是设置加密算法的。

这样 **Shadowsocks** 就安装完成并启动了。接下来可以在自己电脑上使用 **Shadowsocks** 客户端连接代理了。

### 3. 使用 **Shadowsocks** 客户端

1. 下载客户端

    在此链接上下载对应版本的客户端: [Shadowsocks下载链接](https://shadowsocks.org/en/download/clients.html)

    Windows版本下载地址: [Shadowsocks Windows客户端](https://github.com/shadowsocks/shadowsocks-windows/releases)

2. 运行客户端

    直接运行客户端，然后在图形界面中添加刚刚配置过的服务器，注意ip地址、端口、加密算法、密码一定要配置正确。

3. 启动代理

    直接在图形界面上勾选上 `启用系统代理` 即可
