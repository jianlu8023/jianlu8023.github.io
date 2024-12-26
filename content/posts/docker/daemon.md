+++
title = 'docker配置daemon.json'
date = '2024-12-26T16:49:02+08:00'
author = 'jianlu'
draft = false
description = "docker配置daemon.json"
aliases = ["daemon.json", "docker proxy"]
+++

# docker 配置 daemon.json

```shell
# 打开daemon.json
sudo vim /etc/docker/daemon.json

```

```text
说明
registry-mirrors docker加速地址
dns dns地址
data-root docker数据存储位置 默认是/var/lib/docker
log-driver 日志驱动
log-opts 日志配置
```

```json
{
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ],
  "registry-mirrors": [
    "https://dockerproxy.net",
    "https://docker.1ms.run",
    "https://5test4aw.mirror.aliyuncs.com",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://ccr.ccs.tencentyun.com",
    "https://docker.nju.edu.cn",
    "https://dockerhub.azk8s.cn",
    "https://docker.m.daocloud.io",
    "https://mirror.iscas.ac.cn",
    "https://hub.uuuadc.top",
    "https://docker.anyhub.us.kg",
    "https://dockerhub.jobcher.com",
    "https://dockerhub.icu",
    "https://docker.ckyl.me",
    "https://docker.awsl9527.cn",
    "https://docker.rainbond.cc"
  ],
  "insecure-registries": [
  ],
  "dns": [
    "223.5.5.5",
    "8.8.8.8"
  ],
  "data-root": "/usr/local/dev/docker",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "500m",
    "max-file": "3"
  }
}
```

```shell
# 重启docker
sudo systemctl restart docker.service
sudo systemctl daemon-reload
```