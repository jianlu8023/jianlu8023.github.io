+++
title = '给containerd配置镜像'
date = '2025-10-29T12:00:15+08:00'
author = 'jianlu'
draft = false
description = "给containerd配置镜像"
aliases = ["containerd", "registry"]
+++

## 文件存放

```text
/etc/containerd/config.toml
```

## 安装crictl 

```shell
sudo apt install cri-tools
```

## 配置镜像

### containerd v2

```toml
[plugins."io.containerd.cri.v1.images".registry]
[plugins."io.containerd.cri.v1.images".registry.mirrors]
[plugins."io.containerd.cri.v1.images".registry.mirrors."docker.io"]
endpoint = ["https://registry-1.docker.io"]
[plugins."io.containerd.cri.v1.images".registry.mirrors."gcr.io"]
endpoint = ["https://gcr.io"]
[plugins."io.containerd.cri.v1.images".registry.mirrors."仓库域名"]
endpoint = ["加速地址"]
[plugins."io.containerd.cri.v1.images".registry.configs]
[plugins."io.containerd.cri.v1.images".registry.configs."gcr.io".auth]
username = "_json_key"
password = 'paste output from jq'
[plugins."io.containerd.cri.v1.images".registry.configs]
[plugins."io.containerd.cri.v1.images".registry.configs."仓库域名".auth]
username = "用户名"
password = '用户密码'
```

### containerd v1

```text
总结: 
1. 创建certs.d文件夹 
2. 编辑config.toml文件 如果之前没配置过则添加配置，如果已经配置过了 可跳过
3. 创建对应镜像的文件夹和hosts.toml文件 比如代理docker.io 则创建/etc/containerd/certs.d/docker.io/hosts.toml
4. 编辑hosts.toml文件 添加镜像地址和证书地址
5. 重启containerd
```

```shell
# 创建文件夹
sudo mkdir -p /etc/containerd/certs.d/
# 编写config.toml
```

```toml
# mirrors
[plugins."io.containerd.grpc.v1.cri".registry]
config_path = "/etc/containerd/certs.d"
[plugins."io.containerd.grpc.v1.cri".registry.auths]
[plugins."io.containerd.grpc.v1.cri".registry.configs]
[plugins."io.containerd.grpc.v1.cri".registry.configs."docker.io".auth]
username = "_json_key"
password = "paste output from jq"
[plugins."io.containerd.grpc.v1.cri".registry.headers]
[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
endpoint = ["https://docker.mirrors.ustc.edu.cn", "https://hub-mirror.c.163.com"]
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
endpoint = ["https://gcr.azk8s.cn"]
[plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
tls_cert_file = ""
tls_key_file = ""
```

```shell
sudo mkdir -p /etc/containerd/certs.d/docker.io/
sudo touch /etc/containerd/certs.d/docker.io/hosts.toml
sudo vim /etc/containerd/certs.d/docker.io/hosts.toml
```

```toml
server = "https://docker.io"

[host."https://hub.mirror.c.163.com"]
capabilities = ["pull", "resolve"]

[host."https://dockerproxy.com"]
capabilities = ["pull", "resolve"]
[host."https://docker.m.daocloud.io"]
capabilities = ["pull", "resolve"]
skip_verify = true
```

```shell
sudo systemctl restart containerd
```

