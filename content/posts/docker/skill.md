+++
title = 'docker技巧合集'
date = '2023-08-31T13:17:23+08:00'
author = 'jianlu'
draft = false
description = "docker技巧"
aliases = ["docker", "skill"]
+++

<a id="top"></a>

----

# 目录

* [docker批量导出镜像](#1)
* [docker批量导入镜像](#2)
* [限制容器占用资源](#3)
* [docker images format](#4)
* [alpine mysql](#5)
* [普通用户运行程序](#6)
* [gosu和tini使用](#7)
* [容器内grpcurl安装及使用](#8)
* [python安装依赖](#9)

## docker 批量导出镜像

<a id="1"></a>

### 步骤

0. 创建文件夹保存导出镜像

```shell
# 创建文件夹
mkdir ~/docker && cd ~/docker
```

1. 根据需求改写下方shell

```shell
#bin/bash

ehco "start..."

key=""

if [[ ! -z $1  ]]; then
	key="$1"
fi

docker_rmi=$(docker images --format "{{.Repository}}:{{.Tag}}" | grep "$key")

for i in $docker_rmi ;do
  echo "current image: " +$i
  set -x
  docker_name=$(echo $i | sed -e 's/\//-/g')
  docker image save $i -o './'""$docker_name""'.tar'
  { set +x; } 2>/dev/null
  sleep 1
done
```

2. 创建save.sh

```shell

cd ~/docker
touch save.sh
# 粘贴修改后的步骤1中shell
```

3. 加权

```shell
cd ~/docker

chmod +x *.sh
```

4. 执行

```shell

./save.sh

# bash save.sh
```

## 批量导入docker镜像

<a id="2"></a>

0. 准备要导入的镜像的tar

1. 创建load.sh

```shell
touch load.sh
```

2. 将下方shell脚本输入到load.sh中

```shell
#bin/bash

folders=""
echo "load image start..."
# $1 不为 0
if [[ ! -z  $1 ]]; then
    folders="$1"
else
  folders="$(pwd)"
fi

files=`ls $folders`

file_type='.tar'
for file in $files ; do
  if [[ $file =~ $file_type ]]; then
    image_origin_name_tar=`basename $file`
    docker image load -i $image_origin_name_tar
  fi
  sleep 1
done

```

3. 加权

```shell
chmod +x *.sh
```

4. 执行

```shell
# 二选一
bash load.sh

./load.sh
```

## 限制容器占用资源

<a id="3"></a>

```yaml
version: "3.8"
networks:
  basic:
    name: basic
services:
  mysql:
    restart: always
    image: mysql:5.7.18
    container_name: mysql
    # 此部分
    deploy:
      resources:
        limits:
          cpus: "1"
          memory: 100M
        reservations:
          cpus: "0.25"
          memory: 20M
    # 此部分
    networks:
      - basic
    volumes:
      - ./mysql/mydir:/mydir
      - ./mysql/datadir:/var/lib/mysql
      - ./mysql/conf/my.cnf:/etc/my.cnf
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
      - ./mysql/source:/docker-entrypoint-initdb.d
    environment:
      - "MYSQL_ROOT_PASSWORD=123456"
      - "MYSQL_DATABASE=basic"
      - "TZ=Asia/Shanghai"
    ports:
      - 3306:3306
```

[comment]: <> (https://blog.csdn.net/qq_36148847/article/details/79427878)

## 技巧4

<a id="4"></a>

docker rmi $(docker images --format "{{.Repository}}:{{.Tag}}" | grep 'gcbaas-gm' )

docker restart `docker ps -a -f "status=exited" --format "{{.ID}}"`

## 制作基于alpine镜像的mysql镜像

<a id="5"></a>

```dockerfile
# 基镜像
# 基础镜像
FROM alpine:3.18

# 设置国科大镜像
RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g" /etc/apk/repositories \
    && apk update \
    && apk upgrade

# 安装 mysql和musql客户端
RUN apk add mysql mysql-client

# 创建目录，作为数据库存储路径
RUN mkdir -p /var/lib/mysql


RUN mkdir /var/run/mysqld && \
    chown -R mysql:mysql /var/run/mysqld && \
    mkdir /docker-entrypoint-initdb.d

RUN echo "mysqld --user=mysql" > /usr/local/bin/entrypoint.sh

RUN chmod +x /usr/local/bin/entrypoint.sh

RUN ln -s /usr/local/bin/entrypoint.sh /



RUN mysql_install_db --user=mysql --datadir=/var/lib/mysql

VOLUME /var/lib/mysql

EXPOSE 3306

WORKDIR /

ENTRYPOINT ["/entrypoint.sh"]
#CMD ["mysqld", "--user=mysql"]


```

[//]: # (curl -L https://mirror.ghproxy.com/https://raw.githubusercontent.com/docker-library/mysql/master/5.7/docker-entrypoint.sh >docker-entrypoint.sh)

## 使用普通用戶运行程序

<a id="6"></a>

```dockerfile
# ubuntu 可替换成任何镜像 这种是最基础的 前提 需要创建好目录 然后chown
FROM ubuntu:latest
#创建非 root 用户
ENV IPFS_PATH /data/ipfs
RUN groupadd -r users && useradd -r -g users -u 1000 -d $IPFS_HOME -M ipfs && \
    chown -R ipfs:users /app && \
    chown ipfs:users /docker-entrypoint.sh 
USER ipfs
# groupadd 解读
# groupadd: 这是一个用于创建新用户组的命令行工具，通常在 Linux 和 Unix 系统中使用。
#-r: 创建一个系统组。系统组通常用于运行系统服务，其 GID 通常小于 1000。

# useradd 解读
# useradd: 这是一个用于创建新用户的命令行工具，通常在 Linux 和 Unix 系统中使用。
#-r 选项表示创建一个系统用户。 系统用户的 UID 通常小于 1000，用于系统服务。
#-u 1000 选项指定新用户的 UID 为 1000。
#-g users 选项将新用户添加到 users 组。
#-d $IPFS_PATH 选项设置新用户的主目录为 $IPFS_PATH。
#-M 选项表示不创建用户的主目录。 如果你需要创建用户的主目录，可以省略 -M 选项。
#ipfs 是新用户的用户名。

# 只适用于 debian ubuntu 两种 
FROM debian:slim
ENV IPFS_PATH /data/ipfs
RUN mkdir -p $IPFS_PATH \
  && adduser -D -h $IPFS_PATH -u 1000 -G users ipfs \
  && chown ipfs:users $IPFS_PATH

# adduser命令解读
#adduser: 这是一个用于创建新用户的命令行工具，通常在 Debian 和 Ubuntu 系统中使用。
#-D: 表示 "使用默认值"。 这会跳过一些交互式提示，例如设置密码。 在 Dockerfile 中，通常使用 -D 选项，因为我们不希望在构建镜像时进行交互。
#-h $IPFS_PATH: 设置新用户的主目录为 $IPFS_PATH。 IPFS_PATH 应该是一个环境变量，它指定了 IPFS 数据的存储位置。
#-u 1000: 指定新用户的用户 ID (UID) 为 1000。 在 Linux 系统中，UID 是用于标识用户的数字。
#-G users: 将新用户添加到 users 组。 在 Linux 系统中，组用于管理用户的权限。
#ipfs: 这是新用户的用户名
```

## gosu 和 tini 使用

<a id="7"></a>

### dockerfile

```dockerfile
FROM debian:stable-slim AS builder
RUN set -eux; \
	apt-get update; \
	apt-get install -y \
		tini \
    # Using gosu (~2MB) instead of su-exec (~20KB) because it's easier to
    # install on Debian. Useful links:
    # - https://github.com/ncopa/su-exec#why-reinvent-gosu
    # - https://github.com/tianon/gosu/issues/52#issuecomment-441946745
		gosu \
    # This installs fusermount which we later copy over to the target image. \
        fuse\
        ca-certificates \
        tzdata \
	; \
	rm -rf /var/lib/apt/lists/*

FROM busybox:stable-glibc AS runtime

COPY --from=builder /usr/sbin/gosu /sbin/gosu
COPY --from=builder /usr/bin/tini /sbin/tini
COPY --from=builder /etc/ssl/certs /etc/ssl/certs
COPY --from=builder /bin/fusermount /usr/local/bin/fusermount

# Add suid bit on fusermount so it will run properly
RUN chmod 4755 /usr/local/bin/fusermount

COPY docker-entrypoint.sh /docker-entrypoint.sh

#创建非 root 用户
RUN mkdir -p /app/log && \
    mkdir -p /app/db && \
    groupadd -r myusers && useradd -r -u 1000 -g myusers appuser && \
    chown -R appuser:myusers /app && \
    chown appuser:myusers /docker-entrypoint.sh
USER appuser

ENTRYPOINT ["/sbin/tini", "--", "/docker-entrypoint.sh"]

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -ksS https://localhost:8080/ || exit 1

# Execute the daemon subcommand by default
CMD ["--config","/app/"]
```

### entrypoint.sh

```shell
#!/bin/bash

user=appuser

if [ "$(id -u)" -eq 0 ]; then
  echo "change user to $user"
  # 使用 gosu 切换到指定用户，并重新执行当前脚本
  exec gosu "$user" "$0" "$@"
fi

echo "starting app ..."
echo "use user $(whoami)"
echo "date $(date +"%Y-%m-%d %H:%M:%S")"
sleep 3
exec /app/app "$@"
```

## 容器内grpcurl安装及使用

<a id="8"></a>

```dockerfile
FROM golang:latest AS grpcurlbuilder

ENV GO111MODULE=on
ENV GOPROXY=https://goproxy.cn,https://goproxy.io,direct
ENV GOSUMDB=sum.golang.google.cn

# GOPATH=/go
RUN go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest

FROM alpine:latest

COPY --from=grpcurlbuilder /go/bin/grpcurl /usr/local/bin/grpcurl

# 启动grpc服务
# --start-period=60s --interval=60s --timeout=3s --retries=3 CMD
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD grpcurl -d '{"message_type":"ping"}' -proto cross/proto/message.proto -plaintext 127.0.0.1:9999 cross.BaseMessageProtoService/send_message || exit 1
    
```

## python安装依赖

<a id="9"></a>


```dockerfile
FROM python:3.11 AS base

RUN pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip3 install --upgrade --no-cache-dir pip

WORKDIR /app
COPY . .

FROM base AS builder

ENV CUSTOM_INSTALL_PATH=/build/site-packages

RUN mkdir -p $CUSTOM_INSTALL_PATH

RUN pip3 install --no-cache-dir -t $CUSTOM_INSTALL_PATH --index-url https://download.pytorch.org/whl/cpu torch==2.6.0+cpu torchvision==0.21.0+cpu torchaudio==2.6.0+cpu && \
    pip3 install --no-cache-dir requirements.txt && \
    pip3 cache purge

# 3.11
# slim 基础依赖包的debian
# bookworm debian 12
FROM python:3.11-slim-bookworm AS runner

WORKDIR /app

ENV CUSTOM_INSTALL_PATH=/build/site-packages

ENV PYTHONPATH=$CUSTOM_INSTALL_PATH

COPY --from=builder $CUSTOM_INSTALL_PATH $CUSTOM_INSTALL_PATH
COPY --from=base /app /app
```
