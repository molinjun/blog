<!-- TOC -->

- [AWS 命令行界面 (AWS CLI)](#aws-命令行界面-aws-cli)
  - [安装](#安装)
    - [下载程序](#下载程序)
    - [解压](#解压)
    - [运行安装](#运行安装)
    - [验证安装](#验证安装)
  - [配置](#配置)
  - [使用 AWS CLI](#使用-aws-cli)
  - [卸载](#卸载)

<!-- /TOC -->

<a id="markdown-aws-命令行界面-aws-cli" name="aws-命令行界面-aws-cli"></a>

## AWS 命令行界面 (AWS CLI)

[AWS 命令行界面 (AWS Command Line Interface, AWS CLI)](https://aws.amazon.com/cn/cli/#file_commands_anchor) 是用于管理 AWS 服务的统一工具。只通过一个工具进行下载和配置，您可以使用命令行控制多个 AWS 服务并利用脚本来自动执行这些服务。

<a id="markdown-安装" name="安装"></a>

### 安装

参考[官网安装说明](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-install.html)。以 [macOS 安装 AWS CLI](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/install-bundle.html) 为例。

<a id="markdown-下载程序" name="下载程序"></a>

#### 下载程序

```
$ curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
```

<a id="markdown-解压" name="解压"></a>

#### 解压

```
$ unzip awscli-bundle.zip
```

<a id="markdown-运行安装" name="运行安装"></a>

#### 运行安装

```
$ sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
```

安装程序在`/usr/local/aws` 中安装 AWS CLI，并在 `/usr/local/bin` 目录中创建符号链接 `aws`。使用 -b 选项创建符号链接将免除在用户的 \$PATH 变量中指定安装目录的需要。这应该能让所有用户通过在任何目录下键入 aws 来调用 AWS CLI。

<a id="markdown-验证安装" name="验证安装"></a>

#### 验证安装

```
$ aws --version
```

<a id="markdown-配置" name="配置"></a>

### 配置

AWS 要求所有传入的请求都进行加密签名。AWS CLI 为您执行该操作。所以要配置 AWS CLI 在与 AWS 交互时使用的设置，包括您的安全凭证、默认输出格式和默认 AWS 区域。官网教程参考[这里](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-configure.html)。
四个参数：

- 访问密钥 Access Key ID
- Secret Access Key
- AWS 区域 (region)
- 输出格式

AWS Access Key ID 和 AWS Secret Access Key 是 AWS 凭证(credential)。在创建的 [IAM](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/getting-started_create-admin-group.html) 用户信息里可以获取到。  
Region 是默认情况下您要将请求发送到的服务器所在的 AWS 区域，通常是离的最近的服务器，如 `us-east-1`。  
输出格式支持：

- json
- text
- table

参考如下配置：

```
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

如果要设置多个 credential, 可以用 `--pfofile` 参数。

```
$ aws configure --profile user1
```

生成的文件位于 `~/.aws`，有两个文件：

- ~/.aws/credentials 凭证信息
- ~/.aws/config 配置信息

<a id="markdown-使用-aws-cli" name="使用-aws-cli"></a>

### 使用 AWS CLI

命令结构：

```
$ aws <command> <subcommand> [options and parameters]
```

与各种 AWS 服务的交互指令，请参考[这个列表](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-services.html)。

AWS CLI 提供了命令补全功能，使用 Tab 键自动补全指令。根据 shell 类型，将 `aws-completer` 所在目录加到 PATH 中。具体参考[这里](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-configure-completion.html)。下面以 zsh 为例, 在 `.zshrc` 文件中加入。

```
export PATH=/usr/local/aws/bin:$PATH
```

使配置生效并启用:

```
$ source ~/.zshrc
$ source /usr/local/aws/bin/aws_zsh_completer.sh
```

<a id="markdown-卸载" name="卸载"></a>

### 卸载

除了可选的符号链接之外，捆绑安装程序不会将任何内容放在安装目录之外，所以卸载十分简单，就是直接删除这两个项目。

```
$ sudo rm -rf /usr/local/aws
$ sudo rm /usr/local/bin/aws
$ rm -rf ~/.aws 删除credentials
```
