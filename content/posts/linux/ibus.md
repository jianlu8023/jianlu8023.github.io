+++
title = 'linux安装ibus'
date = 2024-12-23T10:57:12+08:00
author = "jianlu"
draft = false
description = "linux安装ibus输入法"
aliases = ["ibus-rime", "ibus"]
+++

1. 卸载旧输入法并删除配置文件
```shell
# 卸载其他输入法
sudo apt purge sougoupinyin
sudo apt remove sougoupinyin*
sudo apt purge fcitx
sudo apt remove fcitx*
sudo apt autoremove

# 删除配置文件
cd ~/.config

rm -rf sougoupintin
rm -rf ibus
```

2. 安装ibus
```shell
apt install ibus ibus-rime
```

3. 配置ibus
```text
1. 打开设置
2. 区域与语言 -> 输入源 -> [+](加号)
3. 选择 中文rime
4. 删除其他不需要的输入法
5. 选择ibus
```

4. 重启ibus
```shell
ibus restart
```

5. 个性化配置

* 方案一 简单配置

```shell
cd ~/.config/ibus/rime

vi default_custom.yaml

# 内容
patch:
  schema_list:
    - schema: luna_pinyin_simp
  menu:
    page_size: 5
  ascii_composer:
    switch_key:
      Shift_L: commit_code

# 设置横排显示
cd ~/.config/ibus/rime/build/

vi ibus_rime.yaml

# 内容
style:
  horizontal: true

# 重启ibus
ibus restart
```

* 方案二 使用已有项目

```shell
# 1. clone 项目
git clone https://github.com/ssnhd/rime.git ~/.config/ibus/

# 重启ibus
ibus restart
```

Tips:

* ibus输入法导致wps启动慢

```shell
sudo apt install libcanberra-gtk-module
sudo apt install appmenu-gtk2-module
```




