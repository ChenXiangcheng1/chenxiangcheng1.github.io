---
title: Linux
date: 2023-05-01
update: 2023-07-17

categories:
  - Linux
keywords: Linux; Linux体系结构
description: 主要是Linux基础操作
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_04_30_21_35.png
---



# Linux主要考点

## 体系结构

![image-20230430213547119](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_04_30_21_35.png)

**内存地址空间的划分：**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_17_14_00.png" alt="image-20230517140049591" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_17_14_01.png" alt="image-20230517140127794" style="zoom: 45%;" />

**用户态和内核态：**

在Linux操作系统中，内核态和用户态是两种不同的权限级别，它们用于区分操作系统内核的执行环境和用户程序的执行环境。

* **用户态**（User Mode）：

  * 当用户程序运行时，它处于用户态。在用户态下，程序只能**访问受限资源和执行受限操作**。

  * 用户态的程序**不能直接访问操作系统内核的功能和硬件资源**，例如直接访问硬件设备、修改内核数据结构等。

  * 用户态下的程序**执行速度相对较快**，因为不需要处理特权指令和保护操作系统资源。

* **内核态**（Kernel Mode）：

  * 内核态是操作系统内核执行时的特权级别。在内核态下，操作系统具有对系统硬件和资源的完全控制权限。

  * 内核态的程序可以执行**特权指令**、直接访问硬件设备和修改内核数据结构。

  * 内核态下的程序执行速度相对较慢，因为需要处理特权指令和进行安全检查。

**用户态和内核态的转换过程：**

* 用户态到内核态的转换（触发系统调用）：
  1. 用户程序需要执行一些只有操作系统内核才能提供的特权操作，例如**文件操作、网络通信、进程管理**等。
  2. 用户程序通过**系统调用**（System Call）**将控制权转移到内核态**。
     系统调用是一种特殊的函数调用，它将用户程序的执行从用户态切换到内核态，以便访问操作系统内核提供的服务或资源。
  3. 在系统调用发生时，处理器切换到内核态，并将用户程序的**上下文保存在内核堆栈**中，然后执行内核中相应的系统调用处理函数。
  4. 系统调用执行完毕后，处理器从内核态切换回用户态，并**将结果返回给用户程序**。

* 内核态到用户态的转换：
  1. 当内核态中的操作执行完毕后，处理器需要将控制权返回给用户程序，使其继续执行。
  2. 处理器将用户程序的上下文从内核堆栈中恢复，包括程序计数器、寄存器和其他状态信息。
  3. 处理器从内核态切换回用户态，并将**控制权**交给用户程序，使其继续在用户态下执行。

这种内核态和用户态之间的转换确保了操作系统内核的安全性和稳定性。内核态的权限受到限制，用户态的程序无法直接访问和修改核心资源，必须通过系统调用向内核请求服务。这种机制**确保了操作系统的稳定性和安全性**，并提供了对硬件资源的受控访问。

## 机制和策略

机制是必须的

策略是不同的可选优化

## 系统调用

### Fork()

**写时复制策略** cow(Copy-on-Write)：如果有多个调用者同时要求相同资源（如内存或磁盘上的数据存储)，他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制份专用副本给该调用者，而其他调用者所见到的最初的资源仍然保持不变，这个过程对其他的调用者是透明的。

写时复制策略的优点：如果调用者没有修改该资源，就不会有副本被创建，因此多个调用者只是读取操作时可以共享同一份资源。

写时复制策略的处理过程中：需要维持一个给读请求使用的指针，并在新数据写入完成后更新这个指针，以提升读写并发能力。因此，cow也间接提供了数据更新过程中的原子性。在保证数据的完整性同时，还保证了一定的读写效率。

Linux的Fork系统调用：创建子进程，使用写时复制策略。内核只为子进程创建虚拟空间，父子两进程使用相同的物理空间。只有父子进程发生更改时才会为子进程分配独立的物理空间。

### write() 的缓冲区

write()：将数据从应用程序写入到FD文件描述符所表示的文件或设备

为了提高文件写入效率，在现代操作系统中，当用户调用 write() 函数，将一些数据写入文件时，操作系统通常会将数据暂存到一个内存缓冲区里，当缓冲区的空间被填满或超过了指定阈值后，才真正将缓冲区的数据写入到磁盘里。

这样的操作虽然提高了效率，但也为数据写入带来了安全问题：如果计算机停机，内存缓冲区中的数据会丢失。为此，系统提供了fsync()、fdatasync() 同步函数，可以强制操作系统立刻将缓冲区中的数据写入到硬盘里，从而确保写入数据的安全性

## Shell命令

[Linux命令在线查询网站](https://www.linuxcool.com/)

[]可选、{}必选一个、<>必选

### 如何查找特定的文件find

面试里常用的方式
`find ~ -name "target3.java”`：精确查找文件
`find ~ -name "target*"`：模糊查找文件
`find ~ -iname "target*”`：不区分文件名大小写去查找文件
`man find`：更多关于find指令的使用说明

### 检索文件内容grep |

面试里常用的方式，用管道操作符组合使用 grep全局搜索正则打印

`grep 'xxx\[yyy\]' zzz.log`：查找某个字段，并将相关行展示出来
`grep -o 'xxx\[[0-9a-z]*\]'`：正则匹配
`grep -v 'grep'`：不匹配

### 对日志内容做统计awk

面试里常用的方式

`awk '(print $1,$4)' netstat.txt bbb.txt`：筛选文件内容某些列数据
`awk '$1=="tcp"&&$2==1 {print $0)'netstat.txt`：依据条件筛选文件内容某些列数据
`awk '(enginearr[$1]++)END(for(i in enginearr)print i"\t"enginearr[i])`：对内容逐行统计操作并列出对应的统计结果
`grep 'xx' yy.log | grep -o 'xx' | awk '{arr[$1]++}END{for(i in arr)print i "\t" arr[i]}'`：grep过滤给awk统计处理，打印相关字符串和出现的次数

### 批量替换文件内容sed

面试里常用的方式 sed流式编辑器

`sed -i's/^Str/String/'replacejava`： 逐行将Str打头字符串替换成String
`sed -i's/\.$/\;/'replace.java`：逐行将 "." 结尾的字符串替换成 ";"
`sed -i's/Jack/me/g'replace.java`：逐行将全部Jack替换成me
`sed -i '/^ *$/d' filename`：删除空行

# 我的1

## 轻量级线程的实现

**在传统的多线程模型中**，线程的创建、销毁、切换等操作都由操作系统内核来管理，每个线程都有自己的内核级线程（Kernel-Level Thread）和内核级上下文。这种模型的缺点是创建和销毁线程的开销较大，并且线程的调度和切换需要频繁地切换到内核态，导致性能下降。
**而在Linux中，轻量级线程的切换过程是在用户空间中完成的，不需要涉及操作系统内核的介入，这样就避免了频繁地切换到内核态，从而提高线程的创建、销毁和切换的效率，实现轻量级的线程管理。**

轻量级线程是**在用户空间**中实现的，主要基于以下两个关键技术：

1. **用户级线程库**（User-Level Thread Library）：用户级线程库是一个在用户空间中实现的线程管理库，它负责管理轻量级线程的创建、销毁、调度和切换等操作。用户级线程库提供了一组API，供开发人员在用户程序中使用，以便创建、控制和同步轻量级线程。
2. 协作式调度（Cooperative Scheduling）：轻量级线程采用协作式调度方式，也称为合作式调度。在协作式调度中，线程主动释放CPU控制权，让其他线程执行，而不是由操作系统强制进行调度。线程在适当的时机显式地主动让出CPU，通过调用特定的函数或指令，通常称为"yield"或"yielding"操作。

用户级线程的实现原理如下：

1. 用户级线程的创建和管理完全由用户程序或用户级线程库来负责，不依赖于操作系统内核。
2. 用户级线程库在用户空间中维护线程的管理数据结构，包括线程的状态、调度信息等。
3. 用户级线程库提供了一套API，允许用户程序在用户空间中创建、销毁和切换线程，而不需要系统调用和内核介入。
4. 线程的切换是在用户态下进行的，不涉及内核态和内核调度器的介入。线程的上下文切换仅涉及用户级线程库的管理，无需切换内核级线程的上下文。
5. 用户级线程库根据线程的调度策略，决定在何时切换线程的执行，以实现线程的并发和调度。

用户级线程的优势在于轻量级和灵活性，它可以在用户空间中自定义线程的调度策略、同步机制和资源管理。然而，由于用户级线程是在用户空间中实现的，它无法利用多核处理器的并行性，因为线程的调度和切换仅在单个核心上进行。此外，用户级线程无法直接利用操作系统提供的多线程特性，例如多核调度、多线程同步原语等。

因此，在实际开发中，需要根据具体的应用场景和需求来选择使用用户级线程还是内核级线程。用户级线程适用于轻量级的并发任务和对线程管理具有高度控制需求的应用，而内核级线程适用于需要利用多核处理器的并行计算和利用操作系统提供的丰富多线程特性的应用。

## Shell命令按功能分（基础）

todo：有空学习 <https://www.zhihu.com/question/419924138>

遵循POSIX规范

### 测试端口连通性命令(常用)

```bash
nmap -p 端口号 ip地址  # 端口扫描(正确扫描到也可能会被对方防火墙拦截)  # 最该用
telnet ip地址 端口号  # 测试TCP/IP连接
nc -vz ip地址 端口号  # 创建TCP/UDP连接
traceroute ip地址  # 查路由器问题
```

### 基本操作命令

| 命令       | 释义                                                         | 选项 |
| ---------- | ------------------------------------------------------------ | ---- |
| ls         | 展示当前目录下的文件与目录.并根据颜色区分类型.( –a显示隐藏文件、-i显示inode ) |      |
| dir        | 等同于ls功能,但是不根据颜色区分类型.                         |      |
| ll         | 展示当前目录下的文件或目录 并附加显示详细信息.               |      |
| pwd        | 查看当前目录.                                                |      |
| stat a.txt | 显示文件的详细信息.                                          |      |
| date       | 查看当前服务器时间.                                          |      |
| cat a.txt  | 查看文件内部信息.                                            |      |
|            |                                                              |      |

### 文件和文件夹操作

| 命令                                  | 释义                                                         | 选项 |
| ------------------------------------- | ------------------------------------------------------------ | ---- |
| touch test.sh                         | 创建空文件.                                                  |      |
| mkdir myfloder                        | 创建空目录.  <br />mkdir –p myfloder:  用于创建多级目录文件,如果已经存在,也不报错提示. <br />mkdir无法创建多层目录,所以可以用 : mkdir –p a/b/c ) |      |
| rm  -r  my                            | 中间携带的-r参数专用于删除目录(也可: rm  文件直接删除. ).    |      |
| alias                                 | 可查看当前常用命令的设置的别名含义.                          |      |
| unalias rm                            | 可解除指定命令的别名含义.                                    |      |
| mv  test  test1                       | 可重命名文件.  (mv test .. :移动文件到上一层目录. )          |      |
| rename  部分字符  替换字符  原名称[?] | 将实现多个 文件名批量处理 , 原名称中前缀字符替换为新名称.(补充,?占位符表示单个字符,*表示多个字符.) |      |
| cd ~                                  | 回到当前用户的主目录.                                        |      |
| su -                                  | 不指定名称默认用户切换到root下.(su - user1 切换到普通用户目录下. 如果以上切换不加 – 那么切换账号后,停留在当前目录下.) |      |
| exit                                  | 退出切换后的进入的账号状态. 回到之前的登录账号状态.          |      |
| cp a.txt  b.txt                       | 复制一份.(cp a.txt  myfloder 复制到文件夹内.  cp –r my1 my2  : 复制目录.)<br />target目录要存在，若不存在需mkdir |      |
| scp -r                                | 基于ssh远程连接协议实现复制. 一般需要密码.<br/>本地到远程 : scp /home/test/\*abc        用户名@10.1.1.2:/home/root<br/>远程到本地 : scp 用户名@10.1.1.2:/home/root/\*abc       /home/test<br/>远程到远程 : scp 用户名1@10.1.1.2:/home/root/*abc       用户名2@10.1.1.3:/home/root<br/>（备注:如果复制目录: scp –r 源地址  目标地址  如果显示进度: scp –v xx xx 不显示进度q）<br /><br />可以配置SHH |      |
| tree /root                            | 观察目录树状结构                                             |      |

### 链接

| 命令                   | 释义                                 | 选项 |
| ---------------------- | ------------------------------------ | ---- |
| ln a.txt a_link.txt    | 创建硬链接，可以识别到. cp –l 也可以 |      |
| ln –s a.txt a_link.txt | 创建软链接，cp –s 也可以             |      |
|                        |                                      |      |

硬链接：也就是一个文件。有两个链接，真实占用磁盘的两个地址,相互独立.这就是硬链接.

软链接(符号链接)：就是一个文件,只有一个链接,占用了磁盘的一个存储地址, 将为该链接地址又创建了新的访问快捷方式.这就是符号链接.

### 用户

\# root用户

$ 普通用户

| 命令                                | 释义                                  | 选项          |
| ----------------------------------- | ------------------------------------- | ------------- |
| groupadd xxgroup                    | 创建新的用户组                        | -r 系统用户组 |
| useradd  xxuser                     | 创建新的用户                          | -r 系统用户   |
| groups  user1  /  passwd  password2 | 位于root状态下，查看user1用户的所属组 |               |

```bash
cat /etc/group  # 查看组
cat /etc/passwd  # 查看用户
```

### 权限

| 命令                                   | 释义                                                         | 选项 |
| -------------------------------------- | ------------------------------------------------------------ | ---- |
| ll                                     | 查看列表文件详细信息                                         |      |
| chmod u+x a.txt<br />chmod  775  a.txt | 只有root和owner才有权限修改文件的访问权限<br />u拥有者, g所属组 , o其他用户, a所有用户，+-=授权<br />添加rwxrwxr-x权限.<br />-R 递归文件夹添加权限. |      |

ll：分为七列。

-rw - r- - r - -. 4 root root 36 12月 23 09:02 module

1. 第一列详解：-  rw-  r--  r--  

   首位置字符 **– 表示文件.  d 表示目录. l  表示链接**.
   第一个权限: 指的是**所有者权限**. 第二个权限: **所属组的权限**. 第三个权限:**其他组权限**. (补充: 权限字符含义: r读 w写 x执行 )

2. 第二列详解: 文件的链接数量

3. 第三列详解: 所属用户.

4. 第四列详解: 所属组.

5. 第五列详解:文件大小.

6. 第六列详解: 最后修改时间.

7. 第七列详解: 文件名.

### 查看文件

| 命令                    | 释义                                                         | 选项 |
| ----------------------- | ------------------------------------------------------------ | ---- |
| cat a.txt               | 查看全部内容                                                 |      |
| cat a.txt  b.txt        | 展示多个文件全部内容.                                        |      |
| cat a.txt b.txt > c.txt | 合并文件到c.txt.                                             |      |
| tac a.txt               | 按照反序展示全部内容.                                        |      |
| cat –b  a.txt           | 展示全部内容并展示行号.                                      |      |
| cat –A a.txt            | 展示内容并转义展示. 当文件中出现特殊换行空格等,会展示出来.   |      |
| more  a.txt             | 分屏查看内容. <br />空格可以切换至下一页.<br />回车可以展示下一行.<br />b返回上一页<br />q退出. |      |

### tar压缩,解压

| 命令                                                      | 释义                                                         | 选项 |
| --------------------------------------------------------- | ------------------------------------------------------------ | ---- |
| tar –zcvf a.tar.gz<br />tar –xzvf a.tar.gz -C /target_dir | vf<br />c压缩、x解压、t查看压缩包内容不解压<br />z：gzip、j：bzip压缩算法 |      |

### 查看文件大小

| 命令            | 释义                              | 选项 |
| --------------- | --------------------------------- | ---- |
| du  -ch  [dir]  | 查看对应目录及子目录所占空间大小. |      |
| du  -sh [ dir ] |                                   |      |
|                 |                                   |      |

### 程序在线安装

#### wget 下载工具

wget 不是安装方式，是一种下载工具，支持HTTP，HTTPS和FTP协议，下载完后还需要使用 rpm 或 dpkg 安装。

| 命令     | 释义     | 选项 |
| -------- | -------- | ---- |
| wget url | 下载xx源 |      |
|          |          |      |

下载工具推荐 aria2c，但 wget 也必装因为部分软件下载会依赖wget
其它curl -O

#### RPM 软件包安装方式

适用于 Redhat 系列系统 (包括 CentOS 系统)

rpm -ivh package.rpm 安装一个rpm包
rpm -ivh --nodeeps package.rpm 安装一个rpm包而忽略依赖关系警告
rpm -U package.rpm 更新一个rpm包但不改变其配置文件
rpm -F package.rpm 更新一个确定已经安装的rpm包
rpm -e package_name.rpm 删除一个rpm包
rpm -qa 显示系统中所有已经安装的rpm包
rpm -qa | grep httpd 显示所有名称中包含 "httpd" 字样的rpm包
rpm -qi package_name 获取一个已安装包的特殊信息
rpm -qg "System Environment/Daemons" 显示一个组件的rpm包
rpm -ql package_name 显示一个已经安装的rpm包提供的文件列表
rpm -qc package_name 显示一个已经安装的rpm包提供的配置文件列表
rpm -q package_name --whatrequires 显示与一个rpm包存在依赖关系的列表
rpm -q package_name --whatprovides 显示一个rpm包所占的体积
rpm -q package_name --scripts 显示在安装/删除期间所执行的脚本l
rpm -q package_name --changelog 显示一个rpm包的修改历史
rpm -qf /etc/httpd/conf/httpd.conf 确认所给的文件由哪个rpm包所提供
rpm -qp package.rpm -l 显示由一个尚未安装的rpm包提供的文件列表
rpm --import /media/cdrom/RPM-GPG-KEY 导入公钥数字证书
rpm --checksig package.rpm 确认一个rpm包的完整性
rpm -qa gpg-pubkey 确认已安装的所有rpm包的完整性
rpm -V package_name 检查文件尺寸、 许可、类型、所有者、群组、MD5检查以及最后修改时间
rpm -Va 检查系统中所有已安装的rpm包- 小心使用
rpm -Vp package.rpm 确认一个rpm包还未安装
rpm2cpio package.rpm | cpio --extract --make-directories *bin* 从一个rpm包运行可执行文件
rpm -ivh /usr/src/redhat/RPMS/`arch`/package.rpm 从一个rpm源码安装一个构建好的包
rpmbuild --rebuild package_name.src.rpm 从一个rpm源码构建一个 rpm 包

##### YUM 软件包管理器

适用于 Redhat 系列系统

| 命令                                    | 释义          | 选项 |
| --------------------------------------- | ------------- | ---- |
| yum install vim / tree / jdk / mysql …. | –y默认全部为y |      |
| yum clean all                           | 清空安装包    |      |
| yum remove tree                         | 删除程序      |      |

yum install package_name 下载并安装一个rpm包
yum localinstall package_name.rpm 将安装一个rpm包，使用你自己的软件仓库为你解决所有依赖关系
yum update package_name.rpm 更新当前系统中所有安装的rpm包
yum update package_name 更新一个rpm包
yum remove package_name 删除一个rpm包
yum list 列出当前系统中安装的所有包
yum search package_name 在rpm仓库中搜寻软件包
yum clean packages 清理rpm缓存删除下载的包
yum clean headers 删除所有头文件
yum clean all 删除所有缓存的包和头文件

#### DEB 软件包安装方式

适用于 Debian 系列系统 (包括 Ubuntu 系统)

dpkg -i package.deb 安装/更新一个 deb 包
dpkg -r package_name 从系统删除一个 deb 包
dpkg -l 显示系统中所有已经安装的 deb 包
dpkg -l | grep httpd 显示所有名称中包含 "httpd" 字样的deb包
dpkg -s package_name 获得已经安装在系统中一个特殊包的信息
dpkg -L package_name 显示系统中已经安装的一个deb包所提供的文件列表
dpkg --contents package.deb 显示尚未安装的一个包所提供的文件列表
dpkg -S /bin/ping 确认所给的文件由哪个deb包提供

##### APT 软件包管理器

适用于 Debian 系列系统 (包括 Ubuntu 系统)

apt-get install package_name 安装/更新一个 deb 包
apt-cdrom install package_name 从光盘安装/更新一个 deb 包
apt-get update 升级列表中的软件包
apt-get upgrade 升级所有已安装的软件
apt-get remove package_name 从系统删除一个deb包
apt-get check 确认依赖的软件仓库正确
apt-get clean 从下载的软件包中清理缓存
apt-cache search searched-package 返回包含所要搜索字符串的软件包名称

#### Pacman 软件包管理器

适用于 ArchLinux 系列系统

[别人的笔记](https://hustlei.github.io/2018/11/msys2-pacman.html#%E6%9C%80%E5%B8%B8%E7%94%A8%E7%9A%84pacman%E5%91%BD%E4%BB%A4%E5%B0%8F%E7%BB%93s)

[通过代理更新密钥](https://wiki.archlinuxcn.org/wiki/Pacman/%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%AD%BE%E5%90%8D)

| 命令 pacman <operation> [options] [package(s)] | 释义                                                         | 选项      | 释义                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ | --------- | ------------------------------------------------------------ |
| -S                                             | 查询同步库<br />pacman -Syyu                                 | s <regex> | 在远程仓库搜索对应的包(本地已安装的会标记)                   |
|                                                |                                                              | i         | 打印软件包信息                                               |
|                                                |                                                              | y         | 从服务器下载最新的软件包数据库(即软件列表清单core,extra,community,archlinuxcn)，单y短时内多次运行可能不同步 |
|                                                |                                                              | u         | 升级已安装的包，不包括不在软件库中的本地包                   |
|                                                |                                                              | --needed  | 不重新安装最新包                                             |
|                                                |                                                              | yy        | 强制下载最新的软件列表清单，即使已经是最新                   |
|                                                |                                                              | c         | 清理缓冲的旧软件包                                           |
| -Q                                             | 查询本地库                                                   | s <regex> | 在本地仓库搜索对应的包<br />'^pkg_name$'                     |
|                                                |                                                              | i         | 打印软件包信息                                               |
|                                                |                                                              | e         | 打印明确安装的软件包                                         |
|                                                |                                                              | u         | **打印可用更新**                                             |
|                                                |                                                              | o         | 查询该文件属于哪个包                                         |
| -R                                             | 删除                                                         | s         | 删除不需要的依赖项                                           |
|                                                |                                                              | n         | 移除配置文件                                                 |
| -U                                             | `pacman -U 本地软件包路径.pkg.tar.xz` <br />`pacman -U http://www.example.com/repo/example.pkg.tar.xz` |           |                                                              |

非官方软件包：就是软件包是开发者签名的，Arch仓库并不信任，用户需要用自己的密钥对开发者公钥签名，使arch信任公钥
需要安装 **pacman-key(用于管理GPG密钥)**，将开发者公钥导入 pacman-keyring

| 命令 pacman <operation> [options] [package(s)] | 释义                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| pacman-key --init                              | 初始化本地keyring密钥环，并生成系统主密钥，**安装时执行以后不需要执行** |
| pacman-key --populate                          | 验证主密钥，**安装时执行以后不需要执行**                     |
| pacman-key --refresh-keys                      | 刷新密钥，需要设置代理，如不用镜像最好使用德国代理。定期执行 |
| pacman-key -l                                  | 查看所有GPG密钥                                              |

##### 配置

`/etc/pacman.d/mirrorlist`

[软件包列表](https://archlinux.org/mirrorlist/) Use mirror status:按status排序

`/etc/pacman.conf`
[arch wiki#pacman](https://wiki.archlinuxcn.org/wiki/Pacman#) | [man#pacman](https://man.archlinux.org/man/pacman.conf.5) | [man#pacman](https://pacman.archlinux.page/pacman.conf.5.html)

```/etc/pacman.conf
[options]  # 通用选项
VerbosePkgLists  # 升级前对比版本
ParallelDownloads = 5  # 并行下载
# IgnoreGroup = yyy  # 不更新yyy软件包组
# IgnorePkg   = xxx  # 不更新xxx包
# Include = /etc/pacman.d/xx  # 导入配置文件xx
SigLevel    = Required DatabaseOptional  # Required必需验证签名，DatabaseOptional不强制验证数据文件.db.zst签名  # 这是默认配置
LocalFileSigLevel = Optional  # 不强制验证本地文件签名
Color  # 输出处于TTY彩色输出
CheckSpace  # install之前检查磁盘空间
CleanMethod = KeepCurrent  # KeepCurrent保留当前版本 KeepInstalled保留已安装版本

[core]
Include = /etc/pacman.d/mirrorlist
[extra]
Include = /etc/pacman.d/mirrorlist
[community]
Include = /etc/pacman.d/mirrorlist

[archlinuxcn]  # 添加archlinuxcn仓库
Server = http://repo.archlinuxcn.org/$arch
```

##### 中文社区仓库

[archlinuxcn 中文社区仓库](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)(服务器在欧洲) | [github](https://github.com/archlinuxcn/repo) | [镜像 github](https://github.com/archlinuxcn/mirrorlist-repo)
[软件仓库](http://repo.archlinuxcn.org)

使用欧洲服务器仓库：

```bash
pacman -S archlinuxcn-keyring  # 导入GPG key
```

```/etc/pacman.conf
[archlinuxcn]
https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/
```

使用镜像：

```bash
pacman -S archlinuxcn-keyring
pacman -S archlinuxcn-mirrorlist-git
```

```/etc/pacman.conf
[archlinuxcn]
Include = /etc/pacman.d/archlinuxcn-mirrorlist
```

##### Arch User Repository

[wiki#Arch User Repository](https://wiki.archlinux.org/title/Arch_User_Repository) | [AUR首页(搜索软件包)(重要)](https://aur.archlinux.org/)

[wiki#makepkg](https://wiki.archlinux.org/title/Makepkg)：通过PKGBUILD(软件包生成shell脚本)，makepkg -s生成pkgname.pkg.tar.zst，再由pacman -U安装。官方定期会从中挑选软件包进入extra仓库
makepkg配置: 如果有需要再去设置MAKEFLAGS等变量

```/etc/makepkg/.conf.d/custom.conf
# /etc/makepkg.conf.d/custom.conf
MAKEFLAGS="-j$(nproc)"  # 并行
BUILDDIR=/tmp/makepkg  # df -h /tmp, /tmp是tmpfs  # 内存文件系统
LDFLAGS="... -Wl,--separate-debug-file"  # 使用mold有的项目不能编译取消了
# LDFLAGS="... -fuse-ld=mold -Wl,--separate-debug-file"  # paru -S mold  # -Wl表示后面的选项是给链接器的  # --separate-debug-file表示生成独立的.debug调试信息文件
```

手动下载/更新软件包：pacman不支持AUR，所以需要手动升级或使用pacman封装
 1git pull <AUR_Git_URL> 或 下载快照
 2解压
 3检查PKGBUILD等文件
 4makepkg -s
 5pacman -U

pacman封装：不能root安装软件包
[wiki#AUR helpers](https://wiki.archlinux.org/title/AUR_helpers)
[github#yay](https://github.com/Jguer/yay) | [aur#yay](https://aur.archlinux.org/packages/yay)
[github#paru(chroot隔离)(推荐)](https://github.com/morganamilo/paru)

| paru(new operations)  | 释义                       | 选项 | 释义           |
| --------------------- | -------------------------- | ---- | -------------- |
| -P                    | show                       | s    | 系统软件包信息 |
| -Sss                  | 相比-Ss 多显示URL、AUR URL |      |                |
|                       |                            |      |                |
| **pacman operations** | 扩展pacman支持AUR          |      |                |

### Linux高级使用

#### 进程监控

| 命令                    | 释义                                                         | 选项                                                         |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ps [options]            | 查看活动状态的服务进程(瞬时)<br />ps –ef \|grep 进程名称  查看指定进程 | a 显示所有进程，包括其他用户的进程<br />x 显示没有控制终端的进程，即后台运行的进程<br />u 面向用户的格式，显示详细信息(进程用户,CPU占用率,内存占用)<br />–e 查看所有进程信息<br />-f 显示完整的进程信息(进程用户,进程ID,父进程ID,CPU使用率,内存使用率) |
| netstat                 | 查看当前tcp/udp等网络链接状态.                               |                                                              |
| netstat –apn \| grep 80 | 查看指定网络端口是否被进程占用.                              |                                                              |
| kill  (进程编号:pid)    | 杀死进程-自杀.                                               |                                                              |
| kill -9  (进程编号:pid) | 杀死进程-谋杀.                                               |                                                              |
| top                     | 查看当前动态进程(监控)q退出.                                 |                                                              |

#### iptables 防火墙

| 命令     | 释义                                                         | 选项                                    |
| -------- | ------------------------------------------------------------ | --------------------------------------- |
| iptables | 配置和管理iptables防火墙，可以实现对数据包的过滤、修改、转发<br />Docker就是通过iptables实现了容器和宿主机的网络隔离、请求转发 | --list 列出一条链或所有链的规则<br />-t |
|          |                                                              |                                         |
|          |                                                              |                                         |

  临时操作:

   service iptables status 查看防火墙状态.

   service iptables stop 临时关闭防火墙状态.

   service iptables start 打开防火墙.

永久操作:

   chkconfig iptables  off : 永久关闭防火墙.

   chkconfig iptables on : 永久打开防火墙.

   chkconfig –list iptables 查看防火墙状态.

   vim /etc/inittab : 也可查看服务器启动默认防火墙初始化状态.

  防火墙规则:

​        iptables –nL : 查看防火墙规则

​        修改   /etc/sysconfig/iptables 文件: 修改防火墙规则.

#### 其他功能

| 命令     | 释义         | 选项              |
| -------- | ------------ | ----------------- |
| date     | 查看当前时间 |                   |
| shutdown | 关机         | –h 12:59 定时关机 |
| halt     | 关闭系统     |                   |
| reboot   | 重启         |                   |
| exit     | 退出         |                   |

#### 网络

| 命令                              | 释义                                                         | 选项                                                         |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ip                                | 显示配置网卡参数                                             | addr 看所有网卡对应的 IP<br />route 查看路由表信息           |
| ping                              | ping 指定主机 **网络层ICMP**                                 |                                                              |
| telnet                            | 远程登入，可以测试端口的连通性 **应用层**                    |                                                              |
| arp                               | 显示和修改动态IP地址转换表(ARP缓存表) **数据链路层**         | -a 显示当前ARP缓存表项<br />-s 添加IP地址-Mac地址记录(entry) |
| tracepath [options] <destination> | 路径探测跟踪                                                 |                                                              |
| curl [options...] <url>           | client URL，发送 http 请求<br />访问 cip.cc 可用于测试是否使用代理<br/> | -X Get 指定HTTP请求方法<br />-A 指定User-Agent<br />-b 指定Cookie<br />-d 指定Post请求正文<br />-F 上传二进制文件<br />-k 跳过 SSL 检测<br />-o 将响应保存为文件<br />-x 指定代理 |

# 我的2

## 虚拟机配置

### 网络问题

#### Bridged(桥接模式) 推荐

在桥接模式下，虚拟机相当于是**局域网中的一台独立主机**，它可以访问局域网内任何一台机器
如果你想利用VMware在局域网内新建一个虚拟服务器，为局域网用户提供网络服务，就应该选择桥接模式

<table>
    <tr>
        <td><center></center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_13_12_43.png" style="zoom:80%;" /></center></td>
        <td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_13_12_43.PNG" style="zoom:50%;" /></center></td>
    </tr>
    <tr>
     <td><center><strong>用的是VMnet0，和主机网卡有关</strong></center></td>
        <td><center>我的配置</center></td>
    </tr>
</table>

要求：
IP与主机在同一个网段内
子网掩码和网关和DNS与主机相同
不过你需要多于一个的IP地址(电脑由WIFI连接)，并且需要手工(虚拟机的dhcp服务器无效)为虚拟系统配置IP地址、子网掩码，而且还要和宿主机器处于同一网段，这样虚拟系统才能和宿主机器进行通信

#### NAT(网络地址转换模式)

<table>
    <tr>
     <td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_13_12_44.png" style="zoom:67%;" /></center></td>
        <td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_13_12_44.PNG" style="zoom:50%;" /></center></td>
    </tr>
    <tr>
     <td><center><strong>用的是VMnet8(以太网适配器 VMware Network Adapter VMnet8)</strong></center></td>
        <td><center>我的配置</center></td>
    </tr>
</table>

#### Host-only(主机模式)

<table>
    <tr>
     <td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_03_13_12_45.png" style="zoom:67%;" /></center></td>
    </tr>
    <tr>
     <td><center><strong>用的是VMnet1</strong></center></td>
    </tr>
</table>

### vmware-tools问题

ls /usr/lib/systemd/system
systemctl enable run-vmblock\\x2dfuse.mount
open-vm-tools和open-vm-tools-desktop
systemctl status vmware-tools
systemctl status open-vm-tools.service

### VMware配置

进程优先级-抓取输出-高

### GNONE配置

关闭锁屏、空白屏幕、蓝牙

## WSL

[文档](https://learn.microsoft.com/zh-cn/windows/wsl/) | [**最佳实践**](https://dowww.spencerwoo.com/1-preparations/1-0-intro.html)

TODO: 有空继续看 目前看到这里 [Dev on WSL](https://dowww.spencerwoo.com/3-vscode/3-4-c_cpp.html#%E5%AE%89%E8%A3%85-c-c-%E6%8F%92%E4%BB%B6)

### 配置

[文档#wsl-config](https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config)

全局配置：`%UserProfile%/.wslconfig`(不用)
单发行配置：`/etc/wsl.conf`

存储位置：`E:\Applications\Scoop\persist\archwsl\data\ext4.vhdx`，240525目前占用5.5G  20250215:11G

```bash
# 优化 WSL 2 虚拟磁盘占用空间
Optimize-VHD -Path E:/Applications/Scoop/persist/archwsl/data/ext4.vhdx -Mode Full
```

#### .wslconfig

windows version >= 19041

```.wslconfig
[wsl2]  # path必须
networkingMode=NAT  
# NAT(推荐): 虚拟机和宿主机不在同一个子网, 虚拟机流量被宿主机代理, 此模式宿主机可以访问虚拟机端口, 但是docker容器内部无法访问宿主机端口! 
# mirrored: 镜像网络模式, 虚拟机和宿主机在同一个子网, 共享网络接口(同localhost), 不需要这个模式

# nestedVirtualization = true  # win11支持
dnsProxy=true  # DNS使用宿主机NAT  # 需要networkingMode=NAT
firewall=true  # windows防火墙作用域WSL流量
dnsTunneling=true  # DNS请求代理到windows
autoProxy=true  # WSL使用windows的HTTP代理信息
defaultVhdSize=1099511627776  # 虚拟硬盘VHD大小, 1TB

[experimental]
bestEffortDnsParsing=true  # 需要dnsTunneling=true
```

#### wsl.conf

```/etc/wsl.conf
[automount]
root = /mnt/
enabled = true
options = "metadata"
mountFsTab = true
# 问题：能win目录内ls背景色问题
# 原因：挂载win权限问题
# 解决：
# options = "metadata,umask=22,fmask=111"  # 但是不推荐，建议通过 ~/.zshrc 解决

[network]
hostname = "WSL-ARCH"
generateHosts = true
generateResolvConf = true

[interop]
enabled = true
appendWindowsPath = true

[user]
default = nemesis

[boot]
systemd=true
# command="echo 'hello wsl'"  # win11
```

### 网络互访

WSL IP：**127.0.0.1**、`cat /etc/resolv.conf`内的固定IP、wsl-hostname.local

宿主机IP：**主机名**

```bash
ip route | grep default | awk '{print $3}'
cat /etc/resolv.conf | grep nameserver | awk '{print $2}'
```

```bash
>ip route  // 显示路由表信息
default via 172.26.128.1 dev eth0  // 表示默认路由是宿主机Windows主机的IP地址172.26.128.1(同cat /etc/resolv.conf)
172.26.128.0/20 dev eth0 proto kernel scope link src 172.26.135.176  // 是WSL的IP地址172.26.135.176
```

### 文件互访

磁盘：/dev/sda
分区：/dev/sda1

```bash
# WSL环境执行Win程序
explorer.exe .
cat pacman.conf | clip.exe  # 复制WSL内容到Win，好用

#Win环境执行WSL命令
wsl ls 
```

### 远程开发

[文档](https://www.jetbrains.com/help/idea/2023.2/remote.html)

## 性能优化

[archlinux wiki#性能优化](https://wiki.archlinuxcn.org/wiki/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96)

## 软件

| 我安装的  | 释义                                                         |
| --------- | ------------------------------------------------------------ |
| git       |                                                              |
| openssh   | 实现了应用层的 SSH 协议，能用于免密登录                      |
| sudo      |                                                              |
| neofetch  |                                                              |
|           |                                                              |
| grub      |                                                              |
| wget      |                                                              |
| aria2c    |                                                              |
| zsh       |                                                              |
| oh my zsh |                                                              |
| unzip     |                                                              |
| nmap      | 是 netcat(nc) 的高级替代。可以使用 ncat 命令，用于配置 SSH 通过代理访问目标主机 |
| neovim    |                                                              |
| inetutils | 含telnet用于测试tcp                                          |

### systemd

systemd：是现代Linux中的init system、进程监督(服务管理)工具(PID 1 用户空间的第一个进程)

[archwiki#systemd](https://wiki.archlinuxcn.org/wiki/Systemd)

```bash
systemctl list-unit-files --help

# unit类型
unit
automount
path
socket
timer
```

#### cron

cron：定时任务工具，有daemon，cronie内含crond、anacron

[archwiki#cron](https://wiki.archlinux.org/title/Cron)

```bash
pacman -S cronie

crontab -l  # list
crontab <file>  # new
crontab -r  # remove
crontab -d  # edit
```

```/etc/crontab
# /etc/crontab

# [s] minute h d month w cmd [y]
*/10 * * *  root  sleep 1;/usr/sbin/anacron


# 通配符
# 由于月星期可能0或1开始，推荐使用JUL、MON英文替代
# 日：L表示月最后一天
# 星：W表示工作日

# * 每一
# , 并
# - []
# 步长/ 除  # */2等价于0/2  # 遇到不能被整除的无法实现每隔x时长执行一次
```

anacron：争对非持续运行系统(会记录任务的最后执行时间)，无daemon

#### systemd.timer

```bash
systemctl list-timers --all
systemctl daemon-reload
systemctl enable demo.timer
```

全局（所有用户适用）：`/etc/systemd/system/` ~~`/usr/lib/systemd/system/`~~
用户级别：`~/.config/systemd/user/`

```/etc/systemd/system/demo.timer
# /etc/systemd/system/demo.timer
# doc: https://man.archlinux.org/man/systemd.timer.5
[Unit]
Description=Timer to run demo.service every week.

[Timer]
# timer doc: https://man.archlinux.org/man/systemd.timer.5
# time doc: https://man.archlinux.org/man/systemd.time.7
OnCalendar=Mon,Tue *-*-01..04 12:00:00  #   # 星期 年-月-日 时:分:秒  # 支持通配符, * ..([])
OnCalendar=weekly  # 多次执行
Persistent=true  # 系统宕机重启后，补上未执行任务

AccuracySec=1us
RandomizedDelaySec=12h

[Install]
WantedBy=timers.target
```

```/etc/systemd/system/demo.service
# /etc/systemd/system/demo.service  # service文件定义任务
# archwiki#编写单元文件: https://wiki.archlinuxcn.org/wiki/Systemd#%E7%BC%96%E5%86%99%E5%8D%95%E5%85%83%E6%96%87%E4%BB%B6
[Unit]
Description=Run Python script demo.py every week
Documentation=https://xxxx
OnFailure=xxxx.service

[Service]
ExecStart=/usr/bin/python3 /path/to/demo.py
Environment=XXX=YYY

Type=oneshot
WorkingDirectory=/path/to/
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### pdsh

pdsh(Parallel Distributed Shell)

[github](https://github.com/chaos/pdsh)

```bash
pdsh -R ssh -w host_name1,host_name2,host_name3 uptime
pdsh -R ssh -w host[1] uptime
```

### nvim

```bash
export EDITOR=nvim
```

### sudo

使用

```bash
sudo [command]
```

配置

```bash
export EDITOR=nvim  # 默认是vi，可以设置成vim、nvim  # export该shell临时生效
EDITOR=nvim visudo  # 环境变量在该命令临时生效
```

```bash
## sudoers file.
##
## This file MUST be edited with the 'visudo' command as root.
## Failure to use 'visudo' may result in syntax or file permission errors
## that prevent sudo from running.
##
## See the sudoers man page for the details on how to write a sudoers file.

## User privilege specification用户权限说明
root ALL=(ALL:ALL) ALL  # moren 

## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL:ALL) ALL  # wheel、sudo组是被分配sudo权限的常用组
# %sudo ALL=(ALL:ALL) ALL  # <用户名> <主机名>=(\<运行用户:运行组>) <命令列表>

## Same thing without a password
# %wheel ALL=(ALL:ALL) NOPASSWD: ALL

# Defaults targetpw
# ALL ALL=(ALL:ALL) ALL  # WARNING: only use this together with 'Defaults targetpw'

## Read drop-in files from /etc/sudoers.d
@includedir /etc/sudoers.d
```

### OpenSSH

OpenSSH 软件是SSH协议的实现，用于提供加密的通信会话

[arch wiki#openssh](https://wiki.archlinuxcn.org/wiki/OpenSSH)

#### 服务端

配置：
`/etc/ssh/sshd_config.d/*.conf`， 配置好后使用 `sshd -t` 检查配置
`~/.ssh/authorized_keys` 保存客户端公钥，可参考 Network.md#SSH

```/etc/ssh/sshd_config

```

使用

```bash
ssh-keygen -A
```

### 网络管理器

[网络管理器](https://wiki.archlinuxcn.org/wiki/%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE#%E7%BD%91%E7%BB%9C%E7%AE%A1%E7%90%86%E5%99%A8)

NetworkManager：支持 GUI

#### systemd-networkd

```bash
systemctl enable systemd-resolved
```

### 文件系统

>设备：
>
>字符设备：字节流，如键盘鼠标、串口
>
>块设备：数据块，如硬盘、USB存储

|                      | ext4                                            | vfat                                             | XFS                  | Btrfs(推荐)                                                  | openZFS                                |
| -------------------- | ----------------------------------------------- | ------------------------------------------------ | -------------------- | ------------------------------------------------------------ | -------------------------------------- |
| **磁盘空间分配机制** | 日志文件系统Journaling<br />类似关系型数据库log | 文件分配表FAT(File Allocation Table)<br />索引表 | 日志Journaling       | CoW<br />Copy on Write 修改不就地覆盖数据而是写入新位置，之后更新文件指向新位置 (复制 更新 替换指针) (对数据库 日志文件可关闭)<br />目的：提高数据的一致性和可靠性 | CoW、存储池(整合了逻辑卷管理+文件系统) |
| 透明压缩             | 不支持                                          | 不支持                                           | 不支持               | 支持多种                                                     | 支持多种                               |
| 快照                 | 不支持                                          | 不支持                                           | 不支持               | 支持                                                         | 支持                                   |
| 碎片整理             | 需要定期                                        | 需要定期                                         | 需要定期             | 不需要，COW替换比原地修改减少了磁盘碎片                      | 不需要                                 |
| 加密分区             |                                                 |                                                  |                      | 不支持                                                       |                                        |
| 主要应用场景         | 大多数Linux发行版的默认、资源占用低             | U盘，FAT32最大4GB                                | 文件服务器、io性能优 | 高级功能丰富                                                 | 高级功能丰富、吃内存                   |

#### btrfs

[官方文档](https://btrfs.readthedocs.io/en/latest/) | [arch wiki#Btrfs](https://wiki.archlinuxcn.org/wiki/Btrfs)
[btrfs挂载选项](https://man.archlinux.org/man/btrfs.5#MOUNT_OPTIONS)
GRUB是支持btrfs的，其他引导加载程序不清楚

```bash
pacman -S btrfs-progs  # btrfs工具软件包
```

|                   | 命令                                                         | 释义                                                         |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 创建文件系统      | mkfs.btrfs -L arch /dev/vda2                                 | 在/dev/vda2分区上**创建文件系统**，标签为arch                |
| 配置文件系统-压缩 | mount -o compress=zstd[:level] /dev/sd*XX* /mnt/yyy          | 分区挂载后就**启动压缩**<br />现存文件启动压缩 `btrfs filesystem defragment -c <zstd|lzo|zlib>[:level]` |
| 配置文件系统-子卷 | btrfs subvolume create /mnt/@root                            | 创建子卷<br />可以为不同子卷设置不同的snapper快照配置、不同的btrfs挂载选项<br />我的 @ @root @home @var @snapshots @opt |
|                   | btrfs subvolume list -p path                                 | 列出path路径下的子卷                                         |
|                   | btrfs subvolume delete /path/to/subvolume                    | 删除子卷                                                     |
|                   | mount -o subvol=@root /dev/vda2 /mnt/root                    | 挂载子卷，[btrfs挂载选项](https://man.archlinux.org/man/btrfs.5#MOUNT_OPTIONS)<br />opts=noatime,compress=zstd,x-mount.mkdir<br />noatime：禁止访问时间戳atime，减少触发CoW提升性能<br />nodatacow：新文件不启用CoW |
|                   | lsattr /mnt/.snapshots                                       | 查看是否包含C字符，是否启动了CoW特性                         |
|                   | chattr +C /mnt/var/tmp 对单文件使用<br />挂载参数nodatacow对子卷使用 | 新文件不启用CoW，在追求性能的场景使用                        |
| 使用              | btrfs filesystem df /                                        | 检查已分配空间的使用情况                                     |
|                   | btrfs filesystem defragment -r /                             | 手动整理根目录碎片                                           |
|                   | btrfs filesystem resize [+\|-\| ]size /                      | 将文件系统扩展到特定大小，已有数据大小<size<=该设备可用空间  |
|                   | 自动快照                                                     | 去使用Snapper等快照管理器                                    |

```bash
btrfs subvolume create /mnt/@snapshots
btrfs subvolume delete /.snapshots  # 删除子卷/.snapshots
btrfs subvolume list /dev/vda2  # 列出/dev/vda2分区上的所有子卷
```

```bash
opts=noatime,compress=zstd,space_cache=v2,x-mount.mkdir
mount -o $opts,subvol=@snapshots /dev/vda2 /mnt/.snapshots
```

### 快照管理器

[btrfs自动快照](https://wiki.archlinuxcn.org/wiki/Btrfs#%E8%87%AA%E5%8A%A8%E5%BF%AB%E7%85%A7)

#### snapper

snapper 是 openSUSE发行版组织维护的

软件包
snapper：快照管理工具
snap-pac：pacman扩展，在pacman更新时自动创建、删除快照。配置/etc/snap-pac.conf
grub-btrfs：在引导加载程序GRUB中添加Btrfs快照启动项，实现快照启动功能(引导进入快照)

[arch wiki#snapper](https://wiki.archlinux.org/title/Snapper) | [snapper-configs官方文档](http://snapper.io/manpages/snapper-configs.html)

配置：
`/etc/snapper/configs/xxx`

```/etc/snapper/configs/root
# limits for timeline cleanup时间线清理算法
TIMELINE_MIN_AGE="3600"  # 最小保存时间s
TIMELINE_LIMIT_HOURLY="5"  # 保留 5 个每小时快照
TIMELINE_LIMIT_DAILY="7"
TIMELINE_LIMIT_WEEKLY="0"  # 0每周快照不专门保留
TIMELINE_LIMIT_MONTHLY="2"
TIMELINE_LIMIT_YEARLY="0"
```

使用：

| snapper                                                | 释义                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| snapper -c <config-name> create-config <dir>           | # 会为挂载在/处的子卷创建配置文件/etc/snapper/configs/root<br /># 会在/.snapshots处创建子卷(推荐换成自己创建的独立子卷@snapshots)<br /># 快照目标为/ |
| snapper list-configs                                   | 列出所有以创建配置                                           |
| snapper -c root delete N                               | 删除序号N快照                                                |
| snapper -c <config-name> undochange  <snapshot-number> | 回滚到需要N快照                                              |

服务：

```bash
snapper-timeline.timer  # 自动每小时创建一个快照，默认每小时
snapper-cleanup.timer  # 默认每天运行清理

systemctl enable --now grub-btrfs.timer  # grub-btrfs提供的，用于生成快照菜单项，并不会在GRUB显示
grub-mkconfig -o /boot/grub/grub.cfg  # 更新GRUB配置，会写入快照启动项，实现快照启动功能
```

### swap的分区和缓存

[arch wiki#swap](https://wiki.archlinuxcn.org/wiki/Swap)

交换空间：
当RAM(物理内存)耗尽或休眠 (hibernate)或者睡眠(sleep)就会使用swap区域
total memory = physical + zram + disk swap

* 二级swap：开swap会影响硬盘寿命、基于磁盘
   /swap分区：
   swapfile：调整大小比 partition 方便
   解决内存小的问题
* 一级swap：基于内存
   zswap：压缩内存，必须要有物理交换设备swap作为后备存储，FIFO。 zswap 会在 zram 之前被用作 swap 缓存
   zRAM(推荐)：压缩内存，如果内存足够大可以完全使用zRAM避免使用基于磁盘的二级swap
   解决swap慢的问题

> zswap 和 zram 两者的目的相同但实现方法不同：
> zswap依靠一块压缩的**内存缓存**来工作，不需要（也不允许）过多的用户空间配置；
> 而zram内核模块可用于在内存中创建一个**压缩的块设备**。
> zswap需与swap设备配合工作，而zram则不需要。

#### zram

是一个块设备驱动程序，以内核模块的形式加载，该模块创建基于RAM的块设备
`ls /lib/modules/$(uname -r)/kernel/drivers/block/  # 查看内核是否含有zRAM模块`
[arch wiki#zram](https://wiki.archlinuxcn.org/wiki/Zram) | [内核源码#zRAM](https://github.com/torvalds/linux/blob/master/drivers/block/zram/Kconfig) | [内核文档#zram](https://www.kernel.org/doc/Documentation/admin-guide/blockdev/zram.rst) | [man#modprobe](https://man7.org/linux/man-pages/man5/modprobe.d.5.html) | [zRAM调优](https://www.cnblogs.com/linhaostudy/p/18324329)

配置

```bash
$ echo 0 > /sys/module/zswap/parameters/enabled  # 禁用zswap
$ echo zram >> /etc/modules-load.d/zram.conf  # 自动加载内核模块  # 取代手动 modprobe zram
# vim /etc/modprobe.d/zram.conf  # 配置modprobe自动加载zram的参数
$ reboot
```

##### zram-generator

zram-generator 本质是 sytemd 服务生成器，需要配合 `systemctl daemon-reload` 使用

配置

[arch wiki#zram-generator conf](https://man.archlinux.org/man/zram-generator.conf.5)

```/etc/systemd/zram-generator.conf
[zram0]  # zRAM设备  # zramN.Service
zram-size = min(ram / 2, 4096)  # MiB
compression-algorithm = zstd  # cat /sys/block/zram0/comp_algorithm 显示所有可用的压缩算法
```

使用

```bash
$ pacman -S zram-generator
$ vim /etc/systemd/zram-generator.conf
$ systemctl daemon-reload
$ systemctl enable --now systemd-zram-setup@zram0.service

$ modinfo zram
$ lsmod | grep zram  # 看模块是否加载
$ systemctl status systemd-zram-setup@zram0.service  # 看服务是否启动
$ free -m  # 查看内存swap空间状态(swap=swapfile+zram)
$ zramctl  # 查看zRAM状态
NAME       ALGORITHM DISKSIZE DATA COMPR TOTAL STREAMS MOUNTPOINT
/dev/zram0 zstd        446.5M   4K   69B   20K       2 [SWAP]
$ cat /proc/swaps  # 查看虚拟swap空间
Filename                                Type            Size            Used            Priority
/swap/swapfile                          file            8388604         0               -2
/dev/zram0                              partition       457212          0               100  # 优先级高
```

##### zramd

配置：`/etc/default/zramd`

使用：

```bash
systemctl enable --now zramd.service
```



## Linux Kernel6.16

宏内核

[Linux kernel map](https://makelinux.github.io/kernel/map/)



系统编程圣经: 

Advanced Programming in the UNIX Environment(UNIX环境高级编程)(APUE)

The Linux Programming Interface(Linux/UNIX系统编程手册)(TLPI): APUE的超集



OS可以看Barebone System



```bash
git log --oneline --no-merges v5.10..v5.11 -- io_uring/*.h
git log --oneline --no-merges v5.10..v5.11 | grep -i "io_uring"
```



[vger.kernel.org](https://subspace.kernel.org/vger.kernel.org.html)

```mail
# plain text email
io-uring+help@vger.kernel.org

io-uring+subscribe@vger.kernel.org
io-uring+subscribe-digest@vger.kernel.org
io-uring+unsubscribe@vger.kernel.org

io-uring+faq@vger.kernel.org  // frequently asked questions(faq)
```



## 内核模块

[archlinux#Kernel module](https://wiki.archlinux.org/title/Kernel_module)

模块所在目录：  
`/lib/modules/$(uname -r)/`  
`/usr/lib/modules/$(uname -r)/`  

```bash
`modprobe <module name>` 手动加载内核模块
`echo <module name> >> /etc/modules-load.d/<module name>.conf` 自动加载内核模块

`modprobe -c` 打印模块配置
`modprobe -c | grep <module_name>` 
`modprobe -c | less` 
`modprobe --show-depends module_name` 打印模块依赖项及其本身

`modinfo tcp_bbr` 显示模块信息
`lsmod` 显示当前加载的内核模块
```

### BBR

> 参考 Network.md#TCP#BBRv3 笔记

```bash
echo "tcp_bbr" >> /etc/modules-load.d/tcp_bbr.conf
echo "net.core.default_qdisc=fq" >> /etc/sysctl.d/bbr.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.d/bbr.conf
```

### tls

alias: tcp-ulp-tls(Upper Layer Protocol)、ktls

```bash

```

## 系统调用

### std.os.fcntl

fd status flag: 只能在文件打开后修改  

fd flag：
O_RDONLY：只读模式打开文件  
O_WRONLY：只写模式打开文件  
O_RDWR：读写模式打开文件  
O_CREAT：如果文件不存在，则创建文件  
O_TRUNC：截断文件  
O_APPEND：追加到文件末尾  
O_NONBLOCK：非阻塞I/O  
O_SYNC：同步写入  

```zig
const flags = try std.os.fcntl(stdout.handle, std.os.F.GETFL, 0);  
if (flags & std.os.O.APPEND != 0) {  // flags包含std.os.O.APPEND
    const new_flags = flags & ~@as(usize, std.os.O.APPEND);
    _ = try std.os.fcntl(stdout.handle, std.os.F.SETFL, new_flags);
}
```

### I/O复用并发模型

* BIO: read()/write()  

* NIO(): select()/poll()/epoll()  
只支持 network sockets 和 pipes  

* 传统AIO: io_submit()/io_getevents()  
只支持direct IO(零拷贝 使用O_DIRECT打开文件)  
不支持缓冲IO(写入page cache不保证写入磁盘)  

* 新AIO：io_uring

I/O复用是实现AIO的，而非并行I/O  

* 1单线程Accept  
非并发

* 2单线程Accept + 多线程读写任务
客户端数量:服务端线程数=1:1  

* 3单线程多路I/O复用  
非并发

* 4单线程多路I/O复用 + 多线程读写任务(worker poll)  
最高的读写并行通道为1(事件检测、任务分发是串行)
传统的工作线程模型  
SpringBoot  

* 5单线程多路I/O复用 + 多线程多路I/O复用
最高的读写并行通道为N(CPU核数)(事件检测、任务分发可并发 同一个通道的读写是串行)  
常用  

* 5.5单线程多路I/O复用 + 多进程多路I/O复用  
将accept()在子进程中执行(避免IPC)
多进程更安全稳定  

* 6单线程多路I/O复用 + 多线程多路I/O复用 + 多线程  
客户端数量:服务端线程数=1:1  
多线程可利用多核CPU，这是协程所做不到的

13非并发 但3能异步  
2每个连接创建独立线程，客户端多了后占资源太多  
4固定数量的工作线程，因为条件不阻塞accept接受请求，epoll异步使得多余任务可异步等待执行，才可以固定工作线程数量  
24事件检测、任务分发是串行(最高的读写并行通道为1)  
5事件检测在子线程中执行，一个子线程中监听多个connFD  
6客户端数量:服务端线程数=1:1  



### I/O

IO

* 同步
  * 传统方式：一个线程只能处理一个IO(等待特定IO请求)
  * I/O多路复用(空间复用 批量通知 等待多个IO请求 硬件中断 **轻!**)
    本质是事件批量通知

* 异步

| IO接口                                                       | 数据结构                                            | 一个进程所能打开的最大连接数                                 | 消息传递方式                       | fd列表的遍历速度                                             | tag                                |                                                              |
| ------------------------------------------------------------ | --------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| read()/write()                                               |                                                     |                                                              |                                    |                                                              | BIO                                |                                                              |
| select(nfds, readfds, writefds, exceptfds, timeout)          | fd_set{long int fds_bits[16]} 位图(16*64=1024)      | 单个进程所能打开的最大连接数由 FD_SETSIZE 宏定义<br />**连接数有限 小** |                                    | FD_ISSET(fd, *fdsetp) 每次都需要遍历fdsetp<br />O(all_fs)    | IO多路复用<br />同步               |                                                              |
| poll(pollfd *fds, nfds_t nfds, int timeout)                  | pollfd{int fd, short int events, short int revents} | 但**连接数无限制**                                           |                                    | 需要手动遍历`for (nfds_t i = 0; i < pfds.size(); ++i) { if pfds[i].}`<br />O(all_fd) | IO多路复用<br />同步               | **本质上与select没有区别**，仅突破fd_num限制                 |
| epoll_create1(flags) 返回的fd用于找到内核对象file再找到eventloop(创建epoll实例)<br />epoll_ctl(fd, op, 目标fd, *event) 添加EPOLL_CTL_ADD/修改EPOLL_CTL_MOD/删除EPOLL_CTL_DEL监控的文件描述符<br />epoll_wait(epfd, epoll_event, maxevents, timeout) | epoll_event{uint32_t events, epoll_data_t data}     |                                                              | 内核态将数据拷贝到空户空间的events | 直接返回就绪列表epoll_event[ret_nfds]<br />O(ready_fd)       | IO多路复用<br />同步               | 返回ready_fd<br />在epoll_ctl()会检查file_can_poll()目标fd是否支持poll操作(file_op->poll=true)，**块设备没有就绪状态概念不支持poll**<br />维护一个红黑树和就绪列表 |
| io_setup() io_submit() io_getevents()                        |                                                     |                                                              |                                    |                                                              | 传统AIO                            |                                                              |
| io_uring                                                     |                                                     |                                                              |                                    |                                                              | 异步IO(由内核态拷贝数据到用户空间) |                                                              |

> 虽然有栈协程比无栈协程更重，但是有栈能并发调用，而无栈协程复用空间，无法并发，所以在多连接需要高并发的领域有栈协程更好(快)
> 而在多连接不需要高并发的领域 epoll 更好(轻)

> 网络：
> 	file_op->poll() 
> 		1注册回调poll_wait(**是为之后的操作准备的，不影响本次处理**, 内核epoll_wait()会等待之前注册的回调被触发)
> 		2非阻塞立即返回文件是否就绪(只支持socket、pipe(FIFO))
> 		用于事件驱动的IO多路复用，**中断驱动**
> 	DPDK使用轮询
>
> 块设备：
> 	iopoll() 是 NVMe SSD的硬件**轮询**特性



* NIO(): select()/poll()/epoll()  
  只支持 network sockets 和 pipes  

* 传统AIO: io_submit()/io_getevents()  
  只支持direct IO(零拷贝 使用O_DIRECT打开文件)  
  不支持缓冲IO(写入page cache不保证写入磁盘)  

* 新AIO：io_uring

#### IO多路复用

看hello_epoll.cpp、hello_poll.cpp、hello_select.cpp



#### io_uring(universal ring)

> IO类型：
>
> * Direct I/O(基于文件 O_DIRECT 绕过page cache 需要大小对齐)：BIO效果最差
>   数据库系统通常自己管理缓存，使用Direct I/O
> * buffered I/O：POSIX效果最差(hot cache)
> * socket I/O
>
> 
>
> Linux I/O
>
> * BIO：read()/write()
>   如果数据在page cache是不会阻塞的
> * NIO：select()/poll()/epoll()
>   只支持 network sockets 和 pipes  
>   * POSIX AIO(线程池): aio_read/aio_write()
>     伪异步
>   * Linux AIO：
>     两个接口: 将IO请求提交到内核、从内核接收完成事件
>     接口不容易扩展只支持Direct I/O
>     非确定性行为(可能阻塞)
>   * io_uring(很全面)
>     共享buffer**减少系统调用开销**
>     批处理、易扩展能重写大多数系统调用
>     峰值170w IOPS(4KB Block)  



核心：

每个 io_uring 实例都有两个用户空间和内核空间共享的 ring buffer 环形缓冲区
SQ(submission queue提交队列)、CQ(completion queue完成队列)，E(entries)  
该队列选择基于数组，因为连续可以**内存映射**到用户空间

> io_uring makes processing async I/O go brrrr by keeping syscalls to a minimum. This is done through batching reads/writes through ring buffers that are setup between the user space and the kernel space.  
> io_uring 通过将系统调用保持在最低限度，使异步 I/O 处理变得非常快。这是通过在用户空间和内核空间之间设置的环形缓冲区批量执行读写操作来实现的。



>C runtime:
>
>* [glibc](https://sourceware.org/glibc/libc.html)(GNU C Library 是**系统基础库，是用户空间与kernel的桥梁**(`extend ret xx_syscall(a, b)`对系统调用封装)提供了 C runtime底层实现(stdio stdlib string sys bits asm)、[POSIX接口](https://pubs.opengroup.org/onlinepubs)(unistd UNIX Standard)、GNU扩展(需要 #define _GNU_SOURCE)(syscall))
>* musl libc
>* MSVCRT(Microsoft Visual CRT)



资料：

* [man 7 io_uring](https://man.archlinux.org/man/io_uring.7)
  [源码](https://elixir.bootlin.com/linux/v6.15.3/source/io_uring/io_uring.c)
  [Efficient IO with io_uring(重要)](https://kernel.dk/io_uring.pdf)

> 通信基础：
> POSIX aio(): 不支持iopoll
> io_uring: 通过使用共享的环形缓冲区，我们可以消除应用与内核间共享锁，转而巧妙利用内存排序和屏障来处理

* 202506glibc还未提供io_uring的封装 [sourceware](https://sourceware.org/bugzilla/buglist.cgi?quicksearch=io_uring)
* [man3#liburing](https://man.archlinux.org/listing/extra/liburing/)(作者是Linux Kernel io_uring的作者Jens Axboe)	|	[git.kernel](https://git.kernel.dk/cgit/liburing/) [github](https://github.com/axboe/liburing)
  liburing仅是对系统调用的封装
  TODO: 有空可以看下项目下的example，进阶一点看eBPF(Extended Berkeley Packet Filter)跟踪io_uring进行性能分析监控
* [zig#IoUring](https://ziglang.org/documentation/master/std/#std.os.linux.IoUring)



##### 手动薄封装syscall_io_uring

薄封装需要手动管理SQE、CQE(内存排序/屏障)
还可以用事件循环封装(开一个后台线程处理CQE)

```bash
# https://github.com/microsoft/WSL2-Linux-Kernel
git clone https://github.com/microsoft/WSL2-Linux-Kernel.git
cd WSL2-Linux-Kernel
sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev cpio qemu-utils  # ubuntu
pacman -S pahole cpio qemu-img  # archlinux
pacman -S base-devel flex bison pahole libssl libelf cpio qemu-img extra/bc  # archlinux

export CC="gcc -std=gnu11"  # wsl Makefile中KBUILD_CFLAGS += -std=gnu11
export HOSTCC="gcc -std=gnu11"
make menuconfig KCONFIG_CONFIG=Microsoft/config-wsl  # 可视化修改 ./Microsoft/config-wsl
make KCONFIG_CONFIG=Microsoft/config-wsl HOSTCFLAGS="-std=gnu11" KCFLAGS="-std=gnu11" -j$(nproc)
make INSTALL_MOD_PATH="$PWD/modules" modules_install
...  # 略  # 推荐去Microsoft Store更新WSL
```

```c
// hello_syscall.c
...
```



##### liburing-2.11

编译: 

```bash
# BUILD liburing.so liburing-ffi.so liburing.a liburing-ffi.a

./configure --help
Usage: configure [options]
Options: [defaults in brackets after descriptions]
  --help                   print this message
  --prefix=PATH            install in PATH [/usr]
  --includedir=PATH        install headers in PATH [/usr/include]
  --libdir=PATH            install runtime libraries in PATH [/usr/lib]
  --libdevdir=PATH         install development libraries in PATH [/usr/lib]
  --mandir=PATH            install man pages in PATH [/usr/man]
  --datadir=PATH           install shared data in PATH [/usr/share]
  --cc=CMD                 use CMD as the C compiler
  --cxx=CMD                use CMD as the C++ compiler
  --use-libc               use libc for liburing (useful for hardening)
  --enable-sanitizer       compile liburing with the address and undefined behaviour sanitizers. (useful for debugging)

# Prepare build config (optional).
cd liburing
./configure --prefix="$PWD/install" --cc=clang --cxx=clang++
# Build liburing.
make -j$(nproc)
# Build liburing.pc
make liburing.pc  # 生成 pkg-config 元数据文件(./liburing.pc)
# Install liburing (headers, shared/static libs, and manpage).
sudo make install
```

使用: 看hello_liburing.cpp



## 目录划分

/opt 大软件安装位置

/usr/local 小软件安装位置

| 文件位置                                                     | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /                                                            | 根目录                                                       |
| /bin                                                         | binary，二进制可执行命令文件，也就是一些常用命令.            |
| /boot                                                        | 存放系统引导时使用的各种文件.                                |
| /boot/grub/grub.cfg                                          | 配置grub开机引导                                             |
| /dev                                                         | 存放设备文件.                                                |
| **/etc（etcetera附加物表示配置）**                           | 存放软件的全局配置文件                                       |
| /etc/bash.bashrc                                             | 系统级bashrc<br />对env起作用的配置文件顺序：`/etc/bash.bashrc`、`~/.bashrc`、`~/.bash_profile`<br />export PATH="/root/goroot/bin:$PATH"  # 前面的优先级高 |
| /etc/fstab                                                   | 配置开机自动挂载配                                           |
| **/etc/group**                                               | 查看所有组，配置 组名:密码占位符(x或*):组ID(GID):组成员列表（逗号分隔） |
| /etc/hosts                                                   | DNS解析，配置映射关系hostname:IP                             |
| **/etc/hostname**                                            | 配置 hostname                                                |
| /etc/my.cnf                                                  | mysql配置文件                                                |
| **/etc/passwd**                                              | 查看所有用户，配置 用户名:密码占位符(x或*):用户ID(UID):组ID(GID):用户描述:家目录:登录Shell |
| **/etc/profile**                                             | 环境变量，注释符 #，<br />更改完环境变量使用source profile命令<br />推荐去改 /etc/profile.d/custom.sh |
| /etc/rc.d/rc.local                                           | 配置开机启动项                                               |
| /etc/resolv.conf                                             | 手动配置DNS服务器地址，也可以用网络管理器(systemd-resolved等)替代 |
| /etc/services                                                | 服务及其端口                                                 |
| /etc/shadow                                                  | 存储用户加密密码信息                                         |
| /etc/yum.repos.d/xx                                          | yum.repo源                                                   |
| /etc/subuid                                                  | 在容器、沙箱中可能需要使用subuid(范围uint32)、subgid<br/># root uid是0不需要分配<br/>nemesis:100000:65536  <br/>user2:165536:65536 |
| /etc/subgid                                                  |                                                              |
| /etc/sysconfig/network                                       | 配置hostname                                                 |
| /etc/sysconfig/network-scripts/ifcfg-eth0_or_33              | 网络接口配置文件，修改需要root权限                           |
| /home                                                        | 用于存储非root的其他用户根目录.<br />\home\<UserName>\XXX    |
| /lib                                                         | lib 静态链接库，是系统中的运行程序和内核模块<br />DLL 动态链接库，是程序接口，当需要时才会调用的模块 |
| /mnt                                                         | 挂载目录.                                                    |
| /opt                                                         | 额外安装的可选的应用程序安装位置.<br />/opt 目录是存放某些大型软件或者某些特殊软件的目录，比如谷歌浏览器(Google Chrome)默认就是安装在/opt中。<br />jdk、tomcat |
| **/opt/modules/software**                                    | 软件的安装目录                                               |
| /opt/modules/source                                          | 软件的安装包的目录                                           |
| /proc                                                        | 虚拟进程文件系统，当前内存中的映射文件.启动时,产生,关机时消失. |
| /proc/swaps                                                  | 虚拟交换空间(zram、swapfile)                                 |
| /proc/$pid/environ                                           | 存储进程环境变量。环境变量是一组键值对，用于存储进程运行时所需的配置信息、系统路径、用户设置等。 |
| /root                                                        | 超级管理员根目录.                                            |
| ~                                                            | 当前用户的主目录 /username，~目录下有一些隐藏 .用户配置文件  |
| ~/.config                                                    | 用户级别的应用程序或工具的配置                               |
| ~/.bashrc                                                    | 每次打开shell触发<br />bash用户,配置文件<br />执行命令 source ~/.zshrc 来生效修改的配置，而不需要重新登录<br /><br />CMD：链接器配置文件，是存放链接器的配置信息的，我们简称为命令文件,该文件的作用是指明如何链接程序的。<br />.xxrc：rc 是run command 表示与运行终端有关的配置<br /><br />对env起作用的配置文件顺序：`/etc/bash.bashrc`、`~/.bashrc`、`~/.bash_profile`<br /><br />export PATH="/root/goroot/bin:$PATH"  # 前面的优先级高 |
| ~/.bash_profile                                              | 登陆时触发<br />对env起作用的配置文件顺序：`/etc/bash.bashrc`、`~/.bashrc`、`~/.bash_profile`<br />export PATH="/root/goroot/bin:$PATH"  # 前面的优先级高 |
| ~/maven_resp                                                 | maven的本地仓库（默认是在根目录的.m2，不是太建议）           |
| ~/software下                                                 | 软件的安装包                                                 |
| ~/app                                                        | 软件的安装目录                                               |
| ~/data                                                       | 要使用的数据                                                 |
| ~/lib                                                        | 作业jar存放的目录，静态库                                    |
| ~/shell                                                      | 要使用的脚本                                                 |
| /sbin                                                        | super binary，存储 root 用户才可以执行的高级二进制可执行命令文件 |
| /sys                                                         | 动态生成的伪文件系统 (pseudo filesystem)，允许动态查看和修改内核模块信息 |
| **/usr**                                                     | 系统级的目录，可以理解为`C:/Windows/`<br />存放命令、帮助文件、安装的Linux发行版官方提供的软件包等 |
| /usr/lib                                                     | /lib 存放系统启动和维护系统基本功能必需的库文件<br />/usr/lib 存放非启动必需但系统默认安装的库文件<br />/usr/local/lib 存放本地编译安装的非系统自带库文件 |
| **/usr/local**<br /><br />我在/usr/local/modules/java 下安装了JDK | **用户级的程序目录**，可以理解为`C:/Progrem Files/`。<br />存放用户通过源码包自编译安装的软件，即不是通过“新立得”或apt-get安装的软件。<br /><br />Oracle的不能用root安装才安装到这个目录 |
| /usr/share                                                   | 存放系统公用程序和共享数据和帮助文档                         |
| /var                                                         | 特别是/var/log 子目录. 各大程序执行日志存储目录.             |
|                                                              |                                                              |

### sys

`cat /sys/firmware/efi/fw_platform_size`

```fw_platform_size
64  # 引导模式：64：x64 UEFI、32：IA32 UEFI、文件不存在：BIOS
```

`cat /sys/block/sdc/queue/io_poll`

```
0  # 当前块设备未启用iopoll
```

`cat /sys/devices/system/cpu/vulnerabilities/meltdown`

```
Intel CPU硬件安全漏洞(推测执行、乱序执行): 
Meltdown(CPU乱序执行，通过缓冲计时器攻击恢复数据)
已修复
```

`cat /sys/devices/system/cpu/vulnerabilities/spectre_v1`

`cat /sys/devices/system/cpu/vulnerabilities/spectre_v2`

```
Spectre_v1(幽灵 CPU推测执行，通过分支预测训练后绕过边界检查攻击同进程内存)
未修复
```

### etc

`/etc/resolv.conf` 配置DNS客户端(resolver)

```resolv.conf
nameserver 127.0.0.53
options edns0 trust-ad  # Extension DNS，启用Authenticated Data标记，能提升安全性
search .  # 搜索域
```

`/etc/fstab`
[arch wiki#fatab](https://wiki.archlinuxcn.org/wiki/Fstab)

```/etc/fstab
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
/dev/vda3               /               ext4            rw,relatime     0 1

/dev/vda2               /boot/efi       vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro    0 2

/swap/swapfile none swap defaults 0 0
```

```bash
genfstab -U /mnt >> /mnt/etc/fstab  # genfstab检查所有挂载点，一般只在chroot环境使用
```

`/etc/hosts`

配置局域网DNS
[man#hosts](https://man.archlinux.org/man/hosts.5)

```/etc/hosts
# Static table lookup for hostnames.
# See hosts(5) for details.


# The following lines are desirable for IPv4 capable hosts
127.0.0.1  localhost

# The following lines are desirable for IPv6 capable hosts
::1    localhost
127.0.0.1  arch.localdomain  arch  
# IP地址 域名 别名
```

`/etc/systemd/network/xx.network` 配置 systemd-networkd，数组越大优先级越高

可预测网络接口名：
 enX: Ethernet 接口，X 是一个数字
 wlanX: 无线网络接口，X 是一个数字
 enpXsY: Ethernet 接口，基于 PCI 插槽位置命名 (p 表示 PCI, X 是总线 ID, s 是插槽 ID, Y 是功能 ID)
 wlpXsY: 无线网络接口，基于 PCI 插槽位置命名
 enWWXsY: Ethernet 接口，基于板载设备 MAC 地址命名

配置DNS:
 不推荐手动配置`/etc/resolv.conf`
 不推荐使用 systemd-resolved 的 `/etc/systemd/resolved.conf` 配置DNS
 推荐使用 systemd-networkd 的 `/etc/systemd/network/*.network` 在[Network] DNS项配置DNS
 推荐交给 systemd-networkd 的 DNS=DHCP

```default.netwok
# default.netwok
# 静态
[Match]
Name=eth0

[Network]
Gateway=172.17.63.253
Address=172.17.0.156/18
```

```20-wired.network
# 20-wired.network  # 有线配置
[Match]
Name=enp1s0  # 注意网口名称匹配，可模糊匹配(en*),

[Network]
DHCP=yes
 
[DHCPv4]
RouteMetric=100  # Metric优先级(越小越优先)
 
[IPv6AcceptRA]
RouteMetric=100
```

```25-wireless.network
# 25-wireless.network  # 无线配置
[Match]
Name=wlp2s0  # 注意网口名称匹配

[Network]
DHCP=yes
 
[DHCPv4]
RouteMetric=600
 
[IPv6AcceptRA]
RouteMetric=600
```

## Shell

### shell和终端的区别

shell：是命令解释器，是用于解释并执行命令的程序

TTY(Teletypewriter)：指终端设备，可以是串口、终端窗口、伪终端。每个TTY都有一个设备文件 /dev/tty1 与终端设备进行通信

终端界面：/dev/tty1，用于输入命令和查看输出

### 快捷键

| 快捷键    | 释义                       |
| --------- | -------------------------- |
| tab + tab | 看当前目录下模糊匹配文件名 |
| ctrl u    | 删除当前字符之前的命令行   |
| ctrl k    | 删除当前字符之后的命令行   |

### Shell命令按字典序（重点）

[Linux命令在线查询网站](https://www.linuxcool.com/)

[GNU Bash](https://www.gnu.org/software/bash/manual/bash.html)

[]可选、{}必选一个、<>必选

| shell命令                                                    | 释义                                                         | 选项                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| arp                                                          | 显示和修改动态IP地址转换表(ARP缓存表) **数据链路层**         | -a 显示当前ARP缓存表项<br />-s 添加IP地址-Mac地址记录(entry) |
| **awk** [options] ‘program’ file                             | 逐行处理，一次读取一行文本，按输入分隔符进行**切片**，切成多个组成部分。适合处理表格数据<br/>将切片直接保存在内建的变量中，`$1`，`$2`(`$0` 表示行的全部)<br/>支持对单个切片的判断，支持循环判断，默认分隔符为空格<br/><br/>`awk '{print $1, $4}' filename`  打印列1列4<br/>NR表示当前数据行数<br/>`awk '($1=="tcp" && $2==1) || NR==1 {print $0}' filename` 打印列1为tcp列2为1的行和第一行 | -F "," 指定 "," 为分隔符                                     |
| cat                                                          | less更好用。cat f1 f2连接多个文件内容并打印                  |                                                              |
| chgrp 组名 文件                                              | 改变文件的用户组用命令                                       |                                                              |
| chown 拥有者名称 文件                                        | 更改文件拥有者和所属组<br />`chown $USER:$USER ~/.ssh/authorized_keys` | -R 递归                                                      |
| chsh                                                         | 改变SHELL(bash、zsh)<br />cat /etc/shells 查看目前支持的shell<br />echo $SHELL 打印当前SHELL |                                                              |
| clear                                                        | 清屏，向下一页                                               |                                                              |
| curl [options...] <url>                                      | client URL，发送 http 请求<br />访问 cip.cc 可用于测试是否使用代理<br/> | -X Get 指定HTTP请求方法<br />-A 指定User-Agent<br />-b 指定Cookie<br />-d 指定Post请求正文<br />-F 上传二进制文件<br />-k 跳过 SSL 检测<br />-o 将响应保存为文件<br />-x 指定代理<br />-f 快速失败，根据HTTP响应状态码返回成功0失败非0 |
| df                                                           | 打印**文件系统使用情况**(已用 可用 挂载点)<br />fdisk 打印**磁盘分区情况**或操作<br />lsblk 打印**块设备信息** | -h 人类可读                                                  |
| dmesg                                                        | 显示kernel ring buffer(内核日志)(和 io_uring ring buffer不是同个东西)消息 | -HTK 人类可读 人类可读时间戳 内核消息                        |
| echo $HADOOP_HOME                                            | 可以看一些软件的安装路径                                     |                                                              |
| env                                                          | 查看环境变量<br />env \|grep -i poxy #查看系统代理配置情况   |                                                              |
| fdisk                                                        | 打印磁盘分区情况或操作<br />df 打印文件系统使用情况(已用 可用 挂载点)<br />lsblk 打印块设备信息 | -l 打印磁盘分区情况                                          |
| file <file>                                                  | 识别文件类型(看elf file的动态加载器是什么)                   |                                                              |
| **find** [path] [options] expression                         | 在指定目录下递归查找文件路径<br />find / -name "head*"<br />-iname 对大小写不敏感 |                                                              |
| firewall-cmd --list-ports                                    | 查看firewall已经开放的端口                                   |                                                              |
| firewall-cmd --zone=public --*add*-*port=80/tcp* --permanent | 开启端口，–zone #作用域  添加端口，端口/通讯协议  永久生效，没有此参数重启后失效 |                                                              |
| firewall-cmd --reload                                        | #重启firewall                                                |                                                              |
| free -h                                                      | 查看内存swap空间状态(swap=swapfile+zram)                     | h human-readable<br />m  mebitypes单位                       |
| fsck /dev/sdax                                               | file system check 检查并修复文件系统错误 (建议在umount的设备上执行) | -f 强制                                                      |
| genfstab                                                     | genfstab -U /mnt >> /mnt/etc/fstab                           | genfstab会将当前系统中已挂载的文件系统的挂载信息添加到 `/etc/fstab` 文件中<br />`/etc/fstab` 系统文件表用于系统启动时挂载指定的文件系统 |
| **grep** [option] pattern [file]                             | 查找文件里符合条件的字符串的整行内容。Globally search a Regular Expression and Print<br /><br />`grep “txt” “head*” # 支持通配符` | -v 过滤掉相关字符串的内容。可以通过管道操作符组合使用 `grep -v 'grep'`<br /> -o 只输出了匹配到的部分，而不是整行的内容<br />-i 忽略大小写<br />-E 扩展正则表达式无需转义支持+? |
| groupadd xxgroup                                             | 创建新的用户组                                               | -r 系统用户组                                                |
| groups  user1  /  passwd  password2                          | 位于root状态下，查看user1用户的所属组                        |                                                              |
| head                                                         | 打印前10行                                                   | -n N  # 打印前N行                                            |
| hostname  xxxx                                               | 临时修改hostname主机名                                       |                                                              |
| id [user]                                                    | 打印uid gid groups信息                                       |                                                              |
| ~~ifconfig~~                                                 | ip add 更好用                                                |                                                              |
| **info**                                                     | 显示帮助文档<br />q 退出                                     |                                                              |
| install                                                      | 复制文件                                                     | -D 递归创建<br />-m 设置权限                                 |
| ip                                                           | 显示配置网卡参数                                             | addr 查看网络层信息(动态IP(DHCP)、静态IP(子网掩码、网关))<br /><br />link 查看链路层信息(MAC)<br />`<接口编号>: <接口类型(lo本地回环接口|eth有限以太网接口)>:<接口状态(回环|广播,多播,up启用)> mtu最大传输单元 qdisc state mode group qlen`<br /><br />route 查看路由表信息 |
| journalctl                                                   | 查询systemd日志                                              | x 添加消息解释<br />e 跳至结尾<br />-u <xx.service><br />-f 实时 |
| jps                                                          | 由JDK提供，查看Java进程                                      | # MAC地址 广播地址                                           |
| kill -9 Pid                                                  | 强制杀死进程                                                 |                                                              |
| ldconfig                                                     | glibc包含的动态加载器(Dynamic Linker)(/lib64/ld-linux-x86-64.so.2)配置工具 | -p 打印缓存                                                  |
| **less**                                                     | pager 分页器<br />查看文件，h显示帮助                        | `ls xxx | less`                                              |
| ln                                                           | 创建硬链接                                                   | -s 创建软链接                                                |
| ls -lath<br />**ll -ath**                                    | 展示当前目录下的文件与目录.并根据颜色区分类型.<br />–a显示隐藏文件<br />-i显示inode编号 <br /> -l 附加显示详细信息<br />-t 按时间排序<br />-r 反序<br />-h 文件大小单位变为kb<br />白(一般文件)、蓝(目录)、浅蓝(链接文件)、绿(可执行)、红(压缩)<br />黄背景(Set Group ID)、红背景(Set User ID)、绿背景(粘滞位) |                                                              |
| lsblk                                                        | 打印**块设备信息**                                           | -f 多打印一列文件系统信息                                    |
| lsmod                                                        | 显示当前加载的内核模块                                       |                                                              |
| man <section> <page>                                         | manual page, 可以用info命令替代<br />`pacman -S man-db man-pages-zh_cn`<br />`alias cman='LC_ALL=zh_CN.UTF-8 LANG=zh_CN.UTF-8 man'`<br />[Online_man_pages](https://wiki.archlinux.org/title/Man_page#Online_man_pages) \| [kernel#man page version](https://git.kernel.org/pub/scm/docs/man-pages/man-pages.git/refs/tags)<br />[archlinux#man page(推荐)](https://man.archlinux.org/) \| [archlinux#man page version](https://archlinux.org/packages/?name=man-pages)<br />man7.org(不推荐 更新慢) | 1shell命令 2内核提供的系统调用 3库函数 4特殊文件 5file format and config file 6游戏 7Miscellaneous宏(需要`#define MISC`) 8系统管理命令 9kernel routines |
| mkdir myfloder                                               | 创建空目录.  <br />mkdir –p myfloder:  如果已经存在,也不报错提示. <br />mkdir无法创建多层目录,所以可以用 : mkdir –p a/b/c ) |                                                              |
| modinfo <modulename\|fielname>                               | 显示内核模块信息                                             |                                                              |
| modprobe                                                     | 用于加载内核模块                                             | `modprobe <module name>` 手动加载内核模块<br />`echo <module name> >> /etc/modules-load.d/<module name>.conf` 自动加载内核模块<br /><br />`modprobe -c` 打印模块配置 `modprobe -c |grep <module_name>``modprobe -c \|less`<br />`modprobe --show-depends module_name` 打印模块依赖项及其本身 |
| mount <磁盘分区> <目录><br />mount <虚拟分区>                | 挂载分区<br />mount /.snapshots/  # 将名为.snapshots的分区挂载到.snapshots目录<br />挂载后可用`df -h`查看挂载情况 | -o 参数<br />-a 挂载`/etc/fstab`中匹配值的文件系统<br />-B --bind 挂载一个子树 |
| ~~more~~                                                     | less命令更好用                                               |                                                              |
| nc -l 9001                                                   | 开放9001TCP，等待客户端连接，可以传输字符串                  |                                                              |
| **neofetch**                                                 | new fetch 显示系统信息和Logo                                 |                                                              |
| ~~netstat -tulpn~~ 弃用                                      | 用ss命令更好用，查看占用的端口                               |                                                              |
| pgrep                                                        | process global rep 就是对 ps \| grep \| awk 的封装 用于打印pid | -f full process name match<br />-a <br />-v 反向匹配         |
| ping                                                         | ping 指定主机 **网络层ICMP**                                 |                                                              |
| ps -ef \| grep “tomcat” \| grep -v “grep”                    | 查看当前时刻活动进程信息<br />root      21772  21674  0 15:59 pts/3    00:00:00 grep --color=auto tomcat<br />UID         PID   PPID  C STIME TTY  TIME CMD<br />PPID：父进程的<br />C：CPU使用的资源百分比<br />TTY：与进程关联的终端（tty）<br />TIME：使用掉的 CPU 时间<br />CMD：所下达的指令名称 | –e 所有进程<br />-f 完整格式<br />--forest 进程树<br />-o pid,command |
| pstree [pid]                                                 | 显示进程树                                                   | -h 高亮当前进程及其父进程<br />-a 显示命令行参数<br />-l 不截断长行<br />-p 显示pid<br />-s 显示指定进程的父进程 |
| rm                                                           | 删除xxyy文件                                                 | -r<br />-f force 忽略文件不存在时的删除失败提示              |
| rpm -ivh xxx.rpm<br />rpm -qa \| grep xxxx                   | RedHat Package Manager<br />只能安装已经下载到本地机器上的 rpm 包，没有解决软件包依赖问题。<br />参数 i-install、v-verbose可视、u-Update、q-Query、p-Package、l-list、e删除、a-all、h-hash哈希后人可读、--nodeps忽略依赖关系<br /><br />yum remove 若你要移除的包被别的软件包需要的话，yum会把其他软件包一起移除<br />rpm -e 会告诉你该包被别的包需要，所以无法移除 |                                                              |
| **rz** -ebq                                                  | 上传到当前目录  <br />e 表示强制escape转义所有控制字符，b二进制传输，q 安静quiet没有进度条<br />如过路径有中文则必须用e参数 |                                                              |
| **scp -rq 主机名:**                                          | security copy<br />基于ssh远程连接协议实现复制. 一般需要密码.<br/>本地到远程 : scp /home/test/\*abc        用户名@10.1.1.2:/home/root<br/>远程到本地 : scp 用户名@10.1.1.2:/home/root/\*abc       /home/test<br/>远程到远程 : scp 用户名1@10.1.1.2:/home/root/*abc       用户名2@10.1.1.3:/home/root | r复制递归目录<br />v显示进度<br />p不显示进度                |
| **sed** [option] 'sed command' filename                      | stream editor流式编辑器，适合对文本的行内容进行处理，替换、删除、移动、搜索数据<br />`sed 's/^Str\.&/\;/g' filename`：在终端将Str.替换成;   s表示处理字符串 g表示全文替换否则只替换行首次匹配<br />`sed -i '/^ *$/d' filename`：在文件中删除空行，d表示删除行<br />`sed -i '/xxx/d' filename`：删除xxx所在行<br />`sed -i "s/ALLOW_USERS= \".*\" /ALLOW_USERS= \"$(whoami)\" /" /etc/snapper/configs/root`<br />s/ 替换字符<br />/d 删除 | -i 在文件中修改                                              |
| sh -c [string]                                               | 执行脚本                                                     |                                                              |
| shutdown [option] [time] [message]                           |                                                              |                                                              |
| shutdown -h now                                              | 马上关机                                                     |                                                              |
| **source <file>** (等价于. <file>)                           | 在当前shell环境，执行脚本文件中的命令                        |                                                              |
| **ss**                                                       | 显示sockets                                                  | t 显示TCP sockets<br />u 实现 udp<br />n 显示端口，不解析为服务名称<br />l 显示正在监听的 |
| **ssh 用户名@ip**                                            | ssh登录远程主机                                              | -T                                                           |
| ssh-add <私钥文件>                                           | 向ssh-agent添加私钥身份identity<br />OpenSSH要求私钥文件不能被其他用户访问 | -L 打印所有公钥<br />-l 打印所有fingerprints <br />-D 删除所有<br />-d 删除该密钥<br />-v Verbose<br />-K 从FIDO验证器操作常驻密钥 |
| sshd -t                                                      | 测试`~/.ssh`配置                                             |                                                              |
| ssh-keygen                                                   | [arch wiki#SSH_keys](https://wiki.archlinux.org/title/SSH_keys)<br />`shh-keygen -A`<br />`ssh-keygen -t ed25519 -C "xxyy"`  <br/>Enter passphrase通信短语 (empty for no passphrase):<br />[github#关于SSH密钥的通信短语](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases#about-passphrases-for-ssh-keys) | -A # 在Server端执行，在**`/etc/ssh`**目录下生成` ssh_host_<ed25519|dsa|ecdsa|rsa>_key`私钥 和` .pub`**服务端host临时公钥**  <br /># 用于启动sshd.service服务  <br /># 用于Client首次连接到Server, Client将Server公钥保存到`~/.ssh/known_hosts`<br /><br /># 在Client端执行，在**`~/.ssh`**目录下生成id_rsa私钥 和 .pub 公钥 <br />-t 算法<br />-b 密钥长度bits, 推荐4096<br />-C 注释信息<br />-f /etc/ssh/ssh_host_xxx_key |
| sshd                                                         |                                                              | -t 检查sshd配置文件                                          |
| sudo -i                                                      | 切换到root用户                                               |                                                              |
| su xxyy                                                      | 换到普通用户                                                 |                                                              |
| sysctl                                                       | 运行时检查和更改内核参数的工具                               | -a 显式所有变量 (/proc/sys/xxx)<br />--system 手动加载所有配置文件<br />-p/--load=<file.conf> 加载单个配置文件<br />-w k=v 临时写入变量值<br /><br />配置文件：`/etc/sysctl.d/99-sysctl.conf` `/usr/lib/sysctl.d/xxx`<br />net.ipv4.ip_local_port_range = 30000 65535 临时端口范围<br />net.core.default_qdisc = cake <br />net.ipv4.tcp_congestion_control = bbr<br />增加net.ipv4.tcp_max_tw_buckets可放DOS攻击 |
| systemctl [OPTIONS...] COMMAND ...                           | 查询或发送控制命令到系统管理器<br />UNIT服务单元：network,mysql,firewalld,mongod,mysqld<br />q 退出<br />本质是启动 unit.service | status [PATTERN...\|PID...] 显示正在运行的该服务状态<br />start UNIT...<br />enable<br />stop UNIT...<br />disable UNIT... 开机不启动<br />reload UNIT... 重载配置文件<br />restart UNIT... 重启服务<br />**list-unit-files --type=service 列出所有服务单元文件**<br />**daemon-reload 用于重新加载缓存的systemd配置、unit文件** |
| sz xxyy                                                      | 导出xxyy到本地快速访问download中                             |                                                              |
| tail -n 20 filename                                          | 查看文件最后末尾20行                                         |                                                              |
| tar –zcvf a.tar.gz<br />tar –xzvf a.tar.gz -C /target_dir    | `tar -tzf <file> | head -10`                                 | v：verbose<br />f：指定文件<br />c压缩、x解压、t查看压缩包内容不解压<br />z：gzip、j：bzip压缩算法 |
| telnet                                                       | 远程登入，可以测试端口的连通性 **应用层**                    |                                                              |
| timedatectl                                                  | 查看系统时间                                                 |                                                              |
| top                                                          | htop                                                         | -d <间隔second><br />-p <pid>                                |
| tracepath [options] <destination>                            | 路径探测跟踪                                                 |                                                              |
| ulimit -n                                                    | 打印打开fd的最大值(即fd数量)                                 |                                                              |
| **uname -a**                                                 | 打印所有系统信息，包括linux版本，主机名                      |                                                              |
| useradd  xxuser                                              | 创建新的用户<br />sudo useradd -m -g users -s /bin/bash nemesis<br />sudo passwd nemesis | -r 系统用户<br />-m 创建该用户home目录<br />-G 所属的附属组(supplementary GROUPS)<br />-s 登录的shell<br />-g 所属的主要组 |
| usermod -aG groupname username                               | 修改所属的附属group                                          | -a 添加<br />-r 删除<br />-G 附属组(supplementary GROUPS)    |
| vim                                                          | 编辑文本                                                     |                                                              |
| visudo                                                       | export EDITOR=vim  # 默认是vi，export临时生效<br />testuser ALL=(ALL:ALL) ALL  # <用户名> <主机名>=(\<运行用户:运行组>) <命令列表> |                                                              |
| wget url                                                     | 下载xx源                                                     |                                                              |
| which [cmd]                                                  | 显示命令所在路径                                             |                                                              |
| whoami                                                       | 打印当前用户名                                               |                                                              |
| xargs [OPTION]... COMMAND [INITIAL-ARGS]...                  | extended argument ，默认情况下，将换行符和空格作为分隔符，把stdin分解成一个个命令行参数 |                                                              |
| yum [list remove install]                                    | Yellow dog Updater, Modified<br />属于 Redhat 系列 (如，CentOS) 包管理工具；Debin系列 (如，Ubuntu) 使用 apt-get<br />基于RPM包管理工具，能从指定的服务器离线下载 rpm 包并且自动安装，会自动处理依赖问题，还能更新系统。 |                                                              |
| zramctl                                                      | 查看zram设备状态                                             |                                                              |
| **\|**                                                       | 管道操作符，将左指令的stdout作为右指令的stdin<br />需要注意：**右边命令必须能接受标准输入流**，否则传递过程中的数据会被抛弃、不处理左边命令的错误<br />常用接收stdin的命令: sed, awk, grep查, cut, head查, top资源, less, more, wc统计, join, sort, split |                                                              |
| >                                                            | `agent_run_state=$(ssh-add -l >/dev/null 2>&1; echo $?)  # stdout重定向到/dev/null、stderr重定向到stdout` <br />文件描述符 1:stdout 2:stderr<br />`echo $?` 获取前一个命令的退出状态码 |                                                              |
| >>                                                           | 输出重定向<br />cat ~./ssh/id_rsa.pub  >>  ~/.ssh/authorized_keys  把公钥追加到authorized_keys中 |                                                              |
| &                                                            | 表示在后台执行                                               |                                                              |
| &&                                                           | 表示前一条命令执行成功才执行后一条命令                       |                                                              |
| $()                                                          | `$()` 捕获括号内的stdout<br />powershell：${}，例如${pwd}    |                                                              |

# 安装系统

## 了解现况

轻量云没内存，适合作中间代理流量转发，不适合部署服务

> # 阿里云云服务器ESC文档
>
> ### **公共镜像的启动模式（系统定义）**
>
> 不同版本的公共镜像默认支持的启动模式说明如下：
>
> * UEFI版本的公共镜像：默认支持UEFI启动模式。
>
>   例如Alibaba Cloud Linux 2.1903 64位UEFI版、Ubuntu 18.04 64位UEFI版、Debian 11.6 64位UEFI版等操作系统名称带UEFI的公共镜像的启动模式是UEFI。
>
> * Arm版本的公共镜像：默认支持UEFI启动模式。
>
>   例如Ubuntu 20.04 64位Arm版、CentOS 8.4 64位Arm版等操作系统名称带Arm的公共镜像的启动模式是UEFI。
>
> * 其他公共镜像：默认支持BIOS（Legacy）启动模式、UEFI-Preferred启动模式。
>
>
>
> ## **启动模式简介**
>
> ECS的启动模式包括BIOS（Legacy）和UEFI两类。
>
> * BIOS（Legacy）模式：BIOS是系统启动过程中的基础软件层，负责初始化硬件并提供基本的硬件服务，以支持操作系统的启动。BIOS是一种传统的固件接口标准，其功能相对有限。
> * UEFI模式：UEFI是BIOS的现代替代品，是一个更高级、模块化的固件接口标准，提供更强大、灵活和安全的启动环境。UEFI模式相对于BIOS（Legacy）模式有一些优势，具体说明如下。
>   优势：
>   UEFI启动时只需要加载必要的驱动程序，而传统BIOS（Legacy）启动时需要扫描所有设备
>   反正UEFI 启动速度、安全性、可扩展性都更好

系统引导

```bash
$ cat /sys/firmware/efi/fw_platform_size  # 引导模式：64：x64 UEFI、32：IA32 UEFI、文件不存在：BIOS
64

$ cat /etc/default/grub
GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR=`( . /etc/os-release; echo ${NAME:-Ubuntu} ) 2>/dev/null || echo Ubuntu`
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX=" vga=792 console=tty0 console=ttyS0,115200n8 net.ifnames=0 noibrs iommu=pt crashkernel=0M-1G:0M,1G-4G:192M,4G-128G:384M,128G-:512M nvme_core.io_timeout=4294967295 nvme_core.admin_timeout=4294967295"
```

网络

```bash
$ cat /etc/hostname 
iZj6c6jjq4hjrfg9oja020Z

$ cat /etc/hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

$ cat /etc/resolv.conf  # DNS服务器
nameserver 127.0.0.53
options edns0 trust-ad
search .

$ ls ~/.ssh  # ssh密钥
authorized_keys

$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:0d:8d:f8 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    altname ens3
    inet 172.17.0.156/18 metric 100 brd 172.17.63.255 scope global dynamic eth0  # 说明的DHCP  # 静态IP记IP地址、子网掩码、网关
       valid_lft 313758500sec preferred_lft 313758500sec
    inet6 fe80::216:3eff:fe0d:8df8/64 scope link 
       valid_lft forever preferred_lft forever

$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:0d:8d:f8 brd ff:ff:ff:ff:ff:ff  # MAC地址 广播地址
    altname enp0s3
    altname ens3
```

磁盘

```bash
$ lsblk -f  # 块设备信息
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
vda                                                                           
├─vda1                                                                        
├─vda2 vfat   FAT32       4AA9-6FC3                             190.8M     3% /boot/efi
└─vda3 ext4   1.0         b4cbd348-1941-41f0-908b-42f34f456dcc   25.1G     9% /

$ fdisk -l  # 磁盘信息
Disk /dev/vda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt  # gpt分区表、dos: MBR分区表
Disk identifier: 78A99BB0-93BF-4A0F-8E8F-22CCC1C6BD0C

Device      Start      End  Sectors  Size Type
/dev/vda1    2048     4095     2048    1M BIOS boot
/dev/vda2    4096   413695   409600  200M EFI System
/dev/vda3  413696 62914526 62500831 29.8G Linux filesystem

$ df -h  # 文件系统信息
Filesystem      Size  Used Avail Use% Mounted on
tmpfs            71M  1.1M   70M   2% /run
efivarfs        256K  6.6K  245K   3% /sys/firmware/efi/efivars
/dev/vda3        30G  2.8G   26G  10% /
tmpfs           352M     0  352M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda2       197M  6.2M  191M   4% /boot/efi
tmpfs            71M   12K   71M   1% /run/user/1000

$ fsck -f /dev/sda
```

服务

```bash
$ systemctl list-unit-files --type=service --state=enabled
UNIT FILE                              STATE   PRESET 
aegis.service                          enabled enabled
aliyun.service                         enabled enabled
apparmor.service                       enabled enabled
apport.service                         enabled enabled
blk-availability.service               enabled enabled
chrony.service                         enabled enabled
cloud-config.service                   enabled enabled
cloud-final.service                    enabled enabled
cloud-init-local.service               enabled enabled
cloud-init.service                     enabled enabled
cloudmonitor.service                   enabled enabled
console-setup.service                  enabled enabled
cron.service                           enabled enabled
dmesg.service                          enabled enabled
e2scrub_reap.service                   enabled enabled
ecs_mq.service                         enabled enabled
finalrd.service                        enabled enabled
getty@.service                         enabled enabled
gpu-manager.service                    enabled enabled
grub-common.service                    enabled enabled
grub-initrd-fallback.service           enabled enabled
kdump-tools.service                    enabled enabled
keyboard-setup.service                 enabled enabled
lvm2-monitor.service                   enabled enabled
ModemManager.service                   enabled enabled
multipathd.service                     enabled enabled
networkd-dispatcher.service            enabled enabled
open-iscsi.service                     enabled enabled
open-vm-tools.service                  enabled enabled
pmcd.service                           enabled enabled
pmie.service                           enabled enabled
pmlogger.service                       enabled enabled
pmproxy.service                        enabled enabled
pollinate.service                      enabled enabled
rsyslog.service                        enabled enabled
secureboot-db.service                  enabled enabled
setvtrgb.service                       enabled enabled
snapd.apparmor.service                 enabled enabled
snapd.autoimport.service               enabled enabled
snapd.core-fixup.service               enabled enabled
snapd.recovery-chooser-trigger.service enabled enabled
snapd.seeded.service                   enabled enabled
snapd.service                          enabled enabled
snapd.system-shutdown.service          enabled enabled
sysstat.service                        enabled enabled
systemd-network-generator.service      enabled enabled
systemd-networkd-wait-online.service   enabled enabled
systemd-networkd.service               enabled enabled
systemd-pstore.service                 enabled enabled
systemd-resolved.service               enabled enabled
tuned.service                          enabled enabled
ua-reboot-cmds.service                 enabled enabled
ubuntu-advantage.service               enabled enabled
udisks2.service                        enabled enabled
ufw.service                            enabled enabled
unattended-upgrades.service            enabled enabled
vgauth.service                         enabled enabled
```

操作系统

```bash
$ uname -a
Linux iZj6c6jjq4hjrfg9oja020Z 6.8.0-40-generic #40-Ubuntu SMP PREEMPT_DYNAMIC Fri Jul  5 10:34:03 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

## 开始安装

方案一：**使用ISO**(需要~~USB或光盘作为安装介质~~、或PXE、或initramfs内存文件系统)
方案二：**~~使用LiveCD镜像~~** (LiveCD 是含Linux的可引导CD，PC能从CD启动该Linux。有的LiveCD提供了安装完整发行版的功能
方案三：**使用~~Bootstrap tarball~~**的arch-chroot进入chroot环境
方案四：**~~搭建arch-install-scripts包可运行的环境~~**(chroot环境(改根目录))

初始环境：
debian：(阿里云上只有一个分区 /)、(bios)
ubuntu：(/grub_bios /esp /)、(bios uefi)

云服务器上无法使用 Live CD 或者其他物理启动介质

### 需要磁盘(不能格式化)

#### 1bootstrap tarball(无法格式化)

使用 bootstrap tarball 在Btrfs分区上安装Arch Linux

> 具体思路要么是直接在宿主系统上运行 pacman，要么是在宿主系统里运行一个 Arch 系统，这个嵌套系统位于 chroot 中。

临时环境：archlinux-bootstrap-x86_64/root.x86_64，配置最小化系统的引导
新系统：/mnt

先 [archlinux wiki#Install Arch Linux from existing Linux(准备步骤)](https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux#From_a_host_running_another_Linux_distribution)，后 [archlinux wiki#安装指南](https://wiki.archlinux.org/title/Installation_guide)
从3.2创建chroot开始执行
主要在临时环境bootstrap根(最小化Linux环境)中执行

```bash
防火墙放通 ICMP:ALL TCP:80 443 22(SSH端口) 3389

# 方案二：bootstrap tarball，适合小内存VPS
# 直接从3.2创建chroot开始https://wiki.archlinuxcn.org/wiki/%E4%BB%8E%E7%8E%B0%E6%9C%89_Linux_%E5%8F%91%E8%A1%8C%E7%89%88%E5%AE%89%E8%A3%85_Arch_Linux#%E5%88%9B%E5%BB%BA_chroot

# 下载bootstrap tarball并验证、解压
$ cd /tmp
$ curl -O https://mirror.xtom.com.hk/archlinux/iso/2025.02.01/archlinux-bootstrap-x86_64.tar.zst
$ curl -O https://archlinux.org/iso/2025.02.01/archlinux-bootstrap-2025.02.01-x86_64.tar.zst.sig
# pacman -Qs gnupg  # GnuPG(primary guard)  # 如果没安装 pacman -S gnupg
$ gpg --help
# '/home/admin/.gnupg'  # 配置文件
# '/home/admin/.gnupg/pubring.kbx'  # 存储公钥
$ gpg --version  # GnuPG 2.3后移除了--keyserver-options auto-key-retrieve  # 自动从密钥服务器下载公钥，需要到配置文件~/.gnupg/gpg.conf中设置  # 建议还是手动管理公钥不要启动auto-key-retrieve
$ gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org  # 通过wkd协议获取公钥
# [阮一峰gpg教程](https://www.ruanyifeng.com/blog/2013/07/gpg.html)
$ gpg --list-keys
$ gpg --keyserver-options auto-key-retrieve --verify archlinux-bootstrap-2025.02.01-x86_64.tar.zst.sig archlinux-bootstrap-x86_64.tar.zst    # GnuPG用公钥验证数字签名，输出Good signature表示文件正确
$ tar xf archlinux-bootstrap-x86_64.tar.zst --numeric-owner  # 解压出 pkglist.x86_64.txt、root.x86_64/
# pacman -Ss zstd  # 看是否安装
# zstd -d archlinux-bootstrap-x86_64.tar.zst
# tar -xzvf archlinux-bootstrap-x86_64.tar --numeric-owner  # --numeric-owner 解压出文件的uid gid

# 使用bootstrap镜像中的arch-chroot进入临时环境
$ vim /tmp/root.x86_64/etc/pacman.d/mirrorlist  # pacman软件仓库服务器列表  # 把HK、CN的放前面 https://archlinux.org/mirrorlist/
$ bash --version  # 要求支持 >4.0
$ unshare --help | grep -E "fork|pid"  # 要求支持--fork --pid
$ sudo mount --bind /tmp/root.x86_64 /tmp/root.x86_64  # 解决后面pacman -S的error: could not determine cachedir mount point /var/cache/pacman/pkg
$ sudo /tmp/root.x86_64/bin/arch-chroot /tmp/root.x86_64/

# 3.3使用chroot环境
$ ls -Ra  # 增加熵用于初始化随机数生成器
$ ls -Ra
$ ls -Ra
$ ls -Ra
$ ls -Ra
$ uname -a
Linux iZj6c6jjq4hjrfg9oja020Z 6.8.0-40-generic #40-Ubuntu SMP PREEMPT_DYNAMIC Fri Jul  5 10:34:03 UTC 2024 x86_64 GNU/Linux
$ head -n 15 /etc/pacman.d/mirrorlist  # 看下配置好了没有
$ pacman-key --init
$ pacman-key --populate
$ pacman -Syyu
$ pacman -S base
$ pacman -S base-devel  # Basic tools to build Arch Linux packages
$ pacman -S parted  # A program for creating, destroying, resizing, checking and copying partitions 分区工具
```

Btrfs
失败噜，无法格式化~
由于磁盘分区还在被原有的系统内核使用中，所以无法卸载并重新格式化

```bash
# 创建分区
$ lsblk   
$ fdisk -l  # 选择要被分区的块磁盘(我是Disk /dev/vda)
$ parted /dev/vda  # parted修改分区表，默认进入交互模式，-s进入脚本编辑模式  # 也可以使用其他分区工具fdisk
$ (parted) help
$ (parted) print  # 打印当前分区表
Model: Virtio Block Device (virtblk)
Disk /dev/vda: 32.2GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  2097kB  1049kB                     bios_grub
 2      2097kB  212MB   210MB   fat32              boot, esp
 3      212MB   32.2GB  32.0GB  ext4
$ (parted) mklabel gpt  # 说明创建的是gpt(GUID)分区表
# label name start end
$ (parted) mkpart "BIOS boot partition" 1MiB 2MiB  # 存放GRUB的代码，看作一个裸设备而不需要格式化，GPT_BIOS也不需要挂载
$ (parted) mkpart "Efi system partition" 2MiB 212MiB  # GPT_UEFI可以挂载到/boot/efi BIOS挂载到/boot
$ (parted) mkpart "Arch btrfs partition" 212MiB 100%
$ (parted) set 1 bios_grub on  # 指定该分区是BIOS boot分区，只用于BIOS启动模式 https://wiki.archlinuxcn.org/wiki/GRUB#GUID_%E5%88%86%E5%8C%BA%E8%A1%A8_(GPT)_%E7%89%B9%E6%AE%8A%E6%93%8D%E4%BD%9C
$ (parted) set 2 boot on  # 指定该分区是可引导 (bootable) 分区
$ (parted) set 2 esp on  # 指定该分区是ESP，只用于UEFI启动模式  # https://wiki.archlinuxcn.org/zh-sg/EFI_%E7%B3%BB%E7%BB%9F%E5%88%86%E5%8C%BA#GPT_%E5%88%86%E5%8C%BA%E7%A3%81%E7%9B%98  # parted分区工具: 需要创建文件系统类型为fat32的分区，并设置esp标志  # 也可以使用其他分区工具
$ (parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/vda: 32.2GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name                  Flags
 1      1049kB  2097kB  1049kB               BIOS boot partition   bios_grub
 2      2097kB  222MB   220MB                Efi system partition  boot, esp
 3      222MB   32.2GB  32.0GB               Arch btrfs partition
$ (parted) quit
# 使用GPT_BIOS启动，之后需要安装GRUB grub-install --target=i386-pc /dev/vda
# 使用GPT_UEFI启动，之后需要挂载EFI系统分区 mount --mkdir /dev/efi_system_partition /mnt/boot

# 创建文件系统格式化分区  # https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97#%E6%A0%BC%E5%BC%8F%E5%8C%96%E5%88%86%E5%8C%BA
pacman -S btrfs-progs
lsblk
fdisk -l
mkfs.btrfs -L arch /dev/vda3  # -L 指定卷标
pacman -S dosfstools
mkfs.fat -F 32 /dev/vda2  # -F指定Fat32  # 格式化ESP  

# 挂载分区子卷
mount /dev/vda2 /mnt
btrfs subvolume create /mnt/@root  # @顶层子卷  # 系统根目录
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var  # 可变数据(日志、缓存)
btrfs subvolume create /mnt/@snapshots  # snapper快照子卷
btrfs subvolume create /mnt/@opt
umount /mnt
opts=noatime,compress=zstd,space_cache=v2,x-mount.mkdir  # 变量opts
mount -o $opts,subvol=@ /dev/vda2 /mnt
mount -o $opts,subvol=@root /dev/vda2 /mnt/root
mount -o $opts,subvol=@home /dev/vda2 /mnt/home
mount -o $opts,subvol=@opt /dev/vda2 /mnt/opt
mount -o $opts,subvol=@var /dev/vda2 /mnt/var
mount -o $opts,subvol=@snapshots /dev/vda2 /mnt/.snapshots
lsattr /mnt/.snapshots  # 查看是否包含C字符，是否启动了CoW特性

# 安装基础系统和所需的软件
pacstrap -K /mnt base linux linux-firmware grub openssh sudo git neofetch neovim fish  # -K 初始化pacman keyring  # 是安装软件包到/mnt 新系统
# -c 从主机系统中包缓存而不是临时系统(挂载点)
# base：Minimal package set to define a basic Arch Linux installation
# https://wiki.archlinux.org/title/Kernel
# linux：The Linux kernel and modules  # 也可以选择linux-lts内核、linux-zen(多媒体)、linux-rt(低延迟)、linux-hardened(安全增强)
# linux-firmware：Firmware files for Linux 固件
# grub: GNU GRand Unified Bootloader
```

后 [archlinux wiki#安装指南](https://wiki.archlinux.org/title/Installation_guide)
从3配置系统开始执行
主要在新安装的ArchLinux内执行

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
arch-chroot /mnt
$ cat /etc/fstab  # 看下和之前有什么区别
```

##### 设置时区

```bash
# 3.3设置时区
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  # 链接
$ hwclock --systohc  # 生成/etc/adjtime 校准
$ timedatectl set-ntp true  # 启动Network Time Protocol客户端设置时间同步，过一会儿生效
$ timedatectl status  # 查看是否启动
$ date
```

##### 区域和本地化设置

```bash
# 3.4区域和本地化设置
$ vim /etc/locale.gen  # 取消注释 en_SG.UTF-8 UTF-8  
$ locale-gen  # 生成locales信息
$ echo LANG=en_SG.UTF-8 >> /etc/locale.conf  # LANG变量
$ echo KEYMAP="us" >> /etc/vconsole.conf  # 标准美式键盘（中国最常见的键盘布局）
```

##### 网络配置

```bash
# 3.5网络配置 https://wiki.archlinuxcn.org/wiki/Systemd-networkd
$ echo <Alpha(yourhostname)> >> /etc/hostname  
$ vim /etc/hosts

$ ip addr  # 查看网络接口名称、静态IP还是DHCP
$ ls /etc/systemd/network
$ vim /etc/systemd/network/default.network
# vim /etc/systemd/network/20-wired.network
# vim /etc/systemd/network/25-wireless.network

# 配置DNS
$ systemctl status systemd-resolved
$ systemctl enable systemd-resolved  # systemd-networkd的DNS解析依赖systemd-resolved    
# 不推荐手动配置/etc/resolv.conf
# 不推荐使用systemd-resolved的/etc/systemd/resolved.conf配置DNS
# 推荐使用systemd-networkd的/etc/systemd/network/*.network在[Network] DNS配置DNS  # 也可DNS交给DHCP
$ systemctl status systemd-networkd  # 确保是enable
$ systemctl restart systemd-networkd  # DHCP可以使用独立的客户端或内置客户端的网络管理器，我选择网络管理器  
$ ip addr
$ ping archlinux.org
```

##### sudo用户

```bash
# 3.7设置root密码
$ pacman -S sudo
$ passwd
# 添加用户、设置密码、配置sudo权限、配置附属组
$ cat /etc/group
$ cat /etc/passwd
$ useradd -m -G wheel -s /bin/bash nemesis
$ passwd nemesis
$ cat /etc/subuid
$ cat /etc/ # 将用户nemesis添加到附属组users
$ getent group users  # 从/etc/group查询
$ id nemesis  # 查用户的uid、gid、groups
```

##### 配置引导程序

```bash
# 3.8安装引导程序  
# https://wiki.archlinuxcn.org/wiki/GRUB#%E5%AE%89%E8%A3%85_2
pacman -S grub
grub-install --help
grub-install --target=i386-pc /dev/vda  # BIOS  # 安装GRUB到磁盘/dev/vda、/boot/grub/x86_64-efi/
# --target <目标平台名称> [目标磁盘设备(BIOS需要)(/dev/vda) 使用MBR启动扇区] --removable 设备可移动，将grub安装到/esp/EFI/BOOT/BOOTIA32.EFI

# 生成主配置文件/boot/grub/grub.cfg
vim /etc/default/grub  # 取消注释GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3"  # 取消quiet 显示详细启动信息  # loglevel=3只输出比err严重的信息，熟悉后可改为5
grub-mkconfig -o /boot/grub/grub.cfg  # 根据/etc/default/grub、/etc/grub.d/自动生成主配置文件
```

```bash
pacman -S grub efibootmgr
# 挂载ESP分区到/boot/efi
grub-install --help
grub-install --target=x86_64-efi --efi-directory=/boot/efi  --bootloader-id=GRUB  # UEFIx86_664  # 安装/boot/grub/x86_64-efi、安装grub到/esp/EFI/<bootloader-id>/grubx64.efi
#  --efi-directory <ESP挂载路径> --bootloader-id <名称，会安装/boot/efi/EFI/<bootloader-id>/grubx64.efi> --removable 设备可移动，将grub安装到/esp/EFI/BOOT/BOOTX64.EFI
$ install -Dm700 /boot/efi/EFI/GRUB/grubx64.efi /boot/efi/EFI/BOOT/BOOTX64.EFI  # 复制文件  # 如果你更新了UEFI，启动项可能会在更新后丢失。因此可以创建一个“removable”启动项作为后备。

# 生成主配置文件/boot/grub/grub.cfg
vim /etc/default/grub  # 取消注释GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3"  # 取消quiet 显示详细启动信息  # loglevel=3只输出比err严重的信息，熟悉后可改为5
grub-mkconfig -o /boot/grub/grub.cfg  # 根据/etc/default/grub、/etc/grub.d/自动生成主配置文件
```

##### 配置pacman、reflector

配置pacman

```bash
# 配置pacman
vim /etc/pacman.conf  # 参考Linux.md#中文社区仓库 
pacman-key --refresh-keys
pacman -Syyu
```

```/etc/pacman.conf
#
# /etc/pacman.conf
#
# See the pacman.conf(5) manpage for option and repository directives

#
# GENERAL OPTIONS
#
[options]  # 通用选项
# The following paths are commented out with their default values listed.
# If you wish to use different paths, uncomment and update the paths.
#RootDir     = /
#DBPath      = /var/lib/pacman/
#CacheDir    = /var/cache/pacman/pkg/
#LogFile     = /var/log/pacman.log
#GPGDir      = /etc/pacman.d/gnupg/
#HookDir     = /etc/pacman.d/hooks/
HoldPkg     = pacman glibc
#XferCommand = /usr/bin/curl -L -C - -f -o %o %u
#XferCommand = /usr/bin/wget --passive-ftp -c -O %o %u
CleanMethod = KeepCurrent  # # KeepCurrent(推荐 避免清理软件包缓存过度)当前版本 KeepInstalled保留已安装版本
Architecture = auto

# Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
#IgnorePkg   =  # 不更新xxx包
#IgnoreGroup =  # 不更新yyyy软件包组

#NoUpgrade   =
#NoExtract   =

# Misc options
#UseSyslog
Color  # 输出处于TTY彩色输出
#NoProgressBar
CheckSpace  # install之前检查磁盘空间
VerbosePkgLists  # 升级前对比版本
ParallelDownloads = 5  # 并行下载
DownloadUser = alpm
#DisableSandbox

# By default, pacman accepts packages signed by keys that its local keyring
# trusts (see pacman-key and its man page), as well as unsigned packages.
SigLevel    = Required DatabaseOptional  Required必需验证签名，DatabaseOptional不强制验证数据文件.db.zst签名
LocalFileSigLevel = Optional  # 不强制验证本地文件签名
#RemoteFileSigLevel = Required

# NOTE: You must run `pacman-key --init` before first using pacman; the local
# keyring can then be populated with the keys of all official Arch Linux
# packagers with `pacman-key --populate archlinux`.

#
# REPOSITORIES
#   - can be defined here or included from another file
#   - pacman will search repositories in the order defined here
#   - local/custom mirrors can be added here or in separate files
#   - repositories listed first will take precedence when packages
#     have identical names, regardless of version number
#   - URLs will have $repo replaced by the name of the current repo
#   - URLs will have $arch replaced by the name of the architecture
#
# Repository entries are of the format:
#       [repo-name]
#       Server = ServerName
#       Include = IncludePath
#
# The header [repo-name] is crucial - it must be present and
# uncommented to enable the repo.
#

# 同一repo name header上面的Include lines优先级高

#[core-testing]
#Include = /etc/pacman.d/mirrorlist  # 导入配置文件mirrorlist

[core]
Include = /etc/pacman.d/mirrorlist

#[extra-testing]
#Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

# If you want to run 32 bit applications on your x86_64 system,
# enable the multilib repositories as required here.

#[multilib-testing]
#Include = /etc/pacman.d/mirrorlist

#[multilib]
#Include = /etc/pacman.d/mirrorlist

# An example of a custom package repository.  See the pacman manpage for
# tips on creating your own repositories.
#[custom]
#SigLevel = Optional TrustAll
#Server = file:///home/custompkgs
```

Arch Linux 中文社区仓库镜像 [archlinuxcn-mirrorlist-git](https://github.com/archlinuxcn/mirrorlist-repo)

```bash
vim /etc/pacman.conf
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch  # 荷兰
Server = https://mirrors.xtom.hk/archlinuxcn/$arch  # Hong Kong
Server = https://mirrors.cloud.tencent.com/archlinuxcn/$arch  # 腾讯云
Server = https://mirrors.cernet.edu.cn/archlinuxcn/$arch  # 重定向最近的教育网镜像

pacman -Syu  # download database file
pacman -S archlinuxcn-keyring  # 导入 GPG key
pacman -S archlinuxcn-mirrorlist-git  # 获得/etc/pacman.d/archlinuxcn-mirrorlist镜像列表
vim /etc/pacman.d/archlinuxcn-mirrorlist

vim /etc/pacman.conf
[archlinuxcn]
Include = /etc/pacman.d/archlinuxcn-mirrorlist
```

配置reflector

```bash
# https://wiki.archlinuxcn.org/zh-sg/Reflector
pacman -S reflector  # 会覆盖/etc/pacman.d/mirrorlist
reflector --help
# 手动更新 
reflector --verbose -l 35 -p https --sort rate --save /etc/pacman.d/mirrorlist  # 列出35个、使用https、按下载速率排序、保存到/etc/pacman.d/mirrorlist
paman -Syyu
```

##### 配置openssh_sshd

[arch wiki#OpenSSH](https://wiki.archlinux.org/title/OpenSSH) | [官方使用说明](https://www.openssh.com/manual.html) | [sshd-config](https://man.openbsd.org/sshd_config)

```bash
# nemesis配置openssh，自定义端口、强制公钥认证、root不能登录、客户端公钥authorized_keys
ls -lath /etc/ssh
ssh-keygen -A
cat /etc/services  # https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers  # 找个没被使用的端口作为ssh端口
vim /etc/ssh/sshd_config.d/80-custom.conf
```

```bash
# 将id_rsa.pub公钥复制到Server端 `~/.ssh/authorized_keys
# 客户端执行:
ssh-keygen -t ed25519 -b 4096 -C "arch.alpha_work" -f ~/.ssh/id_ed25519_archalpha
ssh-add ~/.ssh/id_rsa_archalpha
ssh-add -L

# 服务端执行:
mkdir ~/.ssh
chmod 700 ~/.ssh
ls -athl  # 校验权限正确
vim ~/.ssh/authorized_keys  # id_ed25519_archalpha.pub
chmod 600 ~/.ssh/authorized_keys
chown $USER:$USER ~/.ssh/authorized_keys
ls -athl  # 校验权限、所属用户组正确
sshd -t
```

```bash
# 服务端执行:
systemctl enable sshd
systemctl status sshd

# 客户端执行:
ssh -p 39xxx username@server_ip
ssh -p 39224 nemesis@47.238.67.168

tcpdump -i any -n src host 101.228.115.196 and port 22
```

**sshd配置**

`/etc/ssh/sshd_config(vps2arch版)` 默认配置，只在第一次设定时解析所以越上面优先级越高

```/etc/ssh/sshd_config
# 导入插入式drop-in配置
Include /etc/ssh/sshd_config.d/*.conf

# This is the ssh client system-wide configuration file.  See
# ssh_config(5) for more information.  This file provides defaults for
# users, and the values can be changed in per-user configuration files
# or on the command line.

# Configuration data is parsed as follows:
#  1. command line options  # ~/.ssh/authorized_keys(用于公钥验证)
#  2. user-specific file
#  3. system-wide file  # /etc/ssh/sshd_config
# 任何一个configuration value只会在第一次解析时设置. 因此host-specific配置值应放配置文件开头，defaults值放末尾.

# Site-wide defaults for some commonly used options.  For a comprehensive
# list of available options, their meanings and defaults, please see the
# ssh_config(5) man page.

# Host *
#   ForwardAgent no
#   ForwardX11 no
#   PasswordAuthentication yes
#   HostbasedAuthentication no
#   GSSAPIAuthentication no
#   GSSAPIDelegateCredentials no
#   BatchMode no
#   CheckHostIP no
#   AddressFamily any
#   ConnectTimeout 0
#   StrictHostKeyChecking ask
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   IdentityFile ~/.ssh/id_ecdsa
#   IdentityFile ~/.ssh/id_ed25519
#   Port 22  # SSHD监听端口
#   Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
#   MACs hmac-md5,hmac-sha1,umac-64@openssh.com
#   EscapeChar ~
#   Tunnel no
#   TunnelDevice any:any
#   PermitLocalCommand no
#   VisualHostKey no
#   ProxyCommand ssh -q -W %h:%p gateway.example.com
#   RekeyLimit 1G 1h
#   UserKnownHostsFile ~/.ssh/known_hosts.d/%k

###############################################################
# custom

# Authentication
#PermitRootLogin no  # yes, no, prohibit-password  # 是否允许root登录
#PubkeyAuthentication yes  # 是否允许公钥验证
#AuthorizedKeysCommand none  # 指定查找用户公钥的程序
#AuthorizedKeysCommandUser nobody  # 指定AuthorizedKeysCommand运行的用户
#PasswordAuthentication yes  # 是否允许密码验证
#PermitEmptyPasswords no  # 当允许密码验证时是否允许空密码
# AuthenticationMethods publickey  # 指定必须通过的身份验证方法  # publickey,password publickey,keyboard-interactive,any

# Listener
#ListenAddress 0.0.0.0  # 多网卡才需要设置
#ListenAddress ::
#AddressFamily any  # any inet inet6

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# UI
#PrintLastLog yes  # 打印之前最后一次登录的日期时间
#Banner none  # 在验证身份之前发送指定文件(banner)内容给远程用户
```

`10-custom.conf`

```30-custom.conf
Port 39xxx
ClientAliveInterval 600  # 超时时间阈值
ClientAliveCountMax 3  # 最大超时次数
PermitRootLogin no

#PasswordAuthentication yes  
#AuthenticationMethods password

#PasswordAuthentication no
#AuthenticationMethods publickey
```

`20-systemd-userdb.conf(vps2arch版)`

```20-systemd-userdb.conf
# SPDX-License-Identifier: LGPL-2.1-or-later
#
# Make sure SSH authorized keys recorded in user records can be consumed by SSH
#
AuthorizedKeysCommand /usr/bin/userdbctl ssh-authorized-keys %u
AuthorizedKeysCommandUser root
```

`99-archlinux.conf(vps2arch版)`

```99-archlinux.conf
# sshd_config defaults on Arch Linux
KbdInteractiveAuthentication no
UsePAM yes
PrintMotd no
```

###### 问题

问题1：

```bash
ssh -T nemesis@aa.bbb.cc.ddd
ssh -p 22 nemesis@aa.bbb.cc.ddd
Connection closed by 47.238.67.168 port 22
```

原因：机场禁用22端口
解决1：修改sshd监听端口(推荐)
解决2：Clash->订阅->编辑规则->添加前置规则

```
prepend:
  - 'DST-PORT,22,DIRECT'
append: []
delete: []
```

问题2

```bash
systemctl restart sshd
Job for sshd.service failed because the control process exited with error code.
See "systemctl status sshd.service" and "journalctl -xeu sshd.service" for details.
```

原因：sshd配置错误

```bash
sshd -t  # 检查错误
```

**ssh配置**

`21-systemd-ssh-proxy.conf`

```20-systemd-ssh-proxy.conf
# SPDX-License-Identifier: LGPL-2.1-or-later
#
# Allow connecting to the local host directly via ".host"
Host .host machine/.host
        ProxyCommand /usr/lib/systemd/systemd-ssh-proxy unix/run/ssh-unix-local/socket %p
        ProxyUseFdpass yes
        CheckHostIP no

# Make sure unix/* and vsock/* can be used to connect to AF_UNIX and AF_VSOCK paths.
# Make sure machine/* can be used to connect to local machines registered in machined.
#
Host unix/* vsock/* machine/*
        ProxyCommand /usr/lib/systemd/systemd-ssh-proxy %h %p
        ProxyUseFdpass yes
        CheckHostIP no

        # Disable all kinds of host identity checks, since these addresses are generally ephemeral.
        StrictHostKeyChecking no
        UserKnownHostsFile /dev/null
```

###### 配置ssh-agent

cmd and powershell

```powershell
Get-Service -Name ssh-agent  # 获取服务dispaly名称OpenSSH Authentication Agent
Get-Service -Name ssh-agent | Set-Service -StartupType Automatic  # 设置OpenSSH Authentication Agent服务自动启动
Start-Service ssh-agent
ssh-add ~/.ssh/id_ed25519_github
ssh-add -L
```

git bash

[github#git for win 自动启动ssh-agent](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows)

```bash
vim ~/.bashrc
source ~/.bashrc
ssh-add -L

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

##### 配置snapper

```bash
# 配置snapper(删除/.snapshots/子卷、修改/etc/fstab自动挂载 mount -a重新挂载、修改配置、安装snap-pac grub-btrfs)
ll -a /  # 查看是否存在.snapshots
sudo pacman -S snapper snap-pac grub-btrfs
snapper list-configs  # 查看snapper配置文件所在
df -h  # 查看@snapshots挂载情况
# umount /.snapshots  # 不需要
# rm -r /.snapshots  # 不需要，应该让.snapshots挂载到@snapshots
snapper -c root create-config /  # 如果.snapshots每被挂载，在创建配置的同时会创建/.snapshots/子卷
df -h  # 查看.snapshots挂载情况
btrfs subvolume list /
btrfs subvolume delete /.snapshots/
vim /etc/fstab
# /dev/vda2  /.snapshots    btrfs  subvol=@snapshots,noatime,compress=zstd,space_cache=v2,x-mount.mkdir 0 0
# 分区 子卷的挂载点 文件系统 subvol=@子卷,参数 dump命令参数 fsck命令参数
mount -a 
vim /etc/snapper/configs/root
# TIMELINE_MIN_AGE="1800"  # 30min
# TIMELINE_LIMIT_HOURLY="6"
# TIMELINE_LIMIT_DAILY="7"
# TIMELINE_LIMIT_MONTHLY="0"
# TIMELINE_LIMIT_YEARLY="0"
# SUBVOLUME="/@snapshots"  # 可能不需要
systemctl enable --now snapper-timeline.timer
systemctl enable --now snapper-cleanup.timer
systemctl list-unit-files --type=service  # 查看是否有grub-btrfs.timer、grub-btrfsd.service
systemctl enable --now grub-btrfsd
# systemctl enable --now grub-btrfs.timer
# systemctl enable --now grub-btrfsd.service
# grub-mkconfig -o /boot/grub/grub.cfg  # 刚开始不需要执行
```

##### 配置swapfile

```bash
# btrfs配置swapfile
btrfs subvolume create /swap  # 创建子卷存储swapfile
btrfs filesystem mkswapfile --size 4g --uuid clear /swap/swapfile  # 创建交换文件
swapon /swap/swapfile  
vim /etc/fstab
```

```bash
# 配置swapfile
$ fallocate --help
$ fallocate -l 8G /swap/swapfile  # 给文件预分配空间  # 也可用dd(不推荐)
# dd if=/dev/zero of=/swap/swapfile bs=1M count=512 status=progress  # 创建空文件swapfile
$ chmod 0600 /swap/swapfile
$ ls -athl
$ mkswap -U clear /swap/swapfile  # 格式化
$ swapon /swap/swapfile  # 启用交换文件
$ echo "/swap/swapfile none swap defaults 0 0" >> /etc/fstab
```

##### 配置内核zRAM

```bash
# 配置zRAM
$ echo 0 > /sys/module/zswap/parameters/enabled
$ echo zram >> /etc/modules-load.d/zram.conf
$ pacman -S zram-generator
$ vim /etc/systemd/zram-generator.conf  # 参考Linux.md#zram-generator
$ reboot
$ systemctl daemon-reload
$ systemctl enable --now systemd-zram-setup@zram0.service

$ modinfo zram
$ lsmod | grep zram  # 看模块是否加载
$ systemctl status systemd-zram-setup@zram0.service  # 看服务是否启动
$ free -m  # 查看内存swap空间状态(swap=swapfile+zram)
$ zramctl  # 查看zRAM状态
NAME       ALGORITHM DISKSIZE DATA COMPR TOTAL STREAMS MOUNTPOINT
/dev/zram0 zstd        446.5M   4K   69B   20K       2 [SWAP]
$ cat /proc/swaps  # 查看虚拟swap空间
Filename                                Type            Size            Used            Priority
/swap/swapfile                          file            8388604         0               -2
/dev/zram0                              partition       457212          0               100  # 优先级高
```

##### 安装 neofetch

```bash
pacman -S neofetch
neofetch  # 打印系统信息
```

##### 收尾

```bash
# check
timedatectl  # 确保时间、时区、NTP服务正确
ping archlinux.org
cat /proc/swaps
systemctl list-unit-files --type=service  
systemctl list-unit-files --type=service --state enable  # 自启动服务：systemd-networkd、systemd-resolved、sshd  # snapper-timeline.timer、snapper-cleanup.timer
$ free -h  # 看内存占用
               total        used        free      shared  buff/cache   available
Mem:           893Mi       269Mi       543Mi       2.1Mi       214Mi       624Mi
Swap:          8.4Gi          0B       8.4Gi
$ neofetch  # 看OS内存占用
                   -`                    nemesis@Alpha 
                  .o+`                   ------------- 
                 `ooo/                   OS: Arch Linux x86_64 
                `+oooo:                  Host: Alibaba Cloud ECS pc-i440fx-2.1 
               `+oooooo:                 Kernel: 6.13.1-arch1-1 
               -+oooooo+:                Uptime: 15 hours, 41 mins 
             `/:-:++oooo+:               Packages: 167 (pacman) 
            `/++++/+++++++:              Shell: bash 5.2.37 
           `/++++++++++++++:             Resolution: 1024x768 
          `/+++ooooooooooooo/`           Terminal: /dev/pts/0 
         ./ooosssso++osssssso+`          CPU: Intel Xeon Platinum (2) @ 2.500GHz 
        .oossssso-````/ossssss+`         GPU: 00:02.0 Cirrus Logic GD 5446 
       -osssssso.      :ssssssso.        Memory: 133MiB / 893MiB 
      :osssssss/        osssso+++.
     /ossssssss/        +ssssooo/-                               
   `/ossssso+/:-        -:/+osssso+-                             
  `+sso+:-`                 `.-/+oso:
 `++:.                           `-/+/
 .`                                 `/

# 4重新启动计算机
exit  # 退出chroot环境
umount -R /mnt  # 手动卸载被挂载的分区，用于查看是否有繁忙分区
reboot  # systemd会自动卸载被挂载的分区
```

##### 不打算做的

```
# 未配置的
git neovim
# 不打算安装的
starship  # 感觉不需要git prompt 
```

#### vps2arch脚本(推荐)

vps2arch Bash脚本
本质：使用`Bootstrap tarball chroot` 中的 `ld.so` 来启动 `chroot` 工具 + 配置ssh + 配置grub + root密码
特性：只格式化根分区，不修改其他分区

[tinyurl](https://gitlab.com/drizzt/vps2arch/-/blob/master/vps2arch)
[felixonmars(推荐)](https://github.com/felixonmars/vps2arch)：get_country，配置 Reflector

```bash
# apt-get install ca-certificates  # 解决Debian缺少CA公钥证书(.pem)
$ sudo wget https://felixc.at/vps2arch
# sudo wget http://tinyurl.com/vps2arch
$ sudo chmod +x vps2arch
$ ./vps2arch -h
usage: vps2arch [options]

  Options:
    -b (grub|syslinux)           Use the specified bootloader. When this option is omitted, it defaults to grub.
    -n (systemd-networkd|netctl) Use the specified networking configuration system. When this option is omitted, it defaults to systemd-networkd.
    -m mirror                    Use the provided mirror (you can specify this option more than once).

    -h                           Print this help message

    Warning:
      On OpenVZ containers the bootloader will be not installed and the networking configuration system will be enforced to netctl.
# curl  https://mirrors.tuna.tsinghua.edu.cn/archlinux/core/os/x86_64/core.db
$ sudo ./vps2arch -m https://mirror.xtom.com.hk/archlinux
Hi,
your VM has successfully been reimaged with Arch Linux.
This script configured grub as bootloader and systemd-networkd for networking.
When you are finished with your post-installation, you'll need to reboot the VM the rough way:
# sync ; reboot -f
Then you'll be able to connect to your VM using SSH and to login using your old root password (or "vps2arch" if you didn't have a root password).

# 进行其他配置
$ su root
$ sync ; reboot -f

$ pacman -Qe
base 3-2
efibootmgr 18-3
grub 2:2.12-3
linux 6.12.10.arch1-1
lvm2 2.03.30-1
openssh 9.9p1-2
reflector 2023-3
vim 9.1.1065-1

$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    254:0    0   30G  0 disk 
├─vda1 254:1    0    1M  0 part 
├─vda2 254:2    0  200M  0 part /boot/efi
└─vda3 254:3    0 29.8G  0 part /

$ systemctl list-unit-files --type=service --state=enabled
UNIT FILE                            STATE   PRESET  
aegis.service                        enabled disabled
getty@.service                       enabled enabled 
sshd.service                         enabled disabled
systemd-network-generator.service    enabled enabled 
systemd-networkd-wait-online.service enabled enabled 
systemd-networkd.service             enabled enabled 
systemd-timesyncd.service            enabled enabled 

7 unit files listed.
```

### 无需本地存储设备

#### 2copytoram(OOM)(推荐)

关键词：GRUB引导iso(GRUB's ISO boot capabilities)、copytoram安装archlinux
initramfs：是RAM内存文件系统
需要 >=1.5GiB 的内存

[iso结构](https://mirror.xtom.com.hk/archlinux/iso/latest/arch/)
grub启动界面：[bilibili#腾讯云安装archlinux+btrfs](https://www.bilibili.com/video/BV1rq4y1L7cB/https://www.bilibili.com/video/BV1rq4y1L7cB/)
grub配置：[若依Blog#grub2引导iso](https://blog.lilydjwg.me/2014/2/2/load-arch-linux-iso-with-grub2.42632.html) | [阿里云安装archlinux](https://brothereye.cn/blog/linux/ali-ecs2arch/)

```bash
$ cd /
$ sudo curl -O https://mirror.xtom.com.hk/archlinux/iso/latest/archlinux-x86_64.iso
$ sudo mount -o loop,ro archlinux-x86_64.iso /mnt  # 挂载iso为loop0  # 将iso中内容拷贝到/就不要了
$ fdisk -l
$ sudo cp -r /mnt/* /  # 在启动过程中系统可以识别ISO镜像中的GRUB引导程序，在GRUB的启动菜单可选择从该ISO镜像启动  # 也可以不用解压的方式，用修改GRUB配置从ISO启动  
$ du -sh /mnt/arch/x86_64/airootfs.sfs  # 查看airootfs.sfs大小
825M    airootfs.sfs
$ sudo umount /mnt
$ lsblk -l
$ sudo e2label /dev/vda3 archiso  # 给iso所在的文件系统设置标签
$ sudo vim /boot/grub/grub.cfg  # set timeout=60
# 重启实例
```

重启实例，通过VNC方式(阿里轻量的救援模式)连接实例，进入GRUB启动菜单(阿里云按ESC)
**VNC(Virtual Network Computing)**: 是一种性能不如RDP的远程桌面协议，支持连接启动中、停止中的实例

```bash
# 推荐
ls /
linux /arch/boot/x86_64/vmlinuz-linux archisobasedir=arch archisolabel=archiso copytoram=y
# archisolabel 指定iso所在的文件系统标签，也可以通过img_dev指明iso所在的文件系统
# copytoram 将rootfs复制到内存, 这样可以解除系统盘占用以格式化分区
# img_loop 指明iso文件在loop0里的绝对位置
# archisobasedir
initrd /arch/boot/x86_64/initramfs-linux.img  # 初始化initramfs(RAM内存文件系统)、临时根文件系统用于内核挂载真正的根文件系统
boot  # 启动initramfs，开始系统的引导过程
```

(boot 没成功 can't access tty: job control turned off，copytoram有问题)

> **Out of memory**
> Killed
> ERROR:while copy '/run/archiso/bootmnt/arch/x86_64/airootfs.sfs'to '/run/archiso/copytoram/airootfs.sfs'
> sh:can't access tty;job control turned off

```bash
# 修改GRUB配置从ISO启动
$ sudo vim /boot/grub/grub.cfg
set timeout=60
menuentry 'ArchISO' --class iso { 
  set imgdevpath=/dev/vda3  # NOTE
  set isofile=/archlinux-x86_64.iso  # NOTE
  loopback loop0 $isofile
  echo "GRUB: starting from dev=$imgdevpath iso=$isofile..."
  linux (loop0)/arch/boot/x86_64/vmlinuz-linux archisolabel=archiso img_dev=$imgdevpath img_loop=$isofile copytoram=y archisobasedir=arch  # NOTE
  initrd (loop0)/arch/boot/x86_64/initramfs-linux.img  # NOTE
}
```

#### Netboot

[arch wiki#网络引导](https://wiki.archlinuxcn.org/wiki/%E7%BD%91%E7%BB%9C%E5%BC%95%E5%AF%BC) | [netbootxyz官网](https://netboot.xyz/) | [netbootxyz github](https://github.com/netbootxyz/netboot.xyz) | [archlinux#PXE](https://wiki.archlinuxcn.org/wiki/PXE)

netbootxyz 基于 PXE(Preboot Execution Environment 允许计算机通过网络引导操作系统)
System Requirements: at least 4GB of RAM is recommended

> **方案三：netboot.xyz**
>
> 1. **引导到 netboot.xyz**：
>    * **通过网络引导**：使用 iPXE 或其他网络引导方式启动 netboot.xyz。
>    * **选择操作系统**：在 netboot.xyz 菜单中，选择 `Linux Network Installs`，然后选择 `Arch Linux`。
> 2. **安装 Arch Linux**：
>    * **加载安装程序**：netboot.xyz 会自动下载并加载 Arch Linux 安装程序。
>    * **按照官方安装步骤进行安装**：从分区、格式化、挂载到安装基本系统，按照官方指南进行操作。
> 3. **配置并完成安装**：
>    * **配置系统**：设置主机名、网络、时区等。
>    * **安装引导加载程序**：根据系统环境，安装并配置引导加载程序。
>    * **设置 root 密码**：使用 `passwd` 设置 root 用户密码。
>    * **重启系统**：完成安装后，重启进入新的 Arch Linux 系统。

## 其他方案

方案一：[archstrap Bash脚本](https://github.com/wick3dr0se/archstrap)

## 分区

| 分区表格式 `fdisk -l`         | 启动模式                    |                                                              | 缺点            | 优点              |      |
| ----------------------------- | --------------------------- | ------------------------------------------------------------ | --------------- | ----------------- | ---- |
| MBR(Master Boot Record) (dos) | BIOS模式                    | 不需要单独的BIOS引导分区，GRUB可以安装在主分区<br />如果使用特殊文件系统还是建议使用/boot引导分区+通用文件系统如FAT32 | 仅支持4个主分区 | 大多数Linux都支持 |      |
| GPT (GUID) (推荐)             | UEFI模式、BIOS模式(使用MBR) | 需要1MiB的BIOS引导分区[GNU GRUB](https://www.gnu.org/software/grub/manual/grub/html_node/BIOS-installation.html#BIOS-installation) |                 | 支持UEFI          | 推荐 |

| 分区                                       | 文件系统推荐 | 大小             | 挂载点     |
| ------------------------------------------ | ------------ | ---------------- | ---------- |
| BIOS boot 分区 (bios_grub)                 | 不需要格式化 | 1M               | 不需要挂载 |
| EFI System 分区 (ESP efi_system_partition) | fat32        | 200M (100-500MB) | /boot/efi  |
| 根分区                                     | Btrfs        | 100G             |            |
|                                            |              |                  |            |
| boot分区                                   | ext4         | 1G               |            |
| swap分区(可用swapfile替代)                 |              |                  |            |
|                                            |              |                  |            |

[引导加载程序](http://wiki.archlinuxcn.org/wiki/Arch_%E7%9A%84%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B#%E5%BC%95%E5%AF%BC%E5%8A%A0%E8%BD%BD%E7%A8%8B%E5%BA%8F)：GRUB(需要BIOS引导分区，但其实也可以安装到主分区)

分区工具：parted、fdisk、lsblk

格式化分区：mkfs.ext4、mkswap(交换分区)、mkfs.fat(EFI分区)



# TODO

```bash
make defconfig  # 生成.config(默认)
make compile_commands.json
```





