---
title: GitLab CE 搭建小结
comments: true
date: 2017-03-01 19:35:00
categories: gitlab
tags: gitlab git
toc: true
---
项目的开发通常都离不开对代码的版本管理。简单的方式可以在内网搭建一个仓库，然后添加各个组员的公钥来共同开发。这种方式不仅不利于管理和维护，而且功能过于单一。我们很希望有像GitHub这样的平台服务，功能齐全且好维护。但由于GFW的原因，有时候访问延迟过大。更重要的是，github免费版只支持开源项目，私有项目需要付费，而且比较昂贵，并不适合公司的项目。
[GitLab](https://doc.gitlab.cc/) 是一个类似与GitHub的项目，功能十分强大且界面美观，支持代码管理、issue管理、代码review和CI等功能。它提供免费的社区版和付费版，社区版足够满足我们的项目需求。本篇文章我将介绍GitLab社区版的安装、配置以及一些遇到的问题。
## 安装
GitLab CE的源码安装十分麻烦，需要安装和配置的东西太多，就不过多在环境搭建上折腾了。可以选择官方提供的 [Omnibus GitLab](https://doc.gitlab.cc/omnibus/README.html) 一键安装包或者使用docker安装。
安装GitLab有一些硬件要求。官方建议：2 核 4G内存 5-10G的硬盘存储，不然可能会出现一些问题。
### Omnibus 包安装
[Omnibus GitLab](https://doc.gitlab.cc/omnibus/README.html) 是官方提供的一键安装包，集成了GitLab以及所需的服务和依赖。官方提供不同linux发行版的[安装方法](https://www.gitlab.cc/downloads/)。这里以ubuntu 14.04 为例。
#### 安装依赖
```
$ sudo apt-get install curl openssh-server ca-certificates postfix
```
postfix 用来发送邮件，在安装期间选择【Internet Site】。也可以使用sendmail或者[配置SMTP服务](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md)并[使用SMTP发送邮件](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md#smtp-on-localhost)。
#### 添加GitLab仓库并安装
```
$ curl -sS http://packages.gitlab.cc/install/gitlab-ce/script.deb.sh | sudo bash
$ sudo apt-get install gitlab-ce
```
也可以下载deb包安装。
```
$ curl -LJO https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/trusty/main/g/gitlab-ce/gitlab-ce-<版本>.deb
$ dpkg -i gitlab-ce-<版本>.deb
```
#### 启动GitLab
```
$ sudo gitlab-ctl reconfigure
```

### docker安装
使用[docker安装](https://doc.gitlab.cc/omnibus/docker/README.html)更方便，且易于升级管理。
确保系统安装了[docker](https://docs.docker.com/engine/installation/),并拉取[GitLab官方镜像](https://hub.docker.com/r/gitlab/gitlab-ce/)。
```
$ docker pull gitlab/gitlab-ce
```
启动GitLab 容器。
```
sudo docker run --detach \ 
    --hostname gitlab.example.com \
    --publish 443:443 \
    --publish 1226:80 \
    --publish 23:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```
关于启动参数：
```
--detach   设置容器后台运行
--hostname 设置容器的hostname
--publish  暴露 https、http和ssh端口
--name     容器名称
--restart always  每次启动容器就重启GitLab
--volume 设置GitLab数据挂载点
```
这里把GitLab的相关数据目录，都挂载到宿主机上的/srv/gitlab目录。
```
/srv/gitlab/data    应用程序数据
/srv/gitlab/logs    GitLab的log
/srv/gitlab/config  GitLab的配置文件
```
> 这里需要注意的是，如果以前安装过，需要把以前的volume删除。

### 访问GitLab
安装完成之后，可以通过浏览器访问GitLab的web界面, 如上面docker启动的gitlab.example.com:1226。第一次访问GitLab的时候，需要设置管理员的密码。设置完成之后，即可登陆进入主界面。

> 用户名和密码可以做相应修改。默认用户名为root。


![首次登陆GitLab](http://obv0ef5sf.bkt.clouddn.com/gitlab-first-login.png "首次登陆GitLab")

到此，就可以创建自己的组(group)和项目(project)了。但是，现在用户pull和push代码还有些问题，需要做进一步的设置。下面以docker安装的gitlab的配置为例，其它方式安装可以参考或者直接看官网文档。

## 配置
Omnibus GitLab的配置文件位于 /etc/gitlab/gitlab.rb。可以通过进入容器，修改配置做相应的自定义设置。
```
$ docker exec -it gitlab bash
$ vim /etc/gitlab/gitlab.rb
```
或者
```
$ docker exec -it gitlab vim /etc/gitlab/gitlab.rb
$ docker restart gitlab
```
### 基础设置
用户可以点击右上角图标的下拉菜单，选择【settings】做一些基础设置。

![user setting](http://obv0ef5sf.bkt.clouddn.com/gitlab-user-setting.png)

用户可以设置自己的用户名、密码、邮箱等信息，同时可以设置web的主题等等。但最重要的是设置自己的SSH Keys，这样才能正常的push和pull代码。
使用现有的公钥。
```
$ cat ~/.ssh/id_rsa.pub
```
或者创建新的ssh keys。更多参考{% post_link ssh-key %}。
```
$ ssh-keygen -t rsa -b 4096 -C "你的邮箱"
```
然后复制，并添加到GitLab中。

### external_url 设置
[external_url](https://doc.gitlab.cc/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab)是很重要的一环。GitLab的资源如图片、git仓库地址都是基于这个url。
默认的设置为：
```
external_url 'GENERATED_EXTERNAL_URL'
// 默认 http://hostname
```
这个值默认是基于hostname的。需要把external_url设置为一个可访问的主机域名或ip地址。
项目的仓库地址为：
```
// http
http://<external_url>:<port>/<组名>/<项目名>.git

// ssh
git@<external_url>:<组名>/<项目名>.git
```
可以根据自己情况修改external_url。可以直接设置成ip。
```
external_url 'http://10.2.237.56'
```
> 注意：external_url 必须带如http:// 的协议头。

这里也可以使用一个自定义的域名，但是需要每个人添加hosts。设置之后，应用配置。
```
$ sudo gitlab-ctl reconfigure
```
### 端口设置
修改了external_url 之后，clone代码的时候还是提示需要用户名密码。明明已经添加了ssh key ，莫非哪里打开方式不对？原来是容器启动的时候，ssh端口映射为23，而不是默认的22。
GitLab 容器默认使用了以下端口：
```
- 80  http端口
- 433 https端口
- 8080 Unicorn端口
- 22   ssh 端口
```
因为这几个端口在宿主机经常使用，所以需要更改端口。修改/etc/gitlab/gitlab.rb 配置文件。
#### 修改external_url
如果是http或者https端口，需要修改external_url。
```
// http
external_url "http://gitlab.example.com:1226" //启动时使用1226端口

// https
external_url "https://gitlab.example.com:1227" //启动时使用1227端口
```
#### 修改 ssh 端口
如果是ssh端口，需要修改gitlab_shell_ssh_port。
```
// 使用23作为ssh端口
gitlab_rails['gitlab_shell_ssh_port'] = 23
```
重启配置生效。
```
$ sudo gitlab-ctl reconfigure
```
此时项目的仓库地址就变了。如果ssh端口地址不是默认的22，就会加上ssh:// 协议头。
```bash
# 22端口
git@<external_url>:<组名>/<项目名>.git

# 非22端口
ssh://git@<external_url>:port/<组名>/<项目名>.git
```
如果需要设置http为域名，ssh为ip的话，可以通过gitlab_rails['gitlab_ssh_host']设置ssh 的host。
```
// 设置external_url 和 http端口
external_url 'http://gitlab.example.com:1226

// 设置ssh host
gitlab_rails['gitlab_ssh_host'] = '10.2.123.123'
// 设置ssh 端口
gitlab_rails['gitlab_shell_ssh_port'] = 23
```
这里有个坑。按上面设置external_url之后，居然访问不了，百思不得其解啊，都是按官网设置的。后来才发现一条，关于[nginx的设置](https://doc.gitlab.cc/omnibus/settings/nginx.html#setting-the-nginx-listen-port)。原来默认下，nginx监听的端口为external_url中定义的，或者默认的80/443。所以刚nginx是监听的1226端口。而docker run的时候是暴露的80端口，所以根本就访问不了。
修改nginx的监听端口，一切ok。
```
# nginx['listen_port'] = nil
nginx['listen_port'] = 80
```
至此，就可以顺利的通过项目下的仓库地址来pull代码了。

### 预配置
启动docker容器，然后进入容器修改配置，这种方式也没啥毛病，但如果想提前设置配置项，gitlab也提供了一个环境变量 GITLAB_OMNIBUS_CONFIG 来实现。
```
sudo docker run --detach \
    --hostname gitlab.example.com \
    --env GITLAB_OMNIBUS_CONFIG="external_url 'http://gitlab.example.com'; gitlab_rails['gitlab_shell_ssh_portv'] = '23';" \
    --publish 443:443 \
    --publish 1226:80 \
    --publish 23:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```
这个 GITLAB_OMNIBUS_CONFIG 环境变量设置的值并不会写入到gitlab.rb配置文件，只是在启动时加载，所以每次启动都需要带上这个启动项。其实我觉得并不是很方便。

如果安装了docker-compose的话，启动会方便一些，只需要配置一份docker-compose.yml文件，以后都用该文件启动。以下是一个docker-compose.yml的模板。
```
gitlab:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.dennis.com'
    container_name: 'gitlab-ce'
    environment:
        GITLAB_OMNIBUS_CONFIG: |
            external_url 'http://gitlab.dennis.com:1226'
            gitlab_rails['gitlab_shell_ssh_port'] = 23
            nginx['listen_port'] = 80
            gitlab_rails['smtp_enable'] = true
            gitlab_rails['smtp_address'] = "smtp.126.com"
            gitlab_rails['smtp_port'] = 25
            gitlab_rails['smtp_user_name'] = "*@126.com"
            gitlab_rails['smtp_password'] = "**"
            gitlab_rails['smtp_domain'] = "126.com"
            gitlab_rails['gitlab_email_from'] = "*@126.com"
    ports:
        - '443:443'
        - '1226:80'
        - '23:22'
    volumes:
        - '/srv/gitlab/config:/etc/gitlab'
        - '/srv/gitlab/logs:/var/log/gitlab'
        - '/srv/gitlab/data:/var/opt/gitlab'
```
这里通过环境变量设置了external_url和ssh 端口。
之后在docker-compose.yml所在文件目录，启动GitLab。
```
$ docker-compose up -d
```

## 常用功能
GitLab有需要不错的功能，下面列举几个。
### 通告
如果需要升级或者别的需求，需要通知到每个用户，可以设置通告。仅限管理员使用。

![admin-message](http://obv0ef5sf.bkt.clouddn.com/gitlab-admin-message.png)

管理员点击右上角的小把手，然后点击【message】的tab,然后再输入通告内容和展示的开始时间和结束时间即可。

### 登录首页
默认的登录首页是关于GitLab CE的一段描述。当然可以根据自己的需求自定义。
登录管理员账户，进入【Admin Area】,点击右上角的配置图标，点击下拉菜单的【Appearance】,然后输入标题、内容以及logo等信息即可。

![登录首页修改](http://obv0ef5sf.bkt.clouddn.com/gitlab-login-page.png)

描述的内容可以使用markdown的语法。有个两个logo
的设置。page的logo是显示在页面上的图标。header log是显示在顶端的一个logo。
设置后的界面如下图所示：
http://obv0ef5sf.bkt.clouddn.com/updated-login-page.png
![修改后的登录页面](http://obv0ef5sf.bkt.clouddn.com/updated-login-page.png)
## 升级
使用docker的方式，升级也很方便。
首先关闭正在运行的gitlab。
```
$ docker stop gitlab
$ docker rm gitlab
```
再拉取最新的gitlab-ce镜像。
```
$ docker pull gitlab/gitlab-ce:latest
```
最后，在按之前的方式重启gitlab。
```
sudo docker run --detach \
--hostname gitlab.example.com \
--publish 443:443 \
--publish 1226:80 \
--publish 23:22 \
--name gitlab \
--restart always \
--volume /srv/gitlab/config:/etc/gitlab \
--volume /srv/gitlab/logs:/var/log/gitlab \
--volume /srv/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```
如果是通过docker-compose的方式，则通过
```
$ docker-compose pull
```
拉取最新的镜像，然后重启。
```
$ docker-compose up -d
```
## 总结
本文大体有以下内容：
- GitLab的安装方式：可以直接安装，也可以使用docker安装，建议使用后者。
- 配置：介绍了基于docker安装的一些简单配置，如external_url、ssh端口等等，以及使用 GITLAB_OMNIBUS_CONFIG环境变量或者使用docker-compose的方式提前设置相关项启动。
- 小功能：介绍一些GitLab的小功能，如通过等。
- 升级：介绍GitLab的升级方法。

通过以上介绍的方法，基本就可以使用GitLab了。当然，这只是一些最基础的配置，可以深入学习官方文档做更细化的配置。后续将会介绍GitLab CI的实践方法。
## 参考
[1] [Omnibus GitLab documentation ](https://doc.gitlab.cc/omnibus/README.html)
[2] [Configuration options 参数配置](https://doc.gitlab.cc/omnibus/settings/configuration.html#configuration-options)
[3] [NGINX settings ](https://doc.gitlab.cc/omnibus/settings/nginx.html#nginx-settings)
