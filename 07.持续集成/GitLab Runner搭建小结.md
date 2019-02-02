---
title: GitLab Runner小结
comments: true
date: 2017-03-01 19:35:19
categories: gitlab
tags: gitlab CI
toc: true
---
[GitLab Runner](https://doc.gitlab.cc/runner/) 用来运行GitLab CI 任务（job），runner通过解析项目中的.gitlab-ci.yaml文件，来获取需要运行的job,并将结果反馈给GitLab。GitLab Runner 用Go语言实现，可以很方便在Linux，macOS 甚至windows上安装。官方也提供docker image，可以直接docker安装。
本文将介绍GitLab runner 的安装、配置以及过程中遇到的一些问题。通过安装的过程，我们也会进一步了解GitLab CI的工作原理。

## 安装
GitLab Runner 有多种安装方式：下载二进制文件、通过repository下载deb包安装 以及docker 安装。具体的安装方式请参考[官网安装教程](https://docs.gitlab.com/runner/install/index.html)。本人采用docker安装，使用docker安装不仅方便，而且能很方便的升级以及管理。
安装docker engine，参考[docker官网](https://docs.docker.com/engine/installation/)。
### 下载GitLab Runner镜像
```
$ docker pull gitlab/gitlab-runner
```
根据GitLab Runner的[Dockerfile](https://hub.docker.com/r/gitlab/gitlab-runner/~/dockerfile/)可知，安装GitLab Runner 大体做以下几件事：
- 安装gitlab-ci-multi-runner。
- 创建gitlab-runner用户
- 启动GitLab Runner
配置文件位于 /etc/gitlab-runner,工作目录/home/gitlab-runner。

### 运行GitLab Runner
启动 GitLab Runner 镜像时，需要挂载一个volume，用来存配置及其他的资源。
```
docker run -d --name gitlab-runner --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```
注意：GitLab Runner 可以使用多种 executor 来运行job。如果使用docker方式的executor，必须在启动镜像时加上：
```
-v /var/run/docker.sock:/var/run/docker.sock
```
-- restart 参数表示每次重启容器时，重新启动gitlab-runner 服务。

## 注册 runner
注册 Runner 就是把 Runner 绑定到一个 GitLab 实例的过程。Runner分为两种：shared runner 和 specific runner。
shared runner：可以为多个项目运行job，只需要在项目的runner中，设置 allow shared runner。
specific runner: 为某些特殊需求的项目运行任务。当然也可以应用在多个项目上，和shared runner不同的是，需要在每个项目中配置使用该runner。

关于注册runner可以参考[官方教程](https://doc.gitlab.cc/runner/register/index.html)。
可以直接运行如下命令。
```
docker exec -it gitlab-runner gitlab-runner register
```
也可以进入容器内操作。
注册runner需要根据是shared runner还是specfic runner来获取相应的token。具体的注册过程才考[官网教程](https://doc.gitlab.cc/runner/register/index.html)。
### 注册 shared runner
注册shared runner 需要admin权限。首先登陆admin账户，在admin/runner页面，得到token。

![|admin token|](http://obv0ef5sf.bkt.clouddn.com/admin_runner_token.png)

使用如下指令完成注册：
```
$ sudo gitlab-ci-multi-runner register
```

![注册runner](http://obv0ef5sf.bkt.clouddn.com/register_runner.png)

注册时，有以下几个iterm需要设置：
- **coordinate url**
gitlab安装的主机地址。
- **token**
admin runner的token
- **description**
runner的描述
- **tag**
runner 的tag
- **executor**
executor 是选择何种方式来运行任务。这里选择docker。
- **image**
executor的基础镜像，这样在.gitlab-ci.yml中就可以不写该image。
至此，就成功地创建了一个shared runner。

## 注册 specific runner
可以为项目注册specific runner。specific runner和shared runner的注册过程类似，不过需要在项目的runner tab下获取。

![project token](http://obv0ef5sf.bkt.clouddn.com/specific_runner.png)

这里需要说的是设置GitLab实例url的时候，一定可以访问的。如果是自定义域名，需要做一些特别的设置，在下面的配置说明中将做介绍。

## 配置
配置文件位于<code>/etc/gitlab-runner/config.toml</code>,也就是在宿主机的<code>/srv/gitlab-runner/config/config.toml</code>。修改配置可以进入容器修改，也可以修改宿主机的配置然后重启容器。建议使用后者。
具体的配置说明，参考官网的[配置文档](https://doc.gitlab.cc/runner/configuration/advanced-configuration.html)。这里只介绍几个比较重要且容易造成后面gitlab ci错误的配置。

### 私有仓库
当使用[docker executor](https://doc.gitlab.cc/runner/executors/docker.html)时，需要指定build过程中或service使用的镜像。这些镜像默认从docker hub中获取，这比较慢。我们通常会从[私人镜像库](https://doc.gitlab.cc/runner/executors/docker.html#define-an-image-from-a-private-docker-registry)中拉取。这个时候需要权限验证来拉取镜像。在.gitlab-ci.yml中配置肯定不方便，gitlab提供了一个<code>DOCKER_AUTH_CONFIG</code>的secret variable。配置过程可以[参考官网](https://doc.gitlab.cc/runner/configuration/advanced-configuration.html#using-a-private-container-registry)。
在runner的宿主机上，docker login登录。可以查看<code>~/.docker/config.json</code>中的配置，如下:
```
{
	"auths": {
		"registry.gezhiqiang.com": {
			"auth": "dGVzdDp0ZXN0MTIz"
		}
	}
}
```
在gitlab项目secret variable下添加DOCKER_AUTH_CONFIG变量，值为上边的值。这样在docker build的时候拉取私有镜像，就会使用上面的配置。
### 构建镜像
在执行完测试之后，我们通常会需要在gitlab中将项目打成image，上传到私有仓库。这个时候，我们就要在job中使用docker build 和 docker push等指令。官网提供[3种方法](https://docs.gitlab.com/ce/ci/docker/using_docker_build.html#use-docker-in-docker-executor)。这里只介绍其中两种。
#### 方法1：docker in docker
使用官方提供的docker in docker镜像[docker:dind](https://hub.docker.com/_/docker/)，并且使其运行在priviledged模式。可以在注册runner的时候，使用以下指令使其使用docker：dind并且设置priviledged模式。
```
sudo gitlab-runner register -n \
  --url https://gitlab.com/ \
  --registration-token REGISTRATION_TOKEN \
  --executor docker \
  --description "My Docker Runner" \
  --docker-image "docker:latest" \
  --docker-privileged
```
如果runner已经注册，可以修改runner的配置文件config.toml，如下：
```
[runners]]
  url = "https://gitlab.com/"
  token = TOKEN
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "docker:latest" // 指定docker镜像
    privileged = true       // priviledged 模式
    disable_cache = false
    volumes = ["/cache"]
  [runners.cache]
    Insecure = false
```
这样就可以在build的过程中使用docker，注意要使用docker:dind的service。
```
image: docker:latest

# When using dind, it's wise to use the overlayfs driver for
# improved performance.
variables:
  DOCKER_DRIVER: overlay2

services:
- docker:dind

before_script:
- docker info

build:
  stage: build
  script:
  - docker build -t my-docker-image .
  - docker run my-docker-image /script/to/run/tests
```
### 方法2：docker socket 绑定
方法2就是绑定<code>/var/run/docker.sock</code>到容器。可以按下面的方式注册runner。
```
sudo gitlab-runner register -n \
  --url https://gitlab.com/ \
  --registration-token REGISTRATION_TOKEN \
  --executor docker \
  --description "My Docker Runner" \
  --docker-image "docker:latest" \
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock // 绑定
```
或者修改配置文件:
```
[[runners]]
  url = "https://gitlab.com/"
  token = REGISTRATION_TOKEN
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "docker:latest"
    privileged = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"] //绑定socket
  [runners.cache]
    Insecure = false
```
这样就可以使用docker指令了。注意这里不需要使用docker:dind service了。
```
image: docker:latest

before_script:
- docker info

build:
  stage: build
  script:
  - docker build -t my-docker-image .
  - docker run my-docker-image /script/to/run/tests
```

docker in docker的方式，是官方推荐的方法，虽然privileged模式也有缺点。而绑定docker socket的方法，容器中使用的是runner的 Docker Daemon。这样如果有docker rm等操作，将会删除runnder中的镜像，非常不安全。所以选择使用docker in docker的方式。
这里细心的同学也发现了，在注册里使用<code>--docker-属性</code>,其实就是修改config.toml里的[runners.docker]里的[属性]的值。
```
--docker-image "docker:latest"
相当于
[runners.docker]
    image="docker:latest"
```
### 清除空间
由于gitlab runner会使用宿主机的docker下载一些镜像，以及会有一些缓存，长期使用所占的空间对于一个磁盘空间不足的机器来说是个问题，需要定时清理相应文件。可以使用自己写定时任务清理，也可以使用官方提供的[cleanup镜像](https://gitlab.com/gitlab-org/gitlab-runner-docker-cleanup)来定时清理。

```
docker run -d \
    -e LOW_FREE_SPACE=10G \
    -e EXPECTED_FREE_SPACE=20G \
    -e LOW_FREE_FILES_COUNT=1048576 \
    -e EXPECTED_FREE_FILES_COUNT=2097152 \
    -e DEFAULT_TTL=10m \
    -e USE_DF=1 \
    --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --name=gitlab-runner-docker-cleanup \
    quay.io/gitlab/gitlab-runner-docker-cleanup

```
详细的使用方法，参考[官网](https://gitlab.com/gitlab-org/gitlab-runner-docker-cleanup)。
## 更新GitLab Runner
拉取最新的镜像。
```
$ docker pull gitlab/gitlab-runner:latest
```
停止并删除当前container。
```
$ docker stop gitlab-runner && docker rm gitlab-runner
```
按之前的方式启动，注意volume的地址和之前一样。
```
docker run -d --name gitlab-runner --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```
## 参考
[1] [GitLab Runner 注册](https://doc.gitlab.cc/ce/ci/runners/README.html)
[2] [GitLab Runner中文文档](https://doc.gitlab.cc/runner/)
[3] [gitlab-runner-docker-cleanup](https://gitlab.com/gitlab-org/gitlab-runner-docker-cleanup)
