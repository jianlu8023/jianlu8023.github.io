+++
title = 'linux上tmux的使用'
date = '2025-06-30T17:15:34+08:00'
author = 'jianlu'
draft = false
description = "linux上tmux的使用"
aliases = ["linux", "tmux"]
+++

# 安装

```shell

sudo apt install tmux

sudo yum install tmux
```

# 常用操作

```text
# 查看有所有tmux会话
指  令：tmux ls
快捷键：Ctrl+b s

# 新建tmux窗口
指  令：tmux new -s <session-name>

# 重命名会话
指  令：tmux rename-session -t <old-name> <new-name>
快捷键：Ctrl+b $

# 分离会话
指  令：tmux detach  或者使用  exit(关闭窗口)
快捷键：Ctrl+b d

# 重新连接会话
指  令：tmux attach -t <session-name>  或者使用 tmux at -t <session-name>

#平铺当前窗格（个人很喜欢的快捷键，注意：平铺的是当前选中的窗格）
快捷键：Ctrl+b z (再次 Ctrl+b z 则恢复)

# 杀死会话
指  令：tmux kill-session -t <session-name>

# 切换会话
指  令：tmux switch -t <session-name>

# 划分上下两个窗格
指  令：tmux split
快捷键：Ctrl+b “

# 划分左右两个窗格
指  令：tmux split -h
快捷键：Ctrl+b %

# 光标切换到上方窗格
指  令：tmux select-pane -U
快捷键：Ctrl+b 方向键上

# 光标切换到下方窗格
指  令：tmux select-pane -D
快捷键：Ctrl+b 方向键下

# 光标切换到左边窗格
指  令：tmux select-pane -L
快捷键：Ctrl+b 方向键左

# 光标切换到右边窗格
指  令：tmux select-pane -R
快捷键：Ctrl+b 方向键右
```

# 常用命令

```text
ctrl+b 激活控制台
举个例子：
    帮助命令的快捷键是Ctrl+b ?
    它的用法是：在 Tmux 窗口中，先按下Ctrl+b，再按下?，就会显示帮助信息。
```

## 系统操作

```text
?	列出所有快捷键；按q返回
d	脱离当前会话；这样可以暂时返回Shell界面，输入tmux attach能够重新进入之前的会话
D	选择要脱离的会话；在同时开启了多个会话时使用
Ctrl+z	挂起当前会话
r	强制重绘未脱离的会话
s	选择并切换会话；在同时开启了多个会话时使用
:	进入命令行模式；此时可以输入支持的命令，例如kill-server可以关闭服务器
[	进入复制模式；此时的操作与vi/emacs相同，按q/Esc退出
~	列出提示信息缓存；其中包含了之前tmux返回的各种提示信息
```

## 窗口操作

```text
c	创建新窗口
&	关闭当前窗口
数字键	切换至指定窗口
p	切换至上一窗口
n	切换至下一窗口
l	在前后两个窗口间互相切换
w	通过窗口列表切换窗口
,	重命名当前窗口；这样便于识别
.	修改当前窗口编号；相当于窗口重新排序
f	在所有窗口中查找指定文本
```

## 面板操作

```text
”	        将当前面板平分为上下两块
%	        将当前面板平分为左右两块
x	        关闭当前面板
!	        将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板
Ctrl+方向键	以1个单元格为单位移动边缘以调整当前面板大小
Alt+方向键	以5个单元格为单位移动边缘以调整当前面板大小
Space	        在预置的面板布局中循环切换；依次包括even-horizontal、even-vertical、main-horizontal、main-vertical、tiled
q	        显示面板编号
o	        在当前窗口中选择下一面板
方向键	        移动光标以选择面板
{	        向前置换当前面板
}	        向后置换当前面板
Alt+o	        逆时针旋转当前窗口的面板
Ctrl+o	        顺时针旋转当前窗口的面板
```

# 配置

```text
tmux启动后 会调用~/.tmux.conf文件中的配置

更改文件后 需要在tmux中执行source ~/.tmux.conf
ctrl+b => shift+: => source ~/.tmux.conf

```

## 常用配置项

```text
# 右下角类似效果：21:58:48 12-12
set -g status-right "%H:%M:%S %d-%b"

# 会话计数：从 1 开始（Setting base-index assures newly created windows start at 1 and count upwards）
set -g base-index 1
# 窗口计数：从 1 开始编号，而不是从 0 开始
set -g pane-base-index 1
# 关掉某个窗口后，编号重排
set -g renumber-windows   on    

set -g status-interval 1    # 状态栏刷新时间(右下角秒针会跳动)
set -g status-justify left  # 状态栏窗口列表(window list)左对齐
set -g visual-activity on # 启用活动警告
set -wg monitor-activity on # 非当前窗口有内容更新时在状态栏通知
set -g message-style "bg=#202529, fg=#91A8BA" # 指定消息通知的前景、后景色

set -wg window-status-current-format " #I:#W#F " # 状态栏当前窗口名称格式(#I：序号，#w：窗口名 称，#F：间隔符)
set -wg window-status-current-style "fg=#d7fcaf,bg=#60875f" # 状态栏当前窗口名称的样式
set -wg window-status-separator "" # 状态栏窗口名称之间的间隔

# 鼠标支持 - 设置为 on 来启用鼠标(与 2.1 之前的版本有区别，请自行查阅 man page)
set -g mouse on

# 命令回滚/历史数量限制
set -g history-limit 20480
set -sg escape-time 0
set -g display-time 1500
set -g remain-on-exit off

# Vim 风格的快捷键实现窗格间移动 
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# 编码
# 设置默认终端模式为 256color
set -g default-terminal "screen-256color"

# 启用 UTF-8 编码
setw -g utf8 on
set -g status-utf8 on
```

# 懒人启动脚本

## 启动

### 启动脚本

```shell
#!/usr/bin/env bash

echo "create windows count:"
read winCnt

cmd=$(which tmux) # tmux path
session=work # session name

if [ -z $cmd ]; then
    echo "You need to install tmux."
    exit 1
fi

$cmd has -t $session

if [ $? != 0 ]; then
    #new session, window name is "env"
    $cmd new -s $session -d -n window0

    for ((i=0; i < winCnt; i++))
    do
        winName=window$i
        echo $winName
        
        if [ $winName != window0 ]; then
            #new window "i"
            $cmd neww -n $winName -t $session -d

            $cmd selectw -t $session:$winName

        fi

        #split window ""
        $cmd splitw -v -p 50 -t $winName
        $cmd splitw -h -p 50 -t $winName
        $cmd selectp -t 0
        $cmd splitw -h -p 50 -t $winName
        $cmd selectp -t 0
    done

    #select first window
    $cmd selectw -t $session:window0
fi

$cmd att -t $session

exit 0
```

### 使用

```shell
bash start.sh 
# 输入3 表示创建3个窗口
# 每个窗口分4个panel
```

## 关闭

### 关闭脚本

```shell
#!/usr/bin/env bash

tmux kill-session -t work
```

### 使用

```shell
bash stop.sh
```
