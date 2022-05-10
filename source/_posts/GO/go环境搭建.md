---
title: go环境搭建
#tags: [hexo建站,hexo部署,github部署,个人博客]      #添加的标签
categories: 
  - GO
description: 
#cover: 
---



## go中gRPC的环境搭建

首先需要安装 gRPC golang版本的软件包。 安装官方安装命令：

```text
go get google.golang.org/grpc
```

是会报错误的，正确的安装方式：

```text
# 下载grpc-go
git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc
# 下载golang/net
git clone https://github.com/golang/net.git $GOPATH/src/golang.org/x/net
# 下载golang/text
git clone https://github.com/golang/text.git $GOPATH/src/golang.org/x/text
# 下载sys
git clone https://github.com/golang/sys.git $GOPATH/src/golang.org/x/sys
# 下载go-genproto
git clone https://github.com/google/go-genproto.git $GOPATH/src/google.golang.org/genproto
# 下载go-protobuf
git clone https://github.com/protocolbuffers/protobuf-go.git  $GOPATH/src/google.golang.org/protobuf
cd $GOPATH/src/
go install google.golang.org/grpc
```



### 安装protoc编译器

protoc 用于编译 protocolbuf (.proto文件) 和 protobuf 运行时。

### 下载

安装编译器最简单的方式是去[protobuf仓库地址](https://link.zhihu.com/?target=https%3A//github.com/protocolbuffers/protobuf/releases)下载预编译好的 protoc 二进制文件，仓库中可以找到每个平台对应的编译器二进制文件。

解压后进入bin文件夹中，window下，找到可执行文件：protoc.exe 。然后将它放入到$PATH中，可以在环境变量中添加当前路径到path的方式实现。也可以复制这个exe到已有的path中。比如我就是直接将其复制到了go的安装目录的/bin下。

linux下，

新增环境变量

```text
$ vim /etc/profile 
```

增加以下内容

```text
#记得改成自己的路径
export PATH=$PATH:/data/jbchen5/src/protoc-3.18.0-linux-x86_64/bin
```

使其生效

```text
$ source /etc/profile
```

最后查看是否成功

```text
$ protoc --version
libprotoc 3.18.0
```



### 安装Go Plugins（protoc-gen-go）

除了安装 protoc 之外还需要安装各个语言对应的编译插件，这里我们用的Go 语言，所以还需要安装一个 Go 语言的编译插件。

```text
git clone https://github.com/golang/protobuf.git  $GOPATH/src/github.com/golang/protobuf
cd $GOPATH/src/
go install github.com/golang/protobuf/protoc-gen-go/
```

> 编译器插件 protoc-gen-go 将安装在默认位于GOPATH/bin下面，请务必将GOPATH/bin/protoc-gen-go拷贝到/usr/bin下面，或添加GOPATH/bin真实路径到PATH中。