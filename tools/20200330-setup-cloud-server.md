# 搭建自己的云服务器

## 购买服务器

首先需要到[阿里云官网](https://www.aliyun.com)选择一台适合自己配置的 ECS 主机。阿里云的主机并不算便宜, 及时关注官网的优惠活动。如果没有阿里云账号，可以注册一个新号，实名认证一下，可以享受折扣不错的新人优惠。
选择自己的 CPU 和内核等配置购买以后，控制台下的 ECS 主机可以看到运行的实例，你可以得到一个公网 IP 。
默认的主机的安全组只打开了 22 等几个端口，我们需要根据自己的情况，打开几个常用端口，如 80，443 等。在控制台下，【云服务器】-【网络与安全】- 【安全组】。点击并进入已有的安全组，然后添加安全组规则。

## 连接服务器

服务器启动之后，我们需要本地连接服务器。默认我们需要通过 root 用户登录。第一次需要在控制台【重置实例密码】，然后通过 ssh 登录。

```bash
$ ssh root@<your-ip>
```

输入刚才重置的密码，成功登录服务器。不过，平时直接使用 root 用户不够安全，我们新建一个带 sudo 权限的用户。

### sudo 用户密码登录

#### 创建新用户

root 登录到服务器之后，我们创建一个新用户。

```
$ adduer molin
```

配置密码等相关信息，就成功创建了用户，并在 home 创建了相应的工作目录。
这里如果你使用 useradd 指令就会相对复杂点，需要配置 shell 等信息，我顺便列一下指令。

```
1. 创建用户，并创建home目录
$ useradd -m molin

2. 设置密码
$ passwd molin

3. 设置shell
$ usermod -s /bin/bash molin

或者：
$ useradd -m -s "/bin/bash" molin
```

#### 添加 sudo 权限

此时用户并没有 sudo 权限。

```
molin is not in the sudoers file.  This incident will be reported.
```

我们需要修改`/etc/sudoers`文件，添加 sudo 权限。

```
root ALL=(ALL:ALL) ALL
molin ALL=(ALL:ALL) ALL // 添加这行，molin替代为您的用户名
```

此时就可以使用刚创建的用户，使用密码登录了。

### 配置 ssh 免密登录

每次使用密码登录还是比较麻烦，我们可以配置 ssh 登录。查看本地电脑是否已经配置 ssh key。

```
$ ls ~/.ssh
```

如果没有公私钥文件`id_rsa` 和 `id_rsa.pub`, 需要先生成。

```
$ ssh-keygen -t rsa -b 4096 -C "你的邮箱"
```

下面的步骤，会要输入密码等，可以选择一路默认。 这样，就会在用户目录.ssh 文件夹下生成 ssh key 文件。

```
id_rsa.pub   公钥文件
id_rsa       私钥文件
```

有了 ssh key 之后，就可以将公钥加到服务器 ssh 文件夹下的`authorized_keys`文件中。

```
$ ssh-copy-id -i ~/.ssh/id_rsa.pub user@服务器ip
```

至此，就可以通过 ssh 免密登录了。

## 开发环境配置

### 安装 git

```
$ sudo apt-get update
$ sudo apt-get install git
```

### 安装 Nodejs

我们使用[nvm](https://github.com/nvm-sh/nvm)，这样利于 node 的版本管理。

```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

执行完脚本就成功安装了 nvm。需要重新进入 terminal 或者直接`bash` 指令，使 nvm 生效。
然后使用 nvm 安装制定版本的 node。

```
$ nvm install v8.10.0
```

然后验证安装。

```
$ node -v
$ npm -v
```

### 安装 Docker(选装)

Docker 不是必须安装的。我因为习惯使用 Docker 来跑一些服务，如 MongoDB、ELK 等，这里就简单介绍一下 Docker 的安装。
可以到[Docker 官网](https://docs.docker.com/install/linux/docker-ce/ubuntu/)根据自己的系统，一步步安装。这里以 ubuntu 16.04 为例。

#### 确保 APT 支持 https 和 CA 证书

```
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

#### 添加 GPG key

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository \
 "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
 \$(lsb_release -cs) \
 stable"
```

#### 安装 Docker CE

```
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

至此 Docker 安装成功，但是通常你的用户不是 root 用户，需要添加到 Docker 组。

```
$ sudo usermod -aG docker <your-user>
```

重新进入 terminal，就可以使用 docker 了。

### 安装 nginx

通常我们需要 nginx 做反向代理以及托管页面，所以我们先安装 nginx。可以到[nginx 官网](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)根据自己的操作系统及版本选择安装方式。这里以 ubuntu 16.04 为例。
在`/etc/apt/sources.list.d/`目录下新建文件`nginx.list`, 文件中添加源。

```

deb https://nginx.org/packages/ubuntu/ xenial nginx
deb-src https://nginx.org/packages/ubuntu/ xenial nginx

```

然后开始安装 nginx。

```

$ sudo apt-get update
$ sudo apt-get install nginx

```

如果安装中出现如下错误：

```

W: GPG error: https://nginx.org/packages/ubuntu xenial Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY \$key

```

则将添加`$key`, 再安装。

```

// $key 为报错中的NO_PUBKEY 后的值
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $key
$ sudo apt-get update
\$ sudo apt-get install nginx

```

至此，nginx 已安装成功。可查询安装的 nginx 版本验证安装。

```

\$ nginx -v

```

安装完成之后需要启动 nginx。

```

// 启动
$ sudo service nginx start /
// 停止
$ sudo service nginx stop
// 重启
// sudo service nginx restart
```

nginx 的一些关键目录。

```
配置文件： /etc/nginx/ningx.conf
error log: /var/log/nginx/error.log
http log: /var/log/nginx/access.log
模块: /usr/lib/nginx/modules
```

我们可以在浏览器打开上面的公网 ip 地址，可以看到 nginx 的欢迎界面。

> 如果不能访问，可能是忘记添加主机的安全组规则，打开 80 端口。

## 域名与备案

### 购买域名

上面我们还是使用 ip 来访问我们的服务，很不方面。现在购买一个域名还是挺实惠的，这里我们来购买并配置一个自己的域名。
控制台下，【云服务】-【域名服务】-【域名注册】，进入域名选购页面。可以选择一个可用，并经济允许的域名购买。域名尽量选择`.com`等常用域名。

### 配置解析记录

购买完成后，进入域名列表页，添加解析记录。

```
www	  A	 公网IP
```

添加一条 A 记录，将域名指向我们的云主机公网 ip。此时我们就可以通过我们的域名访问我们的服务了。

### 备案

内地的主机现在都需要备案，可以直接使用阿里云备案提交资料快速备案。这里我们建议使用阿里云 app 快速备案。备案过程需要填写个人资料，提交身份证，网站名称及类型，并提交相应资料进行备案。备案完成后，我们就可以得到我们的 ICP 备案号。

> 这里个人经验貌似域名购买之后，需要 2-3 天之后才能备案。

## SSL 证书

阿里云提供免费的个人 DV 证书，但是只包含主域名，不包含子域名，这肯定不方便。所以我们使用 LetsEncrypt kj 自签一个通配符 SSL 证书。

### 生成 SSL 通配符证书

### 配置 nginx https
