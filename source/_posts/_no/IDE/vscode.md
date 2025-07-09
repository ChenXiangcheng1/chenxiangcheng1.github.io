# vscode

[Manual](https://code.visualstudio.com/docs)

缺点：不如Pycharm
1没有Override列表，要逐个重写
2不能部分格式化

优点：节约内存、自定义工具链

## 配置

对env起作用的配置文件顺序：`/etc/bash.bashrc`、`~/.bashrc`、`~/.bash_profile`

对应 defaultSettings.json 来设置 Perferences: User Settings (JSON)

`/{user}/.vscode-server/data/Machine/settings.json`

```json
{
 "http.useLocalProxyConfiguration": false
}
```

### tasks.json

tasks json 设置build构建(编译、链接)
tasks 生成可执行文件

> C and C++ projects may define multiple (sometimes dozens or even hundreds) of executables, creating a launch.json may be difficult.
大项目不推荐

[vscode#variables-reference(其中workspaceFolder是根据打开文件决定的)](https://code.visualstudio.com/docs/editor/variables-reference)
[CMake#command substitution](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/cmake-settings.md#command-substitution)

[cmaketools#tasks](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/build.md)

```json
// 终极版  // 有时注释会消失
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "echo cmd substitution",
            "detail": "显示",
            "type": "shell",
            "command": "echo",
            "args": [
                "${command:cmake.activeFolderPath}",
                "${command:cmake.launchTargetPath}"
            ],
            "problemMatcher": []
        },
    ]
}
```

```json
{
    "version": "2.0.0", 
    "tasks": [
        {
            "label": "compile",
            "type": "process",
            "command": "clang++",
            "args": ["-g", "${file}", "-o", "${fileBasenameNoExtension}"]
        },
        {
            "label": "run-executable",
            "type": "process",
            "command": "./${fileBasenameNoExtension}",
            "args": []
        },
        {
            "label": "build-and-run",
            "dependsOrder": "sequence",  // 按顺序执行
            "dependsOn": ["compile", "run-executable"]
        }
    ]
}
```

```json
// https://code.visualstudio.com/docs/editor/tasks
// https://code.visualstudio.com/docs/cpp/config-clang-mac
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

### launch.json

launch.json 设置debugger
launch进行调试

> CMake Tools lets you start a debugger on a target without creating a launch.json。
>
> Note: Only the debugger from Microsoft's vscode-ms-vscode.cpptools extension supports quick-debugging. See Debug using a launch.json file, below, for information about launch.json and using other debuggers.
>
> 如果使用clangd或llvm需要手动配置launch.json

[cmaketools#launch](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/debug-launch.md#debug-using-a-launchjson-file)

```json
// 终极版
{
    "version": "0.2.0",
    "configurations": [
        {
            // vscode:
            "name": "LLDB by CodeLLDB CMake_Tools",
            "type": "lldb",  // lldb是CodeLLDB插件 lldb-dap是LLDB_DAP插件
            "request": "launch",  // launch创建新进程 attch连接到现有进程
            // "console": "integratedTerminal",  // CodeLLDB的"terminal"优先级更高
            "preLaunchTask": "CMake: 生成",
            // CodeLLDB:
   "sourceLanguages": ["cpp"],
            // CodeLLDB_launch:
            "program": "${command:cmake.launchTargetPath}",  // "${command:cmake.launchTargetPath}" // 可执行文件路径 // "${workspaceFolder}/build/<executable file>"
            "args": [],
            "cwd": "${workspaceFolder}",  // current working directory // ${command:cmake.launchTargetDirectory}
            "env": {
    // "PATH": "${env:PATH}:${command:cmake.getLaunchTargetDirectory}",  // 将build目录添加到PATH中
    // "OTHER_VALUE": "Something something"
   },
            "terminal": "integrated",  // console integrated(默认) external
            "stopOnEntry": false,  // 是否停止调试器(debugee)在入口处
        }
    ]
}
```

```json
{
 // 使用 IntelliSense 了解相关属性。
 // 悬停以查看现有属性的描述。
 // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
 "version": "0.2.0",
 "configurations": [
  {
   "name": "Launch Package",
   "type": "go",
   "request": "launch",
   "mode": "auto",
   "program": "${fileDirname}",  // 打开要执行的文件，点击开始调试
   "cwd": "${workspaceFolder}",
   "env": {},
   "args": []
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
            "type": "cppdbg", // 配置类型，cppdbg对应cpptools插件提供的调试功能，此处只能是cppdbg。cppvsdbg对应msvc ctest。lldb对应CodeLLDB插件
            "request": "launch",
            "program": "${fileDirname}/exeFiles/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false,  // 入口处停下
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录
            "environment": [],
            "externalConsole": false, // 为true时使用单独的cmd窗口，与其它IDE一致；为false可调用VSC内置终端
            "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，你应该不需要对gdb手动输命令吧？
            "MIMode": "gdb",  // Machine Interface指定连接的调试器
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

## 插件

[插件市场](https://marketplace.visualstudio.com/vscode)

### Comment Translate

能够翻译悬浮窗口中的API文档 (强烈推荐)

### Vim

<https://github.com/VSCodeVim/Vim>

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

### Zig

### C++

#### LSP

##### C/C++扩展

[Microsoft C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
GUN的子项目
MinGW(Minimalist GNU For Windows)：是一直到Windows的GNU工具集合，停止更新GCC版本停滞
MinGW-w64：能编译生成64位可执行程序 `scoop install main/mingw`
 GCC13.2.0(GNU C Compiler / GNU Compiler Collection)：编译C/C++
 G++：编译C++

**LLVM**(Low Level Virtual Machine)的子项目(推荐)
Clang前端：兼容GCC

C/C++插件提供了IntelliSense, debugging的功能，配置是json with comment格式在.vscode目录中

###### c_cpp_properties.json配置

c_cpp_properties.json 设置编译器路径和智能感知

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

##### clangd

LS: `.clangd` [官网](https://clangd.llvm.org/)  
build system(Generator): CMake [cmake-tools](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/README.md)、Bazel  
build: (ninja、makefile)  

[clangd官网](https://clangd.llvm.org)   |   [github](https://github.com/clangd/clangd)  |   [clangd插件](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd
)

> clangd is a language server后台语言服务器  
clangd is based on the Clang C++ compiler, and is part of the LLVM project

[compilation database](https://clang.llvm.org/docs/JSONCompilationDatabase.html)

> Required:
Make sure the Microsoft C/C++ extension is not installed  
**compile flags**(how your project is built). A compile_commands.json file can usually be generated by your build system

Using:

1. pacman -Ss clang  # 含clangd、clang-tidy、clang-format  
2. 安装插件

```bash
clang --help-hidden
clangd --help-hidden

g++ -v --help  # 可用于查看支持的std版本
clang++ -std=invalid -x c++ /dev/null  # 通过错误查看支持的std版本  # https://clang.llvm.org/cxx_status.html
```

###### clang

[官网](https://clang.llvm.org/) |   [cxx_status](https://clang.llvm.org/cxx_status.html)  
[文档](https://clang.llvm.org/docs/UsersManual.html)    |   [UsersManual](https://clang.llvm.org/docs/UsersManual.html)    |   [工具链](https://clang.llvm.org/docs/Toolchain.html)  
TODO: 看文档

clang 语言前端(clang兼容GCC、clang-cl兼容MSVC)

```bash
-W  # 启用警告，all启用常用警告，extra(未使用参数、符号比较等)，error, everything启用所有警告
-pedantic  # 对语言扩展的警告
-fdiagnostics-show-category=name  # 显示warn类别

-stdlib=libc++(clang) -stdlib=libstdc++(GNU)  # 指定C++ standard library
-std=c++23 -std=c23  # 指定clang使用的language mode(c++23:严格ISO C++标准 gnu++23:扩展的)

-x <language>  # 将后续输入文件看作<language>处理(treat)
clang -x c++-header test.h -o test.h.pch  # 生成PCH文件

-E  # Only run the preprocessor预处理
--precompile  # Only precompile the input解析
-emit-llvm  # Use the LLVM representation for assembler and object files编译器前端生成中间表示IR(intermediate representation)
-S  # Only run preprocess and compilation steps 得到汇编文件
-c  # machine code object files 汇编得到机器码目标文件

-MMD  # 输出包含用户头文件的依赖文件
-E  # only run the preprocessor
-dM  # 打印宏定义
-MM  # MMD + E + stdout
-H  # 显示头文件和深度
-print-file-name=<file>  # 打印文件的完整库路径

# 用于debug
-g  # Generate source-level debug information
-O  # set optimization level

-Wa,<arg>  # 传参给assembler
-Wr,<arg>  # 传参给linker
-Wp,<arg>  # 传参给preprocessor

# 性能优化
-ftime-trace  # Turn on time profiler. Generates JSON file(显示编译耗时) based on output filename.

# 命令行的-D<macro>=<value> 等价于#define <macro> 
-D_POSIX_C_SOURCE=200809L
```

###### 宏macro

> 要么在包含任何头文件之前使用使用宏定义
> 要么在编译命令中包含等效定义(cc -D_XX_SOURCE)

```bash
echo | clang -dM -E -x c -  # 打印c的预定义宏
echo | clang -dM -E -x c++ -  # 打印c++的预定义宏
echo | clang -dM -E -x c++ - | grep "GNU"
```

Feature Test Macros
是用于控制库函数的可见性的宏，需要被定义在包含任何头文件之前

文档: `man 7 feature_test_macros`、[glibc手册#Feature Test Macros](https://www.gnu.org/software/libc/manual/html_node/Feature-Test-Macros.html)

> 源码: usr/include/features.h
> 应用程序不应该直接使用__XX宏

```cpp
// clang++默认启用了_GNU_SOURCE但clang没启用
#define _GNU_SOURCE  // 含_DEFAULT_SOURCE、_POSIX_C_SOURCE=200809L  
#define _DEFAULT_SOURCE  // 含__USE_MISC
#define _POSIX_C_SOURCE 200809L  // 表示支持POSIX标准2008版本, POSIX2008支持sigset_t

_XOPEN_SOURCE  // 用于定义SUS的不同版本
```

###### clangd插件配置

[vscode-clangd](https://github.com/clangd/vscode-clangd/blob/master/package.json)

```json
// clangd配置 Start
"clangd.path": "clangd",
"clangd.arguments": [ // clangd --help
    // clangd compilation flags options
    "--compile-commands-dir=${workspaceFolder}/build",  // 指定compile_commamds.json目录
    "--query-driver=/usr/bin/**/clang-*,/usr/bin/**/g++-*",
    // clangd feature options
    "--all-scopes-completion",
    "--background-index",
    "--background-index-priority=normal",
    "--clang-tidy",
    "--completion-style=detailed",
    "--experimental-modules-support",
    "--fallback-style=LLVM",  // doc: https://clang.llvm.org/docs/ClangFormatStyleOptions.html
    "--function-arg-placeholders=true",  // NOTE: 感觉不需要
    "--header-insertion=iwyu",  // iwyu(include what you use)
    "--header-insertion-decorators",
    "--import-insertions",  // NOTE: only for Objective-C
    "--limit-references=1000",
    "--limit-results=100",
    "--rename-file-limit=50",
    // clangd miscellaneous options
    // "--check=xx.cpp",  // Note: useful for debugging clangd
    "--enable-config",  // Note: 启用.clangd in project directory
    "-j=5",  // CPU Core-1
    "--malloc-trim",
    "--pch-storage=memory",  // precompiled headers
    // clangd protocol and logging options
    "--log=error",  // verbose info verbose
    "--offsetEncoding=utf-16",
    "--pretty",
],
// clangd配置 End
```

###### .clangd

配置优先级: vscode-settings.json(直接作为命令行参数) > 用户配置 > 内部项目配置 > 外部项目配置  
用户配置文件: /root/.config/clangd/config.yaml  
项目配置文件需要clangd插件启用--enable-config  

关键是找到compile_commands.json  

```.clangd
---
If:
  # PathMatch: [.*\.cpp, .*\.cxx, .*\.cc, .*\.h, .*\.hpp, .*\.hxx]
  PathMatch: [.*\.cpp]
CompileFlags:
  Add: [
    -xc++, -std=c++23,
    # -static-libstdc++,  # 使用静态版本的libgcc库, 确保在没有DLL的系统上也能运行
    # -fexec-charset=UTF-8,  # 源文件字符集
  ]
  Compiler: clang++
---
If:
  PathMatch: [.*\.c]
CompileFlags:
  Add: [-std=c23]  # cmake不支持c++2y  # -D_GNU_SOURCE
  Compiler: clang
```

#### linter

##### clang-tidy

[extra clang tools](https://clang.llvm.org/extra/index.html)    |   [clang-tidy文档](https://clang.llvm.org/extra/clang-tidy/index.html)

[llvm](https://github.com/llvm/llvm-project/blob/main/.clang-tidy)  |   [.clangd#clangtidy](https://clangd.llvm.org/config#clangtidy)  

clang-tidy is a clang-based C++ “linter” tool. 提供checks(包括clang diagnostic、check diagnostic)、fix

需要clangd插件启用--clang-tidy

.clang中Diagnostics.ClangTidy优先级比.clang-tidy高

```bash
clang-tidy --help-hidden
clang-tidy -list-checks  # 列出默认启动的check
clang-tidy -list-checks -checks=*  # 列出所有check
clang-tidy -list-checks -checks=-*,boost-*  # 列出boost相关的check

clang-tidy --verify-config
clang-tidy --dump-config -p <build_path> -checks=* > .clang-tidy.yaml  # 可作用初始配置

clang-tidy test.cpp -checks=-*,xxx --use-color @parameters_file -- -<编译选项>
```

.clang-tidy.yaml 可以参考--dump-config默认配置  

```.clang-tidy
Checks: "clang-diagnostic-*,clang-analyzer-*,boost-*,bugprone-*,clang-analyzer-*,concurrency-*,performance-*,portability-*,modernize-*,misc-*"
# CheckOptions:
```

#### format

##### clang-format

[文档](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)

如果.clang-format不存在，则看--fallback-style  

```bash
clang-format --help-hidden
clang-format --style=LLVM --dump-config > .clang-format  # 查看配置

clang-format -dump-config  # 可作用初始配置 可用于查看配置是否生效

clang-format --style=file -i main.cpp
```

```.clang-format
BasedOnStyle: LLVM
IndentWidth: 4
---
Language: Cpp
# ColumnLimit: 100
```

#### debug

cppdbg

##### codelldb

[codelldb#github](https://github.com/vadimcn/codelldb)  
[文档](https://github.com/vadimcn/codelldb/blob/master/MANUAL.md)

使用:  

```debug console
help  # debug console
```

```bash
# 生成调试配置launch
@vscode /startDebugging main.cpp #codebase
```

###### lldb

[lldb#github](https://github.com/llvm/llvm-project/blob/main/lldb/docs/index.rst)  
TODO: 看文档 [lldb#doc](https://lldb.llvm.org/)  

##### lldb-dap

[lldb-dap#github](https://github.com/llvm/llvm-project/tree/main/lldb/tools/lldb-dap)

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

TODO: [github](https://github.com/astral-sh/ruff) | [文档](https://docs.astral.sh/ruff/configuration/#fix-safety)

核心是ruff-lsp，代替 flake8、isort、bandit

**配置：**

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

#### format

##### autopep8

<https://github.com/hhatto/autopep8>

##### yapf

<https://github.com/google/yapf>

支持多种格式化的风格

##### black

<https://github.com/psf/black>

统一的格式化标准，约定大于配置

#### 类型检查

alpha阶段：  
Pyrefly from Meta  
Ty from Ruff's Astral Labs  

[pyrefly#vscode插件](https://marketplace.visualstudio.com/items?itemName=meta.pyrefly)
[pyrefly#官网](https://pyrefly.org/)
[pyrefly#github](https://github.com/facebook/pyrefly)

##### pyright

vscode-python类型检查默认就使用pyright

##### mypy

[个人 github](https://github.com/matangover/mypy-vscode)

据说pylance比mypy快

### Golang

### markdown

````md
# 1

## 2

### 3

#### 4

##### 5

###### 6

ctrl shift o  搜索当前文件中的header
ctrl t 搜索当前工作区中的header
shift alt rightarrow 智能选择
f2 更新所有指向该元素的链接/header
双击
视觉分割推荐使用 ---代替空行, 空行的最佳实践是br

123(行尾空格换行 br分段)  
123(行尾没空格同行)
123

**123**

`123`

```python
123
```

$$
\displaystyle
\frac{1}{\Bigl(\sqrt{\phi \sqrt{5}}-\phi\Bigl)}
= 1+\frac{e^{-2\pi}} {1+\frac{e^{-4\pi}} {1+\cdots}}
\tag{使用KaTeX}
$$

Math block:

$$
\displaystyle
\left( \sum_{k=1}^n a_k b_k \right)^2
\leq
\Bigl( \sum_{k=1}^n a_k^2 \Bigr)
( \sum_{k=1}^n b_k^2 )
$$

----------

![alt](https://)

$x^2$

*123*

[text](https://)

[text](##)

1. first
2. second
3. third

> 123

~~123~~

- first
- second
- third
````

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

## 问题

问题1：deproxy+wsl内vscode插件使用127代理
原因：当wsl上http_proxy不存在，会继承宿主机http_proxy
解决：proxy、deproxy+Clash关闭系统代理(宿主机也无http_proxy，走TUN)

# 未整理

<https://zhuanlan.zhihu.com/p/138579730>

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

#### 更改

```json
{
    "key": "alt+left",
    "command": "-workbench.action.navigateBack",  // 删除键绑定，因为和alt+H冲突了
    "when": "canNavigateBack"
}
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
