+++
title = 'Grpc'
date = '2025-04-08T19:05:47+08:00'
author = 'jianlu'
draft = true
description = "desc"
aliases = ["", ""]
weight = 10
+++

<!--
    // 注意设置go环境变量中的架构和系统为本地系统，然后安装如下两个插件
    // go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
    // go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
    // 检查安装情况
    // protoc-gen-go --version
    // protoc-gen-go-grpc --version
    // 运行生成代码
    // protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative cross/proto/message.proto

    https://gitcode.com/gh_mirrors/gr/grpc-opentracing

    https://blog.csdn.net/Mr_XiMu/article/details/124937025#:~:text=%E6%9C%AC%E6%96%87%E8%AF%A6%E7%BB%86%E4%BB%8B%E7%BB%8D%E4%BA%86%E5%A6%82%E4%BD%95%E5%9C%A8gRPC%E4%B8%AD%E5%AE%9E%E7%8E%B0TLS%E5%8D%95%E5%90%91%E8%AE%A4%E8%AF%81%E3%80%81%E5%8F%8C%E5%90%91%E8%AE%A4%E8%AF%81%E4%BB%A5%E5%8F%8AToken%E8%AE%A4%E8%AF%81%EF%BC%8C%E6%B6%89%E5%8F%8A%E8%AF%81%E4%B9%A6%E7%9A%84%E7%94%9F%E6%88%90%E3%80%81%E9%85%8D%E7%BD%AE%E5%8F%8A%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E3%80%82%20%E9%87%8D%E7%82%B9%E8%AE%B2%E8%A7%A3%E4%BA%86%E8%AF%81%E4%B9%A6%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E7%9A%84%E5%88%9B%E5%BB%BA%E3%80%81%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%92%8C%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%BB%A3%E7%A0%81%E7%9A%84%E4%BF%AE%E6%94%B9%EF%BC%8C%E4%BB%A5%E5%8F%8A%E6%8B%A6%E6%88%AA%E5%99%A8%E7%9A%84%E4%BD%BF%E7%94%A8%EF%BC%8C%E7%A1%AE%E4%BF%9D%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93%E7%9A%84%E5%AE%89%E5%85%A8%E6%80%A7%E3%80%82,%E7%A0%81%E4%BA%91%EF%BC%9A%20https%3A%2F%2Fgitee.com%2FXiMuQi%2Fgo-grpc
-->

## 准备工作

* 安装 protoc

```shell
# 网址 https://github.com/protocolbuffers/protobuf/

# 选择安装的版本

# 下方是下载v24.4版本的命令
wget https://github.com/protocolbuffers/protobuf/releases/download/v24.4/protoc-24.4-linux-x86_64.zip

# 解压
unzip protoc-24.4-linux-x86_64.zip

# 记住安装目录, 添加配置到/etc/profile .bashrc .zshrc 等 使用什么放到哪个
# 内容如下
export PROTOC_HOME={安装目录}
export PATH=$PATH:$PROTOC_HOME/bin

# 生效
source ~/.zshrc
```

* 安装 protoc-gen-go

```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest

# 检查安装情况 
protoc-gen-go --version
```

* 安装 protoc-gen-go-grpc

```shell
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# 检查安装情况
protoc-gen-go-grpc --version
```



## 编写 proto 文件


* 说明 

```text
message : protobuf中定义一个消息类型是通过关键字message字段指定的。struct

required 消息中必填字段
optional 消息中可选字段
repeated 消息中可重复字段

消息号 每个字段都必须有一个唯一的标识号 [1,2^99-1]的整数
```

```protobuf
syntax = "proto3";

package login;

option go_package = "proto/user/login";

message LoginRequest{
  string username = 1;
  string password = 2;
}

message LoginResponse{
  int64 code = 1;
  string message = 2;
}

service UserService {
  rpc Login(LoginRequest) returns (LoginResponse){};
}
```

## 生成 go 文件 

```shell
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative demo.proto
```
