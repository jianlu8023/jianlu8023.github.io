+++
title = '移除ubuntu20.04系统的snapd'
date = '2025-06-23T17:41:21+08:00'
author = 'jianlu'
draft = false
description = "移除ubuntu20.04系统的snapd,ubuntu22.04也适用"
aliases = ["snapd", "ubuntu", "ubuntu20.04", "ubuntu22.04"]
+++

## 卸载所有已安装的snap软件

```shell
# 此卸载部分 一般要执行两道三次 直到出现 
# No snaps are installed yet. Try 'snap install hello-world'.

# 程序正确执行 会提示如下 
# snap "Name" is not installed
# core20 removed
# snapd removed
for p in $(snap list | awk '{print $1}'); do
  sudo snap remove $p
done
```

## 删除 Snap 的 Core 文件

```shell
sudo systemctl stop snapd
sudo systemctl disable --now snapd.socket

for m in /snap/core/*; do
   sudo umount $m
done
```

## 删除 Snap 的管理工具

```shell
sudo apt autoremove --purge snapd
```

## 删除 Snap 的目录

```shell
rm -rf ~/snap
sudo rm -rf /snap
sudo rm -rf /var/snap
sudo rm -rf /var/lib/snapd
sudo rm -rf /var/cache/snapd
```

## 配置 APT 参数：禁止 apt 安装 snapd

```shell
sudo sh -c "cat > /etc/apt/preferences.d/no-snapd.pref" << EOL
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOL
# 解读
# a -> Archive
# c -> Component
# o -> Origin
# l -> Label
# n -> Architecture

# 验证 
apt list snapd
#Listing... Done
#snapd/jammy-updates 2.56.2+22.04ubuntu1 amd64
#N: There is 1 additional version. Please use the '-a' switch to see it

sudo apt install snapd
#Reading package lists... Done
#Building dependency tree... Done
#Reading state information... Done
#Package snapd is not available, but is referred to by another package.
#This may mean that the package is missing, has been obsoleted, or
#is only available from another source
#
#E: Package 'snapd' has no installation candidate

```

## 禁用 snap Firefox 的更新（Server 版也可以配置）

```shell
sudo sh -c "cat > /etc/apt/preferences.d/no-firefox.pref" << EOL
Package: firefox
Pin: release a=*
Pin-Priority: -10
EOL
```

## 服务器版本 安装桌面环境 

```shell
sudo apt update
#sudo apt install ubuntu-desktop  #完整 GNOME 桌面环境包括 LibreOffice、视频播放器、游戏等等
sudo apt install ubuntu-desktop-minimal  #仅浏览器和基本工具
sudo reboot
```

# 安装软件商店和 Firefox

## 安装 Gnome 软件商店

```shell
sudo apt install gnome-software
# 确认没有 snapd 在按 y
#sudo apt install --install-suggests gnome-software
# 网传使用 `--install-suggests` 参数防止安装上 Snap 版本的软件包管理器，本例无效。
```

## 安装 DEB 格式的 Firefox

### 添加 Firefox 官方 PPA（Personal Package Archives）仓库

```shell
sudo add-apt-repository ppa:mozillateam/ppa
```

### 将 Firefox 官方 PPA 仓库中的 firefox 设为高优先级

```shell
sudo sh -c "cat > /etc/apt/preferences.d/mozillateam-firefox.pref" << EOL
Package: firefox*
Pin: release o=LP-PPA-mozillateam
Pin-Priority: 501
EOL
```

### 安装最新版的 deb 版 Firefox

```shell
sudo apt update
apt list firefox  #现在已经不是 Snap firefox
sudo apt install firefox
# 或者 Firefox ESR
sudo apt install firefox-esr
```


# 恢复 Snap 的方法

```shell
sudo rm /etc/apt/preferences.d/no-snap.pref
sudo rm /etc/apt/preferences.d/no-firefox.pref
sudo rm /etc/apt/preferences.d/mozillateam-firefox.pref
sudo apt update
sudo snap install snap-store
sudo apt install firefox
```

[//]: # (https://sysin.cn/blog/ubuntu-remove-snap/)
