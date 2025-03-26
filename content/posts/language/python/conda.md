+++
title = 'ubuntu20.04安装conda'
date = '2025-03-25T19:12:38+08:00'
author = 'jianlu'
draft = false
description = "ubuntu安装conda"
aliases = ["python", "conda"]
+++


## 安装anaconda


## 安装miniconda

```shell
# 1. 下载miniconda3的sh
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh

# 2. 安装miniconda3
# -b 无交互模式安装
# -p 指定安装路径
bash miniconda3.sh -b -p $HOME/miniconda3

# 3. 激活miniconda3
source $HOME/miniconda3/bin/activate
```

## 基本用法

```shell
# 确保conda命令在所有shell中都能用
conda init --all

# 验证conda是否正确安装
conda --version 

# 查看虚拟环境列表
conda env list 
conda info --envs
conda info -e

# 创建my-python环境 使用3.11的python版本
conda create -n my-python python=3.11

# 进入虚拟环境
conda activate my-python

# 退出虚拟环境
conda deactivate

# 删除虚拟环境 -n 指定虚拟环境的名字 --all 不加就是删除当前环境的包 加上是删除环境
conda remove -n my-python --all
conda remove numpy # 删除一个包

# 检查更新当前conda
conda update conda

#查看--安装--更新--删除包

conda list：
conda search package_name# 查询包
conda install package_name
conda install package_name=1.5.0
conda update package_name
conda remove package_name


```

## pip 镜像

```shell
# 清华pip镜像
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 阿里pip镜像
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# 腾讯pip镜像
pip config set global.index-url http://mirrors.cloud.tencent.com/pypi/simple

# 豆瓣pip镜像
pip config set global.index-url http://pypi.douban.com/simple/

# 网易pip镜像
pip config set global.index-url https://mirrors.163.com/pypi/simple/

# 临时使用 替换“xxx”为你需要安装的模块名称
pip install xxx -i https://mirrors.163.com/pypi/simple/
```

## conda 镜像

```shell
# 1. conda config --set show_channel_urls yes 生成.condarc
# - Linux: ${HOME}/.condarc
# - macOS: ${HOME}/.condarc
# - Windows: C:\Users\<YourUserName>\.condarc

# 修改.condarc

# 清华镜像
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch-lts: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  deepmodeling: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/


# 北外conda镜像
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/pkgs/main/
#Conda Forge


conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/conda-forge/
#msys2（可略）
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/msys2/
#bioconda（可略）
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/bioconda/
#menpo（可略）
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/menpo/
#pytorch
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/pytorch/
# for legacy win-64（可略）
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/peterjc123/
conda config --set show_channel_urls yes

# 阿里conda镜像
channels:
  - defaults
show_channel_urls: true
default_channels:
  - http://mirrors.aliyun.com/anaconda/pkgs/main
  - http://mirrors.aliyun.com/anaconda/pkgs/r
  - http://mirrors.aliyun.com/anaconda/pkgs/msys2
custom_channels:
  conda-forge: http://mirrors.aliyun.com/anaconda/cloud
  msys2: http://mirrors.aliyun.com/anaconda/cloud
  bioconda: http://mirrors.aliyun.com/anaconda/cloud
  menpo: http://mirrors.aliyun.com/anaconda/cloud
  pytorch: http://mirrors.aliyun.com/anaconda/cloud
  simpleitk: http://mirrors.aliyun.com/anaconda/cloud
```

* [docs](https://www.anaconda.com/docs/main)



