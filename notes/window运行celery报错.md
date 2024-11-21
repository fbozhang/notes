
### 报错信息
```python
Traceback (most recent call last):
File "c:\program files\python36\lib\site-packages\billiard\pool.py", line 359, in workloop
result = (True, prepare_result(fun(*args, **kwargs)))
File "c:\program files\python36\lib\site-packages\celery\app\trace.py", line 518, in _fast_trace_task
tasks, accept, hostname = _loc
ValueError: not enough values to unpack (expected 3, got 0)
```

### 原因

`Celery 4`和`Windows 10` 冲突

## 解决方案

原来的执行命令
```python
celery -A <mymodule> worker -l info
```


**安装包**

```python
pip install eventlet
```
运行
```python
celery -A <mymodule> worker -l info -P eventlet
```

### 本地开发

建议用`solo`方便调试

```python
celery -A <mymodule> worker -l info -P solo
```

## 总结

不用**win**，可以省掉一大堆奇奇怪怪的**win**专属问题

## 参考连接

[https://github.com/celery/celery/issues/4081](https://github.com/celery/celery/issues/4081)
