

# 1 celery 简要概述

Celery是一个简单，灵活，可靠的分布式系统，用于处理大量消息，同时为操作提供维护此类系统所需的工具。
它是一个任务队列，专注于实时处理，同时还支持任务调度。

celery 的优点：

 - 简单：celery的 配置和使用还是比较简单的, 非常容易使用和维护和不需要配置文件
 - 高可用：当任务执行失败或执行过程中发生连接中断，celery 会自动尝试重新执行任务

- 如果连接丢失或发生故障，worker和client 将自动重试，并且一些代理通过主/主或主/副本复制方式支持HA。

- 快速：一个单进程的celery每分钟可处理上百万个任务

- 灵活： 几乎celery的各个组件都可以被扩展及自定制


---


## celery 可以做什么?
典型的应用场景, 比如：
- 异步发邮件发短信 , 一般发邮件比较耗时的操作,需要及时返回给前端,这个时候 只需要提交任务给celery 就可以了.之后 由worker 进行发邮件的操作 .
- 比如有些 跑批接口的任务,需要耗时比较长,这个时候 也可以做成异步任务 .
- 定时调度任务等

# celery 的核心模块

```python
生产者（任务，函数）
    @app.task
    def celery_send_sms_code(mobie,code):

        CCP().send_template_sms(mobie,[code,5],1)


    app.autodiscover_tasks(['celery_tasks.sms'])
消费者
    celery -A proj worker -l INFO

    在虚拟环境下执行指令
    celery -A celery实例的脚本路径 worker -l INFO

    本案例使用
    celery -A celery_tasks.main worker -l INFO

队列（中间人、经纪人）
    #2. 设置broker
    # 我们通过加载配置文件来设置broker
    app.config_from_object('celery_tasks.config')

    # 配置信息 key=value
    # 我们指定 redis为我们的broker(中间人，经纪人，队列)
    broker_url="redis://127.0.0.0:6379/15"

Celery -- 将这3者实现了。

    # 0. 为celery的运行 设置Django的环境
    import os
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'meiduo_mall.settings')

    # 1. 创建celery实例
    from celery import Celery
    # 参数1： main 设置脚本路径就可以了。 脚本路径是唯一的
    app=Celery('celery_tasks')
```

## celery 的5个角色
代码如下（示例）：
**Task**

就是任务，有异步任务和定时任务

**Broker**

中间人，接收生产者发来的消息即Task，将任务存入队列。任务的消费者是Worker。

Celery本身不提供队列服务，推荐用Redis或RabbitMQ实现队列服务。

**Worker**

执行任务的单元，它实时监控消息队列，如果有任务就获取任务并执行它。

**Beat**

定时任务调度器，根据配置定时将任务发送给Broker。

**Backend**

用于存储任务的执行结果。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a8a0c88bcfec402816a67e63971daacf.png)


## 引入库

```python
pip install celery
```


## celery 和django如何结合起来
### 项目结构
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed289ab791fb5a84ecac7ba14e058725.png)

### 项目入口 文件 main.py


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/36206438e5327a01253c17be1d080ea6.png)

### django实例的创建

```python
# 設置django環境
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mall_django.settings')

# 創建celery實例
from celery import Celery

# main設置脚本路徑
app = Celery(main='celery_tasks')
```


### task装饰绑定任务

 這個函數必須要讓celery的實例的task裝飾器 裝飾
 需要celery自動檢測指定包的任務



```python
from libs.yuntongxun.sms import CCP
from celery_tasks.main import app

@app.task
def celery_send_sms_code(mobile, code):
    print('绑定任务')
```

###  worker 启动入口
启动 Worker，监听 Broker 中是否有任务
启动 worker 可以通过 命令:

```python
celery -A celery_tasks.main worker -l INFO
```



















---

# 总结
本文简单介绍了 celery 的基本的功能 , 以及celery 能够处理的任务特点,以及可以和 django结合起来使用. 简单分析了 celery 的工作机制 . 当然 如果想要深入了解 celery,可以 参考 celery的官方文档.









# 参考链接
 celery 文档 [http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html](http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html)
