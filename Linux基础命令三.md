## Linux基础命令三

#### 软连接

- 创建一个软连接

  ```shell
  [root@MyHost test]# cd ; ll
  -rw-r--r-- 1 root root    7 Jul 31 18:47 a.txt
  
  [root@MyHost ~]# cd test/; ll
  [root@MyHost test]# ln -s ../a.txt b
  [root@MyHost test]# ll
  lrwxrwxrwx 1 root root 8 Jul 31 18:50 b -> ../a.txt
  
    # 注意： 上面的链接文件的大小8 是后面生成的软连接的字符数：8个
  ```

- 文件详细

  ```
  drwxr-xr-x. 4                          root root 30 Jul 31 08:55 s12
  	权限     在磁盘上的引用次数（硬链接）    属主  属组 大小  atime      文件名 
  ```

- 软连接是新生成的，不能是之前存在的文件

  ```shell
  [root@MyHost test]# touch a
  [root@MyHost test]# ll
  -rw-r--r-- 1 root root 0 Jul 31 18:59 a    
  lrwxrwxrwx 1 root root 8 Jul 31 18:50 b -> ../a.txt
  [root@MyHost test]# ln -s /root/a.txt a    # 软连接生成的是非存在的
  ln: failed to create symbolic link ‘a’: File exists   
  [root@MyHost test]# ln -s /root/a.txt c
  [root@MyHost test]# ll
  -rw-r--r-- 1 root root  0 Jul 31 18:59 a
  lrwxrwxrwx 1 root root  8 Jul 31 18:50 b -> ../a.txt    
  lrwxrwxrwx 1 root root 11 Jul 31 19:01 c -> /root/a.txt  # 注意字节大小的变化，就是路径大小
  [root@MyHost test]# cat c         #当我们访问c时，其实是作用到了a.txt上
  adfadf
  ```

- 软连接总结

  ```
  1、一个符号链接指向另外一个文件
  2、ls -l可以查看链接的名称和引用的文件
  3、可以对目录进行软链接
  4、可以跨分区
  5、指向的是另外一个文件的路径；其大小为指向的路径字符串的长度；对文件的引用计数不影响
  6、软连接源文件删除，目标文件失效
  7、软连接可以链接一个目录
  语法 ln -s filename [linkname]
  ```

- 软连接误区

  ```
  1、当一个文件中存在软连接时，我们再创建同名文件时，是无效的
  [root@MyHost test]# touch b
  [root@MyHost test]# ll
  -rw-r--r-- 1 root root  0 Jul 31 18:59 a
  lrwxrwxrwx 1 root root  8 Jul 31 18:50 b -> ../a.txt
  
  ```



#### 硬链接

- 硬链接总结

  ```
  1、创建硬连接会增加额外的引用文件计数器
  2、对应于同一文件上一个物理文件
  3、每个目录引用相同的inode号
  4、创建时链接树递增
  5、删除文件时
  	5.1、rm命令递减技术的链接
  	5.2、当文件存在，至少有一个链接数
  	5.3、当链接数为0时，该文件被删除
  	5.4、源文件删除对目标不会受影响
  6、不能跨越分区
  7、源文件发生改变，目标会发生改变
  8、复制，相当于深拷贝，互不受影响
  9、硬链接不允许作用于目录
  语法：ln filename [linkname]
  ```

- 具体使用

  ```shell
  [root@MyHost test]# ln a d     # 如果d文件已经存在，则会报错
  ln: failed to create hard link ‘d’: File exists
  [root@MyHost test]# ln a e;ll
  -rw-r--r-- 2 root root  0 Jul 31 18:59 a    # 引用次数加一，本身为1
  -rw-r--r-- 2 root root  0 Jul 31 18:59 e
  
  [root@MyHost test]# echo adsafs > a
  [root@MyHost test]# cat e
  adsafs
  
  [root@MyHost test]# rm a
  rm: remove regular file ‘a’? y       #硬链接删除其中一个并不影响另一个
  [root@MyHost test]# cat e
  adsafs
  
  ```

- ##### 查看文件类型

  ```shell
  [root@MyHost test]# file b
  b: symbolic link to `../a.txt'
  [root@MyHost test]#  file .
  .: directory
  ```

- 文件默认引用值为2

  ```
  [root@MyHost ~]# ll
  drwxr-xr-x 2 root root 4096 Jul 31 19:24 test
   # 因为在test中有一个 . 是对父文件的引用
  ```

  

##### 硬链接和软连接的区别

> ```
> 	硬链接(hard link)：A是B的硬链接（A和B都是文件名），则A的目录项中的inode节点号与B的目录项中的inode节点号相同，即一个inode节点对应两个不同的文件名，两个文件名指向同一个文件，A和B对文件系统来说是完全平等的。如果删除了其中一个，对另外一个没有影响。每增加一个文件名，inode节点上的链接数增加一，每删除一个对应的文件名，inode节点上的链接数减一，直到为0，inode节点和对应的数据块被回收。注：文件和文件名是不同的东西，rm A删除的只是A这个文件名，而A对应的数据块（文件）只有在inode节点链接数减少为0的时候才会被系统回收。
> 
>     软链接(soft link)：A是B的软链接（A和B都是文件名），A的目录项中的inode节点号与B的目录项中的inode节点号不相同，A和B指向的是两个不同的inode，继而指向两块不同的数据块。但是A的数据块中存放的只是B的路径名（可以根据这个找到B的目录项）。A和B之间是“主从”关系，如果B被删除了，A仍然存在（因为两个是不同的文件），但指向的是一个无效的链接。
>     
>     也就是说：硬链接是指向同一个源文件，删除源文件，互不影响，即深拷贝，创建硬链接后，就是生成了一个存放数据的地址，不同的文件进行引用，一个删除，另一个不受影响，但是修改硬链接文件，会影响到另一个文件；软连接，是两个文件，只是‘从’中存放的是‘主’的文件源，删除源文件无影响，但是会失效；
>     硬链接和软连接的修改都会受到影响，但是删除，软连接会失效
> ```



#### IO操作

##### 标准输入和输出

```
标准输入 stdin 0 默认接受来自键盘的输入
标准输出 stdout 1 默认输出到终端窗口
标准错误 stderr 2 默认输出到终端窗口
```

- ##### 详解

  ```
  标准输入输出:
     Linux的大部分命令都具有标准的输入/输出设备端口，下图列出了标准设备信息:
            名称   文件描述     含义    设备        说明
           STDIN    0     标准输入    键盘     命令在执行时所要的输入数据通过它来取得
           STDOUT   1    标准输出    显示器    命令在执行后的输出结果从该端口送出
           STDERR   2   标准错误     显示器    命令执行时的错误信息通过该端口送出
  
  系统重定向:重定向就是不适用系统的标准输入端口，标准输出端口和标准错误输出端口，而进行重新的指定，所以重定向分为输入、输出和错误重定向，通常情况下重定向到一个文件。
  ```



#### io重定向

##### 把输出和错误重新定向到文件中

- **>  覆盖：**

  ```shell
  [root@MyHost ~]# cat a.txt;echo adfasfff > a.txt ;cat a.txt 
  aadf
  adfasfff
    # 只会覆盖
    
  >  将stdout重定向到文件
  
  2> 把stderr重定向到文件
  [root@MyHost ~]# ls /xxx  > a.txt 
  ls: cannot access /xxx: No such file or directory   # 错误信息不能正常输入到文件中，走的不是一个路口
  [root@MyHost ~]# ls /xxx  2> a.txt 
  [root@MyHost ~]# ls /xxx  2> a.txt ; cat a.txt 
  ls: cannot access /xxx: No such file or directory    # 查看写入的错误信息
  
  &> 把所有输出重定向到文件  
  [root@MyHost ~]# ls ; ls /sss &> a.txt    # 默认只把最后的错误信息写入了
  adfasfff  a.txt  test
  [root@MyHost ~]# cat a.txt
  ls: cannot access /sss: No such file or directory
  [root@MyHost ~]# (ls ; ls /sss) &> a.txt; cat a.txt   # 通过加括号来将正确和错的信息都输入
  adfasfff
  a.txt
  test
  ls: cannot access /sss: No such file or directory
  
  ```

- **>>追加：**

  ```shell
  >> 将stdout追加到文件
  2>> 把stderr追加到文件
  &>> 把所有输出追加到文件
  
  [root@MyHost ~]# (ls; ls /xxxx) &> c.txt; cat c.txt   # 没有c.txt就自动生成
  adfasfff
  a.txt
  b.txt
  c.txt
  test
  ls: cannot access /xxxx: No such file or directory
  
  [root@MyHost ~]# > 6.txt      #这两种方法可以生成文件，如果没有的话
  [root@MyHost ~]# >> 7.txt
  ```

- 标准输出和错误输出各自定向到不同的文件

  ```shell
   # 语法：command > /path/to/file.out 2> /path/to/error.out
  [root@centos ~]#ls /dada /opt > a.txt 2>b.txt    # 正确的信息走> 错误走错误
  
  [root@MyHost ~]# ls test dsfsdf > info.log 2> error.log;cat info.log error.log
  test:
  b
  c
  d
  e
  ls: cannot access dsfsdf: No such file or directory
  ```



#### 合并输出

- &> 覆盖重定向

- &>> 追加重定向

- command > /path/to/file.out 2>&1

  ```shell
  [root@MyHost ~]# ls test xxxx > info.log 2>&1;cat info.log   # 将错误信息转交给标准输出
  ls: cannot access xxxx: No such file or directory
  test:
  b
  c
  d
  e
  ```

- () 合并多个程序的stdout

  ```shell
  [root@MyHost ~]# ls test xxxx &> /dev/null 
  [root@MyHost ~]# (ls test xxxx) &> 11.txt; cat 11.txt
  ls: cannot access xxxx: No such file or directory
  test:
  b
  c
  d
  e
  ```

- /dev/null 丢到外太空



#### tr 替换或者删除字符

- 语法： tr  替换对象  替换内容

  ```shell
  [root@MyHost ~]# tr ab 12  # 将输入的ab替换成12
  ab
  12
  sadf
  s1df
  
  [root@MyHost ~]# echo adfsdfsfafabab > 12.txt
  [root@MyHost ~]# tr ab 12 < 12.txt   #将12.txt中的内容进行替换
  1dfsdfsf1f1212
  
  
  [root@MyHost ~]# tr 'a-z' 'A-Z' < 12.txt ; cat 12.txt 
  ADFSDFSFAFABAB            # 默认输出到终端，本身不变
  adfsdfsfafabab
  [root@MyHost ~]# cat 12.txt 
  adfsdfsfafabab
  
  [root@MyHost ~]# tr abc 12;  # 当替换对象大于 替换值，那么替换值的最后一位补充
  adc
  1d2
  adcc
  1d22
  abcc
  1222
  
  [root@MyHost ~]# tr -t abc 12;   # -t 用于避免上面情况，截断不同的不替换
  aabc
  112c
  adcd
  1dcd
  abcd
  12cd
  
  [root@MyHost ~]# tr -d abc    # -d 删除匹配到的部分
  abcd
  d
  afbdxece    
  fdxee
  
  [root@MyHost ~]# cat 12.txt    #文件读入
  adfsdfsfafabab
  [root@MyHost ~]# tr -d abc < 12.txt 
  dfsdfsff
  
  [root@MyHost ~]# tr -d abc < 12.txt >12.txt ; cat 12.txt 
  [root@MyHost ~]# cat 12.txt 
  [root@MyHost ~]# echo adfafadsfasdf > 12.txt 
  [root@MyHost ~]# tr -d abc < 12.txt >13.txt ; cat 13.txt   #输入和输出的文件名不能是相同的，否则源文件为空
  dffdsfsdf
  
  [root@MyHost ~]# tr -s abc   #-s 对数据进行去重
  aabc     # 输入aabc
  abc
  aaaa
  a
  
  [root@MyHost ~]# tr -s aabc   #不完整的去重规则，不同区间上的不能去重
  adf
  adf
  adfa
  adfa
  aaffaab
  affab
  bbaabbccssfdsa
  babcssfdsa
  
  [root@MyHost ~]# tr -sc abc   # -sc 表示给取反元素去重
  aabbccddeex
  aabbccdex
  aabbccdexxeedd    
  aabbccdexed
  
  [root@localhost jiangyi]#tr -dc abc
  aaaaaaaaaaaaabbbbbbbbbbbbccccccccccccccccccdddddddddddddddwqweqweqwqeqwqwqwq
  wqqqqqqqqqqqqqqqqqqqqqqqqq
  ctrl+d结束输入： # 原因是因为-dc对abc以外的取反，那么换行符也省略了
  
  [root@MyHost ~]# tr -dc "abc\n"  # 除去换行符取反
  abcaffssxxsd
  abca
  
  [root@MyHost ~]# seq 1 10 > f1
  [root@MyHost ~]# tr -d '\n' < f1
  12345678910[root@MyHost ~]#     #横着显示
  
  [root@localhost jiangyi]tr "\n" " "<f1   # 替换为空格
  [root@localhost jiangyi] tr " " "\n" <f2  # 竖着显示
  ```



#### 多行输入

- 可视化输入多行使用 ： cat  > f1  

  ```shell
   # 方式一
  [root@MyHost ~]# cat > f1 <<OK
  > a
  > d
  > 
  > d
  > OK
  [root@MyHost ~]# cat f1
  a
  d
  
  d
   
   #方式二
  [root@localhost jiangyi]# cat > f4
  asdas
  ctrl+d结束 ctrl+c也可以
  两者区别
      第一种方式输出结束，文件才会产生(使用OK的方式)
      第二方式，回车一次就会写入文件 （cat > f1)
      EOF 约定俗成
  ```

  

#### 管道

使用“|”来连接多个命令

命令1|命令2|命令3|...

- 将命令的stdout发送给命令2的stdin，将命令2的stdout命令发送给命令3的stdin...(接力)

- stderr默认是不能通过管道传递

  ```shell
  [root@localhost jiangyi]#ls /dadadasda|tr -s "a-z" "A-Z"
  ls: cannot access /dadadasda: No such file or directory
  [root@localhost jiangyi]#ls|tr "a-z" "A-Z"
  
  lrwxrwxrwx 1 root root  8 Jul 31 18:50 b -> ../a.txt
  lrwxrwxrwx 1 root root 11 Jul 31 19:01 c -> /root/a.txt
  -rw-r--r-- 1 root root  0 Jul 31 19:13 d
  -rw-r--r-- 1 root root  7 Jul 31 19:18 e
  [root@MyHost test]# ls | tr "a-e" "A-Z"  # 过滤，替换
  B
  C
  D
  E
  ```



#### 查看当前的登录的用户信息

```shell
whoami  获取登录的用户
[jiangyi@localhost ~]$who am i
jiangyi（用户）  pts/4（登录的终端）        2019-07-31 08:27（登录的时间） (192.168.182.1（登录ip地址）)

w 可以查看当前登录的所有用户执行的命令
[root@MyHost test]# w
 21:59:09 up 22:15,  1 user,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/1    106.121.8.240    18:11    5.00s  0.18s  0.00s w
 
[root@MyHost test]# whoami
root
```



#### 文件的权限

![1564579312494](C:\Users\RootUser\Desktop\知识点复习\Django\gif\1564579312494.png)

- ##### 文件属性操作

  ```shell
  chown 设置文件的所有者
  chgrp 设置文件的所属组
   
  [root@MyHost test]# ll d    # 当d是文件时，ll会打印它的信息
  -rw-r--r-- 1 root root 0 Jul 31 19:13 d
  ```

- ##### 修改文件的属主和属组

  - ##### 修改文件的属主：chown

    ```shell
    语法：chown [option]…[owner][:[group]] file
    
    [root@MyHost test]# ll
    -rw-r--r-- 1 root root  0 Jul 31 19:13 d
    [root@MyHost test]# chown wang:wang d     # 修改 属主和属组
    [root@MyHost test]# ll
    -rw-r--r-- 1 wang wang  0 Jul 31 19:13 d
    
     #命令中的冒号也可以用.替代
    [root@MyHost test]# chown root.root d
    
     #-R递归修改文件的属主和属组
    [root@MyHost test]# mkdir -p a/b/c    
    [root@MyHost test]# chown -R wang a
    [root@MyHost test]# ll a
    
     # --reference 以谁作为模板
    [root@MyHost test]# ll
    drwxr-xr-x 3 wang root 4096 Jul 31 21:33 a
    -rw-r--r-- 1 root wang    0 Jul 31 19:13 d
    [root@MyHost test]# chown --reference=d a;ll
    total 8
    drwxr-xr-x 3 root wang 4096 Jul 31 21:33 a
    -rw-r--r-- 1 root wang    0 Jul 31 19:13 d
    
    
    ```

    - **总结：**

      ```shell
      chown jiangyi d    #修改属主
      chown jiangyi:jiangyi d  #修改属主和属组
      chown root.root d
      chown :jangyi d    #只修改属组信息
      chown -R jiangyi a #递归修改目录下的所有文件
      chown --reference=b f3  #指定源文件
      ```

      

  - #### 修改文件属组

    ```shell
    [root@MyHost test]# chgrp wang d
    [root@MyHost test]# ll
    -rw-r--r-- 1 root wang    0 Jul 31 19:13 d
    ```

    - ##### 总结：

      ```shell
      chown jiangyi d    #修改属主
      chown jiangyi:jiangyi d  #修改属主和属组
      chown root.root d
      chown :jangyi d    #只修改属组信息
      chown -R jiangyi a #递归修改目录下的所有文件
      chown --reference=b f3  #指定源文件
      ```

      

#### 权限

```
-rw-r--r--
三位为一组，第一个是文件类型，后面每三个为一组
属主      属组     其他
u         g        o

r  read 可以读这个文件或者文件夹
w  write 可以对这个文件或者文件夹有写的权限
x  excut 执行的权限
```

- 那么文件、目录都有自己的权限，那么分别的r、w、x代表什么哪？

  ```shell
  每个文件针对每类访问者都定义了三种权限
  	r read
  	w write
  	x excut
  文件
  	r 可以使用文件查看类工具获取其内容(获取文件内容)
  	w 可以修改其内容
  	x 可以执行
  目录
  	r 可以使用ls 查看此目录的文件列表（获取列表可以使用ls查看 可以cd进去）
  	w 可在此目录中创建文件，也可以删除此目录中的文件
  	x 可以使用ls查看，可以cd将进入，如果没有x权限的话，w权限不会生效，r权限可以查看有哪些文件
  
  	
   # 注意：root不受任何限制，读写一把梭，无人能挡
  ```

- ##### 禁止覆盖

  ```shell
  set -C 禁止覆盖
  set +C 允许覆盖
  
  [root@MyHost test]# > a.txt
  [root@MyHost test]# set -C
  [root@MyHost test]# > a.txt
  -bash: a.txt: cannot overwrite existing file
  ```

- ##### 权限的限制

  ```shell
  [wang@MyHost ~]$ ll 
  drwxr-xr-x 2 root root 4096 Jul 31 22:05 test
  [wang@MyHost ~]$ cd test/
  [wang@MyHost test]$ touch a
  touch: cannot touch ‘a’: Permission denied
  
   # 只要不是属主和属组，那么就是other组了，撒谎给你吗的test显然是other，所以x可以cd进入，r可以使ls
  
   #root端
  [root@MyHost wang]# chmod o+r test/; ll
  drwxr-xr-x 2 root root 4096 Jul 31 22:05 test
  [root@MyHost wang]# chmod o-r test/; ll
  drwxr-x--x 2 root root 4096 Jul 31 22:05 test
  
   #wang用户端
  [wang@MyHost test]$ ls
  [wang@MyHost test]$ ls
  ls: cannot open directory .: Permission denied
  
  ```



#### chmod方法的使用

- 可以直接用+-来操作

  - 可以用[u|g|o]+ - = r w x

    ```SHELL
    [root@MyHost ~]# ll 11.txt 
    -rw-r--r-- 1 root root 64 Jul 31 20:29 11.txt
    [root@MyHost ~]# chmod o=w 11.txt ;ll 11.txt     # 使用等号，确定某一组
    -rw-r---w- 1 root root 64 Jul 31 20:29 11.txt
    
    ```

  - 可以什么都不写，表示全部 +-

- 还可以用数字表示

  \---

  r--    100        4

  -w-   010       2

  --x    001       1

  ```shell
  r:4
  w:2
  x:1
  [root@MyHost wang]# ll
  drwxr-x--x 2 root root 4096 Jul 31 22:05 test
  [root@MyHost wang]# chmod 755 test
  [root@MyHost wang]# ll
  drwxr-xr-x 2 root root 4096 Jul 31 22:05 test
  ```

- 使用x权限允许python文件

  ```shell
  [root@MyHost wang]# echo '#!/usr/bin/python' > a.py
  [root@MyHost wang]# echo '#coding:utf-8' >> a.py
  [root@MyHost wang]# echo 'print("床前明月光")' >> a.py
  [root@MyHost wang]# ./a.py
  -bash: ./a.py: Permission denied
  [root@MyHost wang]# ll
  total 8
  -rw-r--r-- 1 root root   57 Jul 31 22:22 a.py
  drwxr-x--x 2 root root 4096 Jul 31 22:05 test
  [root@MyHost wang]# chmod o+x a.py 
  [root@MyHost wang]# ./a.py
  床前明月光
  ```

- ##### 强大的bin目录

  ```shell
  [root@MyHost wang]# cp a.py /bin
  [root@MyHost wang]# chmod o+x /bin/a.py 
  [root@MyHost wang]# a.py
  床前明月光
    # 系统默认会先去bin下查找文件，也就是环境变量PATH
  ```

#### 设置文件特殊权限

```shell
chattr +i 不能删除，改名，变更
chattr +a 只能追加内容，不能删除，不能改名
lsattr    显示特殊权限

[root@MyHost wang]# chattr +i test/
[root@MyHost wang]# echo sadf > test/a
-bash: test/a: Permission denied
[root@MyHost wang]# lsattr test/
[root@MyHost wang]# lsattr 
----i--------e-- ./test
-------------e-- ./a.py
[root@MyHost wang]# chattr -i test/
[root@MyHost wang]# echo sadf > test/a

```



#### 文本处理工具

##### cat

-  常用选项

  ```shell
  cat [option] ... [file] ...
  -E 显示行结束符$
  -n 对显示出的每一行进行编号
  -b 非空行编号
  -s 压缩连续的空行成一行,将空行折叠为一行，含有空格的不算空行
  
  [root@MyHost ~]# echo " " >> passwd 
  [root@MyHost ~]# echo " " >> passwd 
  [root@MyHost ~]# cat -b passwd 
       1	root:x:0:0:root:/root:/bin/bash
       ...
      25	admin:x:1000:1000::/home/admin:/bin/bash
      26	wang:x:1001:1001::/home/wang:/bin/bash
      27	 
      28	 
  ```



##### tac 倒叙显示文件

- 和cat相反，倒叙

  ```shell
  [root@MyHost ~]# cat a
  2
  3
  4
  5
  [root@MyHost ~]# tac a
  5
  4
  3
  2
  ```

##### less 分屏显示

- 将内容分屏显示

  ```
  [root@MyHost ~]# less passwd 
  
  - 可以分屏显示
  - 空格一屏  回车一行
  - /来搜索，输入内容回车即可
  - n 向下搜索 N 向上搜索
  - q来退出
  ```

##### more 

- 类似于less

  ```
  more 分页查看文件
  more [options…] file...
  -d 显示翻页以及退出提示
  空格翻一屏，回车翻一行
  ```

  

##### 显示文本前或后行内容

- head  [option] … [file]… 默认10行

  ```shell
  -#  #指定获取前#行
  [root@MyHost ~]# head -5 passwd 
  ...
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  ```

- tail [options]…[file]… 默认10行

  ```shell
  -#  #指定获取后#行
  -f  #追踪显示文件fd新追加的内容，常用与日志监控
  tailf 类似于 tail -f 当文件不增长时并不访问文件
  
  [root@MyHost ~]# tail 5 passwd    # 显示后5行
  ...
  postfix:x:89:89::/var/spool/postfix:/sbin/nologin
  chrony:x:997:995::/var/lib/chrony:/sbin/nologin
  
  [root@MyHost wang]# tail -f v.txt 
  adfadf
  132e
  dfaafdf
  sdfsafd
  
   # 在wang用户中
  [wang@MyHost ~]$ echo sdfsafd >> v.txt 
  ```



#### 抽取文本cut

- 用于从文件中提取信息

  ```shell
  -d 用来指定切割符号
  -f 执行显示哪一个数据
  # 显示指定的数据
  #，#，#，# 离散数据
  #-# 连续数据
  cut -d: -f3 passwd 
  cut -d: -f1,3-7 passwd 
  cut -d: -f3,4,5,6 passwd   # 获取相应字段
  cut -d: -f3-6 passwd   # 获取3-6部分
  
  -c 按照字符切割
  cut -c2-5 passwd
  
  
  [root@MyHost ~]# cat passwd 
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  sync:x:5:0:sync:/sbin:/bin/sync
  ...
  [root@MyHost ~]# cut -d: -f3 passwd   # 取出以:分隔的第三部分
  0
  1
  2
  3
  4
  ...
  
  [root@MyHost ~]# cut -c2-5 passwd   # 从第二个字符到第5个字符
  oot:
  in:x
  aemo
  dm:x
  p:x:
  ...
  ```

  

#### 合并文件

- 合并两个文件同行号到一行

  ```shell
  paste 合并两个文件同行号的列到一行
  paste[option]...[file]...
  -d 分隔符 指定分隔符，默认是tab
  -s 所有行合并成一行显示
  paste f1 f2
  paste -d: f1 f2
  paste -s f1 f2
  
  [root@MyHost ~]# paste -d: a.txt b.txt   # 指定了分隔符
  sdf:q
  2:1
  8:2
  :3
  
  [root@MyHost ~]# paste -s a.txt b.txt 
  sdf	2	8
  q	1	2	3
  
  ```

  

#### 分析文本工具



##### 文本数据统计：wc （word count）

- 奇数单词总数，行总数，字节总数和字符总数

  ```shell
  [root@MyHost ~]# wc passwd 
    29   47 1204 passwd
    
   [root@centos ~]#wc a.sh
   4  5 30 a.sh
   行数  字数 字节数 文件
   -l 只统计行数
   -w 只统计单词总数
   -c 只统计字节总数
   -m 只统计字符总数
   -L 显示文件中最长行的长度
  ```

  

- 可以对文件或者stdin中的数据进行统计



##### 文本排序

- 把整理过的文本显示在stdout不改变原始文件

  ```shell
  sort [options] files 默认是字母排序
  -r 执行倒序排列
  -R 随机排序
  -n 按数字大小排序
  -f 忽略大小写
  -t c 执行分隔符
  -k 按第几列来进行排序
  
  [root@MyHost ~]# cat > f4 << OK
  > 1
  > 4
  > 5
  > 2
  > 6
  > 7
  > 4
  > 3
  > OK
  [root@MyHost ~]# sort -n f4
  1
  2
  3
  4
  4
  5
  6
  7
  
  [root@MyHost ~]# sort -t: -nk3 passwd   # 按照：分隔的第三列的数字排序
  
  ```



##### uniq 删除重复的行

- 删除重复的行

  ```shell
  -c 显示重复出现的次数
  -d 只显示重复的行
  -u 只显示不重复的行
  连续且完全一样的才是重复
  ss -tnp|cut -d: -f2|tr -s " "|cut -d" " -f2|sort -n|uniq -c
  
  [root@MyHost ~]# cat > f5 << OK
  > 1
  > 1
  > 1
  > 2
  > 2
  > 3
  > OK
  [root@MyHost ~]# uniq f5
  1
  2
  3
  
  [root@MyHost ~]# uniq -c f5   # -c显示出现次数
        3 1
        2 2
        1 3
    
  [root@MyHost ~]# uniq -d f5  #显示重复的行
  1
  2
  
  连续且完全一样的才是重复，显示登陆ip的次数排序
  ss -tnp|cut -d: -f2|tr -s " "|cut -d" " -f2|sort -n|uniq -c
  ```

  

##### diff对比两个文件

```shell
[root@MyHost ~]# echo "aacsdf" >> b
[root@MyHost ~]# echo "aa" >> a
[root@MyHost ~]# diff b a
1c1,5
< aacsdf
---
> 2
> 3
> 4
> 5
> aa

```

