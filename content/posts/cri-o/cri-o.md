+++
title = 'ubuntu安装cri-o'
date = '2025-10-29T13:13:03+08:00'
author = 'jianlu'
draft = false
description = "ubuntu安装cri-o"
aliases = ["cri-o"]
+++

## 安装依赖

```shell
sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common -y
```

## 添加cri-o仓库

```shell
export OS=xUbuntu_22.04
export CRIO_VERSION=1.24
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"| sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list
```

## 添加GPG key

```shell
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | sudo apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -
```

## 安装cri-o

```shell
sudo apt update 
sudo apt install cri-o cri-o-runc -y
```

## 安装cni插件

```shell
sudo apt install containernetworking-plugins -y
```

## 安装crictl

```shell
sudo apt install -y cri-tools
```

## 启动cri-o

```shell
sudo systemctl daemon-reload
sudo systemctl start crio
sudo systemctl enable crio
sudo systemctl status crio
# 禁止
sudo systemctl stop crio
sudo systemctl disable crio
```

## crictl completion

```shell
crictl completion bash | sudo tee /etc/bash_completion.d/crictl
```

## crictl info

```shell
# 或者配置 /etc/crictl.yaml
sudo crictl --runtime-endpoint unix:///var/run/crio/crio.sock info
```
