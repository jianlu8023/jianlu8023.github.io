+++
title = 'ubuntu20.04安装podman'
date = '2025-01-14T10:33:28+08:00'
author = 'jianlu'
draft = false
description = "ubuntu20.04安装podman"
aliases = ["podman", "ubuntu20.04"]
+++

# install

* ubuntu 20.04

```shell
# 1. you will need to install some dependencies required to install Podman
sudo apt-get install curl wget gnupg2 -y

# 2.  source your Ubuntu release and add the Podman repository with the following command
source /etc/os-release
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"

# 3. download and add the GPG key with the following command
sudo wget -nv https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable/xUbuntu_${VERSION_ID}/Release.key -O- | sudo apt-key add -

# 4. update the repository and install Podman with the following command
sudo apt-get update -qq -y
sudo apt-get -qq --yes install podman

# 5. verify the Podman version with the following command
podman --version

# 6. check more information with the following command
podman info
```

[//]: # (https://www.atlantic.net/dedicated-server-hosting/how-to-install-and-use-podman-on-ubuntu/)

* ubuntu 20.10 later

```shell
sudo apt-get update
sudo apt-get -y install podman
```

# Add OCI Registry

```shell
# 7. When you pull an image using the Podman command, it will look for a list of registries from the registry configuration file /etc/containers/registries.conf. You can edit and add different registries in the configuration file.
nano /etc/containers/registries.conf
```

* Add the following lines at the end of the file
*
    * v2 版本 打开registries.conf 发现有unqualified-search-registries, 这个就是v2 版本

```toml
# unqualified-search-registries 和 registries.search 含义相同
# podman pull docker.io/mysql 只会从docker.io 获取镜像
# podman pull mysql 没有明确指定仓库时,使用下方配置顺序去获取
unqualified-search-registries = ["docker.io", "quay.io"]
# unqualified-search-registries = ["registry.access.redhat.com", "registry.fedoraproject.org", "docker.io"]

# 配置仓库地址 这是一组
# prefix 是镜像前缀, location 是仓库地址, insecure 是否使用http
# prefix 不写 默认和 location 一致

[[registry]]
prefix = "docker.io"
location = "docker.io"
insecure = false

# 针对 k8s.gcr.io 前缀的 镜像 
# registry.aliyuncs.com/googlecontainersmirror 也是第三方用户的
[[registry]]
prefix = "k8s.gcr.io"
location = "registry.aliyuncs.com/google_containers"
insecure = false

# 镜像地址 可以配置多个， 前提 至少有一个 [[registry]] 配置
# 无论unqualified-search-registries 选择了哪个仓库
# 都会从这里上下顺序开始拉取镜像 最后才会匹配 [[registry]] 里 prefix 对应的 location 仓库拉取镜像
# 每个 [[registry.mirror]] 都可以有两个配置项 location 和 insecure
[[registry.mirror]]
location = "mirror.ustc.edu.cn"
[[registry.mirror]]
location = "docker.1ms.run"
[[registry.mirror]]
location = "hub-dev.cnbn.org.cn"
```

*
    * 自用registries.conf

```toml
unqualified-search-registries = [
  "docker.io", "quay.io", "ghcr.io", "gcr.io", "k8s.gcr.io",
  "registry.access.redhat.com", "container-registry.oracle.com", "registry.suse.com",
  "registry.fedoraproject.org", "registry.opensuse.org"
]

[[registry]]
prefix = "quay.io"
location = "quay.io"
insecure = false
[[registry.mirror]]
location = "quay.mirrorify.net"
[[registry.mirror]]
location = "quay.mirrors.ustc.edu.cn"
insecure = false

[[registry]]
prefix = "ghcr.io"
location = "ghcr.io"
inseucre = false
[[registry.mirror]]
location = "ghcr.mirrorify.net"

[[registry]]
prefix = "docker.io"
location = "docker.io"
insecure = false
[[registry.mirror]]
location = "dockerproxy.net"
insecure = false
[[registry.mirror]]
location = "docker.1ms.run"
[[registry.mirror]]
insecure = false
location = "docker.xuanyuan.me"
[[registry.mirror]]
location = "docker.m.daocloud.io"
[[registry.mirror]]
location = "hub-dev.cnbn.org.cn"
[[registry.mirror]]
location = "mirror.ustc.edu.cn"
insecure = false
[[registry.mirror]]
location = "hub-mirror.c.163.com"
insecure = false
[[registry.mirror]]
location = "mirror.baiduce.com"
insecure = false
[[registry.mirror]]
location = "docker.mirrors.sjtug.sjtu.edu.cn"
insecure = false
[[registry.mirror]]
location = "docker.nju.edu.cn"
insecure = false

[[registry]]
prefix = "k8s.gcr.io"
location = "k8s.gcr.io"
inseucre = false
[[registry.mirror]]
location = "registry.aliyuncs.com/google_containers"
insecure = false
[[registry.mirror]]
location = "k8s.mirrorify.net"
[[registry.mirror]]
location = "k9s.m.daocloud.io"

[[registry]]
prefix = "gcr.io"
location = "gcr.io"
insecure = false

[[registry.mirror]]
location = "gcr.lank8s.io"
```

*
    * v1 版本

```text
[registries.search]
registries=["registry.access.redhat.com", "registry.fedoraproject.org", "docker.io"]
```

[//]: # (https://github.com/containers/image/issues/1054)

# Working with Podman

* To search

```shell
# podman search <image-name>
podman search nginx
```

* To download

```shell
#podman pull <image-name>
podman pull nginx
```

* To list images

```shell
podman images
```

* To create a new container

```shell
podman run -itd nginx
podman run quay.io/podman/hello
```

* To list containers

```shell
podman ps
```

* To create a new image from running container

```shell
#podman commit <container-id> <image-name>
podman commit --author "Author Name Hitesh" c1692863973d docker.io/library/new-nginx
```

* To stop the running container

```shell
#podman stop <container-id>
podman stop c1692863973d
```

* To remove the running container

```shell
#podman rm <container-id>
podman rm c1692863973d
```

* To stop and start the latest container

```shell
podman stop --latest
podman start --latest
```

* To remove the latest container

```shell
podman rm --latest
```

# ubuntu 20.04 安装 podman-compose

```shell
sudo apt-get install python3-dotenv
sudo curl -sSL -o /usr/local/bin/podman-compose https://mirror.ghproxy.com/https://raw.githubusercontent.com/containers/podman-compose/main/podman_compose.py
sudo chmod +x /usr/local/bin/podman-compose
sudo ln -s /usr/local/bin/podman-compose /usr/bin/podman-compose
podman-compose --version
```

#  podman 查看网络信息

```shell
# 1. 查看slirp4netns进程
ps -ef | grep netns

# 如下结果
# /usr/bin/slirp4netns --disable-host-loopback --mtu=65520 --enable-sandbox --enable-seccomp -c -r 3 --netns-type=path /run/user/1000/netns/rootless-cni-ns tap0

# 2. podman unshare 命令
podman unshare nsenter --net=/run/user/1000/netns/rootless-cni-ns

# 3. ip addr 即可出现容器ip信息
ip addr
```
