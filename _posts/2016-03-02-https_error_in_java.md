---
layout: post
title: Java https服务器证书认证问题解决方案
date: 2016-03-02 16:48:52
categories: code
tags: Java
---

Java https连接的"unable to find valid certification path to requested target","PKIX path building failed"错误

## 问题原因
这个问题的是由于Java自带的根证书库中不包含HTTPS服务器上的根证书，因此无法得到认证。

## 解决方案

比较容易实现的方案有两种：

 1. 导入服务器证书到本地Java环境
 2. 代码中忽略证书信任问题

由于第二种方案会导致安全性问题，因此并不推荐。

## 证书导入注意事项
将服务器的根证书导入到Java运行环境的根证书库中，能解决对应的服务器https连接问题。

实现方式是使用$JAVA_HOME/jre/bin下的keytool工具将服务器端的证书添加到jre/lib/security/cacerts文件中。
需要注意的是：

 1. 确定当前Java程序所用的java运行环境jre的路径（可能为jdk下的jre，也可能是单独的jre）。
 2. 确定有jre/lib/security/cacerts文件的写入权限（可以用管理员权限运行keytool）。


## 详细步骤：

### 1. 获取服务器端的证书文件

可以使用浏览器打开服务器网站页面，然后导出证书文件，假设导出的证书文件为test.crt 。

### 2. 生成keystore文件

利用keytool生成密钥文件keystore：

``` bash
keytool -importcert -noprompt -trustcacerts -alias test -file test.cer -keystore ~/mykeystore
```
这里会要求设置口令，设置后请记住口令 。

### 3. 导入证书到Java运行时环境

将证书导入jre/lib/security/cacerts：

```bash
keytool -importkeystore -srckeystore ~/keystore -destkeystore [path_to_jre]/lib/security/cacerts
```

会要求输入目标密钥库口令（也就是jre/lib/security/cacerts 的口令，默认是changeit或者changeme），以及源密钥库口令（之前设置的口令）.

最后会显示是否导入成功。如果成功则重启Java程序。
