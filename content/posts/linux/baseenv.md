+++
title = '配置linux开发环境'
date = '2023-08-17T17:20:12+08:00'
author = 'jianlu'
draft = false
description = "配置linux的开发环境"
aliases = ["develop", "environment"]
+++

<a id = "top"></a>

----

## 目录

* [go](#1)
* [java](#2)
* [docker](#3)
* [docker-compose](#4)
* [git](#5)

<a id = "1"></a>

## go

```shell
mkdir -p /usr/local/dev && cd /usr/local/dev/
# 下载go
wget https://golang.google.cn/dl/go1.17.5.linux-amd64.tar.gz
tar xzvf go1.17.5.linux-amd64.tar.gz -C ./

vi /etc/profile
# 添加内容
export GOROOT=/usr/local/dev/go
export GOPATH=$HOME/go
export GO111MODULE=on
export GONOSUMDB=github.com/hyperledger/fabric*,chainmaker.org/*
export GOSUMDB=sum.golang.google.cn
export GOPROXY=https://goproxy.cn,direct
export GOINSECURE=chainmaker.org/chainmaker/*
export GOPRIVATE=chainmaker.org/chainmaker/*
export GOTMPDIR=$GOPATH/tmp
export GOCACHE=$GOPATH/cache
export GOENV=$GOPATH/config

export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
# 保存文件 按esc :wq
source /etc/profile

```

<a id = "2"></a>

## java

[华为jdk镜像](https://repo.huaweicloud.com/java/jdk/)

```shell
cd /usr/local/dev
wget https://repo.huaweicloud.com/java/jdk/8u201-b09/jdk-8u201-linux-x64.tar.gz
tar xzvf jdk-8u201-linux-x64.tar.gz -C ./jdk-8u201-linux-x64.tar.gz
ls 
# 查看java文件名 
mv javaxxx jdk8
vi /etc/profile

#java环境变量
export JAVA_HOME=/usr/local/dev/jdk8
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JRE_HOME/lib:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/other.jar
export JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8

export PATH=$PATH:$JAVA_HOME/bin
# 保存文件 按esc :wq
source /etc/profile
```

<a id = "3"></a>

## docker

[清华docker](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

* 安装最新版

```shell
curl -sSL https://get.daocloud.io/docker | sh

export DOWNLOAD_URL="https://mirrors.tuna.tsinghua.edu.cn/docker-ce"
# 如您使用 curl
curl -fsSL https://raw.githubusercontent.com/docker/docker-install/master/install.sh | sudo -E sh
# 如您使用 wget
wget -O- https://raw.githubusercontent.com/docker/docker-install/master/install.sh | sudo -E sh
```

* 安装指定版本

```shell
# 如果你过去安装过 docker，先删掉
sudo apt-get remove docker docker-engine docker.io containerd runc
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
# 安装依赖
sudo apt-get update
sudo apt-get install  apt-transport-https ca-certificates curl gnupg-agent software-properties-common gnupg
# 信任 Docker 的 GPG 公钥并添加仓库
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) \
  stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
# 新版本安装 集成了compose buildx
apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

apt-cache madison docker-ce
```

* 结果如下

```text
  docker-ce | 5:18.09.1~3-0~ubuntu-xenial | https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 5:18.09.0~3-0~ubuntu-xenial | https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 18.06.1~ce~3-0~ubuntu       | https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 18.06.0~ce~3-0~ubuntu       | https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  xenial/stable amd64 Packages
  ...
```

* 继续执行

```shell
# <VERSION_STRING> 上方第二列 如 18.06.0~ce~3-0~ubuntu
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io

sudo docker run hello-world
```

* 普通用户加入docker组

```shell
# 查看是否有docker组
cat /etc/group | grep docker 
# 查看用户
cat /etc/passwd | grep $(whoami)
# 没有docker组 则新增docker组
sudo groupadd docker
# 当前用户加入docker组
sudo usermod -aG docker $(whoami)
```

* 安装loki插件

```shell
docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
```

* 如何使用loki

1. daemon.json

```json
  {
  "log-driver": "loki",
  "log-opts": {
    "loki-url": "http://loki:3100",
    "loki-batch-size": "400"
  }
}
```

2. compose 文件

```yaml
    jaeger:
    image: jaegertracing/all-in-one:1.51
    ports:
      - '6831:6831'
      - '16686:16686'
    logging:
      driver: loki
      options:
        loki-url: 'http://localhost:3100/api/prom/push'
        # Prevent container from being stuck when shutting down
        # https://github.com/grafana/loki/issues/2361#issuecomment-718024318
        loki-timeout: 1s
        loki-max-backoff: 1s
        loki-retries: 1
```

* 获取镜像tag等信息

1. [轩辕docker](https://dockers.xuanyuan.me/)



<a id = "4"></a>

## docker-compose

* docker-compose版本查看
  [docker-compose查看版本](https://github.com/docker/compose/releases)

```shell
# v2.21.0位置替换指定版本
sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

安装最新版

```shell
sudo curl -L "https://mirror.ghproxy.com/https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 创建软连接
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# 加权限
chmod +x /usr/local/bin/docker-compose
```

<a id = "5"></a>

## git

```shell
# 方式一
apt install git

# 方式二
add-apt-repository ppa:git-core/ppa
apt update
apt install git
```
