# Dockerfile 配置小结

当我们需要把我们的项目代码构建镜像时， 我们需要用到 `Dockerfile` 文件。 Dockerfile 指定了一组在构建镜像时，需要执行的指令。
Docker 是根据 Dockerfile 和 context 来构建镜像的，context 即为指定路径下的需要打到镜像中的文件。build 镜像的过程中首先会将完整的 context 发送给 docker daemon。docker daemon 再根据 Dockerfile 的指令，顺序执行，完成镜像的 build。

## 使用方法

通常 `docker build` 根据项目根目录下的 Dockerfile 来执行。

```
$ docker build -t [name] .
```

也可以通过 -f 选项来指定 Dockerfile 的路径。

```
$ docker build -f /path/to/dockerfile -t [name] .
```

通常使用当前目录 . 来 build 镜像过，也可以通过 [.dockerignore 文件](https://docs.docker.com/engine/reference/builder/#dockerignore-file) 来排除一些不要的文件。

dockerfile 中使用 # 来注释，且只有行首的 # 号才有效。其他位置的 # 号将只会当做普通字符。

## 常用指令

详尽的指令介绍请查看[官网介绍](https://docs.docker.com/engine/reference/builder/#dockerignore-file)。这里只介绍一些常用的执行。

> 虽然指令不区分大小写，但是通常大写来区分与选项的不同。

### FROM

FROM 指令用来指定基础镜像。Dockerfile 必须将 FROM 指令作为第一条指令。

```
FROM <image>
FROM <image>:<tag>
FROM <image>@<digest>
```

如果没有指明 tag 或者 digest，将会默认为 latest。

### MAINTAINER

指定镜像的作者。

```
MAINTAINER Dennis.ge <gedennis@163.com>
```

不过该指令已经 deprecated。建议使用 label 来代替。

```
label maintainer "gedennis@163.com"
```

### LABEL

LABEL 指令用于添加元数据到镜像中。一个 label 其实就是一个键值对。如果值中包括空格，需要使用引号或者反破折号（\）。

```
LABEL version="1.0"
LABEL name="dennis ge"
```

如果有多个 label，因为每个 label 都会增加一个层，所以建议将其合并为一行。

```
LABEL version="1.0" name="dennis ge"
```

### ENV

设置环境变量，有两种形式：

```
ENV <key> <value>
ENV key1=value1 key2=value2 ...
```

建议使用第二种方式，因为只会产生一个 cache 层。

### WORKDIR

WORKDIR 指令用来设置工作目录。对 RUN、CMD、COPY、ADD 和 ENTRYPOINT 指令都有效。WORKDIR 可以定义多次，指令中出现相对路径都是基于工作目录的。

```
WORKDIR /path/to/workdir
```

### ADD

ADD 指令用于将文件、目录或者 urls 添加到镜像文件中。

```
ADD src dest
```

dest 为绝对路径或者相对于 WORKDIR 的相对路径。

### COPY

COPY 指令和 ADD 指令类似，也是用来添加文件。

```
COPY src dest
```

不过 COPY 和 ADD 还是有几点不同之处：

- ADD 支持 src 为 url 或者压缩包，并且会在添加过程中解压。
- COPY 会创建不存在的目录。
  > 官方建议使用 COPY，因为 ADD 权限可能越界。

### RUN

RUN 指令用来执行命令。有两种形式：

- **shell 形式**

```
RUN <command>
```

shell 形式是将指令放在一个 shell 中执行，相当于

```
/bin/sh -c <command>
```

shell 形式可以使用环境变量。
shell 形式可以使用反破折号（\）来写多行指令。

```
RUN apt-get update && \
    apt-get install git
    ...
```

- **exec 形式**

```
RUN ["executable", "param1", "param2"...]
```

exec 的方式，不会默认使用/bin/sh。而且因为 exec 形式会转化为 json array，所以参数都是用双引号，而不能使用单引号。

> 问题：何为 json array，为何不能用单引号。

### EXPOSE

EXPOSE 指令用来设置 container 监听的端口。

```
EXPOSE <port1> [port2]
```

暴露端口，并不代表客户端能访问此端口，需要在 docker run 的时候，通过-p 或者-P 来指定端口映射。

### VOLUME

VOLUME 用来指定数据卷的挂载点。可以是 json array 的形式，也可以是字符串。

```
VOLUME ["/etc/conf", "/var/db"] // json array
VOLUME /etc/conf /var/db
```

### CMD

CMD 指令用来设置镜像在启动为 container 时的一些默认选项（运行指令或者参数）。通常 CMD 用来指定启动容器后执行的指令。当然也可以使用 CMD 和 ENTRYPOINT 配合使用，CMD 为 ENTRYPOINT 指定默认的参数。
CMD 有三种形式：

- **exec 形式**

```
CMD ["executable", "param1", "param2"...]
```

- **shell 形式**

```
CMD executable param1 param2
```

- 参数形式
  这种方式只能配合 ENTRYPOINT 使用。

```
CMD ["param1", "param2"]
```

配合 ENTRYPOINT 使用时，两种指令都只能用 json array 的方式。如果每次启动容器都执行相同的命令，可以考虑使用 ENTRYPOINT。对于 CMD 指令，官方建议使用 exec 的形式。

> Dockerfile 中只应有一条 CMD 指令，多条也只有最后一条有效。

### ENTRYPOINT

ENTRYPOINT 将会把 container 配置为一个可执行文件。ENTRYPOINT 也有两种形式：

- **exec 形式**

```
ENTRYPOINT ["executable", "param1", "param2"...]
```

docker run 时的命令行参数会被追加到 exec 形式的参数后面，并会覆盖 CMD 指定的默认参数。ENTRYPOINT 不会被命令行参数覆盖，除非使用 `--entrypoint` 选项。

- **shell 形式**

```
ENTRYPOINT command param1 param2
```

shell 形式，所有 CMD 和 run 时的命令行参数都会失效。不过，ENTRYPOINT 指定的命令将会作为/bin/sh -c 的子命令来执行。

### RUN CMD 和 ENTRYPOINT 区别

这三个指令从某种意义来讲，都是用来执行命令的，而且都有两种形式：shell 形式和 exec 形式，所以经常容易被混淆。

- RUN 指令和其它两个指令有本质上的区别：  
  RUN 是用来构建镜像时用的，而 CMD 和 ENTRYPOINT 是启动容器时指定执行的命令。
- CMD 是用来指定默认参数的，所以使用只用 CMD 指定的指令，将会被 docker run 的命令行参数覆盖。
- ENTRYPOINT 不会被会命令行参数覆盖。exec 形式时，只是会将其追加到 ENTRYPOINT 指定的参数后面，而 shell 形式时，命令行参数和 CMD 指令都将失效，但其是通过/bin/sh -c 运行，所以启动的可执行文件 PID 不是 1，所以收不到 unix 的信号，无法通过 docker stop 停止。所以**ENTRYPOINT 不建议使用 shell 形式，建议使用 exec 形式**。

**使用建议**

- RUN 当需要用环境变量的时候，用 shell 的形式，其他尽量用 exec 的形式。
- 如果不需要将容器配置为可执行文件，也不需要预处理一些操作，且用可能通过 docker run 指定运行指令时，可使用 CMD 的方式，且最好使用 exec 的形式。
- 建议使用 ENTRYPOINT 和 CMD 配合的方式。ENTRYPOINT 指定一个指令，然后 CMD 来指定默认的参数。
- 推荐一种方法：使用 ENTRYPOINT 来指定一个 shell 脚本，可以预处理一些环境，如根据环境变量替换配置文件等，然后再 shell 里指定需要执行的指令。

```
## Dockerfile
ENTRYPOINT ['/entrypoint.sh']
CMD ["executable", "param1"]

## entrypoint.sh
#!/bin/bash
# do something
exec "#@"  // 将会执行参数命令，即为CMD指定的命令
```

## dockerfile 的基本准则

- 尽量是容器短小精悍。
- 合理利用.dockerignore 文件来排除不许要的文件。
- 避免安装不必要的包。
- 一个容器一个进程。
- 使层的数量尽可能少。
- 有效地利用 cache 来减少重复的 build。

## 总结

Dockerfile 是 build 镜像的基础。关于 Dockerfile 有以下几点内容：

- Dockerfile 的使用
- 熟悉 Dockerfile 中常用的指令，以及区分 ADD 和 COPY，RUN、CMD 和 ENTRYPOINT 指令的别。
- Dockerfile 的一些书写技巧，比如怎么利用好 cache 提高打镜像的效率等。

## 参考

[1][dockerfile reference](https://docs.docker.com/engine/reference/builder/)  
[2][best practices for writing dockerfiles](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#/build-cache)  
[3][docker run vs cmd vs entrypoint](http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/)  
[4][docker学习笔记—dockerfile及命令](https://segmentfault.com/a/1190000007182608)  
[5][快速掌握dockerfile](https://segmentfault.com/a/1190000006186977)  
[6][跟我一起学docker——run、entrypoint与cmd](https://www.binss.me/blog/learn-docker-with-me-about-run-entrypoint-and-cmd/)
