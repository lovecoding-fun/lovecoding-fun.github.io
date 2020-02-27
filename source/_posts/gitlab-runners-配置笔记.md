---
title: gitlab runners 配置笔记
catalog: true
date: 2020-02-27 22:46:42
subtitle:
header-img:
tags: FE
---

runners 用于跑 `.gitlab-ci.yml` 代码，有 shared runners 、specific runners 、group runners 三种类型，关于这三种类型的区别可以查看官方文档这一[部分](https://docs.gitlab.com/ee/ci/runners/README.html)。
之前想迁移文档到 gitlab pages 上时因为没有找到可用的 shared runners，决定在自己的 Linux 虚拟机上试着注册一个，文档上的[步骤](https://docs.gitlab.com/runner/register/)写得很详细，照着做就 OK 了。

### Install the runner
根据操作系统类型选择的是这个[链接](https://docs.gitlab.com/runner/install/linux-repository.html)。

添加 gitlab 的官方仓库：
```
// add GitLab’s official repository
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
```

安装最新版本的 gitlab runner ：
```
// install the latest version of GitLab Runner
sudo apt-get install gitlab-runner
```

### Obtain a token
在项目的 **Setting > CI/CD > Runners > Specific Runners** 找到 url 和 token，下一步注册时需要用到。

### Register the runner
Linux 上的注册方法见此[链接](https://docs.gitlab.com/runner/register/index.html)。

开始注册：
```
sudo gitlab-runner register
```

输入第二步中的 url 和 token ：
```
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com

Please enter the gitlab-ci token for this runner
xxx
```

简单描述一下该 runner 的信息：
```
Please enter the gitlab-ci description for this runner
[hostname] my-runner
```

定义 tags ，这个 tags 就是我们在 `.gitlab-ci.yml` 中为 jobs 指定的 tags ：
```
Please enter the gitlab-ci tags for this runner (comma separated):
my-tag,another-tag
```

选择执行器，选择的是 docker 。注意电脑上必须先安装好 docker ：
```
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
docker
```

为 docker 选择默认镜像，因为做的是 JS 项目，所以选择 node 镜像：
```
Please enter the Docker image (eg. ruby:2.6):
node:11.7.0
```

### Start Runner
注册完成后，就可以运行了：
```
gitlab-runner start
```
如果在 **Setting > CI/CD > Runners > Specific Runners** 看到 runner 实例并且显示 "activated" ，证明已经成功注册并启动了，接下来就可以开始愉快地跑 pipeline 了。

更多命令介绍可以参考这个[链接](https://docs.gitlab.com/runner/commands/)。