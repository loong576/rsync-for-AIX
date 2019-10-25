## 一、RSYNC简介

   rsync（remote synchronize）是一个远程数据同步工具，可通过LAN/WAN快速同步多台主机间的文件。rsync使用所谓的“rsync算法”来使本地和远程两个主机之间的文件达到同步，这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快。   

   rsync的基本特点如下：1.可以镜像保存整个目录树和文件系统；2.可以很容易做到保持原来文件的权限、时间、软硬链接等；3.无须特殊权限即可安装；4.优化的流程，文件传输效率高；5.可以使用rsh、ssh等方式来传输文件，当然也可以通过直接的socket连接；6.支持匿名传输。

## 二、Openssh安装

### 1.下载软件

**Openssl下载:**
https://www14.software.ibm.com/webapp/iwm/web/reg/download.do?source=aixbp&S_PKG=openssl&lang=en_US

**Openssh下载:**
http://sourceforge.net/projects/openssh-aix 

**下载得到软件为:** openssl-0.9.8.1302.tar.Z 和openssh_5.4p1.tar.z

### 2.安装openssh

一般是先装openssl再装openssh。通过smit install 安装 openssl和openssh, 如下，输入安装文件的路径，选择接受新的许可协议。

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadpiay4CPHQmGiaLPiafBhtriblyH1Xw84tWpZAYia9lWH7S6j11ibZsic2ribQkg/640?wx_fmt=png)

### 3.检查安装包

装完ssl和ssh后安装包如下：

**ssh:** openssh.base.client、openssh.base.server、openssh.license、openssh.man.en_US、openssh.msg.en_US;

**ssl:** openssl.license、openssl.man.en_US、openssl.base

### 4.重启服务

```bash
tppc01:/ #stopsrc -s 
sshd0513-044 The sshd Subsystem was requested to stop.
tppc01:/ #startsrc -s sshd
0513-059 The sshd Subsystem has been started. Subsystem PID 
is 1581116.
tppc01:/ #ssh –V
OpenSSH_5.4p1, OpenSSL 0.9.8m 25 Feb 2010
```

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadprBGZ6Nzpw9XeVKeD5ofV78pdAPXVFkMwjiaMEibdEyxmT1fJ34g4Kfpw/640?wx_fmt=png)

## 三、RSYNC安装

### 1.安装rsync包

下载安装包port和rsync:

popt-1.7-2.aix5.1.ppc.rpm

rsync-2.6.2-1.aix5.1.ppc.rpm

链接为：

```bash
ftp://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/rsync/rsync-2.6.2-1.aix5.1.ppc.rpm
ftp://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/rpm/popt-1.7-2.aix5.1.ppc.rpm
```

用smitty进行安装，将软件包放置于/tmp/rsync目录下，安装如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadp23N3ia548rfnaltgYFAjgHEeq97WicE4DbaianAAqTOiccDdGgYuu9pfmQ/640?wx_fmt=png)

### 2.服务器端配置

服务器端为源端（172.27.34.237），源端配置文件主要为rsyncd.conf(主配置文件)、rsyncd.pwd(密码文件)、rsyncd.motd(rsync服务器信息)，在/etc下新建rsync目录，进入/etc/rsync新建配置文件**rsyncd.conf、rsyncd.pwd、rsyncd.motd.**

**rsyncd.conf内容如下：**

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadpWRGWZkseDr5GjTxlSBvd7InvxZvZDjiaaCrBgfibROA9EnCCnlPpNopQ/640?wx_fmt=png)

rsyncd.conf 是rsync服务器主要配置文件，该文件默认不存在需手动创建。

```bash
uid=root
gid=system
#max connections=4
use chroot=true
log file=/var/log/rsyncd.log
pid file=/var/run/rsyncd.pid
motd file = /etc/rsync/rsyncd.motd
#lock file=/var/run/rsyncd.lock
#auth users=root
secrets file=/etc/rsync/rsyncd.pwd
transfer logging = true
#port = 873
#limit access to private LANs
hosts allow=172.27.34.238
#hosts deny=*
[rsync]
path=/home/rsync
comment = home rsync 
#ignore errors
read only = yes
list = yes 
auth users = root 
secrets file=/etc/rsync/rsyncd.pwd
```

**rsyncd.pwd内容如下：**

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadpANFngxTLhV5wdNRg8ia3PicqMLqCicEbCltq88MICM2JgCMs4CpODaRNQ/640?wx_fmt=png)

rsyncd.pwd为密码文件，格式为：user:password；此用户必须系统中存在，密码为rsync同步密码，可以与系统密码不同，服务器端与客户端保持一致即可；为保证密码的安全性，**密码文件权限应设置为600，属主为root**。

**rsyncd.motd内容为：**

```bash
++++++++++++++++++++++++++++++++++++++++++++++
 
　　Welcome to use the Location A To Location B rsync services!
 
                  2018------2019
 
++++++++++++++++++++++++++++++++++++++++++++++
```

rsyncd.motd是定义rysnc 服务器信息的，也就是用户登录信息。比如让用户知道这个服务器是谁提供的等；类似ftp服务器登录时，我们所看到的提示信息。在全局定义变量时，并不是必须的。

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadptdiboZRSgcFahSAzjF2oxuHjNGYAYYtIjzYNyMSUSRYa4dstDBSueow/640?wx_fmt=png)

### 3.客户端配置

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadpx2G0KT0uTGVMvd3Lem15L71qVQBe00icq0CPS5ic11zm7ibkBoVeT3Asg/640?wx_fmt=png)

**注意：**客户端密码文件格式与服务器端不同，密码文件权限属性为属主可读。

### 4.启动rsync服务

**服务器端:**
启动rsync进程

```bash
/usr/bin/rsync --daemon --config=/etc/rsync/rsyncd.conf
```

此服务项不会开机启动，服务端机器重启后需启动该服务

检查服务是否启动，查看进程：

```bash
ps –ef|grep rsync
```

检查端口（rsync默认端口为873，端口监听证明服务拉起）：

```bash
netstat –an|grep 873
```

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadpKhia7g4pUlwWfqeTRWaZ89emiarWWyxa6Im6EFEKUpcampoEmLUF8wMw/640?wx_fmt=png)

以上为正常程序正常启动

**客户端：**

```bash
rsync -vzrtopg --progress --delete --exclude "diff_bak/" --password-file=/etc/rsync/rsyncd.pwd root@172.27.34.237::rsync /home/rsync
```

![img](https://mmbiz.qpic.cn/mmbiz_png/ibicF8t44mKPlJzEH2ttMQ2B9gCaCiaGiadpuB5z1cnEnhMNZBL3bKvT9F2ib3EubjVFzttjp5EOwVHXMNTarU0zibQg/640?wx_fmt=png)

为了保证定时同步，客户端同步命令可以写成定时任务形式定时同步。

## 四、配置信息详解

### 1.服务器端定义

**全局定义**

| 参数                            | 说明                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| uid=root                        | 服务器端传输文件时，要发哪个用户和用户组来执行，默认是nobody |
| gid=system                      | 服务器端传输文件时，要发哪个用户和用户组来执行，默认是nobody |
| max connections=4               | 客户端最多连接数                                             |
| use chroot=true                 | 在传输文件之前，服务器守护程序在将chroot 到文件系统中的目录中，这样做的好处是可能保护系统被安装漏洞侵袭的可能。缺点是需要超级用户权限。另外对符号链接文件，将会排除在外。也就是说，你在rsync服务器上，如果有符号链接，你在备份服务器上运行客户端的同步数据时，只会把符号链接名同步下来，并不会同步符号链接的内容 |
| log file=/var/log/rsyncd.log    | rsync服务器的日志                                            |
| pid file=/var/run/rsyncd.pid    | 告诉进程写到/var/run/rsyncd.pid文件中                        |
| motdfile=/etc/rsync/rsyncd.motd | 定义motd file路径，rsyncd.motd内容是定义服务器信息的，用户登录时会看到这个信息 |
| transfer logging = true         | 传输文件日志                                                 |
| port = 873                      | 指定运行端口，默认是873，可以自己指定                        |
| hosts allow=172.27.34.238       | 可以指定单个IP，也可以指定整个网段，能提高安全性。格式是ip 与ip 之间、ip和网段之间、网段和网段之间要用空格隔开 |
| read only = yes                 | 只读选择，不让客户端上传文件到服务器上                       |

**模块定义**

| 参数                              | 说明                                                         |
| :-------------------------------- | :----------------------------------------------------------- |
| [rsync]                           | 模块名。主要是定义服务器哪个目录要被同步。每个模块都要以[name]形式。这个名字就是在rsync 客户端看到的名字。服务器真正同步的数据是通过 path来指定的。我们可以根据自己的需要，来指定多个模块。每个模块要指定认证用户，密码文件、但排除并不是必须的 |
| path=/home/rsync                  | 指定文件目录所在路径                                         |
| comment = home rsync              | 注释。注释内容可自己定义，起提示作用                         |
| ignore errors                     | 忽略IO错误                                                   |
| exclude = beinan/ samba/          | 把/home目录下的beinan和samba 排除在外，beinan/和samba/目录之间有空格分开 |
| list = yes                        | list 意思是把rsync 服务器上提供同步数据的目录在服务器上模块是否 显示列出来。默认是yes 。如果你不想列出来，就no ；如果是no是比较安全的，至少别人不知道你的服务器上提供了哪些目录。你自己知道就行了 |
| auth users = root                 | 认证用户是root，用户必须在服务器上存在，如果想用多个用户，需以,隔开，如auth users = root,user1 |
| secretsfile=/etc/rsync/rsyncd.pwd | 密码文件保存路径                                             |

### 2.客户端定义

**rsync命令格式：**



> - 1.rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST
>
> - 2.rsync [OPTION]... [USER@]HOST:SRC DEST
>
> - 3.rsync [OPTION]... SRC [SRC]... DEST
>
> - 4.rsync [OPTION]... [USER@]HOST::SRC [DEST]
>
> - 5.rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST
>
> - 6.rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
>
>   

**rsync有六种不同的工作模式：**

> - 1.拷贝本地文件；当SRC和DES路径信息都不包含有单个冒号":"分隔符时就启动这种工作模式。
> - 2.使用一个远程shell程序（如rsh、ssh）来实现将本地机器的内容拷贝到远程机器。当DST路径地址包含单个冒号":"分隔符时启动该模式。
> - 3.使用一个远程shell程序（如rsh、ssh）来实现将远程机器的内容拷贝到本地机器。当SRC地址路径包含单个冒号":"分隔符时启动该模式。
> - 4.从远程rsync服务器中拷贝文件到本地机。当SRC路径信息包含"::"分隔符时启动该模式。
> - 5.从本地机器拷贝文件到远程rsync服务器中。当DST路径信息包含"::"分隔符时启动该模式。
> - 6.列远程机的文件列表。这类似于rsync传输，不过只要在命令中省略掉本地机信息即可。

**rsync中的参数：**

| 参数                                | 说明                                                         |
| :---------------------------------- | :----------------------------------------------------------- |
| -a                                  | 以archive模式操作、复制目录、符号连接 相当于-rlptgoD         |
| -r                                  | 递归                                                         |
| -l                                  | 是链接文件，意思是拷贝链接文件                               |
| -p                                  | 表示保持文件原有权限                                         |
| -t                                  | 保持文件原有时间                                             |
| -g                                  | 保持文件原有用户组                                           |
| -o                                  | 保持文件原有属主                                             |
| -D                                  | 相当于块设备文件                                             |
| -P                                  | 传输进度                                                     |
| -v                                  | 传输时的进度等信息                                           |
| -e                                  | ssh的参数建立起加密的连接                                    |
| -u                                  | 只进行更新，防止本地新文件被重写，注意两者机器的时钟的同步   |
| --progress                          | 指显示出详细的进度情况                                       |
| --delete                            | 指如果服务器端删除了这一文件，那么客户端也相应把文件删除，保持真正的一致 |
| --password-file=/password/path/file | 指定密码文件，这样就可以在脚本中使用而无需交互式地输入验证密码了，这里需要注意的是这份密码文件权限属性要设得只有属主可读 |



