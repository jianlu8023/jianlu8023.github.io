+++
title = 'docker部署etcd'
date = '2025-01-17T11:49:16+08:00'
author = 'jianlu'
draft = false
description = "docker方式 部署 etcd podman也适用"
aliases = ["docker", "etcd","podman"]
+++

# 单节点部署

## 挂载配置文件

```yaml
networks:
  etcd:
    name: "etcd"

services:
  etcd:
    image: quay.io/coreos/etcd:v3.5.17
    container_name: etcd
    hostname: etcd
    networks:
      - etcd
    ports:
      - "2379:2379"
      - "2380:2380"
    volumes:
      - ./compose/etcd/data:/etcd-data
      - ./compose/etcd/log:/var/log/etcd/
      - ./compose/etcd/conf/etcd.conf.yml:/etc/etcd/etcd.conf.yml
    environment:
      - ETCD_CONFIG_FILE=/etc/etcd/etcd.conf.yml
    restart: always
```

* 使用的etcd配置文件

```yaml
# This is the configuration file for the etcd server.

# Human-readable name for this member.
name: 'etcd'

# Path to the data directory.
data-dir: /etcd/data

# Path to the dedicated wal directory.
wal-dir:

# Number of committed transactions to trigger a snapshot to disk.
snapshot-count: 10000

# Time (in milliseconds) of a heartbeat interval.
heartbeat-interval: 100

# Time (in milliseconds) for an election to timeout.
election-timeout: 1000

# Raise alarms when backend size exceeds the given quota. 0 means use the
# default quota.
quota-backend-bytes: 0

# List of comma separated URLs to listen on for peer traffic.
listen-peer-urls: http://0.0.0.0:2380

# List of comma separated URLs to listen on for client traffic.
listen-client-urls: http://0.0.0.0:2379

# Maximum number of snapshot files to retain (0 is unlimited).
max-snapshots: 5

# Maximum number of wal files to retain (0 is unlimited).
max-wals: 5

# Comma-separated white list of origins for CORS (cross-origin resource sharing).
cors:

# List of this member's peer URLs to advertise to the rest of the cluster.
# The URLs needed to be a comma-separated list.
initial-advertise-peer-urls: http://localhost:2380

# List of this member's client URLs to advertise to the public.
# The URLs needed to be a comma-separated list.
advertise-client-urls: http://localhost:2379

# Discovery URL used to bootstrap the cluster.
discovery:

# Valid values include 'exit', 'proxy'
discovery-fallback: 'proxy'

# HTTP proxy to use for traffic to discovery service.
discovery-proxy:

# DNS domain used to bootstrap initial cluster.
discovery-srv:

# Comma separated string of initial cluster configuration for bootstrapping.
# Example: initial-cluster: "infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380"
initial-cluster:

# Initial cluster token for the etcd cluster during bootstrap.
initial-cluster-token: 'etcd-cluster'

# Initial cluster state ('new' or 'existing').
initial-cluster-state: 'new'

# Reject reconfiguration requests that would cause quorum loss.
strict-reconfig-check: false

# Enable runtime profiling data via HTTP server
enable-pprof: true

# Valid values include 'on', 'readonly', 'off'
proxy: 'off'

# Time (in milliseconds) an endpoint will be held in a failed state.
proxy-failure-wait: 5000

# Time (in milliseconds) of the endpoints refresh interval.
proxy-refresh-interval: 30000

# Time (in milliseconds) for a dial to timeout.
proxy-dial-timeout: 1000

# Time (in milliseconds) for a write to timeout.
proxy-write-timeout: 5000

# Time (in milliseconds) for a read to timeout.
proxy-read-timeout: 0

client-transport-security:
  # Path to the client server TLS cert file.
  cert-file:

  # Path to the client server TLS key file.
  key-file:

  # Enable client cert authentication.
  client-cert-auth: false

  # Path to the client server TLS trusted CA cert file.
  trusted-ca-file:

  # Client TLS using generated certificates
  auto-tls: false

peer-transport-security:
  # Path to the peer server TLS cert file.
  cert-file:

  # Path to the peer server TLS key file.
  key-file:

  # Enable peer client cert authentication.
  client-cert-auth: false

  # Path to the peer server TLS trusted CA cert file.
  trusted-ca-file:

  # Peer TLS using generated certificates.
  auto-tls: false

  # Allowed CN for inter peer authentication.
  allowed-cn:

  # Allowed TLS hostname for inter peer authentication.
  allowed-hostname:

# The validity period of the self-signed certificate, the unit is year.
self-signed-cert-validity: 1

# Enable debug-level logging for etcd.
log-level: debug

logger: zap

# Specify 'stdout' or 'stderr' to skip journald logging even when running under systemd.
log-outputs: [stdout,stderr,/var/log/etcd/etcd.log]

# Enable log rotation of a single log-outputs file target.
enable-log-rotation: true

#Configures log rotation if enabled with a JSON logger config. MaxSize(MB), MaxAge(days,0=no limit), MaxBackups(0=no limit), LocalTime(use computers local time), Compress(gzip)".
log-rotation-config-json: '{"maxsize": 100, "maxage": 0, "maxbackups": 0, "localtime": false, "compress": false}'

# Force to create a new one member cluster.
force-new-cluster: false

auto-compaction-mode: periodic
auto-compaction-retention: "1"

# Limit etcd to a specific set of tls cipher suites
cipher-suites: [
  TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
  TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
]

# Limit etcd to specific TLS protocol versions 
tls-min-version: 'TLS1.2'
tls-max-version: 'TLS1.3'
```

[//]: #etcd配置文件 (https://github.com/etcd-io/etcd/blob/main/etcd.conf.yml.sample)

[//]: #配置项  (https://etcd.io/docs/v3.5/op-guide/configuration/#configuration-file)

## 环境变量

```yaml
networks:
  etcd:
    name: "etcd"

services:
  etcd:
    image: quay.io/coreos/etcd:v3.5.17
    container_name: etcd
    hostname: etcd
    networks:
      - etcd
    ports:
      - "2379:2379"
      - "2380:2380"
    volumes:
      - ./compose/etcd/data:/etcd-data
      - ./compose/etcd/log:/var/log/etcd/
    environment:
      - ETCD_LOG_LEVEL=debug
      - ETCD_LOG_OUTPUTS=stdout,/var/log/etcd/etcd.log
      - ETCD_ENABLE_LOG_ROTATION=true
      - ETCD_LOG_ROTATION_CONFIG_JSON='{"maxsize":100,"maxage:10,"maxbackups":10,"localtime":true,"compress":true}'
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379 # 监听所有网络接口的 2379 端口
      - ETCD_ADVERTISE_CLIENT_URLS=http://127.0.0.1:2379 # 客户端访问的地址
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380  # 监听所有网络接口的 2380 端口
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://127.0.0.1:2380 # peer 访问的地址
      - ETCD_INITIAL_CLUSTER=etcd=http://127.0.0.1:2380 # 初始化集群信息
      #- ETCD_INITIAL_CLUSTER=etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380 # 初始化集群信息
      - ETCD_INITIAL_CLUSTER_STATE=new
      - ETCD_DATA_DIR=/etcd-data
      - ETCD_NAME=etcd
    restart: always
```
