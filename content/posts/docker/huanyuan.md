+++
title = 'Docker镜像换源'
date = '2024-11-07T11:54:07+08:00'
author = 'jianlu'
draft = false
description = "docker镜像换源"
+++

# Dockerfile中给镜像换源

## alpine

```dockerfile
FROM alpine:latest as builder
# 替换阿里云
#RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 替换科技大学
#RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

# 替换清华源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories

RUN apk update && \
    apk add --no-cache tzdata

```

## ubuntu

```dockerfile
FROM ubuntu:22.04 AS builder

# ubuntu 换源 并安装tzdata(日期的工具)
RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
     sed -i s@/security.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
    apt update && \
    apt install tzdata -y
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone 


```

## debian:stable-slim

```dockerfile
FROM debian:stable-slim AS builder

RUN sed -i s@deb.debian.org@mirrors.aliyun.com@g /etc/apt/sources.list.d/debian.sources

RUN apt-get update && \
	 apt-get install -y tzdata ca-certificates
```
