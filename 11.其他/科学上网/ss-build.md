---
title: ubuntu搭建shadowsocks及优化加速
comments: true
date: 2016-11-29 00:32:22
categories: shadowsocks
tags: shadowsocks 翻墙
toc: true
---
由于GFW，需要google我们就必须得翻墙。相信程序猿都比较熟悉shadowsocks这款翻墙软件了。它非常轻量，而且配置机器简单，而且支持多平台运行。因为刚好有开通了AWS的EC2,所以就刚好搭建一个shadowsocks服务器。
下面就简单介绍一下在ubuntu下搭建shadowsocks服务器的方法。

## 安装
因为ss是python写的，所以需要先安装python的包管理工具<code>pip</code>。
```bash
$ sudo apt-get install python-pip
```
安装ss
```
pip install shadowsocks
```
没错，就是这么简单。ss已经安装完毕。

## 配置
启动ss服务器的方法也很简单。
```
$ ssserver -p 443 -k password -m rc4-md5
-p 端口
-k 密码
-m 加密方式
```
后台运行
```
sudo ssserver -p 443 -k password -m rc4-md5 --user nobody -d start
```
停止
```
sudo ssserver -d stop
```
日志文件位于/var/log/shadowsocks.log。

当然，我更建议使用配置文件。创建配置文件<code>/etc/shadowsocks.json</code>。
单端口
```
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
多端口
```
{
    "server":"0.0.0.0",
    "port_password": {
        "端口1": "连接密码1",
        "端口2" : "连接密码2"
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
启停
```
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```
> 加密方式AES更安全但是较慢。rc4-md5更快，但相对不安全。不过据说之所以不安全是因为 Shadowsocks 在每个连接上重复使用 key，没有使用 IV。现在已经重新正确实现，可以放心使用。更多可以看 [issue](https://github.com/shadowsocks/shadowsocks/issues/178)。

根据以上步骤，ss服务器已经搭建好了。只要在自己的平台上，安装好ss的客户端，配置可以翻墙了。不过，如果需要看youtube等的话，速度还不是辣么的满意，所以需要做一些优化。
## 优化加速
网上查了一些优化的方式，一些几种是实践之后比较推荐的。
### 锐速 serverspeeder
锐速是一款tcp加速工具。网上也很多人推荐，不过好像现在是付费了。具体的方法及安装步骤请查看官网。
不过，还是有[破解版](https://www.91yun.org/archives/683)的。安装也很方便。
```
$ wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh
$ sudo bash serverspeeder-all.sh
```
不过，破解版对内核有要求，内核不匹配需要重装下内核。如果你按上述命令安装不成功，那恭喜你，请移步官网，找具体的[解决方法](https://www.91yun.org/archives/683)。

### netspeeder
<code>net-speeder</code>是在丢包状况下，多次发包来实现的。安装也及其方便，请参考[官网](https://github.com/snooda/net-speeder)。
不过，netspeeder容易被发现墙掉。

### TFO
TCP Fast Open可以优化tcp的三次握手次数，加快速度。可以在配置文件中，将<code>fast_open</code>设为<code>true</code>。然后官网说通过下面命令,将fastopen数设置成3。
```
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
```
但是，我的内核并不允许我修改此文件。其实我也不建议直接该这种内核文件。可以通过另一个配置方法。
修改/etc/sysctl.conf文件。
```
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096
# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1
# for high-latency network
#net.ipv4.tcp_congestion_control = hybla
# for low-latency network
net.ipv4.tcp_congestion_control = htcp
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
```
然后，使配置生效。
```
sudo sysctl -p
```
我个人还是选择使用第三种方法。不安装更多第三方软件，而且速度相对还算可以。看下图：
![](http://obv0ef5sf.bkt.clouddn.com/ss-speed)

## 参考
[1] [科学上网之EC2搭建shadowsocks](https://segmentfault.com/a/1190000003101075)
[2] [github shadowsocks wiki](https://github.com/shadowsocks/shadowsocks/wiki)
[3] [Shadowsocks搭建以及用锐速加速](http://www.jianshu.com/p/65128dd81827)
[4] [科学上网之 Shadowsocks 安装及优化加速](http://wuchong.me/blog/2015/02/02/shadowsocks-install-and-optimize/)


