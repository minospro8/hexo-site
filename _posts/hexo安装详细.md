title: hexo安装详细教程
author: 雪落无痕
tags: []
categories:
  - note
date: 2018-12-04 16:06:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fxurxwo76oj20hs07sjrm.jpg)

## 0x00 前言

hexo安装需要git环境和node.js环境，这里以centos 7为例，以root用户登录。

## 0x01 环境安装

安装git

```shell
$ yum install git-core	#安装git
$ git --version	#查看安装的git版本
```

用nvm管理node.js

curl:

```shell
$ curl https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
```

wget:

```shell
$ wget -qO- https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
```

任选以上任何一种方式安装nvm，安装完成以后重启服务器，执行以下命令安装node.js

```shell
$ nvm install stable
```

至此，所依赖的环境安装完成，执行`node -v`命令可查看当前node.js版本。

## 0x02 hexo安装

```shell
$ npm install -g hexo-cli
```

## 0x03 博客初始化


```shell
$ cd /usr/local	#进入目录	
$ mkdir hexo-site	#创建博客目录	
$ cd hexo-site	#进入博客目录	
$ hexo init #初始化	
$ npm install	#安装依赖	
```

以上操作都完成以后,在/usr/local/hexo-site目录下执行 

```shell
$ hexo g	#生成静态文件	
$ hexo s	#启动hexo服务,在当前服务器ip的4000端口能打开博客首页，前提是服务器防火墙开放4000端口
```

<!--more-->
## 0x04 博客使用第三方主题

这里以minos主题为例,[github链接](https://github.com/ppoffice/hexo-theme-minos)。

```shell
$ cd /usr/local/hexo-site/themes	#进入博客目录的主题文件夹
$ git clone https://github.com/ppoffice/hexo-theme-minos.git minos	#clone主题
$ cd minos	
$ git tag --list	
$ git checkout -b tag2.2.1 2.2.1	#以2.2.1这个tag切出来一个本地分支名叫tag2.2.1,并切换到该分支，当然直接用master分支的代码也可以，这里用最新tag的代码切出分支是稳妥起见。	
$ git branch -a	#查看目前分支情况以及当前在哪个分支	
$ mv _config.yml.example _config.yml	#把主题的配置文件开起来	
```

上面就完成了主题的安装，当然主题的配置文件_config.yml有更详细的配置，请参考github上的指导。

接下来就是使用主题

```shell
$ cd /usr/local/hexo-site	
$ vi _config.yml	#修改博客配置文件	
```

把配置文件中的theme: landscape这一行改为theme: minos,然后在博客目录执行`hexo g`和`hexo s`分别编译和启动博客程序即可。

注意: 这里会有以下提醒

> ERROR Package hexo-renderer-sass is not installed.	
> ERROR Please install the missing dependencies in the root directory of your Hexo site.

这是因为博客主题minos需要安装hexo-renderer-sass插件，只要在博客目录(/usr/local/hexo-site）执行以下命令即可安装

```shell
$ npm install --save hexo-renderer-sass	
```

之后再按照上面重新编译和启动博客就不会报错了。

{% colorquote info %}
关于minos主题的一些配置及使用方式，以后有机会可以专门写一篇文章介绍。
{% endcolorquote %}

## 0x05 使用hexo-admin后台管理博客

这是一款插件,地址在[这里](https://github.com/jaredly/hexo-admin)。

进入到博客的主目录`/usr/local/hexo-site`

执行以下命令

```shell
$ npm install --save hexo-admin	#安装插件
```

安装完成以后正常启动`hexo s`即可，可在博客主页路径之后加上/admin即可访问管理后台。

菜单大概是这个样子，如下图:

![](https://ws1.sinaimg.cn/large/683a46dcly1fxy5ggzdrsj20l703ct8v.jpg)

这里对图中的菜单做下简单说明:

**Posts**: 文章管理		
**Pages**: 页面管理，主要是一些关于页面之类		
**About**: hexo-admin插件的一些信息		
**Deploy**: 自动发布博客，这个后面会详细介绍		
**Settings**: 插件的一些设置，设置密码登录就可以参考这里

接下来主要讲一下在线发布和设置用户密码。

__1.在线发布__

![](https://ws1.sinaimg.cn/large/683a46dcly1fxy5wlftgkj20jm06st93.jpg)

deploy的输入位置是对应hexo-admin配置里面的deployCommand项，如果配置里面已经配置了该项，则这里不用输入任何文字，直接点击Deploy按钮即可完成在线发布。

示例:

```
admin:
  deployCommand: './hexo-deploy'	
```

这样，只要在博客主目录`/usr/local/hexo-site`下，有名为hexo-deploy的脚本就可以了。

脚本内容可以参考下面:

```
hexo clean && hexo g
```

这里只是清除及重新编译生成博文。这里的脚本是可以根据需求自己定义的。但是这里的hexo-deploy**需要有可执行权限**。(chmod +x hexo-deploy)

__2.设置用户名密码__

进入后台管理Settings->Setup authentification 菜单,如下图:

![](https://ws1.sinaimg.cn/large/683a46dcly1fxy680es2yj20on0nw76d.jpg)

填写相关配置，然后把红框部分的内容复制下来配置到博客主目录的_config.yml文件(不是主题目录下的_config.yml)末尾即可。

结合第1点在线发布，完整的配置应该如下:

```
admin:
  username: xxx
  password_hash: xxx
  secret: xxx
  deployCommand: './hexo-deploy'
```

修改完配置，保存重启博客即可，再次登录后台则需要输入用户名密码。

## 0x06 让hexo一直后台运行

官方建议是`hexo s &`命令运行则是后台启动，但是实际体验下来还是会莫名其妙死掉进程。参考了这篇文章[链接](https://blog.csdn.net/Tangcuyuha/article/details/80331169)。

安装pm2

```shell
$ npm  install -g pm2
```

然后在博客根目录创建一个脚本**hexo_run.js**,内容如下:

```
const { exec } = require('child_process')
exec('hexo server',(error, stdout, stderr) => {
        if(error){
                console.log('exec error: ${error}')
                return
        }
        console.log('stdout: ${stdout}');
        console.log('stderr: ${stderr}');
})
```

最后在根目录运行如下命令即可后台启动hexo程序

```
$ pm2 start hexo_run.js
```

可通过命令`netstat -apn | grep hexo`来查看当前hexo进程。

## 0x07 安装live2d插件

插件地址[github](https://github.com/EYHN/hexo-helper-live2d)

在博客根目录执行以下命令安装

```
$ npm install --save hexo-helper-live2d@3.x
```

安装完成以后编辑博客根目录的_config.yml文件，在最后加如下配置

```
live2d:
  model:
    scale: 1
    hHeadPos: 0.5
    vHeadPos: 0.618
  display:
    superSample: 2
    width: 150
    height: 300
    position: left
    hOffset: 0
    vOffset: -20
  mobile:
    show: true
    scale: 0.5
  react:
    opacityDefault: 0.7
    opacityOnHover: 0.2
```

重新编译博客即可。