---
title: zabbix agent安装 
comments: true
date: 2016-11-29 23:14:01
categories: zabbix
tags: zabbix 运维
toc: true
---
根据上篇文章{% post_link zabbix-server-install %}，已经成功安装好了zabbix server。接下来，我们需要在需要监控的主机上安装zabbix agent。zabbix agent用来收集主机的资源利用率和应用数据，并上报给zabbix server做统计。
下面就详细介绍下zabbix agent的安装过程。
<!--more-->
## 添加apt repository
```
For Ubuntu 16.04 LTS:
$ wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+xenial_all.deb
$ sudo dpkg -i zabbix-release_3.0-1+xenial_all.deb
$ sudo apt update

For Ubuntu 14.04 LTS:

$ wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+trusty_all.deb
$ sudo dpkg -i zabbix-release_3.0-1+trusty_all.deb
$ sudo apt-get update
```
## 安装zabbix agent
添加好apt repository之后，通过下面命令安装agent。
```
$ sudo apt-get install zabbix-agent
```
## 修改zabbix agent配置
zabbix agent 的配置文件位于/etc/zabbix/zabbix_agentd.conf。修改一下配置:
```
#Server=[zabbix server ip]
#Hostname=[Hostname of client system ]

Server=192.168.1.11
Hostname=Server2
```
## 启停zabbix agent
配置好了之后，可以通过下面命令启动和停止zabbix agent。
```
# /etc/init.d/zabbix-agent restart
# /etc/init.d/zabbix-agent start
# /etc/init.d/zabbix-agent stop
```
至此，zabbix agent安装完毕。zabbix就会收集监机的数据，并上报给zabbix server。

## 参考
[1] [How To Install Zabbix Agent on Ubuntu ](http://tecadmin.net/install-zabbix-agent-on-ubuntu-and-debian/)
