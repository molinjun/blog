---
title: GitLab CI 配置小结 
comments: true
date: 2017-12-03 01:40:24
categories: gitlab
tags: gitlab ci
---

Gitlab提供可持续集成服务。只要在项目根目录创建一个.gitlab-ci.yml文件，并为这个项目制定一个runner，当有合并请求或者push的时候就会触发build。快速上手，请参考[官网教程](https://doc.gitlab.cc/ce/ci/quick_start/README.html).
本文将介绍写gitlab-ci.yml中一些常用的功能。

## 基础模板
GitLab使用YAML文件来配置项目的build，且这个文件放在项目的根目录。
下面是一个用于nodejs项目的gitlab-ci最基础的配置模板：
```
image: node:<版本号>

services:
    - mongo:<版本>
    - redis:<版本号>

cache:
    key: <KEY>
    paths:
        - <需要缓存的路径>
        - node_modules

before_script:
    - <执行job之前的命令>

after_script:
    - <build完之后运行的命令>

stages:
    - test
    - build
    - deploy

<job_name>:
    script:
        - <job执行的指令或脚本>
    only：
        - <只应用在某个分支>
    except:
        - <不应用到某个分支>
    variables:

```
## 基础指令
### image 和 service
image和service用来指定job中使用的基础镜像，以及需要的附加服务，如mongo和redis等。image和service也可以在注册runner时指定。
```
sudo gitlab-runner register \
  --url "https://gitlab.example.com/" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --description "docker-ruby-2.1" \
  --executor "docker" \
  --docker-image node:9 \
  --docker-mongo latest \
  --docker-redis latest
```
相当于在config.toml中有如下配置：
```
[runners.docker]
  image = "node:9"
  services = ["mongo:latest", "redis:latest"]
```
上面注册的runner将会使用node:9 image，并且会运行mongo:latest 和 redis:latest 里那个service。
这里需要了解一下service的连接方法。例如你在.gitlab-ci.yml中使用 tutum/wordpress镜像，就会创建一个容器，link到job容器。这样job容器中就可以通过hostname <code>mongo</code>连接MongoDB。关于hostname需要特别注意。如下定义MongoDB的镜像,包含/。
```
services:
- dennis/mongo:latest
```
这个时候有两个可用的hostname: <code>dennis-mongo</code> 和<code> dennis__mongo</code>。这里有几条规则。
```
- 冒号：后面的tag都会被忽略。
- slash被替换为__,作为primary alias。
- slash替换为-，作为secondary alias。
```
这种方式多少容易忘记，容易混淆。最方便的方式还是使用alias的方式。
```
services:
- name: mongo:latest
  alias: db-mongo
```
### before_script 和 after_script
before_script用来定义在所有jobs开始之前运行的指令。例如项目中使用了submodule，可以先更新子模块。
```
before_script:
  - if which git >/dev/null; then
      git submodule sync --recursive;
      git submodule update --init --recursive;
    fi
```
after_scirpt用来指定在所有builds完成后执行的指令。

### stages
stages在job中定义build的各个阶段。stages指定的顺序决定了它的执行顺序。
- 同一个stage中的元素并行执行。
- 下一个stage必须在上一stage任务全部执行完之后执行。

```
stages:
  - build
  - test
  - deploy
```
### variables
可以添加变量到job的系统环境。variable通常定义一些非敏感信息，对于敏感数据需要放在secret varialbes。
```
variables:
  DATABASE_URL: "postgres://postgres@postgres/my_database"
```
### cache
cache用来指定builds之间进行缓存的一组文件或文件夹。指定的文件夹必须在项目目录。
```
 cache:
    key: "$CI_BUILD_REF_NAME"
    paths:
      - node_modules/
```
可以为cache指定不同的key，来限制cache的级别。

### jobs
job就是真正要执行的任务。每个job都必须有个唯一的名称，并且必须指定至少一个script指令。
```
job_name:
  script:
    - npm i
    - npm run test
  stage: test
  only:
    - master
  except:
    - develop
  tags:
    - node
  allow_failure: true
```
### only 和 except
only和except用来限制需要执行该任务的分支。这个指令非常重要，特别是对于部署任务，例如develop分支部署到测试环境，master部署到生产环境。也可以使用通配符。
### when
when用于指定执行该任务的条件。非常有用，例如部署生产环境，采用manual更为稳妥。
```
- on_success: 前面的任务全部成功
- on_failure: 前面的任务出现错误
- always： 任何时候执行
- manual：增加手动执行按钮。
```

## anchors 锚记
对于一些重复的script，可以使用锚记。
```
.job_template: &job_definition  # Hidden key that defines an anchor named 'job_definition'
  image: ruby:2.1
  services:
    - postgres
    - redis

test1:
  <<: *job_definition           # Merge the contents of the 'job_definition' alias
  script:
    - test1 project

test2:
  <<: *job_definition           # Merge the contents of the 'job_definition' alias
  script:
```
通过<code>.</code>来定义锚记。 <code>$</code>用来指定锚记的名称。<code><<</code>是指把后面的指令插入到当前位置。<code>*</code>用来引入锚记名称，类似于指针。


## 总结
本文介绍了如何配置.gitlab-ci.yml指令以及给了一些模板。配置文件的编写在gitlab ci是非常重要的一环，好的配置文件也会提高效率减少消耗，希望大家还是多参考官方文档。

## 参考
[1] [Configuration of your jobs with .gitlab-ci.yml](https://docs.gitlab.com.cn/ce/ci/yaml/README.html#anchors)
[2] [use docker image](https://docs.gitlab.com.cn/ce/ci/docker/using_docker_images.html#available-settings-for-services-entry)
