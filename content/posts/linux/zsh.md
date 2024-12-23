+++
author = "jianlu"
title = 'linux安装zsh'
date = 2024-12-12
description = "zsh"
draft = false
aliases = ["oh-my-zsh", "zsh"]
+++

1. 首先安装zsh
```shell
# ubuntu 
sudo apt-get install zsh
```

2. 克隆oh-my-zsh 仓库 获取zshrc模板
```shell
git clone https://mirror.ghproxy.com/https://github.com/robbyrussell/oh-my-zsh.git ${HOME}/.oh-my-zsh
```

3. 拷贝zshrc模板到~目录
```shell
cp $HOME/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

4. 进行zsh终端
```shell 
zsh
```

5. 生效zsh配置
```shell
source ~/.zshrc
```

6. 获取插件
```shell
git clone https://mirror.ghproxy.com/https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh}/plugins/zsh-syntax-highlighting
git clone https://mirror.ghproxy.com/https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh}/plugins/zsh-autosuggestions
```

7. 修改zsh配置文件 加入插件
```shell
vi ~/.zshrc
# 在zshrc中找到plugins=(git)
# 将plugins=(git) 修改为
plugins=(git zsh-syntax-highlighting zsh-autosuggestions jsontools)
# 保存 
```

8. 使配置生效
```shell
source ~/.zshrc
```

9. 修改默认shell
```shell
chsh -s /bin/zsh
```

Tips:

* 遇到Syntax error:"(" unexpected 如何解决

1. 执行命令
```shell
sudo dpkg-reconfigure dash
```

2. 在弹出窗口点击“否”


