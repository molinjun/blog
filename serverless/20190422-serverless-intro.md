<!-- TOC -->

- [Serverless 实践初探](#serverless-实践初探)
  - [核心概念](#核心概念)
  - [基本用法](#基本用法)
    - [环境配置](#环境配置)
    - [创建一个 service](#创建一个-service)
    - [部署服务](#部署服务)
    - [删除服务](#删除服务)

<!-- /TOC -->

<a id="markdown-serverless-实践初探" name="serverless-实践初探"></a>

## Serverless 实践初探

> 注意： 这里我们介绍的是 serverless CLI 工具，而不是 serverless 框架。

[Serverless](https://serverless.com/framework/docs/) 是一个 CLI 工具，允许用户构建和部署自动伸缩(auto-scaling)、按执行付费(pay-per-execution)、事件驱动 (event-driven) 的 function 服务。 Serverless 依赖于不同的云提供商 (cloud provider), 下面主要以 AWS 为例。更详细的文档，请参考官网[基于 AWS 的开发文档](https://serverless.com/framework/docs/providers/aws/guide/intro/)。

<a id="markdown-核心概念" name="核心概念"></a>

### 核心概念

Serverless 框架就是用来帮助我们开发和部署我们的 `AWS Lambda functions`。它区别于别的框架的地方在于，它不仅需要管理我们的业务代码，还要管理我们业务代码所需要用到的基础设施 (infrastructure)， 例如 DynamoDB，S3 等等。以下是一些核心概念。

- Functions  
  这里说的 Function 是指 AWS Lambda function。它是一个独立的部署单元，类似于微服务。每个 Function 相对单一的任务，例如保存一个用户到数据库。不建议在一个 Function 中执行多个任务。
- Events  
  Event 是指任何触发 AWS Lambda function 执行的事件。例如一个 AWS API Gateway 的 http 请求等。一旦在 serverless 框架里定义了一个 function 的 event，serverless 都会自动创建所需要的 infrastructure。
- Resources
  Resources 是指 function 所需要的基础设施，例如 DynamoDB Table，S3 Bucket 等。
- Services  
  Service 是框架的组织单元，类似于一个项目文件，通过它来配置 function、event 以及所使用的 resources。通常放在 `serverless.yml` 文件中。
- Plugins  
  可以在配置文件中加一些 `Plugin 插件`来扩展一些功能。

<a id="markdown-基本用法" name="基本用法"></a>

### 基本用法

<a id="markdown-环境配置" name="环境配置"></a>

#### 环境配置

系统需要安装 `node` 以及配置好 AWS CLI。然后安装 `serverless`。

```
$ npm i -g serverless
```

建议在项目中，都在本地安装，并保持本地安装的版本和 `serverless`配置中的一致。

```
$ npm i -D serverless
```

验证安装成功。

```
$ serverless --version
```

<a id="markdown-创建一个-service" name="创建一个-service"></a>

#### 创建一个 service

使用 nodejs 模板创建一个服务。

```
$  serverless create --template aws-nodejs --path serverless-demo
```

serverless 支持多种不同的模板，也可以创建基于 typescript 的模板。

```
$ serverless create --template aws-nodejs-typescript --path typescript-demo
```

<a id="markdown-部署服务" name="部署服务"></a>

#### 部署服务

指定 test 这个 profile 来部署。

```
$ serverless deploy --aws-profile test -v
```

平时开发如果只更新 function, 可以使用下面的指令，省的花时间更新所有的 infrastructure。

```
$ serverless deploy function -f hello
```

调用服务, 并打印日志。

```
$ serverless invoke -f hello --aws-profile test -l
```

使用以下指令，可以查看实时日志。

```
$ serverless logs -f hello --aws-profile test -t
```

<a id="markdown-删除服务" name="删除服务"></a>

#### 删除服务

删除服务及相关的所有资源。

```
$ serverless remove --aws-profile test
```

以上是简单的实践，具体参看[官网文档](https://serverless.com/framework/docs/providers/aws/guide/intro/)。
