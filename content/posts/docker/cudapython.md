+++
title = '基于cuda镜像安装python'
date = '2025-07-16T11:20:02+08:00'
author = 'jianlu'
draft = false
description = "基于cuda镜像安装python"
aliases = ["cuda", "python"]
+++

## 说明

* configure区别

```text
若 ./configure
    则安装后可执行文件默认放在/usr/local/bin
    库文件默认放在/usr/local/lib
    配置文件默认放在/usr/local/include
    其它的资源文件放在/usr/local/share
    
    无需配置环境变量
    
    ln -s 镜像中无需mv cuda镜像不带python 直接ln即可
        mv /usr/bin/python /usr/bin/python.bak
        ln -s /usr/local/bin/python3.7 /usr/bin/python
        mv /usr/bin/pip /usr/bin/pip.bak
        ln -s /usr/local/bin/pip3 /usr/bin/pip

若./configure --prefix=/usr/local/python3.7.1
    则可执行文件放在/usr/local/python3.7.1/bin
    库文件放在/usr/local/python3.7.1/lib
    配置文件放在/usr/local/python3.7.1/include
    其它的资源文件放在/usr/local/python3.7.1/share
    
    需要配置环境变量
        PATH=$PATH:/usr/local/python3.7.1/bin
        
    ln -s 镜像中需要mv cuda镜像带python 需要mv
        mv /usr/bin/python /usr/bin/python.bak
        ln -s /usr/local/python3.7.1/bin/python3.7 /usr/bin/python
        mv /usr/bin/pip /usr/bin/pip.bak
        ln -s /usr/local/python3.7.1/bin/pip3 /usr/bin/pip
```

* 下载地址

```text
aliyun https://mirrors.aliyun.com/python-release/?spm=a2c6h.13651104.d-5283.1.65642609NtUUYx
官方 https://www.python.org/ftp/python/3.6.12/Python-3.6.12.tgz
```

* pytorch镜像

```text
https://mirrors.aliyun.com/pytorch-wheels/
pip install --index-url=https://mirrors.aliyun.com/pytorch-wheels/ torch==2.6.0+cpu torchvision==0.21.0+cpu torchaudio==2.6.0+cpu
```

## Commands

* 构建镜像

```shell
docker buildx build -t cuda:12.2.2-runtime-ubuntu20.04-python3.11 -f Dockerfile .
```

## Dockerfile

```dockerfile
#FROM nvidia/cuda:12.2.2-devel-ubuntu20.04
FROM nvidia/cuda:12.2.2-runtime-ubuntu20.04

ENV PYTHON_VERSION=3.11.0
# 解决tzdata前台卡住问题
ENV DEBIAN_FRONTEND noninteractive

# 1 更改镜像源 2 安装tzdata并跳过前台设置时区 3 安装编译python的工具 4 下载python 5 编译python
# 6 安装python 7 设置pip镜像并更新pip 8 删除cache
RUN sed -i "s#archive.ubuntu.com#mirrors.aliyun.com#g" /etc/apt/sources.list && \
    sed -i "s#security.ubuntu.com#mirrors.aliyun.com#g" /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y tzdata && \
    ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get install -y vim wget gcc make automake libtool zlibc tk-dev libssl-dev build-essential libsqlite3-dev liblzma-dev && \
    apt-get-install -y libreadline-gplv2-dev libncursesw5-dev libsqlite3-dev libgdbm-dev libc6-dev libbz2-dev libffi-dev zlib1g-dev && \
    apt-get install -y libglib2.0-dev libsm6 libxrender1 libxext-dev libreadline-dev && \
    cd /root && \
    wget https://mirrors.aliyun.com/python-release/source/Python-$PYTHON_VERSION.tgz && \
    tar xzvf Python-$PYTHON_VERSION.tgz && \
    cd Python-$PYTHON_VERSION && \
    ./configure --enable-optimizations --prefix=/usr/local --with-ssl && \
    make -j$(nproc) && \
    make install && \
    ln -s /usr/local/bin/python3.11 /usr/bin/python && \
    ln -s /usr/local/bin/pip3 /usr/bin/pip && \
    pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip3 install --no-cache-dir --upgrade pip && \
    rm /root/Python-$PYTHON_VERSION.tgz && \
    rm -rf /root/Python-$PYTHON_VERSION && \
    apt-get clean && \
    apt-get autoremove -y && \
    apt-get autoclean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/*

WORKDIR /

ENTRYPOINT ["python"]
CMD ["--version"]
```

```dockerfile
#FROM nvidia/cuda:12.2.2-devel-ubuntu20.04
FROM nvidia/cuda:12.2.2-runtime-ubuntu20.04

# 解决tzdata前台卡住问题
#ENV DEBIAN_FRONTEND noninteractive

# 1 更改镜像源 2 安装tzdata并跳过前台设置时区 3 安装编译python的工具 4 下载python 5 编译python
# 6 安装python 7 设置pip镜像并更新pip 8 删除cache
RUN sed -i "s#archive.ubuntu.com#mirrors.aliyun.com#g" /etc/apt/sources.list && \
    sed -i "s#security.ubuntu.com#mirrors.aliyun.com#g" /etc/apt/sources.list && \
    apt-get update && \
    DEBIAN_FRONTEND noninteractive apt-get install -y tzdata && \
    ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get install -y python3 python3-pip && \
    ln -s /usr/bin/python3.8 /usr/bin/python && \
    pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip3 install --no-cache-dir --upgrade pip && \
    apt-get clean && \
    apt-get autoremove -y && \
    apt-get autoclean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/*

WORKDIR /

ENTRYPOINT ["python"]
CMD ["--version"]
```