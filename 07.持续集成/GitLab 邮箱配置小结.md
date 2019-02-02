---
title: gitlab 邮箱配置小结
comments: true
date: 2017-03-05 01:37:30
categories: gitlab
tags: gitlab
toc: true
---
在前两篇文章中，介绍了GitLab CE的安装以及一些常用的配置。这篇文章将会介绍关于邮箱的配置。因为GitLab中涉及的邮箱过多，且容易混淆，而且学习的过程中有需要的疑惑和容易出错的地方，因此就单列一篇来介绍邮箱的配置，以便后续查阅。
GitLab 中有两种邮箱的配置：系统邮箱配置和用户邮箱设置。系统邮箱用来为系统给用户发送一些系统邮件，而用户邮箱则用来接收系统邮件，例如代码合并、CI测试结果通知等。
下面将会将我遇到的问题以及整理的结果一一讲解。
## 系统邮箱设置
系统邮箱即为GitLab CE安装的时候配置的邮箱，用来给用户发送系统邮件。如果安装了sendmail或者postfix等邮箱，则可配置相应的通用邮箱的设置即可，因为我是使用官方的docker镜像安装的，所以并没有安装sendmail和postman，所以需要配置SMTP来启用邮箱功能。
## 通用邮箱设置
不管是使用sendmail或者SMTP都需要做如下的配置。按之前的方式，进入容器，修改配置文件。不清楚的可以查看{%post_link gitlab-ce-install %}。
```
//进入容器
$ docker exec -it gitlab bash
// 修改配置文件
$ vim /etc/gitlab/gitlab.rb
```
大体有以下几项配置：
```
### Email Settings
# gitlab_rails['gitlab_email_enabled'] = true //是否开启系统邮箱，默认开启
# gitlab_rails['gitlab_email_from'] = 'example@example.com'
// 系统邮箱发件人
# gitlab_rails['gitlab_email_display_name'] = 'Example'
// 显示的发件人名称，默认GitLab
# gitlab_rails['gitlab_email_reply_to'] = 'noreply@example.com'
// 设置收件邮箱
# gitlab_rails['gitlab_email_subject_suffix'] = ''
// 邮件标题后缀
```
gitlab_email_from 用来设置发件人的邮箱,如果是配置SMTP，这个邮箱必须和校验的邮箱一致，后续会再提到。其他的几项可以按默认设置,可以按自己的需求自定义。收件邮箱我觉得需求不大，就不配置了。
## SMTP 设置
如果是docker安装，需要配置SMTP。关于SMTP以及关于IMTP和POP3的邮箱具体知识，可参考[这篇文章](https://www.zhihu.com/question/24605584)。

### 官方配置模板
SMTP的配置，无非就是配置一个邮件客户端，用来发送邮件。官方提供默认的配置模板，以及一些常用邮箱服务商的配置细节，可具体参考[官方配置](https://doc.gitlab.cc/omnibus/settings/smtp.html)。
以下是默认的官方配置项。
```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.server"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "smtp user"
gitlab_rails['smtp_password'] = "smtp password"
gitlab_rails['smtp_domain'] = "example.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'peer'

# 如果你使用的SMTP服务是默认的 'From:gitlab@localhost'
# 你可以修改这里的 'From' 的值。
gitlab_rails['gitlab_email_from'] = 'gitlab@example.com'
gitlab_rails['gitlab_email_reply_to'] = 'noreply@example.com'
```
### 163邮箱配置
由于国内我常用网易邮箱，而官网并没提供配置，所以特别提下。以下配置模板适合126和163邮箱。用户根据填入自己的邮箱名和邮箱密码即可。
```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.126.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "xxx@126.com"
gitlab_rails['smtp_password'] = "xxx"
gitlab_rails['smtp_domain'] = "126.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
```
### 常见问题及解决方法
这里有几个特别容易需要错误的地方，需要注意。

- 配置的密码不是邮箱的登陆密码而是网易邮箱的客户端授权密码, 在网易邮箱web页面的设置-POP3/SMTP/IMAP-客户端授权密码查看。

![客户端密码配置](http://obv0ef5sf.bkt.clouddn.com/email-pwd-setting.png)


- 出现下面的文件未抵达的错误。
```
WARN: EOFError: end of file reached
```
这种错误是使用默认配置时，用的465端口。而126、163邮箱用的端口为25。
- 出现未授权账户错误
```
WARN: Net::SMTPFatalError: 553 Mail from must equal authorized user
```
这个错误可参考[官方解释](http://www.mail163.cn/fault/analysis/1109.html)。原因是网易服务器smtp机器要求身份验证帐号和发信帐号必须一致，如果用户在发送邮件时，身份验证帐号和发件人帐号是不同的，因此拒绝发送。刚才的SMTP配置IDE邮箱即为身份验证账号，而通用配置中的gitlab_email_from，即为发信账号，要保证这两个账号一致。
综合配置如下：
```
#Sending application email via SMTP
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "xxuser@163.com"
gitlab_rails['smtp_password'] = "xxpassword"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = :login
gitlab_rails['smtp_enable_starttls_auto'] = true

##修改gitlab配置的发信人
gitlab_rails['gitlab_email_from'] = "xxuser@163.com"
```
通过以上配置，系统即可发送邮箱了。可以通过测试命令测试配置成功。
```
// 进入容器
$ docker exec -it gitlab bash

// 进入控制台
$ gitlab-rails console

// 发送测试邮件
$ Notify.test_email('收件人邮箱', '邮件标题', '邮件正文').deliver_now

```
这是即可收到系统邮件，配置成功！！

## 用户邮箱
以上是系统邮箱的配置，而各个用户又有多个邮箱配置。
起初这些邮箱让我很摸不到头脑，注册邮箱、primary email、notification email、public email 、代码仓库中配置的邮箱还有ssh key中的邮箱。这些都有啥区别，GitLab又是怎么来使用和区分用户的呢？
首先进入用户有几个提到邮箱的地方。点击用户右上角的下拉图标，点击【设置】，进入用户设置页。

![用户设置页](http://obv0ef5sf.bkt.clouddn.com/gitlab-emails.png)

【profile】下有一个邮箱和一个public邮箱。【Emails】下可以添加邮箱，并且显示当前用户邮箱的用途。【Notification】下有用户的通知邮箱。
如图给出了一些解释：
```
- Your Primary Email will be used for avatar detection and web based operations, such as edits and merges.
Primary Email用来做头像检测以及一些基于web的操作，如编辑和merge等。

- Your Notification Email will be used for account notifications.
通知邮箱用来获取用户的通知。

- Your Public Email will be displayed on your public profile.
public email是展示用户的邮件信息。

- All email addresses will be used to identify your commits.
```
虽然模糊理解了意思，但还是很模糊。所以通过几个实验，得出以下结论。
### 各邮箱的区别
**primary email**
primary email即为用户注册时的邮箱，默认状况下，通知邮箱也是同一个，这个可以在用户设置页的【email】tab下看到。同时这个primary email也是【profile】tab下的email项。
这个邮箱是可以修改的，在【profile】的Email，输入新的email。此时系统会发送确认邮件给新邮箱（注意已经配置好了系统邮件），用户在邮箱中点击确认，即完成修改。此时primary邮箱即为新的邮箱了。
这个邮箱也会用来检测头像以及用户的提交。其实通常大家也只是使用一个邮箱来兼任各个功能。

**notification email**
这个邮箱用来接收一些系统邮件，如果测试结果、merge、review结果等。如果没有特别需求，通常设置和primary email一致。

**public email**
public email是用户展示出来的联系方式，可自行设置，GitLab并不使用。
## 恢复真相
现在介绍一下这几个邮箱是如何工作的。
### ssh key
其实都不太关人家的事，只不过设置公钥的时候是用的这个邮箱。用户添加了公钥，就可以在这个公钥的主机上push和pull代码。用户可以添加多个公钥，对应在多台机器上工作。用户的commit后，系统鉴别的头像和用户名都和ssh key中的邮箱没有关系。

### 用户的邮箱
如上所以，用户可设置三种邮箱，通常设为一致即可。用户也可以在email中添加多个邮箱。
### git配置
这个配置的邮箱，才至关重要。
用户在clone代码之后，需要设置git信息：邮箱和用户名。
```
$ git config user.name 'dennis.ge'
$ git config user.email 'gedennis@163.com'
```
如果代码中为设置，则使用全局配置。
```
$ cat ~/.gitconfig
```
用户提交代码之后，GitLab会将git config 中的邮箱，与用户email中的邮箱匹配。如果在用户的邮箱列表中，则可鉴定为这个用户。那么这个用户的commit的头像，即为用户设置的头像，否则为系统生成头像。
> 值得注意的是：当邮箱匹配时，commit的用户名，即为用户的GitLab的用户名，而不是git config中的名称。如果邮箱不匹配，则使用git config 中的用户名和邮箱。

所以，如果只需要单机工作，添加一个邮箱三用即可。如果需要多机工作，git config只要匹配用户的一个邮箱即可。

## 总结
本文主要介绍了GitLab中的一个邮箱配置。
- 系统邮箱 如何配置SMTP以及注意如客户端密码等常见错误。
- 用户邮箱 如何区分各种邮箱以及各邮箱的作用以及如何协调工作。

## 参考
[1] [SMTP邮箱配置](https://doc.gitlab.cc/omnibus/settings/smtp.html)
[2] [553 mail from must equal authorized user](http://www.mail163.cn/fault/analysis/1109.html)
[3] [GitLab 配置通过 smtp.163.com 发送邮件](https://ruby-china.org/topics/20450)
[4] [POP3, SMTP, IMAP 和 Exchange 的区别在哪里？](https://www.zhihu.com/question/24605584)
