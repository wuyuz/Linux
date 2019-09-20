## Linux 基础命令五



#### 查找文件

##### find 实时查找工具，通过便利指定路径完成文件查找

##### 语法：

```
find [OPTION]... [查找路径] [查找条件] [处理动作]
查找路径：指定具体目录路径；默认为当前目录
查找条件：指定的查找标准，可以为文件名、大小、类型、权限等，默认为查找指定路径下的所有文件
处理动作：对符合条件的文件做操作，默认输出值屏幕
```

##### 查找路径：可以指定具体的路径，默认是当前路径

```shell
[root@MyHost ~]# find /etc/ -name virc   # 去指定路径下查找文件，默认是所在路径.
/etc/virc

[root@MyHost ~]# find -name a   #将所有的有a查出来，包括目录下的
./a
./test/a
```



##### 查找条件：用来指定文件查找的标准，可以是大小、文件名、权限、类型

- ##### -name 文件名/目录名

  ```shell
   # 查找条件：文件名，显示目录下的所有满足条件的文件，包括文件下的
  [root@MyHost ~]# find -name 2   # 完全匹配
  ./2
  [root@MyHost ~]# cd test
  [root@MyHost test]# touch 2
  [root@MyHost test]# cd
  [root@MyHost ~]# find -name 2
  ./test/2
  ./2
  
  [root@MyHost ~]# find -name \*2    # 正则
  ./test/2
  ./2
  [root@MyHost ~]# find -name \?2.txt
  ./12.txt
  [root@MyHost ~]# find -name f\*
  ./f4
  ./.cache/pip/http/f/e/d/0/
  
  [root@MyHost ~]# find -name "f?"    # 使用"正则规则"来替代\
  ./f4
  ./f1
  
  [root@MyHost ~]# find -name "f[15]"  # 任选其一
  ./f1
  ./f5
  
  find -name a 完全匹配
  find -name "a*" 所有的以a开头的文件或者文件夹
  find -name "a?" 所有以a开头后面为一个字母的文件或者文件夹
  find -name "a[ab]" 以a开头后面是a或者b的文件或者文件夹
  ```

- ##### -iname 忽略大小写

  ```shell
  [root@MyHost ~]# find -iname A 
  ./a
  ./test/a
  ```

- ##### 按照搜索层级：

  - -madepth level 指定最大的搜索层数，指定的目录为第一层

    ```shell
    [root@MyHost ~]# mkdir -p h/a k/a j/b/a
    [root@MyHost ~]# touch a
    [root@MyHost ~]# find -maxdepth 2 -name a
    ./a
    ./k/a
    ./h/a    # 第三层的j/b/a没有查找到
    ```

  - -mixdepth level 指定最小搜索层数

    ```shell
    [root@MyHost ~]# find -mindepth 3 -name a  # 第三层到正无穷
    ./j/b/a
    ```

    

- ##### -type 指定文件类型

  ```shell
  [root@MyHost ~]# find -name a    # 查找所有a文件名的文件和文件名
  ./a
  ./test/a
  [root@MyHost ~]# find -type f -name a    # 满足类型为文件
  ./a
  [root@MyHost ~]# find -type d -name a
  ./test/a
  
  [root@MyHost test]# touch a   # 创建文件和目录误区
  [root@MyHost test]# ll
  -rw-r--r-- 1 root root    0 Aug  2 08:48 2
  drwxr-xr-x 2 root root 4096 Aug  2 09:07 a    #目录a存在，所以文件a无法创造
  drwxr-xr-x 7 root root 4096 Aug  1 08:45 mnt
  [root@MyHost test]# mkdir 2
  mkdir: cannot create directory ‘2’: File exists    # 文件存在，无法创建同名目录
  
  [root@MyHost ~]# find /run -type s   #列出/run下 类型为s的文件
  
  
  - f 文件
  - d 目录
  - l 链接，链接文件的权限为777 
  - s socket套接字
  - b 块设备
  - c 字符设备文件
  - p 管道文件
  ```

- ##### -empty 空文件和空目录

  ```shell
  [root@MyHost ~]# find -empty   # 查找空文件或空目录
  ./.ssh/authorized_keys
  ./.ssh/.bash_history
  
  [root@MyHost ~]# find -empty -type f  # 查找空的文件
  
  [root@MyHost ~]# find -empty -type d -delete  # 查找出空目录并删除
  ```

- ##### 根据属组、属组来搜索

  - -user username 查找属主是username的文件或者文件夹

    ```shell
    [root@MyHost wang]# chown wang a.py   # 转换为wang属主
    [root@MyHost wang]# ll
    -rw-r--r-- 1 wang root   57 Jul 31 22:22 a.py
    
    [wang@MyHost ~]$ find -user wang   # 在wang用户中查看
    .
    ./a.py
    
    [root@MyHost wang]# chown :wang a.py
    [wang@MyHost ~]$ find -user wang
    .
    ./a.py
    ```

  - -group groupname 查找属组是groupname的文件或者文件夹

  - -uid uid 查找用户id为uid的文件或者文件夹

    ```shell
    [root@MyHost wang]# id        # 查看当前用户id
    uid=0(root) gid=0(root) groups=0(root)
    [root@MyHost wang]# id wang
    uid=1001(wang) gid=1001(wang) groups=1001(wang)
    
    [root@MyHost wang]# find -uid 1001
    .
    ./a.py
    
    [root@MyHost wang]# find -gid 1001
    .
    ./a.py
    ```

  - -gid gid 查找用户组id为gid 的文件或者文件夹

  - -nouser 查找没有属主的文件或者文件夹(uid 为数字)

    ```shell
    [li@MyHost tmp]$ cd test/     # tmp是所有用户都可以创建东西的场所
    [li@MyHost test]$ touch a.txt
    [li@MyHost test]$ ll
    -rw-rw-r-- 1 li li 0 Aug  2 19:39 a.txt   # 使用li创建文件
    
    [root@MyHost li]# userdel li    #删除用户 
    [root@MyHost test]# ll
    -rw-rw-r-- 1 1002 1002 0 Aug  2 19:39 a.txt   # 用户没有了但是id在
    
    [root@MyHost test]# find . -nouser 
    .
    ./a.txt
    
    [root@MyHost test]# find . -nogroup
    
    [root@MyHost test]# useradd xing   #此时我们重新创建了用户，此时它将继承li的遗产，使用1002id的文件
    [root@MyHost test]# ll
    total 0
    -rw-rw-r-- 1 xing xing 0 Aug  2 19:39 a.txt
    ```

  - -nogroup 查找没有属组的文件或者文件夹（gid为数字）

- ##### 组合条件

  - 与  -a

  - 或  -o

    ```shell
    [root@MyHost test]# find -type f -o -nouser  # 查找类型是f的或者没有属主的
    ./a.txt
    ```

  - 非 -not/!

  - 摩根定律
    - (非A) 或 (非B) = 非(A且B)
    
      ```shell
      [root@MyHost test]# find -not -user wusir -a -not -user xiaofeng -ls  # 查看非晓峰和非wusir的文件，并展示出来
      
      [root@MyHost test]# find -not \( -user wusir -o -user xiaofeng  \) -ls |wc -l  # 合并并统计多少行
      
       #找出/tmp目录下，属主不是root，且文件名不以f开头的文件 
      find /tmp \( -not -user root -a -not -name 'f*' \) -ls
      find /tmp -not \( -user root -o -name 'f*' \) –ls
      ```
    
    - (非A) 且  (非B) = 非(A或B)

- ##### 排除目录

  - -path 

    ```shell
      #查找/etc/下，除/etc/sane.d目录的其它所有.conf后缀的文件
    find /etc -path ‘/etc/sane.d’ -a –prune -o -name “*.conf” 
    
      #查找/etc/下，除/etc/sane.d和/etc/fonts两个目录的所有.conf后缀的文件
    find /etc \( -path "/etc/sane.d" -o -path "/etc/fonts" \) -a -prune -o - name "*.conf"
    ```

    

- ##### 文件大小

  - -size[+|-]  #unit 常用单位：k,M,G,c(byte)
  
  - \#unit :(#-1,#] 不包括前面，包括后面（左开右闭）
  
    ```shell
    [root@MyHost test]# dd if=/dev/zero of=ccc bs=1M count=1 #脚本生成1M文件
    1+0 records in  
    1+0 records out   
    1048576 bytes (1.0 MB) copied, 0.00138471 s, 757 MB/s
    [root@MyHost test]# find -size 1M  #小于等于1M，大于0M
    . 
    ./ccc  
    ```
  
  - -#unit: [0,#-1] 0-#-1  （从0-数字-1）
  
    ```shell
    [root@MyHost test]# find -size -2M
    ./a.txt
    ```
  
  - +#unit:(#,….) #到无穷 
  
    ```shell
    [root@MyHost test]# find -size +2M -type f  # 大于2M
    ```
  
- ##### 根据时间戳：

  - 以“天”为单位

    - -atime [+|-]#      根据最后一次访问时间进行查找

      - \#:[#,#+1)

        ```shell
        [root@MyHost test]# find -atime 1 -ls  #过去的一天到两天之间，闭
        [root@MyHost test]# find -atime 0 -ls
        393249    4 drwxrwxr-x   2 xing     xing         4096 Aug  2 19:59 .
        393256 1024 -rw-r--r--   1 root     root      1048576 Aug  2 19:59 ./ccc
        ```

      - +#:[#+1,….]

        ```shell
        [root@MyHost ~]# find  -atime +1 -ls
        132538    4 -rw-r--r--   1 root     root           42 Jul 30 23:43 ./.oracle_jre_usage/5531418b80d1c901.timestamp
        132525    4 -rw-r--r--   1 root     root          101 Aug 18  2017 ./.pip/pip.conf
        ```

      - -#:[0,#)

        ```shell
        [root@MyHost ~]# find  -atime -1 -ls  #今天到过去#天访问的
        131073 4 dr-xr-x--- 7 root  root  4096 Aug  2 19:19 .
        132537 4 drwxr-xr-x 2 root  roo   4096 Aug 23  2017 ./.oracle_jre_usage
        ```

    - -mtime  # 文件的修改时间

    - -ctime     #  创建时间，用户和上类似

  - 以“分钟为单位“

    - -amin
    - -mmin
    - -cmin

- 根据权限

  - -perm[/|-] mode

    ```shell
    [root@MyHost ~]# find -perm 644 -ls   #根据权限搜索
    
    find -perm 755 会匹配权限模式恰好是755的文件
        • 只要当任意人有写权限时，find -perm /222就会匹配
        • 只有当每个人都有写权限时，find -perm -222才会匹配
        • 只有当其它人(other)有写权限时，find -perm -002才会匹配
    ```

    - mode 精确权限匹配

    - /mode 任何一类对象权限只要有以为匹配即可

    - -mode 每一类必须都同事拥有指定权限

      

##### 处理动作：对符合条件的文件进行操作，默认是直接输出到屏幕上

- -print 默认的处理动作，显示之屏幕

- -ls 类似于查找到的文件执行“ls -l”命令

  ```shell
  [root@MyHost ~]# find -name "f?" -ls
  132565    4 -rw-r--r--   1 root   root   16 Aug  1 00:25 ./f4
  ```

- -delete 删除查找到的文件

  ```shell
  [root@MyHost ~]# find -name "f?" -delete
  ```

- -fls file 查找的所有文件的长格式信息保存至指定的文件中

  ```shell
  [root@MyHost ~]# find -perm 644 -fls a1.txt  #将找到的文件信息打印到a1中
  [root@MyHost ~]# cat a1.txt 
  ```

- -ok command {} \;对查找到的每个文件执行command指定的命令，对于每个文件执行命令之前，都会交互式要求用户确认（ok)；{}表示我们匹配到的文件

  ```shell
  [root@MyHost ~]# find -perm 777 -ok rm -rf {} \;  #删除aaa链接文件
  < rm ... ./aaa > ? y  
  ```

- -exec command {} \; 对查找到的每个文件执行command命令

  - {} 用于引用查找到的文件名称本身

  - find 传递查找到的文件至后面指定的命令时，查找到所有符合条件的文件一次性传递给后面的命令

    ```shell
    [root@MyHost ~]# find -perm 777 -exec rm -rf {} \; # 直接删除
    ```

    

#### 参数替换xargs

- 由于很多命令不支持管道来传递参数，而日常工作中有这个必要，所以就有了xargs命令

  ```shell
  [root@MyHost ~]# echo a{1..100}|touch
  touch: missing file operand
  
  [root@MyHost test2]# echo a{1..1000}|xargs touch  # 创建1000个文件
  ....
  -rw-r--r-- 1 root root 0 Aug  2 20:41 a988
  -rw-r--r-- 1 root root 0 Aug  2 20:41 a989
  
  [root@MyHost test2]# echo a{1..1000}|xargs rm
    #以上是将echo输出的参数，交给rm后面当参数，将1000个删除
    
  [root@MyHost ~]# find -name "1?.txt"|xargs rm
  [root@MyHost ~]# ls a*|xargs rm -f
  ```

- xargs 用于产生某个命令参数，xargs可以读入stdin的数据，并且以空格符合或者回车符号作为stdin的数据分隔符

- 有些命令不能接受过多参数，命令执行可能会失败，xargs可以借鉴

  ```SHELL
  ls f* |xargs rm
  find /sbin -perm +700 |ls -l 这个命令是错误的
  find /sbin -perm +7000 | xargs ls –l 查找特殊权限的文件
  ```

- find和xargs格式:find | xargs COMMAND



#### grep：Linux文本处理三剑客之一（sed、awk、grep）

##### grep ：Global search REgular expression and Print out the line（全局正则搜索并打印符合条件的行）

​	作用：文本搜索工具，根据用户指定的"模式"对目标文件逐行进行匹配检查并打印出匹配的行

​	模式： 由正则表达式字符及文本字符所编写的的过滤条件

​	格式： grep [options] pattern [file...]

​	其中PATTERN项需要使用''或者"",如果需要对模式进行转换，则需要使用""，如果不需要进行转换，则使用''或""都可以。模式还可以使用正则表达式来表示。

- ##### grep的基本使用

  ```shell
  [root@MyHost ~]# grep 'root' passwd   #查找passwd文件下的root的行
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/root:/sbin/nologin
  
  ```

- ##### 常用参数

  ```shell
  --color=auto 对匹配到的文本进行颜色显示
  -v 取反
  [root@MyHost ~]# grep -v 'root' passwd    # 显示没有匹配到的行
  
  -i 忽略大小写
  [root@MyHost ~]# grep -i 'ROOt' passwd   # 匹配忽略大小写
  
  -n 显示匹配到的行号
  [root@MyHost ~]# grep -in 'ROOt' passwd   # 显示行数
  
  -c 统计匹配到的行数
  [root@MyHost ~]# grep -ico 'ROOt' passwd    # 匹配到多少行
  2
  
  -o 仅显示匹配到的字符串
  [root@MyHost ~]# grep -ino 'ROOt' passwd 
  1:root
  1:root
  1:root
  
  -q 静默模式，不输出任何信息
  [root@MyHost ~]# grep -q 'ROOt' passwd 
  [root@MyHost ~]# echo $?   # $? 表示上一个命令执行的状态，1表示失败，0表示成功，状态码0-255
  1
  
  -A # after，后#行
  [root@MyHost ~]# grep -nA 2 'root' passwd 
  1:root:x:0:0:root:/root:/bin/bash  #后两个
  2-bin:x:1:1:bin:/bin:/sbin/nologin
  3-daemon:x:2:2:daemon:/sbin:/sbin/nologin
  --
  10:operator:x:11:0:operator:/root:/sbin/nologin  #后两个
  11-games:x:12:100:games:/usr/games:/sbin/nologin
  12-ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
  
  -B # before，前#行
  -C # context 前后各#行
  -e 实现多个选项的逻辑or关系
  [root@MyHost ~]# grep -e 'root' -e 'mail' passwd   # 同时查找多个条件
  root:x:0:0:root:/root:/bin/bash
  mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
  
  -E 扩展的正则表达式
  -r  连带文件夹以下目录也查找
  [root@MyHost ~]# grep -r 'a' test2   # 查找当前文件夹下的test目录下的所有含有a的字符
  test2/b.txt:adfadfadf
  
  -w 匹配整个单词
  ```

  

#### 正则表达式

- 字符匹配

  - . 匹配任意单个字符

  - [abc] 匹配执行范围内的任意单个字符  [0-9]

  - [^abc] 取反

  - [:alnum:] 数字大小写 [a-zA-Z0-9] 

    ```shell
    [root@MyHost ~]# grep "[[:alnum:]]+" b.txt 
    [root@MyHost ~]# grep "[[:alnum:]]*" b.txt 
    123
    dsaaf
    3reawr
    123drwer
    [root@MyHost ~]# grep "\w*" b.txt 
    123
    dsaaf
    3reawr
    123drwer
    
    [root@MyHost ~]# grep "a*.f" b.txt 
    dsaaf
    
    [root@MyHost ~]# grep "[[:punct:]]" b.txt   #匹配标点
    ,
    ```

  - [:alpha:] 大小写字母 [a-zA-z]

  - [:lower:] 小写字母 [a-z]

  - [:upper:] 大写字母 [A-Z]

  - [:digit:] 数字 [0-9]

  - [:punct:] 标点符号

- 匹配次数

  - \* 0次或者多次，是贪婪匹配

  - \？0次或者一次

  - \\+ 最少一次

  - \\{n\\} 匹配n次

  - \\{m,n\\} 最少m次，最多n次

  - \\{,n\\} 最多n次

    ```shell
    [root@MyHost ~]# grep ".*a\{,1\}f" b.txt   #正则
    dsaaf
    ```

  - \\{m,\\} 最少m次

- 位置锚定

  - ^ 行首锚定
  - $ 结尾
  - ^$ 空行

- 向后引用

  - \1:表示前面第一个括号内匹配之后产生的字符，在\1的位置要在出现一次

  - \2:表示引用前面的第二个左括号与之对应的右括号中的模式所匹配到的内容

    ```shell
    [root@MyHost ~]# cat c.txt 
    like is like
    love is love
    like is not love    # 从此文件中取出前两行
    
    [root@MyHost ~]# grep "\(l..e\).*\1" c.txt 
    like is like
    love is love
    ```



#### egrep

egrep =grep -E

支持扩展正则表达式，与标准增长表达式的区别就是不需要转义

```shell
[root@MyHost ~]# grep -E "(l..e).*\1" c.txt 
like is like
love is love
```



#### 压缩

##### gzip

**语法**：gzip [option] ….file…

```shell
[root@MyHost ~]# ll
-rw-r--r-- 1 root root  1237 Aug  2 21:03 passwd

[root@MyHost ~]# gzip passwd
[root@MyHost ~]# ll
-rw-r--r-- 1 root root   541 Aug  2 21:03 passwd.gz   #之前的passwd自动被删除

 # 解压
[root@MyHost ~]# gunzip passwd.gz 
```

- -d 解压缩， 相当于gunzip

  ```shell
  [root@MyHost ~]# gzip -d passwd.gz 
  ```

- -c 结果输出之标准输出，保留原来文件，默认是不保留源文件

  ```shell
  [root@MyHost ~]# gzip -c passss.txt   # 这样写会乱码，需要指定输出地址
  
  [root@MyHost ~]# gzip -c passss.txt > pass.txt
  
  [root@MyHost ~]# gzip -c -d passwd.gz > passwd  #解压并保存源文件
  ```

- -# 1-9，指定压缩比，值越大压缩比例越大

  ```shell
  [root@MyHost ~]# gzip -9 passwd
  gzip: passwd.gz already exists; do you wish to overwrite (y or n)? y
  ```

- zcat 不显式加压缩的前提下查看文本文件内容

  ```shell
  gzip -c messages >messages.gz
  gzip -c -d messages.gz > messages 
  zcat messages.gz > messages   #查看时另存为
  cat messages | gzip > m.gz
  
  [root@MyHost ~]# zcat passwd.gz   # 查看压缩文件在不打开的前提下
  ```



#### bzip2/bunzip2/bzact

**语法：**bzip2  [option]…file...

```shell
[root@MyHost ~]# yum install bzip2
[root@MyHost ~]# bzip2 pass.txt 
[root@MyHost ~]# ll
-rw-r--r-- 1 root root   137 Aug  2 21:59 pass.txt.bz2

[root@MyHost ~]# bzip2 -d pass.txt.bz2   # 解压后自动删除源压缩文件
```

- -k 保留原文件

  ```shell
  [root@MyHost ~]# bzip2 -k b.txt   # 保留源文件
  ```

- -d 解压缩

- -# 1-9 压缩比，默认是9

  ```shell
  [root@MyHost ~]# bzip2 -9 b.txt    # 设置压缩比
  ```

- bzcat 不显式加压缩的前提下查看文本文件内容

  ```shell
  [root@MyHost ~]# bzcat b.txt.bz
  ```



#### xz/unxz/xzcat

**语法**：xz [option]…file...

```shell
[root@MyHost ~]# xz c.txt    # 压缩
[root@MyHost ~]# unxz c.txt.xz   #解压
```

- -k 保留原文件

  ```shell
  [root@MyHost ~]# xz -k c.txt 
  [root@MyHost ~]# xz -d -k c.txt.xz 
  ```

- -d 解压缩

  ```shell
  [root@MyHost ~]# xz -d c.txt.xz 
  ```

- -# 1-9 压缩比，默认是6

- xzcat 不显式加压缩的前提下查看文本文件内容



#### tar 工具

##### 归档工具

**语法**：tar [option]… file

```shell
c  #创建归档
v  #显示过程
f  #指定文件
r  #追加
x  #解档
-C  #指定解压位置

--------以上是归档，使用下面的选项来进行压缩------
j  #使用bzip2来压缩
z  #使用gzip来压缩
J  #使用xz来压缩
--exculde=     #排除

[root@MyHost ~]# tar cvf xxx.tar 2 b.bz2     # 将文件2 b.bz2文件归档为xxx.tar

[root@MyHost ~]# tar -r -f xxx.tar  error.log  # 向归档文件追加新的文件error.log

[root@MyHost ~]# rm -f 2 xxx b.txt error.log   # 先在本地删除相应的文件
[root@MyHost ~]# tar xf xxx.tar   #解压

[root@MyHost ~]# tar xf xxx.tar -C test   # 用C指定解压到的位置

```

- ##### tar指定bzip2压缩

  ```shell
  [root@MyHost test]# tar jcvf w.tar.bz 2 e  # j使用bzip2来压缩
  [root@MyHost test]# ll
  -rw-r--r-- 1 root root 192 Aug  2 22:34 w.tar.bz
  
  [root@MyHost test]# tar jxvf w.tar.bz   # 解压
  ```

- ##### tar指定gzip压缩

  ```shell
  [root@MyHost test]# tar zcvf xxxx.tar.gz 2.txt e.txt
  [root@MyHost test]# ll
  -rw-r--r-- 1 root root 182 Aug  2 22:38 xxxx.tar.gz
  [root@MyHost test]# 
  
  [root@MyHost test]# tar zxvf xxxx.tar.gz    # 解压
  ```

- ##### tar指定xz压缩

  ```shell
  [root@MyHost test]# tar Jcvf xxxx.tar.xz 2 e
  [root@MyHost test]# ll
  -rw-r--r-- 1 root root 236 Aug  2 22:40 xxxx.tar.xz
  
  [root@MyHost test]# tar Jxvf xxxx.tar.xz   #解压
  ```

- ##### 自动解压

  ```shell
  [root@MyHost test]# tar xf xxxx.tar.gz   # tar自动选择解压方式
  ```

- ##### 当使用绝对路径指定压缩目录时，会自动去除/

  ```shell
  [root@MyHost test2]# tar zcf etc.tar.gz /etc/
  tar: Removing leading `/' from member names
  
   #也就是说默认情况下是解压到当前目录下，如果想指定解压路径要使用-C
  ```

- ##### 直接压缩排除后的文件

  ```shell
  [root@MyHost test2]# tar zvcf test.tar.gz --exclude=/etc/yum.repos.d --exclude=/etc/yum.conf  /etc/   # 排除两个文件后压缩
  
  [root@MyHost test2]# tar -xf test.tar.gz 
  ```



- ##### 分卷压缩

  ```shell
  split -b size file -d tarfile 
  -b  指定每一个分卷的大小
  -d 指定数字 默认是字母
  -a 指定后缀个数
  合并：
  cat tarfile* > file.tar.gz
  dd if=/dev/zero of=b bs=10M count=2
  split -b 5M b b.tar.gz
  split -b 5M b -d  b.tar.gz
  split -b 5M b -d -a 3 b.tar.gz
  
  [root@MyHost test2]# ll                #文件下有一个.gz文件
  -rw-r--r-- 1 root root 9794619 Aug  2 23:39 test.tar.gz
  
  [root@MyHost test2]# split -b 2M test.tar.gz -d  t.tar.gz
  
    # split -b 表示切割的大小每一个的  大小  切割文件  -d 表示后面用数字拼接 t.xxx.xx是切割后的文件名
  [root@MyHost test2]# ll
  -rw-r--r-- 1 root root 9794619 Aug  2 23:39 test.tar.gz
  -rw-r--r-- 1 root root 2097152 Aug  3 00:13 t.tar.gz00
  -rw-r--r-- 1 root root 2097152 Aug  3 00:13 t.tar.gz01
  -rw-r--r-- 1 root root 2097152 Aug  3 00:13 t.tar.gz02
  -rw-r--r-- 1 root root 2097152 Aug  3 00:13 t.tar.gz03
  -rw-r--r-- 1 root root 1406011 Aug  3 00:13 t.tar.gz04
  
   # 切割后的合并
  [root@MyHost test2]# cat t.tar.gz0* > xx.tar.gz
  [root@MyHost test2]# ll
  -rw-r--r-- 1 root root 9794619 Aug  2 23:39 test.tar.gz
  -rw-r--r-- 1 root root 2097152 Aug  3 00:13 t.tar.gz00
  -rw-r--r-- 1 root root 2097152 Aug  3 00:13 t.tar.gz01
  -rw-r--r-- 1 root root 2097152 Aug  3 00:13 t.tar.gz02
  -rw-r--r-- 1 root root 2097152 Aug  3 00:13 t.tar.gz03
  -rw-r--r-- 1 root root 1406011 Aug  3 00:13 t.tar.gz04
  -rw-r--r-- 1 root root 9794619 Aug  3 00:17 xx.tar.gz
  
  ```

  