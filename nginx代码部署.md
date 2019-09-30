### 1.DNS域名解析流程

- 当浏览器访问域名的时候

```python
1.取本地hosts文件中寻找是否有对应的域名和ip的记录
2.取本地的dns缓存中寻找是否有记录
3.取dns域名服务器中寻找记录
4.如果dns服务器中找到了记录
5.操作系统会将dns记录,缓存到本地,便于下次加速解析
```

- linux的域名解析命令

```python
nslookup pythonav.cn  
# Server:	192.168.108.2
# Address:	192.168.108.2#53

vim /etc/resolv.conf  修改linux的dns配置文件   可以写两个有主有备
#search localdomain
#nameserver  223.5.5.5
#nameserver  223.6.6.6 
```

### 2. linux之间互传

```python
scp 远程传输命令
语法
scp  你想传输的内容   想要发送到哪里 
scp  /tmp/test.txt pengpeng@ip:/tmp/    
scp  pengpeng@ip:tmp/支付宝密码.txt  /tmp/
```

### 3.linux安装python环境搭建

- python3在linux下的编译过程

1. 首先解决环境依赖问题,如gcc编译工具等  得先保证yum源配置好

- 配置步骤

  - 打开阿里云开源镜像站的官网https://opsx.alibaba.com/mirro

  - 获取cengtos的yum源

    ```python
    #yum源的工作目录,/etc/yum.repos.d目录下,只要在这个目录下名字叫做repo的文件,都会被yum取读取
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
    ```

  - 获取epel的yum源(第三方软件仓库,如nginx,redis等等)

    ```python
    wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
    ```

2. 解决编译python3的基础环境依赖

   ```python
   yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y
   ```

3. 下载python3源码包

   ```python
   wget https://www.python.org/ftp/python/3.6.7/Python-3.6.7.tar.xz
   ```

4. 解压缩

   ```python
   xz -d Python-3.6.7.tar.xz 
   tar -xvf Python-3.6.7.tar 
   ```

5. 进入py源码包  开始编译三部曲

   ```python
   #1.第一曲 [执行configure脚本文件,指定安装路径]  ,释放makefile编译文件 ,让gcc工具去编译
   ./configure --prefix=/opt/s21/python367/  
   
   #2.第二曲 ,指定make指令,读取makefile,开始编译
   	
   #3.第三曲,执行make install ,开始安装python3,这一步会生成python3解释器 
   make && make install
   ```

6. 编译完成之后,配置path环境变量,让系统可以补全python3的命令

   ```python
   #进入 刚刚第一步曲的python环境
   cd /opt/s21/python367
   cd bin 
   pwd  #查看当前工作目录  并复制
   
   #获取当前环境变量
   echo $PATH
   /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
                   
   #添加python3的环境变量,注意,要添加到开头
   #注意要写入到全局变量配置文件中,每次开机都加载/etc/profile中
   vim /etc/profile# 到最低行,加入如下配置
   PATH='/opt/s21/python367/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin'
   
   #注意,修改完毕/etc/profile 必须 source读取一下
   source /etc/profile 
   ```

7. 安装虚拟环境,管理python的解释器

   1. 安装虚拟环境工具,装在物理解释器下

   ```python
   pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple virtualenv
   ```

   2. 通过命令创建虚拟环境

   ```python
   virtualenv --no-site-packages --python=python3  django1  虚拟环境的名字
   --no-site-packages 	 创建干净隔离的虚拟环境,没有任何模块	
   --python=python3   #指定以哪个解释器去分身 
   ```

   3. 激活虚拟环境，进入虚拟环境,无论是否激活python虚拟环境,影响的只是python相关的东西,和操作系统无关

   ```python
   source s21Django1/bin/activate  #激活
   deactivate  #退出
   ```

   4. 在虚拟环境下启动crm项目

   ```python
   -上传crm代码到linux服务器
   ```

   5. 激活虚拟环境 安装django 1.11.1

   ```python
   source s21Django1/bin/activate
   pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple django==1.11.1
   		pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pymysql 
   		pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple django-multiselectfield
   ```

   6. 安装mariadb(mysql)

   ```python
   yum install mariadb-server mariadb -y
   ```

   7. 启动mariad,通过yum安装的软件,都可以用systemctl管理

   ```python
   systemctl start mariadb
   netstat -tunlp  | grep 3306 #查看是否启动成功
   
   python3 manage.py runserver 192.168.108.130：8000 #启动django
   ```

   8. 解决完毕问题之后,启动python项目,注意防火墙,ALLOW_HOSTS相关的修改

   ```python
   iptables -F
   
   #同时注意 修改 settings中配置 ALLOWED_HOSTS=['*']
   ```

   9. 可以退出虚拟环境了
   10. mysql数据从windows迁移到linux
   
   ```python
   #到windows迁出
   mysqldump -uroot -p123 -r c:data.txt(文件名) -B crm(库名)
   #使用xftp传入centos
   #进入linux的数据库
   mysql -uroot -p
   source /opt/s21/data.txt(路径)
   
   #尽量使用xftp 防止乱码
   ```

### 4.nginx学习

#### 4.1 基本安装

- nginx是个web服务器,常用作静态文件服务器,反向代理服务器,邮件代理服务器,负载均衡服务器 

1. 安装淘宝nginx,编代码编译安装,先解决模块依赖

   ```python
   yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel openssl openssl-devel -y
   ```

2. 获取淘宝nginx的源代码

   ```python
   wget http://tengine.taobao.org/download/tengine-2.3.2.tar.gz
   ```

3. 解压缩源代码包

   ```python
   tar -zxvf tengine-2.3.2.tar.gz
   ```

4. 进入源代码目录开始编译三部曲

   1. 指定安装路径

   ```python
   ./configure --prefix=/opt/s21/tngx
   ```

   2. 编译且安装

   ```python
   make && make install 
   ```

   3. 安装完成之后,进入nginx的目录,

   ```python
   [root@wupeiqi tngx]#pwd
   /opt/s21/tngx
   [root@wupeiqi tngx]#ls
   conf  html  logs  sbin
   
   conf 存放配置文件 , 指定了这个软件各种功能的一个文件而已  
   html 存放前端页面
   logs nginx的运行日志
   sbin  nginx的可执行命令目录
   ```

   4. 进入sbin目录,启动nginx

   ```python
   ./nginx  
   ./nginx -s stop 停止nginx
   ./nginx -t  检查nginx.conf的语法是否正确
   ./nginx -s reload  不重启nginx,重新加载nginx配置
   ```

#### 4.2 nginx的核心学习

1. 找到nginx.conf,学习语法

   ```python
   #这里的所有配置是nginx的核心功能
   http {
   
   ....
   }
   ```

2. nginx的访问日志功能

   ```python
   http {
       include       mime.types;
       default_type  application/octet-stream;
   	#日志格式化
       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
   
       access_log  logs/access.log  main;                                               
   	...
   	
   }
   ```

3. nginx的虚拟主机配置,核心功能再次

   ```python
   http {
   	#nginx支持多虚拟主机,只需要写入多个server关键字即可
   	#虚拟主机1
   	
   	    server {
   			#基于端口的虚拟主机区分 
   			listen       80;
   			#基于域名的虚拟主机区分
   			server_name  www.old21.com;
   			#charset koi8-r;
   			#access_log  logs/host.access.log  main;
   			#access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;
   			
   			#这里是nginx的url匹配,如同django的url规则一样
   			#当我的请求时 http://192.168.182.130:81/chouhuo.jpg  这样的时候,就进入如下location匹配
   			#这个是最低级的匹配,所有请求都会走到这里
   			location / {
   				#root关键字定义虚拟主机的根目录, 这里是可以修改的
   				root   /opt/alex/;
   				#必须保证首页文件存在
   				index  index.html index.htm;
   			}
   		}
   
   	#虚拟主机2 
   	server {
   			listen       80;
   			server_name  www.old22.com;
   			#charset koi8-r;
   			#access_log  logs/host.access.log  main;
   			#access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;
   			
   			#这里是nginx的url匹配,如同django的url规则一样
   			#当我的请求时 http://192.168.182.130/wupeiqi.jpg  这样的时候,就进入如下location匹配
   			#这个是最低级的匹配,所有请求都会走到这里
   			location / {
   				#root关键字定义虚拟主机的根目录, 这里是可以修改的
   				root   /opt/wupeiqi/;
   				#index参数是定义首页文件的名字的
   				index  index.html index.htm;
   			}
   	
   	}
   
   }
   ```

4. nginx的错误页面 404优化

   ```python
   server {
           listen 80;
           server_name www.old666.com;   
   #通过这个参数定义即可,		
           error_page  404              /404.html;
           location / {
                   root   /opt/wupeiqi;
                   index index.html;
           }
   
   }
   ```

5. nginx反向代理

   ```python
   代理:
   
   用户,客户端    中介,代理服务器,   房东,资源服务器
   
   租房的客户  ->  中介,代理  ->   房东 
   
   浏览器 -> nginx  ->  django 
   ```

   - 反向代理服务器配置如下“

   ```python
   1.打开192.168.182.130 机器的nginx.conf,修改为如下
   找到server{}虚拟主机,修改location如下
       server {
   		listen       80;
   		server_name  www.oldchouhuo.com;
   		#charset koi8-r;
   		#access_log  logs/host.access.log  main;
   		#access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;
   
   		location / {
   			#    root   /opt/alex/;
   			#   index  index.html index.htm;
   			
   			#实现反向代理的功能参数
   			#实现反向代理的功能参数
   			#实现反向代理的功能参数
   			proxy_pass http://192.168.182.131;                                                         
   		}
   	}
   ```

#### 4.3 nginx负载均衡

1. 准备好2台资源服务器,本应该提供一样的数据,进行负载均衡,实验目的,看到不同的页面,所以准备不同的页面数据
   - 192.168.182.131  资源服务器1  ,返回alex的页面 
   - 192.168.182.132  资源服务器2 ,返回武大郎的页面

2. 准备负载均衡服务器,配置如下

   ```python
   #在nginx配置文件中,添加如下配置,定义负载均衡池,写入后端项目地址
   	#默认轮询方式
   	upstream mys21django  {
   		server 192.168.182.131;
   		server 192.168.182.132;                                                                    
   	}
   	
   	#权重方式
   		upstream mys21django  {
   		server 192.168.182.131	weight=4;
   		server 192.168.182.132   weight=1;                                                                    
   	}
   
   	#ip哈希方式,根据用户的来源ip计算出哈希值,永远只指派给一个服务器去解析
   	#ip哈希不得与权重共同使用 
   	#ip哈希不得与权重共同使用 
   		upstream mys21django  {
   			server 192.168.182.131	;
   			server 192.168.182.132   ;         
   			ip_hash;
   	}
   	
   	
   	
   	#虚拟主机配置如下
       server {
           listen       80;
           server_name  www.oldchouhuo.com;
           #charset koi8-r;
           #access_log  logs/host.access.log  main;
           #access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;
           location / {
           #    root   /opt/alex/;
            #   index  index.html index.htm;
   		#请求转发给负载均衡池
           proxy_pass http://mys21django;
           }
   }
   ```

### 5.项目部署 uwsgi+nginx+crm

- nginx + uwsgi + django + mysql

1. 后端搞起

   1. 上传crm项目到linux服务器
   2. 安装uwsgi命令,这是python的一个模块

   ```python
   pip3 install -i https://pypi.douban.com/simple  uwsgi 
   ```

   3. 激活一个虚拟环境去使用

   ```python
   source s21Django1/bin/activate  #激活
   virtualenv --no-site-packages --python=python3   s21uwsgi   #这是创建虚拟环境
   ```

2. 使用uwsgi的命令,参数形式启动  crm项目

   1. 以往的python3 manage.py runserver 调用wsgiref去启动django,性能很低,单进程web。使用uwsgi启动django,可以支持并发,多进程,以及日志设置,多种功能。

   - 必须在django项目,第一层敲这个命令

   ```python
   uwsgi --http :8000 --module mysite.wsgi
       --http 指定是http协议,去启动项目
       --module  指定django目录下的wsgi文件
   #这样加载的 无法加载静态文件
   ```

   - uwsgi支持的热加载命令

   ```python
   uwsgi --http :9000 --module Aida_crm.wsgi   --py-autoreload=1 
   ```

3. uwsgi以配置文件的形式启动 ,就是把你的启动参数,写入到一个文件中,然后,执行这个文件即可

   - 配置文件名字可以叫做 uwsgi.ini ,内容如下,这个文件是手动生成的
     touch uwsgi.ini ,写入如下内容

   ```python
   [uwsgi]
   # Django-related settings
   # the base directory (full path)
   #填入项目的绝对路径 ,项目的第一层路径
   chdir           = /opt/s21/Aida_crm
   
   # Django's wsgi file
   #指定第二层项目下的wsgi文件
   module          = Aida_crm.wsgi
   
   # the virtualenv (full path)
   #找到虚拟环境的绝对路径
   home            = /opt/s21/s21uwsgi
   
   
   # process-related settings
   # master
   master          = true
   
   # 以cpu核数来填写,uwsgi的工作进程数
   processes       = 2
   
   # the socket (use the full path to be safe
   #这是以uwsgi_socket协议启动的项目,无法再去通过浏览器访问,必须通过nginx以uwsgi协议去反向代理
   socket          = 0.0.0.0:8000
       
   #也可以使用http协议去启动(仅用作调试使用)
   #http = 0.0.0.0:9000
   # ... with appropriate permissions - may be needed
   # chmod-socket    = 664
   # clear environment on exit
   vacuum          = true
   
   #后台运行参数,将uwsgi运行在后台,并且将django日志输出到uwsgi.log中
   daemonize = uwsgi.log 
   
   ```

4. 指定配置文件启动django

   ```python
   uwsgi --ini  uwsgi.ini   #到这里 配置文件生效 项目已经启动 因为使用了socket 所以必须使用nginx反向代理
   ```

   1. nginx的配置  反向代理uwsgi
      - 修改nginx.conf如下（vim /opt/s21/tngx/conf/nginx.conf）

   ```python
   
   server {
           listen       80;
           server_name  www.oldchouhuo.com;
           #charset koi8-r;
           #access_log  logs/host.access.log  main;
           #access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;
           location / {
           #转发请求的方式配置在这里
           #转发请求的方式配置在这里
           #转发请求的方式配置在这里
   			include    uwsgi_params;
   			uwsgi_pass 0.0.0.0:8000;
           }
   		}  
   ```

   2. 重新加载nginx,访问nginx,查看是否反向代理

   ```python
   去浏览器访问nginx的地址,查看能否访问到crm的内容
   #注意： 去将要访问的电脑修改host host文件在
   c:windows/system32/driver/etc/hosts 
   添加  192.168.108.130  域名（www.oldchouhuo.com）
   ```

   3. 收集crm的所有静态文件,丢给nginx去解析

   ```python
   #对django的settings.py配置修改如下
   #添加如下参数
   STATIC_ROOT='/opt/s21static'                                                                                                                        
   
   STATIC_URL = '/static/'
   STATICFILES_DIRS = [
       os.path.join(BASE_DIR, 'static')
   ]
   
   执行命令,收集django的所有静态文件,系统会自动创建'/opt/s21static'  这个文件夹
   python3 manage.py collectstatic
   ```

   4. 配置nginx,找到crm的这些静态资源

   ```python
   location / {
   				include    uwsgi_params;
   				uwsgi_pass 0.0.0.0:8000;
   			}
           #添加一个location,针对nginx的url进行匹配处理
           #当请求时 www.oldchouhuo.com/static/.....  这样的url的时候,nginx进行别名修改,去/opt/s21static底下去寻找资源文件                                                                                                       
           location  /static {
               alias /opt/s21static;
           }
   ```

### 6. 前后端分离项目部署

- 后端部署 ：uwsgi + drf + redis + mysql 

1. 准备后端代码

   ```python
   wget https://files.cnblogs.com/files/pyyu/luffy_boy.zip
   ```

3. 创建且激活新的虚拟环境

   ```python
   virtualenv --no-site-packages --python=python3   s21luffy
   ```

4. 解决模块依赖问题,尝试调试启动drf后台

   ```python
   certifi==2018.11.29
   chardet==3.0.4
   crypto==1.4.1
   Django==2.1.4
   django-redis==4.10.0
   django-rest-framework==0.1.0
   djangorestframework==3.9.0
   idna==2.8
   Naked==0.1.31
   pycrypto==2.6.1
   pytz==2018.7
   PyYAML==3.13
   redis==3.0.1
   requests==2.21.0
   shellescape==3.4.1
   urllib3==1.24.1
   uWSGI==2.0.17.1
   	
   pip3 install -i https://pypi.douban.com/simple -r requirements.txt 
   ```

5. 使用uwsgi，启动drf后台

   ```python
   touch uwsgi.ini 
   
   [uwsgi]
   chdir           = /opt/s21vue+drf/luffy_boy  #项目一层目录
   module          = luffy_boy.wsgi #项目二层目录
   home            = /opt/s21vue+drf/s21luffy #项目环境
   master          = true
   processes       = 2
   socket          = 0.0.0.0:8888  #必须跟nginx调用接口相同
   vacuum          = true
   daemonize = uwsgi.log 
   ```

- 前端部署

1. 下载前端vue代码

   ```python
   wget https://files.cnblogs.com/files/pyyu/07-luffy_project_01.zip
   ```

2. 逻辑

   ```python
   vue+nginx的端口 81端口,这是第一个虚拟主机 先访问vue 然后通过vue访问 端口 访问后端
   nginx反向代理端口,    9500 ,  这是第二个虚拟主机
   drf的后台端口  8888 
   ```

3. 访问步骤

   ```python
   第一步: 192.168.182.130:81  查看到路飞的首页内容,是vue的静态页面
   第二部:   当我点击  课程列表的时候,vue向后台发送请求了,发送的地址应该是 192.168.182.130:9500
   第三部:此时  nginx的反向代理,转发请求给了 drf的后台 8888
   ```

4. 修改vue请求地址,向服务器的ip地址发送请求

   ```python
   sed -i  "s/127.0.0.1:8000/192.168.182.130:9500/g"  src/restful/api.js 
   参数解释:
     i 将替换结果写入到文件
     "s/你想替换的内容/替换之后的内容/g"  s是替换模式  g是全局替换 
   ```

5. 配置nodejs的解释器环境,打包编译vue代码,生成静态文件夹dist

   ```python
   #这里的node代码包,是二进制包,已经编译好了,可以直接使用
   wget https://nodejs.org/download/release/v8.6.0/node-v8.6.0-linux-x64.tar.gz
   #解压缩node的代码包
   tar -zxvf node-v8.6.0-linux-x64.tar.gz 
   #配置PATH环境变量
   PATH='/opt/s21/python367/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/opt/s21/tngx/sbin:/opt/s21vue+drf/node-v8.6.0-linux-x64/bin' 
   #检查node是否可用
   [root@wupeiqi bin]#node -v
   v8.6.0
   [root@wupeiqi bin]#npm -v
   5.3.0
   ```

6. 进入vue代码目录,开始编译代码,生成dist静态文件夹

   ```python
   cd /opt/s21vue+drf/07-luffy_project_01/
   #开始安装这个项目所有需要的node模块,默认去读取 package.json
   npm install 
   #开始编译vue代码,生成dist静态网页文件夹,丢给nginx 
   npm run build 
   ```

7. 在正确生成dist之后,配置nginx 

   ```python
   #nginx配置如下
   #这里配置和luffy学成有关的代码
   #这个是nginx+vue的虚拟主机
   server {
           listen 81;
           server_name localhost;
           error_page  404              /404.html;
           #请求来到这里时,返回vue的页面
           location / {
                   root   /opt/s21vue+drf/07-luffy_project_01/dist;
                   index index.html;
   				try_files $uri $uri/ /index.html;
           }
   }
   #这个是nginx反向代理,转发vue请求给drf的虚拟主机
   	server {
   		listen 9500;
   		server_name localhost;
   		location / {
   		include    uwsgi_params; #在nginx/conf下
   		uwsgi_pass 0.0.0.0:8888;
   		}
   
   	}    
   ```

8. 部署redis

   ```python
   # 这个前后端分离项目过程中有redis
   yum install  redis -y
   systemctl start redis
   ```

9. 此时访问vue地址即可,看到路飞页面,且可以看到课程列表的数据,可以添加linux,和python的课程,到购物车中,整个部署过程结束