+++
author = "jianlu"
title = 'linux安装zsh'
date = 2023-03-29
description = "zsh"
draft = false
aliases = ["oh-my-zsh", "zsh"]
+++

## 安装

1. 首先安装zsh

```shell
# ubuntu 
sudo apt-get install zsh
```

2. 克隆oh-my-zsh 仓库 获取zshrc模板

```shell
git clone https://github.com/robbyrussell/oh-my-zsh.git ${HOME}/.oh-my-zsh
```

3. 拷贝zshrc模板到~目录

```shell
cp $HOME/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
# 在~/.zshrc 第一行添加 zmodload zsh/zprof
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
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh}/plugins/zsh-autosuggestions
git clone https://github.com/lukechilds/zsh-nvm ${ZSH_CUSTOM:-~/.oh-my-zsh}/custom/plugins/zsh-nvm # fnn 可代替nvm rust编写 curl -fsSL https://fnm.vercel.app/install | bash  
```

7. 修改zsh配置文件 加入插件

```shell
vi ~/.zshrc
# 在zshrc中找到plugins=(git)
# 将plugins=(git) 修改为
plugins=(git zsh-syntax-highlighting zsh-autosuggestions jsontools zsh-nvm)
# 开启懒加载
export NVM_LAZY_LOAD=true
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

## 主题配置

### p10k

```shell
# 1. clone 仓库
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"

# 2. 修改zshrc中theme
ZSH_THEME="powerlevel10k/powerlevel10k"

# 3. 配置p10k
p10k configure

# 只显示最后一个文件夹 永久生效
# 在~/.p10k.zsh 中找到 POWERLEVEL9K_SHORTEN_STRATEGY
# 将 truncate_to_unique 改为 truncate_to_last

# tab 补全时显示全路径
# 找到 POWERLEVEL9K_DIR_MAX_LENGTH
# 将 80 改为 1

# 4. 生效配置
source ~/.zshrc
```

## 问题解决

* 遇到Syntax error:"(" unexpected 如何解决

```shell
# 1. 执行命令
sudo dpkg-reconfigure dash

# 2. 在弹出窗口点击“否”
```

* 遇到 zsh: corrupt history file /home/user/.zsh_history 如何解決

```shell
# 1. cd ~
cd ~
# 2. mv 
mv .zsh_history .zsh_history_bad
# 3. strings
# -a --all：扫描整个文件而不是只扫描目标文件初始化和装载段
# -f –print-file-name：在显示字符串前先显示文件名
# -n –bytes=[number]：找到并且输出所有NUL终止符序列
# - ：设置显示的最少的字符数，默认是4个字符
# -t --radix={o,d,x} ：输出字符的位置，基于八进制，十进制或者十六进制
# -o ：类似--radix=o
# -T --target= ：指定二进制文件格式
# -e --encoding={s,S,b,l,B,L} ：选择字符大小和排列顺序:s = 7-bit, S = 8-bit, {b,l} = 16-bit, {B,L} = 32-bit
# @ ：读取中选项

strings -eS .zsh_history_bad > .zsh_history
# 4. fc
fc -R .zsh_history
```

* lazyload 未尝试

```shell
sukka_lazyload_add_command() {
    eval "$1() { \
        unfunction $1 \
        _sukka_lazyload__command_$1 \
        $1 \$@ \
    }"
}

sukka_lazyload_add_completion() {
    local comp_name="_sukka_lazyload__compfunc_$1"
    eval "${comp_name}() { \
        compdef -d $1; \
        _sukka_lazyload_completion_$1; \
    }"
    compdef $comp_name $1
}

_sukka_lazyload__command_pyenv() {
  # pyenv 初始化
  eval "$(command pyenv init -)"
}
_sukka_lazyload__compfunc_pyenv() {
  # pyenv 命令补全
  source "$(brew --prefix pyenv)/completions/pyenv.zsh"
}
# 添加 pyenv 的 lazyload
sukka_lazyload_add_command pyenv
sukka_lazyload_add_completion pyenv

_sukka_lazyload__command_fuck() {
  # fuck 初始化
  eval $(thefuck --alias)
}
# 添加 fuck 的 lazyload
sukka_lazyload_add_command fuck

_sukka_lazyload__completion_hexo() {
  # hexo 的 completion
  eval $(hexo --completion=zsh)
}
# 添加 hexo completion 的 lazyload
sukka_lazyload_add_completion hexo
```

* 关闭 oh-my-zsh 的 dirty 检查

```shell
git config --global oh-my-zsh.hide-dirty 1
git config --global oh-my-zsh.hide-status 1
```
