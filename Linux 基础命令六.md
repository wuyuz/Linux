##  Linux 基础命令六



#### 用户和组管理命令

##### 开机字符图片

- **介绍motd：**message of the day的缩写，意思就是当天的提示信息，通常在用户成功登陆到Linux后出现，该信息可以从/etc/motd/文本文件中找到。

- 定制开机字符图片，准备一份图片字符写入motd文件中

  ```shell
  [root@MyHost etc]# vim /etc/motd 
  赋值字符粘贴， 重新登陆
  ```

  ![1564895896416](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564895896416.png)



##### 用户分类

- 超级管理员： root  uid 0

- 普通用户

  - 系统用户： 一般情况下用来启动服务或者运行进程，一般情况下系统用户是不可以登陆，使用的nologin的shell

    uid 1-999（centos7）1-499（centos6）

  - 可登陆用户：可以登陆系统的用户，从已知的最大id开始

    uid 1000-65535（centos7） 500-65535（centos6）

##### 添加用户  useradd

- **useradd** ：只适用于root用户

  ```shell
  [root@MyHost home]# useradd demo    # 创建一个用户
  
  [root@MyHost home]# useradd -h   # 查看帮助
  
    #默认情况下在/home/ 目录下创建家目录，可以使用-d选项指定家目录，但是注意不能放在tmp目录下，因为会随时消失，默认用户的家目录不需要手动创建
   
  [root@MyHost home]# useradd -d /opt/demo demo1
  [root@MyHost home]# ll /opt
  drwx------ 2 demo1 demo1 4096 Aug  4 14:28 demo   
  
    #默认情况下，会自动创建以用户名为默认的组，可以使用-g指定组，这就是属组的权限，也就是说我们配置demo1的权限也就相当于配置了整个组的权限，-g创建主组，有且只有一个
  [root@MyHost demo]# useradd -g demo1 alex  #将alex用户指定到demo1用户组下
  [root@MyHost demo]# id alex
  uid=1006(alex) gid=1005(demo1) groups=1005(demo1)
  
   #附加组可以有多个 -G
  [root@MyHost demo]# useradd -G demo1,root alex2
  [root@MyHost demo]# id alex2
  uid=1007(alex2) gid=1007(alex2) groups=1007(alex2),0(root),1005(demo1)
  
   # -N 不创建组，默认继承users，-M不创建家目录
  [root@MyHost demo]# id alex3
  uid=1008(alex3) gid=100(users) groups=100(users)
  
   # -r 创建系统用户， 普通用创建的id是以最大的已创建的id+1，系统用户是最小的id-1（以1000最大）
  [root@MyHost demo]# id alex4
  uid=996(alex4) gid=994(alex4) groups=994(alex4)
  
  [root@MyHost demo]# useradd -s /sbin/nologin alex5  #-s设置了用户设置的shell，此shell即使设置了shell，也无法登陆，登陆后即刻弹出
  
   # -u设置uid，此后都是2000之后创建，除非重新指定-u
  [root@MyHost demo]# useradd -u 2000 alex6
  [root@MyHost demo]# id alex6
  uid=2000(alex6) gid=2000(alex6) groups=2000(alex6)
   
   # 查看useadd命令的配置信息
  [root@MyHost demo]# vim /etc/default/useradd 
  GROUP=100
  HOME=/home
  INACTIVE=-1
  EXPIRE=
  SHELL=/bin/bash
  SKEL=/etc/skel    #这里家目录的隐藏的配置文件就是从这里复制过去的
  CREATE_MAIL_SPOOL=yes
  
   #-D显示默认的配置
  [root@MyHost skel]# useradd -D
  GROUP=100
  HOME=/home
  INACTIVE=-1
  EXPIRE=
  SHELL=/bin/bash
  SKEL=/etc/skel
  CREATE_MAIL_SPOOL=yes
   
   # 修改默认的配置
  useradd  -D -s /sbin/nologin 修改默认的登录后shell
  useradd  -D -b /opt/ 修改默认的家目录
  useradd  -D -g 3000 修改默认的组
  ```

  - **总结**：

    ```shell
    -d 指定用户的家目录，不能创建在/tmp，默认用户的家目录不需要手动创建
    -g 组信息   主组有且只能有一个
    -G 指定附加组 可以有多个
    -M 不创建家目录
    -N 不创建组，默认继承至user组
    -r 创建一个系统用户，id从1000依次递减
    -s 登录以后使用的shell /sbin/nologin 可以登录看到提示，但是会立马被踢掉
    -u 指定uid 
    /etc/default/useradd 默认的配置文件
    -D 显示默认配置
    useradd  -D -s /sbin/nologin 修改默认的登录后shell
    useradd  -D -b /opt/ 修改默认的家目录
    useradd  -D -g 3000 修改默认的组
    
    /etc/default/useradd 默认配置文件
    /etc/skel/* 默认复制文件
    /etc/login.defs 默认用户与组设置文件
    ```

    

- ##### passwd 设置密码

  ```shell
  [root@MyHost home]# passwd --help    # 查看帮助
  
  passwd [options] username 用来设置密码
  [root@MyHost home]# passwd li   # 给li用户设置密码
  
  -d 删除用户的密码 不能登录
  [root@MyHost home]# passwd -d li
  
  -l 锁定用户
  [root@MyHost home]# passwd -l li
  -u 解锁用户
  -e 强制用户下次登录的时候修改密码
  [root@MyHost home]# passwd li    # 先设置密码
  [root@MyHost home]# passwd -e li   #下次登陆自动重置密码
  
  Changing password for li.
  (current) UNIX password: 
  
  -x maxdays 密码的最长有效期
  -n mindays 密码的最短有效期 
  -w wandays 提前多长时间开始警告用户
  -i days 密码过期多长时间以后账户被禁用
  --stdin 从标准输入读入数据 echo "password" |passwd --stdin username 
  [root@MyHost home]# echo "2" |passwd --stdin li  #有时候不想二次确认
  ```

- #####  chage 交互式修改用户信息

  ```shell
  [root@MyHost home]# chage li
  	Minimum Password Age [0]: 
  	Maximum Password Age [99999]: 
  	Last Password Change (YYYY-MM-DD) [2019-08-04]: 
  	Password Expiration Warning [7]: 
  	Password Inactive [-1]: 
  	Account Expiration Date (YYYY-MM-DD) [-1]: 
  
  -d 将密码修改时间设置为执行的时间
  -E 设置用户的过期时间
  -I 密码过期多长会时间以后账户被禁用
  -l 显示密码的策略
  -m 密码的最短使用期限
  -M 密码的最长使用期限
  -W 密码过期的警告天数
  ```

- ##### shadow文件格式

  ```shell
  [root@MyHost home]# vim /etc/shadow
  ....
  bin:*:17110:0:99999:7:::
  ....
  1、登录用户
  2、用户密码：一般用sha512加密
  3、从1970年1月1日起到木马最近一次被更改的时间
  4、密码再过几天可以被变更（0表示随时可以被变更）
  5、密码再过几天必须别变更（99999表示永不过期）
  6、密码过期前几天系统系统用户（默认为一周）
  7、密码过期几天后账户被锁定
  8、从1970年1月1日算起，多少天后账户失效
  ```

  

- ##### passwd文件格式

  ```shell
    #所有的用户信息写在 /etc/passwd/中
  login name 登录用户名
  passwd 密码占位符（x）
  UID 用户身份编号
  GID 登录默认所在组编号
  GECOS 用户全名或注释
  home directory 用户家目录
  shell 用户登录后默认使用的shell
  ```

- ##### usermod 修改用户信息

  ```shell
  -L  #锁定用户
  [root@MyHost skel]# usermod -L alex4
  [root@MyHost skel]# passwd alex4
  
  -U 解锁
  [root@MyHost skel]# usermod -U alex4
  
  -d 新的家目录不会自己创建，要想创建要使用-m选项
  [root@MyHost opt]# useradd -d /opt/demo/ demo3   # 创建用户和家目录
  [root@MyHost demo]# usermod -md /opt/demo3/ demo3  #修改家目录
  [root@MyHost demo]# passwd demo3   #设置密码
  
  -bash-4.2$ pwd   #用户登陆后的位置发生改变
  /opt/demo3/    
  
  -g 修改主组
  -G 修改附加组
  -a 追加附加组  
  [root@MyHost demo]# id demo3
  uid=2002(demo3) gid=2002(demo3) groups=2002(demo3)
  [root@MyHost demo]# usermod -a -G alex4 demo3  # 将alex4添加到demo3
  [root@MyHost demo]# id demo3
  uid=2002(demo3) gid=2002(demo3) groups=2002(demo3),994(alex4)
  
  -l 改名
  [root@MyHost demo]# usermod -l x demo3
  [root@MyHost demo]# id x
  uid=2002(x) gid=2002(demo3) groups=2002(demo3),994(alex4)
  
  -s 修改登录后的shell 
  -u 修改uid
  -e yyyy-mm-dd 指明用户账户过期日期
  -i inactivedays 密码过期后经过多少天改账户会被禁用
  ```

- ##### userdel 删除用户

  ```shell
  userdel [option] login
  -r 删除用户家目录
  [root@MyHost home]# userdel -r demo2
  
  -f 强制删除
  ```



#### 用户切换

- ##### su

  ```shell
  - su username 切换用户，切换用户要输入切换到的用户的密码
  [root@MyHost ~]# su li    # root 随意切换，一般用户需要密码
  [li@MyHost root]$ 
  
  [li@MyHost ~]$ cd /root   # 原本li用户是无法访问root文件的，但是此时是root切换，所以可以，但是没有权限访问文件
  -bash: cd: /root: Permission denied
  
  - su - username 完全切换，会切换用户的目录还会切换用户的环境变量
  [root@MyHost ~]# su - li
  Last login: Sun Aug  4 16:31:27 CST 2019 on pts/0
  [li@MyHost ~]$ 
  
  - root 切换到别的用户不需要输入密码
  ```

- ##### 切换到相应的用户，执行命令后返回当前用户

  ```shell
  [li@MyHost ~]$ su - root -c "ls /root"   # - 可写可不写
  Password: 
  2  b.bz2  b.txt.bz2  c.txt  c.txt.xz  error.log  info.log  pa  passss.txt  pass.txt.bz2  passwd.gz  test  test2  xxx.tar
  ```

- ##### sudo  使用root的权限执行命令

  ```shell
  [li@MyHost ~]$ sudo useradd demo2
  
  We trust you have received the usual lecture from the local System
  Administrator. It usually boils down to these three things:
  
      #1) Respect the privacy of others.
      #2) Think before you type.
      #3) With great power comes great responsibility.
  
  [sudo] password for li:   #输入li的密码，但是li没在sudo的名单中，也就是说没权力执行root的权限
  li is not in the sudoers file.  This incident will be reported.
  
  -----------------------------------------------
   #切换到root用户，修改sudoers名单
  [root@MyHost li]# vim /etc/sudoers
  
   #打开sudoers文件，在里面搜索/root，使用n向下查找，在下面一行添加li用户的名单
   root    ALL=(ALL)       ALL
   li      ALL=(ALL)       ALL
  
   #上面修改后，wq！保存，在li用户中使用sudo
  [li@MyHost ~]$ sudo ls /root/
  [sudo] password for li:  #输入自己的密码
  2  b.bz2  b.txt.bz2  c.txt  c.txt.xz  error.log  info.log  pa  passss.txt  pass.txt.bz2  passwd.gz  test  test2  xxx.tar
  
  
   #完全体sudo，根本不需要输入密码，直接执行，修改之前的配置文件
  root    ALL=(ALL)       ALL
  li      ALL=(ALL)       NOPASSWD: ALL
  ```

  - ##### 小提示：

    ```shell
    用root用户修改/etc/sudoers文件
    修改下面配置文件
    xiaofeng ALL=(ALL（命令）)      NOPASSWD（不需要输入密码）: ALL
    
    表示一个组
    %wheel  ALL=(ALL)       ALL
    ```



#### 用户组

- 超级用户组  root 0

- 普通用户组

  - 系统组  gid 1-999（centos7）1-499（centos6）
  - 可登陆用户组 gid 1000-65535（centos7） 500-65535（centos6）

- ##### groupadd 创建用户组

  ```shell
  [root@MyHost li]# groupadd demo4
  [root@MyHost li]# cat /etc/group    # 只能在此看group信息
  
  -g 用来指定gid
  -r 用来指定系统组
  [root@MyHost li]# groupadd -r dem   # 添加一个系统组
  [root@MyHost li]# vim + /etc/group
  ...
  dem:x:993:
  
  ```

- ##### /etc/group/的文件格式

  ```shell
  [root@MyHost li]# cat /etc/group 
  alex4:x:994:x
  demo3:x:2002:
  
  - 组名
  - 密码占位符
  - gid
  - 以当前组为附加组的用户
  ```

- ##### groupmod和groupdel

  ```shell
  groupmod [option] ... groupname
  -n group_name 新名字
  -g GID 新的GID
  [root@MyHost li]# groupmod  -g  480 demo4
  
  groupdel groupname
  ```

- ##### gshadow的配置文件

  ```shell
  [root@MyHost li]# vim /etc/gshadow
  root:::
  ....
  群组名称
  群组密码
  组管理员密码：组管理员的列表，更改组密码和成员
  以当前组为附加组的用户列表
  ```

  

#### 登陆远程机器

**两种认证方式：**

- 用户名+密码（一般的方式）

- 用户名+key

  ```shell
  [root@MyHost li]#ssh-keygen    #生成key，默认保存在/root/.ssh/id_rsa中
   
   #此处会提示输入两次密码，不填就表示对方链接不输入密码，
   #上面的命令可以不设置密码，生成两个文件：一个公钥、一个私钥：
   /root/.ssh/id_rsa
   /root/.ssh/id_rsa.pub
  
   # 怎么使用？
  ssh-copy-id 复制key到远程机器（公加私解，也就是说公钥是给别人的加密过的，私钥是解密的）公钥加密私钥解密
  
  [root@MyHost li]# ssh-copy-id root@192.168.182.130 #将公钥给它
   #会提示输入一次它的登陆密码，以后都不需要输入密码了（如果前面的两次密码为空）， 两台机器互联，
  [root@MyHost li]# ssh root@192.168.182.130   #直接登陆另一台机器
  ```

  

#### 磁盘

- ##### df  磁盘占用率

  ```shell
  查看磁盘占用率
  [root@MyHost li]# df
  Filesystem     1K-blocks    Used Available Use% Mounted on
  /dev/vda1       41151808 1990572  37047804   6% /
  
  -h 显示人类可读的信息，对单位进行了转化
  [root@MyHost li]# df -h
  ```

- ##### mount 查看挂载信息

  ```shell
  [root@MyHost li]# mount
  ```

- ##### du  显示目录的占用空间

  ```shell
  -h 显示人类易读的信息
  [root@MyHost ~]# du     #没有指定路径，表示当前目录下的文件的大小
  8	./.oracle_jre_usage
  38296	./test2
  ...
  
  -s 显示的是目录本身
  [root@MyHost ~]# du -s
  38540	.
  [root@MyHost ~]# du -sh    #一类可读的模式显示
  38M	.
  [root@MyHost ~]# du -sh /  #显示根目录大小
  1.9G	/
  
  [root@MyHost ~]# du -sh /*   # 显示根下所有文件的目录大小
  ...
  0	/sys
  1.1M	/tmp
  1.5G	/usr
  189M	/var
  ```

- ##### dd 生成文件

  ```shell
  [root@MyHost ~]# dd if=/dev/zero of=/dev/null bs=2M count=2
  2+0 records in
  2+0 records out
  4194304 bytes (4.2 MB) copied, 0.00142904 s, 2.9 GB/s
  
  [root@MyHost ~]# dd if=/dev/zero of=test1 bs=2M count=2  #当前目录生成test1文件
  2+0 records in
  2+0 records out
  4194304 bytes (4.2 MB) copied, 0.00372646 s, 1.1 GB/s  #测速
  ```

  - ##### 总结

    ```shell
    dd if=/dev/zero of=/dev/null bs=10M count=2
    bs=#:block size, 复制单元大小
    count=#:复制多少个bs
    of=file 写到所命名的文件而不是标准输出
    if=file 从所命名文件读取而不是标准输入
    ```

    

#### RAID 整列卡

将多个硬盘做成一个硬盘

##### RAID0

- 读、写性能提升，根据你的硬盘数提升

- 可用空间N*大小（还是为N）

- 无容错能力，其中任意一块坏都坏

- 最少磁盘数 2

  ![1564914303064](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564914303064.png)

##### RAID1

- 读性能提升、写性能略有下降, 根据硬盘个数下降写的速度

- 可用空间 大小/个数

- 用冗余能力，相当于多了一个备份硬盘

- 最少磁盘数 2 N

  ![1564914419568](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564914419568.png)

##### RAID5

- 读、写性能提升
- 可用空间（n-1）*大小
- 有容错能力 允许最多1块磁盘损害
- 最少磁盘数 3，3+

- RAID5校验位算法原理

  P=D1 xor D2 xor D3 … xor Dn （D1,D2,D3 … Dn为[数据块](https://baike.baidu.com/item/数据块)，P为校验，xor为异或运算）

  XOR(Exclusive OR)的校验原理如下表：

| A值  | B值  | Xor结果 |
| ---- | ---- | ------- |
| 0    | 0    | 0       |
| 1    | 0    | 1       |
| 0    | 1    | 1       |
| 1    | 1    | 0       |

![1564915324578](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564915324578.png)



##### RAID6

- 读、写性能提升
- 可用空间（N-2）*大小
- 有容错能力：允许坏2块硬盘
- 最少磁盘数: 4

![1564915827520](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564915827520.png)



##### RAID10

- 读、写性能提升
- 可用空间：N*大小/2
- 有容错能力：没组镜像最多只能坏一块
- 最少磁盘数：4

![1564915901154](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564915901154.png)

##### RAID01

- 多块磁盘先实现raid0 在实现raid1

![1564915932142](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564915932142.png)



#### 软件包管理

##### 程序包管理器

​		功能：将编译好的应用程序的各组成部分打包一个或者几个程序包文件，从而方便快捷的实现程序包的安装、卸载、查询、升级和校验等管理工作

- debian：deb文件 dpkg包管理器

- redhat：rpm文件，rpm包管理器

- rpm： Redhat Package Manager RPM Package Manager

- windows exe文件

##### 包命名格式

- **源码**：name-version.tar.gz|bz2|xz

- rpm包 命名方式：name-version-release.arch.rpm

  ```
  python-2.7.5-80.el7_6.x86_64
  ```

  - version 版本 major.minor.release

  - release：relases.os  （作者的更新次数）

  - arch：架构

    - x86：i386 i486 i586 i686

    - x86_64:x64,x86_64,amd64

    - powerpc:ppc

    - 跟平台无关：noarch

      ```shell
      [root@MyHost ~]# rpm -q yum
      yum-3.4.3-150.el7.centos.noarch
      ```

      

##### 程序包的来源

- 管理程序包的方式：rpm或者yum
- 获取软件包的途径：
  - 系统发版的光盘或者官方的服务器
    - https://www.centos.org/download/
    - [http://mirrors.aliyun.com](http://mirrors.aliyun.com/)
    - [http://mirrors.sohu.com](http://mirrors.sohu.com/)
    - [http://mirrors.163.com](http://mirrors.163.com/)
  - 项目官方站点
  - 第三方组织
    - epel
    - [http://pkgs.org](http://pkgs.org/)
    - [http://rpmfind.net](http://rpmfind.net/)
    - https://sourceforge.net/
  - 自己制作
  - 第三方包建议要检查其合法性，来源合法性，程序包的完整性



##### yum

​		yum : rpm的前端程序，可解决软件包相关依赖性，可以在多个库之间定位软件,它会**自动解决你安装的包的所有依赖问题**

​		yum 存储了多个rpm包，以及包的相关元数据（放在特定目录下的repodata下

##### yum的命令

```shell
安装包  yum install xxx   # 安装xxx软件
 #清除缓存  yum clean
[root@MyHost ~]# yum clean all   #清除所有缓存

 #列出所有的包 yum list
[root@MyHost ~]# yum list|grep *.i386

 #列出需要更新的包：yum check-update
[root@MyHost ~]# yum check-update

 #更新包 yum update
[root@MyHost ~]# yum update python   # 会自动安装相关依赖的包

 #搜索  yum search
[root@MyHost ~]# yum search python3

 #详细信息 yum info
[root@MyHost ~]# yum info python

 #列出yum仓库信息 yum repolist    #列出仓库信息，里看又多少rpm包
[root@MyHost ~]# yum repolist
repo id     repo name        status
base/7/x86_64  CentOS-7                                               10,019
epel/x86_64     Extra Packages for Enterprise Linux 7 - x86_64        13,329
extras/7/x86_64 CentOS-7                                              419
updates/7/x86_6  CentOS-7                                             2,500
repolist: 26,267

 #重新安装 yum reinstall xxx   
[root@MyHost ~]# yum reinstall redis

 #卸载包 yum remove
[root@MyHost ~]# yum remove redis

 #检查的依赖关系 yum deplist
[root@MyHost ~]# yum deplist python   # 检测python包依赖谁
 
 #列出包组列表
[root@MyHost ~]# yum group list
 
 #列出安装包组
[root@MyHost ~]# yum group install

 #列出安装包组信息
[root@MyHost ~]# yum group info "Development Tools"

 #重建缓存 yum makecache
[root@MyHost ~]# yum makecache 

 # 搜索指定的命令由那个包生成
[root@MyHost yum.repos.d]# yum provides cd
Filename    : /usr/bin/cd
...
Filename    : /bin/cd
...
Filename    : /usr/bin/cd
```

- **总结**

  ```
  命令语法 yum [options] [command] [pkg]
  显示仓库列表 yum repolist [all|enabled|disabled]
  显示程序包 yum list [installed|updates]
  安装程序包 yum [install|reinstall] pkg1 pkg2
  升级程序包 yum update pkg1 pkg2
  降级程序包 yum downgrade pkg1 pkg2
  检查可用升级 yum check-update
  卸载程序包 yum remove pkg1 pkg2
  查看程序包 yum info
  清理本地缓存：/var/cache/yum/$basearch/$releasever缓存
  yum clean [packages|all]
  重新构建缓存 yum makecache
  搜索 yum search string1 以指定的关键字搜索程序包名以及summary信息
  查看指定包所依赖的包 yum deplist pkg1
  日志 /var/log/yum.log
  安装本地包 yum install rpmfile1 rpmfile2
  ```

  

##### rpm命令

```shell
 #rpm -q package 检查这个包是否安装
[root@MyHost ~]# rpm -q redis
package redis is not installed

[root@MyHost ~]# rpm -q python
python-2.7.5-80.el7_6.x86_64

 #-qa 列出所有已经安装的包
[root@MyHost ~]# rpm -qa|grep python  #列出所有安装的python的包

 #-qf 查询文件由那个包生成
[root@MyHost ~]# rpm -qf /etc/redis.conf
openssh-server-7.4p1-16.e17.x84_64
 
 #-l  查询包生成的文件
[root@MyHost ~]# rpm -ql python
/usr/bin/pydoc
/usr/bin/python
/usr/bin/python2
...

 #-i 查询包的信息
[root@MyHost ~]# rpm -qi redis

 #-c 查找包生成的配置文件
[root@MyHost ~]# rpm -qc redis
/etc/logrotate.d/redis
/etc/redis-sentinel.conf
...

 #rpm卸载包
[root@MyHost ~]#rpm -e Package 卸载
```

- ##### 总结

  ```shell
  rpm -q [select-options] [query-options]
  [select-options]
  -a 所有包
  -f 查看指定的文件由那个程序包安装生成
  rpm -q -f /etc/ssh/sshd_config
  [query-options]
  --changelog 查询rpm包的changelog rpm -q --changelog openssh-server
  -c 查询程序的配置文件
  -d 查询程序的文档
  -i 程序说明信息
  -l 查看指定的程序包安装后生成的所有文件
  -R 查询指定的程序包所依赖的库文件
  #常用用法
  -qi
  -qc
  -ql
  -qd
  -qa
  ```



##### rpm包安装

```shell
rpm -i [install-options] pkg_name
-v 详细信息
-h 以#显示程序包管理执行进度
install-options
   --test 测试安装，但是不真正执行安装
   --nodeps 忽略依赖关系
   --nosignature 不检查来源合法性
   --nodigest 不检查包完整性
   
rpm -ivh pkg_name
```



##### 软件包升级

```shell
rpm -U [install-options] pkg_name
rpm -F [install-options] pkg_name
upgrade 安装有旧版本包，则‘升级’，如果不存在旧版本包，则‘安装’
freshen 安装有旧版本包，则‘升级’，如果不存在旧版本包，则不执行升级操作
--oldpackage 降级
--force 强制安装
```



#### yum 参考的配置文件

**位置**：/etc/yum.repos.d/

```shell
[root@MyHost ~]# cd /etc/yum.repos.d/
[root@MyHost yum.repos.d]# ll
total 8
-rw-r--r-- 1 root root 675 Jul 30 23:41 CentOS-Base.repo
-rw-r--r-- 1 root root 230 Jul 30 23:41 epel.repo    #这里面装的是第三方提供的包

```

**后缀**：repo

```shell
[root@MyHost yum.repos.d]# vim epel.repo 
...
[epel] #名字 通过yum repolist可以查看这个名字                                                                                           
name=Extra Packages for Enterprise Linux 7 - $basearch #描述信息
baseurl=http://mirrors.aliyun.com/epel/7/$basearch #仓库的地址 可以是http:// https:// ftp:// file://(本地)
failovermethod=priority # 设置访问规则
enabled=1  #是否禁用 0表示禁用 1表示启用，设置为0，这个第三方包就用不了了，且yum repolist也看不到了
gpgcheck=0  # 要不要检查key，1表示检查 0表示不检查
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 
$release 系统版本
$basearch 架构
```



##### yum 命令行选项

```
- - nogpgcheck 禁止进行gpg check

-y 自动回答为 “yes”
-q 静默模式
```



##### 程序包的编译安装

- 优点：可以自定义功能
- 缺点：安装比较耗时

```shell
[root@MyHost yum.repos.d]#yum install zlib-devel
[root@MyHost yum.repos.d]#yum install gcc     # 为编译做准备
[root@MyHost yum.repos.d]#wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tar.xz  # 先下载包
[root@MyHost yum.repos.d]#tar xf Python-3.6.8.tar.xz     #将下载，可以在到解压好的文件中有一个configure文件，我们可以
[root@MyHost yum.repos.d]#./configure --help   #查看帮助
[root@MyHost yum.repos.d]#./configure --prefix=/opt/python36   #检查环节预处理，指定安装目录
[root@MyHost yum.repos.d]#make   #释放makefile文件，依赖于gcc，必须安装
[root@MyHost yum.repos.d]#make install   # 真正的安装


---------------------------安装好后就在/opt/python36文件生成----------------------------
 #cd进去看看
[root@MyHost python36]#cd bin/  # 进去可以发现由python可执行文件
 
 #创建软连接/添加环境变量
[root@MyHost python36]#PATH=$PATH:/opt/python36/bin
[root@MyHost python36]#python3   #成功

 #但是退出系统后，重新登陆，就无效了
[root@MyHost python36]#vim /etc/profile.d/python.sh
 # 在里面写：PATH=$PATH:/opt/python36/bin  保存退出
[root@MyHost python36]# source /etc/profile.d/python.sh  #重载此文件 ，可不用重启
```

- ##### 小提示

  ```shell
  [root@MyHost yum.repos.d]# yum install lrzsz   # 此方法可以再windows在linux中文件拖动
  [root@MyHost yum.repos.d]#rz   #运行后就可以拖动文件了
  
   #如果安装出错了，必须删除解压包中的Makefile
  [root@MyHost yum.repos.d]#rm -rf Makefile
  [root@MyHost yum.repos.d]#./configure --prefix=/opt/python36   #重新配置
  [root@MyHost yum.repos.d]#make  
  [root@MyHost yum.repos.d]#make install  
  ```

  

- 下载
  
  - wget
- 解压
  
  - tar
- 编译
  
  - ./configure
- 安装
  - make 构建应用程序
  - make install 复制文件到相应目录
- 安装前可以查看INSTALL，README文件



##### 包组管理的相关命令

```
yum groupinstall group1
yum groupupdate group1
yum grouplist group1
yum groupremove group
yum groupinfo group1
```



##### 系统光盘yum仓库

```
1、挂在光盘值某目录，mount /dev/cdrom /mnt/cdrom
2、创建配置文件指定baseurl为file://mnt/cdrom
```

