---
title: ssh配置
comments: true
date: 2016-11-25 15:33:24
categories: ssh
tags: ssh github
toc: true
---
SSH(Secure SHell)是一种网络安全协议，用于计算机的加密登录。这种方式的登录，即使被截获，密码也不会泄露。在github上的项目，下载和push等，都需要在github账户设置ssh key。
下面就简单介绍主机的ssh key生成方法。
<!--more-->
### 生成ssh key
终端输入如下指令：
```bash
$ ssh-keygen -t rsa -b 4096 -C "你的邮箱"
```
下面的步骤，会要输入密码等，可以选择一路默认。如果需要特别配置，参考具体手册。
这样，就会在用户目录.ssh文件夹下生成ssh key文件。
```
id_rsa.pub   公钥文件
id_rsa       私钥文件
```
### 使用ssh key
在github的用户设定中，加入ssh key。
```bash
$ cat ~/.ssh/id_rsa.pub
```
然后将输入的公钥复制到github。这样，就可以下载github上的repository了。

---
**参考**
[Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#platform-linux)
[SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
