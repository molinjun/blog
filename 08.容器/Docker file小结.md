---
title: dockerfile 小结
comments: true
date: 2017-02-21 23:58:11
categories: docker
tags: docker dockefile
toc: true
---
使用docker，就少不了要经常打镜像，也自然熟悉通常放在项目根目录下的Dockerfile文件。
Docker是根据Dockerfile 和 context来构建镜像的，context即为指定路径下的需要打到镜像中的文件。build镜像的过程中首先会将完整的context发送给docker daemon。docker daemon再根据Dockerfile的指令，顺序执行，完成镜像的build。
<!--more-->
## 使用方法
通常docker build 根据项目根目录下的Dockerfile来执行。
```
$ docker build -t [name] .
```
也可以通过 -f 选项来指定Dockerfile的路径。
```
$ docker build -f /path/to/dockerfile -t [name] .
```
通常使用当前目录 . 来build镜像过，也可以通过[.dockerignore文件](https://docs.docker.com/engine/reference/builder/#dockerignore-file)来排除一些不要的文件。

dockerfile中使用 # 来注释，且只有行首的 # 号才有效。其他位置的 # 号将只会当做普通字符。

## 常用指令
详尽的指令介绍请查看[官网介绍](https://docs.docker.com/engine/reference/builder/#dockerignore-file)。这里只介绍一些常用的执行。
> 虽然指令不区分大小写，但是通常大写来区分与选项的不同。
### FROM
FROM指令用来指定基础镜像。Dockerfile必须将FROM指令作为第一条指令。
```
FROM <image>
FROM <image>:<tag>
FROM <image>@<digest>
```
如果没有指明tag或者digest，将会默认为latest。

### MAINTAINER
指定镜像的作者。
```
MAINTAINER Dennis.ge <gedennis@163.com>
```
不过该指令已经deprecated。建议使用label来代替。
```
label maintainer "gedennis@163.com"
```
### LABEL
LABEL指令用于添加元数据到镜像中。一个label其实就是一个键值对。如果值中包括空格，需要使用引号或者反破折号（\）。
```
LABEL version="1.0"
LABEL name="dennis ge"
```
如果有多个label，因为每个label都会增加一个层，所以建议将其合并为一行。
```
LABEL version="1.0" name="dennis ge"
```
### ENV
设置环境变量，有两种形式：
```
ENV <key> <value>
ENV key1=value1 key2=value2 ...
```
建议使用第二种方式，因为只会产生一个cache层。

### WORKDIR
WORKDIR指令用来设置工作目录。对RUN、CMD、COPY、ADD和ENTRYPOINT指令都有效。WORKDIR可以定义多次，指令中出现相对路径都是基于工作目录的。
```
WORKDIR /path/to/workdir
```
### ADD
ADD指令用于将文件、目录或者urls添加到镜像文件中。
```
ADD src dest
```
dest为绝对路径或者相对于WORKDIR的相对路径。

### COPY
COPY指令和ADD指令类似，也是用来添加文件。
```
COPY src dest
```
不过COPY和ADD还是有几点不同之处：
- ADD 支持src为url或者压缩包，并且会在添加过程中解压。
- COPY 会创建不存在的目录。
> 官方建议使用COPY，因为ADD权限可能越界。

### RUN
RUN指令用来执行命令。有两种形式：
- **shell形式**
```
RUN <command>
```
shell形式是将指令放在一个shell中执行，相当于
```
/bin/sh -c <command>
```
shell形式可以使用环境变量。
shell形式可以使用反破折号（\）来写多行指令。
```
RUN apt-get update && \
    apt-get install git
    ...
```
- **exec 形式**
```
RUN ["executable", "param1", "param2"...]
```
exec的方式，不会默认使用/bin/sh。而且因为exec形式会转化为json array，所以参数都是用双引号，而不能使用单引号。

### EXPOSE
EXPOSE指令用来设置container监听的端口。
```
EXPOSE <port1> [port2]
```
暴露端口，并不代表客户端能访问此端口，需要在docker run的时候，通过-p或者-P来指定端口映射。
### VOLUME
VOLUME用来指定数据卷的挂载点。可以是json array的形式，也可以是字符串。
```
VOLUME ["/etc/conf", "/var/db"] // json array
VOLUME /etc/conf /var/db
```

### CMD
CMD 指令用来设置镜像在启动为container时的一些默认选项（运行指令或者参数）。通常CMD用来指定启动容器后执行的指令。当然也可以使用CMD和ENTRYPOINT配合使用，CMD 为ENTRYPOINT指定默认的参数。
CMD有三种形式：
- **exec形式**
```
CMD ["executable", "param1", "param2"...]
```
- **shell 形式**
```
CMD executable param1 param2
```
- 参数形式
这种方式只能配合ENTRYPOINT使用。
```
CMD ["param1", "param2"]
```
配合ENTRYPOINT使用时，两种指令都只能用json array的方式。如果每次启动容器都执行相同的命令，可以考虑使用ENTRYPOINT。对于CMD指令，官方建议使用exec的形式。
> Dockerfile中只应有一条CMD指令，多条也只有最后一条有效。

### ENTRYPOINT
ENTRYPOINT将会把container 配置为一个可执行文件。ENTRYPOINT也有两种形式：
- **exec形式**
```
ENTRYPOINT ["executable", "param1", "param2"...]
```
docker run 时的命令行参数会被追加到exec 形式的参数后面，并会覆盖CMD指定的默认参数。ENTRYPOINT 不会被命令行参数覆盖，除非使用--entrypoint选项。
- **shell形式**
```
ENTRYPOINT command param1 param2
```
shell形式，所有CMD和run时的命令行参数都会失效。不过，ENTRYPOINT指定的命令将会作为/bin/sh -c 的子命令来执行。
### RUN CMD 和 ENTRYPOINT 区别
这三个指令从某种意义来讲，都是用来执行命令的，而且都有两种形式：shell形式和exec形式，所以经常容易被混淆。
- RUN指令和其它两个指令有本质上的区别：
RUN是用来构建镜像时用的，而CMD和ENTRYPOINT是启动容器时指定执行的命令。
- CMD是用来指定默认参数的，所以使用只用CMD指定的指令，将会被docker run的命令行参数覆盖。
- ENTRYPOINT不会被会命令行参数覆盖。exec形式时，只是会将其追加到ENTRYPOINT指定的参数后面，而shell形式时，命令行参数和CMD指令都将失效，但其是通过/bin/sh -c 运行，所以启动的可执行文件PID不是1，所以收不到unix 的信号，无法通过docker stop停止。所以**ENTRYPOINT 不建议使用shell形式，建议使用exec形式**。

**使用建议**
- RUN 当需要用环境变量的时候，用shell的形式，其他尽量用exec的形式。
- 如果不需要将容器配置为可执行文件，也不需要预处理一些操作，且用可能通过docker run 指定运行指令时，可使用CMD的方式，且最好使用exec的形式。
- 建议使用ENTRYPOINT和CMD配合的方式。ENTRYPOINT 指定一个指令，然后CMD来指定默认的参数。
- 推荐一种方法：使用ENTRYPOINT来指定一个shell脚本，可以预处理一些环境，如根据环境变量替换配置文件等，然后再shell里指定需要执行的指令。
```
## Dockerfile
ENTRYPOINT ['/entrypoint.sh']
CMD ["executable", "param1"]

## entrypoint.sh
#!/bin/bash
# do something
exec "#@"  // 将会执行参数命令，即为CMD指定的命令
```
## dockerfile的基本准则
- 尽量是容器短小精悍。
- 合理利用.dockerignore 文件来排除不许要的文件。
- 避免安装不必要的包。
- 一个容器一个进程。
- 使层的数量尽可能少。
- 有效地利用cache来减少重复的build。

## 总结
dockerfile是build镜像的基础。关于dockerfile有以下几点内容：
- dockerfile的使用
- 熟悉dockerfile 中常用的指令，以及区分ADD和COPY，RUN、CMD和ENTRYPOINT指令的别。
- dockerfile的一些书写技巧，比如怎么利用好cache提高打镜像的效率等。

## 参考
[1] [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
[2] [Best practices for writing Dockerfiles](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#/build-cache)
[3] [Docker RUN vs CMD vs ENTRYPOINT](http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/)
[4] [Docker学习笔记—DockerFile及命令](https://segmentfault.com/a/1190000007182608)
[5] [快速掌握dockerfile](https://segmentfault.com/a/1190000006186977)
[6] [跟我一起学Docker——RUN、ENTRYPOINT与CMD](https://www.binss.me/blog/learn-docker-with-me-about-run-entrypoint-and-cmd/)
