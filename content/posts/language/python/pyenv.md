+++
title = '使用pyenv安装python'
date = '2024-02-03T11:14:25+08:00'
author = 'jianlu'
draft = false
description = "使用pyenv安装python"
aliases = ["python", "pyenv"]
+++

# 使用pyenv安装python

## 安装pyenv

0. 安装前准备

```bash
sudo apt update
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```

1. 自动安装方式

```bash
# 使用脚本执行clone项目并安装过程
curl https://pyenv.run | bash
echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init --path)"' >> ~/.zshrc
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
source ~/.zshrc
```

2. 手动安装方式

```shell
# 1. clone 项目
git clone https://github.com/pyenv/pyenv.git ~/.pyenv

# 2. 编辑.zshrc | .bashrc | .profile
# pyenv
export PYENV_ROOT="$HOME/.pyenv"
# 安装时使用镜像
export PYTHON_BUILD_MIRROR_URL="https://registry.npmmirror.com/-/binary/python"
export PYTHON_BUILD_MIRROR_URL_SKIP_CHECKSUM=1
#eval "$(pyenv init --path)"
#eval "$(pyenv virtualenv-init -)"

#path
export PATH=$PYENV_ROOT/bin:$PATH

# pyenv
eval "$(pyenv init --path)"
eval "$(pyenv virtualenv-init -)"
eval "$(pyenv init -)"
```

## 问题 virtualenv-init 找不到解决方案

```bash
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
source ~/.zshrc
```

## 基本命令

```shell
pyenv virtualenvs # 展示虚拟环境
pyenv virtualenv 3.11.11 my-python # 创建虚拟环境 使用3.11.11的python
pyenv activate my-python # 使用虚拟环境
pyenv deactivate # 退出虚拟环境

```
