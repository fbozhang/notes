# 报错信息

```python
requests.exceptions.SSLError: HTTPSConnectionPool(host='https://www.youtube.com/', port=443): 
Max retries exceeded with url: / (Caused by SSLError(SSLEOFError(8, 'EOF occurred in violation 
of protocol (_ssl.c:1125)')))
```


# 原因

**urllib3 1.26之后更新了主架构**

### urllib3 1.26
```python
proxy = { 'https': 'http://127.0.0.1: 80' }
```
### urllib3 schema旧版

```python
proxies ={
　　'http':'http://127.0.0.1: 80',
　　'https':'https://127.0.0.1: 80'
}
```

# 解决方案
## 方法一
降低版本为==1.25.11==（本人使用该方法，比较通用）

```python
pip install urllib3==1.25.11
```

## 方法二

把**V代P理N服务器**的**ip**拷贝出来作为**requests**的**proxies** 使用，端口一般是7890，然后本机不使用V科P学N上网则不会报错

```python
import requests
proxies ={
　　'http':'http://127.0.0.1: 7890',
　　'https':'https://127.0.0.1: 7890'
}

requests.get(url,headers=headers,proxies=proxies)
```

# 大佬解析原理

## [传送门](https://zhuanlan.zhihu.com/p/350015032)
