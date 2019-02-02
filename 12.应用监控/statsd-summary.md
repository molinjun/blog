---
title: statsd学习小结
comments: true
date: 2017-01-25 17:31:12
categories: statsd
tags: statsd monitor
toc: true
---
应用程序的监控是微服务中很重要的一环。监控主要包括四个方面的内容：指标（metrics）的采集、存储、展示以及相应的报警机制。目前相关的解决方案以及工具非常多。今天就介绍一款用于采集数据的工具——statsd。
Statsd 最早是 2008 年 Flickr 公司用 Perl 写的针对 Graphite、datadog 等监控数据后端存储开发的前端网络应用，2011 年 Etsy 公司用 node.js 重构。statsd狭义来讲，其实就是一个监听UDP（默认）或者TCP的守护程序，根据简单的协议收集statsd客户端发送来的数据，聚合之后，定时推送给后端，如graphite和influxdb等，再通过grafana等展示。
statsd系统包括三部分：**客户端（client）**、**服务器（server）**和后端**（backend）**。客户端植入于应用代码中，将相应的metrics上报给statsd server。statsd server聚合这些metrics之后，定时发送给backends。backends则负责存储这些时间序列数据，并通过适当的图表工具展示。
<!--more-->
## 基本原理与概念
statsd采用简单的行协议：
```
<bucket>:<value>|<type>[|@sample_rate]
```
### bucket
bucket是一个metric的标识，可以看成一个metric的变量。
### value
metric的值，通常是数字。
### type
metric的类型，通常有**timer**、**counter**、**gauge**和**set**四种。
### sample_rate
如果数据上报量过大，很容易溢满statsd。所以适当的降低采样，减少server负载。
这个频率容易误解，需要解释一下。客户端减少数据上报的频率，然后在发送的数据中加入采样频率，如0.1。statsd server收到上报的数据之后，如cnt=10，得知此数据是采样的数据，然后flush的时候，按采样频率恢复数据来发送给backend，即flush的时候，数据为cnt=10/0.1=100，而不是容易误解的10*0.1=1。

### UDP 和 TCP
statsd可配置相应的server为UDP和TCP。默认为UDP。UDP和TCP各有优劣。但
UDP确实是不错的方式。
- UDP不需要建立连接，速度很快，不会影响应用程序的性能。
- “fire-and-forget”机制，就算statsd server挂了，也不会造成应用程序crash。
当然，UDP更适合于上报频率比较高的场景，就算丢几个包也无所谓，对于一些一天已上报的场景，任何一个丢包都影响很大。另外，对于网络环境比较差的场景，也更适合用TCP，会有相应的重发，确保数据可靠。

> 建议使用UDP。TCP还是有许多弊端的。

## 安装
statsd的安装非常简单。可选择两种方式：克隆源码和docker。
### 克隆源码
首先需要安装node环境。不清楚的可以参考这篇文章。然后到github克隆代码，修改相关配置启动即可。
```
1、 git clone git@github.com:etsy/statsd.git
2、 cd path/to/statsd
3、 根据exampleConfig文件定义自己的配置文件
4、 node stats.js path/to/config
```
这样statsd server就搭建成功了。
### docker
用docker也是个好选择。
```
docker run -p 8125:8125 -p 8126:8126 --name statsd -d dockerana/statsd

```
statsd 默认监听8125来收集udp包。
可以通过nc指令测试数据收发。
```
echo "foo:1|c" | nc -w 1 -u 127.0.0.1 8125
```
## 配置
statsd提供默认的配置文件exampleConfig.js。可以参考相应的注释按需配置，接下来将简单介绍一些配置项。
- **端口**
默认为8125端口。
```
port: 8125
```
- **后端**
默认有console、greaphite等，也有[influxdb](https://github.com/bernd/statsd-influxdb-backend)等backend。console的后端通常加上prettyprint。可以同时配置多个backends。backends都要放在代码目录的backends目录下。
```
backends: ["./backends/console", "./backends/graphite"],
console: {
    prettyprint: true
}
```
- **flush interval**
statsd 默认是10s执行一次flush。可通过flushInterval设置，单位ms。
```
flushInterval: 2000  // 设为2s
```
- **reload 配置**
设置automaticConfigReload，watch配置文件，如果修改，即reload配置文件。默认为true。（然而reload配置之后，并没有生效。）

- **delete系列配置**
metric上报时，每次flush之后，就会重置为0（gauge是保持原有值）。如果不上报这些空闲值，可以通过delete*来设置。
```
deleteGauges: true,
deleteTimers: true,
deleteSets: true,
deleteCounters: true
```
- **percentThreshold**
对于timer数据，会计算一个百分比的数据（过滤掉峰值数据），默认是90%。可以通过percentThreshold修改这个值或配置多个值。
```
//分别计算50%和80%的相关值
percentThreshold: [50, 80]
```
只列举了部分配置项，具体请参考配置文件。
## 指标 metric
statsd 有四种指标类型：counter、timer、gauge和set。
### 计数器 counter
counter类型的指标，用来计数。在一个flush区间，把上报的值累加。值可以是正数或者负数。
```
user.logins:10|c        // user.logins + 10
user.logins:-1|c        // user.logins - 1
user.logins:10|c|@0.1   // user.logins + 100
                        // users.logins = 10-1+100=109
```
### 计时器 timer
timers用来记录一个操作的耗时，单位ms。statsd会记录平均值（mean）、最大值（upper）、最小值（lower）、累加值（sum）、平方和（sum_squares）、个数（count）以及部分百分值。
```
rpt:100|g
```
如下是在一个flush期间，发送了一个rpt的timer值100。以下是记录的值。
```
count_80: 1,
mean_80: 100,
upper_80: 100,
sum_80: 100,
sum_squares_80: 10000,
std: 0,
upper: 100,
lower: 100,
count: 1,
count_ps: 0.1,
sum: 100,
sum_squares: 10000,
mean: 100,
median: 100

```
对于百分数相关的数据需要解释一下。以90为例。statsd会把一个flush期间上报的数据，去掉10%的峰值，即按大小取cnt*90%（四舍五入）个值来计算百分值。
举例说明，假如10s内上报以下10个值。
```
1,3,5,7,13,9,11,2,4,8
```
则只取10*90%=9个值，则去掉13。百分值即按剩下的9个值来计算。
```
$KEY.mean_90   // (1+3+5+7+9+2+11+4+8)/9
$KEY.upper_90  // 11
$KEY.lower_90  // 1
```

### 标量 gauge
gauge是任意的一维标量值。gague值不会像其它类型会在flush的时候清零，而是保持原有值。statsd只会将flush区间内最后一个值发到后端。另外，如果数值前加符号，会与前一个值累加。
```
age:10|g    // age 为 10
age:+1|g    // age 为 10 + 1 = 11
age:-1|g    // age为 11 - 1 = 10
age:5|g     // age为5,替代前一个值
```
### sets
记录flush期间，不重复的值。
```
request:1|s  // user 1
request:2|s  // user1 user2
request:1|s  // user1 user2
```
## statsd 客户端
statsd的客户端已经支持多种语言的实现,[参看列表](https://github.com/etsy/statsd/wiki)。nodejs相关有几个推荐的：lynx、node-statsd和node-statsd-client，使用都差不多，星也差不多。以(node-statsd-client)[https://github.com/msiebuhr/node-statsd-client]为例：
```
const SDC = require('statsd-client'),
const sdc = new SDC({ host: 'localhost', port: 8125 });

//counter
sdc.counter('cnt', 10, 0.1); // 100/0.1=1000
sdc.increment('cnt', 10); // +10
sdc.decrement('cnt', 10); // -10

//gauge
sdc.gauge('rpt', 100);
sdc.gaugeDelta('rpt', -10);  // -10

//sets
sdc.set('ips', '1');

//timer
sdc.timing('rpt', 200);

//close
sdc.close()
```
## 总结
- 基本原理：statsd是一个udp或tcp的守护进程。使用简单的行协议收集客户端的metic数据。statsd使用udp的好处。
- 安装及配置
- metric类型：counter、timer、gauge和sets。
- statsd的node客户端。

## 参考
[1] [StatsD Metric](http://www.jianshu.com/p/2b0aa5898dd7)
[2] [introduction-to-statsd](https://github.com/wyvernnot/introduction-to-statsd)
[3] [Counting & Timing](http://code.flickr.net/2008/10/27/counting-timing/)
[4] [Measure Anything, Measure Everything](https://codeascraft.com/2011/02/15/measure-anything-measure-everything/)
[5] [如果查看应用性能图表是一种信仰](https://segmentfault.com/a/1190000004132825)
[6] [Collecting Metrics Using StatsD, a Standard for Real-Time Monitoring](http://thenewstack.io/collecting-metrics-using-statsd-a-standard-for-real-time-monitoring/)

