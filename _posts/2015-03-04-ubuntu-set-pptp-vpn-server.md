---
layout: post
title: Ubuntu下搭建PPTP VPN服务器
date: 2015-03-04 11:25:19
tags: Linux
categories: tech
---

在Ubuntu Linux下搭建PPTP VPN服务器过程很简单。主要用到了pptpd程序。
可以参照[Ubuntu Community Help Wiki](https://help.ubuntu.com/community/PPTPServer)。

### 1.设置PPTP服务器

安装pptpd

```bash
sudo apt-get install pptpd
```

设置pptpd

```bash
sudo vi /etc/pptpd.conf
```

将服务器IP地址和客户端IP地址范围写到文件末尾。可以如下设置：

```txt
localip 192.168.1.1
remoteip 192.168.1.100-255
```

设置DNS服务器

```bash
sudo vi /etc/ppp/pptpd-options
```

去掉ms-dns的注释，可以设置其为：

```txt
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

添加VPN用户到/etc/ppp/chap-secrets文件：

```bash
sudo vi /etc/ppp/chap-secrets
```

第一列是用户名，第二列是服务器名（可以设置为"pptpd"），第三列是密码，最后一列是IP地址（可以设置为*，允许所有IP）。

重启服务

```bash
/etc/init.d/pptpd restart
```

### 2.设置IP Forwarding

修改/etc/sysctl.conf文件，添加规则：

```bash
sudo vi /etc/sysctl.conf
```

去掉以下行的注释：

```txt
net.ipv4.ip_forward=1
```

重载配置：

```bash
sudo sysctl -p
```

安装iptables并设置

```bash
apt-get install iptables
iptables -t nat -I POSTROUTING -j MASQUERADE
```

重启服务

```bash
/etc/init.d/pptpd restart
```

配置结束
