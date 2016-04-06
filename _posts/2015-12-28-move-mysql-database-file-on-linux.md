---
layout: post
title: Linux下更改数据库文件路径
date: 2015-12-28 22:15:20
tags: Linux MySQL
categories: tech
---

Linux下经常遇到要更改数据库路径的情况，以下的操作都是在Ubuntu下完成的。

### 1.停止mysql服务

``` bash
sudo service mysql stop
```

### 2.修改my.cnf文件，将datadir改为目标路径

``` bash
sudo vim /etc/mysql/my.cnf
```

### 3.将原来的datadir路径下的所有文件拷贝到新的datadir下

``` bash
cp -R (src_path) (dst_path)
```

### 4.在文件 /etc/apparmor.d/tunables/alias下添加以下的一行内容

``` bash
alias /var/lib/mysql/ -> /newpath/,
```

不要忘记末尾的逗号

### 5.重启apparmor

``` bash
sudo /etc/init.d/apparmor reload
```

### 6.重启mysql服务

``` bash
sudo service mysql start
```
