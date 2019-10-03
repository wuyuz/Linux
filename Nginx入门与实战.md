## Nginx入门与实战



#### 网站服务

![1570085837386](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570085837386.png)

​	想必我们大多数人都是通过访问网站而开始接触互联网的吧。我们平时访问的网站服务 就是 Web 网络服务，一般是指允许用户通过浏览器访问到互联网中各种资源的服务。

Web 网络服务是一种被动访问的服务程序，即只有接收到互联网中其他主机发出的 请求后才会响应，最终用于提供服务程序的 Web 服务器会通过 HTTP(超文本传输协议)或 HTTPS(安全超文本传输协议)把请求的内容传送给用户。

目前能够提供 Web 网络服务的程序有 IIS、Nginx 和 Apache 等。其中，IIS(Internet Information Services，互联网信息服务)是 Windows 系统中默认的 Web 服务程序

2004 年 10 月 4 日，为俄罗斯知名门户站点而开发的 Web 服务程序 Nginx 横空出世。 Nginx 程序作为一款轻量级的网站服务软件，因其稳定性和丰富的功能而快速占领服务器市 场，但 Nginx 最被认可的还当是系统资源消耗低且并发能力强，因此得到了国内诸如新浪、 网易、腾讯等门户站的青睐。



#### web服务器和web框架的关系

web服务器（nginx）：接收HTTP请求（例如www.pythonav.cn/xiaocang.jpg）并返回数据

web框架（django，flask）：开发web应用程序，处理接收到的数据



#### Nginx是什么

```
nginx是一个开源的，支持高性能，高并发的www服务和代理服务软件。它是一个俄罗斯人lgor sysoev开发的，作者将源代码开源出来供全球使用。
nginx比它大哥apache性能改进许多，nginx占用的系统资源更少，支持更高的并发连接，有更高的访问效率。
nginx不但是一个优秀的web服务软件，还可以作为反向代理，负载均衡，以及缓存服务使用。
安装更为简单，方便，灵活。nginx可以说是非常nb了

特点;
    支持高并发，能支持几万并发连接
    资源消耗少，在3万并发连接下开启10个nginx线程消耗的内存不到200M
    可以做http反向代理和负载均衡
    支持异步网络i/o事件模型epoll
```

​	Tengine是由淘宝网发起的Web服务器项目。它在Nginx的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如淘宝网，天猫商城等得到了很好的检验。它的最终目标是打造一个高效、稳定、安全、易用的Web平台。



#### 安装环境

- 准备工作

  ```
  一. gcc 安装
  安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：
  
  yum install gcc-c++
  二. PCRE pcre-devel 安装
  PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：
  
  yum install -y pcre pcre-devel
  三. zlib 安装
  zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。
  
  yum install -y zlib zlib-devel
  四. OpenSSL 安装
  OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
  nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
  yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel openssl openssl-devel -y
  ```

- 安装和启动

  ```
  1.下载源码包
  wget -c https://nginx.org/download/nginx-1.12.0.tar.gz
  2.解压缩源码
  tar -zxvf nginx-1.12.0.tar.gz
  3.配置，编译安装  开启nginx状态监测功能
  ./configure --prefix=/opt/nginx1-12/ --with-http_ssl_module --with-http_stub_status_module 
  make && make install 
  4.启动nginx，进入sbin目录,找到nginx启动命令
  cd sbin
  ./nginx #启动
  ./nginx -s stop #关闭
  ./nginx -s reload #重新加载
  
  netstat -tunlp |grep 80
  curl -I 127.0.0.1
  #如果访问不了，检查selinux，iptables
  ```

  nginx默认站点是Nginx目录下的html文件夹，这里可以从nginx.conf中查到

  ```
   location /{
              root   html;  #这里是默认的站点html文件夹，也就是 /opt/nginx1-12/html/文件夹下的内容，要该文件路劲需要index.html文件
              index  index.html index.htm; #站点首页文件名是index.html
          }
  ```

  如果要部署网站业务数据，只需要把开发好的程序全放到html目录下即可

  ```
  [root@oldboy_python /tmp 11:34:52]#ls /opt/nginx1-12/html/
  index.html  jssts.jpeg  lhy.mp4  man.jpg  wget-log
  ```

  因此只需要通过域名/资源，即可访问

  ```
  http://www.pyyuc.cn/man.jpg
  ```

- Nginx目录结构

  ```
  [root@oldboy_python /opt/nginx1-12 11:44:02]#ls
  client_body_temp  conf  fastcgi_temp  html  logs  proxy_temp  sbin  scgi_temp  static  uwsgi_temp
  
  conf 存放nginx所有配置文件的目录,主要nginx.conf
  html 存放nginx默认站点的目录，如index.html、error.html等
  logs 存放nginx默认日志的目录，如error.log access.log
  sbin 存放nginx主命令的目录,sbin/nginx
  ```

  

#### Nginx主配置文件解析

Nginx主配置文件`/etc/nginx/nginx.conf`是一个纯文本类型的文件，整个配置文件是以区块的形式组织的。一般，每个区块以一对大括号`{}`来表示开始与结束。

```
######Nginx配置文件nginx.conf中文详解#####

#定义Nginx运行的用户和用户组
user www www;

#nginx进程数，建议设置为等于CPU总核心数。
worker_processes 8;
 
#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
error_log /usr/local/nginx/logs/error.log info;

#进程pid文件
pid /usr/local/nginx/logs/nginx.pid;

#指定进程可以打开的最大描述符：数目
#工作模式与连接数上限
#这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
#现在在linux 2.6内核下开启文件打开数为65535，worker_rlimit_nofile就相应应该填写65535。
#这是因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
worker_rlimit_nofile 65535;


events
{
    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型
    #是Linux 2.6以上版本内核中的高性能网络I/O模型，linux建议epoll，如果跑在FreeBSD上面，就用kqueue模型。
    #补充说明：
    #与apache相类，nginx针对不同的操作系统，有不同的事件模型
    #A）标准事件模型
    #Select、poll属于标准事件模型，如果当前系统不存在更有效的方法，nginx会选择select或poll
    #B）高效事件模型
    #Kqueue：使用于FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X.使用双处理器的MacOS X系统使用kqueue可能会造成内核崩溃。
    #Epoll：使用于Linux内核2.6版本及以后的系统。
    #/dev/poll：使用于Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+。
    #Eventport：使用于Solaris 10。 为了防止出现内核崩溃的问题， 有必要安装安全补丁。
    use epoll;

    #单个进程最大连接数（最大连接数=连接数*进程数）
    #根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为。
    worker_connections 65535;

    #keepalive超时时间。
    keepalive_timeout 60;

    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。
    #分页大小可以用命令getconf PAGESIZE 取得。
    #[root@web001 ~]# getconf PAGESIZE
    #4096
    #但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。
    client_header_buffer_size 4k;

    #这个将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
    open_file_cache max=65535 inactive=60s;

    #这个是指多长时间检查一次缓存的有效信息。
    #语法:open_file_cache_valid time 默认值:open_file_cache_valid 60 使用字段:http, server, location 这个指令指定了何时需要检查open_file_cache中缓存项目的有效信息.
    open_file_cache_valid 80s;

    #open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。
    #语法:open_file_cache_min_uses number 默认值:open_file_cache_min_uses 1 使用字段:http, server, location  这个指令指定了在open_file_cache指令无效的参数中一定的时间范围内可以使用的最小文件数,如果使用更大的值,文件描述符在cache中总是打开状态.
    open_file_cache_min_uses 1;
    
    #语法:open_file_cache_errors on | off 默认值:open_file_cache_errors off 使用字段:http, server, location 这个指令指定是否在搜索一个文件是记录cache错误.
    open_file_cache_errors on;
}
 
 
 
#设定http服务器，利用它的反向代理功能提供负载均衡支持
http
{
    #文件扩展名与文件类型映射表
    include mime.types;

    #默认文件类型
    default_type application/octet-stream;

    #默认编码
    #charset utf-8;

    #服务器名字的hash表大小
    #保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.
    server_names_hash_bucket_size 128;

    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    client_header_buffer_size 32k;

    #客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
    large_client_header_buffers 4 64k;

    #设定通过nginx上传文件的大小
    client_max_body_size 8m;

    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    #sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile on;

    #开启目录列表访问，合适下载服务器，默认关闭。
    autoindex on;

    #此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    tcp_nopush on;
     
    tcp_nodelay on;

    #长连接超时时间，单位是秒
    keepalive_timeout 120;

    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k;    #最小压缩文件大小
    gzip_buffers 4 16k;    #压缩缓冲区
    gzip_http_version 1.0;    #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2;    #压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml;    #压缩类型，默认就已经包含textml，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;

    #开启限制IP连接数的时候需要使用
    #limit_zone crawler $binary_remote_addr 10m;



    #负载均衡配置
    upstream jh.w3cschool.cn {
     
        #upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
        server 192.168.80.121:80 weight=3;
        server 192.168.80.122:80 weight=2;
        server 192.168.80.123:80 weight=3;

        #nginx的upstream目前支持4种方式的分配
        #1、轮询（默认）
        #每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
        #2、weight
        #指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
        #例如：
        #upstream bakend {
        #    server 192.168.0.14 weight=10;
        #    server 192.168.0.15 weight=10;
        #}
        #2、ip_hash
        #每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
        #例如：
        #upstream bakend {
        #    ip_hash;
        #    server 192.168.0.14:88;
        #    server 192.168.0.15:80;
        #}
        #3、fair（第三方）
        #按后端服务器的响应时间来分配请求，响应时间短的优先分配。
        #upstream backend {
        #    server server1;
        #    server server2;
        #    fair;
        #}
        #4、url_hash（第三方）
        #按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
        #例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
        #upstream backend {
        #    server squid1:3128;
        #    server squid2:3128;
        #    hash $request_uri;
        #    hash_method crc32;
        #}

        #tips:
        #upstream bakend{#定义负载均衡设备的Ip及设备状态}{
        #    ip_hash;
        #    server 127.0.0.1:9090 down;
        #    server 127.0.0.1:8080 weight=2;
        #    server 127.0.0.1:6060;
        #    server 127.0.0.1:7070 backup;
        #}
        #在需要使用负载均衡的server中增加 proxy_pass http://bakend/;

        #每个设备的状态设置为:
        #1.down表示单前的server暂时不参与负载
        #2.weight为weight越大，负载的权重就越大。
        #3.max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
        #4.fail_timeout:max_fails次失败后，暂停的时间。
        #5.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

        #nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
        #client_body_in_file_only设置为On 可以讲client post过来的数据记录到文件中用来做debug
        #client_body_temp_path设置记录文件的目录 可以设置最多3层目录
        #location对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
    }
     
     
     
    #虚拟主机的配置
    server
    {
        #监听端口
        listen 80;

        #域名可以有多个，用空格隔开
        server_name www.w3cschool.cn w3cschool.cn;
        index index.html index.htm index.php;
        root /data/www/w3cschool;

        #对******进行负载均衡
        location ~ .*.(php|php5)?$
        {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }
         
        #图片缓存时间设置
        location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires 10d;
        }
         
        #JS和CSS缓存时间设置
        location ~ .*.(js|css)?$
        {
            expires 1h;
        }
         
        #日志格式设定
        #$remote_addr与$http_x_forwarded_for用以记录客户端的ip地址；
        #$remote_user：用来记录客户端用户名称；
        #$time_local： 用来记录访问时间与时区；
        #$request： 用来记录请求的url与http协议；
        #$status： 用来记录请求状态；成功是200，
        #$body_bytes_sent ：记录发送给客户端文件主体内容大小；
        #$http_referer：用来记录从那个页面链接访问过来的；
        #$http_user_agent：记录客户浏览器的相关信息；
        #通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
        log_format access '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';
         
        #定义本虚拟主机的访问日志
        access_log  /usr/local/nginx/logs/host.access.log  main;
        access_log  /usr/local/nginx/logs/host.access.404.log  log404;
         
        #对 "/" 启用反向代理
        location / {
            proxy_pass http://127.0.0.1:88;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
             
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             
            #以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;

            #允许客户端请求的最大单文件字节数
            client_max_body_size 10m;

            #缓冲区代理缓冲用户端请求的最大字节数，
            #如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
            #无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
            client_body_buffer_size 128k;

            #表示使nginx阻止HTTP应答代码为400或者更高的应答。
            proxy_intercept_errors on;

            #后端服务器连接的超时时间_发起握手等候响应超时时间
            #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_connect_timeout 90;

            #后端服务器数据回传时间(代理发送超时)
            #后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
            proxy_send_timeout 90;

            #连接成功后，后端服务器响应时间(代理接收超时)
            #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
            proxy_read_timeout 90;

            #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            #设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
            proxy_buffer_size 4k;

            #proxy_buffers缓冲区，网页平均在32k以下的设置
            #设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
            proxy_buffers 4 32k;

            #高负荷下缓冲大小（proxy_buffers*2）
            proxy_busy_buffers_size 64k;

            #设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
            proxy_temp_file_write_size 64k;
        }
         
         
        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status on;
            access_log on;
            auth_basic "NginxStatus";
            auth_basic_user_file confpasswd;
            #htpasswd文件的内容可以用apache提供的htpasswd工具来产生。
        }
         
        #本地动静分离反向代理配置
        #所有jsp的页面均交由tomcat或resin处理
        location ~ .(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8080;
        }
         
        #所有静态文件由nginx直接读取不经过tomcat或resin
        location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|
        pdf|xls|mp3|wma)$
        {
            expires 15d; 
        }
         
        location ~ .*.(js|css)?$
        {
            expires 1h;
        }
    }
}
```

拆分解析

- CoreModule核心模块

  ```
  user www;                       #Nginx进程所使用的用户
  worker_processes 1;             #Nginx运行的work进程数量(建议与CPU数量一致或auto)
  error_log /log/nginx/error.log  #Nginx错误日志存放路径
  pid /var/run/nginx.pid          #Nginx服务运行后产生的pid进程号
  ```

- events模块

  ```
  events {            
      worker_connections  //每个worker进程支持的最大连接数
      use epool;          //事件驱动模型, epoll默认
  }
  ```

- http内核模块

  ```
  //公共的配置定义在http{}
  http {  //http层开始
  ...    
      //使用Server配置网站, 每个Server{}代表一个网站(简称虚拟主机)
      'server' {
          listen       80;        //监听端口, 默认80
          server_name  localhost; //提供服务的域名或主机名
          access_log host.access.log  //访问日志
          //控制网站访问路径
          'location' / {
              root   /usr/share/nginx/html;   //存放网站代码路径
              index  index.html index.htm;    //服务器返回的默认页面文件
          }
          //指定错误代码, 统一定义错误页面, 错误代码重定向到新的Locaiton
          error_page   500 502 503 504  /50x.html;
      }
      ...
      //第二个虚拟主机配置
      'server' {
      ...
      }
      
      include /etc/nginx/conf.d/*.conf;  //包含/etc/nginx/conf.d/目录下所有以.conf结尾的文件
  
  }   //http层结束
  ```

  

#### 虚拟主机

![1570086810590](C:\Users\wanglixing\Desktop\知识点复习\Linux\1570086810590.png)

​	如果每台linux服务器只运行了一个小网站，那么人气低，流量小的草根站长需要承担高额的服务器租赁费，也造成了硬件资源浪费。虚拟主机就是将一台服务器分割成多个“虚拟服务器”，每个站点使用各自的硬盘空间，由于省资源，省钱，众多网站都使用虚拟主机来部署网站。

```
虚拟主机的概念就是在web服务里的一个独立的网站站点，这个站点对应独立的域名（IP），具有独立的程序和资源目录，可以独立的对外提供服务。
这个独立的站点配置是在nginx.conf中使用server{}代码块标签来表示一个虚拟主机。
Nginx支持多个server{}标签，即支持多个虚拟主机站点。

虚拟主机类型：
    基于域名的虚拟主机
    通过不同的域名区分不同的虚拟主机，是企业应用最广的虚拟主机。

    基于端口的虚拟主机
    通过不同的端口来区分不同的虚拟主机，一般用作企业内部网站，不对外直接提供服务的后台，例如www.pythonav.cn:9000

    基于IP的虚拟主机
    通过不同的IP区分不同的虚拟主机，此类比较少见，一般业务需要多IP的常见都会在负载均衡中绑定VIP
```



#### Nginx状态信息配置

平滑重启nginx

```
./sbin/nginx -s reload
```

![1570086994637](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570086994637.png)

#### 案例演示

nginx可以自动识别用户请求的域名，根据不同的域名请求服务器传输不同的内容，只需要保证服务器上有一个可用的ip地址，配置好dns解析服务。

/etc/hosts是linux系统中本地dns解析的配置文件，同样可以达到域名访问效果，修改nginx.conf

```
[root@oldboy_python ~ 14:33:16]#egrep -v '#|^$' /opt/nginx1-12/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.pyyuc.cn;
        location /{
            root   html/pyyuc;
            index  index.html index.htm;
        }
    }
}

[root@oldboy_python /opt/nginx1-12/html 14:36:08]#mkdir pyyuc
[root@oldboy_python /opt/nginx1-12/html 14:36:18]#echo "<meta charset=utf8>我是pyyuc站点" > pyyuc/index.html
[root@oldboy_python /opt/nginx1-12/html 14:37:21]#cat pyyuc/index.html
<meta charset=utf8>我是pyyuc站点

[root@oldboy_python /opt/nginx1-12/html 14:37:28]#../sbin/nginx -t
nginx: the configuration file /opt/nginx1-12/conf/nginx.conf syntax is ok

#平滑重启nginx
[root@oldboy_python /opt/nginx1-12/html 14:39:18]#../sbin/nginx -s reload
```

上述作用创建了一个html/pyyuc站点目录，对应于虚拟主机配置文件里的root根目录的设置html/pyyuc，然后生成一个首页文件index.html，内容是“我是pyyuc站点”

- 配置多个域名

  ```
  egrep -v '#|^$' /opt/nginx1-12/conf/nginx.conf
  worker_processes  1;
  events {
      worker_connections  1024;
  }
  http {
      include       mime.types;
      default_type  application/octet-stream;
      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
      access_log  logs/access.log  main;
      sendfile        on;
      keepalive_timeout  65;
      server {
          listen       80;
          server_name  www.pyyuc.cn;
          location /{
              root   html/pyyuc;
              index  index.html index.htm;
          }
  }
      server {
          listen       80;
          server_name  www.pythonav.cn;
          location /{
              root   html/pythonav;
              index  index.html index.htm;
          }
  }
  }
  
  创建pythonav虚拟主机站点的目录和文件
  [root@oldboy_python /opt/nginx1-12 14:47:21]#mkdir -p /opt/nginx1-12/html/pythonav
  [root@oldboy_python /opt/nginx1-12 14:49:33]#echo "<meta charset=utf8>我是pythonav，未成年禁止入内"> /opt/nginx1-12/html/pythonav/index.html
  [root@oldboy_python /opt/nginx1-12 14:50:44]#./sbin/nginx -t
  nginx: the configuration file /opt/nginx1-12/conf/nginx.conf syntax is ok
  nginx: configuration file /opt/nginx1-12/conf/nginx.conf test is successful
  [root@oldboy_python /opt/nginx1-12 14:51:32]#./sbin/nginx -s reload
  ```



#### Nginx访问日志（access_log)

日志功能对每个用户访问网站的日志信息记录到指定的日志文件里，开发运维人员可以分析用户的浏览器行为，此功能由ngx_http_log_module模块负责，官网地址是：

http://nginx.org/en/docs/http/ngx_http_log_module.html

控制日志的参数

```
log_format    记录日志的格式，可定义多种格式
accsss_log    指定日志文件的路径以及格式

　　log_format main '$remote_addr - $remote_user [$time_local] "$request" '
　　'$status $body_bytes_sent "$http_referer" '
　　'"$http_user_agent" "$http_x_forwarded_for"';
　　
$remote_addr    记录客户端ip
$remote_user    远程用户，没有就是 “-”
$time_local 　　 对应[14/Aug/2018:18:46:52 +0800]
$request　　　 　对应请求信息"GET /favicon.ico HTTP/1.1"
$status　　　  　状态码
$body_bytes_sent　　571字节 请求体的大小
$http_referer　　对应“-”　　由于是直接输入浏览器就是 -
$http_user_agent　　客户端身份信息
$http_x_forwarded_for　　记录客户端的来源真实ip 97.64.34.118

nginx.conf默认配置
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;
```



#### Nginx限制网站来源ip访问

如果哪天发现你的nginx很慢，或者检查access.log时候，有一个some body疯狂请求你的nginx server，那么可以禁止这个IP访问

```
限制ip或ip段访问
禁止访问/av/底下的资源

location /av {
deny 122.71.240.254;
#alias /opt/nginx1-12/html/av;
allow 10.1.1.0/16;  
}
```

#### Nginx错误页面优化

在网站运行过程中，可能因为页面不存在等原因，导致网站无法正常响应请求，此时web服务会返回系统的错误码，但是默认的错误页面很不友好。因此我们可以将404，403等页面的错误信息重定向到网站首页或者其他指定的页面，提升用户访问体验。

```
server {
        listen       80;
        server_name  www.pythonav.cn;
        root html/pythonav;
        location /{
            index  index.html index.htm;
        }
　　　　　#在pythonav路径下的40x.html错误页面
        error_page 400 403 404 405 /40x.html;
        }
```



#### Nginx代理

![1570087504743](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570087504743.png)

- 正向代理

  正向代理，也就是传说中的代理,他的工作原理就像一个跳板（VPN），简单的说：我是一个用户，我访问不了某网站，但是我能访问一个代理服务器，这个代理服务器呢，他能访问那个我不能访问的网站，于是我先连上代理服务器，告诉他我需要那个无法访问网站的内容，代理服务器去取回来，然后返回给我。

- 反向代理

  对于客户端而言，代理服务器就像是原始服务器。反向代理就是服务器通过代理把东西给客户端

  ![1570088008851](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570088008851.png)

  nginx实现负载均衡的组件

  ```
  ngx_http_proxy_module    proxy代理模块，用于把请求抛给服务器节点或者upstream服务器池
  ```

- 实现简单的反向代理

  ```
  机器准备，两台服务器
  master  192.168.11.63　　主负载
  slave   192.168.11.64　　web1
  
  主负载均衡节点的配置文件：
  worker_processes  1;
  error_log  logs/error.log;
  pid        logs/nginx.pid;
  events {
      worker_connections  1024;
  }
  http {
      include       mime.types;
      default_type  application/octet-stream;
      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
      access_log  logs/access.log  main;
      sendfile        on;
      keepalive_timeout  65;
          upstream slave_pools{
      server 192.168.11.64:80 weight=1;
  }
      server {
          listen       80;
          server_name  localhost;
          location / {
          proxy_pass  http://slave_pools;
              root   html;
              index  index.html index.htm;
          }
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
          }
      }
  }
  
  #启动nginx
  [root@master 192.168.11.63 /opt/nginx1-12]$/opt/nginx1-12/sbin/nginx
  #检查端口
  [root@master 192.168.11.63 /opt/nginx1-12]$netstat -tunlp|grep nginx
  tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      8921/nginx: master
  ```

  此时访问master的服务器192.168.11.63:80地址，已经会将请求转发给slave的80端口

  除了页面效果的展示以外，还可以通过log(access.log)查看代理效果



#### Location配置学习

- 正则表达式

  ```
  匹配符 匹配规则 优先级
  =    精确匹配    1
  ^~    以某个字符串开头    2
  ~    区分大小写的正则匹配    3
  ~*    不区分大小写的正则匹配    4
  !~    区分大小写不匹配的正则    5
  !~*    不区分大小写不匹配的正则    6
  /    通用匹配，任何请求都会匹配到    7
  ```

  

#### nginx.conf配置文件实例

```
server {
    listen 80;
    server_name pythonav.cn;

    #优先级1,精确匹配，根路径
    location =/ {
        return 400;
    }

    #优先级2,以某个字符串开头,以av开头的，优先匹配这里，区分大小写
    location ^~ /av {
       root /data/av/;
    }

    #优先级3，区分大小写的正则匹配，匹配/media*****路径
    location ~ /media {
          alias /data/static/;
    }

    #优先级4 ，不区分大小写的正则匹配，所有的****.jpg|gif|png 都走这里
    location ~* .*\.(jpg|gif|png|js|css)$ {
       root  /data/av/;
        }

    #优先7，通用匹配
    location / {
        return 403;
    }
}
```



- nginx语法之root和alias区别实战

  ```
  nginx指定文件路径有root和alias两种方法
  区别在方法和作用域：
  
  方法：
  
  root
  语法  root  路径;
  默认值 root   html;
  配置块  http{}   server {}   location{}
  
  alias
  语法： alias  路径
  配置块  location{}
  
  
  root和alias区别在nginx如何解释location后面的url，这会使得两者分别以不同的方式讲请求映射到服务器文件上
  
  root参数是root路径+location位置
  
  root实例：
  
      location ^~ /av {
          root /data/av;   注意这里可有可无结尾的   /
      }
  请求url是pythonav.cn/av/index.html时
  web服务器会返回服务器上的/data/av/av/index.html
  
  root实例2：
  location ~* .*\.(jpg|gif|png|js|css)$ {
         root  /data/av/;
  }
  
  请求url是pythonav.cn/girl.gif时
  web服务器会返回服务器上的/data/static/girl.gif
  
  
  
  
  alias实例：
  alias参数是使用alias路径替换location路径
  alias是一个目录的别名
  注意alias必须有 "/"  结束！
  alias只能位于location块中
  
  
  请求url是pythonav.cn/av/index.html时
  web服务器会返回服务器上的/data/static/index.html
  
  location ^~ /av {
      alias /data/static/;
  }
  ```



#### Nginx负载均衡

```
Web服务器，直接面向用户，往往要承载大量并发请求，单台服务器难以负荷，我使用多台WEB服务器组成集群，前端使用Nginx负载均衡，将请求分散的打到我们的后端服务器集群中，实现负载的分发。那么会大大提升系统的吞吐率、请求性能、高容灾

Nginx要实现负载均衡需要用到proxy_pass代理模块配置
Nginx负载均衡与Nginx代理不同地方在于
Nginx代理仅代理一台服务器，而Nginx负载均衡则是将客户端请求代理转发至一组upstream虚拟服务池
Nginx可以配置代理多台服务器，当一台服务器宕机之后，仍能保持系统可用。
```

![1570088489675](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570088489675.png)

#### upstream配置

在nginx.conf > http 区域中

```
upstream django {
       server 10.0.0.10:8000;
       server 10.0.0.11:9000;
}
```

在nginx.conf > http 区域 >  server区域  > location配置中

添加proxy_pass

```
location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://django;
}
```

此时初步负载均衡已经完成，upstream默认按照轮训方式负载，每个请求按时间顺序逐一分配到后端节点。



#### upstream分配策略

weight 权重

```
upstream django {
       server 10.0.0.10:8000 weight=5;
       server 10.0.0.11:9000 weight=10;#这个节点访问比率是大于8000的
}
```

ip_hash

```
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器
upstream django {　　　　ip_hash;
       server 10.0.0.10:8000;
       server 10.0.0.11:9000;
}
```

**backup**

**在非backup机器繁忙或者宕机时，请求backup机器，因此机器默认压力最小**

```
upstream django {
       server 10.0.0.10:8000 weight=5;
       server 10.0.0.11:9000;
       server node.oldboy.com:8080 backup;
}
```



#### 负载均衡实验环境规划

```
角色            ip                    主机名
lb01        192.168.119.10        lb01    
web01        192.168.119.11        web01
web02        192.168.119.12        web02
```

- 关闭防火墙

  ```
  iptables -F
  sed  -i 's/enforcing/disabled/' /etc/selinux/config
  
  systemctl stop firewalld
  systemctl disable firewalld
  ```

- web02服务器配置nginx，创建index.html

  ```
  server {
          listen       80;
          server_name  192.168.119.11;
          location / {
              root /node;
              index  index.html index.htm;
              }
  }
  
  mkdir /node
  echo 'i am web01' > /node/index.html
  
  #启动NGINX
  ./sbgin/nginx
  ```

- web01服务器配置nginx,创建index.html

  ```
  server {
      listen       80;
      server_name  192.168.119.12;
      location / {
          root /node;
          index  index.html index.htm;
  }
  
  mkdir /node
  echo 'i am web02...' > /node/index.html
  #启动nginx
  ./sbing/nginx
  ```

- 配置lb01服务器的nginx负载均衡

  ```
  http {
      include       mime.types;
      default_type  application/octet-stream;
      sendfile        on;
      keepalive_timeout  65;
      upstream node {
      　　server 192.168.119.11:80;
      　　server 192.168.119.12:80;
  }
      server {
          listen       80;
          server_name 192.168.119.10;
          location / {
          　　proxy_pass http://node;
          　　include proxy_params;  #需要手动创建
          }
      }
  }
  
  手动创建proxy_params文件，文件中存放代理的请求头相关参数
  [root@lb01 conf]# cat /opt/nginx/conf/proxy_params
  proxy_set_header Host $http_host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  
  proxy_connect_timeout 30;
  proxy_send_timeout 60;
  proxy_read_timeout 60;
  
  proxy_buffering on;
  proxy_buffer_size 32k;
  proxy_buffers 4 128k;
  
  启动lb01负载均衡nginx服务
  ./sbin/nginx
  ```

  

#### 部署VUE + uwsgi + nginx 路飞项目

```
路飞学城django代码
https://files.cnblogs.com/files/pyyu/luffy_boy.zip
vue代码
https://files.cnblogs.com/files/pyyu/07-luffy_project_01.zip
```

- 将代码搞到服务器上

  ```
  在linux上直接下载
  wget https://files.cnblogs.com/files/pyyu/luffy_boy.zip
  wget https://files.cnblogs.com/files/pyyu/07-luffy_project_01.zip
  ```

- 先从前端vue搞起

  ```
  要在服务器上，编译打包vue项目，必须得有node环境
  
  下载node二进制包，此包已经包含node，不需要再编译
  wget https://nodejs.org/download/release/v8.6.0/node-v8.6.0-linux-x64.tar.gz
  解压缩
  tar -zxvf node-v8.6.0-linux-x64.tar.gz
  进入node文件夹
  [root@web02 opt]# cd node-v8.6.0-linux-x64/
  [root@web02 node-v8.6.0-linux-x64]# ls
  bin  CHANGELOG.md  etc  include  lib  LICENSE  README.md  share
  [root@web02 node-v8.6.0-linux-x64]# ls bin
  node  npm  npx
  [root@web02 node-v8.6.0-linux-x64]# ./bin/node -v
  v8.6.0
  [root@web02 node-v8.6.0-linux-x64]# ./bin/npm -v
  5.3.0
  
  ----------------------------------------------
  将node命令，添加至linux环境变量,修改/etc/profile，写入
  
  PATH=$PATH:/opt/node-v8.6.0-linux-x64/bin
  读取文件，生效PATH
  
  source /etc/profile
  测试path
  
  [root@web02 node-v8.6.0-linux-x64]# node -v
  v8.6.0
  [root@web02 node-v8.6.0-linux-x64]# npm -v
  5.3.0
  ```

- node环境有了，安装node模块，以及打包node项目

  ```
  进入vue源码目录
  cd 07-luffy_project_01/
  安装vue模块，默认去装package.json的模块内容，如果出现模块安装失败，手动再装
  npm install 
  
  此时注意，你本地写的vue代码，接口很可能连接的服务器地址有问题，注意Axios.POST提交的地址，一定得发送给django应用（如果用了nginx，就发送给nginx的入口端口）
  超哥这里为了试验方便，将vue项目和django项目放在了一台服务器，通过nginx反向代理功能(8000端口)，转发vue请求给django(9000)
  
  准备编译打包vue项目，替换配置文件所有地址，改为服务器地址
  sed -i 's/127.0.0.1/192.168.119.12/g' /opt/07-luffy_project_01/src/restful/api.js
  
  确保vue的route模式是history
  路径：opt/luffy/07-luffy_project_01/src/router/index.js
  export default new Router({
  linkActiveClass:'is-active',
  mode: 'history',//改成history模式
  
  
  此时打包vue项目，生成一个dist静态文件夹
  npm run build
  
  检查dist文件夹
  [root@web02 07-luffy_project_01]# ls dist/
  index.html static
  ```

  至此vue代码就结束了，只需要让nginx配置，找到vue的index.html首页文件即可；nginx这里不做解释，编译安装好即可

  ```
  server {
  　　　　 #用户访问域名或者ip，默认是nginx的80端口
  
          listen       80;
          server_name  192.168.119.12;
  　　　　 #url匹配  /   也就是请求地址是192.168.119.12时，进入此location，返回vue的dist下index.html路飞学城首页
          location / {
          root /opt/07-luffy_project_01/dist;
          index index.html;
          }
      }
  ```

- 配置后端代码，解决虚拟环境，保证项目干净隔离

  ```
  激活虚拟环境venv1，在虚拟环境下，安装路飞项目所需的依赖模块
  [root@web02 opt]# cat requirements.txt
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
  
  这个路飞代码数据库用的是sqllite，不需要配置数据库了，购物车用都的是redis，因此要启动服务器的redis-server服务端
  redis-server /etc/redis.conf
  
  ps -ef|grep redis
  redis-server *:6379
  ```

- 通过uwsgi启动路飞项目

  ```
  编写ini文件
  [uwsgi]
  # Django-related settings
  # the base directory (full path)
  chdir           = /opt/luffy_boy
  # Django's wsgi file
  module          = luffy_boy.wsgi
  # the virtualenv (full path)
  home            = /opt/venv1
  # process-related settings
  # master
  master          = true
  # maximum number of worker processes
  processes       = 1
  # the socket (use the full path to be safe
  socket          = 0.0.0.0:9000
  # clear environment on exit
  vacuum          = true
  #后台运行uwsgi
  daemonize=yes
  
  (venv1) [root@web02 opt]# uwsgi --ini luffy_boy/uwsgi.ini
  ```

- 配置nginx

  ```
  worker_processes  1;
  
  events {
      worker_connections  1024;
  }
  
  
  http {
      include       mime.types;
      default_type  application/octet-stream;
      sendfile        on;
      keepalive_timeout  65;
      server {
          listen       80;
          server_name  192.168.119.12;
          location / {
          root /opt/07-luffy_project_01/dist;
          index index.html;
  　　　　#这一条参数确保vue页面刷新时候，不会出现404页面
  　　　　  try_files $uri $uri/ /index.html;
          }
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
             root   html;
          }
      }
  
  server {
  listen       8000;
          server_name  192.168.119.12;
          location / {
          uwsgi_pass 0.0.0.0:9000;
          include /opt/nginx/conf/uwsgi_params;
          }
          location /static {
          alias /opt/static;
  }
  }
  }
  ```

- 原理图

  ![1570089375351](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570089375351.png)