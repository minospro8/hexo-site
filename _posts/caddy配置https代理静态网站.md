title: caddy配置https代理静态网站
author: 雪落无痕
tags: []
categories:
  - note
date: 2018-12-07 15:33:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fxya43euxvj20m80cit8u.jpg)

## 0x00 前言

使用https必须有域名，这里的前提是**已经有域名且域名的A记录已经指向服务器公网ip**。本文使用centos 7环境。

## 0x01 安装caddy

这里参考了网上别人的一键脚本，这是最方便的。

```shell
$ wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/caddy_install.sh && chmod +x caddy_install.sh && bash caddy_install.sh install http.filemanager
```

一路回车就能安装完成。

## 0x02 配置caddy

脚本安装的caddy配置文件放在`/usr/local/caddy`路径下，这里执行如下命令:

```shell
$ vi /usr/local/caddy/Caddyfile
```

复制如下内容:

```
:80 {
    root /var/www
    gzip {
        ext .html .htm .php
        level 6
    }
}
```

之后再执行如下命令

```shell
$ vi /var/www/index.html
```

复制以下内容并保存

```
<!DOCTYPE html>
<html>
  <head>
    <title>Hello from Caddy!</title>
  </head>
  <body>
    <h1 style="font-family: sans-serif">This page is being served via Caddy</h1>
  </body>
</html>
```

如此一来，执行启动caddy命令`service caddy start`然后在浏览器输入ip地址即可看到网页输出的内容。

注意：防火墙要开放80端口。

<!--more-->

## 0x03 配置https

{% colorquote warning %}
确保防火墙开放了80和443端口。
{% endcolorquote %}

重新编辑caddy配置文件`vi /usr/local/caddy/Caddyfile`,用如下内容覆盖原先写的配置:

```
http://<example.com> {
    redir https://<example.com>{url}
}
 
https://<example.com> {
    root /var/www/
    gzip
 
    tls <user@example.com> {
        ciphers ECDHE-ECDSA-WITH-CHACHA20-POLY1305 ECDHE-ECDSA-AES256-GCM-SHA384 ECDHE-ECDSA-AES256-CBC-SHA
        curves p384
        key_type p384
    }
 
    header / {
        Strict-Transport-Security "max-age=31536000;"
        X-XSS-Protection "1; mode=block"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "DENY"
    }
}
```

>  example.com: 替换自己的域名					
>  user@example.com：替换自己的邮箱		

保存退出。

```shell
$ service caddy stop	#关闭caddy		
$ /etc/init.d/caddy start	#启动caddy，这里用了命令方式启动
```

启动完成之后，等待一段时间证书颁布完成，访问域名则可以看到已经是https了。

可以去[SSL Labes](https://www.ssllabs.com/)网站检测下https配置评分，差不多能拿到A+评级。

## 0x04 caddy管理命令

```shell
$ service caddy start	#启动		
$ service caddy stop	#关闭		
$ service caddy restart	#重启		
$ service caddy status	#查看状态
```

caddy申请证书路径如下:

/root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/域名/域名.crt		
/root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/域名/域名.key

caddy配置文件路径：/usr/local/caddy/		
caddy默认开机启动,如想关闭开机启动则执行`chkconfig caddy off`,想恢复开机启动则`chkconfig caddy on`。

## 0x05 caddy配置文件管理插件

刚才的安装命令其实已经包含了http.filemanager插件

只要在caddy的配置文件中增加以下一段配置即可开启文件管理

```
filemanager /download /data/file {
        database /usr/local/caddy/filemanager.db
    }
```

配置的意思前面的/download，后面的/data/file是指服务器存放文件根目录，database 是指文件管理的一些数据信息存放路径，一般建议跟caddy配置文件放同一路径方便管理。

如此一来，结合上面https的配置，访问域名+/download路径即可访问文件管理页面，默认用户名密码为admin/admin，配置可以登录以后修改。