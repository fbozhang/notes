

# 问题描述
使用channels启动ASGI结果却是普通启动，如下：

```python
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
July 15, 2023 - 18:23:49
Django version 4.2, using settings 'staffSystem_django.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
我希望得到的启动方式如下:

```python
System check identified no issues (0 silenced).
July 15, 2023 - 18:23:49
Django version 4.2, using settings 'staffSystem_django.settings'
Starting ASGI/Channels version 3.0.1 development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

**setting.py:**

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'channels',
    'app01.apps.App01Config',
]


ASGI_APPLICATION = 'staffSystem_django.asgi.application'
```
**asgi.py:**

```python
import os

from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from . import routing

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'staffSystem_django.settings')

# application = get_asgi_application()

application = ProtocolTypeRouter({
    'http': get_asgi_application(),
    'websocket': URLRouter(routing.websocket_urlpatterns)
})

```
我打断点就拦不到，甚至==ASGI_APPLICATION=‘’== 都不报错。




---

# 原因分析：
`盲猜是版本问题，查不到一个官方解释。人类疑惑高版本咋会失去低版本有的实用功能~`

简直是人类迷惑，如果直接使用**pip install channels**的话会自动下载比较高的版本（我下载的**4.0.1**的版本），所以在注册channels的时候，Django的settings.py中**ASGI_APPLICATION**没有被配置识别，使得总是在使用**WSGI_APPLICATION**中的配置，我还以为是**WSGI_APPLICATION**影响了，但是注释掉也不用**ASGI_APPLICATION**。然后还不会报错，简直无语到家。我甚至还看见有另一个叫做**django-channels**，下载注册app后也没有卵用。


---

# 解决方案：

## 下载低版本
第一次尝试，我下载了**2.3.0**

```python
pip install channels==2.3.0
```
重新启动django程序，终于报错，看到报错是真开心不会一脸懵逼了。

报错提示信息说没有==as_asgi()== 这个函数。这个函数是我在安装**channels=4.0.1**时写代码类名点出来的不是直接写出来的，证明**父类**是一定有这个函数。我点进下载**2.3.0**后的源码查看确实没有这个函数，那就证明是后来才有这个函数。

第二次尝试，下载**3.0.1**，这次就成功了！！！

```python
pip install channels==3.0.1
```
启动django程序

```python
System check identified no issues (0 silenced).
July 15, 2023 - 18:17:31
Django version 4.2, using settings 'staffSystem_django.settings'
Starting ASGI/Channels version 3.0.1 development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```
看到==Starting ASGI/Channels== 就是成功了。

再发送一个**ws**请求试试看，可以看到握手与连接的请求成功收到并返回

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/70b84718467e151dd233b53a79d63475.png)


看看浏览器抓包，可以看到黄色方框的**http**请求成功，红色方框的**websocket**请求也是建立连接成功。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9eb3514769ca418ad1f9c77e9595bd39.png)
