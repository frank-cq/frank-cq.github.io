---
layout: post
title: Ubuntu修改DNS
categories: []
tags: [Code]
comments: true
---



Ubuntu12.04后的系统/etc/resolv.conf文件由程序动态生成，直接修改会被覆盖。要想永久有效，如下

1、修改/etc/resolvconf/resolv.conf.d/base，或者添加文件/etc/resolvconf/resolv.conf.d/tail，
2、在文件中加入：
```
nameserver   <DNS服务器地址>  
```
3、resolvconf -u

若有问题：dns解析文件错误：resolvconf: Error: /etc/resolv.conf isn't a symlink, not doing anything.
解决办法：dpkg-reconfigure resolvconf

<br/>
附上三种方法：

1、打开文件  /etc/resolv.conf ，添加DNS地址（重启后失效）
```
nameserver <DNS服务器地址>
```

2、打开文件 /etc/network/interfaces，添加DNS（重启后生效）
```
dns-nameservers <DNS服务器地址>
```

3、打开文件  /etc/resolvconf/resolv.conf.d/base，添加DNS
```
nameserver <DNS服务器地址>
```
保存，退出，执行

resolvconf -u
