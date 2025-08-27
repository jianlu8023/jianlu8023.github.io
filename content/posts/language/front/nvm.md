+++
title = '安装nvm'
date = '2023-05-04T15:33:06+08:00'
author = 'jianlu'
draft = false
description = "linux安装nvm"
aliases = ["nvm", "node"]
+++


## 使用git工具进行安装

### 步骤

```shell
# 1 cd ~/ from anywhere
cd ~/
# 2 git clone 
git clone https://mirror.ghproxy.com/https://github.com/nvm-sh/nvm.git .nvm
# 3 cd ~/.nvm
cd ~/.nvm
# 4 
git checkout v0.39.1
# 5 
. ./nvm.sh
# 6 edit ~/.bashrc ~/.zshrc or ~/.profile 
edit ~/.zshrc
# 7 
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
# 8 
source ~/.zshrc
```

## 一件安装shell

```shell
# v0.39.1 是 nvm 的版本 
curl -o- https://mirror.ghproxy.com/https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

```shell
wget -qO- https://mirror.ghproxy.com/https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```