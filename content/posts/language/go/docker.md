+++
title = 'go应用容器化部署'
date = '2024-12-27T15:51:11+08:00'
author = 'jianlu'
draft = false
description = "go应用容器化部署"
+++

# golang语言编写的应用使用容器化部署

## 目录结构

```text
# 目录结构一
app                # 应用根目录
├── cmd
├── configs
├── doc.go
├── docker-compose.yaml
├── Dockerfile
├── docs
├── go.mod
├── go.sum
├── internal
├── LICENSE
├── Makefile
└── README.md

# 目录结构二
app-module
├── app
│   ├── cert
│   ├── configs
│   ├── Dockerfile
│   ├── Dockerfile-alpine
│   ├── Dockerfile-busybox
│   ├── go.mod
│   ├── go.sum
│   ├── internal
│   ├── Makefile
│   ├── README.md
│   └── src
├── go.work
├── go.work.sum
└── tools
    ├── bootstarp.go
    ├── chain
    ├── config
    ├── crypto
    ├── cst
    ├── db
    ├── go.mod
    ├── go.sum
    ├── internal
    ├── ipfs-cluster
    ├── logger
    ├── tools
    └── web
```

## 针对目录结构一的Dockerfile

### busybox 运行

```dockerfile
FROM golang:1.22-alpine3.19 AS gobuilder

ENV GO111MODULE=on
ENV GOPROXY=https://goproxy.cn,https://goproxy.io,direct
ENV GOSUMDB=sum.golang.google.cn

WORKDIR /build/app

COPY go.mod go.sum ./

RUN go mod download -x 

COPY . . 

RUN if [[ -f "app" ]];then  rm app; fi && \
    go build -ldflags="-s -w" -o app ./cmd/app/


FROM debian:stable-slim AS toolbuilder

RUN sed -i s@deb.debian.org@mirrors.aliyun.com@g /etc/apt/sources.list.d/debian.sources

RUN set -eux; \
	apt-get update; \
	apt-get install -y \
		tini \
        tzdata \
		gosu \
        fuse \
        ca-certificates \
	    ; \
	rm -rf /var/lib/apt/lists/*
RUN echo "Asia/Shanghai" > /etc/timezone

FROM busybox:glibc AS runner

WORKDIR /app

# 从 toolbuilder 层 拷贝工具
COPY --from=toolbuilder /usr/sbin/gosu /sbin/gosu
COPY --from=toolbuilder /usr/bin/tini /sbin/tini
COPY --from=toolbuilder /bin/fusermount /usr/local/bin/fusermount
COPY --from=toolbuilder /etc/ssl/certs /etc/ssl/certs
COPY --from=datebuilder /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY --from=datebuilder /etc/timezone /etc/timezone

# 从 gobuilder 层 拷贝编译好的应用
COPY --from=gobuilder /build/app/app ./app
COPY --from=gobuilder /build/app/configs /app/configs

VOLUME /app/configs
VOLUME /app/log

EXPOSE 7500

ENTRYPOINT ["/sbin/tini", "--", "/app/app"]

CMD ["--config", "/app/configs/"]
```

### ubuntu 运行

```dockerfile
FROM golang:1.22-alpine3.19 AS gobuilder

ENV GO111MODULE=on
ENV GOPROXY=https://goproxy.cn,https://goproxy.io,direct
ENV GOSUMDB=sum.golang.google.cn

WORKDIR /build/app

WORKDIR /build/app

COPY go.mod go.sum ./

RUN go mod download -x 

COPY . . 

RUN if [[ -f "app" ]];then  rm app; fi && \
    go build -ldflags="-s -w" -o app ./cmd/app/

FROM ubuntu:22.04 AS datebuilder

RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
     sed -i s@/security.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
      apt update && \
      apt install tzdata -y && \
      echo "Asia/Shanghai" > /etc/timezone
RUN echo "done"

FROM ubuntu:22.04 AS runner

WORKDIR /app

# 代码中使用了carbon包 需要设置这个东西
#ENV GOROOT=/usr/local/go
#RUN mkdir -p $GOROOT/lib/time
#COPY --from=gobuilder /usr/local/go/lib/time/zoneinfo.zip $GOROOT/lib/time/zoneinfo.zip

RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
     sed -i s@/security.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
      apt update && \
      apt install curl -y

COPY --from=gobuilder /build/app/app ./
COPY --from=gobuilder /build/app/app/configs /app/configs
COPY --from=datebuilder /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY --from=datebuilder /etc/timezone /etc/timezone

VOLUME /app/configs
VOLUME /app/log

EXPOSE 7500

HEALTHCHECK --interval=60s --timeout=5s --retries=3 \
    CMD curl -ksS https://localhost:7500/health || exit 1

ENTRYPOINT ["./app"]

CMD ["--config", "/app/configs/"]
```

### alpine 无法运行

```dockerfile
FROM golang:1.22-alpine3.19 AS gobuilder

ENV GO111MODULE=on
ENV GOPROXY=https://goproxy.cn,https://goproxy.io,direct
ENV GOSUMDB=sum.golang.google.cn

WORKDIR /build/app

COPY go.mod go.sum ./

RUN go mod download -x 

COPY . . 

RUN if [[ -f "app" ]];then  rm app; fi && \
    go build -ldflags="-s -w" -o app ./cmd/app/


FROM alpine3.19 AS toolbuilder

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories && \
    apk update && \
    apk add --no-cache tzdata

RUN echo "Asia/Shanghai" > /etc/timezone

FROM alpine3.19 AS runner

WORKDIR /app

# 从 toolbuilder 层 拷贝工具
COPY --from=toolbuilder /usr/sbin/gosu /sbin/gosu
COPY --from=toolbuilder /usr/bin/tini /sbin/tini
COPY --from=toolbuilder /bin/fusermount /usr/local/bin/fusermount
COPY --from=toolbuilder /etc/ssl/certs /etc/ssl/certs
COPY --from=datebuilder /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY --from=datebuilder /etc/timezone /etc/timezone

# 从 gobuilder 层 拷贝编译好的应用
COPY --from=gobuilder /build/app/app ./app
COPY --from=gobuilder /build/app/configs /app/configs

VOLUME /app/configs
VOLUME /app/log

EXPOSE 7500

ENTRYPOINT ["/sbin/tini", "--", "/app/app"]

CMD ["--config", "/app/configs/"]
```

## 针对目录结构二的Dockerfile

### 专用makefile
```makefile
CUR_DIR:=$(notdir $(shell dirname $(abspath $(MAKEFILE_LIST))))
CDIR:=$(dir $(abspath $(MAKEFILE_LIST)))
CDIRNAME:=$(shell dirname $(CDIR))

# ubuntu 
#DOCKER_FILE:= $(lastword $(CUR_DIR))/Dockerfile
# busybox
#DOCKER_FILE:= $(lastword $(CUR_DIR))/Dockerfile-busybox
# alpine
DOCKER_FILE:= $(lastword $(CUR_DIR))/Dockerfile-alpine
CUR_TIME:= $(shell date +"%Y%m%d%H%M")
IMAGE_VERSION:=v$(CUR_TIME)
# ubuntu
#IMAGE_NAME:=ubuntu/app/app-server:$(IMAGE_VERSION)
# busybox
#IMAGE_NAME:=busybox/app/app-server:$(IMAGE_VERSION)
# alpine
IMAGE_NAME:=alpine/app/app-server:$(IMAGE_VERSION)

build:
	@echo "Current Dir: $(CUR_DIR)"
	@echo "Dockerfile : $(DOCKER_FILE)"
	@echo "Current Time: $(CUR_TIME)"
	@echo "Image Version: $(IMAGE_VERSION)"
	@cd "$(CDIRNAME)" && docker buildx build -t "$(IMAGE_NAME)" --no-cache -f "$(DOCKER_FILE)" .
    @echo "Image Name: $(IMAGE_NAME)"
	@echo "done"
.PHONY: build
```

### ubuntu  

```dockerfile
FROM golang:1.22-alpine3.19 AS gobuilder

ENV GO111MODULE=on
ENV GOPROXY=https://goproxy.cn,https://goproxy.io,direct
ENV GOSUMDB=sum.golang.google.cn

WORKDIR /build/app

COPY . .

RUN cd /build/app/app && \
    if [[ -f "app" ]];then  rm app; fi && \
    go mod tidy && \
    go build -ldflags="-s -w" -o app ./src

FROM ubuntu:22.04 AS datebuilder

RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
     sed -i s@/security.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
      apt update && \
      apt install tzdata -y && \
      echo "Asia/Shanghai" > /etc/timezone

FROM ubuntu:22.04 AS runner

WORKDIR /app

#ENV GOROOT=/usr/local/go
#RUN mkdir -p $GOROOT/lib/time
#COPY --from=gobuilder /usr/local/go/lib/time/zoneinfo.zip $GOROOT/lib/time/zoneinfo.zip

RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
     sed -i s@/security.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
      apt update && \
      apt install curl -y

COPY --from=gobuilder /build/app/app/app ./
COPY --from=gobuilder /build/app/app/cert /app/cert
COPY --from=gobuilder /build/app/app/configs /app/configs
COPY --from=datebuilder /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY --from=datebuilder /etc/timezone /etc/timezone

VOLUME /app/configs
VOLUME /app/cert
VOLUME /app/log

EXPOSE 7500

HEALTHCHECK --interval=60s --timeout=5s --retries=3 \
    CMD curl -ksS https://localhost:7500/health || exit 1

ENTRYPOINT ["./app"]

CMD ["--config", "/app/configs/"]
```

### busybox 

```dockerfile
FROM golang:1.22.2-alpine3.19 AS gobuilder

ENV GO111MODULE=on
ENV GOPROXY=https://goproxy.cn,https://goproxy.io,direct
ENV GOSUMDB=sum.golang.google.cn

WORKDIR /build/app

COPY . .

RUN cd /build/app/app && \
    if [[ -f "app"]];then  rm app; fi && \
    go mod tidy && \
    go build -ldflags="-s -w" -o app ./src

FROM debian:stable-slim AS datebuilder

RUN sed -i s@deb.debian.org@mirrors.aliyun.com@g /etc/apt/sources.list.d/debian.sources

RUN set -eux; \
	apt-get update; \
	apt-get install -y \
		tini \
        tzdata \
		gosu \
    fuse \
    ca-certificates \
	; \
	rm -rf /var/lib/apt/lists/*
RUN echo "Asia/Shanghai" > /etc/timezone

FROM busybox:glibc AS runner

WORKDIR /app

COPY --from=datebuilder /usr/sbin/gosu /sbin/gosu
COPY --from=datebuilder /usr/bin/tini /sbin/tini
COPY --from=datebuilder /bin/fusermount /usr/local/bin/fusermount
COPY --from=datebuilder /etc/ssl/certs /etc/ssl/certs
COPY --from=gobuilder /build/app/app/app ./abc
COPY --from=gobuilder /build/app/app/cert /app/cert
COPY --from=gobuilder /build/app/app/configs /app/configs
COPY --from=datebuilder /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY --from=datebuilder /etc/timezone /etc/timezone

VOLUME /app/configs
VOLUME /app/cert
VOLUME /app/log

EXPOSE 7500

ENTRYPOINT ["/sbin/tini", "--", "/app/abc"]

CMD ["--config", "/app/configs/"]


```

### alpine  无法运行

```dockerfile
FROM golang:1.22.2-alpine3.19 AS gobuilder

ENV GO111MODULE=on
ENV GOPROXY=https://goproxy.cn,https://goproxy.io,direct
ENV GOSUMDB=sum.golang.google.cn

WORKDIR /build/app

COPY . .

RUN cd /build/app/app && \
    if [[ -f "app"]];then  rm app; fi && \
    go mod tidy && \
    go build -ldflags="-s -w" -o app ./src

FROM alpine:3.18 AS datebuilder

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories && \
    apk update && \
    apk add --no-cache tzdata

RUN echo "Asia/Shanghai" > /etc/timezone

FROM alpine:3.18 AS runner

WORKDIR /app

COPY --from=gobuilder /build/app/app/app ./abc
COPY --from=gobuilder /build/app/app/cert /app/cert
COPY --from=gobuilder /build/app/app/configs /app/configs
COPY --from=datebuilder /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY --from=datebuilder /etc/timezone /etc/timezone

VOLUME /app/configs
VOLUME /app/cert
VOLUME /app/log

EXPOSE 7500

ENTRYPOINT ["/sbin/tini", "--", "/app/abc"]

CMD ["--config", "/app/configs/"]
```