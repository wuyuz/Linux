## 消息队列与kafka



#### kafka

为什么用消息队列，举例

```
比如在一个企业里，技术老大接到boss的任务，技术老大把这个任务拆分成多个小任务，完成所有的小任务就算搞定整个任务了。
那么在执行这些小任务的时候，可能有一个环节很费时间，并且优先级很低，推迟完成也不影响整个任务运转，那么技术老大就会将这个很费时间，且不重要的任务，丢给他的小弟去解决，自己继续完成其他任务。
```

转化为计算机思想

```
那个技术老大就是一个 程序系统，那个小弟就是消息队列。
当程序系统发现某些任务耗费时间且优先级较低，迟点完成也不影响整个任务，就把这个任务丢给消息队列。
```

场景

```
在程序系统中，例如外卖系统，订单系统，库存系统，优先级较高
发红包，发邮件，发短信，app消息推送等任务优先级很低，很适合交给消息队列去处理，以便于程序系统更快的处理其他请求。
```

消息队列工作流程

```
消息队列一般有三个角色：
    队列服务端
    队列生产者
    队列消费者
    
消息队列工作流程就如同一个流水线，有产品加工，一个输送带，一个打包产品
输送带就是 不停运转的消息队列服务端
加工产品的就是 队列生产者
在传输带结尾打包产品的 就是队列消费者
```

队列产品

```
RabbitMQ
	Erlang编写的消息队列产品，企业级消息队列软件，支持消息负载均衡，数据持久化等。

ZeroMQ 
	saltstack软件使用此消息，速度最快。

Redis
	key-value的系统，也支持队列数据结构，轻量级消息队列

Kafka
	由Scala编写，目标是为处理实时数据提供一个统一、高通量、低等待的平台
```

一个app系统消息队列工作流程

```
消费者，一个后台进程，不断的去检测消息队列中是否有消息，有消息就取走，开启新线程去处理业务，如果没有一会再来
```



#### kafka是什么

在流式计算中，Kafka一般用来缓存数据，Storm通过消费Kafka的数据进行计算。

​	1）Apache Kafka是一个开源**消息**系统，由Scala写成。是由Apache软件基金会开发的一个开源消息系统项目。

​	2）Kafka最初是由LinkedIn公司开发，并于 2011年初开源。2012年10月从Apache Incubator毕业。该项目的目标是为处理实时数据提供一个统一、高通量、低等待的平台。

​	3）**Kafka是一个分布式消息队列。**Kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，此外kafka集群有多个kafka实例组成，每个实例(server)成为broker。

​	4）无论是kafka集群，还是producer和consumer都依赖于**zookeeper**集群保存一些meta信息，来保证系统可用性。

## 消息通信图



------

点对点模式（一对一，消费者主动拉取数据，轮询机制，消息收到后消息清除，ack确认机制）

​	点对点模型通常是一个基于`拉取`或者`轮询`的消息传送模型，这种模型从队列中请求信息，而不是将消息推送到客户端。这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。

------

发布/订阅模式（一对多，数据生产后，推送给所有订阅者）

​	发布订阅模型则是一个基于推送的消息传送模型。发布订阅模型可以有多种不同的订阅者，临时订阅者只在主动监听主题时才接收消息，而持久订阅者则监听主题的所有消息，即使当前订阅者不可用，处于离线状态。



#### 消息队列作用

1）程序解耦

​	允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

2）冗余：

​	消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。

许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

3）峰值处理能力：

​	(大白话，就是本来公司业务只需要5台机器，但是临时的秒杀活动，5台机器肯定受不了这个压力，我们又不可能将整体服务器架构提升到10台，那在秒杀活动后，机器不就浪费了吗？因此引入消息队列)

在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。

如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。

使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

4）可恢复性：

​	系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

5）顺序保证：

​	在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka保证一个Partition内的消息的有序性）

6）缓冲：

​	有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。

7）异步通信：

​	很多时候，用户不想也不需要立即处理消息。比如发红包，发短信等流程。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。



#### kafka架构

1）Producer ：消息生产者，就是向kafka broker发消息的客户端。

2）Consumer ：消息消费者，向kafka broker取消息的客户端

3）Topic ：主题，可以理解为一个队列。

4） Consumer Group （CG）：这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制-给consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic。

5）Broker ：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。

6）Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序。

7）Offset：kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka



#### **分布式模型**

 Kafka每个主题的多个分区日志分布式地存储在Kafka集群上，同时为了故障容错，每个（partition）分区都会以副本的方式复制到多个消息代理节点上。

​		其中一个节点会作为主副本（Leader），其他节点作为备份副本（Follower，也叫作从副本）。主副本会负责所有的客户端读写操作，备份副本仅仅从主副本同步数据。当主副本出现故障时，备份副本中的一个副本会被选择为新的主副本。因为每个分区的副本中只有主副本接受读写，所以每个服务器端都会作为某些分区的主副本，以及另外一些分区的备份副本，这样Kafka集群的所有服务端整体上对客户端是负载均衡的。Kafka的生产者和消费者相对于服务器端而言都是客户端。

**Kafka生产者客户端发布消息到服务端的指定主题，会指定消息所属的分区。**

​		生产者发布消息时根据消息是否有键，采用不同的分区策略。消息没有键时，通过轮询方式进行客户端负载均衡；消息有键时，根据分区语义（例如hash）确保相同键的消息总是发送到同一分区。

**Kafka的消费者通过订阅主题来消费消息，并且每个消费者都会设置一个消费组名称。因为生产者发布到主题的每一条消息都只会发送给消费者组的一个消费者。**

​		所以，如果要实现传统消息系统的“队列”模型，可以让每个消费者都拥有相同的消费组名称，这样消息就会负责均衡到所有的消费者；如果要实现“发布-订阅”模型，则每个消费者的消费者组名称都不相同，这样每条消息就会广播给所有的消费者。

**分区是消费者现场模型的最小并行单位。**

​		如下图（图1）所示，生产者发布消息到一台服务器的3个分区时，只有一个消费者消费所有的3个分区。在下图（图2）中，3个分区分布在3台服务器上，同时有3个消费者分别消费不同的分区。假设每个服务器的吞吐量时300MB，在下图（图1）中分摊到每个分区只有100MB，而在下图（图2）中，集群整体的吞吐量有900MB。可以看到，增加服务器节点会提升集群的性能，增加消费者数量会提升处理性能。

同一个消费组下多个消费者互相协调消费工作，Kafka会将所有的分区平均地分配给所有的消费者实例，这样每个消费者都可以分配到数量均等的分区。Kafka的消费组管理协议会动态地维护消费组的成员列表，当一个新消费者加入消费者组，或者有消费者离开消费组，都会触发再平衡操作。

Kafka的消费者消费消息时，只保证在一个分区内的消息的完全有序性，并不保证同一个主题汇中多个分区的消息顺序。而且，消费者读取一个分区消息的顺序和生产者写入到这个分区的顺序是一致的。比如，生产者写入“hello”和“Kafka”两条消息到分区P1，则消费者读取到的顺序也一定是“hello”和“Kafka”。如果业务上需要保证所有消息完全一致，只能通过设置一个分区完成，但这种做法的缺点是最多只能有一个消费者进行消费。一般来说，只需要保证每个分区的有序性，再对消息假设键来保证相同键的所有消息落入同一分区，就可以满足绝大多数的应用。



#### kafka部署启动

配置jdk环境

```
下载网址
wget https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
找到
jdk-8u201-linux-x64.tar.gz
```

解压缩，配置java环境变量

```
tar -zxvf jdk-8u201-linux-x64.tar.gz

PATH="$PATH:/opt/jdk1.8.0_201/bin"
```

配置zookeeper环境，配置环境变量

```
tar -zxvf zookeeper-3.4.14.tar.gz

PATH="$PATH:/opt/jdk1.8.0_201/bin:/opt/zookeeper-3.4.14/bin"
```

zookeeper端口解释

```
1、2181
2、3888
3、2888

二、3个端口的作用
    1、2181：对cline端提供服务
    2、3888：选举leader使用
    3、2888：集群内机器通讯使用（Leader监听此端口）
    
部署时注意
    1、单机单实例，只要端口不被占用即可
    2、单机伪集群（单机，部署多个实例），三个端口必须修改为组组不一样

如：
myid1 : 2181,3888,2888
myid2 : 2182,3788,2788
myid3 : 2183,3688,2688

3、集群（一台机器部署一个实例）

四、集群为大于等于3个基数，如 3、5、7....,不宜太多，集群机器多了选举和数据同步耗时时长长，不稳定。目前觉得，三台选举+N台observe很不错。
```



#### 启动安装zookeeper

本文以standalone模式运行，并非集群模式

```
1.解压缩zk压缩包，配置好环境变量
2.在zk解压缩包目录下创建 zkData目录
3.修改zk解压缩包目录下conf/zoo_sample.cfg为zoo.cfg
4.编辑zoo.cfg配置文件，修改代码
```

zookeeper-3.4.14/conf/zoo.cfg修改如下参数

```
dataDir=/opt/zookeeper-3.4.14/zkData
server.2=192.168.119.10:2888:3888 #修改为你自己服务器的ip
```

**参数解释**

```
Server.A=B:C:D。

A是一个数字，表示这个是第几号服务器；

B是这个服务器的ip地址；

C是这个服务器与集群中的Leader服务器交换信息的端口；

D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。
```

启动zk服务端

```
zkServer.sh start   #启动
zkServer.sh status  #检查状态
```



#### kafka部署

```
下载二进制kafka代码包
wget http://apache.claz.org/kafka/2.2.0/kafka_2.11-2.2.0.tgz
解压缩
tar -xf kafka_2.11-2.2.0.tgz
修改kafka服务端配置文件
/opt/kafka_2.11-2.2.0/config/server.properties
#创建kafka日志文件夹
mkdir -p /opt/kafka_2.11-2.2.0/logs
```

/opt/kafka_2.11-2.2.0/config/server.properties修改如下参数；如果修改了kafka的启动地址参数，注意可能出现的权限问题，或者删除logs目录下的数据文件；9092是kafka服务端

```
#broker的全局唯一编号，不能重复
broker.id=0
#是否允许删除topic
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘IO的线程数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的最大缓冲区大小
socket.request.max.bytes=104857600
#kafka运行日志存放的路径
log.dirs=/opt/kafka_2.11-2.2.0/logs
#topic在当前broker上的分区个数
num.partitions=1
#用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment文件保留的最长时间，超时将被删除
log.retention.hours=168

#配置连接Zookeeper集群地址，确保zk正确启动2181已经打开
zookeeper.connect=192.168.119.10:2181
```

修改linux的PATH环境变量，支持kafka命令

```
[root@localhost bin]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/jdk1.8.0_201/bin:/opt/zookeeper-3.4.14/bin:/opt/kafka_2.11-2.2.0/bin
```

启动kafka服务端，指定配置文件，后台启动

```
[root@localhost kafka_2.11-2.2.0]# kafka-server-start.sh config/server.properties &
```

看到如下提示，代表kafka启动成功

```
[2019-04-12 23:53:33,229] INFO Kafka version: 2.2.0 (org.apache.kafka.common.utils.AppInfoParser)
[2019-04-12 23:53:33,229] INFO Kafka commitId: 05fcfde8f69b0349 (org.apache.kafka.common.utils.AppInfoParser)
[2019-04-12 23:53:33,231] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```



#### kafka命令行操作

```
查看当前服务器中的所有topic
[root@localhost kafka_2.11-2.2.0]# kafka-topics.sh --zookeeper 192.168.119.10:2181 --list

创建topic
[root@localhost kafka_2.11-2.2.0]# kafka-topics.sh --zookeeper 192.168.119.10:2181 --create --replication-factor 1 --partitions 1 --topic first

选项说明：
--topic 定义topic名
--replication-factor  定义副本数
--partitions  定义分区数

删除topic
kafka-topics.sh --zookeeper 192.168.119.10:2181 --delete --topic first
需要server.properties中设置delete.topic.enable=true否则只是标记删除或者直接重启。

发送消息，9092是kafka的服务端口
[root@localhost kafka_2.11-2.2.0]# kafka-console-producer.sh --broker-list 192.168.119.10:9092 --topic first
>hello kafka
>chaoge niubi

消费消息，注意kafka的版本，以及新参数特性
[root@localhost kafka_2.11-2.2.0]# kafka-console-consumer.sh --bootstrap-server  192.168.119.10:9092 --from-beginning --topic first
--from-beginning：会把first主题中以往所有的数据都读取出来。根据业务场景选择是否增加该配置。
broker
    topic
        partition
        
三者包含关系
```



#### python操作kafka

环境准备

```
[root@localhost pykafka]# python3 -V
Python 3.6.7

启动好zk,kafka，确保2181端口，9092端口启动
```

Python模块安装

```
pip3 install kafka-python
```

**生产者**

```
[root@localhost pykafka]# cat pro.py
import time
from kafka import KafkaProducer
#连接上kafka服务端9092端口
producer = KafkaProducer(bootstrap_servers = ['192.168.119.10:9092'])
# 注册一个主题，名字topic
topic = 'oldboy'

#每秒钟，写入一个消息数据
def test():
    print ('begin produce..')
    n = 1
    try:
        while (n<=100):
              #向主题oldboy中发送byte数据
            producer.send(topic, str(n).encode())
            print("send" + str(n))
            n += 1
            time.sleep(0.5)
    except KafkaError as e:
        print(e)
    finally:
          #关闭连接
        producer.close()
        print('done')

if __name__ == '__main__':
    test()
```

消费者

```
[root@localhost pykafka]# cat consumer.py

from kafka import KafkaConsumer

#connect to Kafka server and pass the topic we want to consume
consumer = KafkaConsumer('oldboy', group_id = 'oldboy_group', bootstrap_servers = ['192.168.119.10:9092'])
try:
    for msg in consumer:
        print(msg)
        print("%s:%d:%d: key=%s value=%s" % (msg.topic, msg.partition,msg.offset, msg.key, msg.value))
except KeyboardInterrupt as  e:
    print(e)
```