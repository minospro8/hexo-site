title: Centos7防火墙
author: 雪落无痕
tags: []
categories:
  - note
date: 2017-12-11 21:48:00
---
centos从6到7，防火墙由原先的iptables变为了firewalld，不熟悉的话操作就很不习惯，这里记录下firewalld的一些常用操作。


## 0x00 系统命令

```sh
$ systemctl start firewalld.service	#启动
$ systemctl stop firewalld.service	#关闭
$ systemctl enable firewalld.service	#设置开机启动
$ systemctl disable firewalld.service	#关闭开机启动
$ systemctl status firewalld	#查看状态
```

<!--more-->
## 0x01 firewall-cmd 命令


```sh
$ firewall-cmd --state #查看状态 running 表示运行
$ firewall-cmd --reload	#重新加载防火墙(一般用于修改配置后)
$ firewall-cmd --permanent --zone=public --add-port=8080-8081/tcp #开启某个端口 永久
$ firewall-cmd --zone=public --add-port=8080-8081/tcp #开启某个端口 临时
$ firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.0.4/24" service name="http" accept"	#设置某个ip 访问某个服务
$ firewall-cmd --permanent --zone=public --list-services #查看开启的服务
$ firewall-cmd --permanent --zone=public --list-ports #查看开启的端口
$ firewall-cmd --list-all #查看防火墙当前配置
```

设置某个ip 访问某个服务：执行命令的效果相当于在配置文件(/etc/firewalld/zones/public.xml)中加入：

```xml
<rule family="ipv4">
 <source address="192.168.0.4/24"/>
 <service name="http"/>
 <accept/>
</rule>
```

设置某个ip 访问某个服务：执行命令的效果相当于在配置文件(/etc/firewalld/zones/public.xml)中加入：

```xml
<port protocol="tcp" port="8080"/>
```

## 0x02 关闭firewall防火墙，使用iptables防火墙

```sh
1、直接关闭防火墙

$ systemctl stop firewalld.service #停止firewall

$ systemctl disable firewalld.service #禁止firewall开机启动

2、设置 iptables service

$ yum -y install iptables-services

如果要修改防火墙配置，如增加防火墙端口3306

$ vi /etc/sysconfig/iptables

增加规则

-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT

保存退出后

$ systemctl restart iptables.service #重启防火墙使配置生效

$ systemctl enable iptables.service #设置防火墙开机启动
```
最后重启系统使设置生效即可
