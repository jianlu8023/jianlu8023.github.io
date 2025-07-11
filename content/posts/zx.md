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

## jetbrains 系列软件关闭 jcef_helper

```text
1. 打开 help -> edit custom properties 

2. 添加以下内容
ide.browser.jcef.enabled=false

3. 重启软件
```

## compose 限制资源占用

```yaml
networks:
  basic: # 定义一个名为 "basic" 的网络
    driver: bridge # 使用 bridge 网络驱动
    name: "basic_net"  # 网络的名称为 "basic_net"
    ipam: # IP 地址管理 (IPAM) 配置
      config:
        - subnet: 10.2.0.0/24 # 子网为 10.2.0.0/24
          gateway: 10.2.0.254 # 网关为 10.2.0.254
    enable_ipv6: false # 禁用 IPv6
    internal: false # 允许外部访问该网络 (非内部网络)
    attachable: false # 禁止外部容器连接到该网络
    driver_opts: # 驱动选项
      com.docker.network.bridge.mtu: "1500" # 设置mtu
    # docker network ls --filter "label=environment=basic" 查看指定环境标签的网络
    labels: # 标签 (元数据)
      environment: "basic" # 设置环境标签 

services:
  nginx:
    image: nginx:latest
    container_name: nginx
    networks:
      basic:
        ipv4_address: 10.2.0.2
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    environment:
      - TZ=Asia/Shanghai
    blkio_config:
      weight: 1000 # 设置块设备 I/O 权重为 1000 默认值500 范围 10-1000
      # bps 限制读取操作 吞吐量 适用场景 需要限制大数据量读取速度的场景  限制每秒最多读取多少数据
      device_read_bps: 
        - path: /dev/sda
          rate: 300kb
      device_write_bps:
        - path: /dev/sda
          rate: 300kb
      # iops 限制读取操作 iops 适用场景 需要限制频繁小文件读取的场景  限制每秒最多执行多少次读取操作
      device_read_iops:
        - path: /dev/sda
          rate: 1000
      device_write_iops:
        - path: /dev/sda
          rate: 1000
    deploy:
      resources:
        limits: # 限制资源
          cpus: "0.5"
          memory: 512m
        reservations: # 保留资源
          cpus: "0.25"
          memory: 256m
    depends_on:
      app:
        condition: service_healthy
        # condition: service_completed_successfully
        # condition: service_started
    restart: unless-stopped
  app:
    image: app-server:v0.0.1
    container_name: app
    networks:
      basic:
        - ipv4_address: 10.2.0.3
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./app/config:/app/config
    ports:
      - "9999:9999"
    blkio_config:
      weight: 1000
      device_read_bps:
        - path: /dev/sda
          rate: 300kb
      device_write_bps:
        - path: /dev/sda
          rate: 300kb
      device_read_iops:
        - path: /dev/sda
          rate: 1000
      device_write_iops:
        - path: /dev/sda
          rate: 1000
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512m
        reservations: 
          cpus: "0.25"
          memory: 256m
    labels:
      environment: "production"
    # dockerfile HEALTHCHECK --interval=60s --timeout=5s --retries=3 --start-period=30s CMD curl -ksS https://localhost:9999/healthz || exit 1
    healthcheck:
      test: [ "CMD","curl","-ksS","http://localhost:9999/healthz","||","exit 1" ] # 健康检查命令
      interval: 60s  # 健康检查间隔为 60 秒
      timeout: 5s # 健康检查超时时间为 5 秒
      retries: 3 # 健康检查失败重试次数为 3 次
      start_interval: 10s # 启动后首次健康检查的间隔为 10 秒
      start_period: 15s # 启动后开始健康检查的延迟时间为 15 秒
      disable: false # 启用健康检查
    restart: unless-stopped
  nvidia:
    image: nvidia/cuda:12.3.1-base-ubuntu20.04
    command: nvidia-smi
    container_name: nvidia-demo
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              device_ids:
                - 2
                - 3
              capabilities:
                - gpu
```
