+++
title = 'ubuntu 安装 consolas 字体'
date = '2025-06-05T16:55:45+08:00'
author = 'jianlu'
draft = false
description = "ubuntu 安装 consolas 字体"
aliases = ["ubuntu", "consolas"]
+++

# 参考

* [不能用](https://www.oryoy.com/news/ubuntu-xi-tong-wan-mei-da-pei-consolas-zi-ti-an-zhuang-yu-you-hua-zhi-nan.html)

## 方式一

```shell
wget https://down.gloriousdays.pw/Fonts/Consolas.zip
unzip Consolas.zip
mkdir ~/.fonts/consolas
cp consola*.ttf ~/.fonts/consolas
cd ~/.fonts/consolas
mkfontscale
mkfontdir
fc-cache -fv ~/.fonts
```

[//]: # (https://blog.gloriousdays.pw/2018/08/29/change-font-within-vsc-to-consolas-on-linux/)

## 方式二

```shell
wget https://img.yangsihan.com/YaHeiConsolas.tar.gz
mkdir -p ~/.fonts/yahei-consolas
tar -zxvf YaHeiConsolas.tar.gz -C ~/.fonts/yahei-consolas
cd ~/.fonts/yahei-consolas
mkfontscale
mkfontdir
fc-cache -fv ~/.fonts
# 安装 tweak tool 重新设定下载的字体
```
