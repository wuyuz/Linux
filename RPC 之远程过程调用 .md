## RPC 之远程过程调用



**将一个函数运行在远程计算机上并且等待获取那里的结果，这个称作远程过程调用（Remote Procedure Call）或者 RPC。**RPC**是一个计算机通信协议。**

比喻：

```
将计算机服务运行理解为厨师做饭，厨师想做一个小葱拌豆腐，厨师需要洗小葱、切豆腐、调汁、凉拌。他一个人完成所有的事，如同古老的集中式应用，一台计算机做所有的事。

制作小葱拌豆腐{
    厨师>洗小葱>切豆腐>凉拌
}
```

`rpc`应用场景：

```
而如今，饭店做大了，有钱了，专职分工来干活，不再是厨师单打独斗，备菜师傅准备小葱、豆腐，切菜师傅切小葱、豆腐，厨师只负责调味，完成食品。

制作小葱拌豆腐{    
	备菜师>洗菜    
	切菜师>切菜    
	厨师>调味
	}
	
此时一件事好多人在做，厨师就得和其他人沟通，通知备菜、洗菜师傅的这个动作就是远程过程调用（RPC）。这个过程在计算机系统中，一个电商的下单过程，涉及物流、支付、库存、红包等多个系统，多个系统又在多个服务器上，由不同的技术团队负责，整个下单过程，需要所有团队进行远程调用。

下单{
    库存>减少库存
    支付>扣款
    红包>减免红包
    物流>生成订单
}
```



#### 到底什么是RPC

```
	rpc指的是在计算机A上的进程，调用另外一台计算机B的进程，A上的进程被挂起，B上的被调用进程开始执行后，产生返回值给A，A继续执行。调用方可以通过参数将信息传递给被调用方，而后通过返回结果得到信息，这个过程对于开发人员来说是透明的。如同厨师一样，服务员把菜单给后厨，厨师告诉洗菜人，备菜人，开始工作，完成工作后，整个过程对于服务员是透明的，他完全不用管后厨是怎么把菜做好的。
```

由于服务在不同的机器上，远程调用必经网络通信，调用服务必须写一坨网络通信代码，很容易出错且很复杂，因此就出现了RPC框架。

```
阿里巴巴的   Dubbo     java
新浪的       Motan    java
谷歌的       gRPC     多语言
Apache      thrift   多语言
```



Python实现RPC：利用RabbitMQ构建一个RPC系统，包含了客户端和RPC服务器，依旧使用pika模块

Callback queue 回调队列：

​		一个客户端向服务器发送请求，服务器端处理请求后，将其处理结果保存在一个存储体中。而客户端为了获得处理结果，那么客户在向服务器发送请求时，同时发送一个回调队列地址`reply_to`。

Correlation id 关联标识：

​		一个客户端可能会发送多个请求给服务器，当服务器处理完后，客户端无法辨别在回调队列中的响应具体和那个请求时对应的。为了处理这种情况，客户端在发送每个请求时，同时会附带一个独有`correlation_id`属性，这样客户端在回调队列中根据`correlation_id`字段的值就可以分辨此响应属于哪个请求。

```
客户端发送请求：某个应用将请求信息交给客户端，然后客户端发送RPC请求，在发送RPC请求到RPC请求队列时，客户端至少发送带有reply_to以及correlation_id两个属性的信息服务器端工作流： 等待接受客户端发来RPC请求，当请求出现的时候，服务器从RPC请求队列中取出请求，然后处理后，将响应发送到reply_to指定的回调队列中客户端接受处理结果： 客户端等待回调队列中出现响应，当响应出现时，它会根据响应中correlation_id字段的值，将其返回给对应的应用
```

过程

```
1.启动rpc客户端，等待接收数据到来，来了之后就进行处理，再将结果丢进队列
2.启动rpc服务端，发起请求
```



- 结合RabbitMQ实现RPC

  rpc_server.py
  
  ```
  import pika
  import uuid
  class FibonacciRpcClient(object):
      def __init__(self):
          # 客户端启动时，创建回调队列，会开启会话用于发送RPC请求以及接受响应
          # 建立连接，指定服务器的ip地址
          self.connection = pika.BlockingConnection(pika.ConnectionParameters(
              host='192.168.119.10'))
          # 建立一个会话，每个channel代表一个会话任务
          self.channel = self.connection.channel()
  
          # 声明回调队列，再次声明的原因是，服务器和客户端可能先后开启，该声明是幂等的，多次声明，但只生效一次
          #exclusive=True 参数是指只对首次声明它的连接可见
          #exclusive=True 会在连接断开的时候，自动删除
          result = self.channel.queue_declare(exclusive=True)
          # 将次队列指定为当前客户端的回调队列
          self.callback_queue = result.method.queue
          # 客户端订阅回调队列，当回调队列中有响应时，调用`on_response`方法对响应进行处理;
          self.channel.basic_consume(self.on_response, no_ack=True,
                                     queue=self.callback_queue)
  
  
      # 对回调队列中的响应进行处理的函数
      def on_response(self, ch, method, props, body):
          if self.corr_id == props.correlation_id:
              self.response = body
  
      # 发出RPC请求
      # 例如这里服务端就是一个切菜师傅，菜切好了，需要传递给洗菜师傅，这个过程是发送rpc请求
      def call(self, n):
          # 初始化 response
          self.response = None
          # 生成correlation_id 关联标识，通过python的uuid库，生成全局唯一标识ID，保证时间空间唯一性
          self.corr_id = str(uuid.uuid4())
          # 发送RPC请求内容到RPC请求队列`s14rpc`，同时发送的还有`reply_to`和`correlation_id`
          self.channel.basic_publish(exchange='',
                                     routing_key='s14rpc',
                                     properties=pika.BasicProperties(
                                         reply_to=self.callback_queue,
                                         correlation_id=self.corr_id,
                                     ),
                                     body=str(n))
          while self.response is None:
              self.connection.process_data_events()
          return int(self.response)
  
  # 建立客户端
  fibonacci_rpc = FibonacciRpcClient()
  
  # 发送RPC请求，丢进rpc队列，等待客户端处理完毕，给与响应
  print("发送了请求sum(99)")
  response = fibonacci_rpc.call(99)
  
  print("得到远程结果响应%r" % response)
  ```
  
  rpc_client.py
  
  ```
  import pika
  # 建立连接，服务器地址为localhost，可指定ip地址
  connection = pika.BlockingConnection(pika.ConnectionParameters(
      host='192.168.119.10'))
  # 建立会话
  channel = connection.channel()
  # 声明RPC请求队列
  channel.queue_declare(queue='s14rpc')
  
  # 模拟一个进程，例如切菜师傅，等着洗菜师傅传递数据
  def sum(n):
      n+=100
      return n
  # 对RPC请求队列中的请求进行处理
  
  
  def on_request(ch, method, props, body):
      print(body,type(body))
      n = int(body)
      print(" 正在处理sum(%s)" % n)
      # 调用数据处理方法
      response = sum(n)
      # 将处理结果(响应)发送到回调队列
      ch.basic_publish(exchange='',
                       # reply_to代表回复目标
                       routing_key=props.reply_to,
                       # correlation_id（关联标识）：用来将RPC的响应和请求关联起来。
                       properties=pika.BasicProperties(correlation_id= \
                                                           props.correlation_id),
                       body=str(response))
      ch.basic_ack(delivery_tag=method.delivery_tag)
  
  # 负载均衡，同一时刻发送给该服务器的请求不超过一个
  channel.basic_qos(prefetch_count=1)
  channel.basic_consume(on_request, queue='s14rpc')
  print("等待接收rpc请求")
  
  
  #开始消费
  channel.start_consuming()
   
  ```
  
   