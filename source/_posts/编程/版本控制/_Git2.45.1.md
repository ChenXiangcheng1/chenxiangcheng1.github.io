# Git2.45.1

分布式版本控制

**强烈建议在Windows上在WSL中使用Git**

![](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/wz/202410160846187.png)

工作区(最底下)：Changes not staged for commit
暂存区：Changes to be committed



## 资料

[Git游戏](https://learngitbranching.js.org/?demonoemo=&locale=zh_CN)



[Git 架构](https://www.jianshu.com/p/c18e472fbf19)

[学习路线](https://www.zhihu.com/question/357385506)

[Git 参考手册](https://git-scm.com/docs)

[Git 备忘单](https://ndpsoftware.com/git-cheatsheet.html#loc=workspace;)

[Git&Github 使用手册](https://githubtraining.github.io/training-manual/#/)

[Github 官方文档](https://docs.github.com/zh)

[Github 备忘单](https://training.github.com/downloads/zh_CN/github-git-cheat-sheet/)

[廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)



## 配置

[官方文档](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git)

`~/.gitconfig` - 全局配置(对所有仓库生效)：

```yaml
[user]  # 此昵称和邮箱与Github登录无关
	name = chenxiangcheng
	email = chenxiangcheng1@gmail.com  # Git提交使用邮件区分
[core]
	autocrlf = false
	# false都不变  #
	# true拉取为CRLF，提交为LF  # win  # 不建议因为本地LF会warn，很烦
    # input拉取不变，提交为LF  # linux
	quotepath = false  # 取消文件名字符转义，使 git status 能显示中文
	editor = vim
	# git config --global core.editor "'E:/Applications/Scoop\apps/notepadplusplus/current/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"  # 设置默认编辑器为notepad++
[http]
    proxy = socks5://127.0.0.1:7890  # 可删，因为若docker内应用host.docker.internal
[https]
    proxy = socks5://127.0.0.1:7890
[credential]  # 感觉不需要配置
	helper = manager  # 使用Https协议访问Git版本库需要用户名和密码  manager选项使用windows存储的密码
	provider = generic  # github(支持token)
	# manager系统凭据管理器、store明文存储于~/.git-credentials
	# git config --global credential.helper
# [credential "helperselector"]  # 不需要配置
# 	selected = manager
# TODO: 查下面这配置什么意思
#[credential "https://codeup.aliyun.com"]
#	provider = generic
[init]
	defaultBranch = main  # 默认master
```

```bash
# .gitattributes

*                  text=auto

# text 文本文件
*.py			   text eol=lf
*.js               text eol=lf
*.css              text eol=lf
*.html             text eol=lf
*.sh			   text eol=lf
*.json             text eol=lf
*.md               text eol=lf

# -text 二进制文件
*.png              -text
*.jpg              -text
*.jpeg             -text
*.pdf              -text
*.svg              -text
```



<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_08_23_18_43.png" alt="image-20230823184300002" style="zoom:50%;" />

命令方式：git config --global core.quotepath false



Win：控制面板=>用户账户=>凭据管理器=>win凭据=>可以删除普通凭据(用户名密码)

*Git安装目录/etc/.vimrc* - GitBash 中的 ViM 编辑器的配置：

```Git目录/etc/.vimrc
set nu

" solve bash vim error, I don't use
"set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
"set termencoding=utf-8
"set encoding=utf-8
"set term=xterm
```



### 配置 SSH 免密登录

争对 ArchLinux ZSH、Git Bash 配置，以访问自己的私有仓库

[官方文档](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/about-ssh)

**1安装 openSSH**

SSH (Secure Shell) 是应用层协议，openSSH是实现。

```zsh
pacman -Ss openssh
pacman -S openssh
ssh --help
OpenSSL version mismatch. Built against 30100010, you have 30000080  // openssh发生错误，有冲突
pacman -Su openssh  // 更新后openssh正常
```

**2生成SSH密钥**

```bash
ssh-keygen -t ed25519 -C "chenxiangcheng1@gamil.com" -f ~/.ssh/id_ed25519_github
# ssh-keygen -t rsa -b 4096 -C "chenxiangcheng1@gamil.com"
```

**3启动ssh-agent代理并添加私钥**

ssh-agent 是基于内存的SSH私钥存储，可在多个会话服用无需每次输入密码
ssh-add

**for ArchLinux ZSH**

```~/.ssh/.zshrc
# 需要设置到 ~/.ssh/.zshrc
eval "$(ssh-agent -s)"  # eval将环境遍历
ssh-add ~/.ssh/id_ed25519_github  # 对私钥高速缓存
```

**for powershell**

```powershell
Get-Service -Name ssh-agent  # 获取服务dispaly名称OpenSSH Authentication Agent
Get-Service -Name ssh-agent | Set-Service -StartupType Manual  # 设置OpenSSH Authentication Agent服务手动启动
Start-Service ssh-agent
ssh-add ~/.ssh/id_ed25519_github
```

**for Git for win**

[github#git for win 自动启动ssh-agent](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows)

```bash
############################################################################################
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

ssh-add /c/Users/qq109/.ssh/id_ed25519_github
echo '启动ssh-agent 并ssh-add添加私钥'

############################################################################################
```

```bash
ssh-add -l -E sha256  # 测试
```

[win自动启动ssh-agent](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows)

**4Github 设置 SSH密钥 新建认证密钥**

```bash
cat ~/.ssh/id_ed25519_github.pub
```

**7代理ssh(可不配置走系统代理)**

去看 `~/.zshrc`

```bash
Host github.com
  User git
  ProxyCommand ncat --proxy-type socks5 --proxy 172.26.128.1:7890 %h %p

Host github.com
     HostName github.com
         #ProxyCommand nc -X connect -x 127.0.0.1:1080 %h %p
         ProxyCommand ncat --proxy 127.0.0.1:1080 %h %p
     User git
     Port 22

Host github.com
    Hostname ssh.github.com  # 实际IP地址
    Port 443
    User git
    ProxyCommand ncat --proxy-type socks5 --proxy ${PROXY_SOCKS5} %h %p


# 配置代理
# https://dowww.spencerwoo.com/2-cli/2-3-cli-tools.html#%E4%BB%A3%E7%90%86%E9%85%8D%E7%BD%AE
# Fetch Windows ip address inside WSL environment
WINDOWS_IP=$(ip route | grep default | awk '{print $3}')
PROXY_HTTP="http://${WINDOWS_IP}:7890"
PROXY_SOCKS5="${WINDOWS_IP}:7890"

# Git & SSH for Git proxy
proxy_git () {
  git config --global http.https://github.com.proxy ${PROXY_HTTP}
  echo "git proxy"

  if ! grep -qF "Host github.com" ~/.ssh/config ; then
    echo "Host github.com" >> ~/.ssh/config
    echo "    Hostname ssh.github.com" >> ~/.ssh/config
    echo "    Port 443" >> ~/.ssh/config
    echo "    User git" >> ~/.ssh/config
    #echo "  ProxyCommand nc -X 5 -x ${PROXY_SOCKS5} %h %p" >> ~/.ssh/config
    echo "    ProxyCommand ncat --proxy-type socks5 --proxy ${PROXY_SOCKS5} %h %p" >> ~/.ssh/config
    echo "ssh proxy: edit1 ~/.ssh/config"
  else
    # line_no=$(($(awk '/Host github.com/{print NR}'  ~/.ssh/config)+3))
    # echo "${line_no}"
    #sed -i "${lino}c\  ProxyCommand nc -X 5 -x ${PROXY_SOCKS5} %h %p" ~/.ssh/config
    #sed -i "${line_no}c\  ProxyCommand ncat --proxy-type socks5 --proxy ${PROXY_SOCKS5} %h %p" ~/.ssh/config
    if ! grep -qF "ProxyCommand" ~/.ssh/config ; then
      echo "    ProxyCommand ncat --proxy-type socks5 --proxy ${PROXY_SOCKS5} %h %p" >> ~/.ssh/config
    fi
    echo "ssh proxy: edit2 ~/.ssh/config"
  fi
}

unset_ssh_proxy () {
  sed -i '/ProxyCommand/d' ~/.ssh/config
  echo "cancel ssh proxy"
}

# Set proxy
set_proxy () {
  export http_proxy="${PROXY_HTTP}"
  export https_proxy="${PROXY_HTTP}"
  echo "http/https proxy: ${PROXY_HTTP}"
  proxy_git
}

# Unset proxy
unset_proxy () {
  unset http_proxy
  unset https_proxy
  git config --global --unset http.https://github.com.proxy
  echo "cancel http/https/git proxy"
  unset_ssh_proxy
}

# Set alias
alias proxy=set_proxy
alias deproxy=unset_proxy
```

>[ncat文档](https://nmap.org/ncat/guide/ncat-tricks.html#ncat-ssh-tunnel) Now you can make connections inside the network using Ncat's proxy client capabilities. For example, to connect to host with IP address 192.168.1.123 that is behind the router, you can use the following command if you spawned the tunnel:
>
>`ncat --proxy-type socks5 --proxy localhost:7890 192.168.1.123`

**配置 GitHub 的公钥指纹**

[github-ssh-key-fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)

```~/.ssh/known_hosts
github.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOMqqnkVzrm0SdG6UOoqKLsabgH5C9okWi0dh2l9GKJl
github.com ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEmKSENjQEezOmxkZMy7opKgwFB9nkt5YRrYMjNuG5N87uRgg6CLrbo5wAdT/y6v0mKV0U2w0WZ2YB/++Tpockg=
github.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCj7ndNxQowgcQnjshcLrqPEiiphnt+VTTvDP6mHBL9j1aNUkY4Ue1gvwnGLVlOhGeYrnZaMgRK6+PKCUXaDbC7qtbW8gIkhL7aGCsOr/C56SJMy/BCZfxd1nWzAOxSDPgVsmerOBYfNqltV9/hWCqBywINIR+5dIg6JTJ72pcEpEjcYgXkE2YEFXV1JHnsKgbLWNlhScqb2UmyRkQyytRLtL+38TGxkxCflmO+5Z8CSSNY7GidjMIZ7Q4zMjA2n1nGrlTDkzwDCsw+wqFPGQA179cnfGWOWRVruj16z6XyvxvjJwbz0wQZ75XK5tKSb7FNyeIEs4TT4jk+S4dhPeAUC5y+bDYirYgM4GC7uEnztnZyaVWQ7B381AK4Qdrwt51ZqExKbQpTUNn+EjqoTwvqNj4kqx5QUCI0ThS/YkOxJCXmPUWZbhjpCg56i+2aB6CmK2JGhn57K5mj0MNdBXA4/WnwH6XoPWJzK5Nyu2zB3nAZp+S5hpQs+p1vN1/wsjk=
```

**问题：**

问题1：

```bash
ssh -vT git@github.com
git@github.com: Permission denied (publickey).
```

原因：没启动 `ssh-agent`、`ssh-add`
解决：先 `ssh-add -l -E sha256` 看看

**5问题2：**

```bash
ssh -vT git@github.com
Connection closed by UNKNOWN port 65535
```

原因：**大多数机场会封22端口** 或 代理没配置好
解决1：不开代理
解决2：配置通过443端口的SSH访问github，[官方文档](https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)	|	[flowercloud文档](https://help.huacloud.dev/huacloud/GITHUB-22-464e1622cc92460b9a419bba45a60f75)

```~/.ssh/config
Host github.com
Hostname ssh.github.com  # 实际IP地址
Port 443
User git
```

**6最终测试**

```bash
# 重启终端：直连(不开代理) 成功
ssh -vT git@github.com
Hi ChenXiangcheng1! You've successfully authenticated, but GitHub does not provide shell access.
git clone git@github.com:ChenXiangcheng1/test.git
```



```bash
telnet 172.30.208.1 7890
nc -vz 172.30.208.1 7890
```

| 命令       | 释义                                                      | 选项 |
| ---------- | --------------------------------------------------------- | ---- |
| ssh-keygen | 生成 SSH 密钥文件，支持 RSA 和 DSA 两种非对称加密密钥格式 |      |
|            |                                                           |      |
| ssh        |                                                           |      |
| ssh-agent  | 运行时密钥存储                                            |      |
| scp        |                                                           |      |
| sftp       |                                                           |      |





## 命令

```bash
git status --short
git clone --depth=1  # 只下载最新提交，适合获取代码简单查看不改 --shallow-submodules # 浅克隆子模块
git switch  # 替代 git checkout
git restore  # 丢弃所有更改，恢复到最新提交状态(HEAD)
```



### git merge 和 rebase 的区别

共：把一个分支的提交合并到另一个分支

merge：合并
rebase：嫁接
	缺点：commit顺序和提交时间不一致，会乱掉

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2024_0525_164838.png" alt="image-20240525164838756" style="zoom: 60%;" />

`git rebase main` 把自己的分支dev变基到目标分支main上，也就是把自己的提交历史放在目标分支的最后，这样可以保持提交历史的整洁和线性

`git merge main` 把目标分支合并到自己的分支上，也就是把目标分支的提交历史和自己的提交历史合并成一个新的提交，这样可以保持提交历史的完整和真实



### 存档库操作

| 命令                                                         | 释义 |
| ------------------------------------------------------------ | ---- |
| git stash list                                               |      |
| git stash<br />git stash push -m "stash(mquant/adapter): only import order" |      |
| git stash apply stash@{9}                                    |      |
| git stash drop stash@{9}                                     |      |



### 场景(重点)

#### 上传新项目

```bash
git init
git add ./README.md
git commit -m "Initial commit"
git remote add origin git@github.com:TrackyTian/testSSH.git  # 添加远程主机名
git branch -m main  # 分支重命名
git push origin main:main  # 需要credentials(推荐使用person access token)
git checkout -b dev  # 切换并根据当前分支创建分支
```

```bash
# 扩展学习
git push -u origin main
||
git push --set-upstream origin main  # 用于git push origin、git pull origin操作省略分支名，如果当前分支只有一个上游分支还可以继续省略远程主机名
git push origin main

git branch –unset-upstream <本地分支名>  # 解除本地main分支与上游分支的关联

git pull <远程主机名> <来源地:远程分支名>:<目的地:本地分支名>  # pull完整命令，拉取当前分支
git push <远程主机名> <来源地:本地分支名>:<目的地:远程分支名>  # push完整命令，推送当前分支
git push <远程主机名> :<目的地:远程分支名>  # 删除远程分支

# 若远程分支比本地分支新，仍想强行推送
git push --force

git pull  # 不推荐
||
git fetch
git merge

git pull --rebase=true  # 推荐
||
git fetch
git rebase

# rebase基操
git rebase --abort  # 撤销rebase
git rebase --continue  # 解决冲突

git log --graph  # 查看提交记录
```



#### 取消git add

```bash
git ls-files  # 查看暂存区中有哪些文件
git rm -r --cached <target_file>  # 删除暂存区文件
```



#### rebase定期从远程主分支拉取更新

```bash
git checkout dev
git stash
git pull --rebase origin main  # 拉取远程主分支的最新更新并合并到本地分支。如果有冲突则需要手动解决冲突
# 若有冲突，手动解决冲突:
# 1 解决冲突后 git rebase --continue
# 2 跳过冲突的提交 git rebase --skip
# 3 中止rebase，回到变基前 git rebase --abort
git stash pop  # 本质上是 git stash apply、git stash drop  # 冲突时不执行drop，再次pop确保 git stash list 为空

# 等价于
git fetch
git rebase
git add ...
git rebase --continue
# git rebase --abort  # 终止rebase回到rebase之前的初始状态

# 之后就可以commit、push origin main啦
```

```bash
# 扩展git pull本质
git pull = fetch + merge

git fetch upstream origin == git remote update upstream  # 远程仓库分支(.git/refs/remotes/*) -> 远程分支(.git/refs/heads/\*)

git merge  # 远程分支 -> 本地分支
git merge --squash  # squash策略的压缩merge
```





#### merge合并和解决冲突

```bash
git merge  # 默认为Fast-Forward Merge
当前分支的 HEAD 指针在要合并的分支的历史提交之前，即当前分支没有任何新的提交。当前分支直接指向最后一个提交

git merge --no-ff
```



方式1：git合并方式

在feature-xxx特性分支上进行开发并提交change
切换回主分支，pull最新的change
执行合并操作 (git merge) 将feature-xxx特性分支的change合并到主分支：git rebase dev(线性结构清晰，发生冲突需要强制推送) 或 git merge dev(发生冲突不需要强制推送，合并提交不清晰)
如果在合并过程中发生冲突，手动解决冲突并提交解决方案。
进行代码审核和测试
推送主分支的更改到远程仓库

```bash
git checkout feature-xxx
git status
git add .
git commit -m "完成开发"
git checkout main
git pull
git merge feature-xxx
手动解决冲突
git add <files>
git commit -m "Resolve merge conflict"
PullRequest 代码审核和测试
git push origin main
```

方式2：git储藏方式

```bash
git checkout feature-xxx
git stash
git pull
git stash apply
手动解决冲突
git stash drop
git add <files>
git commit -m "完成开发"
PullRequest 代码审核和测试
git push origin main
```

场景单分支：

```
git pull origin main
git status
git add .
git commit -m "update: xxxx"
git push orgin main
```



#### 创建新本地分支

`git clone -b <name> <url | base_branch-name(默认为当前分支)>` 只能 clone 到新目录

```bash
# 查看分支
git branch --list
* feature-atxfile
  main

git show-branch
* [feature-atxfile] add get_prev_trade_date
 ! [main] add get_prev_trade_date
--
*+ [feature-atxfile] add get_prev_trade_date

git branch -vv  # 推荐
* feature-atxfile d3030e5 add get_prev_trade_date
  main            d3030e5 add get_prev_trade_date

git branch -a
* feature-atxfile
  main
  remotes/origin/HEAD -> origin/main
  remotes/origin/feature-atxfile
  remotes/origin/main

# 创建新本地分支
git remote add origin <url>
git fetch origin  # 更新远程仓库分支的最新状态(引用) .git/refs/heads/<branch-name>
git checkout -b new-branch origin/main
```



```bash
git branch -d feature/baseinfo  # 删除本地分支
git push origin :feature/baseinfo  # 删除远程分支
```





#### commit

commit 格式 `<type>(<scope>): <subject>`

type
	**feat**：新增功能
	**fix**：修复Bug
	docs：文档变更
	style：代码格式（不影响代码运行的变动）
	refactor：重构（即不是新增功能，也不是修改Bug的代码变动）
	test：增加测试
	chore：构建过程或辅助工具的变动(杂货)
	sync()
scope影响范围
	root
subject内容





#### 撤销commit

```bash
git reset HEAD^  # 回退所有内容到上一个版本
git reset  HEAD~3  # 向前回退到第3个版本

git reset HEAD^ test.txt  # 回退test.txt这个文件的版本到上一个版本

git reset 51363e6  # 回退到某个版本51363e6
```



#### PR

origin：是默认名，一般约定为自己的远程仓库
upstream：一般约定为fork的源仓库

```bash
git remote add upstream https://github.com/user/project.git
git pull upstream master:<branch_name>
修改代码，并commit
git pull --rebase upstream master:<branch_name>
git push origin <branch_name>
提交PR
```



#### 设置上游仓库

```bash
git branch -vv
git branch --set-upstream-to=<远程仓库>/<远程分支>
git branch --unset-upstream
```



## .gitignore

[github .gitignore模板](https://github.com/github/gitignore)

| .gitignore文件  | 用于配置哪些文件需要添加到Git的版本管理中，push又上传哪些文件 |
| --------------- | ------------------------------------------------------------ |
| – 将被 Git 忽略 | 注释                                                         |
| **目录**        |                                                              |
| build/          | 忽略所有 build/ 目录下的所有文件                            |
| /mtk/           | 忽略根目录下的整个文件夹                              |
| **文件**        |                                                              |
| /mtk/do.c       | 过滤某个具体文件                                             |
| /TODO           | 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO         |
| *.a             | 忽略所有 .a 结尾的文件                                       |
| doc/*.txt       | 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt            |
| **include**     |                                                              |
| !lib.a          | 但 lib.a 除外                                                |
| !*.zip          | Git会将满足这类规则的文件添加到版本管理中                    |





## .gitattributes

用于处理行尾

```.gitattributes
# text  # 标记为文本文件
# eol = lf    # 入库时将行尾规范为LF，检出时不操作(保持LF)(推荐)
# eol = crlf  # 入库时将行尾规范为LF，检出时将行尾转换为CRLF

* text=auto
*.cpp text eol=lf
*.h text eol=lf
*.c text eol=lf
*.hpp text eol=lf
*.cmake text eol=lf
*.sh text eol=lf
*.py text eol=lf
*.png binary
```





## 证书

https://open-source-license-chooser.toolsnav.top/zh/

| License           | 许可协议             |
| ----------------- | -------------------- |
| MIT License:      | 要开放就用这个       |
| Apache License2.0 | 别人要用就要申请版权 |
| GPL               |                      |



## 问题

**问题1**： CMD终端中文乱码( `javac --help`、`git status`)
原因：CMD终端编码问题

```cmd
>git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   DDB閲忓寲閲戣瀺鑼冧緥/ddb_example_1.dos
        modified:   WorldQuant 101 Alpha 鍥犲瓙鎸囨爣搴?calc_factor.dos
```

解决：

```cmd
chcp 65001  # 更改活动代码页，936:GB2312，20127:US-ASCII，65001:UTF-8
```



**问题2**：Termius Git Bash 中文乱码(git status)
原因：Termius终端编码问题

```bash
$ git status
On branch main
Your branch is up to date with 'origin/main'.
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        a.txt
        涓枃/
```

解决：

无效：

```.gitconfig
# .gitconfig
[gui]
    encoding = utf-8  #代码库统一使用utf-8
[i18n]
    commitencoding = utf-8  #log编码
[svn]
    pathnameencoding = utf-8  #支持中文路径
```

有效：

```txt
由于Termius终端设置没有本地字符集utf-8选项，需要在系统控制面板>区域>区域设置>选择在不支持Unicode的程序中使用UTF-8支持(Beta)(会造成GB2312代码乱码)
```



**问题3**：`git status`显示三位数字 /315
解决：`git config --global core.quotepath false`



**问题4**：`OpenSSL SSL_connect: Connection was reset in connection to github.com:443`
原因1：代理商问题，换代理商
原因2：若是设置问题
临时解决 `set HTTP_PROXY=xxx; set HTTPS_PROXY=xxx`



**问题5**：git 警告 LF will be replaced by CRLF the next time Git touches it



**问题6**：fatal: unable to access 'https://github.com/username/project.git/': Failed to connect to 172.23.144.1 port 7890 after 131875 ms: Could not connect to server 
原因：出站规则针对程序应用无效
解决：deproxy(走TUN)、设置防火墙出栈规则(针对端口 不要争对应用)、关闭防火墙



# Github

[docs](https://docs.github.com/zh)  | [skills.github.com](https://skills.github.com/)

issue/milestones 版本发布规划



杂

api limit：`https://api.github.com/rate_limit`
仓库大小：`https://api.github.com/repos/Eikanya/Live2d-model`



## Github Pages

[Github Pages官方文档](https://docs.github.com/zh/pages/getting-started-with-github-pages/about-github-pages)	|	[Github Action官方文档](https://docs.github.com/zh/actions/quickstart)
[pages](https://pages.github.com/) 是 github 提供的静态页面服务。分为两类：

* 用户页面<username>.github.io
* 项目页面(需要在settings>pages中设置)

| 仓库                 | Page         | 展示地址                         |
| -------------------- | ------------ | -------------------------------- |
| <username>.github.io |              | <username>.github.io             |
| any                  | gh-pages分支 | <username>.github.io/projectname |
| any                  | main/docs    | <username>.github.io/projectname |
| any                  | main         | <username>.github.io/projectname |

使用私有仓库部署 Github Page 站点条件：Pro、组织账户、使用 Github Enterprise Cloud



### 自定义域名

[Github 文档#配置自定义域](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain)	|	[github 文档#配置站点](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)

| 支持的 自定义域 的类型 | 举例             | 通过DNS提供商配置    |
| ---------------------- | ---------------- | -------------------- |
| 顶级域                 | example.com      | A、ALIAS、ANAME 记录 |
| www子域                | www.example.com  |                      |
| 自定义子域             | xxyy.example.com | CNAME记录            |

机制：
所有项目公共自定义域，例如 xxyy.example.com/proj1、xxyy.example.com/proj2



需要注意：在DNS提供商配置了自定义域却没有将域名添加到 Github，会有安全风险





1 开启 Settings > Pages > Github Actions + Custom domain

2
添加CNAME
国际化域名(特殊字符域名)需要配置Punycode编码版本的CNAME

3
DNS提供商 (例如[阿里云](https://dns.console.aliyun.com/#/dns/setting/haruharu.top))

| 记录类型 |                  | 指向             |
| -------- | ---------------- | ---------------- |
| CNAME    | blog.example.com | <user>.github.io |

4测试：

```bash
# Bash环境
dig blog.haruharu.top
```



实现自定义域名的两种方式：

* CNAME
* settings>pages>自定义域





## app

dependabot：检查 package.json 新版本依赖创建PR

```yaml
# dependabot.yml
version: 2
updates:
- package-ecosystem: npm
  directory: "/"
  schedule:
    interval: daily
  open-pull-requests-limit: 20
```

github-pages

giscus：基于 Github Discussion 的评论区

img-bot：压缩图像



# Gitflow

[掘金教程](https://juejin.cn/post/6982166300806086692)

```bash
git flow help

git flow init  # 创建分支
git flow feature start fname  # 创建共享分支feature/fname
git flow feature finish fname  # git merge —no-ff
git flow release start 1.1.5  # hotfix的基础分支是master、其它是dev
git flow release finish 1.1.5

git checkout -b feature/fname-person  # 创建个人分支

# 写代码之前merge共享分支
git checkout feature/test
git pull origin feature/test
git checkout feature/test-username
git merge feature/test

# 为个人分支创建PR合并到共享分支
```

```bash
# git-flow 等价的 git命令

# 创建一个仓库
mkdir gitflow
cd gitflow
git init

touch README.md
# ... 编辑 README.md
git add README.md
git commit -m "add README.md"

# 创建develop长期分支
git checkout -b develop
# develop分支必须先与feature分支推到远程仓库中
git push origin develop
# 获取远程仓库的develop分支到本地develop分支
# git fetch origin develop:develop

# 在develop的基础上创建功能分支
git checkout -b feature/baseinfo develop
# ... 开发功能 feature/baseinfo
git push origin feature/baseinfo
# 合并开发好且通过测试的feature/baseinfo到develop
git checkout develop
git merge --no-ff feature/baseinfo

# 移除开发完成的功能分支feature/baseinfo
git branch -d feature/baseinfo
# 删除远程仓库的功能分支
git push origin :feature/baseinfo

# ... 继续开发合并其他分支

# 预发布
git checkout -b release/v0.1.0 develop
git commit -a -m "initial version  v0.1.0"
# ... 在release/v0.1.0上修改bugs

# 发布线上版本
git checkout master
git merge --no-ff release/v0.1.0
# commit note: initial version v0.1.1
git tag v0.1.0
git push origin master
git push origin v0.1.0

# 将release中修改过的代码合并到develop中
git checkout develop
git merge --no-ff release/v0.1.0

# 删除不需要的release分支
# 删除本地release分支
git branch -d release/v0.1.0
# 删除远程release分支
git push origin :release/v0.1.0

# 在线上发现一个Bug，需要紧急修复，一般来说，v0是还没有上线的版本
git checkout -b hotfix/v0.1.0 master
# ... 修复bugs
# 重新发布一个版本，修订号加上1
git checkout master
git merge --no-ff hotfix/v0.1.0
# commit note: bump version to v0.1.1
git tag v0.1.1
git push origin master
git push origin v0.1.1

# 将hotfix分支合并到develop中，如果此时有未发布的release分支，则合并到release分支中
git checkout develop
git merge --no-ff hotfix/v0.1.0
git push origin develop

# 删除hotfix分支
git branch -d hotfix/v0.1.0
git push origin :hotfix/v0.1.0
```



# .github

## Actions

CICD持续集成部署

[官网](https://github.com/features/actions)	|	[demo](https://github.com/ChenXiangcheng1/skills-hello-github-actions)	|	[marketplace](https://github.com/marketplace?type=actions)	|	[actions/checkout](https://github.com/actions/checkout)

```yaml
# .github/workflows/xxx.yml
name: Post welcome comment
on:
  workflow_dispatch:  # 人工触发
    inputs:
      xxYY:
        description: "desc"
        required: true
        default: "xx"
      tags:
        description: "tags desc"
  pull_request:
    types: [opened]
  schedule:  # 计时触发
    - cron: "*/15 * * * *"
  fork:
  watch:  # star
  issues:
  push:
permissions:
  pull-requests: write
jobs:  # 运行在runner server上
  build:
    name: Post welcome comment
    runs-on: ubuntu-latest
    steps:
     - name: just a simple ls command
        runs: ls -la
      - name: Checkout repository
        uses: actions/checkout@v4  # 复用 # 克隆仓库到虚拟机上
        with:  # 参数
          arg: 1
      - run: gh pr comment $PR_URL --body "Welcome to the repository!"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_URL: ${{ github.event.pull_request.html_url }}
  run1:
    needs: [build]
  run2:
    needs: [build, run1]
```



## Dependabot

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"

version: 2
updates:
  - package-ecosystem: "npm"  # pip 支持的包管理器有限
    directory: "/frontend" # Update this path if your package.json is in a different directory
    schedule:
      interval: "weekly"
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```





# 未整理

## 架构

![](https://upload-images.jianshu.io/upload_images/3670077-b97c6a8e14d1dafa.png)

工作区

暂存区(索引)

本地仓库

远程仓库

HEAD：最后一次提交，提交树的当前提交位置。
cat .git/COMMIT_EDITMSG 查看HEAD指向
git symbolic-ref HEAD
分离 HEAD 的状态：这是因为不能直接在v1标签上面做 commit。



分支反映了你**最后一次与仓库通信时**的状态



## 命令

决定使用哪个命令：

![](http://justinhileman.info/article/git-pretty/git-pretty.png)



Git 常用命令速查表：

![](https://yqfile.alicdn.com/img_6ff4f6285991c03f6436c41b896eff11.jpg)



### 查询操作

| 查看命令                                                     | 释义                 |
| ------------------------------------------------------------ | -------------------- |
| **git <command> -h (常用)**                                  | 快速参考             |
| git help <command>                                           | 参考手册             |
|                                                              |                      |
| **git status**                                               | 查看工作树状态       |
| git ls-files                                                 | 查看暂存区中文件     |
| **git log**                                                  | 查看提交记录的hash值 |
| git config --list --show-origin<br />git config --global http.proxy 'socks5://host.docker.internal:7890'<br />git config --global https.proxy 'socks5://host.docker.internal:7890'<br />git config --global --unset http.proxy<br />git config --global --unset https.proxy | 查看所有配置         |



### 修订版本

| **revision 修订版本**  | 用于表示 Git 仓库的不同版本             |
| ---------------------- | --------------------------------------- |
| 分支名                 | 例如，main                              |
| 标签名                 | 例如，v1.0                              |
| 提交ID                 | 提交记录哈希的前4个字母                 |
| [分支名,HEAD]~3        | 相对引用，向上移动3个，选择第一个parent |
| [分支名,HEAD]^2        | 相对引用，向上移动1个，选择第2个parent  |
|                        |                                         |
| **refspec**            | 表示 Git 仓库能识别的具体位置           |
| <source>:<destination> | 指定了提交记录的来源和去向              |
|                        |                                         |



### 提交树分支操作

| 分支操作                                   | 释义                                                         | 选项                                                         |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| git show-branch                            | 查看本地分支                                                 |                                                              |
| git branch <分支>                          | 创建分支                                                     |                                                              |
| git branch -f <分支> <提交>                |                                                              | -f，移动分支指针强制指向目标提交记录。                       |
| git branch -u origin/main foo              |                                                              | -u 让 `foo` 本地分支跟踪 `origin/main` 远程分支              |
| git checkout <分支>                        | 切换到目标分支，HEAD 指向(切换到)目标分支提交记录            |                                                              |
| git checkout <ref>                         | HEAD 在提交树中移动                                          |                                                              |
| git checkout -b totallyNotMain origin/main | = git branch xxx<br />git checkout xxx                       | -b 创建并切换到一个名为 `totallyNotMain` 的分支，它跟踪远程分支 `origin/main` |
| git checkout --orphan <branch_name>        |                                                              |                                                              |
| git merge <分支>                           | 把目标分支合并到当前分支，当前分支指向新的提交记录<br />若目标分支继承当前分支，则快速合并 fast-forward 只用移动当前分支指针 |                                                              |
| git rebase <分支>                          | 将当前提交合并到目标分支上                                   |                                                              |
|                                            |                                                              |                                                              |



### 提交树提交记录操作

| 命令关于提交记录       | 释义                                                         |
| ---------------------- | ------------------------------------------------------------ |
| **撤销提交**           |                                                              |
| git reset HEAD~1       | 回退本地提交，把分支记录回退几个提交记录                     |
| git revert <提交>      | 撤销远程提交，创建一个新提交用于撤销目标提交记录             |
|                        |                                                              |
| **整理提交记录**       |                                                              |
| git cherry-pick <提交> | 将提交拷贝到 HEAD 下面                                       |
| git rebase -i <提交>   | -i 整理提交，拷贝当前分支到目标提交下面，(打开新交互窗口，可以修改 [HEAD, 目标提交) 提交顺序、删除提交、合并提交) |
|                        |                                                              |
| **锚点**               |                                                              |
| git tag v1 <提交>      | 打标签                                                       |
| git describe <分支>    | v1_2_gC2                                                     |
| git describe <ref>     | 输出最近标签 <tag>_<numCommits>\_g<hash><br />`tag` 表示的是离 ref最近的tag标签， `numCommits` 是表示这个 ref 与 tag 相差有多少个提交记录， `hash` 表示的是你所给定的 ref 所表示的提交记录哈希值的前几位。 |
|                        |                                                              |







### 远程仓库操作

github仓库是git远程仓库的实例

remote 分支(如，origin) 用于跟踪远程仓库中的分支

| 命令            | 释义 |
| --------------- | ---- |
| git remote add  |      |
| git remote show |      |
|                 |      |



### 同步操作

| stash存档库 | workspace工作区                                              | stage暂存区index                                    | local repository本地仓库  main 本地分支                      | local repository本地仓库  origin/main远程分支                | upstream repository远程仓库                                  |
| ----------- | ------------------------------------------------------------ | --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|             | **git add <file>**                                           | 非 gitignore 文件添加到暂存区，添加索引，会根据匹配 |                                                              |                                                              |                                                              |
|             | 将该文件从暂存区中移除，取消索引，并将暂存区中的文件恢复到与revision提交相同内容，本地文件不变。 | **git reset <revision> <file>**                     |                                                              |                                                              |                                                              |
|             | = git reset HEAD <file>                                      | **git restore --staged <file>**                     |                                                              |                                                              |                                                              |
|             | 会将工作文件的修改撤销到最近一次 git add 处                  | **git restore <file>**                              |                                                              |                                                              |                                                              |
|             |                                                              | **git commit -m "xx"**                              | 将暂存区的所有内容添加到本地仓库                             |                                                              |                                                              |
|             |                                                              | **git commit --amend**                              | 打开文件，修改 HEAD 提交                                     |                                                              |                                                              |
|             | 并在工作区迁出main分支的HEAD版本                             |                                                     |                                                              | fetch                                                        | **git clone -b dev --single-branch <repo>**                  |
|             |                                                              |                                                     |                                                              | 抓取更新(更新本地仓库中的远程分支)，通过http或git协议从远程仓库下载本地仓库中缺失的所有提交记录到各个远程分支<br />更新远程分支(如 `origin/main`)(不会更新你的本地的非远程分支，只是下载提交记录，这样你就可以对远程分支进行检查或者合并了) | **git fetch**                                                |
|             |                                                              |                                                     |                                                              | 到远程仓库的 `place` 分支上，获取所有本地不存在的提交，放到本地的 `origin/place` 上。<br />更新远程分支(如 `origin/main`)(不会更新你的本地的非远程分支，只是下载提交记录，这样你就可以对远程分支进行检查或者合并了) | **git fetch <remote> <place>**                               |
|             |                                                              |                                                     | 更新本地 destination 分支，没有更新远程分支 origin/source 和本地分支 source<br />若本地分支不存在则创建 |                                                              | **git fetch <remote> <source>:<destination>**                |
|             |                                                              |                                                     | 创建新`destination`分支                                      |                                                              | git fetch <remote> :<destination>                            |
|             |                                                              |                                                     | 并将远程分支的最新提交记录合并到本地分支<br />不会覆盖当前未提交的修改 | pull = fetch + stash + cherry-pick/merge origin/main + stash pop | **git pull**                                                 |
|             |                                                              |                                                     | 并将本地分支 rebase 到远程分支的最新提交记录<br />不会覆盖当前未提交的修改 | git pull --rebase = fetch + stash + rebase origin/main + stash pop | **git pull --rebase(偏离的工作常用)**                        |
|             |                                                              |                                                     | = git fetch <remote> <place>; git merge <remote>/<place>     |                                                              | **git pull <remote> <place>**                                |
|             |                                                              |                                                     | = git fetch <remote>  <source>:<destination>; git merge <destination><br />若目的分支不存在则在远程创建分支再被合并 |                                                              | git pull <remote>  <source>:<destination>                    |
|             |                                                              |                                                     |                                                              | **git push <remote> <place>**                                | 当本地提交不是基于远程最新提交点时，需要本地先合并远程最新提交点(解决冲突)，然后再提交<br />切到本地仓库中的`place`分支，获取所有的提交，再到远程仓库`remote`中找到`place`分支，将远程仓库中没有的提交记录都添加上去<br />没有指定remote place参数，则通过分支的属性操作 |
|             |                                                              |                                                     |                                                              | git push origin <source>:<destination>                       | <source>:<destination>指定了提交记录的来源和去向，若目的分支不存在则在远程创建分支 |
|             |                                                              |                                                     |                                                              | git push origin :<destination>                               | 删除了远程仓库中的 `destination` 分支                        |



## 遇到的问题

**1**

Git Bash 内中文乱码

因为：windows 早期支持ANSI，对UTF-8支持不完善

1 设置环境变量 LANG=en_US.UTF-8

2 也可以设置option中Text的Character set为gbk

3 也可以使用winpty



**2**

Tabby 终端中 Git Bash Shell 中的 VIM 键盘乱码

因为：辣鸡 Tabby

1 重启 Tabby



**3**

warning: LF will be replaced by CRLF the next time Git touches it

CRLF(Carriage Return Line Feed) 回车指的是将打印头移回到当前行的起始位置，换行指的是将纸张向下移动一行。
Windows中CRLF表示行结束
Linux和MacOS中LF表示结束

https://stackoverflow.com/questions/17628305/windows-git-warning-lf-will-be-replaced-by-crlf-is-that-warning-tail-backwar

https://www.google.com/search?q=git+warning%3A+LF+will+be+replaced+by+CRLF+the+next+time+Git+touches+it&oq=git+warning%3A+LF+will+be+replaced+by+CRLF+the+next+time+Git+touches+it&aqs=chrome..69i57j69i64.9906j0j7&sourceid=chrome&ie=UTF-8



**4**

Git Bash不支持本机交互式应用程序，即其中不能使用上下移动选择

1 可以使用数字代替

2 也可以使用winpty，如winpty vue.cmd create hello-world（vue.cmd在E:\nodejs中）



**5**

```bash
$ docker container run  -d  busybox /bin/sh
11c1416fa0740b92018ea8c1d44c6d257b8d297b37ca5f43ddeb2f2725f4ca30
docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "E:/Applications/Scoop/apps/git/2.42.0/usr/bin/sh": stat E:/Applications/Scoop/apps/git/2.42.0/usr/bin/sh: no such file or directory: unknown.
```

错误：/ 表示 git/usr 的相对路径
解决：使用 // 表示根目录开始的绝对路径



# 未整理

## 命令

### 操作暂存区

. 全匹配

git init 初始化暂存区和本地仓库

git add target 添加到暂存区，会根据.gitignore匹配

git ls-files 查看暂存区中有哪些文件

git rm -r --cached target 删除暂存区文件

### 操作本地仓库

git commit -m "update .gitignore" 将暂存区的所有内容添加到本地仓库

### 操作远程仓库

到本地仓库目录下，git remote add 名字 地址，你的本地仓库中添加一个名为 "origin" 的远程仓库，并将它与 GitHub 上的仓库关联起来。

git push 名字 分支 将本地仓库的所有内容添加到远程仓库







文档：file:///E:/Develop/Git/mingw64/share/doc/git-doc/



修改并提交代码

    首先使用git clone后面跟https方式 把仓库克隆下来，然后提交代码(clone后面只能跟https的URL)
    提交更改
提交本地代码到远程仓库步骤
git init 生成本地git仓库(有.git文件)
git add . 进行一次提交，提交到暂存区，把本地仓库的文件上传到缓存。
git status	查看当前库的状态，如果是红色说明文件没有在远程git管理中，需要git add . 和git commit、push
git commit -m 'xx注释update token and https' 把add进去的东西，提交到到本地的.git仓库，把第一步上传到缓存的东西上传到本地仓库，其中'xx'是操作标识，内容随便填，方便用户观看。
git push 推到远程仓库
git push origin master  把本地仓库的文件上传到远程仓库。



把本地仓库项目直接添加到远程仓库
git remote add origin https://github.com/cxccoder/empty.git
git remote add origin 地址 (remote远程的意思)
	如果fatal(致命): remote origin already exists.，使用git remote rm origin
git pull --rebase(回扣) origin master  把远程库中的更新合并到本地库中，–rebase的作用是取消掉本地库中刚刚的commit，并把他们接到更新后的版本库之中。
	git pull命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。它的完整格式稍稍有点复杂。
	git pull <远程主机名> <远程分支名>:<本地分支名>

	如果有未提交的更改，是不能git pull的
	git stash #可用来暂存当前正在进行的工作
	git stash pop #从Git栈中读取最近一次保存的内容
git push -u origin master （远程仓库空的才可以直接使用，有东西要先用上一条命令）

1，git远程添加源
2，git拉出回扣源的分支
3，git推入源的分支

git clone 地址
把项目中的东西拷贝到git下载下来的本地仓库，不需要那些不需要版本管理的东西 如node-module、.git

# Git



## 命令

命令行说明格式 | 释义
-- | --
 [] | 表示其中的元素可选择一个或多个，也可以不选
 {}，斜体 | 必须在其中选择一个
 <> | 表示其中的元素至少选择一个
 \| | 管道符号，含义是“或者”（不可同时选择两个）
 ... | 省略号，表示前述元素可以在命令行中多次重复

Git Bash 命令 | 释义
-- | --
git --version | 查看git版本
git config --global user.name "……" | 配置全局的用户名
git config --global user.email "……" | 配置全局的用户邮件
git config --list | 查看配置列表
git [--help] <command> [<args>] | 查看Manual命令手册



# TODO

git tag

git show
