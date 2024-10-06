---
author: zjmantou
title: 用 GitLab CI 进行持续集成
time: 2024-10-01 周二
tags:
  - CICD
---
## 简介

从 GitLab 8.0 开始，GitLab CI 就已经集成在 GitLab 中，我们只要在项目中添加一个 `.gitlab-ci.yml` 文件，然后添加一个 Runner，即可进行持续集成。 而且随着 GitLab 的升级，GitLab CI 变得越来越强大，本文将介绍如何使用 GitLab CI 进行持续集成。

## [](#%E4%B8%80%E4%BA%9B%E6%A6%82%E5%BF%B5 "#%E4%B8%80%E4%BA%9B%E6%A6%82%E5%BF%B5")一些概念

在介绍 GitLab CI 之前，我们先看看一些持续集成相关的概念。

## [](#pipeline "#pipeline")Pipeline

一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。 任何提交或者 Merge Request 的合并都可以触发 Pipeline，如下图所示：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202410011625719.png)


## [](#stages "#stages")Stages

Stages 表示构建阶段，说白了就是上面提到的流程。 我们可以在一次 Pipeline 中定义多个 Stages，这些 Stages 会有以下特点：

- 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始
- 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功
- 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败

因此，Stages 和 Pipeline 的关系就是：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202410011626641.png)


## [](#jobs "#jobs")Jobs

Jobs 表示构建工作，表示某个 Stage 里面执行的工作。 我们可以在 Stages 里面定义多个 Jobs，这些 Jobs 会有以下特点：

- 相同 Stage 中的 Jobs 会并行执行
- 相同 Stage 中的 Jobs 都执行成功时，该 Stage 才会成功
- 如果任何一个 Job 失败，那么该 Stage 失败，即该构建任务 (Pipeline) 失败

所以，Jobs 和 Stage 的关系图就是：

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202410011626185.png)

## [](#gitlab-runner "#gitlab-runner")GitLab Runner

## [](#%E7%AE%80%E4%BB%8B-1 "#%E7%AE%80%E4%BB%8B-1")简介

理解了上面的基本概念之后，有没有觉得少了些什么东西 —— 由谁来执行这些构建任务呢？ 答案就是 GitLab Runner 了！

想问为什么不是 GitLab CI 来运行那些构建任务？ 一般来说，构建任务都会占用很多的系统资源 (譬如编译代码)，而 GitLab CI 又是 GitLab 的一部分，如果由 GitLab CI 来运行构建任务的话，在执行构建任务的时候，GitLab 的性能会大幅下降。

GitLab CI 最大的作用是管理各个项目的构建状态，因此，运行构建任务这种浪费资源的事情就交给 GitLab Runner 来做拉！ 因为 GitLab Runner 可以安装到不同的机器上，所以在构建任务运行期间并不会影响到 GitLab 的性能~

## [](#%E5%AE%89%E8%A3%85 "#%E5%AE%89%E8%A3%85")安装

安装 GitLab Runner 太简单了，按照着 [官方文档](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner "https://gitlab.com/gitlab-org/gitlab-ci-multi-runner") 的教程来就好拉！ 下面是 Debian/Ubuntu/CentOS 的安装方法，其他系统去参考官方文档：

```bash
# For Debian/Ubuntu 
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash $ sudo apt-get install gitlab-ci-multi-runner 
# For CentOS 
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash $ sudo yum install gitlab-ci-multi-runner
```

## [](#%E6%B3%A8%E5%86%8C-runner "#%E6%B3%A8%E5%86%8C-runner")注册 Runner

安装好 GitLab Runner 之后，我们只要启动 Runner 然后和 CI 绑定就可以了：

- 打开你 GitLab 中的项目页面，在项目设置中找到 runners
- 运行 `sudo gitlab-ci-multi-runner register`
- 输入 CI URL
- 输入 Token
- 输入 Runner 的名字
- 选择 Runner 的类型，简单起见还是选 Shell 吧
- 完成

当注册好 Runner 之后，可以用 `sudo gitlab-ci-multi-runner list` 命令来查看各个 Runner 的状态：

```bash
$ sudo gitlab-runner list Listing configured runners ConfigFile=/etc/gitlab-runner/config.toml my-runner Executor=shell Token=cd1cd7cf243afb47094677855aacd3 URL=http://mygitlab.com/ci
```

## [](#%E7%AE%80%E4%BB%8B-2 "#%E7%AE%80%E4%BB%8B-2")简介

配置好 Runner 之后，我们要做的事情就是在项目根目录中添加 `.gitlab-ci.yml` 文件了。 当我们添加了 `.gitlab-ci.yml` 文件后，每次提交代码或者合并 MR 都会自动运行构建任务了。

还记得 Pipeline 是怎么触发的吗？Pipeline 也是通过提交代码或者合并 MR 来触发的！ 那么 Pipeline 和 `.gitlab-ci.yml` 有什么关系呢？ 其实 `.gitlab-ci.yml` 就是在定义 Pipeline 而已拉！

## [](#%E5%9F%BA%E6%9C%AC%E5%86%99%E6%B3%95 "#%E5%9F%BA%E6%9C%AC%E5%86%99%E6%B3%95")基本写法

我们先来看看 `.gitlab-ci.yml` 是怎么写的：

```YAML
# 定义 stages 
stages:
 - build
 - test 
# 定义 job 
job1: 
 stage: test 
 script:
  - echo "I am job1"
  - echo "I am in test stage" 
# 定义 job 
job2:
 stage: build
 script:
  - echo "I am job2"
  - echo "I am in build stage"
```


写起来很简单吧！用 `stages` 关键字来定义 Pipeline 中的各个构建阶段，然后用一些非关键字来定义 jobs。 每个 job 中可以可以再用 `stage` 关键字来指定该 job 对应哪个 stage。 job 里面的 `script` 关键字是最关键的地方了，也是每个 job 中必须要包含的，它表示每个 job 要执行的命令。

回想一下我们之前提到的 Stages 和 Jobs 的关系，然后猜猜上面例子的运行结果？

```Bash
I am job2 
I am in build stage 
I am job1 
I am in test stage
```

根据我们在 `stages` 中的定义，`build` 阶段要在 `test` 阶段之前运行，所以 `stage:build` 的 jobs 会先运行，之后才会运行 `stage:test` 的 jobs。

## [](#%E5%B8%B8%E7%94%A8%E7%9A%84%E5%85%B3%E9%94%AE%E5%AD%97 "#%E5%B8%B8%E7%94%A8%E7%9A%84%E5%85%B3%E9%94%AE%E5%AD%97")常用的关键字

下面介绍一些常用的关键字，想要更加详尽的内容请前往 [官方文档](http://docs.gitlab.com/ce/ci/yaml/README.html "http://docs.gitlab.com/ce/ci/yaml/README.html")

### [](#stages-1 "#stages-1")stages

定义 Stages，默认有三个 Stages，分别是 `build`, `test`, `deploy`。

### [](#types "#types")types

`stages` 的别名。

### [](#before-script "#before-script")before_script

定义任何 Jobs 运行前都会执行的命令。

### [](#after-script "#after-script")after_script

> 要求 GitLab 8.7+ 和 GitLab Runner 1.2+

定义任何 Jobs 运行完后都会执行的命令。

### [](#variables-job-variables "#variables-job-variables")variables && Job.variables

> 要求 GitLab Runner 0.5.0+

定义环境变量。 如果定义了 Job 级别的环境变量的话，该 Job 会优先使用 Job 级别的环境变量。

### [](#cache-job-cache "#cache-job-cache")cache && Job.cache

> 要求 GitLab Runner 0.7.0+

定义需要缓存的文件。 每个 Job 开始的时候，Runner 都会删掉 `.gitignore` 里面的文件。 如果有些文件 (如 `node_modules/`) 需要多个 Jobs 共用的话，我们只能让每个 Job 都先执行一遍 `npm install`。 这样很不方便，因此我们需要对这些文件进行缓存。缓存了的文件除了可以跨 Jobs 使用外，还可以跨 Pipeline 使用。

具体用法请查看 [官方文档](http://docs.gitlab.com/ce/ci/yaml/README.html#cache "http://docs.gitlab.com/ce/ci/yaml/README.html#cache")。

### [](#job-script "#job-script")Job.script

定义 Job 要运行的命令，必填项。

### [](#job-stage "#job-stage")Job.stage

定义 Job 的 stage，默认为 `test`。

### [](#job-artifacts "#job-artifacts")Job.artifacts

定义 Job 中生成的附件。 当该 Job 运行成功后，生成的文件可以作为附件 (如生成的二进制文件) 保留下来，打包发送到 GitLab，之后我们可以在 GitLab 的项目页面下下载该附件。 注意，不要把 `artifacts` 和 `cache` 混淆了。

## [](#%E5%AE%9E%E7%94%A8%E4%BE%8B%E5%AD%90 "#%E5%AE%9E%E7%94%A8%E4%BE%8B%E5%AD%90")实用例子

下面给出一个我自己在用的例子：

```YAML
stages:
  - install_deps
  - test
  - build
  - deploy_test
  - deploy_production

cache:
  key: ${CI_BUILD_REF_NAME}
  paths:
    - node_modules/
    - dist/


# 安装依赖
install_deps:
  stage: install_deps
  only:
    - develop
    - master
  script:
    - npm install


# 运行测试用例
test:
  stage: test
  only:
    - develop
    - master
  script:
    - npm run test


# 编译
build:
  stage: build
  only:
    - develop
    - master
  script:
    - npm run clean
    - npm run build:client
    - npm run build:server


# 部署测试服务器
deploy_test:
  stage: deploy_test
  only:
    - develop
  script:
    - pm2 delete app || true
    - pm2 start app.js --name app


# 部署生产服务器
deploy_production:
  stage: deploy_production
  only:
    - master
  script:
    - bash scripts/deploy/deploy.sh
```

上面的配置把一次 Pipeline 分成五个阶段：

- 安装依赖(`install_deps`)
- 运行测试(`test`)
- 编译(`build`)
- 部署测试服务器(`deploy_test`)
- 部署生产服务器(`deploy_production`)

设置 `Job.only` 后，只有当 develop 分支和 master 分支有提交的时候才会触发相关的 Jobs。 注意，我这里用 GitLab Runner 所在的服务器作为测试服务器。

## [](#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99 "#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99")参考资料

- [https://about.gitlab.com/gitlab-ci/](https://about.gitlab.com/gitlab-ci/ "https://about.gitlab.com/gitlab-ci/")
- [http://docs.gitlab.com/ce/ci/yaml/README.html](http://docs.gitlab.com/ce/ci/yaml/README.html "http://docs.gitlab.com/ce/ci/yaml/README.html")
- [http://docs.gitlab.com/ce/ci/variables/README.html](http://docs.gitlab.com/ce/ci/variables/README.html "http://docs.gitlab.com/ce/ci/variables/README.html")
- [https://gitlab.com/gitlab-org/gitlab-ci-multi-runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner "https://gitlab.com/gitlab-org/gitlab-ci-multi-runner")
- [https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/issues/1232](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/issues/1232 "https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/issues/1232")
- [http://stackbox.cn/2016-02-gitlab-ci-conf/](http://stackbox.cn/2016-02-gitlab-ci-conf/ "http://stackbox.cn/2016-02-gitlab-ci-conf/")