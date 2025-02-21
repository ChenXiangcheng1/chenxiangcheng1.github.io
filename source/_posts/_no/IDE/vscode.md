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



### Comment Translate

能够翻译悬浮窗口中的API文档 (强烈推荐)



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

#### C/C++扩展

[Microsoft C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
GUN的子项目
MinGW(Minimalist GNU For Windows)：是一直到Windows的GNU工具集合，停止更新GCC版本停滞
MinGW-w64：能编译生成64位可执行程序 `scoop install main/mingw`
	GCC13.2.0(GNU C Compiler / GNU Compiler Collection)：编译C/C++
	G++：编译C++

**LLVM**(Low Level Virtual Machine)的子项目(推荐)
Clang前端：兼容GCC

##### 一代配置

```json
// https://code.visualstudio.com/docs/editor/tasks
// 在调试会话之前执行的任务，一般为编译程序，包含编译命令如g++ test.cpp -o test
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build", // 任务名称，与launch.json的preLaunchTask相对应
            "type": "process", // process是vsc把预定义变量和转义解析后直接全部传给command；shell相当于先打开shell再输入命令，所以args还会经过shell再解析一遍
            // "problemMatcher":"$gcc", // 此选项可以捕捉编译时终端里的报错信息；但因为有Lint，再开这个可能有双重报错
            "group": {
                "kind": "build",
                "isDefault": true // 不为true时ctrl shift B就要手动选择了
            },
            "presentation": {
                "echo": true,
                "reveal": "always", // 执行任务时是否跳转到终端面板，可以为always，silent，never。具体参见VSC的文档
                "focus": false,     // 设为true后可以使执行task时焦点聚集在终端，但对编译C/C++来说，设为true没有意义
                "panel": "shared"   // 不同的文件的编译信息共享一个终端面板
            },
            "command": "g++", // 要使用的编译器，C用gcc，C++用g++
            "windows": {
                "args": [
                    "-g", // 生成和调试有关的信息
                    "${file}",
                    "-o", // 指定输出文件名，不加该参数则默认输出a.exe，Linux下默认a.out
                    "${fileDirname}/exeFiles/${fileBasenameNoExtension}.exe",  // 编译输出目录
                    "-Wall", // 开启额外警告，如果不想要额外警告，可以删除
                    "-static-libgcc",     // 静态链接libgcc，一般都会加上
                    "-fexec-charset=GBK", // 生成的程序使用GBK编码，不加这一条会导致Win下输出中文乱码 Linux下不需要加
                    // "-std=c11",
                    //"-std=c++17", // C++最新标准为c++17，或根据自己的需要进行修改
                ]
            }
        }
    ]
}
```

```json
// https://github.com/Microsoft/vscode-cpptools/blob/master/launch.md
// 调试配置，也就是用gdb来调试program，按照args参数执行命令
// 调试会用到mingw中的gdb，gdb不支持中文路径
// 点击变量中Register，会退出调试
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "preLaunchTask": "build", // 在调试会话之前执行的任务，一般为编译程序。与tasks.json的label相对应
            "type": "cppdbg", // 配置类型，cppdbg对应cpptools提供的调试功能，此处只能是cppdbg
            "request": "launch",
            "program": "${fileDirname}/exeFiles/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录
            "environment": [],
            "externalConsole": false, // 为true时使用单独的cmd窗口，与其它IDE一致；为false可调用VSC内置终端
            "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，你应该不需要对gdb手动输命令吧？
            "MIMode": "gdb",  // 指定连接的调试器
            "miDebuggerPath": "E:/Develop/mingw64/bin/gdb.exe", // 调试器路径，Windows下后缀不能省略，Linux下则不要，注意替换成自己的路径
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                }
            ]
        }
    ]
}
//${workspaceFolder}：工作区文件夹
//${fileDirname}：文件所在目录
```



##### 最终配置

```json
{
    "configurations": [
        {
            "name": "Nemesis的配置",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "windowsSdkVersion": "10.0.17134.0",  // TODO
            "cStandard": "c23",  // TODO
            "cppStandard": "c++23",  // TODO
            "intelliSenseMode": "windows-gcc-x64",
            "compilerPath": "E:/Develop/mingw64/bin/g++.exe"  // TODO
        }
    ],
    "version": 4
}
```

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Nemesis生成可执行文件",
            "type": "process",
            "group": "build",
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            },
            "command": "g++",  // TODO
            "windows": {
                "args": [
                    "-g",
                    "${file}",
                    "-o",
                    "${fileDirname}/exeFiles/${fileBasenameNoExtension}.exe",  // TODO
                    "-Wall",
                    "-static-libgcc",
                    "-fexec-charset=UTF-8",
                    "-std=c++23"  // TODO:
                    // g++ -dM -E -x c++ - < /dev/null | grep __cplusplus  // 显示 #define __cplusplus 201703L 表示默认 C++17
                ]
            }
        }
    ]
}
```

```json
{
    "configurations": [
        {
            "name": "Nemesis来DEBUG可执行文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/exeFiles/${fileBasenameNoExtension}.exe",  // TODO
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,  // TODO
            "MIMode": "gdb",
            "miDebuggerPath": "E:\\Develop\\mingw64\\bin\\gdb.exe",  // TODO
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",  // 为 gdb 启用整齐打印
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "Set Disassembly Flavor to Intel",  // 将反汇编风格设置为 Intel
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "Nemesis生成可执行文件"  // TODO
        }
    ],
    "version": "2.0.0"
}
```

```markdown
C/C++插件提供了IntelliSense, debugging的功能，配置是json with comment格式在.vscode目录中

| 配置                  | 释义                      |  文档|
| --------------------- | ------------------------- | ---- |
| c_cpp_properties.json | 设置编译器路径和智能感知  | |
| tasks json            | 设置build构建(编译、链接) | [变量](https://code.visualstudio.com/docs/editor/variables-reference) |
| launch.json           | 设置debugger              | |

tasks 生成可执行文件，launch进行调试
```





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



### Golang

[vscode-go](https://github.com/golang/vscode-go/wiki): 配置有点多，建议抄别人的

LSP: gopls
linter: golangci-lint

TODO: 先学Golang，再看linter



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
| ctrl shift p                    | 打开命令面板     |
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

