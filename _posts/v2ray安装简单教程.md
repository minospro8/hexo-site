title: v2ray安装简单教程
author: 雪落无痕
tags: []
categories:
  - note
date: 2018-12-07 21:05:00
---
![](https://ws1.sinaimg.cn/large/683a46dcly1fxyhgwpwryj20mx0bmjsm.jpg)

## 0x00 前言

本教程只是最简单的安装，环境是centos 7。

## 0x01 安装

执行如下命令:

```shell
$ cd /root	#进入root目录
$ curl -L -s https://install.direct/go.sh > v2ray.sh	#下载安装脚本到v2ray.sh
$ /bin/bash v2ray.sh	#执行脚本安装
```

顺利的话就已经安装完了。

v2ray相关命令如下:

```shell
$ systemctl start v2ray	#启动
$ systemctl restart v2ray	#重启
$ systemctl stop v2ray	#关闭
$ systemctl status v2ray #查看状态
```

<!--more-->
## 0x02 配置

v2ray默认的配置大概长这样:

```
{
	"log": {
        "access": "/var/log/v2ray/access.log",
        "error": "/var/log/v2ray/error.log",
        "loglevel": "warning"
    },
  "inbound": {
    "port": 25536,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "xxxxxx-xxxx-xxxx",
          "level": 0,
          "alterId": 64
        }
      ]
    }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  },
  "outboundDetour": [
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "rules": [
        {
          "type": "field",
          "ip": ["geoip:private"],
          "outboundTag": "blocked"
        }
      ]
    }
  }
}
```

这样已经可以使用，注意防火墙开放port配置的端口号。

如果想兼容ss，则可以这样配置

```
 "inboundDetour": [
        {
            "protocol": "shadowsocks",
            "port": 5566,
            "settings": {
                "method": "chacha20-ietf-poly1305",
                "password": "password",
                "udp": true,
                "level": 1,
                "ota": false
            }
        }
    ],
```

这一段加在outbound和outboundDetour之间就可以了。

结构大概是这样：

```
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  },
  "inboundDetour": [
        {
            "protocol": "shadowsocks",
            "port": 5566,
            "settings": {
                "method": "chacha20-ietf-poly1305",
                "password": "password",
                "udp": true,
                "level": 1,
                "ota": false
            }
        }
    ],
  "outboundDetour": [
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ]
```

别忘了防火墙开放相应端口即可。
