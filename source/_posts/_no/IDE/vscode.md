# vscode

[Manual](https://code.visualstudio.com/docs)

缺点：不如Pycharm
1没有Override列表，要逐个重写
2不能部分格式化

优点：节约内存、自定义工具链



## 配置

对应 defaultSettings.json 来设置 Perferences: User Settings (JSON)



## 插件

[插件市场](https://marketplace.visualstudio.com/vscode)



### Vim

https://github.com/VSCodeVim/Vim

```json
// vim插件配置 Start 
"vim.incsearch": true,  // 增量搜索
"vim.useSystemClipboard": true,  // vim使用系统剪切板
"vim.useCtrlKeys": true,  // 启用vscode Ctrl键，而不是vim键
"vim.hlsearch": true,  // 搜索时高亮所有匹配项
"vim.insertModeKeyBindings": [  // 插入模式键位映射
    {
        "before": ["j", "j"],  // 意思是插入模式快速按两下jj会回到正常模式
        "after": ["<Esc>"]
    }
],
"vim.normalModeKeyBindingsNonRecursive": [  // 正常模式非递归嵌套键位映射，不会再触发其他键位绑定
    {
        "before": ["<leader>", "d"],
        "after": ["d", "d"]
    },
    {
        "before": ["<C-n>"],
        "commands": [":nohl"]  // 关闭搜索高亮
    },
    {
        "before": ["K"],
        "commands": ["lineBreakInsert"],  // 光标前插入换行符
        "silent": true
    }
],
"vim.leader": "<space>",
"vim.handleKeys": {  // 禁用一些Vim快捷键
    "<C-s>": false,
    "<C-z>": false,
    "<C-a>": false,
    "<C-f>": false,
},
"vim.smartRelativeLine": true,  // 插入模式显示绝对行号，正常模式显示相对行号

// 各种插件
"vim.easymotion": true,  // 启用easymotion插件，可以快速跳转到任意位置

"vim.statusBarColorControl": true,  // 启用vim-airline插件，根据对应模式改状态栏颜色
"vim.statusBarColors.normal": ["#8FBCBB", "#434C5E"],
"vim.statusBarColors.insert": "#BF616A",
"vim.statusBarColors.visual": "#B48EAD",
"vim.statusBarColors.visualline": "#B48EAD",
"vim.statusBarColors.visualblock": "#A3BE8C",
"vim.statusBarColors.replace": "#D08770",
"vim.statusBarColors.commandlineinprogress": "#007ACC",
"vim.statusBarColors.searchinprogressmode": "#007ACC",
"vim.statusBarColors.easymotionmode": "#007ACC",
"vim.statusBarColors.easymotioninputmode": "#007ACC",
"vim.statusBarColors.surroundinputmode": "#007ACC",

"vim.autoSwitchInputMethod.enable": false,  // 自动切换输入法，需要安装 im-select-imm。不过我没用
// vim插件配置 End
```



### C++

GUN的子项目
MinGW(Minimalist GNU For Windows)：是一直到Windows的GNU工具集合，停止更新GCC版本停滞
MinGW-w64：能编译生成64位可执行程序 `scoop install main/mingw`
	GCC13.2.0(GNU C Compiler / GNU Compiler Collection)：编译C/C++
	G++：编译C++

**LLVM**(Low Level Virtual Machine)的子项目(推荐)
Clang前端：兼容GCC

MSVC(Microsoft Visual C/C++ Compiler)

|          | MinGW-w64 | Clang |
| -------- | --------- | ----- |
| 构建工具 | Make      | CMake |
| 调试     | GDB       | LLDB  |



MAKE：写makefile
CMAKE(推荐)：写cmakefile.txt

LS: `.clangd` [官网](https://clangd.llvm.org/)
静态代码分析: clang-tidy 通过`.clangd` 配置
格式化: `.clang-format`
Debug: [codelldb](https://github.com/vadimcn/codelldb)、cppdbg
build工具: [cmake-tools](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/README.md)
build system(Generator): ninja

```bash
clang-format --style=LLVM -i main.cpp
clang-format --style=LLVM --dump-config > .clang-format
```



TODO:

clangd配置
clangd插件配置
最佳实践



#### LSP

配置：.clangd 或 使用clangd插件配置



### Python

TODO：看vscode文档#Python  (quick start看完了)

TODO：Python Debug插件、pytest



#### LSP

Language Server Protocol：将静态分析服务与IDE解耦

##### Pylance

[vscode插件](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)

Python Languase Server ，基于**类型检查器 pyright**

F2 重构



##### Ruff(推荐)

[vscode 插件](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff)

核心是ruff-lsp，代替 flake8、isort、bandit



**配置：**

[官方文档](https://docs.astral.sh/ruff/)

TODO：读配置

```toml
# Exclude a variety of commonly ignored directories.
exclude = [
    ".bzr",
    ".direnv",
    ".eggs",
    ".git",
    ".git-rewrite",
    ".hg",
    ".ipynb_checkpoints",
    ".mypy_cache",
    ".nox",
    ".pants.d",
    ".pyenv",
    ".pytest_cache",
    ".pytype",
    ".ruff_cache",
    ".svn",
    ".tox",
    ".venv",
    ".vscode",
    "__pypackages__",
    "_build",
    "buck-out",
    "build",
    "dist",
    "node_modules",
    "site-packages",
    "venv",
]

line-length = 88
indent-width = 4

# 假定 Python 3.11
target-version = "py311"

[lint]
# Enable Pyflakes (`F`) and a subset of the pycodestyle (`E`)  codes by default.
# Unlike Flake8, Ruff doesn't enable pycodestyle warnings (`W`) or
# McCabe complexity (`C901`) by default.
select = [
	"E4", "E7", "E9", "F",  # 默认
	"I",
	"UP",
	"W",
	
]
ignore = []  # lint时忽略

# Allow fix for all enabled rules (when `--fix`) is provided.
fixable = ["ALL"]
unfixable = ["F401"]  # 格式化时忽略F401

# Allow unused variables when underscore-prefixed.
dummy-variable-rgx = "^(_+|(_+[a-zA-Z0-9_]*[a-zA-Z0-9]+?))$"

[format]
quote-style = "single"

# Like Black, indent with spaces, rather than tabs.
indent-style = "space"

# Like Black, respect magic trailing commas.
skip-magic-trailing-comma = false

# Like Black, automatically detect the appropriate line ending.
line-ending = "auto"

# Enable auto-formatting of code examples in docstrings. Markdown,
# reStructuredText code/literal blocks and doctests are all supported.
#
# This is currently disabled by default, but it is planned for this
# to be opt-out in the future.
docstring-code-format = false

# Set the line length limit used when formatting code snippets in
# docstrings.
#
# This only has an effect when the `docstring-code-format` setting is
# enabled.
docstring-code-line-length = "dynamic"
```

| --extend-select<br />[lint] select | 命令行库                     | 简介                           |
| ---------------------------------- | ---------------------------- | ------------------------------ |
| F                                  | pyflakes                     | 提供了一些比较基础的问题检查。 |
| I                                  | isort                        | 对 import 语句进行排序。       |
| UP                                 | pyupgrade                    | 提示新版本的 Python 语法。     |
|                                    |                              |                                |
| E,W                                | pycodestyle errors, warnings | PEP8 检查。                    |
| N                                  | pep8-naming                  | PEP8 命名规范检查。            |
| PL                                 | Pylint                       | 知名静态代码检查器。           |
| PERF                               | pyperf                       | 检测一些性能问题。             |
| RUF                                | Ruff                         | Ruff 社区自己实现的一些规则。  |

#### linter

##### flake8



##### pyline



#### Python格式化

##### autopep8

https://github.com/hhatto/autopep8



##### yapf

https://github.com/google/yapf

支持多种格式化的风格



##### black

https://github.com/psf/black

统一的格式化标准，约定大于配置



#### 类型检查

##### mypy

[个人 github](https://github.com/matangover/mypy-vscode)

据说pylance比mypy快



## 开发容器

我的评价：感觉不够灵活，例如想使用conda等麻烦，做不到无痛隔绝环境

[dev container reference](https://containers.dev/implementors/json_reference/)

```.devcontainer/devcontainer.json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/python
{
	"name": "Python 3",
	// Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
	"image": "mcr.microsoft.com/devcontainers/python:0-3.11",

	// Features to add to the dev container. More info: https://containers.dev/features.
	// "features": {},

	// Configure tool-specific properties.
	"customizations": {
		// Configure properties specific to VS Code.
		"vscode": {
			"settings": {},
			"extensions": [
				"streetsidesoftware.code-spell-checker"
			]
		}
	},
	
	// Use 'forwardPorts' to make a list of ports inside the container available locally.
	// "forwardPorts": [9000],

	// Use 'portsAttributes' to set default properties for specific forwarded ports. 
	// More info: https://containers.dev/implementors/json_reference/#port-attributes
	"portsAttributes": {
		"9000": {
			"label": "Hello Remote World",
			"onAutoForward": "notify"
		}
	},

	// Use 'postCreateCommand' to run commands after the container is created.
	"postCreateCommand": "pip3 install -r requirements.txt"

	// Uncomment to connect as root instead. More info: https://aka.ms/dev-containers-non-root.
	// "remoteUser": "root"
}

```

应该需要配置一个 host.docker.internal:7890 的代理
有空再 find / --name "git"，看以下git配置是如何从宿主机传递的



# 未整理

https://zhuanlan.zhihu.com/p/138579730





## 配置文件

C:\Users\qq109\AppData\Roaming\Code\User\setting.json

包含插件的配置

### 键盘快捷键方式

C:\Users\qq109\AppData\Roaming\Code\User\keybindings.json

[键盘快捷方式默认参考](https://blog.csdn.net/shanghongshen/article/details/61415374)

可视化打开方式：ctrl+k ctrl+s
json打开方式：ctrl+shift+p keyboard.json

目前我的配置：

```json
// 将键绑定放在此文件中以覆盖默认值auto[]
// 下面的配置权重更高，会覆盖上面的配置
[
    {
        "key": "enter", 
        "command": "acceptSelectedSuggestion",  // 选择建议
        "when": "suggestWidgetVisible && textInputFocus"
    },
    // {
    //     "key": "space", 
    //     "command": "acceptSelectedSuggestion",  // 选择建议
    //     "when": "suggestWidgetVisible && textInputFocus"
    // },
    {
        "key": "tab",
        "command": "selectNextSuggestion",  // 不用方向键选择建议
        "when": "editorTextFocus && suggestWidgetMultipleSuggestions && suggestWidgetVisible"
    },
        {
        "key": "shift+tab",
        "command": "selectPrevSuggestion",  // 不用方向键选择建议
        "when": "editorTextFocus && suggestWidgetMultipleSuggestions && suggestWidgetVisible"
    },
    {
        "key": "ctrl+p",
        "command": "editor.action.triggerParameterHints",  // 显示参数建议
        "when": "editorHasSignatureHelpProvider && editorTextFocus"
    },
    {
        "key": "ctrl+shift+enter",
        "command": "-editor.action.insertLineBefore",  // 不在上一行插入
        "when": "editorTextFocus && !editorReadonly"
    },
    {
        "key": "ctrl+enter",
        "command": "-editor.action.insertLineAfter",  // 不在下面插入一行
        "when": "editorTextFocus && !editorReadonly"
    },
    {
        "key": "shift+enter",
        "command": "editor.action.insertLineAfter",  // 在下面插入一行
        "when": "editorTextFocus && !editorReadonly"
    },
    {
        "key": "ctrl+shift+enter",
        "command": "cursorEnd",  // 光标移到行尾
        "when": "textInputFocus"
    },
    {
        "key": "alt+enter",
        "command": "editor.action.triggerSuggest",  // 触发建议
        "when": "editorHasCompletionItemProvider && textInputFocus && !editorReadonly"
    },
]
```

## 以下是我还没整理的

## 切换终端

网上的都是老的方式直接选择终端，现在是终端-启动配置文件-选择默认配置文件

还需要在 settings.json 中改 C++ 的 cmd 编译指令



## 快捷键

| 快捷键                          | 释义             |
| ------------------------------- | ---------------- |
| ctrl k ctrl s                   | 键盘快捷方式设置 |
| ctrl p                          | 快速打开文件     |
| ctrl shift p                    | 快捷命令         |
|                                 |                  |
| ctrl i                          | 代码提示         |
| Shift + Alt + F(右键菜单更方便) | 全局格式化代码   |
|                                 |                  |



ctrl + d：复制一行文本到剪切板，一次有效

shfit + alt + 下：快速复制粘贴到下一行

ctrl + shift + c：打开cmd命令窗口

多个光标：alt + 鼠标点





VSCODE中创建模板（vue代码片段）

- File选项卡  ==>  preference  ==> User snnipts  ==> 搜索框输入 html  ==> 打开html.json   ==> 配置
- 文件选项卡  ==> 首选项  ==> 用户代码片段  ==> 搜索框输入 html  ==> 打开html.json   ==> 配置
  用于快捷键创建文本

