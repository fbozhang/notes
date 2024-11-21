
# 报错信息

```python
 ImportError: Could not import 'rest_framework_jwt.authentication.JSONWebTokenAuthentication' for API setting 'DEFAULT_AUTHENTICATION_CLASSES'. ImportError: cannot import name 'smart_text' from 'django.utils.encoding'
```

# 原因

- JSON Web Token不再维护，故不使用。
- 官方建议的是使用simpleJWT认证
## 要求
官方文档：[djangorestframework-jwt](https://jpadilla.github.io/django-rest-framework-jwt/)

- Python (2.7, 3.3, 3.4, 3.5)
- Django (1.8, 1.9, 1.10)
- Django REST Framework (3.0, 3.1, 3.2, 3.3, 3.4, 3.5)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f7d85da5bdd49267ebc363eb3b65f54.png)
所以新版的django无法使用

# 解决方案
使用**simplejwt**

官方文档：[djangorestframework-simplejwt](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/)

## 安装

```python
pip install djangorestframework-simplejwt
```

## 配置

### setting.py

```python
# settings.py
INSTALLED_APPS = [
	...
    'rest_framework', 
    'rest_framework_simplejwt', 
]

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    # 认证类
    # 先进行token的验证，如果没有携带token就进行session认证，如果没有session就就基本认证
    # 认证顺序是从上到下，需要哪个加哪个
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ),
}
SIMPLE_JWT = {
    # token有效时长(返回的 access 有效时长)
    'ACCESS_TOKEN_LIFETIME': datetime.timedelta(seconds=30),
    # token刷新的有效时间(返回的 refresh 有效时长)
    'REFRESH_TOKEN_LIFETIME': datetime.timedelta(seconds=20),
}
```


### urls.py

```python
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView, TokenVerifyView

urlpatterns = [
    path('authorizations/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('verify/', TokenVerifyView.as_view(), name='token_verify'),
]
```
**或者**
点源码进去看一样的

```python
from rest_framework_simplejwt.views import token_obtain_pair, token_refresh, token_verify

urlpatterns = [
    path('authorizations/', token_obtain_pair, name='token_obtain_pair'),
    path('refresh/', token_refresh, name='token_refresh'),
    path('verify/', token_verify, name='token_verify'),
]
```
- 登录接口，返回refresh和access值
- refresh用来刷新获取新的Token值
- access用来请求身份认证的Token
- access的过期时间参照配置ACCESS_TOKEN_LIFETIME
- refresh的过期时间参照配置REFRESH_TOKEN_LIFETIME
- 当用refresh刷新后，access的过期时间等于【当前刷新的此刻时间+ACCESS_TOKEN_LIFETIME】


### 其他设置

```python
# Django project settings.py

from datetime import timedelta
...

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=5),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=1),
    "ROTATE_REFRESH_TOKENS": False,
    "BLACKLIST_AFTER_ROTATION": False,
    "UPDATE_LAST_LOGIN": False,

    "ALGORITHM": "HS256",
    "SIGNING_KEY": settings.SECRET_KEY,
    "VERIFYING_KEY": "",
    "AUDIENCE": None,
    "ISSUER": None,
    "JSON_ENCODER": None,
    "JWK_URL": None,
    "LEEWAY": 0,

    "AUTH_HEADER_TYPES": ("Bearer",),
    "AUTH_HEADER_NAME": "HTTP_AUTHORIZATION",
    "USER_ID_FIELD": "id",
    "USER_ID_CLAIM": "user_id",
    "USER_AUTHENTICATION_RULE": "rest_framework_simplejwt.authentication.default_user_authentication_rule",

    "AUTH_TOKEN_CLASSES": ("rest_framework_simplejwt.tokens.AccessToken",),
    "TOKEN_TYPE_CLAIM": "token_type",
    "TOKEN_USER_CLASS": "rest_framework_simplejwt.models.TokenUser",

    "JTI_CLAIM": "jti",

    "SLIDING_TOKEN_REFRESH_EXP_CLAIM": "refresh_exp",
    "SLIDING_TOKEN_LIFETIME": timedelta(minutes=5),
    "SLIDING_TOKEN_REFRESH_LIFETIME": timedelta(days=1),

    "TOKEN_OBTAIN_SERIALIZER": "rest_framework_simplejwt.serializers.TokenObtainPairSerializer",
    "TOKEN_REFRESH_SERIALIZER": "rest_framework_simplejwt.serializers.TokenRefreshSerializer",
    "TOKEN_VERIFY_SERIALIZER": "rest_framework_simplejwt.serializers.TokenVerifySerializer",
    "TOKEN_BLACKLIST_SERIALIZER": "rest_framework_simplejwt.serializers.TokenBlacklistSerializer",
    "SLIDING_TOKEN_OBTAIN_SERIALIZER": "rest_framework_simplejwt.serializers.TokenObtainSlidingSerializer",
    "SLIDING_TOKEN_REFRESH_SERIALIZER": "rest_framework_simplejwt.serializers.TokenRefreshSlidingSerializer",
}
```
#### ACCESS_TOKEN_LIFETIME
**datetime.timedelta**指定访问令牌有效时间的对象。该timedelta值在令牌生成期间添加到当前 UTC 时间以获得令牌的默认“exp”声明值。

#### REFRESH_TOKEN_LIFETIME
**datetime.timedelta**指定刷新令牌有效时间的对象。该**timedelta**值在令牌生成期间添加到当前 UTC 时间以获得令牌的默认“exp”声明值。

#### ROTATE_REFRESH_TOKENS
设置为 时**True**，如果将刷新令牌提交给 **TokenRefreshView**，则新的刷新令牌将与新的访问令牌一起返回。这个新的刷新令牌将通过 JSON 响应中的“刷新”键提供。新的刷新令牌将有一个更新的到期时间，该时间是通过将设置中的 **timedelta** 添加**REFRESH_TOKEN_LIFETIME** 到发出请求时的当前时间来确定的。如果黑名单应用程序正在使用中并且**BLACKLIST_AFTER_ROTATION**设置为**True**，则提交到刷新视图的刷新令牌将被添加到黑名单中。

#### BLACKLIST_AFTER_ROTATION
设置为 时，如果黑名单应用程序正在使用且设置设置True为 ，则提交给 的刷新令牌 将被添加到黑名单。您需要在设置文件中添加才能使用此设置。 TokenRefreshViewROTATE_REFRESH_TOKENSTrue'rest_framework_simplejwt.token_blacklist',INSTALLED_APPS


#### 官方文档
详细请看官方文档:[setting](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/settings.html)
