title: vps安装速锐
author: 雪落无痕
tags: []
categories:
  - note
date: 2018-11-30 16:28:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fxq639ivi2j21400seq4t.jpg)

参考网上这篇文章[链接](https://blog.deartanker.com/post/4486.html)

## 0x00 更新 CentOS7 到最新版

```
$ yum install epel-release -y
$ yum update -y
```

## 0x01 同步时间系统时间

```
$ yum install bind-utils net-tools wget ntp -y	#同步时间主要是ntp软件
$ timedatectl set-timezone Asia/Shanghai	#指定东八区
$ timedatectl set-ntp yes	#同步
$ timedatectl	#查看
$ date -R	#查看当前时间和时区
```

重启系统

```
$ systemctl reboot
```
<!--more-->
## 0x02 安装速锐支持内核

CentOS7 强制安装支持锐速的最新内核，并永久指定使用该内核启动系统。

`uname -r`可以查看当前系统内核。

强制更新内核到：kernel-3.10.0-327.36.3.el7.x86_64

```
$ rpm -ivh http://vault.centos.org/7.2.1511/updates/x86_64/Packages/kernel-3.10.0-327.36.3.el7.x86_64.rpm --force
```

永久指定使用该内核启动

```
$ grub2-set-default "CentOS Linux (3.10.0-327.36.3.el7.x86_64) 7 (Core)"
```

检查指定内核是否生效

```
$ grub2-editenv list
```

重启系统

```
$ systemctl reboot
```

再次检查当前运行的内核

```
$ uname -r
```

## 0x03 安装速锐

```
$ wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder-v.sh && bash serverspeeder-v.sh CentOS 7.2 3.10.0-327.el7.x86_64 x64 3.11.20.5 serverspeeder_72327
```

成功之后查看速锐运行情况(速锐默认开机启动)

```
$ /serverspeeder/bin/serverSpeeder.sh status
```

也可以执行以下命令查看

```
$ service serverSpeeder status
```

如果想关闭开机自启，则执行下面命令

```
$ chkconfig serverSpeeder off
```

## 0x04 补充
简单说明下 centos 6 与 centos 7 服务开机启动、关闭设置的方法：

{% colorquote info %}centos 6,使用chkconfig命令即可。centos 7,使用systemctl中的enable、disable 即可。{% endcolorquote %}


我们以apache服务为例:

centos 6

```
$ chkconfig --add apache	#添加nginx服务
$ chkconfig apache on	#开机自启nginx服务
$ chkconfig apache off	#关闭开机自启
$ chkconfig --list | grep apache	#查看
```

centos 7

```
$ systemctl enable apache.service	#开机自启apache服务
$ systemctl disable apache.service	#关闭开机自启
```