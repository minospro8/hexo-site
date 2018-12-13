title: linux机器之间ssh免密登录
author: 雪落无痕
tags: []
categories:
  - note
date: 2018-11-11 20:01:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fxqc9lhvzvj20p00fk0sk.jpg)

这里我们假设有三台机器A，B和C。情况如下所示

|机器|ip地址|ssh端口|
|----|------|-------|
|A  |192.168.A.AA|22|
|B	|192.168.B.BB|22|
|C	|192.168.C.CC|66|

这里分别讨论两种情况：

1.如何实现在A机器上免密登录B机器(ssh和scp都不需要输入密码)	
2.如何实现在A机器上免密登录C机器(ssh和scp都不需要输入密码)	

备注:B和C的不同之处就是B是默认22端口，C则是修改以后的端口为66。

## 实现从A到B免密登录

登录A服务器，执行


```shell
$ cd ~
$ ssh-keygen -t rsa
```

以上生成了密钥对，接下来执行


```shell
$ ssh-copy-id 192.168.BB.BBB
```


这样就把A中的公钥信息拷贝到了B机器，完整的命令格式应该是这样的(__不过上面的简化命令也是同样的效果__)。`$ ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.B.BB`


如此一来，__在A机器上__执行下面的命令则不需要输入密码

```shell
$ ssh root@192.168.B.BB	#登录到B服务器
$ scp -r /root/文件夹/ root@192.168.B.BB:/root/	#拷贝A机器上/root/文件夹,整个到B机器的/root目录
```

<!--more-->

## 实现从A到C免密登录

登录A服务器，执行(如果已经生成过密钥对则这步可以省略)

```
$ cd ~
$ ssh-keygen -t rsa
```

以上生成了密钥对，接下来执行类似与上面的ssh-copy-id命令，只不过需要额外写上C服务器的66端口，但是这里我没有成功根据网上教程，__因此不选择这样操作__，但是还是贴一下命令，如下


```
$ ssh-copy-id -i /root/.ssh/id_rsa.pub "-p 66 root@192.168.C.CC"
```

__这里用另一种方法实现__，其实本质的思想是把A服务上的公钥信息拷贝到C服务器上让其知晓，因此可以按照如下方式操作

先把A服务器上的公钥文件拷贝到C服务器(这里要输入C服务器的密码，因为我们还没完成免密)


```
$ scp -P 66 /root/.ssh/id_rsa.pub root@192.168.C.CC:/root/	#把公钥文件拷贝到C服务器的/root目录
```


__然后登录C服务器__，执行如下命令


```
$ cd /root	#进入/root目录
$ mkdir .ssh	#创建.ssh目录，如果已经存在则不需要创建
$ cat /root/id_rsa.pub >> /root/.ssh/authorized_keys	#把公钥信息追加到authorized_keys文件中，这里如果authorized_keys文件不存在也没关系，执行命令会自动创建该文件
```

ok,回到A服务器，试着执行以下命令则不需要输入密码


```
$ ssh -p 66 root@192.168.C.CC	#登录到C服务器
$ scp -P 66 -r /root/文件夹/ root@192.168.C.CC:/root/	#拷贝A机器上/root/文件夹,整个到C机器的/root目录
```

{% colorquote info %}这里注意ssh中的-p是小写，scp中的-P是大写{% endcolorquote %}

## 补充

如果想在A服务器上免密登录B服务器，执行相关代码，而又不需要在命令行切到B服务器，则可以这样操作


```
$ ssh root@192.168.B.BB "rm -rf /var/www/"	#删除远程B服务器上的/var/www/目录
```

这个技巧在A服务器上写shell脚本需要操作远程其他服务的时候比较有用。