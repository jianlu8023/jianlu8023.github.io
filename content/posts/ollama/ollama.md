+++
title = 'docker启动ollama'
date = '2025-01-14T17:23:37+08:00'
author = 'jianlu'
draft = false
description = "docker ollama"
aliases = ["ollama"]
+++

# docker启动ollama
```yaml
networks:
  ollama:
    name: "ollama"

services:
  ollama:
    image: ollama/ollama
    volumes:
      - ./compose/ollama/data:/root/.ollama
    ports:
      - "11434:11434"
    container_name: ollama
    networks:
      - ollama

```

# curl 执行一些操作
```shell
# ollama pull llama2

curl -X POST 'http://localhost:11434/api/pull' \
--header 'Content-Type: application/json' \
--data-raw '{
    "model":"llama2",
    "stream": true
}'

# ollama delete llama2
curl -X DELETE 'http://localhost:11434/api/delete' \
--header 'Content-Type: application/json' \
--data-raw '{
    "model":"llama2"
}'

# ollama version
curl -X GET 'http://localhost:11434/api/version'

# api/tags
curl -X GET 'http://localhost:11434/api/tags'
```

