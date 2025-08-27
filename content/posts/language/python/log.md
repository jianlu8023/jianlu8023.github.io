+++
title = 'python使用logging的技巧'
date = '2025-06-05T11:31:59+08:00'
author = 'jianlu'
draft = false
description = "python使用logging的技巧"
aliases = ["python", "logging"]
+++


# 方式一 logging.basicConfig

```python

import logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s | %(levelname)-6s | %(module)s:%(lineno)d | %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

logger = logging.getLogger(__name__)
```


# 方式二 logging.config.dictConfig

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,

    'formatters': {
        'verbose': {
            'format': '[{levelname}] {asctime} {name} - {message}',
            'style': '{',
        },
        'standard': {
            'format': '%(asctime)s | %(levelname)-6s | %(module)s:%(lineno)d | %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S',
        },
    },

    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'standard',
            'level': 'INFO',  # 只处理 info 及以上的日志
        },
        'file': {
            'class': 'logging.FileHandler',
            'filename': os.path.join(log_dir,"pricomp.log"),
            'formatter': 'standard',
            'encoding': 'utf-8',
            'level': 'DEBUG',  # 处理 debug 及以上的日志
        },
    },
    'root': {
        'handlers': ['console', 'file'],
        'level': 'DEBUG',
    },
    'loggers': {
        '__main__': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',  # Logger 必须能接收 debug 日志
            'propagate': False,  # 防止冒泡到 root logger
        },
    }
}

logging.config.dictConfig(LOGGING)

logger = logging.getLogger(__name__)
```
