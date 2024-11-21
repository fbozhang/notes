
## 背景

当实现一个使用流响应的程序使用**wsgi**启动没有问题但是换成**asgi**就无法实现正常的流式效果而是一次性返回。

## 解决方案

原来的代码如下,当使用**wsgi**启动程序效果是每隔一秒客户端收到一个数字，而当转为使用**ASGI**后效果却变成了在10秒后一次性返回10个数字

```python
def order_streamrequest(request):

    # 定义一个内部函数作为生成器
    def stream_data_generator():
        for i in range(10):
        	time.sleep(1) # 模拟耗时操作
            # 判断是否是 AJAX 请求，如果是，则使用流式传输
            # if request.is_ajax(): # WSGI 才有这个方法
            # if request.META.get('HTTP_X_REQUESTED_WITH') == 'XMLHttpRequest':# ASGI用这个判断
            yield f"data: {i}\n\n"  # 使用 SSE 格式发送数据

    return StreamingHttpResponse(stream_data_generator(), content_type='text/event-stream')  # 设置 content_type 为 "text/event-stream"
```

为了纠正这种行为，我需要添加一个异步层并使用异步/非阻塞替代方案。
将代码修改为以下

```python
import asyncio
def order_streamrequest(request):

    # 定义一个内部函数作为生成器
    async def stream_data_generator():
        for i in range(10):
            await asyncio.sleep(1) # 模拟耗时逻辑
            # 判断是否是 AJAX 请求，如果是，则使用流式传输
            # if request.is_ajax(): # WSGI 才有这个方法
            # if request.META.get('HTTP_X_REQUESTED_WITH') == 'XMLHttpRequest':# ASGI用这个判断
            yield f"data: {i}\n\n"  # 使用 SSE 格式发送数据

    return StreamingHttpResponse(stream_data_generator(), content_type='text/event-stream')  # 设置 content_type 为 "text/event-stream"
```


## 参考
[Daphne does not stream #413](https://github.com/django/daphne/issues/413)
[Django StreamingHttpResponse stops work when usind django channels #1761](https://github.com/django/channels/issues/1761)
[Add asynchronous responses for use with an ASGI server](https://code.djangoproject.com/ticket/33735)
[StreamingHttpResponse raises SynchronousOnlyOperation in ASGI server](https://code.djangoproject.com/ticket/32798)
