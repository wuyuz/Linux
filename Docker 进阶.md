## Docker 进阶



#### 什么是Docker？

```
Docker 最初是 dotCloud 公司创始人 Solomon Hykes 在法国期间发起的一个公司内部项目，于 2013 年 3 月以 Apache 2.0 授权协议开源，主要项目代码在 GitHub 上进行维护。
Docker 使用 Google 公司推出的 Go 语言 进行开发实现。
docker是linux容器的一种封装，提供简单易用的容器使用接口。它是最流行的Linux容器解决方案。
docker的接口相当简单，用户可以方便的创建、销毁容器。
docker将应用程序与程序的依赖，打包在一个文件里面。运行这个文件就会生成一个虚拟容器。
程序运行在虚拟容器里，如同在真实物理机上运行一样，有了docker，就不用担心环境问题了。
```



#### Docker使用场景

```
web应用的自动化打包和发布
自动化测试和持续集成、发布
在服务型环境中部署和调整数据库或其他应用
```



#### 为什么要用Docker？

我们先看看很久很久以前，服务器是怎么部署应用的！

![1570531948692](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570531948692.png)

由于物理机的诸多问题，后来出现了虚拟机

![1570531963651](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570531963651.png)

```
但是虚拟化也是有局限性的，每一个虚拟机都是一个完整的操作系统，要分配系统资源，虚拟机多道一定程度时，操作系统本身资源也就消耗殆尽，或者说必须扩容
```



#### docker与虚拟机的区别

![1570532374175](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570532374175.png)

#### 环境配置难题

让开发人员最头疼的麻烦事之一就是环境配置了，每台计算机的环境都不相同，应该如何确保自己的程序换一台机器能运行起来呢？用户必须确保的是：

1. 操作系统的相同
2. 各种平台库和组件的安装
3. 例如python依赖包，环境变量等

如何一些低版本的依赖模块和当前环境不兼容，那就头疼了..., .。如果环境配置这么痛苦的话，换一台机器，就得重新配置一下，那么在安装软件的时候，带着原始环境一模一样的复制过来。



- 方案一：虚拟机模板

  ​          虚拟机也可以制作模板，基于模板创建虚拟机，保证环境问题一致

  虚拟机（virtual machine）就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。

  虽然用户可以通过虚拟机还原软件的原始环境。但是，这个方案有几个缺点。

  （1）资源占用多

  ​        虚拟机会独占一部分内存和硬盘空间。它运行的时候，其他程序就不能使用这些资源了。哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。

  （2）冗余步骤多

  ​        虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。

  （3）启动慢

  ​        启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行。

- 方案二：Linux容器

  ​         **现在:自从用上docker容器后，可以实现开发、测试和生产环境的统一化和标准化。**

  **镜像作为标准的交付件，可在开发、测试和生产环境上以容器来运行，最终实现三套环境上的应用以及运行所依赖内容的完全一致。**

   由于虚拟机的诸多问题，Linux发展出了另一种虚拟化技术：Linux容器（Linux Containers，缩写LXC）

  **Linux容器不是模拟一个完整的操作系统，而是对进程进行隔离。在正常进程的外面套了一个保护层，对于容器里面进程来说，它接触的资源都是虚拟的，从而实现和底层系统的隔离。**

  （1）启动快

  ​        容器里面的应用，直接就是底层系统的一个进程，而不是虚拟机内部的进程。所以，启动容器相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多。

  （2）资源占用少

  ​        容器只占用需要的资源，不占用那些没有用到的 资源；虚拟机由于是完整的操作系统，不可避免要占用所有资源。另外，多个容器可以共享资源，虚拟机都是独享资源。

  （3）体积小

  ​        容器只要包含用到的组件即可，而虚拟机是整个操作系统的打包，所以容器文件比虚拟机文件要小很多。

   总之，容器有点像轻量级的虚拟机，能够提供虚拟化的环境，但是成本开销小得多。



#### Docker的三大生命周期概念

```
容器三大基本概念
    镜像 image      可以理解为操作系统的镜像文件 如centos.iso
    容器 container  容器实例，基于Docker镜像运行出的容器实例，可以理解为微型操作系统
    仓库 repositorydocker整个生命周期就是这三个概念。   存储镜像的文件的地方
```

- Docker镜像

  ```
  Docker镜像就是一个只读的模板。
  例如：一个镜像可以包含一个完整的CentOS操作系统环境，里面仅安装了Apache或用户需要的其他应用程序。
  镜像可以用来创建Docker容器。
  
  Docker提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。
  ```

- image的分层存储

  ```
  因为镜像包含完整的root文件系统，体积是非常庞大的，因此docker在设计时按照Union FS的技术，将其设计为分层存储的架构。镜像不是ISO那种完整的打包文件，镜像只是一个虚拟的概念，他不是一个完整的文件，而是由一组文件组成，或者多组文件系统联合组成。
  ```

- Docker容器(container)

  ```
  image和container的关系，就像面向对象程序设计中的 类和实例一样，镜像是静态的定义（class），容器是镜像运行时的实体（object）。
  容器可以被创建、启动、停止、删除、暂停
  Docker利用容器来运行应用。
  
  容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的，保证安全的平台。
  
  可以把容器看做是一个简易版的Linux环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。
  
  注意：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。
  ```

- Docker仓库(repository)

  ```
  仓库是集中存放镜像文件的场所。有时候把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag)。
  
  仓库分为公开仓库(Public)和私有仓库(Private)两种形式。
  
  最大的公开仓库是Docker Hub，存放了数量庞大的镜像供用户下载。国内的公开仓库包括Docker Pool等，可以提供大陆用户更稳定快读的访问。
  
  当用户创建了自己的镜像之后就可以使用push命令将它上传到公有或者私有仓库，这样下载在另外一台机器上使用这个镜像时候，只需需要从仓库上pull下来就可以了。
  
  注意：Docker仓库的概念跟Git类似，注册服务器可以理解为GitHub这样的托管服务。
  ```



#### Docker镜像加速器

```shell
https://www.daocloud.io/mirror#accelerator-dochttps://www.cnblogs.com/pyyu/p/6925606.html

一条命令加速：执行即可
[root@docker ~]#curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://95822026.m.daocloud.io

[root@docker ~]#systemctl restart docker
```



#### yum安装Docker

- 安装

  ```shell
  1.卸载旧版本f,复制粘贴
  sudo yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-selinux \
                    docker-engine-selinux \
                    docker-engine
  
  2.设置存储库
  sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
  
  # 配置yum源，在/etc/yum.repos.d/
  sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
  
  3.安装docker社区版
  	sudo yum install docker-ce
  4.启动关闭docker
  	systemctl start docker
  ```

- 查找linux的yum仓库

  ```
  /etc/yum.repos.d/
  ```

- docker版本

  ```
  Docker 是一个开源的商业产品，有两个版本：社区版（Community Edition，缩写为 CE）和企业版（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到。本文的介绍都针对社区版。
  ```

   系统环境准备

  ```
  docker最低支持centos7且在64位平台上，内核版本在3.10以上
  ```

  [root@oldboy_python ~ 10:48:11]#uname -r
  3.10.0-693.el7.x86_64

- 基本命令解释

  ```
  [root@docker ~]# docker --help
  
  Usage:
  docker [OPTIONS] COMMAND [arg...]
  
         docker daemon [ --help | ... ]
  
         docker [ --help | -v | --version ]
  
  self-sufficient runtime for containers.
  
   
  Options:
  
    --config=~/.docker              Location of client config files  #客户端配置文件的位置
  
    -D, --debug=false               Enable debug mode  #启用Debug调试模式
  
    -H, --host=[]                   Daemon socket(s) to connect to  #守护进程的套接字（Socket）连接
  
    -h, --help=false                Print usage  #打印使用
  
    -l, --log-level=info            Set the logging level  #设置日志级别
  
    --tls=false                     Use TLS; implied by--tlsverify  #
  
    --tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA  #信任证书签名CA
  
    --tlscert=~/.docker/cert.pem    Path to TLS certificate file  #TLS证书文件路径
  
    --tlskey=~/.docker/key.pem      Path to TLS key file  #TLS密钥文件路径
  
    --tlsverify=false               Use TLS and verify the remote  #使用TLS验证远程
  
    -v, --version=false             Print version information and quit  #打印版本信息并退出
  
  
  Commands:
  
      attach    Attach to a running container  #当前shell下attach连接指定运行镜像
  
      build     Build an image from a Dockerfile  #通过Dockerfile定制镜像
  
      commit    Create a new image from a container's changes  #提交当前容器为新的镜像
  
      cp    Copy files/folders from a container to a HOSTDIR or to STDOUT  #从容器中拷贝指定文件或者目录到宿主机中
  
      create    Create a new container  #创建一个新的容器，同run 但不启动容器
  
      diff    Inspect changes on a container's filesystem  #查看docker容器变化
  
      events    Get real time events from the server#从docker服务获取容器实时事件
  
      exec    Run a command in a running container#在已存在的容器上运行命令
  
      export    Export a container's filesystem as a tar archive  #导出容器的内容流作为一个tar归档文件(对应import)
  
      history    Show the history of an image  #展示一个镜像形成历史
  
      images    List images  #列出系统当前镜像
  
      import    Import the contents from a tarball to create a filesystem image  #从tar包中的内容创建一个新的文件系统映像(对应export)
  
      info    Display system-wide information  #显示系统相关信息
  
      inspect    Return low-level information on a container or image  #查看容器详细信息
  
      kill    Kill a running container  #kill指定docker容器
  
      load    Load an image from a tar archive or STDIN  #从一个tar包中加载一个镜像(对应save)
  
      login    Register or log in to a Docker registry#注册或者登陆一个docker源服务器
  
      logout    Log out from a Docker registry  #从当前Docker registry退出
  
      logs    Fetch the logs of a container  #输出当前容器日志信息
  
      pause    Pause all processes within a container#暂停容器
  
      port    List port mappings or a specific mapping for the CONTAINER  #查看映射端口对应的容器内部源端口
  
      ps    List containers  #列出容器列表
  
      pull    Pull an image or a repository from a registry  #从docker镜像源服务器拉取指定镜像或者库镜像
  
      push    Push an image or a repository to a registry  #推送指定镜像或者库镜像至docker源服务器
  
      rename    Rename a container  #重命名容器
  
      restart    Restart a running container  #重启运行的容器
  
      rm    Remove one or more containers  #移除一个或者多个容器
  
      rmi    Remove one or more images  #移除一个或多个镜像(无容器使用该镜像才可以删除，否则需要删除相关容器才可以继续或者-f强制删除)
  
      run    Run a command in a new container  #创建一个新的容器并运行一个命令
  
      save    Save an image(s) to a tar archive#保存一个镜像为一个tar包(对应load)
  
      search    Search the Docker Hub for images  #在docker
  hub中搜索镜像
  
      start    Start one or more stopped containers#启动容器
  
      stats    Display a live stream of container(s) resource usage statistics  #统计容器使用资源
  
      stop    Stop a running container  #停止容器
  
      tag         Tag an image into a repository  #给源中镜像打标签
  
      top       Display the running processes of a container #查看容器中运行的进程信息
  
      unpause    Unpause all processes within a container  #取消暂停容器
  
      version    Show the Docker version information#查看容器版本号
  
      wait         Block until a container stops, then print its exit code  #截取容器停止时的退出状态值
  
  Run 'docker COMMAND --help' for more information on a command.  #运行docker命令在帮助可以获取更多信息
  ```



#### 安装与使用

- 配置阿里云仓库即可安装

  ```shell
  1.卸载旧版本
  sudo yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-selinux \
                    docker-engine-selinux \
                    docker-engine
  
  2.设置存储库
  sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
  
  sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo   #找到aliyun代码平台
  
  3.安装docker社区版
  sudo yum install docker-ce
  4.启动关闭docker
  systemctl start docker
  ```

- 配置aliyun仓库即可安装

  ```
  yum install docker -y
  ```

- 同构yum安装的软件，都可以使用systemctl系统服务管理命令，去启停

  ```
  systemctl status/start/dtop docker
  ```

- 镜像、仓库、容器的增删改查管理命令

  - 使用docker镜像

    ```
    1、从仓库获取镜像
    2、管理本地主机的镜像
    
    从docker registry获取镜像的命令是docker pull。命令格式是：
        docker pull [选项][docker registry地址] 仓库名:标签
        docker register地址：地址的格式一般是 域名:端口，默认地址是docker hub
        仓库名：仓库名是两段格式，用户名/软件名，如果不写用户，默认docker hub用户名是library，也就是官方镜像
        
    docker是把应用程序和其依赖打包在image文件里面，只有通过这个镜像文件才能生成docker容器。
    一个image文件可以生成多个容器实例。image文件是通用，可以共享的
    
    列出服务器所有镜像文件:
        #列出所有的image文件
        docker image ls
        #删除image文件
        docker image rm [imagename]
    ```

  - 搜索并安装Docker镜像

    ```shell
    [root@docker ~]# docker search centos   # 搜索所有centos的docker镜像，带有OK的表示官方的
    [root@docker ~]# docker pull 镜像文件    # 添加镜像文件
    [root@docker ~]# docker run 镜像文件     # 如果不创建就会自动下载
    [root@docker ~]# dokcer ps              # 列出当前记录正在运行的容器进程
    [root@docker ~]# dokcer ps -a            # 列出当前记录正在运行的容器进程，以及挂掉的
    
    #运行一个活着的容器进程
    [root@docker ~]#docker run -d centos /bin/sh -c "while true;do echo hello centos;sleep 1;done"  #-d 后台运行  centos 指的是镜像文件  /bin/sh/要在这个容器内运行的命令 -c 表示一段shell代码
    
    删除
    [root@docker ~]#docker rmi 镜像名字/镜像id   #删除镜像文件，如果有容器正在引用，就删不了，需要删除正在使用的容器 
    [root@docker ~]#docker rm 容器id/容器进程名字  #删除容器
    [root@docker ~]#docker rm `docker ps -aq`  #批量删除容器id
    [root@docker ~]#docker rmi -f 镜像名字/镜像id    #强制删除镜像即相应的容器
    
    查看
    [root@docker ~]#docker log 容器id  # 查看容器日志
    [root@docker ~]#docker log -f 容器id  # 实时查看容器日志
    [root@docker ~]#docker exit -it 容器 /bin/bash  #进入容器空间内 -i 交互式 -t 开启新的终端
    [root@docker ~]#docker run -it ubuntu /bin/bash  # 安装并启动ubuntu，交互式-it
    
    退出容器：exit
    ```

    注意点：

    ```
    1、docker容器随时创建，随时删除，每次docker run都会生成新的容器记录
    2、docker容器进程，如果没有在后台运行的话，就会立即挂掉，（容器中必须有正在工作的进程）
    ```

- 新建容器并启动

  ```
  所需要的命令主要为docker run，例如,下面的命令输出一个hehe,之后终止容器。
  
  [root@docker ~]# docker run centos /bin/echo "hehe"  #这跟在本地直接执行 /bin/echo'hehe' 
  hehe
  
  [root@docker ~]# docker run --name mydocker -it centos /bin/bash#启动一个bash终端,允许用户进行交互。
  
  [root@1c6c3f38ea07 /]# pwd
  /
  
  --name:给容器定义一个名称
  -i:则让容器的标准输入保持打开。
  -t:让Docker分配一个伪终端,并绑定到容器的标准输入上
  /bin/bash:执行一个命令
  ```

  当利用docker run来创建容器时，Docker在后台运行的标准操作包括

  ```
  检查本地是否存在指定的镜像，不存在就从公有仓库下载，利用镜像创建并启动一个容器，分配一个文件系统，并在只读的镜像层外面挂在一层可读写层，从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去，从地址池配置一个ip地址给容器，执行用户指定的应用程序，执行完毕后容器被终止
  ```

-  运行一个ubuntu容器 

  ```shell
  [root@oldboy_python ~ 11:52:22]#docker pull ubuntu:14.04
  #如图，乌班图的镜像下载，是下载每一层的文件
  Trying to pull repository docker.io/library/ubuntu ... 
  14.04: Pulling from docker.io/library/ubuntu
  8284e13a281d: Pull complete 
  26e1916a9297: Pull complete 
  4102fc66d4ab: Pull complete 
  1cf2b01777b2: Pull complete 
  7f7a2d5e04ed: Pull complete 
  Digest: sha256:4851d1986c90c60f3b19009824c417c4a0426e9cf38ecfeb28598457cefe3f56
  Status: Downloaded newer image for docker.io/ubuntu:14.04
  
  [root@oldboy_python ~ 12:18:53]#docker run -it --rm ubuntu:14.04 bash
  #此时会进入交互式的shell界面，即可以使用乌班图操作系统
  root@3efbb2749d7c:/# cat /etc/os-release   #查看版本
  NAME="Ubuntu"
  VERSION="14.04.5 LTS, Trusty Tahr"
  ID=ubuntu
  ID_LIKE=debian
  PRETTY_NAME="Ubuntu 14.04.5 LTS"
  VERSION_ID="14.04"
  HOME_URL="http://www.ubuntu.com/"
  SUPPORT_URL="http://help.ubuntu.com/"
  BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
  
  #使用exit退出容器
  exit
  
  docker run就是运行容器的命令。
  参数
   -it ： -i 是交互式操作，-t是终端
   -rm  :   容器退出后将其删除。也可以不指定参数，手动docker rm，使用-rm可以避免浪费空间。
   ubuntu:14.04   这指的是镜像文件
   bash   :  指定用交互式的shell，因此需要bash命令
  ```

- 后台模式启动docker

  ```
  -d参数：后台运行容器，返回容器ID
  [root@oldboy_python ~ 15:58:14]#docker run -d centos /bin/sh -c "while true;do echo hello 
  
  #检查容器进程
  [root@oldboy_python ~ 15:58:22]#docker ps
  ```

  查看容器内的标准输出

  ```
  docker logs c02
  ```

  停止容器

  ```
  docker stop c02#此时容器进程不存在docker ps 
  ```

  启动容器

  ```
  docker start c02#检查容器进程docker ps
  删除容器
  docker rm c02
  ```

  Docker镜像常用命令

  ```
  docker images #列出所有本级镜像
  docker pull centos #获取新的centos镜像
  docker search nginx #搜索nginx镜像
  ```

  构建镜像

  ```
  1.通过commit修改镜像
  2.编写dockerfile
  ```

  进入容器：使用-d参数时，容器启动后会进入后台。某些时候需要进入容器进行操作,有很多种方法，包括使用docker attach命令或nsenter工具等。

  ```
  docker exec -it 容器id
  docker attach 容器id
  ```

  

#### 提交创建自定义的镜像(docker container commit)

```shell
1.我们进入交互式的centos容器中，发现没有vim命令
	docker run -it centos /bin/bash
	
2.在当前容器中，安装一个vim
	yum install -y vim
	
3.安装好vim之后，exit退出容器，此时的容器以及有我们定义的vim软件了
	exit
	
4.查看刚才安装好vim的容器记录
	docker container ls -a
	
5.提交这个容器，创建新的image,执行之后，使用images 就可以看到新创建的镜像文件了，剩下的就是导出了
	docker commit 容器id  镜像文件（路径/名）
	docker commit 419  s21docker-centos-vim 
	
6.查看镜像文件
[root@master /home]docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
s21docker-centos-vim    latest              fd2685ae25fe        5 minutes ago       348MB

7、导出我们的vim镜像contos文件
[root@master /home]docker save s21docker-centos-vim  > /opt/s21-centos-vim.tar.gz

8、此时我们就可以传递压缩文件，然后再导入
[root@master /home] docker load < /opt/s21-centos-vim.tar.gz
[root@master /home] docker images # 就可以看到我们的镜像了
```



#### 容器内运行一个web程序，进行端口映射

容器中可以运行网络应用，但是要让外部也可以访问这些应用，可以通过-p或-P参数指定端口映射。

```
-P 参数会随机映射端口到容器开放的网络端口
[root@oldboy_python ~ 16:31:37]#docker run -d -P training/webapp python app.py
```

检查映射的端口

```
#宿主机ip:32768 映射容器的5000端口
[root@oldboy_python ~ 16:34:02]#docker ps -l
CONTAINER ID   IMAGE  COMMAND  CREATED   STATUS  PORTS          NAMES
cfd632821d7a   training/webapp  "python app.py"  21 seconds ago   Up 20 seconds       0.0.0.0:32768->5000/tcp   brave_fermi
```

查看容器日志信息

```
#不间断显示log
[root@oldboy_python ~ 16:31:37]#docker logs -f cfd
```

也可以通过-p参数指定映射端口

```
#指定服务器的9000端口，映射到容器内的5000端口
[root@oldboy_python ~ 16:46:13]#docker run -d -p 9000:5000 training/webapp python app.py
```

访问服务器的9000端口，(如果访问失败的话，检查自己的防火墙，以及云服务器的安全组)

![img](https://images2018.cnblogs.com/blog/1132884/201808/1132884-20180816164829039-1757764520.png)

- 案例演示

  - 下载一个flask的docker镜像

    ```
    docker pull trainning/webapp
    ```

    ![1570538669725](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570538669725.png)

  - 运行docker镜像

    ```shell
    docker -d -P  # -d 后台运行 -P 随机端口映射， 随机的宿主机的端口：容器内的端口（自动指定，由代码指定） -p 指定端口映射  宿主机的7777：8500(后面的是容器的端口)
    
    [root@oldboy_python ~ 16:31:37]#docker run -d -P training/webapp python app.py  #相当于创建一个容器空间并执行下面的命令
    [root@oldboy_python ~ 16:34:02]#docker ps -l  #查看端口映射，这样就可以访问了
    [root@oldboy_python ~ 16:46:13]#docker run -d -p 9000:5000 training/webapp python app.py
    ```

    

#### 利用dockerfile定制镜像

​		镜像是容器的基础，每次执行docker run的时候都会指定哪个镜像作为容器运行的基础。我们之前的例子都是使用来自docker hub的镜像，直接使用这些镜像只能满足一定的需求，当镜像无法满足我们的需求时，就得自定制这些镜像。

```
镜像的定制就是定制每一层所添加的配置、文件。如果可以吧每一层修改、安装、构建、操作的命令都写入到一个脚本，用脚本来构建、定制镜像，这个脚本就是dockerfile。Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令 构建一层，因此每一条指令的内容，就是描述该层应当如何构建。
```

- 参数介绍

  ```shell
  FROM scratch #制作base image 基础镜像，尽量使用官方的image作为base image
  FROM centos #使用base image
  FROM ubuntu:14.04 #带有tag的base image
  
  LABEL version=“1.0” #容器元信息，帮助信息，Metadata，类似于代码注释
  LABEL maintainer=“yc_uuu@163.com"
  
  #对于复杂的RUN命令，避免无用的分层，多条命令用反斜线换行，合成一条命令！
  RUN yum update && yum install -y vim \
      Python-dev #反斜线换行
  RUN /bin/bash -c "source $HOME/.bashrc;echo $HOME”
  
  WORKDIR /root #相当于linux的cd命令，改变目录，尽量使用绝对路径！！！不要用RUN cd
  WORKDIR /test #如果没有就自动创建
  WORKDIR demo #再进入demo文件夹
  RUN pwd     #打印结果应该是/test/demo
  
  ADD and COPY 
  ADD hello /  #把本地文件添加到镜像中，吧本地的hello可执行文件拷贝到镜像的/目录
  ADD test.tar.gz /  #添加到根目录并解压
  
  WORKDIR /root
  ADD hello test/  #进入/root/ 添加hello可执行命令到test目录下，也就是/root/test/hello 一个绝对路径
  COPY hello test/  #等同于上述ADD效果
  
  ADD与COPY
     - 优先使用COPY命令
      -ADD除了COPY功能还有解压功能
  添加远程文件/目录使用curl或wget
  
  ENV #环境变量，尽可能使用ENV增加可维护性
  ENV MYSQL_VERSION 5.6 #设置一个mysql常量
  RUN yum install -y mysql-server=“${MYSQL_VERSION}” 
  
  ------这里需要稍微理解一下了-------中级知识---先不讲
  
  VOLUME and EXPOSE 
  存储和网络
  
  RUN and CMD and ENTRYPOINT
  RUN：执行命令并创建新的Image Layer
  CMD：设置容器启动后默认执行的命令和参数
  ENTRYPOINT：设置容器启动时运行的命令
  
  Shell格式和Exec格式
  RUN yum install -y vim
  CMD echo ”hello docker”
  ENTRYPOINT echo “hello docker”
  
  Exec格式
  RUN [“apt-get”,”install”,”-y”,”vim”]
  CMD [“/bin/echo”,”hello docker”]
  ENTRYPOINT [“/bin/echo”,”hello docker”]
  
  
  通过shell格式去运行命令，会读取$name指令，而exec格式是仅仅的执行一个命令，而不是shell指令
  cat Dockerfile
      FROM centos
      ENV name Docker
      ENTRYPOINT [“/bin/echo”,”hello $name”]#这个仅仅是执行echo命令，读取不了shell变量
      ENTRYPOINT  [“/bin/bash”,”-c”,”echo hello $name"]
  
  CMD
  容器启动时默认执行的命令
  如果docker run指定了其他命令(docker run -it [image] /bin/bash )，CMD命令被忽略
  如果定义多个CMD，只有最后一个执行
  
  ENTRYPOINT
  让容器以应用程序或服务形式运行
  不会被忽略，一定会执行
  最佳实践：写一个shell脚本作为entrypoint
  COPY docker-entrypoint.sh /usr/local/bin
  ENTRYPOINT [“docker-entrypoint.sh]
  EXPOSE 27017
  CMD [“mongod”]
  
  [root@master home]# more Dockerfile
  FROm centos
  ENV name Docker
  #CMD ["/bin/bash","-c","echo hello $name"]
  ENTRYPOINT ["/bin/bash","-c","echo hello $name”]
  ```

  

- dockerfile构建一个flask web APP

  - 编写flask代码文件

    ```python
    确保app.py和dockerfile在同一个目录！准备好app.py的flask程序
    [root@master home]# cat app.py
    
    #coding:utf8
    from flask import Flask
    app=Flask(__name__)
    @app.route('/')
    
    def hello():
        return "hello docker"
    
    if __name__=="__main__":
        app.run(host='0.0.0.0',port=8080)
    [root@master home]# ls
    app.py  Dockerfile
    ```

  - 编写Dockerfile文件，touch Dockerfile（名字必须叫Dockerfile）

    ```shell
    [root@master home]#cp /etc/yum.repos.d/CentOS-Base.repo ./
    [root@master home]#cp /etc/yum.repos.d/epel.repo ./
    [root@master home]#ls   # 要确保再同一目录下
    CentOS-Base.repo  epel.repo   Dockerfile  s16-flask.py
    
    [root@master home]# vim Dockerfile
    FROM centos  
    COPY CentOS-Base.repo /etc/yum.repos.d/  #配置yum源
    COPY epel.repo /etc/yum.repos.d/
    RUN yum clean all
    RUN yum install python-setuptools -y  # 安装插件
    RUN easy_install flask  #安装flask
    COPY s16-flask.py /opt/  #拷贝文件
    WORKDIR /opt
    EXPOSE 8080
    CMD ["python","s16-flask.py"]
    ```

  - 构建docker镜像

    ```shell
    [root@master home]# docker build -t yuchao163/flask-hello-docker .
    	 docker build 编译Dockerfile
    	 -t 给镜像加上名字，镜像名字，以仓库地址开头，则可以推送到仓库中管理，如yuchao
    	 . 找到当前的Dockerfile文件
    	 
    5.启动此flask-hello-docker容器，映射一个端口供外部访问
    docker run -d -p 8080:8080 yuchao163/flask-hello-docker
    
    6.检查运行的容器
    docker container ls
    ```

    

    

  #### 推送本地镜像到dockerhub共有仓库，公有仓库

  - 登陆docker仓库

    ```
    docker login
    ```

  - 修改docker镜像文件名字，以docker hub 账号开头

    ```
    docker tag 要推的镜像id wanglix/s21wang-hello  #修改名字，我的账号wanglix
    ```

  - 推送镜像

    ```
    docker push wanglix/s21wang-hello
    ```

  #### 公有仓库

  ```
  1.docker提供了一个类似于github的仓库dockerhub,网址https://hub.docker.com/需要注册使用
  2.注册docker id后，在linux中登录dockerhub
  docker login
  
  注意要保证image的tag是账户名，如果镜像名字不对，需要改一下tag
  docker tag chaoyu/centos-vim yuchao163/centos-vim
  语法是：  docker tag   仓库名   yuchao163/仓库名
  
  3.推送docker image到dockerhub
  docker push yuchao163/centps-cmd-exec:latest
  4.在dockerhub中检查镜像
  https://hub.docker.com/
  5.删除本地镜像，测试下载pull 镜像文件
  docker pull yuchao163/centos-entrypoint-exec
  ```

  #### 私有仓库

  但是上面镜像仓库是公开的，其他人也是可以下载，并不安全，因此还可以使用docker registry官方提供的私有仓库

  ```shell
  1.官方提供的私有仓库docker registry用法
  https://yeasy.gitbooks.io/docker_practice/repository/registry.html
  2.一条命令下载registry镜像并且启动私有仓库容器，私有仓库会被创建在容器的/var/lib/registry下，因此通过-v参数将镜像文件存储到本地的/opt/data/registry下，端口映射容器中的5000端口到宿主机的5000端口
  docker run -d \
      -p 5000:5000 \  #宿主机的端口（自定义，自己考虑去分配）：容器内暴露的端口（django中写了8000）
      -v /opt/data/registry:/var/lib/registry \
      registry   
      
      #也就是说，宿主机和容器的5000对5000向同（公用），宿主机的/opt/data/registry:和容器/var/lib/registry 相通公用
      
  3.检查启动的registry容器
  docker ps
  4.测试连接容器
  telnet 192.168.119.10 5000
  
  5.修改镜像tag,以docker registry的地址端口开头
  docker tag hello-world:latest 192.168.119.10:5000/hello-world:latest
  6.查看docker镜像，找到registry的镜像
  docker images
  
  7.Docker 默认不允许非 HTTPS 方式推送镜像。我们可以通过 Docker 的配置选项来取消这个限制，这里必须写正确json数据
  [root@master /]# cat /etc/docker/daemon.json
  {"registry-mirrors": ["http://95822026.m.daocloud.io"],
  "insecure-registries":["192.168.119.10:5000"]
  }
  
  写入到docker服务中，写入到[Service]配置块中，加载此配置文件
  [root@master home]# grep 'EnvironmentFile=/etc/docker/daemon.json' /lib/systemd/system/docker.service
  [root@master home]# vim /lib/systemd/system/docker.service  #修改配置文件，在server选项头下添加下面键值：EnvironmentFile=-/etc/docker/daemon.json
  
  8.修改了docker配置文件，重新加载docker
  systemctl daemon-reload
  9.重启docker
  systemctl restart docker
  10.重启了docker，刚才的registry容器进程挂掉了，因此重新启动它
  docker run --privileged=true -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry registry
  --privileged=true  docker容器的安全机制：设置特权级运行的容器
    
  11.推送本地镜像
  docker push 192.168.119.10:5000/hello-world
  
  12.由于docker registry没有web节目，但是提供了API数据
  官网教程：https://docs.docker.com/registry/spec/api/#listing-repositories
  
  curl http://192.168.119.10:5000/v2/_catalog
  
  或者浏览器访问http://192.168.119.10:5000/v2/_catalog
  13.删除本地镜像，从私有仓库中下载
  docker pull 192.168.119.10:5000/hello-world
  ```

   