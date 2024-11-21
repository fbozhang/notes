
# 前言
有道是有反爬虫就有反反爬虫，这篇就从TLS指纹识别说起。

---

# TLS指纹

## 什么是TLS指纹
TLS指纹是一种用于识别和验证TLS（传输层安全）通信的技术。

TLS指纹可以通过检查TLS握手过程中使用的密码套件、协议版本和加密算法等信息来确定TLS通信的特征。由于每个TLS实现使用的密码套件、协议版本和加密算法不同，因此可以通过比较TLS指纹来判断通信是否来自预期的源或目标。

TLS指纹可以用于检测网络欺骗、中间人攻击、间谍活动等安全威胁，也可以用于识别和管理设备和应用程序。

TLS指纹识别原理（ja3算法）： [https://github.com/salesforce/ja3](https://github.com/salesforce/ja3)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5bf3182e2c09fb7f18e98c9fc3806bc8.png)
##  测试TLS指纹
测试一下不同客户端之间的指纹差异（**ja3_hash**）
测试网站： [https://tls.browserleaks.com/json](https://tls.browserleaks.com/json)

- **ubuntu** 20.04
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6893317027d43f5fc6075b72acf60acc.png)


- **centos**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/450992b9171be6341ef6a30512bc6719.png)
- **window10**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/111979087897416fb25efb8101ad2ce0.png)



- **chrome**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/386d722a37e01aab88f9a20ff3bff63f.png)


- **python3.9**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/abe40efe01a99c00d340c5bb4aaabf83.png)

可见不同的客户端都存在区别，针对最后一个python的ja3_text做一个简单的说明

- **第一个值** <font color="#00dd00"> 771</font> ：表示 **JA3** 版本，即用于生成指纹的 **JA3** 脚本的版本。
- **第二个值** <font color="#00dd00">4866-4867-4865-49196-49200-49195-49199-163-159-162-158-49327-49325-49188-- 49192-49162-49172-49315-49311-107-106-57-56-49326-49324-49187-49191-49161-49171-49314-49310-103-64-51-50-52393-52392-49245-49249-49244-49248-49267-49271-49266-49270-52394-49239-49235-49238-49234-196-195-190-189-136-135-69-68-157-156-49313-49309-49312-49308-61-60-53-47-49233-49232-192-186-132-65-255</font>：表示加密套件，即客户端可以支持的加密算法。
- **第三个值** <font color="#00dd00">0-11-10-35-22-23-13-43-45-51-21</font>：表示支持的压缩算法。
- **第四个值** <font color="#00dd00">29-23-30-25-24</font>：表示支持的 **TLS** 扩展，如 **SNI**。
- **第五个值** <font color="#00dd00">0-1-2</font>：表示支持的 **elliptic curves**，即椭圆曲线算法。


## 绕过TLS指纹

绕过就是伪造成合法客户端就行，简单来说，就是伪装ja3_text值，让其不被拦截即可，以修改支持的加密算法为主。

### 使用原生urllib

**实现**
```python
import urllib.request
import ssl

url = 'https://tls.browserleaks.com/json'
req = urllib.request.Request(url)
resp = urllib.request.urlopen(req)
print(resp.read().decode())

# 伪造TLS指纹
context = ssl.create_default_context()
context.set_ciphers("ECDHE-RSA-AES128-GCM-SHA256+ECDHE+AESGCM")

url = 'https://tls.browserleaks.com/json'
req = urllib.request.Request(url)
resp = urllib.request.urlopen(req, context=context)
print(resp.read().decode())
```
**效果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb7963aee7fa338c2468805f3d54708f.png)


### 使用其他成熟库！！

可以试试[curl_cffi](https://github.com/yifeikong/curl_cffi)这个库,他是基于**requests**，所以语法基本一样
>Unlike other pure python http clients like httpx or requests, curl_cffi can impersonate browsers' TLS signatures or JA3 fingerprints. If you are blocked by some website for no obvious reason, you can give this package a try.

`也可以试试pyhttpx、pycurl这两个库`
**安装**

```python
pip install curl_cffi --upgrade
```

**实现**

```python
from curl_cffi import requests

print("edge99:", requests.get("https://tls.browserleaks.com/json", impersonate="edge99").json().get("ja3_hash"))
print("chrome110:", requests.get("https://tls.browserleaks.com/json", impersonate="chrome110").json().get("ja3_hash"))
print("safari15_3:", requests.get("https://tls.browserleaks.com/json", impersonate="safari15_3").json().get("ja3_hash"))

# 支持代理
proxies = {"https": "http://localhost:7890"}
r = requests.get("https://tls.browserleaks.com/json", impersonate="safari15_3", proxies=proxies)
print(r.json().get("ja3_hash"))
```

**效果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1026e2f03cdc926e8912de29bad060d7.png)

**支持伪造的浏览器列表如下:**随便挑一个就行了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf10701c65fe9a78c52525368dde9fac.png)



### 修改requests底层代码
**requests** 库的 **SSL/TLS** 认证是基于 **urllib3** 库实现的，所以改底层就是改**urllib3**的代码

修改相关**SSL**代码，文件地址一般为**site-packages/urllib3/util/ssl_.py**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c3db6d007a0ebc194f847f86713050a.png)

```python
# A secure default.
# Sources for more information on TLS ciphers:
#
# - https://wiki.mozilla.org/Security/Server_Side_TLS
# - https://www.ssllabs.com/projects/best-practices/index.html
# - https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
#
# The general intent is:
# - prefer cipher suites that offer perfect forward secrecy (DHE/ECDHE),
# - prefer ECDHE over DHE for better performance,
# - prefer any AES-GCM and ChaCha20 over any AES-CBC for better performance and
#   security,
# - prefer AES-GCM over ChaCha20 because hardware-accelerated AES is common,
# - disable NULL authentication, MD5 MACs, DSS, and other
#   insecure ciphers for security reasons.
# - NOTE: TLS 1.3 cipher suites are managed through a different interface
#   not exposed by CPython (yet!) and are enabled by default if they're available.
```


修改这里的加密算法为你想要的，不嫌麻烦也可以去继承重写就不用改代码了
```python
DEFAULT_CIPHERS = ":".join(
    [
        "ECDHE+AESGCM",
        "ECDHE+CHACHA20",
        "DHE+AESGCM",
        "DHE+CHACHA20",
        "ECDH+AESGCM",
        "DH+AESGCM",
        "ECDH+AES",
        "DH+AES",
        "RSA+AESGCM",
        "RSA+AES",
        "!aNULL",
        "!eNULL",
        "!MD5",
        "!DSS",
    ]
)
```
**修改前**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f905c402fcf15f4b14e2eca9246a4c21.png)
**修改后**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2c1ac96d377a2b904204ccb4d970b83b.png)
# Akamai指纹相关（HTTP/2指纹）

## 什么是Akamai指纹
**Akamai Fingerprint**是**Akamai Technologies**公司提供的一种防止恶意机器人和自动化攻击的技术，它基于**浏览器指纹识别技术**。

**浏览器指纹**是一种用于识别**Web**浏览器的技术，它通过收集并分析浏览器的各种属性和行为，如用户代理字符串、插件、字体、语言、屏幕分辨率等信息来识别浏览器。浏览器指纹在互联网安全领域得到了广泛应用，可以用于检测和识别恶意机器人、欺诈行为、网络钓鱼等。

**Akamai Fingerprint**利用了浏览器指纹技术，将其与其他安全技术结合起来，以识别和拦截自动化攻击。它可以在不影响用户体验的情况下，对访问网站的浏览器进行识别和验证，防止自动化攻击、账户滥用和数据泄露等安全问题。

---

可以在  [https://tls.peet.ws/api/all]( https://tls.peet.ws/api/all) 看到详细的指纹，主要有如下内容

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/19772c5b29805108f9cec82ee445e399.png)

指纹为：`1:65536,2:0,3:1000,4:6291456,6:262144|15663105|0|m,a,s,p`

1. `1:65536`: ==HEADER_TABLE_SIZE==，即头部表大小为**64KB**，指的是用于存储请求头和响应头的大小，它是可以调整的。这个字段指明了使用**64KB**的头部表大小。

1. `2:0`: ==HTTP2_VERSION==，指示此请求使用的**HTTP/2**版本。**0**表示**H2**，表示启用了**HTTP/2**协议。

1. `3:1000`:== MAX_CONCURRENT_STREAMS==，即**最大并发流数**，指的是在任何给定时间内，**客户端和服务器端可以并行发送的最大请求数量**。这个字段指明了最大并发流数为**1000**。

1. `4:6291456`: ==INITIAL_WINDOW_SIZE==，即**初始流窗口大小**，指的是初始的流控窗口大小，**即客户端可以发送的最大字节数量**。这个字段指明了初始流窗口大小为**6MB**（即**6291456**字节）。

1.  `6:262144|15663105|0|m,a,s,p`
   : 以竖杠“|”分隔。具体含义如下：

     - `6:262144`: ==MAX_HEADER_LIST_SIZE==，即动态表大小，指的是接收方可以接收的最大**HTTP**头部大小。这个字段指明了动态表大小为**256KB**（即**262144**字节）。
    - `15663105`: ==WINDOW_UPDATE==，表示收到了==WINDOW_UPDATE==帧，并且窗口大小增加了**15663105**个字节。
    - `0`:` no compression`，表示不启用头部压缩。
    - 以` :`开头的 **header** 的第一个字符参与编码，多个逗号隔开。如 `:method`、`:authority`、`:scheme`、`:path` 编码为 `m,a,s,p`

可在[ Passive Fingerprinting of HTTP/2 Clients](https://www.blackhat.com/docs/eu-17/materials/eu-17-Shuster-Passive-Fingerprinting-Of-HTTP2-Clients-wp.pdf)中查看详细细节


## 测试Akamai指纹

测试网站: [https://tls.browserleaks.com/json](https://tls.browserleaks.com/json)

- **window 10**(不知道是不是curl版本问题导致不行, 参考上面的Ubuntu的curl是可以的)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/51465a86531d1df79f33a6fbc46d62a6.png)
- **centos**(不知道是不是curl版本问题导致不行, 参考上面的Ubuntu的curl是可以的)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8a5335a7eb233bb7fd58042381a0709.png)


- **chrome**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1ad116968ceb97f9ab52b1a677f7d01f.png)

- **python**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6944ed470e2d27e07f852d54ba8698f6.png)
可以看到用**python requests**直接为空被拦截在外了

## 绕过Akamai指纹

伪造指纹中特定的字段即可。
### 使用其他成熟库

还是刚才的 **curl_cffi**这个库，因为这个库主打的就是模拟各种指纹

```python
from curl_cffi import requests

print("edge99:", requests.get("https://tls.browserleaks.com/json", impersonate="edge99").json().get("akamai_hash"))
print("chrome110:", requests.get("https://tls.browserleaks.com/json", impersonate="chrome110").json().get("akamai_hash"))
print("safari15_3:", requests.get("https://tls.browserleaks.com/json", impersonate="safari15_3").json().get("akamai_hash"))
```

**效果**
可以看到已经获取到
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4eb11b5cd09199f7940bc8655668cb73.png)

# 实操
[ https://ascii2d.net]( https://ascii2d.net) 存在**CloudFlare**的指纹护盾，拒绝爬虫，测试一下。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3187ff373be0248df66b7c0bb7df040f.png)
直接请求**title**显示这个**Just a moment...**，这个明显是五秒盾了

尝试绕过一下

```python
from curl_cffi import requests

print(requests.get("https://ascii2d.net", impersonate="chrome110").text)
```

**效果**，可以看得出正确获取到了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c7880c7cedb4b6aa20b9371846161fe8.png)

# 参考
[绕过 Cloudflare 指纹护盾](https://sxyz.blog/bypass-cloudflare-shield/)
[SSL 指纹识别和绕过](https://ares-x.com/2021/04/18/SSL-%E6%8C%87%E7%BA%B9%E8%AF%86%E5%88%AB%E5%92%8C%E7%BB%95%E8%BF%87/)
[HTTP2指纹识别(一种相对不为人知的网络指纹识别方法)](https://www.cnblogs.com/yudongdong/p/16654636.html)
