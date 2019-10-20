## Ansible的介绍和使用



#### ansible介绍

​    官方的title是“Ansible is Simple IT Automation”——简单的自动化IT工具。

 	Ansible跟其他IT自动化技术的区别在于其关注点并非配置管理、应用部署或IT流程工作流，而是提供一个统一的界面来协调所有的IT自动化功能，因此Ansible的系统更加易用，部署更快。

​	 Ansible可以让用户避免编写脚本或代码来管理应用，同时还能搭建工作流实现IT任务的自动化执行。IT自动化可以降低技术门槛及对传统IT的依赖，从而加快项目的交付速度。



#### ansible优缺点

```
 优点：
	- 轻量级，他不需要去客户端安装agent，更新时，只需要在操作机上进行一次更新即可
	- 批量任务执行可以写成脚本，而且不用分发到远程就可以执行
	- 使用python编写的，维护更简单
	- 支持sudo

 缺点：
	- 对于几千台、上万台机器的操作，还不清楚性能、效率情况如何，需要进一步了解。
```



####  ansible架构及工作原理 

![1571567359403](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1571567359403.png)

```
ansible core ： ansible 自身核心模块
host inventory： 主机库，定义可管控的主机列表
connection plugins： 连接插件，一般默认基于 ssh 协议连接
modules：core modules ( 自带模块 ) 、 custom modules ( 自定义模块 )
playbooks ：剧本，按照所设定编排的顺序执行完成安排任务 
```



#### 安装与部署

- 安装epel源

  ```
  yum install http://mirrors.163.com/centos/7.4.1708/extras/x86_64/Packages/epel-release-7-9.noarch.rpm
  ```

  查看epel源并安装ansible

  ```
  [root@node2 ~]ll /etc/yum.repos.d/epel*
  [root@node2 ~]yum install ansible -y
  [root@node2 ~]ansible --version
  ```

- salt 控制节点需要安装 salt-master; 被控制节点安装salt-minion；ansible 通过ssh来连接并控制被控节点

- ssh 的认证方式

  - 密码连接

  - 秘钥连接

    ```
    ssh 秘钥登陆的配置方式
    1、在控制端生成公、私钥： [root@node2 ~]ssh-keygen  # 生成密钥对
    2、[root@node2 ~]ssh-copy-id 192.168.107.131  # 复制密钥到远程主机，之后就可以直接ssh '192.168.107.131' 就可以登陆另一个主机了
    ```



#### ansible命令格式

```
ansible <host-pattern> [options]

常用命令：
	-a MODULE_ARGS --args=MODULE_ARGS # 模块的参数
	-C --check # 检查
	-f FORKS # 用来做高并发的
	-l  # 列出主机列表
	-m MODULE——NAME # 模板名字
	-k 输入密码
```

- 查看ansible安装后生成了多少文件

  ```
  [root@node2 ~]rpm -ql ansible|more
  ```

- 默认情况下ansible是不知道，绑定了那些被控节点的，需要配置/etc/ansible/hosts文件

  ```
  [root@node2 ~] vim /etc/ansible/hosts
  
  # 在空白出添加要控制的节点的ip，需要注意的是这三台ip必须经过密钥认证，否则不行
  ```

  认证过的ip，通过以下命令

  ```
  [root@node2 ~]ansible 192.168.103.11 -m ping
  ```

  未认证的ip，可通过：

  ```
  [root@node2 ~]ansible 192.168.12.1 -m ping -k  # 后输入密码
  ```

  之后我们可以通过all，来发送命令

  ```
  [root@node2 ~]ansible all -m ping
  
  # 或者通过几个
  [root@node2 ~]ansible 192.168.103.11,192.168.103.12 -m ping
  ```

- hosts文件中根据[ ]进行分组ip，之前我们没有分组，有时候操作不方便，如只需要某一些ip主机做事情的时候，只能手输入，不方便，所以分组，格式如下

  ```shell
  [root@node2 ~] vim /etc/ansible/hosts
  ...
  [web]
  192.168.12.12
  192.168.12.11
  
  [db]
  192.168.12.[14:100]  # 支持切片
  192.168.12.121
  ...
  
  [root@node2 ~] ansible web --list-hosts  # 查看该组的ip主机
  hotss(2):
  	192.168.12.12
  	192.168.12.11
  	
  [root@node2 ~] ansible web,db --list-hosts
  [root@node2 ~] ansible 'web:&db' -m ping #取交集
  [root@node2 ~] ansible 'web:!db' -m ping #取差集
  ```



#### ansible-doc 查看模块的帮助信息

```
1、ansible-doc -j  # 以json的方式返回ansible的所有模块
2、ansible-doc -l  # 列出可用模块
3、ansible-doc -s  # 以片段式返回帮助信息 
	如：ansible-doc -s ping
```



#### 命令相关模块 command

```python
[root@node2 ~] ansible web -a 'ls /'  # 列出web组下的机器执行ls命令结果

[root@node2 ~] ansible web -a 'pwd'

[root@node2 ~] ansible web -a 'chdir=/tmp pwd' #切换目录执行命令，在编译安装时

[root@node2 ~] ansible web -a 'useradd alex'  # 给web组下创建用户
```



#### SHELL模块相关

```shell
[root@node2 ~] ansible web -m shell -a 'echo "123"|passwd --stdin alex' # 绑定用户密码和账号

[root@node2 ~] ansible web -m shell -a 'bash a.sh' # 执行远程文件的方式一
[root@node2 ~] ansible web -m shell -a '/root/ a.sh' # 执行远程文件的方式二,但是要求文件要有执行权限
[root@node2 ~] ansible web -m shell -a '/root/ a.py' 
```



#### script模块相关

```shell
[root@node2 ~] vim xxx.sh
#!/bin/bash
mkdir /msss

[root@node2 ~]chomd +x xxx.sh
[root@node2 ~] ansible web -m script -a '/root/xxx.sh' 
```



#### copy模块相关

```shell
backup 备份文件，以时间戳结尾
dest 目的地址
group 属组
mode 文件权限
owner 文件属主
src  源文件
[root@node2 ~] ansible web -m copy -a 'src=/root/m.sh dest=/temp/a.sh' # 将本地文件的m.sh拷贝到web组下的a.sh文件

[root@node2 ~] ansible web -m copy -a 'src=/root/m.sh dest=/temp/a.sh mode=755' # 将本地文件的m.sh拷贝到web组下的a.sh文件,添加权限

[root@node2 ~] ansible web -m copy -a 'src=/root/xxx/ dest=/temp/' # 将本地文件的xxx目录所有文件复制到远端
```



### file模块

```shell
file模块主要用于远程主机上的文件操作。
（group、mode、owner):定义文件/目录

path:定义文件路径
recurse:递归的设置，只对目录有效。
src:要被链接的源文件路径，只应用于state=link的情况
dest:被连接的路径，只应用于state=link的情况
state:directory：如果目录不存在，创建目录
file:即使文件不存在，也不会创建
link：创建软连接
hard：创建硬链接
touch：文件不存在，则会创建。如果存在则会，则更新最后修改的时间。
absent:删除目录、文件或取消链接。

[root@test ~]ansible db -m file -a "src=/etc/fstab dest=/tmp/fstab state=directory"  # 在远程主机上创建文件夹

[root@test ~]ansible db -m file -a "src=/etc/q.txt dest=/tmp/fstab state=touch"  # 在远程主机上创建文件，如果本地q.txt不存在就创建空文件
```



[更多模块使用](https://www.cnblogs.com/sxchengchen/p/7765921.html)

