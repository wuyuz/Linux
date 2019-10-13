## Celery 


​		Celery 是一个强大的分布式任务队列，它可以让任务的执行完全脱离主程序，甚至可以被分配到其他主机上运行。我们通常使用它来实现异步任务（ async task ）和定时任务（ crontab ）。 异步任务比如是发送邮件、或者文件上传, 图像处理等等一些比较耗时的操作 ，定时任务是需要在特定时间执行的任务。它的架构组成如下图：

![1570589515816](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570589515816.png)



#### 任务队列

任务队列是一种跨线程、跨机器工作的一种机制. 任务队列中包含称作任务的工作单元。有专门的工作进程持续不断的监视任务队列，并从中获得新的任务并处理.

- 任务模块
  		包含异步任务和定时任务。其中，异步任务通常在业务逻辑中被触发并发往任务队列，而定时任务由 Celery Beat 进程周期性地将任务发往任务队列。

- 消息中间件 Broker
  		Broker ，即为任务调度队列，接收任务生产者发来的消息（即任务），将任务存入队列。 Celery 本身不提供队列服务，官方推荐使用 RabbitMQ 和 Redis 等。

- 任务执行单元 Worker
  		Worker 是执行任务的处理单元，它实时监控消息队列，获取队列中调度的任务，并执行它。

- 任务结果存储 Backend
  		Backend 用于存储任务的执行结果，以供查询。同消息中间件一样，存储也可使用 RabbitMQ, Redis 和 MongoDB 等。



#### 使用 Celery 实现异步任务的步骤：

> (1) 创建一个 Celery 实例
> (2) 启动 Celery Worker ，通过delay() 或 apply_async()(delay 方法封装了 apply_async, apply_async支持更多的参数 ) 将任务发布到broker
> (3) 应用程序调用异步任务
> (4)存储结果 （发布的任务需要return才会有结果，否则为空）

Celery Beat：任务调度器，Beat进程会读取配置文件的内容，周期性地将配置中到期需要执行的任务发送给任务队列



#### 使用 Celery 实现定时任务的步骤：

> (1) 创建一个 Celery 实例
> (2) 配置文件中配置任务 ，发布任务 celery A xxx beat
> (3) 启动 Celery Worker
> (4) 存储结果



#### 安装Celery

- ```
  pip3 install -U Celery  # 注意python的版本
  
  [root@MyHost MQ]# celery --version
  4.2.0 (windowlicker)
  ```

  也可从官方直接下载安装包:https://pypi.python.org/pypi/celery/

  ```
  tar xvfz celery-0.0.0.tar.gz  
  cd celery-0.0.0
  python setup.py build
  python setup.py install  #注意python的版本
  ```



#### 启动第一个简单项目

- 注意点：

  ```shell
  1、一定要注意redis模块的版本： Redis transport requires redis-py versions 3.2.0 or later. You have 2.10.6
  	[root@MyHost MQ]# pip3 install redis==3.2.0
  2、如果是在linux上启动celery命令报错：versions 3.2.0 or later. You have 2.10.6，需要降低版本
  	# kombu版本
  	pip3 install kombu==4.2.0
  	# celery版本
  	pip3 install celery==4.1.1
  	
  3、报错：'float' object has no attribute 'items'，可能是因为redis模块版本不支持：
  	[root@MyHost MQ]# pip3 install redis==2.10.6
  	
  4、使用celery出现from . import async, base SyntaxError: invalid syntax错误
  	[root@MyHost MQ]#pip3 install --upgrade https://github.com/celery/celery/tarball/master
  ```

- 在随便一个目录下创建tasks.py文件，用来写celery项目：使用celery第一件要做的最为重要的事情是需要先创建一个Celery实例，我们一般叫做celery应用，或者更简单直接叫做一个app。

  ​    app应用是我们使用celery所有功能的入口，比如创建任务，管理任务等，在使用celery的时候，app必须能够被其他的模块导入。我们首先创建vim tasks.py模块, 其内容为:

  ```python
  from celery import Celery
  # 我们这里案例使用redis作为broker,xxx为redis的密码，前提得启动redis-server，且注意密码是否有效，未写入配置文件启动
  app = Celery('demo', broker='redis://:xxx@127.0.0.1/1')
  
  # 创建任务函数
  @app.task
  def my_task():
      print("任务函数正在执行....")
  ```

   		Celery第一个参数是给其设定一个名字， 第二参数我们设定一个中间人broker, 在这里我们使用Redis作为中间人。my_task函数是我们编写的一个任务函数， 通过加上装饰器app.task, 将其注册到broker的队列中。
  现在我们在创建一个worker， 等待处理队列中的任务.打开终端，cd到tasks.py同级目录中，执行命令:

  ```shell
  celery -A tasks worker --loglevel=info  # tasks是我们的celery实例文件名字 tasks.py，一定要进入task.py文件中执行命令
  ```

  ![1570590618403](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570590618403.png)

  ​		

  ​		任务加入到broker队列中，以便刚才我们创建的celery workder服务器能够从队列中取出任务并执行。如何将任务函数加入到队列中，可使用delay()。

  

  

- 存储调用结果/状态：

​      我们通过worker的控制台，可以看到我们的任务被worker处理。调用一个任务函数，将会返回一个AsyncResult对象，这个对象可以用来检查任务的状态或者获得任务的返回值。

如果我们想跟踪任务的状态，Celery需要将结果保存到某个地方。有几种保存的方案可选:SQLAlchemy、Django ORM、Memcached、 Redis、RPC (RabbitMQ/AMQP)。例子：我们仍然使用Redis作为存储结果的方案，任务结果存储配置我们通过Celery的backend参数来设定。我们将tasks模块修改如下:

```python
from  celery import Celery

# 我们这里案例使用redis作为broker,xxx为redis的密码
app = Celery('demo',
             broker='redis://:123@127.0.0.1/1',
             backend='redis://:123@127.0.0.1/2')

# 创建任务函数
@app.task
def my_task(a,b):
    print("任务函数正在执行....")
    return a + b
```

![1570594100024](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570594100024.png)

我们给Celery增加了backend参数，指定redis作为结果存储,并将任务函数修改为两个参数，并且有返回值。 更多关于AsyncResult对象信息，请参阅下列网址:http://docs.celeryproject.org/en/latest/reference/celery.result.html#module-celery.result



#### celery 配置

Celery使用简单，配置也非常简单。Celery有很多配置选项能够使得celery能够符合我们的需要，但是默认的几项配置已经足够应付大多数应用场景了。

配置信息可以直接在app中设置，或者通过专有的配置模块来配置。

- 直接通过 app 来配置

  ```
  from celery import Celery
  app = Celery('demo')
  # 增加配置
  app.conf.update(
      broker='redis://:123@127.0.0.1/1',
      backend='redis://:123@127.0.0.1/2'
  )
  ```

- 专用配置文件

  对于比较大的项目，我们建议配置信息作为一个单独的模块。我们可以通过调用app的函数来告诉Celery使用我们的配置模块。配置模块的名字我们取名为celeryconfig.py, 这个名字不是固定的，我们可以任意取名，建议这么做。我们必须保证配置模块能够被导入

  下面我们在tasks.py模块 同级目录下创建配置模块celeryconfig.py:

  ```
  result_backend = 'redis://:332572@127.0.0.1:6379/2'
  broker_url = 'redis://:332572@127.0.0.1:6379/1'
  ```

  task.py文件修改

  ```python
  tasks.py文件修改为:
  from celery import Celery
  import celeryconfig
   
  # 我们这里案例使用redis作为broker
  app = Celery('demo')
   
  # 从单独的配置模块中加载配置
  app.config_from_object('celeryconfig')
  ```

- 回调时参数讲解

  ```python
  将任务加入到broker队列中，以便刚才我们创建的celery workder服务器能够从队列中取出任务并执行。如何将任务函数加入到队列中，可使用delay()函数或者apply_asyn()函数，后者会提供更多任务执行的控制。
  
  调用任务，可使用delay()方法: my_task.delay(2, 2)
  
  也可以使用apply_async()方法，该方法可让我们设置一些任务执行的参数，例如，任务多久之后才执行，任务被发送到那个队列中等等.
  my_task.apply_async((2, 2), queue='my_queue', countdown=10)
  	
  任务my_task将会被发送到my_queue队列中，并且在发送10秒之后执行。如果我们直接执行任务函数，将会直接执行此函数在当前进程中，并不会向broker发送任何消息。
  
  无论是delay()还是apply_async()方式都会返回AsyncResult对象，方便跟踪任务执行状态，但需要我们配置result_backend.
  ```

  

#### celery 项目

- 项目目录结构

  ```
  testCelery/
  ├── celery
  ├── celeryconfig.py
  ├── celery.py
  ├── __pycache__
  │   ├── celeryconfig.cpython-37.pyc
  │   ├── celery.cpython-37.pyc
  │   └── tasks.cpython-37.pyc
  └── celery.py
  
  #主要有celeryconfig.py、celery.py、celery.py，其他文件都是后面自动生成
  ```

- celery.py, 注意必须叫这个名字，否则启动时无法检测到

  ```
  from celery import Celery
  
  # 创建celery实例
  app = Celery('demo')
  app.config_from_object('testCelery.celeryconfig')
  
  #自动搜索任务
  app.autodiscover_tasks(['testCelery'])
  ```

- celeryconfig.py

  ```
  BROKER_URL = 'redis://:123@127.0.0.1:6379/1'
  CELERY_RESULT_BACKEND = 'redis://123@127.0.0.1:6379/2'
  ```

- tasks.py

  ```python
  from testCelery.celery import app as celery_app
  
  # 创建任务函数
  @celery_app.task
  def my_task1():
      print("任务函数(my_task1)正在执行....")
  
  @celery_app.task
  def my_task2():
      print("任务函数(my_task2)正在执行....")
  
  @celery_app.task
  def my_task3():
      print("任务函数(my_task3)正在执行....")
  ```

  在项目上级目录启动：

  ```
  [root@MyHost MQ]# celery -A testCelery worker -l info
  ```

  ![1570600752221](C:\Users\wanglixing\AppData\Roaming\Typora\typora-user-images\1570600752221.png)

- 查看效果

  ```
  >>> import sys,os
  >>> sys.path.append(os.getcwd())
  >>> from testCelery.tasks import my_task1
  >>> from testCelery.tasks import my_task2,my_task3
  >>> my_task.delay()
  ```



