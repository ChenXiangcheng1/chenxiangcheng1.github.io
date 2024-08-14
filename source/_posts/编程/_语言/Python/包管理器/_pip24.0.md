# pip24.0

[PyPI(Python包索引，托管第三方库的官方平台)](pypi.org)
其他平台：anaconda、conda-forge
截至2021.9 PyPI包含超300,000，conda-forge包含30,000



## 命令

| pip <command> <Subcommands> [options] | 释义                   | 选项                                                         |
| ------------------------------------- | ---------------------- | ------------------------------------------------------------ |
| 通用选项                              |                        | --version 版本并推出                                         |
| 通用选项                              |                        | --help显示帮助                                               |
| pip list                              | 输出已安装的包和版本   |                                                              |
| pip freeze                            | 输出已安装的包和位置   |                                                              |
| pip config list                       | 列出活动配置           |                                                              |
| pip search [packagename]              | 搜索Python包索引       |                                                              |
| pip install [packagename]             | 安装包                 | -i https://pypi.tuna.tsinghua.edu.cn/simple 指定仓库或本地目录(临时换源) |
| pip show [packagename]                | 显示关于已安装包的显示 |                                                              |



## 配置文件

pip配置文件位置：`pip config list -v`

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple  # 不推荐，建议临时换源
[install]
trusted-host=mirrors.aliyun.com  # 不推荐，建议临时换源
```



