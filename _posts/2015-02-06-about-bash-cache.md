---
layout: post
title: 关于linux bash的缓存
date: 2015-02-06 17:10:59
tags: Linux bash
categories: tech
---

在Linux下管理软件包时，经常会遇到莫名其妙的问题。
在Ubuntu下安装Scrapy时，第一次直接 `apt-get install scrapy` ，结果有些依赖问题，查询scrapy官网后，remove了Ubuntu自带源中的scrapy，重新从scrapy官网上下载package进行安装。结果运行的时候报错：

``` bash
bash: /usr/local/bin/scrapy: No such file or directory
```

用 `whereis` 命令查找scrapy，发现在/usr/bin目录下。系统的PATH完全没有问题。

google很久之后才知道是bash的缓存的问题。

Linux中,更改存在于PATH目录中的程序位置之后，很可能会出现No such file or directory的错误提示。
比如在这里，第一次安装scrapy后，scrapy在PATH的/usr/local/bin目录下，remove之后重装，scrapy被安装到/usr/bin目录下，按照正常逻辑，此时直接执行scrapy的话，会找到/usr/bin/scrapy,因为/usr/bin也在环境变量PATH中，但却会出现bash: /usr/local/bin/python: No such file or directory 。

具体原因如下：

bash会保存执行时查找过PATH的程序的完整路径，这样下次就不需要再进行查找，如果在上次执行scrapy命令之后更改了scrapy的实际位置，则bash还会去按照原有hash table里面记录的路径去执行，但因为原执行文件已不存，所以会报出No such file or directory的错误提示。

通过执行 hash 命令可以查看bash缓存：

``` bash
wliu@ubuntu:~$ hash
hits    command
   3    /usr/bin/which
   2    /usr/bin/file
   4    /usr/bin/sudo
   1    /bin/mv
   1    /usr/bin/whereis
   4    /usr/local/bin/scrapy
wsliu@ubuntu:~$
```

解决方法：

重置bash 的hash table即可，执行 `hash -d scrapy` 即可删除hash table中scrapy的记录，再次执行scrapy时，bash将搜索$PATH得到新的路径。

以下为Linux man page上的说明：

Bash uses a hash table to remember the full pathnames of executable files (see hash under SHELL BUILTIN COMMANDS below). A full search of the directories in PATH is performed only if the command is not found in the hash table. If the search is unsuccessful, the shell searches for a defined shell function named command_not_found_handle. If that function exists, it is invoked with the original command and the original command's arguments as its arguments, and the function's exit status becomes the exit status of the shell. If that function is not defined, the shell prints an error message and returns an exit status of 127.

hash命令：

hash [-lr] [-p filename] [-dt] [name]

For each name, the full file name of the command is determined by searching the directories in $PATH and remembered. If the -p option is supplied, no path search is performed, and filename is used as the full file name of the command. The -r option causes the shell to forget all remembered locations. The -d option causes the shell to forget the remembered location of each name. If the -t option is supplied, the full pathname to which each name corresponds is printed. If multiple name arguments are supplied with -t, the name is printed before the hashed full pathname. The -l option causes output to be displayed in a format that may be reused as input. If no arguments are given, or if only -l is supplied, information about remembered commands is printed. The return status is true unless a name is not found or an invalid option is supplied.
