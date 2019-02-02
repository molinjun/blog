---
title: zabbix添加监控主机
comments: true
date: 2016-11-30 00:13:16
categories: zabbix
tags: zabbix
toc: true
---
按照前面两篇文章{% post_link zabbix-server-install %} 和 {% post_link zabbix-agent-install %}，我们已经安装好了zabbix server和zabbix agent。zabbix agent把监控机的数据汇总到zabbix server。zabbix的模板非常丰富，我们可以非常容易的，将一些预定义的模板应用的我们的主机上。
下面我们就介绍一下，如何在界面上添加主机，并如何查看监控的数据。

## 添加主机到zabbix
### 登录zabbix
打开<code>http://{ ip | domain }/zabbix</code> 进入zabbix界面，输入用户名、密码登录。

### 添加主机
点击主菜单的【配置】- 子菜单【主机】- 右边的【创建主机】按钮。如下图。
![添加主机](http://obv0ef5sf.bkt.clouddn.com/zabbix-add-host)

### 填写主机信息
按要求填写主机的信息：
- 主机名称
- 可见名称
- 组
- 代理接口

填写好以上信息，点击【保存】按钮。
![填写主机信息](http://obv0ef5sf.bkt.clouddn.com/zabbix-host-config)

### 应用模板
点击【模板】子菜单，点击【添加】按钮，出现模板集的窗口，勾选所需模板，点击【选择】。然后在主界面点击【更新】按钮。
![添加模板](http://obv0ef5sf.bkt.clouddn.com/zabbix-template)

至此，就添加好主机了。
![主机](http://obv0ef5sf.bkt.clouddn.com/zabbix-hosts)

## 查看监控图表
zabbix的图表做的很漂亮。你可以选择查看不同的监控指标。
![](http://obv0ef5sf.bkt.clouddn.com/zabbix-graph)

好了，zabbix 算是跑起来了。不过，这才是最基本的应用。后续需要进一步细化各种指标，以及不同的应用场景。

## 参考
[1] [How to Add Host in Zabbix Server to Monitor](http://tecadmin.net/add-host-zabbix-server-monitor/)
