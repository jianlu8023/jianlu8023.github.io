+++
title = '使用ssh xxx进行远程连接'
date = '2025-10-13T12:57:49+08:00'
author = 'jianlu'
draft = false
description = "配置ssh,实现ssh xxx进行远程连接"
aliases = ["ssh"]
+++

# 默认config文件

```text

# This is the ssh client system-wide configuration file.  See
# ssh_config(5) for more information.  This file provides defaults for
# users, and the values can be changed in per-user configuration files
# or on the command line.

# Configuration data is parsed as follows:
#  1. command line options
#  2. user-specific file
#  3. system-wide file
# Any configuration value is only changed the first time it is set.
# Thus, host-specific definitions should be at the beginning of the
# configuration file, and defaults at the end.

# Site-wide defaults for some commonly used options.  For a comprehensive
# list of available options, their meanings and defaults, please see the
# ssh_config(5) man page.

Include /etc/ssh/ssh_config.d/*.conf

Host *
#   ForwardAgent no
#   ForwardX11 no
#   ForwardX11Trusted yes
#   PasswordAuthentication yes
#   HostbasedAuthentication no
#   GSSAPIAuthentication no
#   GSSAPIDelegateCredentials no
#   GSSAPIKeyExchange no
#   GSSAPITrustDNS no
#   BatchMode no
#   CheckHostIP yes
#   AddressFamily any
#   ConnectTimeout 0
#   StrictHostKeyChecking ask
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   IdentityFile ~/.ssh/id_ecdsa
#   IdentityFile ~/.ssh/id_ed25519
#   Port 22
#   Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
#   MACs hmac-md5,hmac-sha1,umac-64@openssh.com
#   EscapeChar ~
#   Tunnel no
#   TunnelDevice any:any
#   PermitLocalCommand no
#   VisualHostKey no
#   ProxyCommand ssh -q -W %h:%p gateway.example.com
#   RekeyLimit 1G 1h
    SendEnv LANG LC_*
    HashKnownHosts yes
    GSSAPIAuthentication yes
```

# 编写 config

* tips: win在 c:/Users/用户名/.ssh/config
* linux 全局 /etc/ssh/ssh_config 个人 ~/.ssh/config

```text
# 加载config-*的配置
Include config-*

# 可以放config内 也可以单独一个文件 然后Include即可 
# 假设写在 config-188 
# include 写 Include config-188 或者 Include config-*
Host xxx # 别名 此处xxx 为别名 连接 ssh xxx 即可
  Hostname 10.0.0.188 # ip 
  User root # 用户名
  Port 49022 # 端口
  IdentityFile ~/.ssh/id_rsa # 私钥 如果没有删掉此行
  ServerAliveInterval 180 # 在建立连接后，每180秒客户端会向服务器发送一个心跳，避免用户长时间没有操作连接中断

# 通用配置 
Host Server*
  User zhangsan
  Port 22
  ServerAliveInterval 180
# 会使用Server的配置 
Host Server1
  HostName 10.0.0.188
Host Server2
  HostName 10.0.0.189


# 跳板配置
# 为什么要使用跳板机？
# 使用跳板机通常是出于安全考虑。 
# 例如，你可能不想直接将内部服务器暴露在公网上，而是只允许通过一个安全的跳板机访问。 
# 这样可以集中管理访问权限，并减少安全风险
Host jumper
  Hostname 10.0.0.122
  User zhangsan
# 它指定了通过 jumper 这个 Host 作为跳板机。
# 也就是说，要连接到任何匹配 Server* 的机器，都需要先连接到 jumper (也就是 10.0.0.122)，
# 然后再从 jumper 连接到目标服务器
Host Server*
  User zhangsan
  ProxyJump jumper
  ServerAliveInterval 180
Host Server1
  Hostname 10.0.0.123
```

# authorized_keys

```text
文件用于 SSH 公钥认证 简单来说，它允许你使用密钥对（公钥和私钥）来登录 SSH 服务器，而无需输入密码
位置: 通常位于用户的 .ssh 目录下，即 ~/.ssh/authorized_keys (例如 /home/你的用户名/.ssh/authorized_keys)

工作原理:

当你尝试使用 SSH 密钥登录服务器时，SSH 客户端会使用私钥进行签名。
服务器会在用户的 authorized_keys 文件中查找与该私钥对应的公钥。
如果找到匹配的公钥，并且签名验证成功，服务器就会允许你登录，而无需密码。
```

# 生成ssh密钥对
```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# -t rsa: 指定密钥类型为 RSA (一种常用的加密算法)。也可以使用 -t ed25519 (更安全的算法，如果服务器支持)。
# -b 4096: 指定密钥长度为 4096 位 (对于 RSA 来说，这是推荐的长度)。
# -C "your_email@example.com": 添加一个注释，通常是你的电子邮件地址。这只是一个标识，不会影响密钥的功能

# 选择保存位置: ssh-keygen 会提示你输入保存密钥的文件路径。 默认是 ~/.ssh/id_rsa (私钥) 和 ~/.ssh/id_rsa.pub (公钥)。 建议使用默认位置，除非你有特殊需求

# ~/.ssh/id_rsa: 私钥 (private key)。 务必妥善保管！ 不要泄露给任何人！
# ~/.ssh/id_rsa.pub: 公钥 (public key)。 你需要将这个公钥复制到服务器的 authorized_keys 文件中

# 分发公钥 1
ssh-copy-id root@10.0.0.188
# 将 user 替换为你在服务器上的用户名。
# 将 server_ip_address 替换为服务器的 IP 地址或域名。
# ssh-copy-id 会自动连接到服务器，提示你输入密码，然后将你的公钥添加到 ~/.ssh/authorized_keys 文件中

# 分发公钥 2
cat ~/.ssh/id_rsa.pub | ssh user@server_ip_address "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```
