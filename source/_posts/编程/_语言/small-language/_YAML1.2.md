---
title: YAML
date: 2024-04-20
update: 2024-04-20
tags:
categories:
  - small language
keywords: YAML
description: 主要介绍YAML基础
top_img:
cover:
---



# YAML1.2

YAML是一种数据序列化语言而不是文档标记语言，常用于配置文件(SpringBoot、Docker)

[官网](https://yaml.org/)	|	[在线编辑器](https://yaml.cn/)



## 格式

文件以 `.yaml` `.yml` 结尾

不跨行的`: `后需要空格

缩进对齐一致(2或其他无所谓)

大小写敏感



## 变量类型

变量类型：对象、数组、字符串、布尔、数字
key 永远为字符串



## 语法

```yaml
---  # 表示文档开始
# 单行注释
array1: [s1, true, 3.14]  # 数组
array2:  # 数组
	- ch1 ch2  # string "ch1 ch2"
	- true  # bool
	- 3.14  # num
```

```yaml
any_name1: &config1  # 锚点Anchor, any_name没作用
    k1: v1
service:
    k3: v3
    <<: *config1  # merge
```

```yaml
any_name2: &config2
k2: *config2  # 引用
```

```yaml
any_name3: 
- &config1: {
	address: 127.0.0.1,
  	address2: 127.0.0.1
}
- &config2: {address: 127.0.0.1, address2: 127.0.0.1}  # 同时定义多个锚点
```

& 锚点

\* 引用

<< 合并键
