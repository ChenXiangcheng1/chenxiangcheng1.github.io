# Mamba

## Mamba1.5.7

[mamba官方文档](https://mamba.readthedocs.io/en/latest/advanced_usage/more_concepts.html) [micromamba文档](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html)	|	[github](https://github.com/mamba-org/mamba)	|	[conda-forge通道官方文档](https://conda-forge.org/docs/)



### 命令

```bash
# mamba init 在 git bash 中无效，可以使用 source activate！！！
source activate mamba_python312
```



```bash
doskey /macros
conda="D:\Applications\Scoop\apps\mambaforge\current\condabin\conda.bat" $*
doskey conda=  # 当前会话有效
```





```bash
conda env export > environment.yml
conda env create -f xxx.yaml
conda env update -f xxx.yaml
```

| mamba [command] | 释义                    | 参数 |
| --------------- | ----------------------- | ---- |
| info            |                         | -e   |
| init            | 令shell支持activate命令 |      |

| 环境管理mamba [command]  | 释义 | 参数                               |
| ------------------------ | ---- | ---------------------------------- |
| create [package=version] |      | -n 指定环境名<br />-p 指定环境路径 |
| activate [env]           |      |                                    |
| deactivate               |      |                                    |

| 包管理mamba [command]               | 释义                                                         | 参数                                                 |
| ----------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| search [package]                    |                                                              |                                                      |
| repoquery {whoneeds,depends,search} | [conda-forge仓库(在线搜索)](https://anaconda.org/search?q=python) |                                                      |
| install                             |                                                              | -n 指定环境<br />-c 指定源<br />--upgrade 存在则升级 |
| remove                              | 从conda环境中删指定包和被依赖包                              | -n 指定环境<br />--all 删除环境自身                  |
| list                                |                                                              |                                                      |

​    clean             Remove unused packages and caches.
​    compare           Compare packages between conda environments.
​    config            Modify configuration values in .condarc.

​    doctor            Display a health report for your environment.
​    install           Install a list of packages into a specified conda environment.
​    list              List installed packages in a conda environment.
​    notices           Retrieve latest channel notifications.
​    package           Create low-level conda packages. (EXPERIMENTAL)

​    rename            Rename an existing environment.
​    run               Run an executable in a conda environment.
​    search            Search for packages and display associated information using the MatchSpec format.
​    update (upgrade)  Update conda packages to the latest compatible version.



### 配置文件

~~/.condarc~~

`~/.mambarc`

```ini
show_channel_urls: true
ssl_verify: true
auto_activate_base: false
channels:
  - defaults
```

[镜像配置](https://blog.csdn.net/WannaSeaU/article/details/88427010)	|	[清华镜像源](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/)



## Conda24.1.2

**conda**是一个包管理器，旨在构建和管理任何语言和任何类型的软件。 
Anaconda则是一个打包的集合，里面预装好了conda、某个版本的python、众多packages、科学计算工具等等，就是把很多常用的不常用的库都给你装好了。
miniconda，默认channel为anaconda.org(官方驱动)
**[Miniforge3](https://github.com/conda-forge/miniforge)**，默认channel为conda-forge(社区驱动)，合并了mambaforge(mamba是conda的重新实现比conda快)

[Conda官方文档](https://conda.io/projects/conda/en/latest/commands.html#)	|	[anaconda官方文档](https://docs.anaconda.com/free/anacondaorg/user-guide/)



### 命令

| 命令行                                                       | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| conda --help                                                 |                                                              |
| conda info                                                   | 查看conda的配置信息                                          |
| **config**                                                   |                                                              |
| conda config --show channels                                 | 查看源                                                       |
| conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/ | 设置科大的镜像，如果还是用.condarc文件改好                   |
| conda config --set show_channel_urls yes                     | 配置显示channel-url，会生成.condarc文件                      |
| conda config --remove-key channels                           | 恢复Anaconda的源为默认                                       |
|                                                              |                                                              |
| conda create --name python36 python=3.6                      | 创建一个名为python36的环境，指定Python版本是3.6（不用管是3.6.x，conda会为我们自动寻找3.6.x中的最新版本） |
| conda create --name python36 python=3.6 anaconda             | 安装很多科学包，和基础环境无关                               |
| conda install anaconda                                       | 在当前环境下安装anaconda包集合                               |
|                                                              |                                                              |
| conda remove --name python3 --all                            | 删除一个已有的环境                                           |
|                                                              |                                                              |
| conda env list或者conda info -e                              | 查看当前存在哪些虚拟环境，当前被激活的环境会显示有一个星号   |
|                                                              |                                                              |
| conda list                                                   | 查看conda科学包安装了哪些包                                  |
| conda list -n python34 [package_name]                        | 查看某个指定环境的已安装包                                   |
|                                                              |                                                              |
| conda search numpy                                           | 查找package信息，往往在下载包之前使用，看有没有网络等问题    |
| conda install -n evn_name -c  --upgrate [package_name]=[version] | 对虚拟环境中安装额外的包，**需要指定环境**，如果不用-n指定环境名称，则被安装在当前活跃环境，还可以指定版本，-c指定源（在使用镜像源更快时可去除-c及参数），--upgrade 如果存在该包，则升级该包 |
|                                                              |                                                              |
| conda remove --name env_name package_name                    | 删除环境中的某个包                                           |
|                                                              |                                                              |
| conda env export > environment.yaml                          | 将当前环境下的 package 信息存入名为 environment 的 YAML 文件中 |
| conda env create -f environment.yaml                         | 根据 YAML 文件来创建运行环境                                 |



### 不要随便更新Conda

| 命令                                   | 释义                                               |
| -------------------------------------- | -------------------------------------------------- |
| conda update -n base -c defaults conda | 更新                                               |
| conda update conda                     | 表示更新conda                                      |
| conda update --all                     | 更新所有模块，可能出现部分降级的问题，尽量避免使用 |

