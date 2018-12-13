title: Maven的安装与使用
author: 雪落无痕
tags: []
categories:
  - note
date: 2016-12-11 17:51:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fy2yclg51xj20hs042748.jpg)

maven作为项目管理工具已经是必不可少的技能。

## 0x00 解压

```
$ tar -zxvf apache-maven-3.3.9-bin.tar.gz
```

## 0x01 环境变量设置

`vi /etc/profile`编辑文件增加以下内容


```sh
MAVEN_HOME=/usr/local/maven3
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```

`source /etc/profile`使配置生效

Maven中央仓库：[http://search.maven.org/](http://search.maven.org/)	
Maven仓库: [http://mvnrepository.com/](http://mvnrepository.com/)

maven常用命令:[参考](http://www.cnblogs.com/phoebus0501/archive/2011/05/10/2042511.html)


`mvn clean install -Pdev -Dmaven.test.skip=true`

上传本地jar到仓库

`mvn install:install-file -Dfile=E:\\doc\\dubbo-2.8.4.jar -DgroupId=com.alibaba -DartifactId=dubbo -Dversion=2.8.4 -Dpackaging=jar -DgeneratePom=true`

settings.xml文件详解: [地址](http://blog.csdn.net/yiluoak_47/article/details/12068855)