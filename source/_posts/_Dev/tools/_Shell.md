# Shell

sh：可能不支持方向键，会打印`^[[A`
bash：是对sh的改进，支持自动补全、历史记录、脚本编程
zsh：是对bash的改进，支持高级自动补全、主题、插件

[字体：Fira Code](./../toools/Font.md)



## cmd



## powershell



## bash

可看Linux笔记
目前，我Windows使用bash、Linux使用zsh



### 配置

`~/.bashrc` 等同`C:\Users\qq109\.bashrc`，zsh也有zshrc，每次打开终端时执行，可以自定义命令别名

```
alias hi="echo 'frank is awesome'"
```

`~/.bash_profile` 在登录时执行



## zsh(推荐)

###  oh-my-zsh

oh-my-zsh：zsh管理工具



#### 配置

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
  git  # git命令缩写
  z  # z跳转命令  
  # 第三方插件
  zsh-autosuggestions  # 右键补全
  zsh-syntax-highlighting  # 语法检查
)
```



## fish

不兼容bash语法



## terminal

[tabby](https://github.com/Eugeny/tabby)	|	[WindTerm](https://github.com/kingToolbox/WindTerm)



