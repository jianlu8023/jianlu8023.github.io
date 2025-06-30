+++
title = 'linux上rsync和scp'
date = '2025-06-30T16:49:03+08:00'
author = 'jianlu'
draft = false
description = "linux上rsync和scp的使用"
aliases = ["rsync", "scp", "linux"]
+++

# rsync

## 安装

```shell
# ubuntu
sudo apt-get install rsync

# centos
sudo yum install rsync
```

## 基本使用

```shell
# -r 递归 
# -a 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
# -n 测试同步过程，并不真正同步
# --delete 删除目标文件夹中源文件夹不存在的文件
# --exclude 排除不需要同步的文件
# --include 只同步需要的文件
# --link-dest 使用硬链接方式同步
# --append 只同步文件的追加部分
# --append-verify 只同步文件的追加部分，并校验
# -b --backup 备份文件 指定在删除或更新目标目录已经存在的文件时，将该文件更新后进行备份 
#             默行为是删除 更名规则时添加--suffix 
# --backup-dir 指定备份文件的目录
# --bwlimit 限速 --bwlimit=100k
# -c --checksum 生成文件的校验码 默认情况下，rsync 只检查⽂件的⼤⼩和最后修改⽇期是否发⽣变化，
#               如果发⽣变化，就重新传输；使⽤这个参数以后，则通过判断⽂件内容的校验和，决定是否重新传输
# --ignore-existing 忽略已经存在的文件
# --max-size 限制文件大小
# --min-size 限制文件大小
# -P --progress+--partial 显示同步过程，并保留那些因故没有完全同步的文件，以便下次继续向该文件传输


# ssh协议 -e 'ssh -p 2234' 
rsync -avzP -e 'ssh -p 2234' /home/user/test root@192.168.1.1:/home/user/test
rsync -avzP -e 'ssh -p 2234' root@192.168.1.1:/home/user/test /home/user/test
```

## nohup方式后台运行

```shell
nohup rsync -e 'ssh -p 2234' -avzP /home/user/test root@192.168.1.1:/home/user/test 
# 服务器输出 
# nohup: appending output to 'nohup.out'
# Password:

# 输入密码后ctrl+z 中断进程
# 服务器输出
# [1]+ Stopped nohup rsync -e 'ssh -p 2234' -avzP /home/user/test root@192.168.1.1:/home/user/test

bg 
# 服务器输出
# [1]+ nohup rsync -e 'ssh -p 2234' -avzP /home/user/test root@192.168.1.1:/home/user/test

# 查看进度
tail -f -n 200 nohup.out

```


# scp

## 基本使用

```shell
scp -P 2234 -r /home/user/test root@192.168.1.1:/home/user/test
scp -P 2234 -r root@192.168.1.1:/home/user/test /home/user/test
```
