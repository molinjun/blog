---
title: zabbix server安装
comments: true
date: 2016-11-28 01:10:16
categories: zabbix
tags: zabbix 运维
toc: true
---
zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。
zabbix由2部分构成，zabbix server与可选组件zabbix agent。
zabbix server可以通过SNMP，zabbix agent，ping，端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能。zabbix agent需要安装在被监视的目标服务器上，它主要完成对硬件信息或与操作系统有关的内存，CPU等信息的收集。
下面将详细介绍zabbix server的安装。
<!--more-->
### 安装LAMP环境
#### 安装apache2
```
$ sudo apt-get update
$ sudo apt-get install apache2
```
#### 安装MySQL
```
$ sudo apt-get install mysql-server
```
安装的过程成需要输入<code>root</code>的密码，然后再次输入密码确认。

#### 安装php
**ubuntu 14.04**
```
$ sudo apt-get install php5 php5-cli php5-common php5-mysql
```
**ubuntu 16.04**
```
$ sudo apt-get install php7.0 php7.0-cli php7.0-common php7.0-mysql
```
> ubuntu16.04里apt-get安装的默认就是7.0，已经不是5.0。这也是自己遇到的一个坑。下面的安装中，两个版本还是有细微差异的。

#### 修改时区
**ubuntu 14.04**
修改/etc/php5/apache2/php.ini文件中的timezone
```
date.timezone = 'Shanghai'
```
**ubuntu 16.04**
这时并没有找到php目录下有apache2。需要再安装<code>libapache2-mod-php7.0</code>。
```
$ sudo apt-get install libapache2-mod-php7.0
```
安装完成后，修改/etc/php/7.0/apache2/php.ini文件中的timezone
```
date.timezone = 'Asia/Shanghai'
```
### 增加zabbix server的apt repository
在安装zabbix server之前，先配置好它的rpm repository。
```
For Ubuntu 16.04 LTS:

$ wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+xenial_all.deb
$ sudo dpkg -i zabbix-release_3.0-1+xenial_all.deb
$ sudo apt-get update

For Ubuntu 14.04 LTS:

$ wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+trusty_all.deb
$ sudo dpkg -i zabbix-release_3.0-1+trusty_all.deb
$ sudo apt-get update
```
### 安装zabbix server
设置好apt repository 之后，就安装使用mysql的zabbix server。
```
$ sudo apt-get install zabbix-server-mysql zabbix-frontend-php
```
### 创建数据库schema
现在需要为zabbix server创建数据库的schema。

**创建数据库和用户**
```
$ mysql -u root -p
//创建数据库zabbixdb
mysql> CREATE DATABASE zabbixdb;
//授予zabbixdb给本地的zabbix用户，并设置密码
mysql> GRANT ALL on zabbixdb.* to zabbix@localhost IDENTIFIED BY 'password';
mysql> FLUSH PRIVILEGES;
```
**基于新建的数据库，重启zabbix database schema**。
```
$ cd /usr/share/doc/zabbix-server-mysql
$ zcat create.sql.gz | mysql -u root -p zabbixdb
```
> 这里需要输入密码，是root的密码，也就是创建mysql数据库时的密码，而不是zabbix数据库的密码。

### 编辑zabbix的配置文件
zabbix的配置文件位于/etc/zabbix/zabbix_server.conf。修改以下内容。
```
DBHost=localhost
DBName=zabbixdb
DBUser=zabbix
DBPassword=password
```
### 重启Apache 和 Zabbix
zabbix会创建自己的apache 配置文件，位于/etc/zabbix/apache2.conf。使用如下命令重启apache服务。
```
$ sudo service apache2 restart
```
zabbix的配置文件位于/etc/zabbix/zabbix_server.conf。使用如下命令重启：
```
$ sudo service zabbix-server restart
```
启动好zabbix server之后，就可以开始zabbix 的web安装。

### 启动zabbix web 安装。
启动zabbix server之后，通过浏览器打开如下url，根据提示安装。
```
http://<域名|ip>/zabbix
```
**zabbix安装的欢迎界面**
![](http://obv0ef5sf.bkt.clouddn.com/zabbix-install-welcome)
点击下一步按钮。
**检查需要安装的依赖**
检查系统是否有所有安装的包，如果有没安装的，使用apt-get 安装。
> ubuntu16.04 会出现没有xml-reader、xmlwriter等包的情况。

![](http://obv0ef5sf.bkt.clouddn.com/zabbix-install-pre)
确认好之后，点击下一步。
**配置数据库连接**
根据上面建立的数据库，设置数据库的连接，数据库名字，用户及密码。
![](http://obv0ef5sf.bkt.clouddn.com/zabbix-install-dbcfg)
设置好之后，点击下一步。
**zabbix server详情**
数据设置成功之后，会显示zabbix server的详情。
![](http://obv0ef5sf.bkt.clouddn.com/zabbix-install-detail)
**显示上面设置的概况**
![](http://obv0ef5sf.bkt.clouddn.com/zabbix-install-summary)
**安装zabbix**
![](http://obv0ef5sf.bkt.clouddn.com/zabbix-install-ok)

**登录zabbix**
使用默认账户登录。
```bash
Username:  admin
Password:  zabbix
```
![](http://obv0ef5sf.bkt.clouddn.com/zabbix-dashboard)
至此，zabbix-server就安装成功了。接下来安装zabbix agent 和在zabbix server中增加主机。

### 参考
[1][Ubuntu下安装Zabbix](http://blog.csdn.net/yoara/article/details/41845473)
[2][How to Install Zabbix Server 3.0 on Ubuntu](http://tecadmin.net/install-zabbix-on-ubuntu/)



