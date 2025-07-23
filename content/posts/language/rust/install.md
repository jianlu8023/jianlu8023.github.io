+++
title = 'rust安装'
date = '2025-07-23T17:48:03+08:00'
author = 'jianlu'
draft = false
description = "安装rust"
aliases = ["rust", "rust安装", "自定义安装目录"]
+++

## linux 自定义安装rust目录

1. 在使用的终端(bash\zsh等)中添加如下配置

```shell
export CARGO_HOME=/usr/local/dev/rust/cargo # cargo安装目录
export RUSTUP_HOME=/usr/local/dev/rust/rustup # rustup安装目录
# 默认情况下 
# rustup_dist_server=https://static.rust-lang.org
# rustup_update_root=https://static.rust-lang.org/rustup
export RUSTUP_UPDATE_ROOT=https://mirrors.aliyun.com/rustup/rustup # 用于更新rustup
export RUSTUP_DIST_SERVER=https://mirrors.aliyun.com/rustup # 用于更新toolchain

# 中科大
# dist_server=https://mirrors.ustc.edu.cn/rust-static
# update_root=https://mirrors.ustc.edu.cn/rust-static/rustup

# 清华
# dist_server=https://mirrors.tuna.tsinghua.edu.cn/rustup
# update_root=https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup

# 添加这里的目录是安装后 直接可以执行cargo命令
export PATH=$CARGO_HOME/bin:$PATH
```

2. 使配置生效

```shell
source ~/.zshrc
```

3. 安装rust

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# 此时 安装说明就会输出在上方定义的cargo_home rustup_home目录 可以输入2 然后选择调整 然后输入1 或者直接输入1
```

4. 验证是否成功

```shell
cargo --version
rustc --version
# 如果输出版本号 则说明安装成功
# 否则 source ~/.zshrc再次尝试
```

5. 添加组件(rust-src)

```shell
rustup component add rust-src
```

6. rustrover 配置

```text
Toolchain location 填写 $RUSTUP_HOME/toolchains/stable-x86_64-unknown-linux-gnu/bin
# 如果没有stable-x86_64-unknown-linux-gnu[是安装rust的时候安装的] 则需要手动添加 rustup toolchain install stable

Standard library 填写 $RUSTUP_HOME/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library
# 如果rustlib下没有src 则需要手动添加 rustup component add rust-src
```

7. 配置国内镜像

```shell
# 方式一 修改config文件
cd $CARGO_HOME 
touch config
# 添加内容
```

* 内容

```toml
#[registry]
#index = "https://github.com/rust-lang/crates.io-index"
[source.crates-io]
# 指定官方 crates.io 源。
registry = "https://github.com/rust-lang/crates.io-index"

# 指定使用下面哪个源，修改为source.后面的内容即可
replace-with = 'aliyun'

# rsproxy
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

#阿里云
[source.aliyun]
registry = "sparse+https://mirrors.aliyun.com/crates.io-index/"

# 中国科学技术大学
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"

# 上海交通大学
[source.sjtu]
registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index/"

# 清华大学
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

# rustcc社区
[source.rustcc]
registry = "https://code.aliyun.com/rustcc/crates.io-index.git"

[net]
# 配置网络设置。 
# 用于解决某些情况下 `git2` 依赖的问题，特别是当你的系统 Git 配置和 Cargo 的 `git2` 库不兼容时。
git-fetch-with-cli = true
```

```shell
# 方式二 修改环境变量 只对当前shell有效
#CARGO_REGISTRIES_CRATES_IO_REPLACE_WITH: 指定要替换 crates.io 的源的名称
#CARGO_SOURCE_<源名称>_REGISTRY: 指定源的 registry 地址

# 举例
export CARGO_REGISTRIES_CRATES_IO_REPLACE_WITH='ustc'
export CARGO_SOURCE_USTC_REGISTRY='git://mirrors.ustc.edu.cn/crates.io-index'
```

8. 依赖库查找网址

* [lib.rs](https://lib.rs/)
* [crates.io](https://crates.io/)
