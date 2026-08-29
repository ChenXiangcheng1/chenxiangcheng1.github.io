# mise

[github discussions](https://github.com/jdx/mise/discussions)  
[官网](https://mise.en.dev/): TODO: 他们官方文档还是很全的  

## 配置

安装目录: ~/.local/share/mise/installs
全局配置: ~/.config/mise/config.toml

```config.toml
# 项目工具tools 环境变量env 任务tasks hooks(例如项目enter时)
[tools]
node = "24.18.0"

[settings]
# Read the pnpm version from package.json  # mise config ls 能识别到
idiomatic_version_file_enable_tools = ['pnpm']

[tasks.task_name]  # mise run <task_name>
description = "test the project"
run = """
export FOO=bar
echo $FOO
"""

[tasks.pnpm-install]
description = 'Installs dependencies with pnpm'
run = 'pnpm install'
sources = ['package.json', 'pnpm-lock.yaml', 'mise.toml']
outputs = ['node_modules/.pnpm/lock.yaml']

[tasks.dev]
description = 'Calls your dev script in `package.json`'
run = 'node --run dev'
depends = ['pnpm-install']
```

```mise.toml
[env]
_.path = ['{{config_root}}/node_modules/.bin']
```


## 命令

CLI的包管理工具+makefile

```bash
mise --help
mise [OPTIONS] [TASK] [COMMAND]
mise activate [shell_type] # 在当前会话初始化mise, 直接添加到bashrc
mise cache prune  # 清理30天未访问的缓存
# mise completion [shell]  # 添加到~/.local/share/bash-completion/completions/mise, 但我不推荐 bash-completion 服务端补全, 因为我远程ssh开发下termius也有完整补全
mise config ls  # 列出配置文件, 常用看tool从哪个配置加载
mise doctor  # 打印一堆信息和问题
mise exec [TOOL@VERSION]  # 使用mise工具集, 运行临时命令, 执行但不激活
# mise exec node@26 -- node -v  # "--" 分割运行时参数和要传递给子进程的命令
mise fmt  # 格式化mise.toml
mise generate [command]  # 生成辅助配置文件 git-pre-commit config
mise edit [mise_toml_path]  # 交互式修改mise配置  # 很好用
mise install node@24.18.0  # 安装到~/.local/share/mise/installs 但不激活
mise install  # 根据mise.toml安装所需工具
mise uninstall [INSTALLED_TOOL@VERSION]  # 移除已安装的工具, 不修改mise.toml
mise latest [TOOL@VERSION]  # 获取latest版本号
# mise lock  # 更新lockfile, mise.toml决定装什么, lockfile决定版本号
mise ls  # 获取已知工具的状态, --current指定mise.toml
mise ls --current  # 常用
mise ls-remote node@24  # mise ls-remote github:BurntSushi/ripgrep  # 列出可安装的tool版本
# mise mcp  # 运行mcp server, 暴露mise的功能给agent
# mise oci  # 生成image, 常配合 mise generate devcontainer使用, 用于统一开发环境
mise outdated  # 列出过时工具, 常配合 mise upgrade 使用
# 不推荐使用 mise 传统 tool plugins 
# 推荐使用aqua Backend Plugin(包管理器), 其次github Backend Plugin
# mise plugins ls-remote
# mise plugins install 
# mise plugins ls
# mise plugins update
mise registry | grep ripgrep  # 列出可安装的工具
mise run <task_name> arg1 arg2 ::: <tn2> ::: <tn3> # 运行task(定义于mise.toml 或 .mise/tasks/task_name)  # Makefile(每一行shell隔离), mise run(每个任务shell隔离) (shell隔离为了操作可复现)
mise search [tool_name]  # 在registry内搜索tool, 常配合mise info使用
# mise set NODE_ENV=development  # 不推荐mise.toml管理环境变量, 小心泄漏secrets, 我还是喜欢更通用的.env.example和.env
mise settings  # 显示~/.config/mise/config.toml的内容
mise shell [TOOL@VERSION]  # 设置当前会话的工具版本
mise tasks ls  # (定义于mise.toml 或 .mise/tasks/task_name)
mise token github  # 显示mise使用的token
mise tool <tool_name>  # 获取工具信息
mise upgrade [INSTALLED_TOOL@VERSION]  # 升级过时的工具到latest版本且更新mise.lock  # --bump无视mise.toml约束且更新mise.toml  # -i 交互式
mise use  # interactive selector
mise use [TOOL@VERSION]  # 激活: 添加工具到mise.toml  # --global --path --env(local staging)  # mise.local.toml是本地文件非项目文件, 不应提交到git  # --pin激活精确版本
mise unuse <INSTALLED_TOOL@VERSION>  # 从mise.toml移除已安装的工具  # --global --path --env(local staging)
mise version
# mise watch [TASK] [ARGS]  # 使用 watchexec tool, 用于持续build等任务
mise where <TOOL@VERSION>  # 显示工具的安装目录
mise which <TOOL@VERSION>  # 显示工具可以执行bin的路径
```
