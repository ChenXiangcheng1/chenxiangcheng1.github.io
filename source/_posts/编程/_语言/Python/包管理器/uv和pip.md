

# python包管理器

截至2021.9 PyPI包含超300,000，conda-forge包含30,000，这些源都有预编译的包

| 包管理器 | 能否管理非Python依赖            | 生态/软件源  | feature                              | 问题                                  |
| -------- | ------------------------------- | ------------ | ------------------------------------ | ------------------------------------- |
| uv       | 依赖系统底层库 只管理python依赖 | 官方生态PYPI | 还支持构建和发布项目、Scripts、tools |                                       |
| conda    | 能管理并隔离非Python依赖        | 第三方生态   |                                      | 重，命令行加载慢                      |
| pixi     | 能管理非Python依赖              |              |                                      | 部分依赖在PYPI部分在forge解决冲突麻烦 |
|          |                                 |              |                                      |                                       |



## pip24.0

### 命令

```bash
pip --help

Usage:
  pip <command> [options]

Commands:
  install                     Install packages.
  download                    Download packages.
  uninstall                   Uninstall packages.
  freeze                      Output installed packages in requirements format.
  inspect                     Inspect the python environment.
  list                        List installed packages.
  show                        Show information about installed packages.
  check                       Verify installed packages have compatible dependencies.
  config                      Manage local and global configuration.
  search                      Search PyPI for packages.
  cache                       Inspect and manage pip's wheel cache.
  index                       Inspect information available from package indexes.
  wheel                       Build wheels from your requirements.
  hash                        Compute hashes of package archives.
  completion                  A helper command used for command completion.
  debug                       Show information useful for debugging.
  help                        Show help for commands.

General Options:
  -h, --help                  Show help.
  --debug                     Let unhandled exceptions propagate outside the main subroutine, instead of
                              logging them to stderr.
  --isolated                  Run pip in an isolated mode, ignoring environment variables and user
                              configuration.
  --require-virtualenv        Allow pip to only run in a virtual environment; exit with an error otherwise.
  --python <python>           Run pip with the specified Python interpreter.
  -v, --verbose               Give more output. Option is additive, and can be used up to 3 times.
  -V, --version               Show version and exit.
  -q, --quiet                 Give less output. Option is additive, and can be used up to 3 times
                              (corresponding to WARNING, ERROR, and CRITICAL logging levels).
  --log <path>                Path to a verbose appending log.
  --no-input                  Disable prompting for input.
  --keyring-provider <keyring_provider>
                              Enable the credential lookup via the keyring library if user input is allowed.
                              Specify which mechanism to use [disabled, import, subprocess]. (default:
                              disabled)
  --proxy <proxy>             Specify a proxy in the form scheme://[user:passwd@]proxy.server:port.
  --retries <retries>         Maximum number of retries each connection should attempt (default 5 times).
  --timeout <sec>             Set the socket timeout (default 15 seconds).
  --exists-action <action>    Default action when a path already exists: (s)witch, (i)gnore, (w)ipe,
                              (b)ackup, (a)bort.
  --trusted-host <hostname>   Mark this host or host:port pair as trusted, even though it does not have
                              valid or any HTTPS.
  --cert <path>               Path to PEM-encoded CA certificate bundle. If provided, overrides the default.
                              See 'SSL Certificate Verification' in pip documentation for more information.
  --client-cert <path>        Path to SSL client certificate, a single file containing the private key and
                              the certificate in PEM format.
  --cache-dir <dir>           Store the cache data in <dir>.
  --no-cache-dir              Disable the cache.
  --disable-pip-version-check
                              Don't periodically check PyPI to determine whether a new version of pip is
                              available for download. Implied with --no-index.
  --no-color                  Suppress colored output.
  --no-python-version-warning
                              Silence deprecation warnings for upcoming unsupported Pythons.
  --use-feature <feature>     Enable new functionality, that may be backward incompatible.
  --use-deprecated <feature>  Enable deprecated functionality, that will be removed in the future.
```



| pip <command> <Subcommands> [options] | 释义         | 选项                                                         | 释义                                                         |
| ------------------------------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| pip install  <package \| path>        |              |                                                              |                                                              |
|                                       |              | -r <requirements.txt>                                        | 从给定的 requirements file下载包                             |
|                                       |              | -e<br />pip install -e .                                     | 可编辑模式<br />site-packages是python软件包的安装目录，一般来说其中会存放一份代码副本，而-e可编辑模式会存放一份指向项目源代码的软连接(符号链接) |
|                                       |              | --index-url https://pypi.tuna.tsinghua.edu.cn/simple<br />默认 https://pypi.org/simple |                                                              |
| pip freeze > requirements.txt         |              |                                                              | 输出已安装的包和位置，可重定向为requirements文件             |
| pip list                              |              |                                                              | 输出已安装的包和版本                                         |
|                                       |              | -o                                                           | 显示Outdated Packages过期包                                  |
|                                       |              | -u                                                           | 显示Uptodate Packages已经是最新了                            |
|                                       |              | -e                                                           | 显示可编辑项目包                                             |
| pip show <package>                    |              |                                                              | 显示installed packages的信息                                 |
| pip check                             |              |                                                              | 验证已安装包具有兼容的依赖项                                 |
| pip config                            | 列出活动配置 |                                                              |                                                              |
|                                       |              | list -v                                                      | 显示 active configuration 文件位置                           |
|                                       |              | get <command.option><br />pip config get index-url           |                                                              |
|                                       |              | set <command.option value>                                   |                                                              |
|                                       |              | unset <command.option>                                       | 重置                                                         |
| pip search                            |              |                                                              | 不再支持                                                     |
|                                       |              |                                                              |                                                              |



### 配置

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple  # 不推荐，建议临时换源
[install]
trusted-host=mirrors.aliyun.com  # 不推荐，建议临时换源
```



## uv

[github](https://github.com/astral-sh/uv)	|	[docs](https://docs.astral.sh/uv/guides/)



```bash
# 安装后可以配置Shell autocompletion
echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc  # bash
echo 'uv generate-shell-completion fish | source' > ~/.config/fish/completions/uv.fish  # fish
```



### 命令

```bash
$ uv help
An extremely fast Python package manager.

Usage: uv [OPTIONS] <COMMAND>

Commands:
  auth                       Manage authentication
  run                        Run a command or script
  init                       Create a new project
  add                        Add dependencies to the project
  remove                     Remove dependencies from the project
  version                    Read or update the project's version
  sync                       Update the project's environment
  lock                       Update the project's lockfile
  export                     Export the project's lockfile to an alternate format
  tree                       Display the project's dependency tree
  format                     Format Python code in the project
  tool                       Run and install commands provided by Python packages
  python                     Manage Python versions and installations
  pip                        Manage Python packages with a pip-compatible interface
  venv                       Create a virtual environment
  build                      Build Python packages into source distributions and wheels
  publish                    Upload distributions to an index
  cache                      Manage uv's cache
  self                       Manage the uv executable
  generate-shell-completion  Generate shell completion
  help                       Display documentation for a command

Cache options:
  -n, --no-cache               Avoid reading from or writing to the cache, instead using a temporary directory for the duration of the operation [env: UV_NO_CACHE=]
      --cache-dir <CACHE_DIR>  Path to the cache directory [env: UV_CACHE_DIR=]

Python options:
      --managed-python       Require use of uv-managed Python versions [env: UV_MANAGED_PYTHON=]
      --no-managed-python    Disable use of uv-managed Python versions [env: UV_NO_MANAGED_PYTHON=]
      --no-python-downloads  Disable automatic downloads of Python. [env: "UV_PYTHON_DOWNLOADS=never"]

Global options:
  -q, --quiet...                                   Use quiet output
  -v, --verbose...                                 Use verbose output
      --color <COLOR_CHOICE>                       Control the use of color in output [possible values: auto, always, never]
      --native-tls                                 Whether to load TLS certificates from the platform's native store [env: UV_NATIVE_TLS=]
      --offline                                    Disable network access [env: UV_OFFLINE=]
      --allow-insecure-host <ALLOW_INSECURE_HOST>  Allow insecure connections to a host [env: UV_INSECURE_HOST=]
      --no-progress                                Hide all progress outputs [env: UV_NO_PROGRESS=]
      --directory <DIRECTORY>                      Change to the given directory prior to running the command [env: UV_WORKING_DIR=]
      --project <PROJECT>                          Discover a project in the given directory [env: UV_PROJECT=]
      --config-file <CONFIG_FILE>                  The path to a `uv.toml` file to use for configuration [env: UV_CONFIG_FILE=]
      --no-config                                  Avoid discovering configuration files (`pyproject.toml`, `uv.toml`) [env: UV_NO_CONFIG=]
  -h, --help                                       Display the concise help for this command
  -V, --version                                    Display the uv version

Use `uv help <command>` for more information on a specific command.
```

| uv [OPTIONS] <COMMAND>        | 子命令 | 选项                                                         | 释义                                                         |
| ----------------------------- | ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Global options                |        | -vv                                                          | 详细输出                                                     |
| **uv run <subcmd> <options>** |        | uv run file.py == uv run python file.py<br />uv run ruff check | 在项目虚拟环境中运行该命令                                   |
| **uv init <project>**         |        |                                                              | 初始化uv项目<br />pyproject.toml<br />~~uv.lock锁文件~~<br />~~requirements.txt需求文件~~<br />.python-version |
|                               |        | --package                                                    | 普通项目，可分发包<br />`main.py`                            |
|                               |        | --app                                                        | 命令行应用，完整应用程序                                     |
|                               |        | --lib                                                        | 库<br />`__init__.py`                                        |
|                               |        | --script                                                     |                                                              |
| **uv add <packages>**         |        |                                                              | 添加到项目的虚拟环境中，会修改pyproject.toml并sync同步.venv<br />source .venv/bin/active 后可调用 |
|                               |        | -r <requirements.txt>                                        | 从给定的 requirements file下载包                             |
|                               | -      | --dev                                                        | 添加需求requirements到开发依赖组<br />**推荐 ruff、ipykernel** |
|                               |        | --default-index https://pypi.tuna.tsinghua.edu.cn/simple<br />默认 https://pypi.org/simple<br />https://mirrors.aliyun.com/pypi/simple/ |                                                              |
|                               |        | --index-url                                                  | Deprecated弃用                                               |
| uv remove <packages>          |        |                                                              | 从项目中移除依赖                                             |
|                               |        | --dev                                                        | 添加需求requirements到开发依赖组                             |
| uv sync                       |        |                                                              | 同步项目的依赖项和虚拟环境.venv<br />只有在手动、git修改了pyproject.toml后才需要手动调用`uv lock`、再调用`uv sync` |
|                               |        | --no-dev                                                     |                                                              |
|                               |        | --only-dev                                                   |                                                              |
| uv lock                       |        |                                                              | 为项目依赖创建lockfile<br />更新依赖版本生成lockfile，但不动.venv，用于检测包冲突 |
|                               |        | --check                                                      | up-to-date                                                   |
|                               |        | --check-exists                                               |                                                              |
| **uv tree**                   |        |                                                              | 查看项目的依赖关系树                                         |
|                               |        | --depth <DEPTH>                                              |                                                              |
|                               |        | --package <PACKAGE>                                          |                                                              |
| **uv tool**                   |        |                                                              | 管理发布到PYPI的工具                                         |
|                               |        | run == **uvx**<br />--env-file <ENV_FILE>                    | 在临时环境中运行工具                                         |
|                               |        | install <PACKAGE>                                            | 安装到系统PATH<br />**推荐 ruff**                            |
|                               |        | upgrade                                                      |                                                              |
|                               |        | list<br />--show-paths                                       |                                                              |
|                               |        | uninstall <PACKAGE>                                          |                                                              |
|                               |        | update-shell                                                 | 更新 shell 以包含工具可执行文件 `export PATH="/home/nemesis/.local/bin:$PATH"` |
|                               |        | dir<br />--bin 显示可执行文件目录                            | 显示uv tool目录                                              |
| uv python                     |        |                                                              |                                                              |
|                               |        | list<br />--only-installed                                   |                                                              |
|                               |        | install <version><br />-i <INSTALL_DIR>                      |                                                              |
|                               |        | find <version><br />--show-version                           |                                                              |
|                               |        | **pin <version>**<br />--rm                                  | 持久化到.python-version                                      |
|                               |        | dir                                                          | ~/.local/share/uv/python                                     |
|                               |        | uninstall                                                    |                                                              |
| uv pip                        |        |                                                              | 管理Python包，兼容pip命令                                    |
|                               |        | sync <SRC_FILE><br />uv pip sync docs/requirements.txt       | 使用lockfile(requirements.txt或pylock.toml)同步虚拟环境，安装所需的依赖，更推荐uv sync同步pyproject.toml 和 uv.lock |
|                               |        | install                                                      | 更推荐uv add                                                 |
|                               |        | uninstall                                                    |                                                              |
|                               |        | freeze                                                       | 列出虚拟环境中已安装包(requirements格式)                     |
|                               |        | list                                                         | 列出虚拟环境中已安装包(表格格式)                             |
|                               |        | show <package>                                               |                                                              |
|                               |        | check                                                        | 检查当前环境中的包是否兼容                                   |
|                               |        | tree                                                         | 查看环境的依赖关系树                                         |
|                               |        | compile<br />uv pip compile pyproject.toml requirements-dev.in -o requirements-dev.txt | 将依赖项编译到 lockfile 中                                   |
| uv venv <path>                |        |                                                              | 创建虚拟环境.venv<br />`source <.venv>/bin/activate` 激活虚拟环境 |
|                               |        | --python <version>                                           | 在site-packages中指定python版本                              |
| uv build <SRC>                |        |                                                              | 编译python包 distribution archives                           |
|                               |        | --out-dir <OUT_DIR>                                          |                                                              |
|                               |        | --sdist                                                      | .tar.gz源码包                                                |
|                               |        | --wheel                                                      | .whl二进制包                                                 |
| uv publish <FILE\|glob表达式> |        |                                                              | 发布distributions分发包到package index包存储库               |
| uv cache                      |        |                                                              |                                                              |
|                               |        | clean                                                        | 清理所有                                                     |
|                               |        | prune                                                        | 移除outdated过时的缓存条目                                   |
|                               |        | dir                                                          | ~/.cache/uv                                                  |
|                               |        | size                                                         |                                                              |
| uv self                       |        |                                                              |                                                              |
|                               |        | ~~update~~                                                   | pacman -Syu uv                                               |
| uv help                       |        |                                                              |                                                              |
|                               |        |                                                              |                                                              |

```bash
# 原始方式
python -m venv .venv
source .venv/bin/activate
edit pyproject.toml
pip install -e .
```



### 项目

```toml
# pyproject.toml
[project]
name = "uv-demo"
version = "0.1.0"
description = "xxx"
readme = "README.md"
requires-python = ">=3.13"
dependencies = []

[tool.uv]
index-url = "https://pypi.tuna.tsinghua.edu.cn/simple"
```

```lock
# uv.lock
version = 1
revision = 2
requires-python = ">=3.13"

[[package]]
name = "uv-demo"
version = "0.1.0"
source = { virtual = "." }
```



## pixi

[pixi github](https://github.com/prefix-dev/pixi)



## mise

mise 多语言版本管理器

