# Shell

sh：可能不支持方向键，会打印`^[[A`
bash：是对sh的改进，支持自动补全、历史记录、脚本编程
zsh：是对bash的改进，支持高级自动补全、主题、插件
最佳实践：服务器上不要修改 /etc/passwd 的 /bin/bash，而是在客户端配置

[字体：Fira Code](./../toools/Font.md)



## cmd



## powershell

```powershell
Test-NetConnection -ComputerName 127.0.0.1 -Port 7890

ComputerName     : 127.0.0.1
RemoteAddress    : 127.0.0.1
RemotePort       : 7890
InterfaceAlias   : Loopback Pseudo-Interface 1
SourceAddress    : 127.0.0.1
TcpTestSucceeded : True  # 表示端口可访问且有服务器监听
```



## bash

```bash
Nmap -p7890 127.0.0.1  # 看端口是否开放与是什么服务
```



### 配置

`~/.bashrc` 每次打开终端时执行
`~/.bash_profile` 在登录时执行

```~/.bashrc
alias hi="echo 'frank is awesome'"  # 自定义命令别名
```

```bash
############################################################################################
# gitbash禁用路径转换，不让绝对路径转相对
export MSYS_NO_PATHCONV=1

############################################################################################
# custom
# 启动 ssh-agent
env=~/.ssh/agent.env
agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }
agent_load_env  # 加载环境变量

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }
# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)  # stdout重定向到/dev/null、stderr重定向到stdout
if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start  # 环境变量SSH_AUTH_SOCK未设置、变量agent_run_state==2则启动ssh-agent
    ssh-add
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add
fi

unset env

ssh-add ~/.ssh/id_ed25519_archalpha
echo '启动ssh-agent 并ssh-add添加私钥'

############################################################################################
# bash整合starship
eval "$(starship init bash)"
echo "bash整合starship"

############################################################################################

# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
if [ -f '/d/Applications/Scoop/apps/mambaforge/current/Scripts/conda.exe' ]; then
    eval "$('/d/Applications/Scoop/apps/mambaforge/current/Scripts/conda.exe' 'shell.bash' 'hook')"
fi

if [ -f "/d/Applications/Scoop/apps/mambaforge/current/etc/profile.d/mamba.sh" ]; then
    . "/d/Applications/Scoop/apps/mambaforge/current/etc/profile.d/mamba.sh"
fi
# <<< conda initialize <<<
echo "conda mamba 初始化"

############################################################################################

echo "fake resolve bug in gitbash in vscode"
source /d/Applications/Scoop/apps/mambaforge/current/etc/profile.d/conda.sh  # 解决vscode中gitbash: D:\Applications\Scooppps/mamba: No such file or directory
conda --version

############################################################################################

```



## zsh



### 插件管理器

[zim#比较插件管理器延迟](https://github.com/zimfw/zimfw?tab=readme-ov-file)



#### zinit(半推荐)

之前zinit作者删库不维护了，现在的 [zinit github](https://github.com/zdharma-continuum/zinit) 由社区维护

异步加载

snippet 支持omz插件



####  oh-my-zsh(omz)

oh-my-zsh：zsh管理工具



##### 配置

`~/.zshrc`

```bash
# 主题(不建议图标 CMD不支持) https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="powerlevel10k/powerlevel10k"
```

```bash
p10k configure
```

```bash
# 插件 https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/
plugins=(
  git
  z
  # 第三方插件
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```



## fish(推荐)

缺点：不兼容bash语法，执行bash脚本需要切换shell
优点：基本功能由fish支持、**0配置开箱即用方便同步**
我的选择：感觉zsh的插件管理器还是冷门，omz无异步不想用，还是推荐 fish + starship 只需要配置一下prompt

[官网](https://fishshell.com/)	|	[github](https://github.com/fish-shell/fish-shell)	|	TODO: 有空可以看 [fish教程tutorial](https://fishshell.com/docs/current/tutorial.html)
[插件管理器fisher](https://github.com/jorgebucaran/fisher)	|	[插件](https://github.com/jorgebucaran/awsm.fish)



## 插件

| 插件                                                         | shell | 意图                                                         |
| ------------------------------------------------------------ | ----- | ------------------------------------------------------------ |
| git                                                          | zsh   | git命令别名                                                  |
| docker                                                       | zsh   | docker命令别名                                               |
| autojump                                                     | zsh   | j 跳转                                                       |
| jethrokuan/z (推荐)                                          | zsh   | z 跳转                                                       |
| zsh-syntax-highlighting                                      | zsh   | 语法检查高亮                                                 |
| zsh-autosuggestions                                          | zsh   | ctrl -> 自动补全                                             |
| fzf-tab                                                      | zsh   | tab增强 模糊搜索                                             |
| zsh-history-substring-search                                 | zsh   | 历史命令搜索<br />ctrl r 反向命令搜索使用前缀匹配，而fzf是字符串匹配 |
|                                                              |       |                                                              |
| [gitstatus](https://github.com/romkatv/gitstatus)(推荐)<br />romkatv写的其他插件(p10k)也可以看看，但有限支持 | zsh   | 异步(守护进程) prompt for git                                |
| [hydro](https://github.com/jorgebucaran/hydro) 贡献还比较少，异步很重要，非异步会增加延迟 | fish  | 异步 prompt for git                                          |
|                                                              |       |                                                              |



# **prompt**

不止theme，关键还需要提供异步 prompt for git (会显示[⇡] 表示git 状态)

[基准测试](https://github.com/romkatv/zsh-bench#prompt)



## Powerlevel10k

a zsh prompt theme
作者romkatv，zsh插件大佬



## starship(推荐)

一个**跨平台**(外部进程)的prompt
虽然基准测试显示速度不如gitstatus，但是跨平台

[官方文档](https://starship.rs/zh-CN/config/)	|	[github](https://github.com/starship/starship)



安装：

```bash
scoop bucket add nerd-fonts
scoop install nerd-fonts/FiraCode-NF
# terminal set FiraCode-NF

scoop install main/starship
# powersehll
echo $PROFILE
C:\Users\vp11\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
echo "Invoke-Expression (&starship init powershell)" >> ./Microsoft.PowerShell_profile.ps1
# bash
echo 'eval "$(starship init bash)"' >> ~/.bashrc
# zsh
echo 'eval "$(starship init zsh)"' >> ~/.bashrc
# fish
echo 'starship init fish | source' >> ~/.config/fish/config.fish
# cmd  # 问题1:目录promt不变
# 配置clink
# clink官网: https://chrisant996.github.io/clink/clink.html
# github: https://github.com/chrisant996/clink
# https://chrisant996.github.io/clink/clink.html#gettingstarted_customprompt
scoop install main/clink  # 对cmd.exe的增强，提供的feature含 Scriptable Prompt
clink --help
clink inject  # 注入cmd.exe进程
clink autorun show
clink autorun install  # 在cmd.exe's autorun安装一个用于启动clink的命令
# clink安装几个popular-scripts
scoop install main/clink-completions
# clink集成starship  # https://chrisant996.github.io/clink/clink.html#popular-scripts
clink info  # 找到clink script directory
echo 'load(io.popen('starship init cmd'):read("*a"))()' >> D:\Applications\Scoop\apps\clink\current\starship.lua
```



使用

```bash
starship --help
starship explain  # 解释当前显示的模块
starship preset no-nerd-font -o ~/.config/starship.toml
```



配置：`~/.config/starship.toml`

目前预设preset是Nerd Font Symbols





# terminal

[tabby](https://github.com/Eugeny/tabby)	|	[WindTerm](https://github.com/kingToolbox/WindTerm)	|	[warp(收费)](https://www.warp.dev/)



