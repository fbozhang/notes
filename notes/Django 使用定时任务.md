
# 前言

Windows 系统进行 Django 开发工作，然后原来使用的django-crontab插件没办法在Windows系统上面进行定时任务。因此又想了其他方式来实现定时任务。下面就来说说这些方案的优缺点。

---



# 使用django-crontab插件来实现定时任务
## 安装庫

```python
pip install django-crontab
```

## 注冊app
在settings.py中的 **INSTALLED_APPS**注冊app


```python
INSTALLED_APPS = [
    ...
    'django_crontab'
]
```
## 在settings.py中配置定时任务

```python
"""
# 元素的第一个参数是 频次(多久一次)
*  *  *  * *
分 时 日 月 周    命令

M: 分钟（0-59）。每分钟用 * 或者 */1 表示
H：小时（0-23）。（0表示0点）
D：天（1-31）。
m: 月（1-12）。
d: 一星期内的天（0~6，0为星期天）。

*/5 * * * * ---- 每5分鐘執行一次

# 元素的第二个参数是 定时任务（函数）

finally run this command to add all defined jobs from CRONJOBS to crontab (of the user which you are running this command with):
在終端運行: python manage.py crontab add

show current active jobs of this project:
在終端運行: python manage.py crontab show

removing all defined jobs is straight forward:
在終端運行: python manage.py crontab remove
"""
crontab 只在Linux下才能使用
CRONJOBS = [
    ('*/5 * * * *', 'myapp.cron.my_scheduled_job', ' >> /tmp/logs/confdict_handle.log')
]
```
## 编写定时任务方法
在子應用新建 **crontabs.py** 文件，寫入定时任务


```python
import datetime

# 定时任务
def confdict_handle():
    try:
        loca_time = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        print('本地时间：'+str(loca_time))
    except Exception as e:
        print('发生错误，错误信息为：', e)
```


## 使用&运行

开启定时器

```python
python manage.py crontab add
```

查看开启的定时器

```python
python manage.py crontab show
```

关闭定时器

```python
python manage.py crontab remove
```



## 优缺点

**优点：**
	简单、方便、易于管理和django服务是分离的，不会影响到django对外提供的web服务。
**缺点：**
无法在Windows平台上运行
官方文檔：
<img width="210" alt="image" src="https://github.com/user-attachments/assets/cca7b480-3004-438e-81ed-5a6043177e2f">


就算在Linux系统上，也可能出现运行了没有效果的消息，至今未知原因

# 使用django-apscheduler插件实现定时任务

## 安装庫

```python
pip install django-apscheduler
```

## 注冊app
修改settings.py

```python
INSTALLED_APPS = (
  ...
  "django_apscheduler",
)
```


## 迁移数据库
因为django-apscheduler会创建表来存储定时任务的一些信息，所以将app加入之后需要迁移数据


```python
python manage.py migrate
```
## 完整示例 在views.py中增加你的定时任务代码

```python
from apscheduler.schedulers.background import BackgroundScheduler # 使用它可以使你的定时任务在后台运行
from django_apscheduler.jobstores import DjangoJobStore, register_events, register_job
import time
'''
date：在您希望在某个特定时间仅运行一次作业时使用
interval：当您要以固定的时间间隔运行作业时使用
cron：以crontab的方式运行定时任务
minutes：设置以分钟为单位的定时器
seconds：设置以秒为单位的定时器
'''

try:
    scheduler = BackgroundScheduler()
    scheduler.add_jobstore(DjangoJobStore(), "default")

    @register_job(scheduler, "interval", seconds=5)
    def test_job():
        # 定时每5秒执行一次
        print(time.strftime('%Y-%m-%d %H:%M:%S'))

    register_events(scheduler)
    # 启动定时器
    scheduler.start()
except Exception as e:
    print('定时任务异常：%s' % str(e))
```

## 使用&运行

apscheduler定时任务会跟随django项目一起运行因此直接启动django即可

```python
python manage.py runserver
```
` 詳細查看另一篇文章:  `[django-apscheduler](https://blog.csdn.net/m0_69082030/article/details/126939574)
## 优缺点

**优点：**
简单、定时方式丰富、轻量
**缺点：**
定时任务会跟随django项目一起运行，会影响django对外提供的web服务
使用uwsgi线上运行时，很难启动apscheduler定时任务
按照文档上的设置--enable-threads依然没有正常启动，具体原因未知，也查了很多方法，都没有解决。并且也不建议将定时任务和uwsgi放在一起运行，这样定时任务在运行过程中可能会影响uwsgi的服务。

# 使用Celery插件实现定时任务
## 介紹

```python
"""
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
"""
```

## 安装庫
本例建立在认为你已经知道如何使用**Celery**实现异步任务的基础上，需要学习的请移步Django使用Celery,可參考另一篇博客：[celery异步—生产者消费者](https://blog.csdn.net/m0_69082030/article/details/126872436)，或者官方文檔：[官方文檔](https://docs.celeryq.dev/en/latest/index.html)
本例使用redis作为Borker和backend

```python
pip install -U "celery[redis]"
```

## 配置celery

**settings.py** 增加以下代码

```python
# Celery配置
from kombu import Exchange, Queue
# 设置任务接受的类型，默认是{'json'}
CELERY_ACCEPT_CONTENT = ['application/json']
# 设置task任务序列列化为json
CELERY_TASK_SERIALIZER = 'json'
# 请任务接受后存储时的类型
CELERY_RESULT_SERIALIZER = 'json'
# 时间格式化为中国时间
CELERY_TIMEZONE = 'Asia/Shanghai'
# 是否使用UTC时间
CELERY_ENABLE_UTC = False
# 指定borker为redis 如果指定rabbitmq CELERY_BROKER_URL = 'amqp://guest:guest@localhost:5672//'
CELERY_BROKER_URL = 'redis://127.0.0.1:6379/0'
# 指定存储结果的地方，支持使用rpc、数据库、redis等等，具体可参考文档 # CELERY_RESULT_BACKEND = 'db+mysql://scott:tiger@localhost/foo' # mysql 作为后端数据库
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/1'
# 设置任务过期时间 默认是一天，为None或0 表示永不过期
CELERY_TASK_RESULT_EXPIRES = 60 * 60 * 24
# 设置worker并发数，默认是cpu核心数
# CELERYD_CONCURRENCY = 12
# 设置每个worker最大任务数
CELERYD_MAX_TASKS_PER_CHILD = 100
# 指定任务的位置
CELERY_IMPORTS = (
    'base.tasks',
)
# 使用beat启动Celery定时任务
# schedule时间的具体设定参考：https://docs.celeryproject.org/en/stable/userguide/periodic-tasks.html
CELERYBEAT_SCHEDULE = {
    'add-every-10-seconds': {
        'task': 'base.tasks.cheduler_task',
        'schedule': 10,
        'args': ('hello', )
    },
}
```


## 编写定时任务代码


在本例中是在子應用新建 **tasks.py** 在裏面增加的定时任务


```python
# Create your tasks here
from __future__ import absolute_import, unicode_literals
from celery import shared_task
import time

@shared_task
def cheduler_task(name):
    print(name)
    print(time.strftime('%X'))
```

## 使用&运行
启动定时任务

```python
celery -A dase_django_api beat -l info
```
启动Celery worker 用来执行定时任务

```python
celery -A dase_django_api worker -l -l info
```
`注意：这里的 dase_django_api 要换成你的Celery APP的名称`


## 优缺点
**优点**：
		 稳定可靠、管理方便
		高性能、支持分布式
**缺点：**
实现复杂
部署复杂
非轻量级

# 自建代码实现定时任务
## 创建定时任务
在子應用下创建一个文件名为 **schedules.py**，键入以下内容

```python
import os, sys, time, datetime
import threading
import django
base_apth = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
# print(base_apth)
# 将项目路径加入到系统path中，这样在导入模型等模块时就不会报模块找不到了
sys.path.append(base_apth)
os.environ['DJANGO_SETTINGS_MODULE'] ='base_django_api.settings' # 注意：base_django_api 是我的模块名，你在使用时需要跟换为你的模块
django.setup()
from base.models import ConfDict

def confdict_handle():
    while True:
        try:
            loca_time = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            print('本地时间：'+str(loca_time))
            time.sleep(10)
        except Exception as e:
            print('发生错误，错误信息为：', e)
            continue


def main():
    '''
    主函数，用于启动所有定时任务，因为当前定时任务是手动实现，因此可以自由发挥
    '''
    try:
        # 启动定时任务，多个任务时，使用多线程
        task1 = threading.Thread(target=confdict_handle)
        task1.start()
    except Exception as e:
        print('发生异常：%s' % str(e))

if __name__ == '__main__':
    main()
```
## 使用&运行
直接运行脚本即可

```python
python apps/base/schedules.py
```

`注意：apps/base 為你的子應用路徑 `

## 优缺点
**优点：**
自定义
高度自由
**缺点：**
过于简单
当任务过多时，占用资源也会增加








---

