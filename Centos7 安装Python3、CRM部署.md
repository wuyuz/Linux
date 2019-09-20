## Centos7 安装Python3 以及虚拟环境



#### Python3在linux下的编译过程

- 首先解决环境依赖问题，如gcc编译工具等

- 其次保证yum源配置好，配置步骤

  ```
  1、打开阿里云开源镜像站的官网：https://opsx.alibaba.com/mirror
  2、获取到centos的yum源（点击帮助，找到以下命令，会帮助我们将指定文件下载到指定文件夹下，也就是说/etc/yum.repos/d目录，只要在这个目录下名字以repo结尾的都会自动读取，这个是配置的系统常用工具，但是有些软件的工具包是没有的所以需要第三方库epel）：
  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
  3、找到epel源，点击帮助：
  wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
  ```

  

#### Python3的正式安装

- 解决编译Python3的环境依赖

  ```shell
  yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y
  ```

- 下载Python3源码包

  ```shell
  [root@MyHost ~]#wget https://www.python.org/ftp/python/3.6.7/Python-3.6.7.tar.xz
  [root@MyHost ~]#xz -d Python-3.6.7.tar.xz   #解压xz层
  [root@MyHost ~]#tar -xf Python-3.6.7.tar  #解压
  ```

- 解压缩源代码包，进入，开始编译三部曲

  ```shell
  1、第一曲 执行configure脚本文件，指定安装路径，释放makefile编译文件（之前是没有的，如果失败了就把它给删除，重新执行第一步），让gcc去编译
  [root@MyHost ~]#yum install gcc -y
  [root@MyHost ~]#./configure --prefix=/opt/python
  2、第二曲，执行make指令，读取makefile，开始编译
  [root@MyHost ~]#make && make install
  3、第三曲，执行make install ，开始安装Python3，这一步会生成Python3的解释器
  
  ------------------------------------------------------------------
  用python查看默认安装路径
  >>> import sys
  >>> sys.path
  ['', '/usr/local/lib/python36.zip', '/usr/local/lib/python3.6', 
  '/usr/local/lib/python3.6/lib-dynload', '/usr/local/lib/python3.6/site-packages']
  
  --------------------
  python3.4默认没添加path
  在/etc/profile最后一行添加
  export PATH=$PATH:/opt/python/bin
  然后
  source /etc/profile
  
  ```

- 编译完成之后，配置path环境变量，让系统可以补全python3的命令

  ```shell
  1、获取当前环境变量：
  [root@MyHost ~]# echo $PATH
  /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
  
  2、添加Python3的环境变量，注意，要添加到开头，因为加载的时候是从前往后加载
  [root@MyHost ~]# vim /etc/profile   #在最后添加一句：PATH='/opt/s21/python367/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin' 注意这里的/opt/s21/python367/bin是你具体安装python的位置，且与其他路径用:分隔
  
  [root@MyHost ~]# source /etc/profile #立即生效
  
  PATH='/usr/local/python3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin'
  ```

  

#### 虚拟环境搭建

​	在使用 Python 开发的过程中，工程一多，难免会碰到不同的工程依赖不同版本的库的问题；亦或者是在开发过程中不想让物理环境里充斥各种各样的库，引发未来的依赖灾难。此时，我们需要对于不同的工程使用不同的虚拟环境来保持开发环境以及宿主环境的清洁。这里，就要隆重介绍 virtualenv，一个可以帮助我们管理不同 Python 环境的绝好工具。

**virtualenv 可以在系统中建立多个不同并且相互不干扰的虚拟环境。**



#### Linux下安装、配置virtualenv

- 指定清华源下载pip包，注意如果没有pip3可能是因为环境变量顺序问题

  ```shell
  [root@MyHost ~]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple virtualenv
  
  [root@MyHost ~]# pip3 install --upgrade pip
  
  安装virtualenv
  [root@MyHost ~]# pip3 install virtualenv 
  ```

- 通过命令创建虚拟环境

  ```PYTHON
  创建目录
  [root@MyHost ~]# mkdir Myproject
  [root@MyHost Myproject]# cd Myproject
  
  创建独立运行环境-命名
  [root@MyHost Myproject]#virtualenv --no-site-packages --python=python3 env  #得到独立第三方包的环境，env是生成的目录，前一个参数是创建干净的环境没有第三方包，第二个是指定解释器
  
  进入虚拟环境
  [root@MyHost Myproject]#source venv/bin/activate  #此时进入虚拟环境(venv)Myproject
  
  安装第三方包
  (venv)[root@MyHost Myproject]#: pip3 install django==1.9.8
  #此时pip的包都会安装到venv环境下，venv是针对Myproject创建的
  
  退出venv环境
  [root@MyHost Myproject]#deactivate命令
  
  virtualenv是如何创建“独立”的Python运行环境的呢？原理很简单，就是把系统Python复制一份到virtualenv的环境，用命令source venv/bin/activate进入一个virtualenv环境时，virtualenv会修改相关环境变量，让命令python和pip均指向当前的virtualenv环境。（可以在激活虚拟环境的情况下查看PATH）
  ```

  

#### Linux搭建CRM项目

- 首先我们需要将我们的项目压缩，使用lrzsz可以实现window与linux之间的拖动文件

  ```shell
  [root@MyHost ~]# yum install lrzsz -y
  
   #从服务端发送文件到客户端：sz filename
   #从客户端上传文件到服务端：rz
   
  [root@MyHost ~]# yum install unzip -y 
  [root@MyHost ~]# unzip dev_CRM.zip    #解压文件
  ```

- 搭建虚拟环境

  ```python
  [root@MyHost Myproject]#virtualenv --no-site-packages --python=python3 env #创建env虚拟环境
  [root@MyHost ~]# source env/bin/activate    #启动虚拟环境
  (env) [root@MyHost ~]# 
  ```

- 在虚拟环境下启动Django项目

  ```shell
  (env) [root@MyHost dev_CRM]# cd dev_CRM   #进入项目
  (env) [root@MyHost dev_CRM]# python3 manage.py runserver   #报错，没有Django框架
  
   #安装django
  (env) [root@MyHost dev_CRM]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple django==1.11.11 
  
   #再次运行发现没有没有pymysql模块
  (env) [root@MyHost dev_CRM]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pymysql
   
   #在运行发现没有multiselectfield模块，继续安装
  (env) [root@MyHost dev_CRM]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple django-multiselectfield
  
   #再次运行发现没有Mysql，最新版本mysql
  (env) [root@MyHost dev_CRM]# yum install mariadb-server mariadb -y
   #启动mysql，通过yum安装的软件都可以通过systemctl启动
  (env) [root@MyHost dev_CRM]# systemctl start mariadb
   #查看mysql是否启动
  (env) [root@MyHost dev_CRM]# netstat -tunlp |grep 3306   #查看端口
   tcp   0   0  0.0.0.0:3306  0.0.0.0:*     LISTEN     5680/mysqld      
  (env) [root@MyHost dev_CRM]# ps -ef|grep mariadb  #查看进程
  
  
   #修改setting.py文件
  ALLOWED_HOSTS = ['*']
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'crm',
          'HOST': '127.0.0.1',
          'PORT': 3306,
          'USER': 'root',
          'PASSWORD': '',
      }
  }
  
  
   #修改数据库
  (env) [root@MyHost dev_CRM]# mysql -uroot -p
  MariaDB [(none)]> create database crm charset utf8;
  Query OK, 1 row affected (0.00 sec)
  
   #生成迁移文件、迁移
  (env) [root@MyHost dev_CRM]# python3 manage.py makemigrations
  (env) [root@MyHost dev_CRM]# python3 manage.py  migrate
  
   #启动，但是如果时云服务器上的话
  (env) [root@MyHost dev_CRM]# python3 manage.py runserver 0.0.0.0:8000
  Performing system checks...
  
  System check identified no issues (0 silenced).
  September 17, 2019 - 10:48:40
  Django version 1.11.11, using settings 'crm.settings'
  Starting development server at http://0.0.0.0:8000/
  Quit the server with CONTROL-C.
  
   #清理防火墙
  (env) [root@MyHost dev_CRM]# iptables -F
  (env) [root@MyHost dev_CRM]# python3 manage.py runserver 0.0.0.0:80  #注意只有80阿里云才通过
  
  ```

  - 注意：部署MySQL的时候同样遇到端口问题，追查下去找到了原因：虽然CentOs里防火墙关了，但是阿里云安全组里面有防火墙，而这个防火墙是没开我上面提到的那些端口的，而默认开了80端口，所以80端口可行，而其他的几个都不能用，如图：

    ![1568718099396](C:\Users\wanglixing\Desktop\知识点复习\Linux\assets\1568718099396.png)

  - 之前我们安装依赖包的时候，是一个一个的去试出来的，下面我们将回顾之前讲过的requirement.txt的方法

    ```python
    在原有环境下执行： pip3 freeze > requirement.txt   #可以生成一个txt文件
    将该文件拖入到新环境下，执行：pip3 install -i https://pypi.douban.com/simple -r requirement.txt  #将自动安装所有到处的模块
    ```

    

