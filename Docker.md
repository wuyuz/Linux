## Docker

#### 什么是 Docker？

​		Docker 最初是 dotCloud 公司创始人 Solomon Hykes 在法国期间发起的一个公司内部项目，于 2013 年 3 月以 Apache 2.0 授权协议开源，主要项目代码在 GitHub 上进行维护。 Docker 使用 Google 公司推出的 Go 语言 进行开发实现。 docker是linux容器的一种封装，提供简单易用的容器使用接口。它是最流行的Linux容器解决方案。 docker的接口相当简单，用户可以方便的创建、销毁容器。 docker将应用程序与程序的依赖，打包在一个文件里面。运行这个文件就会生成一个虚拟容器。 程序运行在虚拟容器里，如同在真实物理机上运行一样，有了docker，就不用担心环境问题了。



#### Docker概念

​		Docker是开发人员和系统管理人员 使用容器**开发，部署和运行**应用程序的平台。使用Linux容器部署应用程序称为**容器化**。

容器化越来越受欢迎，因为容器是：

- 灵活：    即使是最复杂的应用也可以集装箱化。
- 轻量级：容器利用并共享主机内核。
- 可互换：您可以即时部署更新和升级。
- 便携式：您可以在本地构建，部署到云，并在任何地方运行。
- 可扩展：您可以增加并自动分发容器副本。
- 可堆叠：您可以垂直和即时堆叠服务。



#### 持续交付和部署

​		对开发和运维（[DevOps](https://zh.wikipedia.org/wiki/DevOps)）人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。

使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。开发人员可以通过 [Dockerfile](https://yeasy.gitbooks.io/docker_practice/image/dockerfile) 来进行镜像构建，并结合 [持续集成(Continuous Integration)](https://en.wikipedia.org/wiki/Continuous_integration) 系统进行集成测试，而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合 [持续部署(Continuous Delivery/Deployment)](https://en.wikipedia.org/wiki/Continuous_delivery) 系统进行自动部署。

而且使用 `Dockerfile` 使镜像构建透明化，不仅仅开发团队可以理解应用运行环境，也方便运维团队理解应用运行所需条件，帮助更好的生产环境中部署该镜像。



#### 更轻松的迁移

​		由于 Docker 确保了执行环境的一致性，使得应用的迁移更加容易。Docker 可以在很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是笔记本，其运行结果是一致的。因此用户可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。



#### Docker 镜像

​		我们都知道，操作系统分为内核和用户空间。对于 Linux 而言，内核启动后，会挂载 `root` 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 `root` 文件系统。比如官方镜像 `ubuntu:18.04` 就包含了完整的一套 Ubuntu 18.04 最小系统的 `root` 文件系统。（镜像是只读的）

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。



#### 分层存储

​		因为镜像包含操作系统完整的 `root` 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

​		镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。





#### 镜像和容器

> ### Images and containers
>
> A container is launched by running an image(通过运行镜像启动一个容器). An **image** is an executable package that includes everything needed to run an application--the code(所有容器需要的东西都放在镜像中), a runtime, libraries, environment variables, and configuration files.
>
> A **container** is a runtime instance of an image(容器是镜像运行起来的一个实例，可以理解为类和对象的关系，也就是说这个镜像有存在于了内存中，即携带者状态、数据)--what the image becomes in memory when executed (that is, an image with state, or a user process). You can see a list of your running containers with the command, `docker ps`, just as you would in Linux.

​		镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 [命名空间](https://en.wikipedia.org/wiki/Linux_namespaces)。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 Docker 时常常会混淆容器和虚拟机。

​		前面讲过镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为 **容器存储层**。

容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

​		按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 [数据卷（Volume）](https://yeasy.gitbooks.io/docker_practice/data_management/volume.html)、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。



#### 容器和传统虚拟机的比较

> A **container** runs *natively* on Linux and shares the kernel of the host machine with other containers. It runs a discrete process, taking no more memory than any other executable, making it lightweight.（一个运行在本地的容器将和其他容器共享Linux内核，并且是一个独立的进程，不再占用其他可执行程序的内存，使其轻量级）
>
> By contrast, a **virtual machine** (VM) runs a full-blown “guest” operating system with *virtual* access to host resources through a hypervisor. In general, VMs provide an environment with more resources than most applications need.(相比之下，虚拟机（VM）运行一个完整的“客户”操作系统，通过虚拟机管理程序对主机资源进行虚拟访问。通常，VM提供的环境比大多数应用程序需要的资源更多)

![1566101763488](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1566101763488.png)



#### 镜像体积

​		如果仔细观察，会注意到，这里标识的所占用空间和在 Docker Hub 上看到的镜像大小不同。比如，`ubuntu:18.04` 镜像大小，在这里是 `127 MB`，但是在 [Docker Hub](https://hub.docker.com/r/library/ubuntu/tags/) 显示的却是 `50 MB`。这是因为 Docker Hub 中显示的体积是压缩后的体积。在镜像下载和上传过程中镜像是保持着压缩状态的，因此 Docker Hub 所显示的大小是网络传输中更关心的流量大小。而 `docker image ls` 显示的是镜像下载到本地后，展开的大小，准确说，是展开后的各层所占空间的总和，因为镜像到本地后，查看空间的时候，更关心的是本地磁盘空间占用的大小。

另外一个需要注意的问题是，`docker image ls` 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。

你可以通过以下命令来便捷的查看镜像、容器、数据卷所占用的空间。

```
[root@MyHost docker]# docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              3                   2                   447MB               98.18MB (21%)
Containers          3                   0                   0B                  0B
Local Volumes       0                   0                   0B                  0B
```



#### 虚悬镜像

上面的镜像列表中，还可以看到一个特殊的镜像，这个镜像既没有仓库名，也没有标签，均为 `<none>`。

```
<none>       <none>              00285df0df87        5 days ago          342 MB
```

​		这个镜像原本是有镜像名和标签的，原来为 `mongo:3.2`，随着官方镜像维护，发布了新版本后，重新 `docker pull mongo:3.2` 时，`mongo:3.2` 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 `<none>`。除了 `docker pull` 可能导致这种情况，`docker build` 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 `<none>` 的镜像。这类无标签镜像也被称为 **虚悬镜像(dangling image)** ，可以用下面的命令专门显示这类镜像：i



####  Centos系统 Docker安装

- **前提**：Docker  运行在 CentOS 7 上，要求系统为64位、系统内核版本为 3.10 以上

  ```shell
  [root@MyHost ~]# uname -r   #查看内核版本
  3.10.0-514.26.2.el7.x86_64
  ```

  从 2017 年 3 月开始 docker 在原来的基础上分为两个分支版本: **Docker CE 和 Docker EE**。

  Docker CE 即社区免费版，Docker EE 即企业版，强调安全，但需付费使用。

  本文介绍 Docker CE 的安装使用。

- 移除旧版本

  ```shell
  [root@MyHost ~]# yum remove docker
  ```

- 安装一些必要的系统工具

  ```shell
  [root@MyHost ~]# yum install -y yum-utils device-mapper-persistent-data lvm2  
  	#设置存储库
  ```

- 添加软件源信息，这样docker才知道要下载那些需要的配置文件

  ```shell
  [root@MyHost yum]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  
  Loaded plugins: fastestmirror
  adding repo from: http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  grabbing file http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
  repo saved to /etc/yum.repos.d/docker-ce.repo
  
  [root@MyHost yum]# ll /etc/yum.repos.d/
  -rw-r--r-- 1 root root  675 Jul 30 23:41 CentOS-Base.repo
  -rw-r--r-- 1 root root 2640 Aug 17 15:52 docker-ce.repo  #添加了源信息
  -rw-r--r-- 1 root root  230 Jul 30 23:41 epel.repo
  
  [root@MyHost yum]# yum makecache fast   #更新yum缓存
  ```

- 安装docker-ce 

  ```shell
  [root@MyHost yum]# yum install docker-ce -y
  ```

- 启动docker并运行测试hello-world

  ```shell
  [root@MyHost yum]# systemctl start docker   #启动
  [root@MyHost yum]# docker run hello-world   #测试，hello-world是镜像名称
  Unable to find image 'hello-world:latest' locally
  latest: Pulling from library/hello-world
  1b930d010525: Pull complete 
  Digest: sha256:6540fc08ee6e6b7b63468dc3317e3303aae178cb8a45ed3123180328bcc1d20f
  ...
  [root@MyHost yum]# docker ps -a  # 查看所有运行的容器
  CONTAINER ID IMAGE  COMMAND  CREATED STATUS PORTS    NAMES
  a12c224e2f8a        hello-world         "/hello"            About a minute ago  
  
  [root@MyHost yum]# docker image ls  #列出下载好的镜像
  REPOSITORY  TAG  IMAGE ID CREATED SIZE
  hello-world  latest   fce289e99eb9  7 months ago    1.84kB
  
  [root@MyHost yum]# docker container ls --all  #列出存在的容器
  CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS   NAMES
  a12c224e2f8a   hello-world "/hello" 8 minutes ago   
  ```

- 创建或修改配置镜像加速：要确保有docker文件 ，没有就创建mkdir -p /etc/docker

  ```shell
  [root@MyHost docker]# vim /etc/docker/daemon.json  #没有就自己创建
  [root@MyHost docker]# systemctl daemon-reload
  
   #添加文件
  {
      "registry-mirrors": [
          "https://1nj0zren.mirror.aliyuncs.com",
          "https://docker.mirrors.ustc.edu.cn",
          "http://f1361db2.m.daocloud.io",
          "https://registry.docker-cn.com"
      ]
  }
  
  [root@MyHost docker]# systemctl restart docker
  ```

- 查看docker帮助信息

  ```shell
  [root@docker ~]# docker --help
  ```



##### 启动容器

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`stopped`）的容器重新启动。因为 Docker 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。

```shell
[root@MyHost docker]# docker ps -l
CONTAINER ID  IMAGE  COMMAND CREATED STATUS    PORTS     NAMES
cb730aaf3b4f  hello-world  "/hello"  17 seconds ago ited (0) 16 seconds ago          interesting_brahmagupta

[root@MyHost docker]# docker run --name hello -d hello-world  #运行并重命名
8c1aa4d89e5dd7e29619aabbf5a7901e9ba1bfe4e1c691e2bbb03a0c4fe9351f
[root@MyHost docker]# docker ps -l
CONTAINER ID IMAGE  COMMAND  CREATED   STATUS    PORTS   NAMES
8c1aa4d89e5d   hello-world  "/hello" 3 seconds ago Exited (0) 3 seconds ago      hello
```

- 进入容器中操作

  ```shell
  [root@MyHost docker]# docker run -it centos bash
  [root@1b693fb7df97 /]#    #输入exit退出
  
  -i 交互式操作
  -t 终端
  --rm 容器退出，并删除容器
  -d 后台运行容器
  ```

- 运行后删除容器

  ```shell
  [root@MyHost ~]# docker run -it --rm ubuntu bash
  ```

- 退出容器不关闭容器

  ```shell
  [root@MyHost ~]# docker run -it centos bash
  [root@1ae6c51822a1 /]#    #按住ctrl+p,q，退出不关闭容器
  [root@MyHost ~]# docker ps
  CONTAINER ID IMAGE COMMAND  CREATED STATUS PORTS  NAMES
  1ae6c51822a1   centos  "bash"  25 seconds ago Up 24 seconds                   sharp_tesla
  ```

- 新建并启动

  所需要的命令主要为 `docker run`。

  例如，下面的命令输出一个 “Hello World”，之后终止容器。

  ```shell
  [root@MyHost ~]# docker run centos /bin/echo 'Hello world'
  Hello world
  
  [root@MyHost ~]# docker run -t -i ubuntu:18.04 /bin/bash  #启动并进入交互模式
  root@af8bae53bdd3:/#
  ```

  当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

  - 检查本地是否存在指定的镜像，不存在就从公有仓库下载
  - 利用镜像创建并启动一个容器
  - 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
  - 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
  - 从地址池配置一个 ip 地址给容器
  - 执行用户指定的应用程序
  - 执行完毕后容器被终止



- ##### 启动已终止容器

  可以利用 `docker container start` 命令，直接将一个已经终止的容器启动运行。

  ```shell
  [root@MyHost ~]# docker stop 1ae
  1ae
  [root@MyHost ~]# docker start 1ae
  1ae
  ```



##### 搜索docker镜像

```shell
[root@docker ~]# docker search centos  #搜索所有centos的docker镜像INDEX      
```



##### 下载镜像

​		之前提到过，[Docker Hub](https://hub.docker.com/explore/) 上有大量的高质量的镜像可以用，这里我们就说一下怎么获取这些镜像。

从 Docker 镜像仓库获取镜像的命令是 `docker pull`。其命令格式为

```shell
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

[root@MyHost docker]# docker pull redis
Using default tag: latest
latest: Pulling from library/redis  
1ab2bdfe9778: Pull complete    #我们之前说的docker是分层存储的，不同的端口号代表不同的镜像依赖
966bc436cc8b: Pull complete 
c1b01f4f76d9: Pull complete 
8a9a85c968a2: Pull complete 
8e4f9890211f: Pull complete 
93e8c2071125: Pull complete 
Digest: sha256:9755880356c4ced4ff7745bafe620f0b63dd17747caedba72504ef7bac882089
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest

[root@MyHost docker]# docker image ls
REPOSITORY          TAG                 IMAGE ID         CREATED     SIZE
redis               latest              f7302e4ab3a8   3 days ago    98.2MB
```

列表包含了 `仓库名`、`标签`、`镜像 ID`、`创建时间` 以及 `所占用的空间`。

​		其中仓库名、标签在之前的基础概念章节已经介绍过了。**镜像 ID** 则是镜像的唯一标识，一个镜像可以对应多个 **标签**。因此，在上面的例子中，我们可以看到 `ubuntu:18.04` 和 `ubuntu:latest` 拥有相同的 ID，因为它们对应的是同一个镜像。



#### 查看本地镜像

```shell
[root@MyHost ~]# docker images
[root@MyHost ~]# docker image ls  #查看镜像

[root@MyHost ~]# docker images -q  #指向显示镜像id
```

- 删除镜像，默认情况下不能删除正在有容器运行的镜像，必须终止

  ```shell
   #方式一：
  [root@MyHost ~]# docker rmi -f hello-world
   #方式二：
  [root@MyHost ~]# docker rm 8c cb7 a12  #删除三个正在运行的容器
  8c
  cb7
  a12
  [root@MyHost ~]# docker rmi hello-world
  
   #方式三：
  [root@MyHost ~]# docker rmi `docker images -q`  #多项删除
  ```

- 删除容器

  ```shell
   #docker rm id|name 删除容器
  默认不能删除启动中的容器
  -f 强制删除
  [root@MyHost ~]# docker rm 8c 
  ```

- 进入正在启动中的容器

  ```shell
   #docker exec -it id|name bash
  bash 是进入容器后执行的命令
  [root@MyHost ~]# docker exec -it 1ae6c51822a1 bash
  [root@1ae6c51822a1 /]# 
  ```

- 用 docker image ls 命令来配合

  像其它可以承接多个实体的命令一样，可以使用 `docker image ls -q` 来配合使用 `docker image rm`，这样可以成批的删除希望删除的镜像。我们在“镜像列表”章节介绍过很多过滤镜像列表的方式都可以拿过来使用。

  比如，我们需要删除所有仓库名为 `redis` 的镜像：

  ```bash
  $ docker image rm $(docker image ls -q redis)
  ```

  或者删除所有在 `mongo:3.2` 之前的镜像：

  ```bash
  $ docker image rm $(docker image ls -q -f before=mongo:3.2)
  ```

  充分利用你的想象力和 Linux 命令行的强大，你可以完成很多非常赞的功能。

  

##### 查看容器日志

```shell
 #docker logs id|name
-f 查看实时日志输出

[root@MyHost ~]# docker run -d redis  #后台运行redis，通过docker ps -a显示
[root@MyHost ~]# docker logs 468   #468是容器的前三位
```



##### 导出镜像

```shell
 #如果要导出镜像到本地文件,可以使用docker save命令。
[root@MyHost ~]# docker save -o /opt/centos.tar.gz centos  # -o 指定到那个文件下
[root@MyHost ~]# ll /opt/centos.tar.gz 
-rw------- 1 root root 209497088 Aug 18 16:27 /opt/centos.tar.gz

 #方式二：
docker save -o name imagename|id
docker save id|imagesname >  /opt/centos.tar.gz
```

##### 导入镜像

```shell
[root@MyHost ~]# docker load -i /opt/centos.tar.gz 
Loaded image: centos:latest

[root@MyHost ~]# docker load < /opt/centos.tar.gz 
```



#### 利用 commit 理解镜像构成

​		注意： `docker commit` 命令除了学习之外，还有一些特殊的应用场合，比如被入侵后保存现场等。但是，不要使用 `docker commit` 定制镜像，定制镜像应该使用 `Dockerfile` 来完成。如果你想要定制镜像请查看下一小节。

​		镜像是容器的基础，每次执行 `docker run` 的时候都会指定哪个镜像作为容器运行的基础。在之前的例子中，我们所使用的都是来自于 Docker Hub 的镜像。直接使用这些镜像是可以满足一定的需求，而当这些镜像无法直接满足需求时，我们就需要定制这些镜像。接下来的几节就将讲解如何定制镜像。

回顾一下之前我们学到的知识，镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。

现在让我们以定制一个 Web 服务器为例子，来讲解镜像是如何构建的。

```bash
$ docker run --name webserver -d -p 80:80 nginx
```

这条命令会用 `nginx` 镜像启动一个容器，命名为 `webserver`，并且映射了 80 端口，这样我们可以用浏览器去访问这个 `nginx` 服务器。

如果是在 Linux 本机运行的 Docker，或者如果使用的是 Docker for Mac、Docker for Windows，那么可以直接访问：[http://localhost](http://localhost/)；如果使用的是 Docker Toolbox，或者是在虚拟机、云服务器上安装的 Docker，则需要将 `localhost` 换为虚拟机地址或者实际云服务器地址。

直接用浏览器访问的话，我们会看到默认的 Nginx 欢迎页面。

![1566119232053](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1566119232053.png)

现在，假设我们非常不喜欢这个欢迎页面，我们希望改成欢迎 Docker 的文字，我们可以使用 `docker exec` 命令进入容器，修改其内容。

```bash
$ docker exec -it webserver bash
root@3729b97e8226:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
root@3729b97e8226:/# exit
exit
```

​		我们以交互式终端方式进入 `webserver` 容器，并执行了 `bash` 命令，也就是获得一个可操作的 Shell。然后，我们用 `<h1>Hello, Docker!</h1>` 覆盖了 `/usr/share/nginx/html/index.html` 的内容。现在我们再刷新浏览器的话，会发现内容被改变了。

![1566119246114](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1566119246114.png)

我们修改了容器的文件，也就是改动了容器的存储层。我们可以通过 `docker diff` 命令看到具体的改动。

```bash
$ docker diff webserver
C /root
A /root/.bash_history
C /run
C /usr
C /usr/share
C /usr/share/nginx
C /usr/share/nginx/html
C /usr/share/nginx/html/index.html
C /var
C /var/cache
C /var/cache/nginx
A /var/cache/nginx/client_temp
A /var/cache/nginx/fastcgi_temp
A /var/cache/nginx/proxy_temp
A /var/cache/nginx/scgi_temp
A /var/cache/nginx/uwsgi_temp
```

现在我们定制好了变化，我们希望能将其保存下来形成镜像。

​		要知道，当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。

`docker commit` 的语法格式为：

```bash
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```

我们可以用下面的命令将容器保存为镜像：

```bash
$ docker commit \
    --author "Tao Wang <twang2218@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2
sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214
```

​		其中 `--author` 是指定修改的作者，而 `--message` 则是记录本次修改的内容。这点和 `git` 版本控制相似，不过这里这些信息可以省略留空。

我们可以在 `docker image ls` 中看到这个新定制的镜像：

```bash
$ docker image ls nginx
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               v2                  07e334659748        9 seconds ago       181.5 MB
nginx               1.11                05a60462f8ba        12 days ago         181.5 MB
nginx               latest              e43d811ce2f4        4 weeks ago         181.5 MB
```

​		我们还可以用 `docker history` 具体查看镜像内的历史记录，如果比较 `nginx:latest` 的历史记录，我们会发现新增了我们刚刚提交的这一层。

```bash
$ docker history nginx:v2
IMAGE  CREATED  CREATED BY    SIZE       COMMENT
07e334659748        54 seconds ago      nginx -g daemon off;                            95 B                修改了默认网页
e43d811ce2f4        4 weeks ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon    0 B
<missing>           4 weeks ago         /bin/sh -c #(nop)  EXPOSE 443/tcp 80/tcp        0 B
<missing>           4 weeks ago         /bin/sh -c ln -sf /dev/stdout /var/log/nginx/   22 B
<missing>           4 weeks ago         /bin/sh -c apt-key adv --keyserver hkp://pgp.   58.46 MB
<missing>           4 weeks ago         /bin/sh -c #(nop)  ENV NGINX_VERSION=1.11.5-1   0 B
<missing>           4 weeks ago         /bin/sh -c #(nop)  MAINTAINER NGINX Docker Ma   0 B
<missing>           4 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0 B
<missing>           4 weeks ago         /bin/sh -c #(nop) ADD file:23aa4f893e3288698c   123 MB
```

新的镜像定制好后，我们可以来运行这个镜像。

```bash
docker run --name web2 -d -p 81:80 nginx:v2
```

​		这里我们命名为新的服务为 `web2`，并且映射到 `81` 端口。如果是 Docker for Mac/Windows 或 Linux 桌面的话，我们就可以直接访问 [http://localhost:81](http://localhost:81/) 看到结果，其内容应该和之前修改后的 `webserver` 一样。

至此，我们第一次完成了定制镜像，使用的是 `docker commit` 命令，手动操作给旧的镜像添加了新的一层，形成新的镜像，对镜像多层存储应该有了更直观的感觉。



##### 简单使用commit

- 进入容器，创建一个文件夹，进行提交

  ```shell
  [root@MyHost ~]# docker exec -it 1ae bash  # 1ae是容器id
  [root@1ae6c51822a1 /]# mkdir /data  #进入后创建一个，使用ctl+p,q
  
  [root@MyHost ~]# docker ps
  CONTAINER ID IMAGE COMMAND CREATED STATUS  PORTS   NAMES
  1ae6c51822a1        centos              "bash"              2 hours ago         Up About an hour      sharp_tesla
  
  [root@MyHost ~]# docker commit -m "更新data" 1ae  #将信息提交
  sha256:6ee6298a139203af27162700c21ca77b274e9cf31edd7538069d0b86a7a29a01
  ```

- 再次查看镜像文件

  ```shell
  [root@MyHost ~]# docker images
  REPOSITORY TAG IMAGE ID CREATED   SIZE
  <none>    <none>     6ee6298a1392       About a minute ago   202MB
  centos   latest      9f38484d220f        5 months ago         202MB
   #发现多了一个无名的仓库，我们可以对其进行配置信息
  ```

- 修改镜像名字

  ```shell
  [root@MyHost ~]# docker tag 6ee mycentos
  [root@MyHost ~]# docker images
  mycentos   latest     6ee6298a1392       About a minute ago   202MB
  centos   latest      9f38484d220f        5 months ago         202MB
  
  #docker tag 原来名称 新名称
  如果不存在tag,则在原来的镜像基础上加上tag信息,如果存在原来的tag信息,则会复制一份
  ```

  像这样创建的仓库，经过拷贝解压，下次还会存在

- 使用新创建的进行运行一条命令

  ```shell
  [root@MyHost ~]# docker run mycentos bash -c "echo s21"
  s21
  [root@MyHost ~]# docker run mycentos /bin/echo "echo s21"
  echo s21
  ```
  


#### 后台运行

​		更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

下面举两个例子来说明一下。

如果不使用 `-d` 参数运行容器。

```bash
$ docker run ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
hello world
hello world
hello world
hello world
```

容器会把输出的结果 (STDOUT) 打印到宿主机上面

如果使用了 `-d` 参数运行容器。

```bash
$ docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
```

此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面(输出结果可以用 `docker logs`查看)。**注：** 容器是否会长久运行，是和 `docker run` 指定的命令有关，和 `-d` 参数无关。

使用 `-d` 参数启动后会返回一个唯一的 id，也可以通过 `docker ps` 命令来查看容器信息。

```
$ docker ps
CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
77b2dc01fe0f  ubuntu:18.04  /bin/sh -c 'while tr  2 minutes ago  Up 1 minute        agitated_wright
```

要获取容器的输出信息，可以通过 `docker logs` 命令。

```bash
$ docker logs [container ID or NAMES]
hello world
hello world
hello world
. . .
```



#### 终止容器

​		可以使用 `docker stop` 来终止一个运行中的容器。此外，当 Docker 容器中指定的应用终结时，容器也自动终止。例如对于上一章节中只启动了一个终端的容器，用户通过 `exit` 命令或 `Ctrl+d` 来退出终端时，所创建的容器立刻终止。终止状态的容器可以用 `docker ps -a` 命令看到。例如

```bash
docker ps -a
CONTAINER ID IMAGE  COMMAND  CREATED  STATUS  PORTS   NAMES
ba267838cc1b  ubuntu:18.04 "/bin/bash" 30 minutes ago  Exited (0) About
```

​		处于终止状态的容器，可以通过 `docker start` 命令来重新启动。此外，`docker restart` 命令会将一个运行态的容器终止，然后再重新启动它。在使用 `-d` 参数时，容器启动后会进入后台。

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令，原因会在下面说明。



#### attach 命令

下面示例如何使用 `docker attach` 命令。

```bash
$ docker run -dit ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia

$ docker attach 243c
root@243c32535da7:/#
```

*注意：* 如果从这个 stdin 中 exit，会导致容器的停止。



#### exec 命令

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```bash
$ docker run -dit ubuntu
69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles

$ docker exec -i 69d1 bash
ls
bin
boot
dev
...

$ docker exec -it 69d1 bash
root@69d137adef7a:/#
```

如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 `docker exec` 的原因。

更多参数说明请使用 `docker exec --help` 查看。



#### 删除容器

可以使用 `docker container rm` 来删除一个处于终止状态的容器。例如

```bash
$ docker  rm  trusting_newton
trusting_newton
```

如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器。



#### 清理所有处于终止状态的容器

用 `docker container ls -a` 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

```bash
$ docker container prune

[root@MyHost ~]# docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
...
[root@MyHost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1ae6c51822a1        centos              "bash"              8 hours ago         Up 7 hours                              sharp_tesla
```



#### 匿名数据卷

将宿主机的文件挂载到容器里面

```shell
[root@MyHost ~]# docker run -it -v /opt/myetc:/etc centos bash
[root@MyHost ~]# docker run -it -v /opt/myetc:/etc centos bash
bash-4.2# pwd     #这里的 /data 目录就会在运行时自动挂载为匿名卷，任何向 /data 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行时可以覆盖这个挂载设置
/   
bash-4.2# cd /etc    # 容器对etc的操作会同步到属主机的/opt/etc上
bash-4.2# touch b
bash-4.2# ll /etc/   # 没有任何东西，都写进属主机的/opt/myetc中了
bash: ll: command not found


 #另一个终端
[root@MyHost ~]# cd /opt/myetc/
[root@MyHost myetc]# ll
-rw-r--r-- 1 root root 0 Aug 18 23:52 b   # 新建的b
-rwxr-xr-x 1 root root 0 Aug 18 23:47 hostname
-rwxr-xr-x 1 root root 0 Aug 18 23:47 hosts
-rwxr-xr-x 1 root root 0 Aug 18 23:47 resolv.conf

```

#### 数据卷

挂载一个主机目录作为数据卷，使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去。

```shell
$ docker run -d -P \   #-P或-p指定端口
    --name web \
    # 类似于 -v /src/webapp:/opt/webapp \ 容器的/opt/webapp数据同步到属主机的/src/webapp
    --mount type=bind,source=/src/webapp,target=/opt/webapp \
    training/webapp \
    python app.py
```

​		上面的命令加载主机的 `/src/webapp` 目录到容器的 `/opt/webapp`目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，以前使用 `-v` 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用 `--mount`参数时如果本地目录不存在，Docker 会报错。

Docker 挂载主机目录的默认权限是 `读写`，用户也可以通过增加 `readonly` 指定为 `只读`

```python
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/opt/webapp:ro \
    --mount type=bind,source=/src/webapp,target=/opt/webapp,readonly \
    training/webapp \
    python app.py
    
 #加了 readonly 之后，就挂载为 只读 了。如果你在容器内 /opt/webapp 目录新建文件，会显示如下错误
/opt/webapp # touch new.txt
touch: new.txt: Read-only file system
```



#### 挂载一个本地主机文件作为数据卷

`--mount` 标记也可以从主机挂载单个文件到容器中

```bash
$ docker run --rm -it \
   # -v $HOME/.bash_history:/root/.bash_history \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:18.04 \
   bash

root@2affd44b4667:/# history
1  ls
2  diskutil list
```

这样就可以记录在容器输入过的命令了。



#### 外部访问容器

​		容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 `-P` 或 `-p` 参数来指定端口映射。当使用 `-P` 标记时，Docker 会**随机**映射一个 `49000~49900` 的端口到内部容器开放的网络端口。

使用 `docker container ls` 可以看到，本地主机的 49155 被映射到了容器的 5000 端口。此时访问本机的 49155 端口即可访问容器内 web 应用提供的界面。

```bash
$ docker run -d -P training/webapp python app.py

$ docker container ls -l   # 查看端口信息
CONTAINER ID  IMAGE  COMMAND  CREATED STATUS PORTS   NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse
```

同样的，可以通过 `docker logs` 命令来查看应用的信息。

```bash
$ docker logs -f nostalgic_morse
* Running on http://0.0.0.0:5000/
10.0.2.2 - - [23/May/2014 20:16:31] "GET / HTTP/1.1" 200 -
10.0.2.2 - - [23/May/2014 20:16:31] "GET /favicon.ico HTTP/1.1" 404 -
```

`-p` 则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 `ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`。



#### 映射所有接口地址

使用 `hostPort:containerPort` 格式本地的 5000 端口映射到容器的 5000 端口，可以执行

```bash
$ docker run -d -p 5000:5000 training/webapp python app.py  #此时默认会绑定本地所有接口上的所有地址。
```



#### 映射到指定地址的指定端口

可以使用 `ip:hostPort:containerPort` 格式指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1

```bash
$ docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py

 #语法
docker run -d -P redis   #端口是随机产生
docker run -d -p 宿主机上的端口:容器内的端口 redis  #指定端口
```



#### 映射到指定地址的任意端口

使用 `ip::containerPort` 绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口。

```bash
$ docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

还可以使用 `udp` 标记来指定 `udp` 端口

```bash
$ docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```



#### 查看映射端口配置

使用 `docker port` 来查看当前映射的端口配置，也可以查看到绑定的地址

```bash
$ docker port nostalgic_morse 5000
127.0.0.1:49155.
```

注意：

- 容器有自己的内部网络和 ip 地址（使用 `docker inspect` 可以获取所有的变量，Docker 还可以有一个可变的网络配置。）
- `-p` 标记可以多次使用来绑定多个端口

例如

```bash
$ docker run -d \
    -p 5000:5000 \
    -p 3000:80 \
    training/webapp \
    python app.py
    
 #查看端口
[root@MyHost ~]# docker run -d -P redis  #使用默认的redis本机端口，宿主机端口随机
a508f359c2217325b074fdd866dc93ef714a089cfaa1c38380f5aae3b1092eff
[root@MyHost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
a508f359c221        redis               "docker-entrypoint.s…"   8 seconds ago       Up 8 seconds        0.0.0.0:32771->6379/tcp   fervent_tu
1ae6c51822a1        centos              "bash"                   17 hours ago        Up 16 hours                                   sharp_tesla
[root@MyHost ~]# docker port a5
6379/tcp -> 0.0.0.0:32771

[root@MyHost ~]# ss -tnlp   #查看属主机的端口信息
State      Recv-Q Send-Q                  Local Address:Port                                 Peer Address:Port              
LISTEN     0      128                                :::32771                                          :::*                   users:(("docker-proxy",pid=21173,fd=4))
```



#### 实时查看容器的资源占用

```shell
[root@MyHost ~]# docker stats a50
```



#### 使用 Dockerfile 定制镜像

​		从刚才的 `docker commit` 的学习中，我们可以了解到，镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。还以之前定制 `nginx` 镜像为例，这次我们使用 Dockerfile 来定制。

在一个空白目录中，建立一个文本文件，并命名为 `Dockerfile`：

```bash
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile   #进入到新建文件中，新建一个Dockerfile文件
```

其内容为：

```dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

这个 Dockerfile 很简单，一共就两行。涉及到了两条指令，`FROM` 和 `RUN`。

- ##### 简单实例

  先创建一个空目录，在目录中创建Dockerfile文件

  ```shell
  [root@MyHost ~]# cd file/
  [root@MyHost file]# ll
  [root@MyHost file]# vim Dockerfile
  FROM mycentos   #处于哪一个镜像
  RUN yum install -y wget  #使用一层执行程序
  RUN mkdir /mydata
  COPY a.txt /mydata
  ```

  构建镜像

  ```shell
  [root@MyHost file]# docker build -t mydocker .  # 当下文件的Dockerfile文件加载
  Sending build context to Docker daemon  14.85kB
  Step 1/4 : FROM mycentos
   ---> 6ee6298a1392
  Step 2/4 : RUN yum install -y wget
   ---> Running in 2da40ce18935
  ...
  
  [root@MyHost file]# docker run -it mydocker bash  #启动我们自定义的镜像，这一样就可以看见我们创建的/mydata 
  [root@108addeb12fc /]#  ll mydata   #进入了我们创建的mydata目录，发现创建的a.txt文件
  -rw-r--r-- 1 root root 7 Aug 19 14:56 a.txt
  ```

- ##### 简单实例2

  如何使用ADD，自动解压打包文件给镜像

  ```shell
  [root@MyHost file]# tar zcf etc.tar.gz /etc/sysconfig/  #现在本地生成一个压缩文件
  tar: Removing leading `/' from member names
  [root@MyHost file]# ll
  -rw-r--r-- 1 root root     7 Aug 19 22:56 a.txt
  -rw-r--r-- 1 root root    51 Aug 19 23:19 Dockerfile
  -rw-r--r-- 1 root root 49835 Aug 19 23:25 etc.tar.gz   #我们向打包的文给镜像
  ```

  编写Dockerfile文件

  ```shell
  [root@MyHost file]# vim Dockerfile 
  FROM mycentos
  RUN mkdir /mydata
  COPY a.txt /mydata   # 将本地文件给镜像中
  ADD etc.tar.gz /mydata
  
  [root@MyHost file]# docker run -it mydocker2 bash
  [root@de649918e028 /]# ll /mydata/
  total 8
  -rw-r--r-- 1 root root  7 Aug 19 14:56 a.txt
  -rw-r--r-- 1 root root 45 Aug 19 15:30 etc.tar.gz
  ```

- ##### 小小总结：

  ```shell
  FROM mycentos  #指定基础镜像
  RUN yum install -y wget #执行命令
  RUN mkdir /mydata
  COPY a.txt /mydata #将本地文件复制到镜像里面
  ADD etc.tar.gz /mydata  #将本地文件复制到进项内,如果是压缩包,则自动解压
  WORKDIR /mydata #指定工作目录,exec 进入时候默认的目录
  ENV #设置变量
  VOLUME #设置数据卷
  EXPOSE 5900 #设置端口
  CMD ["nginx", "-g", "daemon off;"] #执行命令
  
  
  copy 和add的区别
  add 是自动解压
  CMD只能有一个,RUN可以有多个
  ```

  



#### FROM 指定基础镜像

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 `nginx` 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 `FROM` 就是指定 **基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。

在 [Docker Hub](https://hub.docker.com/search?q=&type=image&image_filter=official) 上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像，如 [`nginx`](https://hub.docker.com/_/nginx/)、[`redis`](https://hub.docker.com/_/redis/)、[`mongo`](https://hub.docker.com/_/mongo/)、[`mysql`](https://hub.docker.com/_/mysql/)、[`httpd`](https://hub.docker.com/_/httpd/)、[`php`](https://hub.docker.com/_/php/)、[`tomcat`](https://hub.docker.com/_/tomcat/) 等；也有一些方便开发、构建、运行各种语言应用的镜像，如 [`node`](https://hub.docker.com/_/node)、[`openjdk`](https://hub.docker.com/_/openjdk/)、[`python`](https://hub.docker.com/_/python/)、[`ruby`](https://hub.docker.com/_/ruby/)、[`golang`](https://hub.docker.com/_/golang/) 等。可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如 [`ubuntu`](https://hub.docker.com/_/ubuntu/)、[`debian`](https://hub.docker.com/_/debian/)、[`centos`](https://hub.docker.com/_/centos/)、[`fedora`](https://hub.docker.com/_/fedora/)、[`alpine`](https://hub.docker.com/_/alpine/) 等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

```dockerfile
FROM scratch
...
```

如果你以 `scratch` 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如 [`swarm`](https://hub.docker.com/_/swarm/)、[`coreos/etcd`](https://quay.io/repository/coreos/etcd)。对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 `FROM scratch` 会让镜像体积更加小巧。使用 [Go 语言](https://golang.org/) 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。



#### RUN 执行命令

`RUN` 指令是用来执行命令行命令的。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。其格式有两种：

- *shell* 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 `RUN` 指令就是这种格式。

```Dockerfile
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

- *exec* 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

既然 `RUN` 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这样：

```dockerfile
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

​		之前说过，Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是很多初学 Docker 的人常犯的一个错误。

*Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。*

上面的 `Dockerfile` 正确的写法应该是这样：

```dockerfile
FROM debian:stretch

RUN buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

​		首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 `RUN` 对一一对应不同的命令，而是仅仅使用一个 `RUN` 指令，并使用 `&&` 将各个所需命令串联起来。将之前的 7 层，简化为了 1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加 `\` 的命令换行方式，以及行首 `#` 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。

​		此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 `apt` 缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。很多人初学 Docker 制作出了很臃肿的镜像的原因之一，就是忘记了每一层构建的最后一定要清理掉无关文件。