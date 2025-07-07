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
	- ''  # 字面量字符串，不转义
	- ""  # 转义\
... # 表示文档结束
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

```yaml
image: ${IMAGE}  # 配置.env环境变量使用
```

```env
IMAGE=clickhouse/clickhouse-server:latest
```



# toml

最小化配置，不支持锚点和引用

```toml
# 注释
title = """TOML
Example\
test"""

winpath = 'C:\Users\Toml'  # 字面量字符串，不转义
str = ""  # 转义\

b = [
	{x=1, y=2, z=3},  # 行内表
	{x=1, y=2, z=3},
	'',
	""
]

c.x = "c.x"  # 点分隔符定义表
c.y = "c.y"

[d]  # [tables_name.sub_table_name]  # 标准表
x = "d.x"
y = "d.y"

[[e]]  # table_array
x = "e[0].x"
y = "e[0].y"

[e.physical]  # 任何对表数组的引用都指向该数组里最近定义的表元素
color = "红色"
shape = "圆形"

[[e]]
x = "e[1].x"
y = "e[1].y"


[database]
server = "192.168.1.1"
connection_max = 5000
ports = [ ["gamma", "delta"], [1, 2] ]
enabled = true
dob = 1979-05-27T07:32:00-08:00

# RFC3389: https://tools.ietf.org/html/rfc3339
offset_datetime1 = 1979-05-27T07:32:00Z
offset_datetime2 = 1979-05-27T00:32:00-07:00
offset_datetime3 = 1979-05-27 07:32:00Z
local_datetime1 = 1979-05-27T07:32:00
local_datetime2 = 1979-05-27
local_time1 = 07:32:00

```



## .ini

```ini
[a]
a = str  ; 注释
```



## 格式

大小写敏感
定义层级结构：使用显式键名链，放弃缩进



## 变量类型

变量类型：行内表、数组、字符串、布尔、数字、日期























