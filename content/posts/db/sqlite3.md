+++
title = 'Sqlite操作'
date = '2025-06-04T19:46:49+08:00'
author = 'jianlu'
draft = false
description = "sqlite3操作"
aliases = ["sqlite3操作", "sqlite3"]
+++

# sqlite3 操作相关

```text
查看所有表名:

    select name from sqlite_master where type='table' order by name; 

查看表的字段:

   	PRAGMA table_info([tablename]);   tablename 为实际的数据表名
```

# 修改sqlite字段类型

```text
1.把原表改成另外一个名字作为暂存表

    ALTER TABLE old_table_name RENAME TO temp_table_name;

如果需要，可以删除原表的索引

    DROP INDEX ix_name;

2. 用原表的名字创建新表

    CREATE TABLE old_table_name(field_name INTEGER PRIMARY KEY AUTOINCREMENT, other_field_name text not null);

如果需要，可以创建新表的索引

    CREATE INDEX ix_name ON old_table_name(field_name);

3. 将暂存表数据写入到新表，很方便的是不需要去理会自动增长的 ID

    INSERT INTO old_table_name SELECT * FROM temp_table_name

4. 删除暂存表

    DROP TABLE temp_table_name;
```

## sqlite3 golang 打包操作

```dockerfile
FROM golang AS gobuilder

ENV CGO_ENABLED=1
ENV CGO_CFLAGS="-g -O2 -Wno-return-local-addr"

HEALTHCHECK --interval=60s --timeout=5s --retries=3 --start-period=30s CMD curl -ksS https://localhost:8080/ || exit 1
```
