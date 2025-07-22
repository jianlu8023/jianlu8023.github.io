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

### pip自定义配置

```text
1. pip加载顺序 每次加载配置文件，都会覆盖从之前加载的文件中读取的任何值
    global  系统范围内的配置文件，在用户之间共享
        C:\ProgramData\pip\pip.ini (win7及以上，隐藏但可写)
    
    user  用户级别的配置文件，只对当前用户有效
        %APPDATA%\pip\pip.ini (windows)
        
    site 环境级别的配置文件 ，只对当前环境有效
        %VIRTUAL_ENV%\pip.ini
    PIP_CONFIG_FILE 环境变量指定的配置文件
        export PIP_CONFIG_FILE=${HOME}/.config/pip/pip.ini (linux)
举例：
    global和user都配置了timeout pip执行时会使用user的timeout
``` 

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
extra-index-url = https://pypi.mirrors.ustc.edu.cn/simple/
trusted-host = pypi.tuna.tsinghua.edu.cn
trusted-host = files.pythonhosted.org
# timeout = 60

[install]
find-links = https://pypi.org/simple https://download.pytorch.org/whl/torch_stable.html
trusted-host = pypi.douban.com
# 配置bool值时，true、yes、y、on、1 都表示true，false、no、n、off、0 都表示false
# ignore-installed = true
# no-dependencies = yes
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
