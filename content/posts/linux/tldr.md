+++
title = 'linux上tldr'
date = '2025-06-30T17:33:27+08:00'
author = 'jianlu'
draft = false
description = "linux tldr 安装及使用"
aliases = ["linux", "tldr"]
+++

## 安装
```shell
sudo apt install tldr

npm install -g tldr

pip install tldr
```

## 配置

```shell
# 在bashrc zshrc等添加
export TLDR_LANGUAGE=zh

# 更新
tldr --update

# 顺手指令
alias how="tldr"
```

## 使用

```shell
tldr ls
tldr git
```
