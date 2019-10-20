## Linux基础命令一

##### 直接使用的时阿里云的服务器，使用Xmanager中的Xshell进行远程登陆

首先对于要使用Linux来说，最基本的要了解用户的管理，下面我们先简单的了解下用户管理的基本知识

##### 用户的相关配置文件

- 用户信息文件

  ```shell
  命令: cat /etc/passwd  # 查看用户信息文件
  
  root:x:0:0:root:/root:/bin/bash
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  wang:x:1001:1001::/home/wang:/bin/bash
  
   #解析：
   #每一行代表一个用户信息，一共包括7个字段
   #字段1：账户名称，即登陆时的用户名
   #字段2：密码，x表示加密后保存到了/etc/shadow/
   #字段3：UID，用户id，一个用户对应一个id，通常UID=0表示root
   #字段4：GID，组ID，与/etc/group/有关
   #字段5：用户信息说明栏，默认为空，解释用户是来干嘛的
   #字段6：家目录位置，home目录，用户登陆后自动跳转的位置
   #字段7：shell，用户使用的shell，一般用户使用/bin/bash这个shell，默认用户登陆使用的shell，如果设置为了替代让账号无法登陆的命令，那就是/sbin/nologin。
  ```

  **小提示：**

  ```shell
  1、我们可以通过以下命令，让一般用户无法登陆
  	touch /etc/nologin
  
  2、那么你会问，为什么要使用nologin这种shell哪？
  	原因是有些shell功能很强大了，必须要登陆root/或分配了相应用户的权限的功能
  	
  3、UID：关于上面的UID这里细说一下，其实决定用户是什么权限，是由UID号决定的，分为三类。
  	超级用户：（root   UID=0）
  	普通用户： （UID 500~60000）
  	伪用户：  （UID  1~499）
  	#也就是说我们可以修改用户的id来进行分配：vim /etc/passwd/
  
  4、伪用户:
  	1.伪用户与系统和程序服务相关;
  		 bin、daemon、shutdown、halt等，任何Linux系统默认都有这些伪用户。
      	 mail、news、games、apache、ftp、mysql及sshd等，与linux系统的进程
  	2. 伪用户通常不需要或无法登录系统
  	3. 可以没有宿主目录
  ```

- 密码文件详解

  ```shell
  命令：cat /etc/shadow/
  
  root:$6$J8FA2/77CJw$4khx.eLbexA0FgfVJXbbi0JfdBOwL.n9lcHt0CxS8WMrZVGTeYMuocP0YkmDZFQvagVV0E9nrqvnNX/22aoR.0:18106:0:99999:7:::
  bin:*:17110:0:99999:7:::
  daemon:*:17110:0:99999:7:::
  adm:*:17110:0:99999:7:::
  
   # 一行数据表示一个用户信息，分为8给字段：
   # 字段1：用户名，如root
   # 字段2：密码，使用$隔开，6表示一种类型标记为6的密码散列，第二部分是盐，第三部分是hash值
   # 字段3：表示最后一次修改事件，从1970年1月1日（UNIX元年），开始记天数
   # 字段4：最小时间间隔，表示要经过多久才能修改密码，0表示随时可以修改
   # 字段5：最大时间间隔，也就是说过多久必须修改密码，否则此账号会失效，99999表示不需要修改
   # 字段6：密码变更期期限快到前的警告期，7默认设置七天时提醒更改密码
   # 字段7：之后的都是保留的，以备后面添加功能
  ```

- 用户操作

  ```shell
  1、创建用户
  	useradd lisi
  	passwd lisi
  	# 上面这种是不被推荐的一种创建用户的方式，任何的用户都应该属于某个组。创建这样的“散人”实际中没有太大意义。
  2、查看用户配置信息
  	grep lisi /etc/passwd/
  	
  3、用户切换
  	[root@localhost ~]\# su -- lisi        #root切换到lisi用户
  	[lisi@localhost root]$ su -- root      #lisi用户切换到root
  	Password:                              #普通用户切换root用户是要密码的
  	# 普通用户切换到其它普通用户也是要密码的哟！	
      
  4、修改本用户密码
  	[lisi@localhost root]$ passwd
  ```

  **小问题**：细心的人可能会发现，怎么普通用户可以修改自己的密码？这就涉及到setUID知识了



- 配置主机名

  ```shell
  一般情况下主机名配置文件中etc目录的hosts文件中
  1、阿里云服务器中的主机名配置文件在/etc/hostname/中：vim /etc/hostname/ 
     修改后，reboot重启  #永久有效
     
  2、方式二：[root@localhost ~]\# hostaname wang   #但是重启后失效
  
  3、方式三：[root@localhost ~]\#hostnamectl set-hostname li  # 永久有效
  ```



#### 终端

##### 基本概念

```
1. tty(终端设备的统称):
	tty一词源于Teletypes，原来指的是电传打字机，是通过串行线用打印机键盘通过阅读和发送信息的东西，后来这东西被键盘与显示器取代，所以现在叫终端比较合适。终端是一种字符型设备，它有多种类型，通常使用tty来简称各种类型的终端设备。

2、pty（虚拟终端):
	我们在使用远程telnet到主机或使用xterm时也会产生一个终端交互，这就是虚拟终端pty(pseudo-tty) 
例如，我们在X Window下打开的终端，以及我们在Windows使用telnet 或ssh等方式登录Linux主机，此时均在使用pty设备(准确的说应该是pty从设备)。

3、pts/ptmx(pts/ptmx结合使用，进而实现pty):
	伪终端(Pseudo Terminal)是终端的发展，为满足现在需求（比如网络登陆、xwindow窗口的管理）。它是成对出现的逻辑终端设备(即master和slave设备, 对master的操作会反映到slave上。也就是说pts(pseudo-terminal slave)是pty的实现方法，和ptmx(pseudo-terminal master)配合使用实现pty。
```

- 图形终端 （ctrl+alt+F7）通过图形终端可以创建无数个伪终端

- 虚拟终端（ctrl+alt+F1-6  /dev/tty#）

- 物理终端

- 设备终端

- 串行终端

- 伪终端  /dev/pts/#（无限个）

- tty 查看命令

  

#### 远程连接工具

- xshell
- putty
- SecureCRT



##### 查看ip地址

```shell
ifconfig  #查看ip地址
ip addr 
ip a
```

##### 使用ssh登陆服务器

```
 ssh root@47.95.217.144
 ssh wang@47.95.217.144  #一般用户登陆
```



##### 交互式接口

- GUI图形接口

- CLI（也就是linux中带有得各种解释语言）

  - sh

  - csh

  - tcsh

  - ksh

  - bash（linux、mac上的shell）

  - zshl

    

##### shell命令

```shell
用来在linux系统上的一个接口，用来将用户的输入发送给操作系统去执行，并把得到的结果输出出来
1、查看系统支持的shell命令：
 	cat  /etc/shells 
 	
2、切换shell命令：
	chsh -s /bin/csh  #切换shell，但是要指定到绝对路径
	
3、查看当前运行得shell命令：
	echo $SHELL  #$类似于取值符
```



##### 命令提示符

```shell
1、显示提示符格式：
	[root@centos ~]# echo $PS1
	[\u@\h \W]\$
	[用户@主机名 当前目录] 命令提示符

2、修改提示符格式：
	PS1="\[\e[1;5;41;33m\][\u@\h \W]\\$\[\e[0m\]"
    \e 颜色配置
    \h 主机名简称
    \w 当前工作目录 \t 24小时时间格式 \! 命令历史数
    \u 当前用户
    \H 主机名
    \W 当前工作目录基名 \T 12小时时间格式
    \# 开机后命令历史数
    1表示字体加粗， 0表示默认字体。4表示给字体加上下划线。5表示字体闪烁。7表示用亮色突出显示，来让你的文字更加醒目
    31表示字符颜色。
    可选颜色：红色、绿色、黄色、蓝色、洋红、青色和白色。他们对应的颜色代码是：30（黑色）、31（红色）、32（绿色）、 33（黄色）、34（蓝色）、35（洋红）、36（青色）、37（白色）
    40表示字符背景色。可选颜色 40、41、42、43、44、45、46、47

3、永久生效配置命令行：
	echo 'PS1="\[\e[1;30;35m\][\u@\h \W]\\$\[\e[0m\]"' >> /etc/profile.d/ps.sh 
```



#### 1、执行命令

写完命令后直接回车姐可以执行

- 内部命名

  ```
  安装完系统以后自带的命令，就是内部命令，通过help来获取内部命令的列表
  ```

- 外部命令

  ```
  第三方或自定义的在系统启动后加载的命令文件
  ```

- 相关命令：

  ```shell
  type 查看命令的类型
  which 查看命令的路径
  ```

- 如何区分内部还是外部命令

  ```shell
  [root@centos ~]#type cd
  cd 是 shell 内嵌命令
  
  [root@centos ~]#type cp
  cp 是 `cp -i' 的别名
  
  [root@centos ~]#which cp
  alias cp='cp -i'
      /usr/bin/cp
  ```

  

#### 2、alias别名

- 显示当前shell进程所有可用的命令别名 alias

  ```shell
  [root@centos ~]#alias
  alias cp='cp -i'
  alias egrep='egrep --color=auto'
  alias fgrep='fgrep --color=auto'
  alias grep='grep --color=auto'
  alias l.='ls -d .* --color=auto'
  alias ll='ls -l --color=auto'
  alias ls='ls --color=auto'
  alias mv='mv -i'
  alias rm='rm -i'
  alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
  ```

- 定义别名name，其实就是相当于执行命令value（相当于创建快捷键）

  ```shell
  [root@centos ~]#alias cdetc='cd /etc/'
  [root@centos ~]#cdetc
  [root@centos etc]#
  
  
   #对当前用户，将别名命令写入到当前家目录中得.bashrc文件中，所以要注意位置
  [root@localhost ~]#echo "alias cdetc='cd /etc'" >> .bashrc
   #对所有的用户都生效
  echo "alias cdetc='cd /etc'" >> /etc/bashrc
  
  ls  #相当于list
  ls -a  #显示所有文件，包括隐藏文件
  ls --help  #万能得 --help
  ```

- 取消别名

  ```shell
  unalias cdetc  #取消别名
  ```



#### 执行本身得命令

```shell
 # 我们再执行alias的时候会列出所有的别名命令：
 [wang@MyHost profile.d]$alias
 ...
  alias ls='ls --color=auto' # 可以看出别名ls在ls基础上添加了颜色，哪我想使用原本的ls效果怎么办
 
 #方法一：
 [wang@MyHost etc]$"ls"  #使用双引号
 [wang@MyHost etc]$\ls   #使用斜杠
 [wang@MyHost etc]$'ls'  #使用单号
```

#### 单双引号的区别

```shell
"" 可以直接打印变量的值，结合$取变量对应得值
'' 引号里面写什么就打印什么

[wang@MyHost etc]$name="wang"
[wang@MyHost etc]$echo "name"
name
[wang@MyHost etc]$echo "$name"
wang

[wang@MyHost etc]$echo 'name'
name
[wang@MyHost etc]$echo '$name'
$name
```

#### 时间

- ##### date相关

  ```shell
  [root@localhost ~]#date
  Mon Jul 29 12:18:14 CST 2019
  [root@localhost ~]#date +%F
  2019-07-29
  [root@localhost ~]#date +%H（24小时制）
  12
  [root@localhost ~]#date +%I（12小时制）
  12
  [root@localhost ~]#date +%y
  19
  [root@localhost ~]#date +%m
  07
  [root@localhost ~]#date +%d
  29
  [root@localhost ~]#date +%M
  22
  [root@localhost ~]#date +%S
  25
  [root@localhost ~]#date +%a
  Mon
  [root@localhost ~]#date +%A
  Monday
  [root@localhost ~]#date +%T
  12:23:31
  [root@localhost ~]#date +%y-%m-%d
  19-07-29
  [root@localhost ~]#date +%Y-%m-%d
  2019-07-29
  unix元年
  [root@localhost ~]#date +%s 时间戳
  1564374331
  [root@localhost ~]#date +%W 一年中的多少周
  30
  ```

  **练习**

  ```shell
  打印当前时间： date +"%Y-%m-%d %X"  #命令和参数之间，如果后面太长，需要使用引号
  ```

   

- ##### 时区

  ```shell
  [root@localhost ~]#timedatectl
  [root@localhost ~]#timedatectl set-timezone  Asia/Shanghai  #可以使用三次tab键补全
  ```

- ##### 日历

  ```shell
  cal #展示当月的日历
  cal -y #展示当年的日历
  cal -y # 显示#年的日历
  ```



#### 关机重启

```shell
shutdown 默认是一分钟之后关机
shutdown -c 取消
shutdown -r 重启
TIME
	now  立即  # shutdown -r now 立即关机
	hh：mm     # shutdown -r 12:50  十二点50分关机
	+#  表示多长时间以后重启（多少分钟）  shutdown -r +5 
	

reboot 重启
       -p 切断电源
       
init 6 重启
init 0 关机
poweroff 关机
```



#### 命令的格式

```
command [OPTIONS…][ARGS….]

选项：用于启用或者关闭命令的某个或者某些功能
	   短选项：例如 -l -h
	   长选项：例如 - - all - - help
参数：命令的作用队形，比如文件名，用户名等
注意：
	多个选项以及参数和命令之间使用空格分割
	取消和结束命令执行 ctrl+c
	多个命令可以用；隔开
	一个命令过长可以用\分成多行
```

