title: vps初始化需做的事情
author: 雪落无痕
tags: []
categories:
  - note
date: 2018-11-13 15:57:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fxfqi1sijxj20m807jmzf.jpg)

## 0x00 修改root密码

```
$ passwd root
```
## 0x01 关闭SELinux

1.查看SELinux状态

```
$ /usr/sbin/sestatus -v 
```
如果显示`SELinux status: enabled`则为开启状态，`SELinux status: disabled`则为关闭状态。

也可以用如下命令检查:

```
$ getenforce
```

2.关闭SELinux

①临时关闭(不用重启)

```
$ setenforce 0 #setenforce 0 设置SELinux 成为permissive模式(关闭)
```
备注:setenforce 1 设置SELinux 成为enforcing模式(开启)

②永久关闭需修改配置文件(需重启机器)

```
$ vi /etc/selinux/config
```
将`SELINUX=enforcing`改为`SELINUX=disabled`

<!--more-->

## 0x02 更改ssh默认22端口

{% colorquote info %}
这里需要修改配置文件，但是在此之前需要先确保你要更改的端口(这里假定要修改为66端口)是被防火墙放行的，针对centos 7这里是firewalld而非iptables。
{% endcolorquote %}

```
$ systemctl status firewalld	#查看防火墙状态，如果是关闭的也就无所谓加不加端口了
$ firewall-cmd --list-ports	#查看当前防火墙开放了哪些端口
$ firewall-cmd --zone=public --add-port=66/tcp --permanent	#开放66端口,tcp
$ firewall-cmd --zone=public --add-port=66/udp --permanent	#开放66端口,udp
$ firewall-cmd --reload	#重启防火墙
$ firewall-cmd --zone=public --list-all	#查看重启之后的防火墙状态,也可以用上面查看开放端口的命令
```
端口都搞定之后，接下来就是修改ssh配置文件。
```
$ vi /etc/ssh/sshd_config
```
在 #Port 22 这一行下面 加入 Port 66 然后别忘了防火墙开放66端口(上面已经做过了)。

保存之后重启sshd服务
```
$ service sshd restart
```

## 0x03 一键增加删除防火墙端口

鉴于上面更改ssh默认端口的时候，添加过防火墙端口，但是很繁琐，这里提供一键脚本，出处为[github](https://github.com/yanzi1225627/greennet/blob/master/shadowsocks/myport),这里是[原创博文链接](https://blog.csdn.net/yanzi1225627/article/details/51470962)

新建文件myport，注意没有后缀。复制以下内容到文件

```
#!/bin/bash
num=$#
ok=0
if [ ${num} != 2 ]; then
    echo 'error:you must input two parmas, first is add or remote, second is port number'
    exit 0
fi

case $1 in

add)
firewall-cmd --zone=public --add-port=$2/tcp --permanent
firewall-cmd --zone=public --add-port=$2/udp --permanent
ok=1
;;

remove)
firewall-cmd --zone=public --remove-port=$2/tcp --permanent
firewall-cmd --zone=public --remove-port=$2/udp --permanent
ok=1
;;

*)
echo 'first params must be "add" or "remove"' 
;;

esac
if [ ${ok} == 1 ]; then
firewall-cmd --reload
firewall-cmd --zone=public --list-all
fi
exit 0
```
将脚本增加可执行权限，然后mv到`/usr/local/sbin`目录即可！

使用示例

```
$ myport add 80	#开放80端口,包含tcp和udp
$ myport remove 80	#移除开放的80端口,包含tcp和udp
```

## 0x04 测试上传下载速度

这里使用speedtest-cli脚本工具,出处为[github](https://github.com/sivel/speedtest-cli)。

获取脚本

```
$ wget -O speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
$ chmod +x speedtest-cli
```
也可以用如下命令

```
$ curl -Lo speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
$ chmod +x speedtest-cli
```
使用示例

```
$ ./speedtest-cli	#程序自动找ping最低的节点测试
$ ./speedtest-cli --list | grep China	#列出所有China相关的节点，前面的编号代表这个节点
$ ./speedtest-cli --server=2529	#测试vps到节点2529的网络情况
```
注意，优先关注upload速度。

## 0x05 测试网络下行和IO速度

执行命令

```
$ wget -qO- bench.sh | bash
```
该命令不会保存文件到本地，直接看执行结果就可以了。

## 0x06 查看网络回程路由信息

这里用到besttrace软件。

{% pullquote %} 一款由 IPIP.NET 开发的路由追踪检测工具。目前已经涵盖了 Windows、Android、iOS、Mac平台。Best Trace 可对服务器所经过的路由进行可视化检测，并可以准确标注路由所在位置以及 AS 号。支持查询本机 IP、路由监控、批量 Ping 等。一款由 IPIP.NET 开发的路由追踪检测工具。目前已经涵盖了 Windows、Android、iOS、Mac平台。Best Trace 可对服务器所经过的路由进行可视化检测，并可以准确标注路由所在位置以及 AS 号。支持查询本机 IP、路由监控、批量 Ping 等。{% endpullquote %}

```
$ wget https://cdn.ipip.net/17mon/besttrace4linux.zip   
$ unzip besttrace4linux.zip   
$ chmod +x besttrace   
$ ./besttrace -q 1 IP               //IP为你所在的地区的网关或你的公网IP
```

## 0x07 一些常用工具安装(个人向)

```
$ yum install vim net-tools zip unzip lrzsz wget bind-utils -y
```

vim: vi命令增强

net-tools:网络工具包

zip: zip压缩

unzip: zip解压

lrzsz: 上传下载工具包

wget: 网络下载

bind-utils: 域名查询

## 0x08 修改系统最大文件链接数

```
$ ulimit -a
```
主要关注open files这一行

```
$ vi  /etc/systemd/system.conf
```

找到`#DefaultLimitNOFILE=`这一行修改为`DefaultLimitNOFILE=65535`

保存后再查看

```
$ ulimit -a
```
