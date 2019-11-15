## Linux 安装Python3



#### 问题描述：

- 在Linux下安装Python时出现一个错误：zipimport.ZipImportError: can't decompress data; zlib not available...

  ```
  zipimport.ZipImportError: can't decompress data; zlib not available
  Makefile:1079: recipe for target 'install' failed
  ```

#### 问题解决：

- 这是因为缺少依赖造成的，在安装python之前需要先安装python的依赖环境

  **Ubuntu/Debian下需安装的依赖：**

  ```shell
  sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python-openssl
  ```

  **Fedora/CentOS/RHEL(aws ec2)下需安装的依赖：**

  ```shell
  sudo yum install zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel xz xz-devel libffi-devel
  ```



#### 可以直接使用yum安装

- ##### 通过yum安装python3

  ```
  1、搜索出yum可以安装的python版本
  [root@MyHost ~]# yum search python3
  2、选择想要下载的版本，如：python36-sip.x86_64 : SIP - Python 3/C++ Bindings Generator
  3、执行安装
  [root@MyHost ~]# yum install python36-sip.x86_64 : SIP - Python 3/C++ Bindings Generator
  ```

  不推荐使用这种方式安装，因为没有试过！哈哈哈



#### 第一步

- ##### 下载安装包：去官网找到下载网址

  ```python
  [root@MyHost ~]# wget https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tgz
  ```

- 个人习惯安装在**/usr/local/python3**（具体安装位置看个人喜好），注意wget会将文件下载到当前位置，所以最好先创建文件，切换到文件下后，进行下载，解压：

  ```shell
  [root@MyHost python3]# tar -zxvf Python-3.7.2.tgz 
  ```

- 进入解压好的目录中，编译安装（编译安装前需要安装编译器 **sudo apt install gcc**）

  ```shell
  [root@MyHost python3]#./configure --prefix=/usr/local/python3   #/usr/local/python3为安装目录
  [root@MyHost python3]#./configure --help  #可查看帮助
  
   #执行完configure命令后，configure 命令执行完之后，会生成一个 Makefile 文件，这个 Makefile主要是被下一步的 make 命令所使用（ Linux 需要按照Makefile 所指定的顺序来构建 (build) 程序组件）。
  ```

- make 编译源代码，并生成可执行文件

  ```shell
  [root@MyHost python3]#make
  ```

- make install 实际上是把生成的执行文件拷贝到之前configure命令指定的目录/usr/local/python3下。

  到这里安装已经结束，下面是配置环境

  ```shell
  [root@MyHost python3]#make insatll
  ```

- 安装完成后会在指定目录生成一下文件

  ```shell
  drwxr-xr-x  2 root root     4096 Aug 10 22:28 bin   #可执行文件存放处
  drwxr-xr-x  3 root root     4096 Aug 10 22:28 include
  drwxr-xr-x  4 root root     4096 Aug 10 22:28 lib
  drwxr-xr-x  3 root root     4096 Aug 10 22:28 share
  ```



#### 配置环境

- ##### **建立python3的软链接**

  ```shell
  [root@MyHost python3]#ln -s /usr/local/python3/bin/python3 /usr/bin/python3  #将文件加的python3 拷贝到全局bin下，系统自动调用，或则添加环境变量
  ```

- ##### 将新创建的bin目录添加到环境变量中

  ```shell
  [root@MyHost python36]#vim /etc/profile.d/python.sh
   # 在里面写：export PATH=$PATH:/opt/python36/bin  保存退出
  ```

- ##### 创建pip3连接

  ```shell
  [root@MyHost python3]#ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
  ```




deepin 安装python

 https://blog.csdn.net/little_insect_0/article/details/82142435 