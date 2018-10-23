---
layout: post
title:  "ShadowSocks&BBR&cron&V2ray&kms配置"
date:   2018-10-23 14:10:51 +0800
categories: VPS
tags: VPS
---

 **!!!所有命令都需要root权限!!!**
 **!!!所有命令都需要root权限!!!**
 **!!!所有命令都需要root权限!!!**
 
# 1.SS
# 1.1 一键脚本
[From- Shadowsocks 一键安装脚本（四合一）  |  秋水逸冰](https://teddysun.com/486.html)
```
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master /shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```
# 1.2配置
# 1.2.1配置目录
```
Shadowsocks-Python 版：
/etc/shadowsocks-python/config.json

ShadowsocksR 版：
/etc/shadowsocks-r/config.json

Shadowsocks-Go 版：
/etc/shadowsocks-go/config.json

Shadowsocks-libev 版：
/etc/shadowsocks-libev/config.json
```
# 1.2.2 参考配置
```
{
    "server":"0.0.0.0",
    "port_password":{
         "8088":"g0kUEX1N",
         "8188":"quxXx2dS",
         "8288":"wzCx3TMQ",
         "8388":"uE44oeUS",
         "8488":"ae5iSd8J",
         "8588":"56ST8WHp",
         "8688":"77CFLCeP",
         "8788":"98vH8r4A",
         "8888":"HR9bBjVi",
         "8988":"kWK00nFc"
    },
    "method":"rc4-md5",
    "timeout":300
} 

```
# 1.3 脚本
```
Shadowsocks-Python 版：
/etc/init.d/shadowsocks-python start | stop | restart | status

ShadowsocksR 版：
/etc/init.d/shadowsocks-r start | stop | restart | status

Shadowsocks-Go 版：
/etc/init.d/shadowsocks-go start | stop | restart | status

Shadowsocks-libev 版：
/etc/init.d/shadowsocks-libev start | stop | restart | status
```
# 1.4 说明
- 四合一脚本包括libev/go/ssr/python四个版本
- 传说libev最稳定/ssr拓展性强不过容易被识别/python最省资源/go神秘加成


# 2 Cron
# 2.1 一键脚本
[From- 使用定时任务cron监视Shadowsocks进程  |  秋水逸冰](https://teddysun.com/525.html)
```
wget --no-check-certificate -O /opt/shadowsocks-crond.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-crond.sh
chmod 755 /opt/shadowsocks-crond.sh
```
# 2.2 检查 cron 进程
```
ps -ef | grep -v grep | grep cron
```
如果存在返回值，则表示 cron 已经正确安装并处于启动中。
否则，则需要安装 cron。
```
yum install -y crontabs
```
# 2.3 配置 cron 计划
假设监视脚本路径就是 /opt/shadowsocks-crond.sh
假设设为每 5 分钟监视一次。
那么配置 cron 计划如下：
```
(crontab -l ; echo "*/5 * * * * /opt/shadowsocks-crond.sh") | crontab -
```
以上表示，在保留原有的 cron 设置的前提下，追加设置
*/5 * * * * /opt/shadowsocks-crond.sh
即每过 5 分钟，执行一次脚本 /opt/shadowsocks-crond.sh
这样系统便会每 5 分钟检查一下 Shadowsocks 进程是否存在，如果不存在了会自动重新启动。
脚本每次运行会写日志的，日志完整路径如下：
/var/log/shadowsocks-crond.log
# 2.4 示例日志
```
2018-10-11 02:00:01 Shadowsocks-Go is running with pid 894
2018-10-11 02:05:01 Shadowsocks-Go is running with pid 894
2018-10-11 02:05:01 Shadowsocks-Go is running with pid 894
2018-10-11 02:10:01 Shadowsocks-Go is running with pid 894
2018-10-11 02:10:01 Shadowsocks-Go is running with pid 894
2018-10-11 02:15:01 Shadowsocks-Go is running with pid 894
2018-10-11 02:15:01 Shadowsocks-Go is running with pid 894
2018-10-11 02:20:01 Shadowsocks-Go is running with pid 894
2018-10-11 02:20:01 Shadowsocks-Go is running with pid 894
```
# 3 BBR
# 3.1 一键脚本
[From- 一键安装最新内核并开启 BBR 脚本  |  秋水逸冰](https://teddysun.com/489.html)
```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```
安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。
# 3.2 检查安装
```
uname -r
```
查看内核版本，显示为最新版就表示 OK 了

```
sysctl net.ipv4.tcp_available_congestion_control
```
返回值一般为：
net.ipv4.tcp_available_congestion_control = bbr cubic reno
或者为：
net.ipv4.tcp_available_congestion_control = reno cubic bbr

```
sysctl net.ipv4.tcp_congestion_control
```
返回值一般为：
net.ipv4.tcp_congestion_control = bbr

```
sysctl net.core.default_qdisc
```
返回值一般为：
net.core.default_qdisc = fq

```
lsmod | grep bbr
```
返回值有 tcp_bbr 模块即说明 bbr 已启动。注意：并不是所有的 VPS 都会有此返回值，若没有也属正常。

# 4 V2ray-服务端
# 4.1 安装
[From- V2Ray 配置指南|V2Ray 白话文教程](https://toutyrater.github.io/)
# 4.1.1 时间校准
对于 V2Ray，它的验证方式包含时间，就算是配置没有任何问题，如果时间不正确，也无法连接 V2Ray 服务器的，服务器会认为你这是不合法的请求。所以系统时间一定要正确，只要保证时间误差在一分钟之内就没问题。
# 4.1.2 安装
```
wget https://install.direct/go.sh
bash go.sh
```
# 4.1.3 命令
启动`systemctl start v2ray`

停止`systemctl stop v2ray`

重启`systemctl restart v2ray`

在首次安装完成之后，V2Ray 不会自动启动，需要手动运行上述启动命令。而在已经运行 V2Ray 的 VPS 上再次执行安装脚本，安装脚本会自动停止 V2Ray 进程，升级 V2Ray 程序，然后自动运行 V2Ray。在升级过程中，配置文件不会被修改。

# 4.2 参考配置
```
{
	"log": {
	 "loglevel": "warning"
  },
  "inbound": {
  	"port": 10086,
  	"protocol": "vmess",
  	"settings": {
  		"clients": [{
  			"id": "7f1faaee-e227-41e2-926c-9e1a377bc9b3",
  			"alterId": 64
  		}]
  	}
  },
  "outbound": {
  	"protocol": "freedom",
  	"settings": {}
  }
}
 ```
 
具体的协议、路由功能、拓展功能等参考From

# 5 V2ray-客户端
# 5.1 客户端安装
[安装](https://github.com/v2ray/v2ray-core/releases)
[备用地址](https://v2ray.com/download)
v2ray.exe 是运行 v2ray 的文件，config.json 是配置文件
# 5.2 代理配置
需要手动在浏览器配置socks5代理
Chrome推荐使用SwitchyOmega
Firefox可以在设置Advanced->Network里面配置
# 5.3 客户端配置
参考配置
```
{
  "log": {
    "loglevel": "warning",
    "access": "C:\\Users\\***\\Downloads\\v2ray-windows-64\\v2ray-v3.39-windows-64\\access.log",
    "error": "C:\\Users\\***\\Downloads\\v2ray-windows-64\\v2ray-v3.39-windows-64\\error.log"
  },
  "inbound": {
    "port": 1080,
    "protocol": "socks",
    "domainOverride": ["tls","http"],
    "settings": {
      "auth": "noauth",
      "udp": true
    }
  },
  "outbound": {
    "protocol": "vmess",
    "settings": {
    	"vnext": [
        {
          "address": "***", // 服务器地址，请修改为你自己的服务器 IP 或域名
          "port": 10086,  // 服务器端口
          "users": [
            {
              "id": "7f1facce-e227-41e2-926c-9e1a377bc9b3",  // 用户 ID，必须与服务器端配置相同
              "alterId": 64 // 此处的值也应当与服务器相同
            }
          ]
        }
      ]
    },
    "mux": {"enabled": true}
  },
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct" //如果要使用路由，这个 tag 是一定要有的，在这里 direct 就是 freedom 的一个标号，在路由中说 direct V2Ray 就知道是这里的 freedom 了
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "IPOnDemand",
      "rules": [
        {
          "type": "field",
          "outboundTag": "direct",
          "domain": ["geosite:cn"]
        },
        {
          "type": "chinaip",
          "outboundTag": "direct",
          "ip": ["geoip:cn"]
        }
      ]
    }
  },
  "policy": {
    "levels": {
      "0": {"uplinkOnly": 0}
    }
  }
}

```
具体配置见 #4.1的From
**注意log的file一定是存在的，否则会报错。不配置log的话默认在consle中展示，不会被保存**

# 6 KMS服务
# 6.1 脚本
[From- 一键安装KMS服务脚本](https://teddysun.com/530.html)
```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/kms.sh && chmod +x kms.sh && ./kms.sh
```
安装完成后，输入以下命令查看端口号 1688 的监听情况
```
netstat -nxtlp | grep 1688
```
# 6.2 命令 
启动：`/etc/init.d/kms start`

停止：`/etc/init.d/kms stop`

重启：`/etc/init.d/kms restart`

状态：`/etc/init.d/kms status`

卸载：`./kms.sh uninstall`

# 6.3 使用 -- 在本地使用管理员权限运行cmd
# 6.3.1 Window激活
1.查看系统版本
```
wmic os get caption
```
2.安装从上面列表得到的 key
```
slmgr /ipk xxxxx-xxxxx-xxxxx-xxxxx-xxxxx
```
3.将 KMS 服务器地址设置为你自己的 IP 或 域名，后面最好再加上端口号（:1688）
```
slmgr /skms Your IP or Domain:1688
```
4.手动激活系统
```
slmgr /ato
```
# 6.3.2 Office激活
1.进入 Office 目录(目录需要手动查找)
```
cd "C:\Program Files (x86)\Microsoft Office\Office16"
```
2.注册 KMS 服务器地址
```
cscript ospp.vbs /sethst:Your IP or Domain
```
3.手动激活 Office
```
cscript ospp.vbs /act
```

# 7. 防火墙
# 7.1 CentOS7
Centos7使用的是firewalld而非iptables 

添加新的配置之后**可能**需要手动配置防火墙策略
# 7.1.1 Firewalld基本使用
启动： `systemctl start firewalld`

关闭： `systemctl stop firewalld`

查看状态： `systemctl status firewalld`

开机禁用  ： `systemctl disable firewalld`

开机启用  ： `systemctl enable firewalld`

# 7.1.2 systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体
启动一个服务：`systemctl start firewalld.service`

关闭一个服务：`systemctl stop firewalld.service`

重启一个服务：`systemctl restart firewalld.service`

显示一个服务的状态：`systemctl status firewalld.service`

在开机时启用一个服务：`systemctl enable firewalld.service`

在开机时禁用一个服务：`systemctl disable firewalld.service`

查看服务是否开机启动：`systemctl is-enabled firewalld.service`

查看已启动的服务列表：`systemctl list-unit-files|grep enabled`

查看启动失败的服务列表：`systemctl --failed`

# 7.1.3 配置firewalld-cmd
查看版本： `firewall-cmd --version`

查看帮助： `firewall-cmd --help`

显示状态： `firewall-cmd --state`

查看所有打开的端口： `firewall-cmd --zone=public --list-ports`

更新防火墙规则： `firewall-cmd --reload`

查看区域信息:  `firewall-cmd --get-active-zones`

查看指定接口所属区域： `firewall-cmd --get-zone-of-interface=eth0`

拒绝所有包：`firewall-cmd --panic-on`

取消拒绝状态： `firewall-cmd --panic-off`

查看是否拒绝： `firewall-cmd --query-panic`

添加： `firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）`

查看： `firewall-cmd --zone= public --query-port=80/tcp`

删除： `firewall-cmd --zone= public --remove-port=80/tcp --permanent`

重新载入： `firewall-cmd --reload`