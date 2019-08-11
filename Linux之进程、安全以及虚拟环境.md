## Linux 之网络、任务、进程以及虚拟环境



#### 计划任务

通过规定时间做特定的行为，叫计划任务。linux中也有一次性任务，但用得并不是很多

**一次性任务**：创建at任务方式有两种，从文件输入和从控制台输入。以下分别用两种方式创建1分钟后将当前时间写入 /root/learngit 文件的命令

- 文件输入：

  ```shell
  [root@MyHost ~]# touch at.sh   #创建一个.sh文件，标识是一个bash可识别文件
  [root@MyHost ~]# ls   #当前文件存在
  at.sh
  
  [root@MyHost ~]# vim at.sh   #再文件中 写要执行的命令
  
  #!/bin/bash
  #必须使用上面一句选择响应的处理命令
  echo `date` >> result.txt
  
  [root@MyHost ~]# at -f at.sh now +1 minutes   #一分钟后执行at.sh文件，-f指定文件
  
  job 2 at Sun Aug 11 10:38:00 2019
  [root@MyHost ~]# atq     #打印at的排队队列，之后可以根据序号进行删除
  2	Sun Aug 11 10:38:00 2019 a root
  [root@MyHost ~]# ls   #成功生成文件
  at.sh  result.txt
  ```

- 控制台输入：

  ```shell
  [root@MyHost ~]# at now +2 minutes
  at> echo `date` >> result.txt  
  at> <EOT>    #ctrl+d结束输入
  job 3 at Sun Aug 11 10:44:00 2019
  ```



#### 计划任务

常用于：日志、备份、同步时间

- **方式一**：时间表达法

  ```shell
  [root@MyHost ~]# vim /etc/cron    #下面分别都是关于可编写的计划规则
  cron.d/       cron.daily/   cron.deny   cron.hourly/  cron.monthly/ crontab       cron.weekly/  
  ```

  -  [root@MyHost ~]# vim /etc/crontab   编写计划任务的文本

    ```shell
    SHELL=/bin/bash
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    MAILTO=root
    
    # For details see man 4 crontabs
    
    # Example of job definition:
    # .---------------- minute (0 - 59)  #分钟
    # |  .------------- hour (0 - 23)    #时间
    # |  |  .---------- day of month (1 - 31)  #天数
    # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...  #月
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR  #星期sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * user-name  command to be executed #用户 命令
    
    * 代表所有
    * * * * * root echo "1" >> /tmp/a.txt #每分钟做什么事
    0 * * * * root echo "0" >> /tmp/b.txt #每个小时的第0分钟做什么事
    0 4 * * * root echo "0" >> /tmp/c.txt #每天四点做什么
    0 4 3 * * root echo "0" >> /tmp/c.txt #每个月三号4点做什么
    0 4 * * 4 root echo "0" >> /tmp/c.txt #每周四的四点做什么
    0 4,6,8 * * * root echo "0" >> /tmp/d.txt  #每天的4,6,8点的时候做什么事
    0 8-22 * * *  root echo "上课" >> /tmp/f.txt #每天的早上8点到22点做什么事
    0 8-22/3 * * * root echo "休息" >> /tmp/e.txt  #每天的早上8点到22点每隔3个小时做什么事
    41 *  3,15,25,30 * 6 echo "da" >> /tmp/g.txt  #每个月的3,15,25,30 或者每周6做什么事 (特殊的)
    
        * 代表所有
        #1,#2,#3 #1或者#2或者#3
        #1-#2 #1到#2
        /# 每隔#时间
    ```

    

- **方式二：**crontab

  ```shell
  [root@MyHost ~]# crontab -h  #查看帮助
  -u 指定用户 默认是当前用户
  -e  编辑
  -l 列出
  -r 删除
   #不是真正的存在/etc/crontab,而是存在/var/spool/cron/,这个目录下会为每一个用户创建一个文件
  
  [root@MyHost ~]# crontab -e    #默认给当前用户创建一条定时任务
  15 * * * * echo "dada" >> /root/1.txt   # 写入内容，在每个小时的15分钟写入一条记录
  
  [root@MyHost ~]# crontab -e -uwang   #给wang用户添加一条定时任务
  [root@MyHost ~]# crontab -l     #列出所有的定时任务
  * * * * * echo "data" >> 2.txt
   
  [root@MyHost ~]# crontab -r  #删除所有的定时任务
  ```

  

#### 网络配置

- ip总共多少位?  有多少段? 

  ```
  一个段是8位 总共32位  
   #又分为网络位和主机位
  192.168.182.128
  192.168.182 这个网段，128 主机位 ,可以通过主机来判断当前的网段内可以放多少台终端
  ```

- 网段

  ```
  A：前8位为网络位,后24位为主机位
          0  0000000  #第一位为0，后7为0
          0  1111111
  	所以能使用：1-126 ,127 为回环地址
  	可用网段: 2^7-1
  	主机: 2^24
      主机位全为0,表示网段
      主机位全为1,网段里面的广播地址
  	可用ip是多少:2^24-2
  	共有地址: 所有人都可以访问的地址
  	私有地址:只能内部访问的地址
   
  
  B: 前16位为网络位,后16位为主机位
          10 000000 00000000  128
          10 111111 11111111  191
  
  	可用网段: 2^14
  	可用ip: 2^16-2
  	私有地址: 72.16-172.31
  
  C:前24位为网络位,后8位为主机位
          110 00000  0  0 
          110 11111 255 255
      可用网段 2^21
      可用ip地址 2^8-2
      私有网段
          192.168.0
          192.168.255
  
  D 广播 多播的地址:224-239
  
  E 留作科研使用:240-254
  ```

  - 10.23.34.56/15的网段

    ```
    10.23.34.56/15
    
    10.00010111.34.56
    0000 1010 00010111 
    1111 1111 11111110
    
    10.22.0.0
    可用ip地址:2^17-2
    ```

- ##### CIDR(无类域间路由)

  ```
  网络位向主机位借位
  ip地址/子网掩码 与运算
  子网掩码 网络位全为1,主机位全为0
  ```

- ##### 运算规则

  ```
  与：   全为1才位1,只要有0则为0
      >>> 1&0
      0
      >>> 2&3
  	2
  
  或：   有1则为,全为0才为0
      >>> 1|2
      3
      >>> 5|1
  	5
  
  异或： 相同为0,不同为1
      >>> 2^3
      1
  
  取反： -(n+1)
      >>> ~1
      -2
  
  左移： 2<<2      n*2位移倍数次方
  	>>> 4<<2
  	16
  
  右移： 12>>2   n/2位移倍数次方 向下取正
  ```

  

#### ip获取方式

##### dhcp服务器 分配ip地址

##### 手动

- `ifconfig` 命令查看ip信息 

  ```shell
  [root@MyHost ~]# ifconfig 
  ```

- ##### ip 命令

  - 给当前网卡添加ip地址

    ```shell
    [root@MyHost ~]# ip a    #查看详细的ip信息
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:16:3e:08:6d:6b brd ff:ff:ff:ff:ff:ff
        inet 172.24.27.91/19 brd 172.24.63.255 scope global dynamic eth0
           valid_lft 314751915sec preferred_lft 314751915sec
    
    [root@MyHost ~]#ip addr add 192.168.182.200/24 dev eth0  #dev是固定的，eth0是当前网卡
    [root@MyHost ~]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:16:3e:08:6d:6b brd ff:ff:ff:ff:ff:ff
        inet 172.24.27.91/18 brd 172.24.63.255 scope global dynamic eth0
           valid_lft 314751915sec preferred_lft 314751915sec
        inet 192.168.182.200/24 scope global eth0  #新增的ip，可再另一个窗口使用这个网站登陆
           valid_lft forever preferred_lft forever
    
     #注意：我们设置的ip是内部网址可以访问，如果你在当前root窗口使用ssh wang@192.168.182.200是可以访问的，但是在本地xshell中新建窗口就不行了
     
    [root@MyHost ~]# ip addr add 192.168.182.201/24 dev eth0:1
     ...
     eth0:2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.182.204  netmask 255.255.255.0  broadcast 0.0.0.0
            ether 00:16:3e:08:6d:6b  txqueuelen 1000  (Ethernet) #会创建一个挂载到eth0
    
    [root@MyHost ~]# ip a del 192.168.182.204/24 dev eth0  #删除ip地址
    ```

  

- ##### 网卡配置

  ```shell
  [root@MyHost ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0   #上面的配置重启之后
  
  TYPE=Ethernet            网卡的接口类型:     Ethernet Bridge
  PROXY_METHOD=none  
  BROWSER_ONLY=no
  BOOTPROTO=dhcp  #获取ip地址的协议: dhcp、 none 、static #重启后ip地址可能会变
  DEFROUTE=yes
  IPV4_FAILURE_FATAL=no
  IPV6INIT=yes
  IPV6_AUTOCONF=yes
  IPV6_DEFROUTE=yes
  IPV6_FAILURE_FATAL=no
  IPV6_ADDR_GEN_MODE=stable-privacy
  NAME=ens33   #网卡名称
  UUID=04faa6f8-44e8-479a-aa55-2df5783ce516  #宇宙唯一id uuid
  DEVICE=ens33   #设备
  ONBOOT=yes   #开机是否自启动
  
  IPADDR=192.168.182.130  #ip地址，静态网址
  NETMASK= 255.255.255.0  #子网掩码
  GATEWAY= 192.168.182.2 #网管
  DNS1=114.114.114.114 #dns1
  DNS2=8.8.8.8  #dns2
  ```

  - 常用命令

    ```shell
    [root@MyHost ~]# systemctl restart network
    
    [root@MyHost ~]#vim /etc/resolv.conf  #存放dns的配置文件
    ```

  - 本地解析

    ```shell
    可以写主机和ip地址的映射关系，并且先检查此文件
    [root@MyHost ~]#vim /etc/hosts
    ```

#### ss/netstat

打印网络系统的状态

- 常见命令

  ```shell
  -t   #tcp 
  [root@MyHost ~]# ss -t  #查看使用tcp的网络连接
  State   Recv-Q Send-Q   Local Address:Port   Peer Address:Port                
  ESTAB       0      0    192.168.182.200:ssh    192.168.182.200:47732                
  
  -u  #udp
  [root@MyHost ~]# ss -u
  
  -x  #套接字
  [root@MyHost ~]# ss -x
  
  -a #所有
  [root@MyHost ~]# ss -a
  
  -l #处于监听的
  [root@MyHost ~]# ss -l
  
  -p #相关的程序及pid，正在使用socket的进程
  [root@MyHost ~]# ss -p
  
  -n #显示端口，不显示名字
  
   #各种端口
      22 ssh
      http 80
      mysql 3306
      redis 6379
      mongdb 27017
      windows远程桌面 3389
      oracle数据库 1521
      https 443
      ftp  20    21
      
  ss常用的组合: -tnlp -aulp -tan
  [root@MyHost ~]# ss -tnlp|grep 22  #显示22端口的服务的tcp连接
  
  [root@MyHost ~]# ss -aulp|grep 22  #显示所有的udp的22端口使用情况
  
  
  
  ```

  

#### wget下载文件

- 常见参数

  ```shell
  -O filename 指定生成的文件名
  -P 保存到指定的目录
  -q 静默模式
  -r 递归下载
  -p 下载所有的html元素
  
  wget http://www.xiaohuar.com/d/file/20190809/small620192446e4599c844fc40a3e7b119141565366028.jpg
  wget -P /tmp http://www.xiaohuar.com/d/file/20190809/small620192446e4599c844fc40a3e7b119141565366028.jpg
  wget -O /tmp/xiaohua.png http://www.xiaohuar.com/d/file/20190809/small620192446e4599c844fc40a3e7b119141565366028.jpg
  wget -p http://www.xiaohuar.com/
  wget -q -O /tmp/xiaohua2.png http://www.xiaohuar.com/d/file/20190809/small620192446e4599c844fc40a3e7b119141565366028.jpg
   #其中 -P 和-O 选项冲突
  ```

  

#### 进程

- ##### ps：查看进程

  ```shell
  [root@MyHost ~]#  ps
  PID TTY          TIME CMD    #PID  终端  cpu执行时间  命令
  12715 pts/1    00:00:00 su
  
  [root@MyHost ~]# man ps
  
  ```

- ps输出属性

  ```
  VSZ  #虚拟内存集
  RSS  #常驻内存集合
  STAT #进程状态
  ni   #nice值
  pro  #优先级
  psr  #cpu编号
  ptprio #实时优先级
  ```

  

- 支持的方式

  - unix格式 如：-a -e
  - BSD格式   如：aux
  - GNU格式  如：--help

- 常用命令

  ```shell
  [root@MyHost ~]# ps a  #查看所有ps
  [root@MyHost ~]# ps x  #显示不连接终端显示的进程
  [root@MyHost ~]# ps u  #显示进程所有者信息
  [root@MyHost ~]# ps f  #显示进程数信息
  [root@MyHost ~]# ps L  #显示所有的属性
  [root@MyHost ~]# ps o %cpu,%mem   # 按照指定的属性来显示信息
  %CPU %MEM
   0.0  0.0
  [root@MyHost ~]# ps k %mem    #用来按照内存排序,后面执行排序的属性,如果需要倒叙的话,属性前面加上-
  
  -e  #显示所有的进程
  -f  #显示完整的信息
  -U  #username 用来指定用户
  
  常用组合: aux -ef   #需要跟grep做结合
  [root@MyHost ~]# ps aux
  [root@MyHost ~]# ps -ef
  [root@MyHost ~]# ps -ef|grep ssh
  root      2125     1  0 Aug04 ?        00:00:00 /usr/sbin/sshd -D
  root     11191  2125  0 10:19 ?        00:00:00 sshd: root@pts/0
  
  
  ```

- **pidof** ：跟进名称来查进程

  ```shell
  [root@MyHost ~]# pidof sshd
  12671 12665 11191 2125
  ```

- **kill ：**杀死进程，向进程发送信号,实现对进程的管理,每个信号都有不同的数字对应,

  ```shell
   #根据上面pidof  进程名 查出进程pid，进行结束进程
  root@MyHost ~]# kill 2125  #杀死2125进程
  
   #查看kill 命令帮助
  [root@MyHost ~]# kill -l
   1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP...
  
   #常用信号
  -	1 : sighup 重读配置文件，平滑重启，比如：重启时，其他用户不会被断掉
  -	2: sigint 终止正在运行的进程,相当于ctrl+c
  -	9:sigkill 强制杀死正在运行的进程
  -	19:sigstop:后台休眠
  -	18:sigcont 继续运行
  ```

- ##### kill 的不同模式

  ```shell
  按照pid kill pid
  [root@MyHost ~]# kill -9 2125  #强制杀死
  [root@MyHost ~]#kill -9 $(ps -ef | grep hnlinux) //方法一 过滤出hnlinux用户进程 
  [root@MyHost ~]#kill -u hnlinux //方法二
  
  按照名称  killall name 
  
  杀死所有同名进程
  [root@MyHost ~]#killall nginx
  [root@MyHost ~]#killall -9 bash  
  ```

  

#### 系统工具

##### uptime

```shell
[root@MyHost ~]# uptime
 16:01:40 up 7 days,  2:44,  2 users,  load average: 0.00, 0.01, 0.05
 #当前时间   系统开启的时长    当前在线的人数               1分钟  5分钟 15分钟
```

- 显示当前时间,系统开启的时长,当前在线的人数,系统的平均负载(1分钟,5分钟,15分钟)
- 平均负载: 在特定的时间内cpu上等待运行的进程数
- 如果不超过cpu核心数的2倍则认为是良好的

##### top

- ##### 排序
  - P 按照cpu排序

    ```SHELL
    [root@MyHost ~]# top  
     #SHIFT+p
    ```

  - M 按照内存排序
  - T 按照占用cpu的时长,TIME+

- ##### 首部信息显示：

  - uptime信息:l   （直接按l就可以了）
  - tasks和cpu信息:t
  - 内存信息:m
  - 分别显示cpu信息:1(数字)
  - 修改刷新时间 s
  - 杀死正在运行的进程 k
  - 退出 q

- 选项：

  - -d # 指定刷新时间

    ```
    [root@MyHost ~]# top -d 3  #3秒显示
    ```

  - -b 全部显示所有进程

  - -n # 刷新多少次后退出

    ```
    [root@MyHost ~]# top -n 3  #三次后退出
    ```

    

##### htop：epel源

- -d 指定刷新时间

  ```
  [root@MyHost ~]# yum install -y htop  #安装htop
  ```

- -u Username 仅显示指定用户的进程

- -s colume 以指定字段进行排序

- 子命令

  - s 根据选定进程的系统调用
  - l 显示选定进程打开的文件列表
  - a 讲选定的进程绑定至指定cpu核心
  - t 显示进程数



#### 性能分析

##### free

- 常用命令

  ```shell
  [root@MyHost ~]# free -m
                total        used        free      shared  buff/cache   available
  Mem:           1839         146         144          16        1548        1482
  -g 以GB的方式显示
  -h 易读格式
  -s n 指定刷新频率
  -c n 刷新n次后退出
  ```



##### vmstat

- 常用命令

  ```shell
  [root@MyHost ~]# vmstat
  procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
   r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
  
   #属性说明
  procs:
      r 可运行(正在运行或者等待运行)的进程个数
      b 处于不可中断的进程个数(被阻塞的队列长度)
  memory:
      swpd 交换内存的使用量
      free 空闲内存的总量
      buff 用于buffer的内存总量
      cache 用于cache的内存总量
  swap
      si 从磁盘交换到内存的数据速率(kb/s)
      so 从内存到磁盘的数据的速率(kb/s)
  io
      bi: 从块设备读入数据到系统的速率
      bo: 从系统到块设备
  cpu
      us 用户空间
      sy 系统空间
      id 空闲
      wa 等待
      st 被偷走的时间
  ```

- ##### dstat命令：

  系统资源统计，代替vmstat，iostat

  ```
  -c cpu信息
  -d 显示disk信息
  -g 显示page相关统计信息
  -m 显示内存信息
  -n 显示网卡信息
  -p 显示进程相关信息
  -r 显示io请求相关信息
  -s swap相关信息
  --top-cpu 显示最占用cpu的进程
  --top-io 显示最占用io的进程
  --top-mem 显示最占用内存的进程
  --tcp  显示tcp
  --udp 显示udp
  ```

  

#### linux的作业控制

- 前台作业：通过终端启动，且启动后一直占据终端
- 后台任务：可通过终端启动，但启动后会进入后台运行（释放终端）

让作业运行于后台

- 运行中的作业 ctrl+z
- 尚未启动的作业 command &

后台作业虽然被送往后台运行，但其依然与终端相关，退出终端，将关闭后台作业，如果系统送往后台的任务，剥离与终端的关系

- nohup command &> /dev/null &
- srceen;command





#### 安全

##### systemctl

- systemctl   参数   服务

  ```shell
  [root@MyHost ~]# systemctl start sshd
  [root@MyHost ~]# systemctl start sshd nignx   # 可同时启动多个
  [root@MyHost ~]# systemctl list-unit-files |grep enabled  #查询那些服务是开机自启动的
  ```

  - 参数

    ```shell
    stop #停止
    restart  #重启
    reload #重新加载配置文件
    status #查看服务的状态
    enable #开机自启动
    disable #关闭开机自启动
    ```

    

##### iptables

- firewall

  ```shell
  [root@MyHost ~]# systemctl stop firewall  #关闭防火墙
  
  [root@MyHost ~]# systemctl status firewalld   #查看状态
  ● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:firewalld(1)
  
  ```

- iptables -F 清空防火墙规则

  

##### selinux

是美国国家安全局(NSA=TheNational Security Agency)和SCC(Secure Computing Corporation)开发的Linux的一个强制访问控制的安全模块。

- 常见命令

  ```shell
  	永久关闭
  1.修改配置文件，永久生效关闭selinux
  cp /etc/selinux/config /etc/selinux/config.bak #修改前备份
  2.修改方式可以vim编辑,找到
  [root@MyHost ~]# vim /etc/selinux/config     # 进入修改
  # This file controls the state of SELinux on the system.
  # SELINUX= can take one of these three values:
  #     enforcing - SELinux security policy is enforced.
  #     permissive - SELinux prints warnings instead of enforcing.
  #     disabled - No SELinux policy is loaded.
  SELINUX=disabled
  3.用sed替换
  sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
  4.检查状态
  grep "SELINUX=disabled" /etc/selinux/config
  #出现结果即表示修改成功
  
  
  getenforce #获取selinux状态
  setenforce 0   #临时关闭
  #修改selinux状态
  setenforce 
  usage:  setenforce [ Enforcing | Permissive | 1 | 0 ]
  数字0 表示permissive，给出警告，不会阻止，等同disabled
  数字1表示enforcing，表示开启
  ```

  



#### 虚拟环境

在开发Python应用程序的时候，系统安装的Python3只有一个版本：3.4。所有第三方的包都会被`pip`安装到Python3的`site-packages`目录下。如果我们要同时开发多个应用程序，那这些应用程序都会共用一个Python，就是安装在系统的Python 3。如果应用A需要jinja 2.7，而应用B需要jinja 2.6怎么办？这种情况下，每个应用可能需要各自拥有一套“独立”的Python运行环境。virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境。

##### 使用python2需要virtualenv创建虚拟环境（py3可用)

首先，我们用`pip`安装virtualenv：

```
$ pip3 install virtualenv
$ pip install virtualenv -i https://pypi.douban.com/simple  #可使用指定网址
```

然后，假定我们要开发一个新的项目，需要一套独立的Python运行环境，可以这么做：

第一步，创建一个独立的Python运行环境，命名为`venv`：

```shell
virtualenv --no-site-packages --python=python3 venv   #--python用于指定python的版本
```

命令`virtualenv`就可以创建一个独立的Python运行环境，我们还加上了参数`--no-site-packages`，这样，已经安装到系统Python环境中的所有第三方包都不会复制过来，这样，我们就得到了一个不带任何第三方包的“干净”的Python运行环境。

新建的Python环境被放到当前目录下的`venv`目录。有了`venv`这个Python环境，可以用`source`进入该环境：

```
source venv/bin/activate
(venv)Mac:myproject$
```

注意到命令提示符变了，有个`(venv)`前缀，表示当前环境是一个名为`venv`的Python环境。

下面正常安装各种第三方包，并运行`python`命令：

```
pip install django==1.11.11
```

在`venv`环境下，用`pip`安装的包都被安装到`venv`这个环境下，系统Python环境不受任何影响。也就是说，`venv`环境是专门针对`myproject`这个应用创建的。

退出当前的`venv`环境，使用`deactivate`命令：

```
deactivate
```

此时就回到了正常的环境，现在`pip`或`python`均是在系统Python环境下执行。

完全可以针对每个应用创建独立的Python运行环境，这样就可以对每个应用的Python环境进行隔离。

virtualenv是如何创建“独立”的Python运行环境的呢？原理很简单，就是把系统Python复制一份到virtualenv的环境，用命令`source venv/bin/activate`进入一个virtualenv环境时，virtualenv会修改相关环境变量，让命令`python`和`pip`均指向当前的virtualenv环境。



##### 使用python3的venv命令来创建虚拟环境

```shell
[root@MyHost ~]# python3 -m venv django1
[root@MyHost ~]# ll
drwxr-xr-x 5 root root 4096 Aug 11 16:51 django1
[root@MyHost ~]# cd django1/
[root@MyHost django1]# source bin/activate
(django1) [root@MyHost django1]# 
(django1) [root@MyHost django1]# deactivate
```



#### 二 确保环境一致

```SHELL
 #当执行项目迁移时，可以将window上执行打包三方包
1.通过命令保证环境的一致性，导出当前python环境的包
pip3 freeze > requirements.txt    #进入到运行的环境中，执行pip list可查看当前的依赖包
这将会创建一个 requirements.txt 文件，其中包含了当前环境中所有包及 各自的版本的简单列表。
可以使用 “pip list”在不产生requirements文件的情况下， 查看已安装包的列表。

2.上传至服务器后，在服务器下创建virtualenv，在venv中导入项目所需的模块依赖
pip3 install -r requirements.txt
```

#### 三 virtualenvwrapper

**virtualenv 的一个最大的缺点就是：**每次开启虚拟环境之前要去虚拟环境所在目录下的 bin 目录下 source 一下 activate，这就需要我们记住每个虚拟环境所在的目录。

**并且还有可能你忘记了虚拟环境放在哪。。。**

- 一种可行的解决方案是，将所有的虚拟环境目录全都集中起来，例如/opt/all_venv/，并且针对不同的目录做不同的事。
- 使用virtualenvwrapper管理你的虚拟环境（virtualenv），其实他就是统一管理虚拟环境的目录，并且省去了source的步骤。

#### 步骤1：安装virtualenvwrapper

```
pip3 install virtualenvwrapper -i https://pypi.douban.com/simple 
```

#### 步骤2：设置Linux的环境变量，每次启动就加载virtualenvwrapper

```shell
把下面两行代码添加到 ~/.bashrc文件中
打开文件
vim ~/.bashrc
写入以下两行代码
export WORKON_HOME=~/Envs   #设置virtualenv的统一管理目录
export VIRTUALENVWRAPPER_VIRTUALENV_ARGS='--no-site-packages'   #添加virtualenvwrapper的参数，生成干净隔绝的环境
export VIRTUALENVWRAPPER_PYTHON=/opt/python347/bin/python3     #指定python解释器**
source /opt/python34/bin/virtualenvwrapper.sh #执行virtualenvwrapper安装脚本，先find / -name virtuale..sh的位置，读取文件，使得生效，此时已经可以使用virtalenvwrapper

[root@MyHost django1]#source ~/.bashrc
```

#### 步骤3：基本使用virtualenvwrapper

```shell
创建一个虚拟环境：
$ mkvirtualenv my_django115  #这会在 ~/Envs 中创建 my_django115 文件夹。

在虚拟环境上工作：激活虚拟环境my_django115
$ workon my_django115


再创建一个新的虚拟环境
$ mkvirtualenv my_django2

virtualenvwrapper 提供环境名字的tab补全功能。
当有很多环境， 并且很难记住它们的名字时，这就显得很有用。
workon还可以任意停止你当前的环境，可以在多个虚拟环境中来回切换
workon django1.15

workon django2.0

也可以手动停止虚拟环境
deactivate

删除虚拟环境，需要先退出虚拟环境
rmvirtualenv my_django115
```

#### 步骤四：常用其他命令

```shell
lsvirtualenv #列举所有的环境。
cdvirtualenv #导航到当前激活的虚拟环境的目录中，比如说这样您就能够浏览它的 site-packages 。
cdsitepackages #和上面的类似，但是是直接进入到 site-packages 目录中。

lssitepackages
显示 site-packages 目录中的内容。
完整官网介绍：https://virtualenvwrapper.readthedocs.io/en/latest/command_ref.html
```