title: MySQL安装与启动Windows篇
author: 雪落无痕
tags: []
categories:
  - note
date: 2017-12-11 15:41:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fy2ullriuhj20dv07z3yl.jpg)

有时候我们会要在windows机器上安装mysql数据库，又不想下载安装文件，想通过源码安装，此文专门介绍了如果通过下载mysql.zip包，在windows环境下解压安装的过程。

## 0x00 将下载的压缩包解压到自定义目录

官网下载文件,以**mysql-5.6.28-winx64.zip**为例

解压到一个单独目录: 如 `D:\mysql\mysql-5.6.28-winx64`

## 0x01 添加环境变量

变量名：MYSQL_HOME	
变量值：D:\mysql\mysql-5.6.28-winx64

PATH中新增 %MYSQL_HOME%\bin

## 0x02 将mysql注册为windows系统服务

①从控制台进入到MySQL解压目录下的 bin 目录下

②输入命令：mysqld install MySQL --defaults-file="D:\mysql\mysql-5.6.28-winx64\my-default.ini"

备注: 服务移除命令为 mysqld remove

## 0x03 启动mysql服务

方法一: 输入命令 net start mysql	
方法二: 开始->运行->输入services.msc  启动mysql服务

<!--more-->

## 0x04 安装以后需要注意的事情

### 修改root密码

```
c:>mysql –uroot
mysql>show databases;
mysql>use mysql;
mysql>UPDATE user SET password=PASSWORD("123456") WHERE user='root';
mysql>FLUSH PRIVILEGES;
mysql>QUIT
```

### 修改数据库字符集为utf-8

`show variables like '%char%'`

修改目录下的**my-default.ini**文件，增加这一段配置：

```sh
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
character-set-server=utf8
```

保存，重启mysql。

```
>net  stop mysql
>net  start mysql
```

### 让root用户远程能连mysql

```
>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'admin123' WITH GRANT OPTION;

>FLUSH PRIVILEGES;
```

(如果要指定某个ip则%换成ip，root为用户名，admin123为访问密码)


### 创建用户并分配权限

```
>CREATE USER dev IDENTIFIED BY 'dev123'

>SHOW GRANTS FOR dev

>GRANT ALL PRIVILEGES ON test.* TO dev

>REVOKE ALL PRIVILEGES ON test.* FROM dev
```

可能会碰到的事情:

> 为什么使用了Grant all on db.* to user identified by "pass"后，在主机上访问数据库还会出现ERROR 1045 (28000): Access denied for user 'user'@'localhost' (using password: YES) 的错误提示？

> 解答方法如下：运行命令 Grant all on db.* to 'user'@'localhost' identified by "pass"

> 原因是：当不加@选项时，效果与加@'%'是一样的，'%'从名义上包括任何主机，（%必须加上引号，不然与@放在一起可能不会被辨认出。）不过有些时候（有些版本）'%'不包括localhost，要单独对@'localhost'进行赋值。
