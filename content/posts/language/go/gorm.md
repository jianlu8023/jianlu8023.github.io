+++
title = 'Gorm注意事项'
date = '2025-04-11T11:05:00+08:00'
author = 'jianlu'
draft = false
description = "gorm注意事项"
aliases = ["gorm", "golang"]
+++

## 链接

* [gorm v1](https://v1.gorm.io/zh_CN/docs/)
* [gorm v2](https://gorm.io/zh_CN/docs/index.html)

## gorm 自增主键

```go
package main

type API struct {
	Id1 int64 `json:"id" gorm:"column:id;primary_key;AUTO_INCREMENT"` // 主键自增 成功
	Id2 int64 `json:"id" gorm:"column:id;primaryKey;AUTO_INCREMENT"`  // 主键自增 成功
	Id3 int64 `json:"id" gorm:"column:id;primaryKey;autoIncrement"`   // 主键自增 成功
	Id4 int64 `json:"id" gorm:"column:id;primary_key;autoIncrement"`  // 主键自增 成功
	
	Id5 int64 `json:"id" gorm:"column:id;type:bigint(20);primaryKey;auto_increment"` // 主键不自增 失败
	Id6 int64 `json:"id" gorm:"column:id;type:bigint(20);primaryKey;autoIncrement"`  // 主键不自增 失败
	Id7 int64 `json:"id" gorm:"column:id;type:bigint(20);primaryKey;AUTO_INCREMENT"` // 主键不自增 失败
	
	Id8  int64 `json:"id" gorm:"column:id;type:bigint(20) auto_increment;primary_key"` // 主键自增 成功
	Id9  int64 `json:"id" gorm:"column:id;type:bigint(20) autoIncrement;primaryKey"`   // Error 1064: You have an error in your SQL syntax; 
	Id10 int64 `json:"id" gorm:"column:id;type:bigint(20) AUTO_INCREMENT;primaryKey"`  // 主键自增 成功
	
	Id11 int64 `json:"id" gorm:"column:id;primaryKey;type:bigint(20) autoIncrement;"`  // Error 1064: You have an error in your SQL syntax;
	Id12 int64 `json:"id" gorm:"column:id;primaryKey;type:bigint(20) auto_increment;"` // 主键自增 成功
	Id13 int64 `json:"id" gorm:"column:id;type:bigint(20) auto_increment;primaryKey"`  // 主键自增 成功
}

```

* 结论

```text
如果你不需要指定类型 
Id      int64  `json:"id" gorm:"column:id;primaryKey;AUTO_INCREMENT"`

如果你想要指定类型   auto_increment 作为type 的属性
Id           int64  `json:"id" gorm:"column:id;type:bigint(20) auto_increment;primaryKey"`
```

## comment 区别


### 说明

```text
gorm v1 中，为字段添加说明信息时，需要用 引号包裹 内容： comment:'创建时间' 
gorm v2 中，不需要 引号 了：comment:创建时间
```

### 对比

```go
package main

type User struct {
	// gorm v1
	CreateTimeV1 int64 `json:"create_time" gorm:"column:create_time;comment:'创建时间'"`
	
	// gorm v2
	CreateTimeV2 int64 `json:"create_time" gorm:"column:create_time;comment:创建时间"`
}
```


## 参考
* [auto_increment无效](https://blog.csdn.net/qq1261275789/article/details/123606229)
