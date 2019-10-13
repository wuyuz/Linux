## RabbitMQ 消息队列



#### 为什么用消息队列

举例

```
比如在一个企业里，技术老大接到boss的任务，技术老大把这个任务拆分成多个小任务，完成所有的小任务就算搞定整个任务了。那么在执行这些小任务的时候，可能有一个环节很费时间，并且优先级很低，推迟完成也不影响整个任务运转，那么技术老大就会将这个很费时间，且不重要的任务，丢给他的小弟去解决，自己继续完成其他任务。
```

转化为计算机思想

```
那个技术老大就是一个 程序系统，那个小弟就是消息队列。当程序系统发现某些任务耗费时间且优先级较低，迟点完成也不影响整个任务，就把这个任务丢给消息队列。
```

场景

```
在程序系统中，例如外卖系统，订单系统，库存系统，优先级较高；发红包，发邮件，发短信，app消息推送等任务优先级很低，很适合交给消息队列去处理，以便于程序系统更快的处理其他请求。
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



#### 消息队列的作用

1）程序解耦

​	允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

2）冗余：

​	消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指	出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

3）峰值处理能力：

​	(大白话，就是本来公司业务只需要5台机器，但是临时的秒杀活动，5台机器肯定受不了这个压力，我们又不可能将整体服务器架构提升到10台，那在秒杀活动后，机器不就浪费了吗？因此引入消息队列)，在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

4）可恢复性：

​	系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

5）顺序保证：

​	在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka保证一个Partition内的消息的有序性）

6）缓冲：

​	有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。

7）异步通信：（重要）

​	很多时候，用户不想也不需要立即处理消息。比如发红包，发短信等流程。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。



#### 了解RabbitMQ

- 什么是消息队列

  ```
  生活里的消息队列，如同邮局的邮箱，如果没邮箱的话，邮件必须找到邮件那个人，递给他，才玩完成，那这个任务会处理的很麻烦，很慢，效率很低
  
  但是如果有了邮箱，邮件直接丢给邮箱，用户只需要去邮箱里面去找，有没有邮件，有就拿走，没有就下次再来，这样可以极大的提升邮件收发效率！
  
  	rabbitmq是一个消息代理，它接收和转发消息，可以理解为是生活的邮局。你可以将邮件放在邮箱里，你可以确定有邮递员会发送邮件给收件人。概括：rabbitmq是接收，存储，转发数据的。官方教程：http://www.rabbitmq.com/tutorials/tutorial-one-python.html
  ```

  消息（Message）是指在应用间传送的数据。消息可以非常简单，比如只包含文本字符串，也可以更复杂，可能包含嵌入对象。

  消息队列（Message Queue）是一种应用间的通信方式，消息发送后可以立即返回，由消息系统来确保消息的可靠传递。消息发布者只管把消息发布到 MQ 中而不用管谁来取，消息使用者只管从 MQ 中取消息而不管是谁发布的。这样发布者和使用者都不用知道对方的存在。



#### 公司在什么情况下会用消息队列

- 电商订单

  ​		想必同学们都点过外卖，点击下单后的业务逻辑可能包括：检查库存、生成单据、发红包、短信通知等，如果这些业务同步执行，完成下单率会非常低，如发红包，短信通知等不必要的流程，异步执行即可。

  此时使用MQ，可以在核心流程（扣减库存、生成订单记录）等完成后发送消息到MQ，快速结束本次流程。消费者拉取MQ消息时，发现红包、短信等消息时，再进行处理。

  场景：双11是购物狂节,用户下单后,订单系统需要通知库存系统,传统的做法就是订单系统调用库存系统的接口

  ![1570544131970](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570544131970.png)

  ```
  这种做法有一个缺点:
  
  	1、当库存系统出现故障时,订单就会失败。(这样马云将少赚好多好多钱钱。。。。)
  
  	2、订单系统和库存系统高耦合.
  
  ```

  引入消息队列：

  ![1570544177227](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570544177227.png)

  ```
  订单系统:用户下单后,订单系统完成持久化处理,将消息写入消息队列,返回用户订单下单成功。
  
  库存系统:订阅下单的消息,获取下单消息,进行库操作。 就算库存系统出现故障,消息队列也能保证消息的可靠投递,不会导致消息丢失(马云这下高兴了，钞票快快的来呀~~).
  ```

- 秒杀活动

  流量削峰一般在秒杀活动中应用广泛场景: 秒杀活动，一般会因为流量过大，导致应用挂掉,为了解决这个问题，一般在应用前端加入消息队列。 作用: 

  ​	1.可以控制活动人数，超过此一定阀值的订单直接丢弃(怪不得我一次秒杀都没抢到过....wtf？？？)

  ​    2.可以缓解短时间的高流量压垮应用(应用程序按自己的最大处理能力获取订单)

  ​    3.用户的请求，服务器接收到之后，写入消息队列，超过定义的阈值就直接丢弃请求，或者跳转错误页面。

  ​    4.业务系统取出队列中的消息，再做后续处理。



#### RabbitMQ 安装与使用

- 安装

  ```shell
  rabbitmq-server服务端
  
  1.下载centos源
  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
  2.下载epel源
  wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
  
  3.清空yum缓存并且生成新的yum缓存
  yum clean all
  yum makecache
  4.安装erlang
     $ yum -y install erlang
  5.安装RabbitMQ
     $ yum -y install rabbitmq-server
  6.启动(无用户名密码):
      systemctl start/stop/restart/status rabbitmq-server
      [root@MyHost ~]# netstat -tunlp   # 可查看epmd的进程端口，默认4369
  
  
  设置rabbitmq账号密码，以及角色权限设置
  
  # 设置新用户yugo 密码123
  sudo rabbitmqctl add_user yugo 123
  
  # 设置用户为administrator角色
  sudo rabbitmqctl set_user_tags yugo administrator
  
  # 设置权限，允许对所有的队列都有权限对何种资源具有配置、写、读的权限通过正则表达式来匹配，具体命令如下：set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
  sudo rabbitmqctl set_permissions -p "/" yugo ".*" ".*" ".*"
  
  #开启web界面rabbitmq
  rabbitmq-plugins enable rabbitmq_management
  
  #重启服务生效设置
  service rabbitmq-server start/stop/restart
  
  #访问web界面,并登陆
  http://server-name:15672/
  ```

 

- 使用

  ```go
  rabbitmq相关命令
  
  // 新建用户
  rabbitmqctl add_user {用户名} {密码}
  
  // 设置权限
  rabbitmqctl set_user_tags {用户名} {权限}
  
  // 查看用户列表
  rabbitmqctl list_users
  
  // 为用户授权
  添加 Virtual Hosts ：    
  rabbitmqctl add_vhost <vhost>    
  
  // 删除用户
  rabbitmqctl delete_user Username
  
  // 修改用户的密码
  rabbitmqctl change_password Username Newpassword
      
  // 删除 Virtual Hosts ：    
  rabbitmqctl delete_vhost <vhost>    
      
  // 添加 Users ：    
  rabbitmqctl add_user <username> <password>    
  rabbitmqctl set_user_tags <username> <tag> ...    
  rabbitmqctl set_permissions [-p <vhost>] <user> <conf> <write> <read>    
      
  // 删除 Users ：    
  delete_user <username>   
  
  // 使用户user1具有vhost1这个virtual host中所有资源的配置、写、读权限以便管理其中的资源
  rabbitmqctl  set_permissions -p vhost1 user1 '.*' '.*' '.*' 
  
  // 查看权限
  rabbitmqctl list_user_permissions user1
  
  rabbitmqctl list_permissions -p vhost1
  
  // 清除权限
  rabbitmqctl clear_permissions [-p VHostPath] User
  
  //清空队列步骤
  rabbitmqctl reset 
  需要提前关闭应用rabbitmqctl stop_app ，
  然后再清空队列，启动应用
  rabbitmqctl start_app
  此时查看队列rabbitmqctl list_queues
  
  查看所有的exchange：                              rabbitmqctl list_exchanges
  查看所有的queue：                                 rabbitmqctl list_queues
  查看所有的用户：                                   rabbitmqctl list_users
  查看所有的绑定（exchange和queue的绑定信息）：         rabbitmqctl list_bindings
  查看消息确认信息：
  rabbitmqctl list_queues name messages_ready messages_unacknowledged
  查看RabbitMQ状态，包括版本号等信息：rabbitmqctl status
  
  #开启web界面rabbitmqrabbitmq-plugins enable rabbitmq_management#访问web界面http://server-name:15672/
  ```



#### RabbitMQ组件解释

```
AMQP

AMQP协议是一个高级抽象层消息通信协议，RabbitMQ是AMQP协议的实现。它主要包括以下组件：
1.Server(broker): 接受客户端连接，实现AMQP消息队列和路由功能的进程。

2.Virtual Host:其实是一个虚拟概念，类似于权限控制组，一个Virtual Host里面可以有若干个Exchange和Queue，但是权限控制的最小粒度是Virtual Host

3.Exchange:接受生产者发送的消息，并根据Binding规则将消息路由给服务器中的队列。ExchangeType决定了Exchange路由消息的行为，例如，在RabbitMQ中，ExchangeType有direct、Fanout和Topic三种，不同类型的Exchange路由的行为是不一样的。

4.Message Queue：消息队列，用于存储还未被消费者消费的消息。

5.Message: 由Header和Body组成，Header是由生产者添加的各种属性的集合，包括Message是否被持久化、由哪个Message Queue接受、优先级是多少等。而Body是真正需要传输的APP数据。

6.Binding:Binding联系了Exchange与Message Queue。Exchange在与多个Message Queue发生Binding后会生成一张路由表，路由表中存储着Message Queue所需消息的限制条件即Binding Key。当Exchange收到Message时会解析其Header得到Routing Key，Exchange根据Routing Key与Exchange Type将Message路由到Message Queue。Binding Key由Consumer在Binding Exchange与Message Queue时指定，而Routing Key由Producer发送Message时指定，两者的匹配方式由Exchange Type决定。 

7.Connection:连接，对于RabbitMQ而言，其实就是一个位于客户端和Broker之间的TCP连接。

8.Channel:信道，仅仅创建了客户端到Broker之间的连接后，客户端还是不能发送消息的。需要为每一个Connection创建Channel，AMQP协议规定只有通过Channel才能执行AMQP的命令。一个Connection可以包含多个Channel。之所以需要Channel，是因为TCP连接的建立和释放都是十分昂贵的，如果一个客户端每一个线程都需要与Broker交互，如果每一个线程都建立一个TCP连接，暂且不考虑TCP连接是否浪费，就算操作系统也无法承受每秒建立如此多的TCP连接。RabbitMQ建议客户端线程之间不要共用Channel，至少要保证共用Channel的线程发送消息必须是串行的，但是建议尽量共用Connection。

9.Command:AMQP的命令，客户端通过Command完成与AMQP服务器的交互来实现自身的逻辑。例如在RabbitMQ中，客户端可以通过publish命令发送消息，txSelect开启一个事务，txCommit提交一个事务。
```



Python模块

```
// rabbitmq官方推荐的python客户端pika模块
pip3 install pika

注意：[root@MyHost MQ]# pip install pika==0.12
高级版本的函数的参数发生了改变，所以下面的代码在0.12版本使用
```



#### 应用场景

- 应用场景1：单发送单接收，生产-消费者模型

  ![1570545672624](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570545672624.png)

  ![1570545844115](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570545844115.png)

  生产者:send.py

  ```python
  #我们的第一个程序send.py将向队列发送一条消息。我们需要做的第一件事是建立与RabbitMQ服务器的连接
  
  #!/usr/bin/env python
  import pika
  # 创建凭证，使用rabbitmq用户密码登录
  # 去邮局取邮件，必须得验证身份
  credentials = pika.PlainCredentials("s14","123")
  # 新建连接，这里localhost可以更换为服务器ip
  # 找到这个邮局，等于连接上服务器，注意如果使用的云服务器，那么就使用内网ip
  connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.119.10',credentials=credentials))
  # 创建频道
  # 建造一个大邮箱，隶属于这家邮局的邮箱，就是个连接
  channel = connection.channel()
  # 声明一个队列，用于接收消息，队列名字叫“水许传”
  channel.queue_declare(queue='水许传')
  # 注意在rabbitmq中，消息想要发送给队列，必须经过交换(exchange)，初学可以使用空字符串交换(exchange='')，它允许我们精确的指定发送给哪个队列(routing_key=''),参数body值发送的数据
  channel.basic_publish(exchange='',
                        routing_key='水许传',
                        body='武松又去打老虎啦2')
  print("已经发送了消息")
  # 程序退出前，确保刷新网络缓冲以及消息发送给rabbitmq，需要关闭本次连接
  connection.close()
  ```
  
可以同时存在多个接受者，等待接收队列的消息，默认是轮训方式分配消息,接受者receive.py，可以运行多次，运行多个消费
  
```python
  import pika
  # 建立与rabbitmq的连接
  credentials = pika.PlainCredentials("s14","123")
  connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.119.10',credentials=credentials))
  channel = connection.channel()
  channel.queue_declare(queue="水许传")
  
  def callbak(ch,method,properties,body):
      print("消费者接收到了任务：%r"%body.decode("utf8"))
  # 有消息来临，立即执行callbak，没有消息则夯住，等待消息
  # 老百姓开始去邮箱取邮件啦，队列名字是水许传
  channel.basic_consume(callbak,queue="水许传",no_ack=True)
  # 开始消费，接收消息
  channel.start_consuming()
  ```
  

- 单生产者多消费者，默认是轮询消费机制，即每个消费者一人一次

  ```
  使用场景：一个发送端，多个接收端，如分布式的任务派发。为了保证消息发送的可靠性，不丢失消息，使消息持久化了。同时为了防止接收端在处理消息时down掉，只有在消息处理完成后才发送ack消息。
  ```



#### 确认机制：保证消息正确被处理

ACK机制用于保证消费者如果拿了队列的消息，`客户端`处理时出错了，那么队列中仍然还存在这个消息，提供下一位消费者继续取

no-ack:`不确认机制`也就是说每次消费者接收到数据后，不管是否处理完毕，rabbitmq-server都会把这个消息标记完成，从队列中删除

流程：

```
1.生产者无须变动，发送消息

2.消费者如果no_ack=True啊，数据消费后如果出错就会丢失;反之no_ack=False，数据消费如果出错，数据也不会丢失

3.ack机制在消费者代码中演示


官网资料：http://www.rabbitmq.com/tutorials/tutorial-two-python.html
默认情况下，生产者发送数据给队列，消费者取出消息后，数据将被清除。
特殊情况，如果消费者处理过程中，出现错误，数据处理没有完成，那么这段数据将从队列丢失
```

生产者.py `只负责发送数据即可，无须变动`

```python
#!/usr/bin/env python
import pika
# 创建凭证，使用rabbitmq用户密码登录
# 去邮局取邮件，必须得验证身份
credentials = pika.PlainCredentials("s14","123")
# 新建连接，这里localhost可以更换为服务器ip
# 找到这个邮局，等于连接上服务器
connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.119.10',credentials=credentials))
# 创建频道
# 建造一个大邮箱，隶属于这家邮局的邮箱，就是个连接
channel = connection.channel()

# 新建一个hello队列，用于接收消息
# 这个邮箱可以收发各个班级的邮件，通过
channel.queue_declare(queue='金品没')
# 注意在rabbitmq中，消息想要发送给队列，必须经过交换(exchange)，初学可以使用空字符串交换(exchange='')，它允许我们精确的指定发送给哪个队列(routing_key=''),参数body值发送的数据
channel.basic_publish(exchange='',
                      routing_key='金品没',
                      body='潘金莲又出去。。。')
print("已经发送了消息")
# 程序退出前，确保刷新网络缓冲以及消息发送给rabbitmq，需要关闭本次连接
connection.close()
```

消费者.py`给与ack回复`

拿到消息必须给rabbitmq服务端回复ack信息，否则消息不会被删除，防止客户端出错，数据丢失

```python
import pika

credentials = pika.PlainCredentials("s14","123")
connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.119.10',credentials=credentials))
channel = connection.channel()

# 声明一个队列(创建一个队列)
channel.queue_declare(queue='金品没')

def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body.decode("utf-8"))
    #下面是主动抛错，模拟出错，如果出错，下面的代码将不会执行，即不会清除消息，如果抛错了no_ack=True，消息就会被清除了，所以不确认机制=False，也就是需要确认
    # int('asdfasdf')
    
    # 我告诉rabbitmq服务端，我已经取走了消息
    # 回复方式在这，告诉服务端，我正确消费了，你可以正确标记清除了
    ch.basic_ack(delivery_tag=method.delivery_tag)
# 关闭no_ack，代表给与服务端ack回复，确认给与回复
channel.basic_consume(callback,queue='金品没',no_ack=False)

channel.start_consuming()
```



#### 消息持久化

原因：

```
1.执行生产者，向队列写入数据，产生一个新队列queue
2.重启服务端，队列丢失3.开启生产者数据持久化后，重启rabbitmq，队列不丢失4.依旧可以读取数据
```

​		消息的可靠性是RabbitMQ的一大特色，那么RabbitMQ是如何保证消息可靠性的呢——消息持久化。 为了保证RabbitMQ在退出或者crash等异常情况下数据没有丢失，需要将queue，exchange和Message都持久化。

生产者.py

```python
import pika
# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters('123.206.16.61'))
# 有密码
credentials = pika.PlainCredentials("s14","123")
connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.119.10',credentials=credentials))
channel = connection.channel()
# 声明一个队列(创建一个队列)
# 默认此队列不支持持久化，如果服务挂掉，数据丢失
# durable=True 开启持久化，必须新开启一个队列，原本的队列已经不支持持久化了
'''
实现rabbitmq持久化条件
 	1、delivery_mode=2
	2、使用durable=True声明queue是持久化
 
'''
channel.queue_declare(queue='LOL',durable=True)  #实现队列持久化
channel.basic_publish(exchange='',
                      routing_key='LOL', # 消息队列名称
                      body='德玛西亚万岁',
                      # 支持数据持久化
                      properties=pika.BasicProperties(
                          delivery_mode=2,#代表消息是持久的  2
                      )
                      )
print('消息成功发送')
connection.close()
```

消费者.py

```python
import pika
credentials = pika.PlainCredentials("s14","123")
connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.119.10',credentials=credentials))
channel = connection.channel()
# 确保队列持久化
channel.queue_declare(queue='LOL',durable=True)

'''
必须确保给与服务端消息回复，代表我已经消费了数据，否则数据一直持久化，不会消失
'''
def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body.decode("utf-8"))
    # 模拟代码报错
    # int('asdfasdf')    # 此处报错，没有给予回复，保证客户端挂掉，数据不丢失
   
    # 告诉服务端，我已经取走了数据，否则数据一直存在
    ch.basic_ack(delivery_tag=method.delivery_tag)
# 关闭no_ack，代表给与回复确认
channel.basic_consume(callback,queue='LOL',no_ack=False)
channel.start_consuming()
```



#### Exchange模型

rabbitmq发送消息首先是发给exchange，然后再通过exchange发送消息给队列（queue）

exchange有四种模式

- fanout

  exchange将消息发送给和该exchange连接的所有queue；也就是所谓的广播模式；此模式下忽略routing_key；

- direct

  路由模式，通过routing_key将消息发送给对应的queue; 如下面这句即可设置exchange为direct模式，只有routing_key为“black”时才将其发送到队列queue_name； `channel.queue_bind(exchange=exchange_name,queue=queue_name,routing_key='black')`

  ![1570584131569](C:\Users\wanglixing\Desktop\知识点复习\Linux\RabbitMQ 消息队列.assets\1570584131569.png)

  在上图中，Q1和Q2可以绑定同一个key，如绑定routing_key=‘KeySame’，那么收到routing_key为KeySame的消息时将会同时发送给Q1和Q2，退化为广播模式；

- top

  topic模式类似于direct模式，只是其中的routing_key变成了一个有“.”分隔的字符串，“.”将字符串分割成几个单词，每个单词代表一个条件

- headers

  headers类型的Exchange不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。

  官方教程：http://www.rabbitmq.com/tutorials/tutorial-three-python.html

  ![1570584213093](C:\Users\wanglixing\Desktop\知识点复习\Linux\RabbitMQ 消息队列.assets\1570584213093.png)

  发布订阅和简单的消息队列区别在于，发布订阅会将消息发送给所有的订阅者，而消息队列中的数据被消费一次便消失。所以，RabbitMQ实现发布和订阅时，会为每一个订阅者创建一个队列，而发布者发布消息时，会将消息放置在所有相关队列中。

- 实例

  生成者.py

  ```python
  # -*- coding: utf-8 -*-
  # __author__ = "yugo"
  
  import pika
  credentials = pika.PlainCredentials("root","123")
  connection = pika.BlockingConnection(pika.ConnectionParameters('123.206.16.61',credentials=credentials))
  channel = connection.channel()
  
  # 指定exchange
  channel.exchange_declare(exchange='m1',exchange_type='fanout')
  
  channel.basic_publish(exchange='m1',
                        routing_key='',# 这里不再指定队列，由exchange分配,如果是fanout模式，每一个队列放一份
                        body='haohaio')
  
  connection.close()
  ```

  消费者.py:可以运行多次，运行多个消费者，等待消息

  ```python
  import pika
  
  credentials = pika.PlainCredentials("root","123")
  connection = pika.BlockingConnection(pika.ConnectionParameters('123.206.16.61',credentials=credentials))
  channel = connection.channel()
  
  # exchange='m1',exchange(秘书)的名称
  # exchange_type='fanout' , 秘书工作方式将消息发送给所有的队列
  channel.exchange_declare(exchange='m1',exchange_type='fanout')
  
  # 随机生成一个队列
  result = channel.queue_declare(exclusive=True)
  queue_name = result.method.queue
  
  # 让exchange和queque进行绑定.
  channel.queue_bind(exchange='m1',queue=queue_name)
  
  def callback(ch, method, properties, body):
      print("消费者接受到了任务: %r" % body)
  
  channel.basic_consume(callback,queue=queue_name,no_ack=True)
  
  channel.start_consuming()
  ```



#### 关键字发布Exchange

​		之前事例，发送消息时明确指定某个队列并向其中发送消息，RabbitMQ还支持根据关键字发送，即：队列绑定关键字，发送者将数据根据关键字发送到消息exchange，exchange根据 关键字 判定应该将数据发送至指定队列。

![1570584365035](C:\Users\wanglixing\Desktop\知识点复习\Linux\RabbitMQ 消息队列.assets\1570584365035.png)

- 消费者1.py：路由关键字是sb,alex

  ```python
  # -*- coding: utf-8 -*-
  # __author__ = "maple"
  import pika
  
  credentials = pika.PlainCredentials("root","123")
  connection = pika.BlockingConnection(pika.ConnectionParameters('123.206.16.61',credentials=credentials))
  channel = connection.channel()
  
  # exchange='m1',exchange(秘书)的名称
  # exchange_type='fanout' , 秘书工作方式将消息发送给所有的队列
  channel.exchange_declare(exchange='m2',exchange_type='direct')
  
  # 随机生成一个队列,队列退出时，删除这个队列
  result = channel.queue_declare(exclusive=True)
  queue_name = result.method.queue
  
  # 让exchange和queque进行绑定，只要
  channel.queue_bind(exchange='m2',queue=queue_name,routing_key='alex')
  channel.queue_bind(exchange='m2',queue=queue_name,routing_key='sb')
  
  def callback(ch, method, properties, body):
      print("消费者接受到了任务: %r" % body)
  
  channel.basic_consume(callback,queue=queue_name,no_ack=True)
  
  channel.start_consuming()
  ```

  消费者2.py：路由关键字sb

  ```python
  # -*- coding: utf-8 -*-
  # __author__ = "maple"
  import pika
  
  credentials = pika.PlainCredentials("root","123")
  connection = pika.BlockingConnection(pika.ConnectionParameters('123.206.16.61',credentials=credentials))
  channel = connection.channel()
  
  # exchange='m1',exchange(秘书)的名称
  # exchange_type='fanout' , 秘书工作方式将消息发送给所有的队列
  channel.exchange_declare(exchange='m2',exchange_type='direct')
  
  # 随机生成一个队列
  result = channel.queue_declare(exclusive=True)
  queue_name = result.method.queue
  
  # 让exchange和queque进行绑定.
  channel.queue_bind(exchange='m2',queue=queue_name,routing_key='sb')
  
  def callback(ch, method, properties, body):
      print("消费者接受到了任务: %r" % body)
  
  channel.basic_consume(callback,queue=queue_name,no_ack=True)
  
  channel.start_consuming()
  ```

  生产者.py: 发送消息给匹配的路由，sb或者alex

  ```
  # -*- coding: utf-8 -*-
  # __author__ = "yugo"
  
  import pika
  credentials = pika.PlainCredentials("root","123")
  connection = pika.BlockingConnection(pika.ConnectionParameters('123.206.16.61',credentials=credentials))
  channel = connection.channel()
  
  # 路由模式的交换机会发送给绑定的key和routing_key匹配的队列
  channel.exchange_declare(exchange='m2',exchange_type='direct')
  # 发送消息，给有关sb的路由关键字
  channel.basic_publish(exchange='m2',
                        routing_key='sb',
                        body='aaaalexlaolelaodi')
  
  connection.close()
  ```

  