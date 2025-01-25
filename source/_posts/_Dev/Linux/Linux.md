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

  - 当用户程序运行时，它处于用户态。在用户态下，程序只能**访问受限资源和执行受限操作**。

  - 用户态的程序**不能直接访问操作系统内核的功能和硬件资源**，例如直接访问硬件设备、修改内核数据结构等。

  - 用户态下的程序**执行速度相对较快**，因为不需要处理特权指令和保护操作系统资源。

* **内核态**（Kernel Mode）：

  - 内核态是操作系统内核执行时的特权级别。在内核态下，操作系统具有对系统硬件和资源的完全控制权限。

  - 内核态的程序可以执行**特权指令**、直接访问硬件设备和修改内核数据结构。

  - 内核态下的程序执行速度相对较慢，因为需要处理特权指令和进行安全检查。



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

todo：有空学习 https://www.zhihu.com/question/419924138



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

###  文件和文件夹操作

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
|                                                |                                                              | y         | 从服务器下载最新的软件包数据库(即软件列表清单core,extra,community,archlinuxcn)，单y短时内多次运行可能不同步 |
|                                                |                                                              | u         | 升级已安装的包，不包括不在软件库中的本地包                   |
|                                                |                                                              | yy        | 强制下载最新的软件列表清单，即使已经是最新                   |
| -Q                                             | 查询本地库                                                   | s <regex> | 在本地仓库搜索对应的包                                       |
|                                                |                                                              | e         | 明确安装                                                     |
| -R                                             | 删除                                                         | s         | 删除不需要的依赖项                                           |
| -U                                             | `pacman -U 本地软件包路径.pkg.tar.xz` <br />`pacman -U http://www.example.com/repo/example.pkg.tar.xz` |           |                                                              |

`/etc/pacman.conf`

```/etc/pacman.conf
[options]
Architecture = auto
IgnorePkg   = fakeroot
Color
ParallelDownloads = 5
SigLevel    = Required DatabaseOptional
LocalFileSigLevel = Optional

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

[community]
Include = /etc/pacman.d/mirrorlist

[archlinuxcn]
# The Chinese Arch Linux communities packages.
# SigLevel = Optional TrustedOnly
SigLevel = Optional TrustAll
Server   = http://repo.archlinuxcn.org/$arch
```



##### 中文社区仓库

[archlinuxcn 中文社区仓库](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)	|	[软件仓库](http://repo.archlinuxcn.org)

非官方软件包：就是软件包是开发者签名的，Arch仓库并不信任，用户需要用自己的密钥对开发者公钥签名，使arch信任公钥
需要安装 pacman-key，将开发者公钥导入 pacman-keyring

| 命令 pacman <operation> [options] [package(s)] | 释义                                      |
| ---------------------------------------------- | ----------------------------------------- |
| pacman-key --init                              | 初始化本地keyring密钥环，并生成系统主密钥 |
| pacman-key --populate                          | 验证主密钥                                |
| pacman-key --refresh-keys                      | 刷新密钥，需要设置代理，最好使用德国代理  |



##### AUG用户仓库

[AUR 用户软件仓库](https://wiki.archlinuxcn.org/wiki/Arch_%E7%94%A8%E6%88%B7%E8%BD%AF%E4%BB%B6%E4%BB%93%E5%BA%93_(AUR))：需要自己更新软件包	|	[AUR 助手]()



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



#### sudo功能

  可配置普通用户的权限.

  vim /etc/sudoers : 修改配置.

  user1   ALL=(ALL)   NOPASSWD: ALL  

  sudo service iptables status  : 借助sudo功能帮助user1进行权限操作.



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

[文档](https://learn.microsoft.com/zh-cn/windows/wsl/)	|	[**最佳实践**](https://dowww.spencerwoo.com/1-preparations/1-0-intro.html)

TODO: 有空继续看 目前看到这里 [Dev on WSL](https://dowww.spencerwoo.com/3-vscode/3-4-c_cpp.html#%E5%AE%89%E8%A3%85-c-c-%E6%8F%92%E4%BB%B6) 



### 配置

[文档#wsl-config](https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config)

全局配置：`%UserProfile%/.wslconfig`(不用)
单发行配置：`/etc/wsl.conf`

```/etc/wsl.conf
[boot]
systemd=true

[automount]
enabled = true
options = "metadata"
mountFsTab = true

[network]
hostname = "WSL-ARCH"

# 能解决win目录内ls背景色问题，但是不推荐，建议通过 ~/.zshrc 解决
# 问题原因：挂载win权限问题
#[automount]
#enabled = true
#root = /mnt/
#options = "metadata,umask=22,fmask=111"
#mountFsTab = true

[wsl2]
networkingMode=NAT  # mirrored
dnsTunneling=true
firewall=true
autoProxy=true

[experimental]
bestEffortDnsParsing=true
```



存储位置：`E:\Applications\Scoop\persist\archwsl\data\ext4.vhdx`，240525目前占用5.5G

```bash
# 优化 WSL 2 虚拟磁盘占用空间
Optimize-VHD -Path E:/Applications/Scoop/persist/archwsl/data/ext4.vhdx -Mode Full
```



### 网络互访

WSL IP：**127.0.0.1**、`cat /etc/resolv.conf`内的固定IP、wsl-hostname.local

宿主机IP：**DESKTOP-2DAQ5TI**

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

```bash
# WSL环境执行Win程序
explorer.exe .
cat pacman.conf | clip.exe  # 复制WSL内容到Win，好用

#Win环境执行WSL命令
wsl ls 
```



### 远程开发

[文档](https://www.jetbrains.com/help/idea/2023.2/remote.html)



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
| /etc/group                                                   | 配置 组名:密码占位符(x或*):组ID(GID):组成员列表（逗号分隔）  |
| /etc/hosts                                                   | DNS解析，配置映射关系hostname:IP                             |
| **/etc/hostname**                                            | 配置 hostname                                                |
| /etc/my.cnf                                                  | mysql配置文件                                                |
| **/etc/passwd**                                              | 配置 用户名:密码占位符(x或*):用户ID(UID):组ID(GID):用户描述:家目录:登录Shell |
| **/etc/profile**                                             | 环境变量，注释符 #，<br />更改完环境变量使用source profile命令 |
| /etc/rc.d/rc.local                                           | 配置开机启动项                                               |
| /etc/resolv.conf                                             | DNS服务器地址配置                                            |
| /etc/shadow                                                  | 存储用户加密密码信息                                         |
| /etc/yum.repos.d/xx                                          | yum.repo源                                                   |
| /etc/sysconfig/network                                       | 配置hostname                                                 |
| /etc/sysconfig/network-scripts/ifcfg-eth0_or_33              | 网络接口配置文件，修改需要root权限                           |
| /home                                                        | 用于存储非root的其他用户根目录.<br />\home\<UserName>\XXX    |
| /lib                                                         | lib 静态链接库，是系统中的运行程序和内核模块<br />DLL 动态链接库，是程序接口，当需要时才会调用的模块 |
| /mnt                                                         | 挂载目录.                                                    |
| /opt                                                         | 额外安装的可选的应用程序安装位置.<br />/opt 目录是存放某些大型软件或者某些特殊软件的目录，比如谷歌浏览器(Google Chrome)默认就是安装在/opt中。<br />jdk、tomcat |
| **/opt/modules/software**                                    | 软件的安装目录                                               |
| /opt/modules/source                                          | 软件的安装包的目录                                           |
| /proc                                                        | 虚拟文件系统.当前内存中的映射文件.启动时,产生,关机时消失.    |
| /proc/$pid/environ                                           | 存储进程环境变量。环境变量是一组键值对，用于存储进程运行时所需的配置信息、系统路径、用户设置等。 |
| /root                                                        | 超级管理员根目录.                                            |
|                                                              |                                                              |
| ~                                                            | 当前用户的主目录 /username，~目录下有一些隐藏 .用户配置文件  |
| ~/.bashrc                                                    | 每次打开shell触发<br />bash用户,配置文件<br />执行命令 source ~/.zshrc 来生效修改的配置，而不需要重新登录<br /><br />CMD：链接器配置文件，是存放链接器的配置信息的，我们简称为命令文件,该文件的作用是指明如何链接程序的。<br />.xxrc：rc 是run command 表示与运行终端有关的配置 |
| ~/.bash_profile                                              | 登陆时触发                                                   |
| ~/maven_resp                                                 | maven的本地仓库（默认是在根目录的.m2，不是太建议）           |
| ~/software下                                                 | 软件的安装包                                                 |
| ~/app                                                        | 软件的安装目录                                               |
| ~/data                                                       | 要使用的数据                                                 |
| ~/lib                                                        | 作业jar存放的目录，静态库                                    |
| ~/shell                                                      | 要使用的脚本                                                 |
| /sbin                                                        | super binary，存储 root 用户才可以执行的高级二进制可执行命令文件 |
| **/usr**                                                     | 系统级的目录，可以理解为`C:/Windows/`<br />存放命令、帮助文件、安装的Linux发行版官方提供的软件包等 |
| **/usr/local**<br /><br />我在/usr/local/modules/java 下安装了JDK | **用户级的程序目录**，可以理解为`C:/Progrem Files/`。<br />存放用户通过源码包自编译安装的软件，即不是通过“新立得”或apt-get安装的软件。<br /><br />Oracle的不能用root安装才安装到这个目录 |
| /usr/share                                                   | 存放系统公用程序和共享数据和帮助文档                         |
| /var                                                         | 特别是/var/log 子目录. 各大程序执行日志存储目录.             |
|                                                              |                                                              |



## 软件

| 我安装的  | 释义                                                         |
| --------- | ------------------------------------------------------------ |
| wget      |                                                              |
| aria2c    |                                                              |
| git       |                                                              |
| openssh   | 实现了应用层的 SSH 协议，能用于免密登录                      |
| zsh       |                                                              |
| oh my zsh |                                                              |
| unzip     |                                                              |
| nmap      | 是 netcat(nc) 的高级替代。可以使用 ncat 命令，用于配置 SSH 通过代理访问目标主机 |
|           |                                                              |
|           |                                                              |



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
| cat                                                          | less更好用。打印全部文本                                     |                                                              |
| chgrp 组名 文件                                              | 改变文件的用户组用命令                                       |                                                              |
| chown 拥有者名称 文件                                        | 更改文件拥有者 -R 递归                                       |                                                              |
| chsh                                                         | 改变SHELL(bash、zsh)<br />cat /etc/shells 查看目前支持的shell<br />echo $SHELL 打印当前SHELL |                                                              |
| clear                                                        | 清屏，向下一页                                               |                                                              |
| curl [options...] <url>                                      | client URL，发送 http 请求<br />访问 cip.cc 可用于测试是否使用代理<br/> | -X Get 指定HTTP请求方法<br />-A 指定User-Agent<br />-b 指定Cookie<br />-d 指定Post请求正文<br />-F 上传二进制文件<br />-k 跳过 SSL 检测<br />-o 将响应保存为文件<br />-x 指定代理<br />-f 快速失败，根据HTTP响应状态码返回成功0失败非0 |
| df                                                           | 显示文件系统使用情况                                         |                                                              |
| echo $HADOOP_HOME                                            | 可以看一些软件的安装路径                                     |                                                              |
| env                                                          | 查看环境变量<br />env \|grep -i poxy #查看系统代理配置情况   |                                                              |
| fdisk                                                        | 磁盘分区                                                     |                                                              |
| **find** [path] [options] expression                         | 在指定目录下递归查找文件路径<br />find / -name "head*"<br />-iname 对大小写不敏感 |                                                              |
| firewall-cmd --list-ports                                    | 查看firewall已经开放的端口                                   |                                                              |
| firewall-cmd --zone=public --*add*-*port=80/tcp* --permanent | 开启端口，–zone #作用域  添加端口，端口/通讯协议  永久生效，没有此参数重启后失效 |                                                              |
| firewall-cmd --reload                                        | #重启firewall                                                |                                                              |
| free -h                                                      | 显示内存使用情况                                             |                                                              |
| **grep** [option] pattern [file]                             | 查找文件里符合条件的字符串的整行内容。Globally search a Regular Expression and Print<br />grep “txt” “head*”<br />不指定file，grep "txt" 会等待输入从键盘获取stdin<br />find path \| grep “exp” 正则查找文件<br />grep -o 只输出了匹配到的部分，而不是整行的内容。<br />grep -v 过滤掉相关字符串的内容。可以通过管道操作符组合使用 |                                                              |
| groupadd xxgroup                                             | 创建新的用户组                                               | -r 系统用户组                                                |
| groups  user1  /  passwd  password2                          | 位于root状态下，查看user1用户的所属组                        |                                                              |
| hostname  xxxx                                               | 临时修改hostname主机名                                       |                                                              |
| ~~ifconfig~~                                                 | ip add 更好用                                                |                                                              |
| **info**                                                     | 显示帮助文档<br />q 退出                                     |                                                              |
| ip                                                           | 显示配置网卡参数                                             | addr 看所有网卡对应的 IP<br />route 查看路由表信息           |
| jps                                                          | 由JDK提供，查看Java进程                                      |                                                              |
| kill -9 Pid                                                  | 强制杀死进程                                                 |                                                              |
| **less**                                                     | 查看文件，h显示帮助                                          |                                                              |
| ls -lath                                                     | 展示当前目录下的文件与目录.并根据颜色区分类型.<br />–a显示隐藏文件<br />-i显示inode编号 <br /> -l 附加显示详细信息<br />-t 按时间排序<br />-r 反序<br />-h 文件大小单位变为kb<br />白(一般文件)、蓝(目录)、浅蓝(链接文件)、绿(可执行)、红(压缩)<br />黄背景(Set Group ID)、红背景(Set User ID)、绿背景(粘滞位) |                                                              |
| lsblk -f                                                     | 显示文件系统                                                 |                                                              |
| man                                                          | 可以用info命令替代                                           |                                                              |
| mkdir myfloder                                               | 创建空目录.  <br />mkdir –p myfloder:  如果已经存在,也不报错提示. <br />mkdir无法创建多层目录,所以可以用 : mkdir –p a/b/c ) |                                                              |
| ~~more~~                                                     | less命令更好用                                               |                                                              |
| nc -l 9001                                                   | 开放9001TCP，等待客户端连接，可以传输字符串                  |                                                              |
| **neofetch**                                                 | new fetch 显示系统信息和Logo                                 |                                                              |
| ~~netstat -tulpn~~ 弃用                                      | 用ss命令更好用，查看占用的端口                               |                                                              |
| ping                                                         | ping 指定主机 **网络层ICMP**                                 |                                                              |
| ps -ef \| grep “tomcat” \| grep -v “grep”                    | 查看当前时刻活动进程信息<br />root      21772  21674  0 15:59 pts/3    00:00:00 grep --color=auto tomcat<br />UID         PID   PPID  C STIME TTY  TIME CMD<br />PPID：父进程的<br />C：CPU使用的资源百分比<br />TTY：与进程关联的终端（tty）<br />TIME：使用掉的 CPU 时间<br />CMD：所下达的指令名称 | –e 所有进程<br />-f 完整格式<br />--forest 进程树            |
| pstree [pid]                                                 | 显示进程树                                                   | -h 高亮当前进程及其父进程<br />-a 显示命令行参数<br />-l 不截断长行<br />-p 显示pid<br />-s 显示指定进程的父进程 |
| rm                                                           | 删除xxyy文件                                                 | -r<br />-f force 忽略文件不存在时的删除失败提示              |
| rpm -ivh xxx.rpm<br />rpm -qa \| grep xxxx                   | RedHat Package Manager<br />只能安装已经下载到本地机器上的 rpm 包，没有解决软件包依赖问题。<br />参数 i-install、v-verbose可视、u-Update、q-Query、p-Package、l-list、e删除、a-all、h-hash哈希后人可读、--nodeps忽略依赖关系<br /><br />yum remove 若你要移除的包被别的软件包需要的话，yum会把其他软件包一起移除<br />rpm -e 会告诉你该包被别的包需要，所以无法移除 |                                                              |
| **rz** -ebq                                                  | 上传到当前目录  <br />e 表示强制escape转义所有控制字符，b二进制传输，q 安静quiet没有进度条<br />如过路径有中文则必须用e参数 |                                                              |
| **scp -rq 主机名:**                                          | 基于ssh远程连接协议实现复制. 一般需要密码.<br/>本地到远程 : scp /home/test/\*abc        用户名@10.1.1.2:/home/root<br/>远程到本地 : scp 用户名@10.1.1.2:/home/root/\*abc       /home/test<br/>远程到远程 : scp 用户名1@10.1.1.2:/home/root/*abc       用户名2@10.1.1.3:/home/root<br/>（参数：r复制递归目录，v显示进度，p不显示进度）<br /><br />可以配置SHH |                                                              |
| **sed** [option] 'sed command' filename                      | stream editor流式编辑器，适合对文本的行内容进行处理，替换、删除、移动、搜索数据<br />`sed 's/^Str\.&/\;/g' filename`：在终端将Str.替换成;   s表示处理字符串 g表示全文替换否则只替换行首次匹配<br />`sed -i '/^ *$/d' filename`：在文件中删除空行，d表示删除行<br />`sed -i '/xxx/d' filename`：删除xxx所在行<br /><br />s/ 替换字符<br />/d 删除 | -i 在文件中修改                                              |
| sh -c [string]                                               | 执行脚本                                                     |                                                              |
| shutdown [option] [time] [message]                           |                                                              |                                                              |
| shutdown -h now                                              | 马上关机                                                     |                                                              |
| **source profile**                                           | 使环境变量 profile 文件生效<br />读profile并在当前shell执行  |                                                              |
| ss                                                           | 显示sockets<br />l 显示正在监听的<br />t 只显示TCP sockets   |                                                              |
| **ssh 用户名@ip**                                            | ssh登录远程主机                                              | -T                                                           |
| sudo -i                                                      | 切换到root用户                                               |                                                              |
| su xxyy                                                      | 换到普通用户                                                 |                                                              |
| systemctl [OPTIONS...] COMMAND ...                           | 查询或发送控制命令到系统管理器<br />UNIT服务单元：network,mysql,firewalld,mongod,mysqld<br />q 退出 | status [PATTERN...\|PID...] 显示正在运行的该服务状态<br />start UNIT...<br />stop UNIT...<br />reload UNIT... 重载配置文件<br />restart UNIT... 重启服务<br />disable UNIT... 开机不启动<br />list-unit-files --type=service 列出所有服务单元文件 |
| sz xxyy                                                      | 导出xxyy到本地快速访问download中                             |                                                              |
| tail -n 20 filename                                          | 查看文件最后末尾20行                                         |                                                              |
| tar –zcvf a.tar.gz<br />tar –xzvf a.tar.gz -C /target_dir    | vf<br />c压缩、x解压、t查看压缩包内容不解压<br />z：gzip、j：bzip压缩算法 |                                                              |
| telnet                                                       | 远程登入，可以测试端口的连通性 **应用层**                    |                                                              |
| tracepath [options] <destination>                            | 路径探测跟踪                                                 |                                                              |
| ulimit -n                                                    | 打印最打文件描述符数量(Linux一切皆文件)                      |                                                              |
| **uname -a**                                                 | 打印所有系统信息，包括linux版本，主机名                      |                                                              |
| useradd  xxuser                                              | 创建新的用户<br />sudo useradd -m -g users -s /bin/bash nemesis<br />sudo passwd nemesis | -r 系统用户                                                  |
| vim                                                          | 编辑文本                                                     |                                                              |
| visudo                                                       | export EDITOR=vim  # 默认是vi，export临时生效<br />testuser ALL=(ALL:ALL) ALL  # <用户名> <主机名>=(\<运行用户:运行组>) <命令列表> |                                                              |
| wget url                                                     | 下载xx源                                                     |                                                              |
| which [cmd]                                                  | 显示命令所在路径                                             |                                                              |
| xargs [OPTION]... COMMAND [INITIAL-ARGS]...                  | extended argument ，默认情况下，将换行符和空格作为分隔符，把stdin分解成一个个命令行参数 |                                                              |
| yum [list remove install]                                    | Yellow dog Updater, Modified<br />属于 Redhat 系列 (如，CentOS) 包管理工具；Debin系列 (如，Ubuntu) 使用 apt-get<br />基于RPM包管理工具，能从指定的服务器离线下载 rpm 包并且自动安装，会自动处理依赖问题，还能更新系统。 |                                                              |
| **\|**                                                       | 管道操作符，将左指令的stdout作为右指令的stdin<br />需要注意：**右边命令必须能接受标准输入流**，否则传递过程中的数据会被抛弃、不处理左边命令的错误<br />常用接收stdin的命令: sed, awk, grep查, cut, head查, top资源, less, more, wc统计, join, sort, split |                                                              |
| >>                                                           | 输出重定向<br />cat ~./ssh/id_rsa.pub  >>  ~/.ssh/authorized_keys  把公钥追加到authorized_keys中 |                                                              |
| &                                                            | 表示在后台执行                                               |                                                              |
| &&                                                           | 表示前一条命令执行成功才执行后一条命令                       |                                                              |
| $()                                                          | powershell：${}，例如${pwd}                                  |                                                              |

​           

​      

​      

​      

​      

​      

​            

 

 

​      

​      

​      

​      

 

 

 

​      

 