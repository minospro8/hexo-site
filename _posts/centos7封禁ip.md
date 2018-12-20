title: centos7封禁ip
author: 雪落无痕
tags: []
categories:
  - note
date: 2018-12-20 17:26:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fydc2k6q0mj21hc0u0dh1.jpg)

## 0x00 前言

centos 7中使用的是firewalld防火墙，一般情况下可以用如下命令来封禁某一个ip或ip段

```
$ firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='xx.xx.xxx.x' reject"	#单个ip	
$ firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='xx.xx.xxx.0/24' reject"	#ip段
```
使用上是没什么问题，只不过一旦大量ip需要拉黑，管理上就会很混乱。因此这里介绍firewalld配置ipset来实现。

## 0x01 创建ipset

执行命令

```
$ firewall-cmd --permanent --zone=public --new-ipset=blacklist --type=hash:net	#创建一个名为blacklist的库
```

执行完命令可在`/etc/firewalld/ipsets`路径下看到生成的blacklist.xml文件。

## 0x02 添加/删除要禁止的ip或ip段


```
$ firewall-cmd --permanent --zone=public --ipset=blacklist --add-entry=xxx.x.x.xx	#添加ip
$ firewall-cmd --permanent --zone=public --ipset=blacklist --add-entry=xxx.xx.xx.0/24	#添加ip段
$ firewall-cmd --permanent --zone=public --ipset=blacklist --remove-entry=xxx.x.x.xx	#删除ip
$ firewall-cmd --permanent --zone=public --ipset=blacklist --remove-entry=xxx.xx.xx.0/24	#删除ip段
```

备注: --permanent参数表示永久生效，内容会写入blacklist.xml文件且需要重启防火墙`firewall-cmd --reload`生效，如果不加该参数，则立即生效，内容不会写入blacklist.xml文件，服务重启则规则失效。

## 0x03 封禁ipset

前面只是创建了名为blacklist的ipset，而且也往里面增加了内容，最后一步即防火墙封禁该ipset即可，这样的好处是防火墙只是一条规则就封禁了文件里管理的大量ip。

```
$ firewall-cmd --permanent --zone=public --add-rich-rule='rule source ipset=blacklist drop'	#封禁名为blacklist的ipset
```

最后，重新载入防火墙即可生效：`firewall-cmd --reload`

可通过命令查看当前防火墙状态：`firewall-cmd --list-all`

也可以单独查看当前ipset封禁了哪些ip：`ipset list blacklist`


