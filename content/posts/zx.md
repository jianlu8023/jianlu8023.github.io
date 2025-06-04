+++
title = '杂项'
date = '2025-06-04T19:53:49+08:00'
author = 'jianlu'
draft = false
description = "杂项"
aliases = ["杂项"]
+++

## nmap 扫描端口

```shell
nmap -sV --script ssl-enum-ciphers -p 3389 test.com
```

## logrotate

```text
/app/logs/*.log {
    daily                 # 每天轮转日志
    rotate 7              # 保留最近7个轮转的日志文件
    size 100M             # 当日志文件大小超过100MB时进行轮转
    compress              # 轮转后的日志文件使用gzip压缩
    delaycompress         # 延迟压缩，下一次轮转时才压缩上一次轮转的文件。这样可以避免正在写入的日志文件被压缩
    dateext               # 使用日期作为轮转日志文件的扩展名
    dateformat -%Y-%m-%d  # 日期格式为 -YYYY-MM-DD，例如：logfile-2023-10-27.log
    missingok             # 如果日志文件丢失，不报错，继续处理下一个日志文件
    notifempty            # 如果日志文件为空，不进行轮转
    copytruncate          # 复制原日志文件，然后清空原日志文件。这可以确保正在写入的程序可以继续写入日志，而不会丢失任何数据
    create 0640 1000 1000 # 创建新的日志文件，权限为0640，所有者和组ID都为1000
}
```
