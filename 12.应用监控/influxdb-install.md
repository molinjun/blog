---
title: influxDB的安装简介 
comments: true
date: 2017-01-22 09:53:52
categories: influxDB
tags: influxDB 运维
toc: true
---
influxDB 是一款时序数据库，用go编写，可用来存储大量的时间戳数据，包括DevOps监控数据、应用metrics、IoT 传感器数据以及实时分析数据等等。  
下面就介绍一些influxdb的安装以及基本使用。
<!--more-->
## 安装
influxDB的安装比较简单，也有多种方式，[参见官网下载页](https://www.influxdata.com/downloads/#influxdb)，提供各种操作系统的下载方式。
本文将简单介绍influxdb在ubuntu 和macos下的安装，大家可以按照官方教程根据自己的系统选择安装。当前influxDB最新版本为1.1.1。

### macos
在 macos 下可以通过Homebrew直接安装。
```
$ brew update
$ brew install influxdb
```
### ubuntu
ubuntu 下可以通过apt 来安装。
```
// 配置apt源
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
// 安装
sudo apt-get update && sudo apt-get install influxdb
// 启动
sudo service influxdb start
```
当然，也官网也直接提供deb包来安装。
```
// 下载安装包
$ wget https://dl.influxdata.com/influxdb/releases/influxdb_1.1.1_amd64.deb
// 安装
$ sudo dpkg -i influxdb_1.1.1_amd64.deb
// 启动和停止
$ sudo service influxdb start
$ sudo service influxdb stop
```
### docker
不管是macos还是ubuntu，最方便的方式当然还是使用docker了。
```
$ docker pull influxdb
$ docker run -p 8083:8083 -p 8086:8086 \
      -v $PWD:/var/lib/influxdb \
      influxdb
```
这样就下好influxdb的镜像了，docker run即可运行。
> 这是[docker hub地址](https://hub.docker.com/_/influxdb/),大家最好还是熟读readme和dockerfile，根据自己的需求配置启动。  

## 配置
influxdb提供了默认的内部配置。可以通过**influxd config** 查看。
```
$ influxd config
```
可以通过配置文件将默认配置覆盖。默认的配置文件位于**/etc/influxdb/influxdb.conf**。
通过配置文件启动可以直接通过 -config option或者通过环境变量的方式。option的优先级大于环境变量。
```
// option
$ influxd -config /path/to/config
// env
$ echo $INFLUXDB_CONFIG_PATH /etc/influxdb/influxdb.conf
$ influxd
```
docker方式下可以通过volume制定config来启动
```
// 生成配置文件
$ docker run --rm influxdb influxd config > influxdb.conf
//启动，以当前目录启动为例
$ docker run -p 8086:8086 \
      -v $PWD/influxdb.conf:/etc/influxdb/influxdb.conf:ro \
      influxdb -config /etc/influxdb/influxdb.conf
```
### admin panel 控制面板
influxdb 默认使用8086端口来和客户端如cli或者http api通信，同时提供8083端口来做可视化控制面板。  
不过，这个功能从1.1版本就默认不开启了，后续可能会删掉此功能，官方更建议使用http api 或 CLI的方式来使用。
可以通过修改配置文件来开启控制面板。
```
 [admin]
    # Determines whether the admin service is enabled.
    enabled = true
    # The default bind address used by the admin service.
    bind-address = ":8083"
```  
然后重启服务，然后通过**localhost:8083**进入即可。
安装好了之后，就可以通过influx指令，连接influxdb做接下来的增删改查操作。
> 其实influxdb 在启动和操作的方式上和mongodb很像，有过mongodb经验的朋友可以类比借鉴。

## 小结
- 用途：influxDB可以用来存储各种时间戳的数据。
- 安装：可以通过apt或者docker安装，更建议后者。
- 配置：修改配置文件覆盖默认配置。
- 控制面板：配置启动，不过应该按官网建议，以后使用CLI 和http api来替代此功能。
- 使用：用mongodb的方式来类比使用influxdb。

