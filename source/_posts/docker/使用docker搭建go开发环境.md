---
title: 使用docker搭建go开发环境
#tags: [hexo建站,hexo部署,github部署,个人博客]      #添加的标签
categories: docker                           #添加的分类
description: 
#cover: 
---



## 为什么使用docker

- 可以保持系统软件环境的纯净。
- 开发环境和当前使用系统不再强依赖。无论系统再怎么升级，都不会再影响到我的开发环境。
- 开发软件的管理方式更加统一。各种编程语言都有各自的安装流程和步骤，各种应用服务的安装和配置方式也千差万别。通过 Docker，从一个更高的维度抽象和统一了这些差异。不论是 MySQL，还是 Redis，我都只需要拉镜像，映射端口，然后启动容器就行了。



## 搭建流程

### 测试镜像

使用docker来构建开发环境的第一步就是先找镜像。在 docker hub 上就有 Go 语言的官方镜像：**golang**，先用下面的命令测试一下这个镜像。

`docker run --rm golang:alpine go version`

- 这条命令以 `go` 为分界线，前面的部分属于 docker，表示执行 **golang** 镜像容器。 `alpine` 是这个镜像的标签，我个人喜欢 **alpine** 的小巧，所以选择的这个。
- `--rm` 参数表示执行完成后就删除容器，避免浪费存储空间。



### 执行go代码

上面的 go 命令虽然可以执行了，不过还不能运行 go 代码文件。因为还没提供目录挂载功能。自定义的 Shell 脚本内容如下：

```shell
#!/bin/bash

docker run --rm \
    -v $PWD:/srv/app \
    -w /srv/app \
    golang:alpine myTest
```
- myTest是编译好的go程序。
- `-v` Docker 挂载目录的参数。`$PWD` 代表当前执行命令的位置，`/srv/app` 是镜像容器中的目录。
- `-w` Docker 设置容器运行时的工作主目录。这主要是为了配合上面的 `-v` 参数来执行当前目录下的 go 代码。
