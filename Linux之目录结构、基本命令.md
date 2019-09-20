## Linux基础命令二



#### Tab键的使用

- **内部命令：**也称shell内嵌命令;

- **外部命令：**sehll会根据环境变量从左至右依次查找，找到一个匹配的则返回；

  - 如果说给定的字符串只能搜索到一个的话，则直接显示

  - 如果给定的字符串搜索到多个的话，则需要按两次Tab键

    ```shell
    外部命令：存放在一个文件中，使用时需要去文件中查找，这些文件被定义在$PATH
    type命令可以查看命令类型，以区别是内部命令还是外部命令
    
    [root@centos7 ~]# type ifconfig
    ifconfig is /usr/sbin/ifconfig
    
    [root@centos7 ~]# type cd
    cd is a shell builtin   
    
    #可以看到，cd为shell内嵌命令，ls命令为ls --color=auto的别名，ifconfig命令为外部命令在文件/usr/sbin/ifconfig中。
    ```

- **目录补全：**把用户给定的字符串当做路径的开始部分，来搜索

  - 如果只搜索到一个，则直接显示，直接一个tab

  - 如果说搜索到多个的话，列出一个列表，让用户自行选择，需要按两次tab键来获取列表

    

#### 内部命令与外部命令查看帮助的区别

- ##### 内部命令:

  ```shell
  1、使用help COMMAND
  [root@centos7 ~]# help cd
  cd: cd [-L|[-P [-e]]] [dir]
  Change the shell working directory.
  
  2、使用man命令查看bash帮助文档
  
  3、enable命令可以禁用内部命令;enable -n 内部命令： 禁用内部命令(重启失效)
  [root@MyHost /]#enable -n cd  #禁用cd
  enable 内部命令：解除禁用
  [root@MyHost /]#enable cd  #解除禁用
  ```

- ##### 外部命令：

  ```
  外部命令查看帮助的方法相对与内部命令还是比较多的
  1、command --help 或 command -h
  
  2、使用man手册查看帮助， man command
  [root@centos7 ~]# man ifconfig
  
  3、使用info命令查看信息页
  [root@centos7 ~]# info ifconfig
  
  
  ```

  

#### echo 回显

输入什么就输出什么，并加入一个换行符

```shell
[root@MyHost ~]#echo safasdfadf\
> asfdad\
> fds
safasdfadfasfdadfds
You have new mail in /var/spool/mail/root
```



#### 获取环境变量

```shell
[root@localhost ~]#echo $PATH  #从左至右查找
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```



#### 命令历史

- 可以通过键盘上的上下键回调使用过的命令

- 通过history来获取之前执行过的命令记录

  ```shell
  [root@MyHost etc]#history
  
  [root@MyHost etc]#history 5   #显示最后5条命令记录
     87  history 
     88  cd /etc
     89  history 
     90  history 10
     91  history 5
  
  [root@MyHost etc]#history -c  #清空历史文件，不会保存在家目录下的.bash_history文件 ，且无法使用向上符号了
  
   # 用户登陆之后，会读取家目录下的.bash_history文件，用户退出后会自动写入history到该文件
   
  [root@MyHost ~]#cat /root/.bash_history 
  ```

- 执行刚才执行过的命令（带有参数）

  ```shell
  1、键盘上的向上箭头号
  2、!!
  3、!-1
  4、ctrl + p
  ```

- 使用!:0来执行上一条命令（去掉参数）

  ```shell
  [root@MyHost etc]#!-1
  cd /etc/
  [root@MyHost etc]#!:0  #只返回前面的命令，并执行
  cd
  
   # 使用 esc + . (不是连着一起按下，是先后可返回上一条命令的参数部分)
  [root@MyHost /]#!s   #直接关机
  ```

- !#  调用history中从上至下的第几条命令(带参数)

  ```shell
     87  history 
  [root@MyHost etc]#!86
  cd /etc
  ```

- !string 直接查找最近的一条包含string的命令

  ```shell
  [root@MyHost ~]#echo adad
  adad
  [root@MyHost ~]#!echo
  echo adad
  adad
  ```

- ctrl+r 搜索之前执行过的命令 / ctrl + g 退出搜索

  ```
  [root@MyHost ~]#echo adad
  adad
  (reverse-i-search)`c': echo adad
  ```

- 调用上一个命令的参数

  - esc .



#### 快捷键

- ctrl+l 清屏，相当于clear

- ctrl+s 锁屏，光标不动，但是输入仍然会记录，解屏后会显示

- ctrl+q 解开锁定

- ctrl+c 终止命令

- ctrl+e 移动到行尾，相当于end

- ctrl+a 移动到命令的行首

- ctrl+xx 光标在命令行首和光标之间来回移动

- ctrl+k 删除光标到结尾位置的字符

- ctrl+u 删除光标到行首的字符

- alt+r 删除整行

  

#### 帮助信息

- ##### 内部命令

  - help command
  - man bash

- ##### 外部命令

  - command --help

  - command -h

  - man command        (使用q退出)

  - 官方文档

    ```shell
    Usage: date [OPTION]... [+FORMAT]
      or:  date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
      [] 代表可选
      ... 表示一个列表，表示多个选项的组合
      [-u|--utc|--universal] 任选其中一个
      -lh 代表-l -h
      date 052805271980 设置时间
      
    ntpdate time.windows.com  #自动与时间服务器同步时间
    ```

    

#### man 相关

使用 man 命令来获得命令的详细讲解，

- ##### 查看man的使用

  ```shell
  man man  # 查看man的使用
  ```

- ##### 章节

  ```shell
  1 用户的命令
  2 系统的调用
  3 c库的调用
  4 设备文件或者特殊文件
  5 配置文件
  6 游戏
  7 杂项
  8 管理类的命令
  9 linux内核API
  
   # 具体查看那一章节：man 数字 命令
  ```

  

#### 目录结构

![1564490927647](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1564490927647.png)

- 目录结构是一个倒置的树
- 目录从"/" 开始
- 目录严格区分大小写
- 隐藏文件以 . 开头



#### 文件命名规则

- 文件名最长255个字节
- 包括路径在内文件名称最长4095个字节
- 颜色表示
  - 蓝色 —>目录
  - 绿色—>可执行文件
  - 红色—>压缩文件
  - 蓝绿色—>链接文件
  - 灰色—>其他文件
- 除了斜杠和NULL，所有的字符都有效
- 文件名称大小写敏感



#### 文件系统结构

- /boot 引导文件存放目录，内核文件、引导加载器都存放在此目录
- /bin 所有用户使用的基本命令，不能关联至独立分区，系统启动即会用到程序
- /sbin 管理类的基本命令；不能关联至独立分区，系统启动即会用到程序
- /lib 启动时程序以来的基本共享库以及内核模块文件
- /lib64 专用于X86_64系统上的辅助共享库文件存放位置
- /etc 配置文件目录
- /home/USERNAME 普通用户家目录
- /root 管理员的家目录
- /media 便携式移动设备挂载点
- /mnt 临时文件系统挂载点
- /dev 设备文件及特殊文件存储位置
- /opt 第三方应用程序的安装位置
- /src 系统上允许的服务用到的数据
- /tmp 临时文件存储位置
- /usr 存放安装程序
- /var 存放经常变化的文件，比如日志
- /proc 用于输出内核与进程相关的虚拟文件系统
- /sys 用于输出当前系统上硬件设备相关虚拟文件系统
- /selinux selinux相关的安全策略等信息的存储位置



#### 程序组成部分

- ##### 二进制

  ```
  1、/bin
  2、/sbin
  3、/usr/bin
  4、/usr/sbin
  5、/usr/local/bin
  6、/usr/local/sbin
  ```

- ##### 库文件

  ```
  1、/lib
  2、/lib64
  3、/usr/lib
  4、/usr/lib64
  5、/user/local/lib
  6、/usr/local/lib64
  
  ```

- ##### 帮助文件

  ```
  1、/usr/share/man
  2、/usr/share/doc
  3、/usr/local/share/man
  4、/usr/local/share/doc
  ```

- ##### 配置文件

  ```
  1、/etc
  2、/etc/directory
  3、/usr/local/etc
  ```



#### linux下的文件类型

- \- 普通文件

- d 目录文件

- b 块设备

- c 字符设备

- l 符号链接文件

- p 管道文件pipe

- s 套接字文件socket

  ```shell
  [root@MyHost ~]#ll
  total 0
  -rw-r--r-- 1 root root 0 Jul 30 21:17 a   #第一个-表示它是普通文件
  ```

  

#### 相对路径和绝对路径

- ##### 绝对路径

  - 从根目录下出发
  - 完整的路径

- ##### 相对路径

  - 相对于某个文件出发，查出找文件
  - ../ 代表父级目录；./ 代表当前文件 （/的父目录永远是自己）

- ##### 获取文件名和文件目录

  ```shell
  [root@MyHost ~]#basename /etc/sysconfig/network-scripts/ifcfg-ens33
  ifcfg-ens33
  [root@MyHost ~]#dirname /etc/sysconfig/network-scripts/ifcfg-ens33
  /etc/sysconfig/network-scripts
  ```



#### 常用命令

- ##### cd 命令 

  ```shell
  change directory  # 可以使用相对路径，也可以使用绝对路径
  
  1、使用绝对或者相对路径
  	cd /etc/sysconfig/network-script
  	cd ifcfg-eth0
  	
  2、切换至父目录
  	[root@centos network-scripts]#cd ..
  	[root@centos sysconfig]#
  
  3、切换之当前用户主目录
  	[root@centos sysconfig]#cd
  	[root@centos ~]#
  
  4、切换至上一次的工作目录，切换到上次的目录（上一次在家目录）
  	[root@MyHost ~]#cd /etc/cloud/
  	[root@MyHost cloud]#cd -
  	/root
  
  ```

- ##### pwd命令

  ```shell
  pwd ：printing working directory
  [root@centos ~]#pwd
  /root
  [root@centos ~]#cd /lib
  [root@centos lib]#pwd -P
  /usr/lib
  [root@centos lib]#pwd -L
  /lib
  ```

- ##### ls命令

  ```shell
  列出当前目录或者指定目录的内容
  用法： ls [options] [files_or_dirs]
   
  ls -a 列出所有的文件，包括隐藏文件
  ls -l  使用较长格式列出信息  alias ll='ls -l --color=auto'
  ls - R 目录递归显示,例出目录下的所有文件
  ls -ld 显示目录（本身）和符号链接信息
  ls -1(数字1) 文件分行显示
  ls -S 按从大到小排序
  ls -lSr 升序排序
  ls -t 按创建时间排序
  ls -r 倒序排序
  ls -d */ 显示当前目录下的目录
  ls -lh 按照人类易读方式显示
  l. 显示当前目录下的目录
  ls -u 配合-t选项，显示并按照atime从心到旧排序
  ls -1 (数字1) 竖着显示
  ls -d */ 显示当前目录下的目录，不能指定目录
  ls -h 以人类易读的方式显示
  
  ls -d 显示目录本身
  [root@MyHost etc]#ls -d /etc/
  /etc/
  
  ```

- ##### touch 创建空文件、修改文件时间戳

  ```shell
  touch 用来修改时间和创建文件
  	如果文件存在的话，则修改时间
  	如果不存在，则创建文件
  
  [root@MyHost ~]#mkdir test
  [root@MyHost ~]#touch a
  [root@MyHost ~]#ls -ld
  dr-xr-x---. 7 root root 4096 Jul 30 21:41 
  
  -a：或--time=atime或--time=access或--time=use  只更改存取时间；因为文件有三个记录
  -c：或--no-create  不建立任何文件；
  -d：<时间日期> 使用指定的日期时间，而非现在的时间；
  -f：此参数将忽略不予处理，仅负责解决BSD版本touch指令的兼容性问题；
  -m：或--time=mtime或--time=modify  只更该变动时间；
  -r：<参考文件或目录>  把指定文件或目录的日期时间，统统设成和参考文件或目录的日期时间相同；
  -t：<日期时间>  使用指定的日期时间，而非现在的时间；
  --help：在线帮助；
  --version：显示版本信息。
  
  格式 touch [OPTION]... FILE...
  -a 仅改变atime 和ctime
  -m 仅改变mtime和ctime
  ```



#### 命令的展开

- ##### 创建文件时的命令展开

  ```shell
  a{1..10}   命令展开
  a{1..10..2} 指定步长
  seq 1 10 
  seq 1 2 10
  
  [root@MyHost ~]#touch a{1..10..2}
  [root@MyHost ~]#ls
  a1  a3  a5  a7  a9  test
  [root@MyHost ~]#rm a{1..10..2}
  rm: remove regular empty file ‘a1’? y
  rm: remove regular empty file ‘a3’? y
  rm: remove regular empty file ‘a5’? t^Hy  #如果输入错误，可以按住ctrl再按backspace删除
  rm: remove regular empty file ‘a7’? y
  rm: remove regular empty file ‘a9’? y
  
   #除了步长、数字外还可以展开字符
  [root@MyHost test]#touch a{a..z}
  [root@MyHost test]#ls
  aa  ab  ac  ad  ae  af  ag  ah  ... aq  ar  as  at  au  av  aw  ax  ay  az
  ```

- ##### seq 命令

  ```shell
  [root@MyHost test]#seq 1 10
  1
  2
  ...
  8
  9
  10
  
  [root@MyHost test]#seq 1 3 10
  1
  4
  7
  10
  
  [root@MyHost test]#touch aseq a 2 z       # 会创建aseq、a、2、z四个文件
  [root@MyHost test]#touch a`seq 1 2 10`    # 使用反引号配合seq，seq不能用于字母
  [root@MyHost test]#ll
  -rw-r--r-- 1 root root 0 Jul 30 22:00 a1
  -rw-r--r-- 1 root root 0 Jul 30 21:56 3
  -rw-r--r-- 1 root root 0 Jul 30 21:56 5
  -rw-r--r-- 1 root root 0 Jul 30 21:56 7
  -rw-r--r-- 1 root root 0 Jul 30 21:56 9
  ```

#### 命令的引用

```shell
`date`   #将date作为命令输出结果
$(date)

[root@MyHost test]#echo `date`
Tue Jul 30 22:03:29 CST 2019
[root@MyHost test]#echo $(date)
Tue Jul 30 22:04:07 CST 2019

[root@MyHost test]#echo  `date +%Y-%m-%d`
2019-07-30
```



#### 文件通配符

- ? 代表任意的一个字符

  ```shell
  [root@MyHost test]#touch ad adcd
  [root@MyHost test]#ls ad??
  adcd
  ```

- *代表所有字符，0~无穷

  ```shell
  [root@MyHost test]#ll a*
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a1
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a3
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a5
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a7
  ```

- ~ 代表家目录

  ```shell
  [root@MyHost test]#ls ~
  a5  test
  ```

- [0-9] 代表数字

  ```shell
  [root@MyHost test]#ll a[0-9]    #筛选a后有一个数字
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a1
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a3
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a5
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a7
  -rw-r--r-- 1 root root 0 Jul 30 22:10 a9  
  ```

- [a-z] 字母，从a-z并且包括A-Y

- [A-Z] 字母，从A-Z 并且包括b-z

- [abcdef] 表示其中的任何一个

  ```shell
  [root@MyHost test]#ls ab[cde]
  abc
  ```

- a\[^abcdef] 取反

  ```shell
  [root@MyHost test]#ls [^abcd]
  3  5  7  9
  ```

- [:lower:] 小写字符

  ```shell
  [root@MyHost test]#ls a[[:lower:]]
  ad
  ```

- [:upper:] 大写字符

- [:digit:] 数字

  ```powershell
  [root@MyHost test]#ls a[[:digit:]]
  a1  a3  a5  a7  a9
  ```

- a-zA-Z 所有字母

- [:alpha:] 所有字母

- a-zA-Z0-9 任意字母或者数字

- [:alnum:] 代表所有的字母和数字



#### stat 查看文件状态

```shell
[root@MyHost test]#stat 3 
 
 File: ‘aa’
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: fd00h/64768d	Inode: 19864315    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2019-07-30 11:57:02.279384871 +0800
Modify: 2019-07-30 11:57:02.279384871 +0800
Change: 2019-07-30 11:57:02.279384871 +0800
 Birth: -
访问时间：access      读取文件内容  atime
修改时间：Modify      改变文件的内容 mtime
改变时间：change      改变文件的内容 ctime
```



#### 复制文件和文件夹

```shell
Usage: cp [OPTION]... [-T] SOURCE（源文件） DEST（目标文件）
  or:  cp [OPTION]... SOURCE... DIRECTORY
  or:  cp [OPTION]... -t DIRECTORY SOURCE...  
```

- ##### 常见的复制文件操作

  ```shell
   # 复制给文件进行覆盖
  [root@MyHost test]#touch a.txt   
  [root@MyHost test]#echo 'da' >> a.txt 
  [root@MyHost test]#touch b.txt
  [root@MyHost test]#cp a.txt b.txt    # 源文件 覆盖 目标文件
  cp: overwrite ‘b.txt’? y
  [root@MyHost test]#cat b.txt 
  da
  
   # 复制到文件中去
  [root@MyHost test]#mkdir test
  You have new mail in /var/spool/mail/root
  [root@MyHost test]#cp a.txt test
  [root@MyHost test]#ll test
  total 4
  -rw-r--r-- 1 root root 3 Jul 30 22:30 a.txt
  
   # 源文件没有就会新建文件
  [root@MyHost test]#cp a.txt dsaf
  [root@MyHost test]#ll
  total 16
  -rw-r--r-- 1 root root    3 Jul 30 22:32 dsaf
  drwxr-xr-x 2 root root 4096 Jul 30 22:30 test
  [root@MyHost test]#cat dsaf
  da
  
  ```

  - 如果source是一个文件的话
    - 如果目标不存在，新建一个目标文件，并将数据写入到目标文件里面
    - 如果目标文件存在
      - 如果目标文件是一个目录，直接在目标目标下面新建一个跟源文件同名的文件，并将数据目标文件写入到文件
      - 如果说目标文件是一个文件，直接就覆盖，为了安全起见，建议cp配合-i使用
  - 如果源文件是多个文件的话
    - 目标文件如果是文件的话，则直接报错
    - 如果目标文件是一个目录的话，则直接复制进目录
  - 如果源文件是目录的话
    - 如果目标不存在，则创建指定的目录，必须-r选项
    - 如果说目录存在
      - 如果目录是一个文件的话，则会报错
      - 如果目标是一个目录的话，则在目录下面创建一个新的同名目录，并把文件复制过去

- ##### 复制命令进阶

  ```shell
  1、默认情况 cp 命令使用的是alias 'ls -i' 每次复制时都会有提醒，不想这样
  [root@MyHost test]#"cp" a.txt b.txt 
  
  2、将多个文件复制到一个目录下
  [root@MyHost test]#cp a.txt b.txt test/
  cp: overwrite ‘test/a.txt’? y     
  [root@MyHost test]#ll test/
  total 8
  -rw-r--r-- 1 root root 3 Jul 30 22:39 a.txt
  -rw-r--r-- 1 root root 3 Jul 30 22:39 b.txt
  
  3、复制目录，需要递归，如果test1不存在就创建，有就直接写进去
  [root@MyHost test]#cp -r test test1
  You have new mail in /var/spool/mail/root
  [root@MyHost test]#ll
  drwxr-xr-x 2 root root 4096 Jul 30 22:39 test
  drwxr-xr-x 2 root root 4096 Jul 30 22:40 test1
  ```

- ##### 常用参数

  ```shell
  -i 覆盖之前提示
  -n 不覆盖
  [root@MyHost test]#cp -ni a.txt b.txt    # 以最后一个为准-ni
  cp: overwrite ‘b.txt’? y 
  
  -r 递归复制，复制目录及目录下的所有文件
  -f 强制
  [root@MyHost test]#cp -rf test test1
  [root@MyHost test]#ll test1
  total 12
  -rw-r--r-- 1 root root    3 Jul 30 22:40 a.txt
  -rw-r--r-- 1 root root    3 Jul 30 22:40 b.txt
  drwxr-xr-x 2 root root 4096 Jul 30 22:46 test
  [root@MyHost test]#ll test
  total 8
  -rw-r--r-- 1 root root 3 Jul 30 22:39 a.txt
  
  -v 显示复制过程
  -b 在覆盖之前，对源文件做备份，如果涉及到要覆盖一个文件，就会生成备份文件
  [root@MyHost test]#cp -b 3 5
  cp: overwrite ‘5’? y
  [root@MyHost test]#ll
  total 20
  -rw-r--r-- 1 root root    0 Jul 30 22:00 3
  -rw-r--r-- 1 root root    0 Jul 30 22:49 5
  -rw-r--r-- 1 root root    0 Jul 30 22:00 5~
  
  cp  --backup=numbered 1.cfg 2.cfg 覆盖文件，备份文件添加上数字
  -p 保留原来的属性
  -n 表示有目标文件就不用覆盖
    [root@MyHost ~]# echo adfadfadf >> bb10
    [root@MyHost ~]# cat bb10
    adfadfadf
    [root@MyHost ~]# echo ad >> bb11
    [root@MyHost ~]# cp -n bb10 bb11
    [root@MyHost ~]# cat bb11
    ad
  ```



#### 移动或者重命名

- ##### 常用操作

  ```python
    Usage: mv [OPTION]... [-T] SOURCE DEST
      or:  mv [OPTION]... SOURCE... DIRECTORY
      or:  mv [OPTION]... -t DIRECTORY SOURCE...
    -i  交互式
    -f  强制
    -b  覆盖前做备份
    -v 显示进度
    
    [root@MyHost test]# mv ../a5 .   # 将父目录下的a5移动到当前目录
    mv: overwrite ‘./a5’? y
    
    [root@MyHost test]#mv 11 22     # 重命名
    [root@MyHost test]#ll
    -rw-r--r-- 1 root root    0 Jul 30 22:52 22
    
    [root@MyHost test]#mv -f 3 22   # 强制覆盖
    
    [root@MyHost test]#mv -b 5 22   # 覆盖前备份
    mv: overwrite ‘22’? y
    [root@MyHost test]#ll
    -rw-r--r-- 1 root root    0 Jul 30 22:00 22~
  ```

  

#### 删除

- ##### 常用操作

  ```shell
  -i 交互式
  -f 强制删除
  -r 递归
  rm -rf /*
  
  [root@MyHost test]#rm -f 22
  [root@MyHost test]#"rm" test
  rm: cannot remove ‘test’: Is a directory   # rm只删除文件
  [root@MyHost test]#"rm" ad       # 默认使用别名，有i参数
  
  [root@MyHost test]#rm -rf test1/  # 递归加强制删除
  
  [root@MyHost test]# rm -f *   # 删除当前文件下所有文件，注意不要写成了rm -f /*
  [root@MyHost test]# ll
  total 0 
  ```



#### 目录操作

- tree显示目录树（没有就需要安装）

  ```shell
  tree --help  
  [root@MyHost ~]# tree -L 1   # 控制显示层数
  
  [root@MyHost ~]# tree -d -L 1  # 只显示目录且只有一层
  ```

- 创建目录

  ```shell
  1、mkdir s21  # 在当前目录下创建s21文件
  
  2、同时创建多个文件目录
  [root@MyHost ~]# mkdir s21-{2..10}
  [root@MyHost ~]# ll
  -rw-r--r-- 1 root root   10 Jul 31 18:11 bb10
  -rw-r--r-- 1 root root    3 Jul 31 18:11 bb11
  -rw-r--r-- 1 root root    0 Jul 30 23:55 bb12
  ...
  
  3、递归创建目录
  [root@MyHost ~]# mkdir -p a/b/c/d
  [root@MyHost ~]# yum install -y tree
  [root@MyHost ~]# tree
  .
  ├── a
  │   └── b
  │       └── c
  │           └── d
  
  4、tree 显示目录树 （tree name） 列出某目录下的结构
  	-d 只显示目录
  	-L level 执行显示的层级数目
  [root@MyHost ~]# tree -d
  .
  ├── a
  │   └── b
  │       └── c
  │           └── d
  ├── s21-10
  ├── s21-2
  
  5、分项创建目录
  [root@MyHost ~]# mkdir -p {s11,s12}/{s11,s12}
  [root@MyHost ~]# tree
  .
  ├── s11
  │   ├── s11
  │   └── s12
  ├── s12
  │   ├── s11
  │   └── s12
  
  [root@MyHost ~]# mkdir -pv {s11,s12}/{s11,s12}  #显示创建过程
  ```

- ##### 删除目录

  ```shell
  mkdir 只删 除空目录
  1、-p 递归删除父空目录
  [root@MyHost ~]# rmdir -p s11      #-p只能删除递归删除空目录
  rmdir: failed to remove ‘s11’: Directory not empty
  
  [root@MyHost ~]# rmdir -p a/b/c/d   # 如果d为空则删，它的父目录为空则删...递归
  
  2、-v 显示详细信息
  
  3、rm -r 递归删除目录树（除了删除文件还可以删除目录）
  [root@MyHost ~]# rm -rf bb*   # 还是rm好使，可以删除非空目录
  ```

  



##### 练习

```
    # 创建b1到b20的文件
    touch b{1..20}
    # 创建ba到bf的文件
    touch b{a..f}
    # 创建bb1到bb20偶数的文件
    touch bb{20..1..-2}
    # 列出bb文件
    ll bb
    # 列出b10到b19文件
    ll b1{0..9}
    ll b1[0-9]
    # 列出bb10到bb19的文件
    ll bb1[0-9]
    # 修改bb19的创建日期
    touch -a bb19
    # 将bb10到bb19文件复制到/home目录下面
    cp bb1[0-9] /home/
    # 将bb10重命名为bc10
    mv bb10 bc10
    # 将bb10文件删除
    rm -rf bb10
```

