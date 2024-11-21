
# 背景

服务端和客户端应该怎么通信才能实现客户端能获取服务端最新消息让用户有更好的交互体验，如果是正常的发送一个请求首先要建立TCP连接然后等到服务器返回，如果是开发者可以通过发包情况就能知道建立连接成功与否，是否是在等待服务器响应，但是做为非开发者的普通用户当他点击一个按钮却没有任何反应他会怀疑是不是没点到还是卡住了之类了。不是一直点就是点到暴躁的放弃，不仅造成服务器的负担而且用户体验极差。也许我们可以在前端做一个虚假的转圈动画让客户知道正在处理，但是如果是个需要处理1小时的任务没有个进度条他也不知道是否值得等待。又假如我们做一个投票系统或者一个聊天室，我们要怎么让屏幕前的另一个彦祖及时看到呢？


# 简介
   **Web端即时通讯技术：** 服务器端可以即时地将数据的更新或变化反应到客户端，例如消息即时推送等功能都是通过这种技术实现的。但是在Web中，由于浏览器的限制，实现即时通讯需要借助一些方法。这种限制出现的主要原因是，一般的Web通信都是浏览器先发送请求到服务器，服务器再进行响应完成数据的现实更新。

　　**实现Web端即时通讯的方法：** 实现即时通讯主要有四种方式，它们分别是**轮询**、**长轮询(comet)**、**长连接(SSE)**、**WebSocket**。它们大体可以分为两类，一种是在HTTP基础上实现的，包括短轮询、**comet**和**SSE**；另一种不是在**HTTP**基础上实现是，即**WebSocket**。下面分别介绍一下这四种轮询方式，以及它们各自的优缺点。

# 个人见解
`以下纯属个人见解与实操经验，如有不当之处可以联系修改，感谢`
> 有空再专门详细写一篇理论补充知识，这个得从计算机网络的传输层协议开始说起了，特别是**websocket**协议。他不是简单的三次握手，在这之后还有使用魔法字符串和key加密。这篇文偏向实操，所见即所得方便上手。

==以下使用**python**的**web**框架的**django**实现，原理一样实现大同小异，尽量注释说明清楚，有不清楚的也可以联系我解答。==

# 被动推送
##  轮询

### 简介
　　短轮询的基本思路就是浏览器每隔一段时间向浏览器发送http请求，服务器端在收到请求后，不论是否有数据更新，都直接进行响应。这种方式实现的即时通信，本质上还是浏览器发送请求，服务器接受请求的一个过程，通过让客户端不断的进行请求，使得客户端能够模拟实时地收到服务器端的数据的变化。

　　这种方式的优点是比较简单，易于理解，实现起来也没有什么技术难点。缺点是显而易见的，这种方式由于需要不断的建立http连接，严重浪费了服务器端和客户端的资源。尤其是在客户端，距离来说，如果有数量级想对比较大的人同时位于基于短轮询的应用中，那么每一个用户的客户端都会疯狂的向服务器端发送http请求，而且不会间断。人数越多，服务器端压力越大，这是很不合理的。

**注意注意！！！** 重点是什么，之前为了实现一个毫秒级的进度条，打开**开发者模式(F12)**后一秒发出了一千次请求，不说给服务器造成了什么样压力(是我的服务器我直接把你拉黑了)，重点是不前面的请求记录一瞬间顶到天上去，十分不方便调试！！！！

### 实现

定义轮询方法
```python
function renovate(t) {
           console.log("come in ")
           var sitv = setInterval(function () {

               var prog_url = '/renovate?t=' + t
               $.getJSON(prog_url, function (num_progress) {
                   if (num_progress === 0) {
                       console.log("正在讀取中")
                       $('.progress-bar').css('width', '20%');
                       $('.progress-bar').text('正在讀取中');
                   } else if (num_progress > 99) {
                       console.log("come in 99")
                       clearInterval(sitv);
                       $('.progress-bar').css('width', '99%');
                       $('.progress-bar').text('99%');
                   } else {
                       console.log('t:' + t + '  num_progress' + num_progress)
                       $('.progress-bar').css('width', num_progress + '%');
                       $('.progress-bar').text(num_progress + '%');
                   }
               });
           }, 1000);   // 每1000毫秒查询一次后台进度
       }
```
调用轮询

```python
$(function () {
            bindBtnAddEvent();
        })


        function bindBtnAddEvent() {
            $('#save').click(function () {
                {#alert('開始')#}
                var t = $.now()
                renovate(t)
            });
        }
```

 这里我专门在后端写了个接口来读取数据库数据给前端获取，很简单就不写出来了，传了个时间参数是为了在后端构建一个字典使得获取到正确的属于该用户的进度条，不然可能出现如果两个人同时访问脏数据的可能，就以防万一。


## 长轮询（comet）
### 简介

 **ajax**实现:
　　当服务器收到客户端发来的请求后,服务器端不会直接进行响应，而是先将这个请求挂起，然后判断服务器端数据是否有更新。如果有更新，则进行响应，如果一直没有数据，则到达一定的时间限制(服务器端设置)才返回。。 客户端**JavaScript**响应处理函数会在处理完服务器返回的信息后，再次发出请求，重新建立连接。




**简单来说**：就是服务端维持一个**消息队列**，当收到一个请求就检查**消息队列**看有没有数据有就把数据取出来返回给客户端，因为队列是**先进先出**所以顺序是对的，没有数据就**hold**住这个请求一会，众所周知TCP连接后还有一个响应时间如果在响应时间也就是还没响应时，当你看开发者模式这时的**status**就不是状态码而是**pending**。


举例理解一下
```python
requests.get('www.baidu.com',timeout=(5,10))
```
这里的代码是一个简单的请求，设置的这个**timeout**是一个元组的形式，第一个参数为连接超时，第二个是响应超时，这里我们设置10秒，当10秒后没有响应这个连接也结束了，这样就不会hold住。

### 实现

因为是简单实现一个**demo**，所以其实里面有很多不完善的地方不要纠结细节。

>该**demo**是实现一个聊天室

**urls.py**(这里是配置路由的，就是接口)

```python
from django.conf import settings
from django.contrib import admin
from django.urls import path, re_path
from django.views.static import serve

from app01.views import chat

urlpatterns = [
	path('longPoll/chat/', chat.longPoll_chat),
    path('send/msg/', chat.send_msg),
    path('get/msg/', chat.get_msg),
]
```

**效果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc65e947b870083feb7c5db4c7b0736f.png)





**view.py**(也就是视图函数，啥框架都有的处理接口的那个)

`注意这里用全局变量是不合理的，就是demo而已，深度建议用redis的发布与订阅实现，一是减少开销不用给每个新用户建一个队列浪费空间，二是redis比较快`
```python
import queue

from django.shortcuts import render
from django.http import JsonResponse

# 这里用python的队列来模拟。也可以使用redis的发布和订阅来实现，则不需要建那么多队列，所有访客都可以访问同一个消息队列
USER_QUEUE = {}  # {'asd':queue.Queue(),'qwe':queue.Queue()} 为每一个访客建一个队列

def longPoll_chat(request):
	""" 展示聊天界面 """
    uid = request.GET.get('uid')
    USER_QUEUE[uid] = queue.Queue() # 来了个彦祖，为他创建一个队列
    return render(request, 'longPoll_chat.html', {'uid': uid})


def send_msg(request):
	""" 发送消息接口，当用户在聊天室发一句话点击事件就会触发这个 """
    text = request.GET.get('text') # 获取文本
    for uid, q in USER_QUEUE.items(): # 遍历所有的队列
        q.put(text) # 为每一个队列添加消息，为了让这个聊天室的每个用户消息同步
    return JsonResponse({"msg": 'ok'})


def get_msg(request):
	"""" 获取消息，就是看这个聊天室里面有没有其他人发了消息，有就接收以下 """
    uid = request.GET.get('uid')
    q = USER_QUEUE[uid]  # 获取自己的队列

    result = {'status': True, 'data': None}
    try:
    	# 这里就是 hold 请求了，监听队列10s如果有数据就能get到，如果10s过了还没有就会触发 except queue.Empty，不同的status发送到前端前端就可以判断了
        data = q.get(timeout=10) # 注意这里如果不设置timeout会一直等下去，程序会卡在这里，等的花都谢了
        result['data'] = data
    except queue.Empty as e:
        result['status'] = False

    return JsonResponse(result)
```




**longPoll_chat.html**（这里就是前端部分了，只是前后端不分离用了一些模板引擎的语法，但是光看js怎么发送请求应该就懂了，html怎么写不太重要自己修改即可）


```python
{% extends 'layout.html' %}

{% block css %}
    <style>
        .message {
            height: 300px;
            border: 1px solid #dddddd;
            width: 100%;
        }
    </style>
{% endblock %}

{% block content %}
    <div class="message" id="message"></div>
    <div>
        <input type="text" placeholder="请输入" id="txt">
        <input type="button" value="发送" onclick="sendMessage();"> # 这里给这个按钮绑定点击事件
    </div>
{% endblock %}

{% block js %} // 这是模板引擎的语法，就继承模板的那个挖洞，有点类似vue的组件和插槽的感觉
    <script>
        USER_UID = "{{ uid }}"; // 这是模板引擎的语法，就是接收后端发过来的东西，有点类似vue的那个插值表达式
        const MAX_REQUESTS = 5; // 最大请求次数，主要是不想看他一直发，你可以写不同的逻辑
        let requestCount = 0; // 请求计数器

        function sendMessage() {
            let text = $("#txt").val();// 获取文本，jQuery没啥好说的了，不是这篇文的重点，后面有关jQuery的就不注释说明了

            $.ajax({
                url: '/send/msg/',
                type: 'GET',
                data: {
                    text: text
                },
                success: function (res) {
                    console.log("请求发送成功", res)
                }
            })
        }

        function getMessage() {
            $.ajax({
                url: '/get/msg/',
                type: 'GET',
                data: {
                    uid: USER_UID,
                },
                dataType: "JSON",
                success: function (res) {
                    // 超时，没有数据，也就是status为False，不是True自然是False就不写了
                    // 有新数据，暂时信息数据
                    if (res.status) {
                        //将内容拼成div标签，并添加到message区域
                        var tag = $("<div>");
                        tag.text(res.data)  //<div>啊大大</div>
                        $("#message").append(tag);
                    }
                    // 请求次数
                    requestCount++;
                    // 检查是否达到最大请求次数
                    if (requestCount === MAX_REQUESTS) {
                        console.log('达到最大请求次数，结束长轮询');
                        return; // 结束长轮询
                    }
                    getMessage();//自己调用自己，JS该模式实际不是递归，不会栈溢出
                }
            })
        }

        $(function () {
            getMessage();
        })

    </script>
{% endblock %}
```


　　
轮询与长轮询都是**基于HTTP**的，两者本身存在着缺陷:轮询需要更快的处理速度；长轮询则更要求处理并发的能力;两者都是“**被动型服务器**”的体现:**服务器不会主动推送信息**，而是在客户端发送**ajax**请求后进行返回的响应。而理想的模型是"**在服务器端数据有了变化后，可以主动推送给客户端**",这种"主动型"服务器是解决这类问题的很好的方案。

>**注意！！** 但是啊，虽然**websocket**这种双向通信非常实用但是旧浏览器以及部分老设备的客户端不支持**websocket**协议，所以一般大公司(eg:微信)是使用**长轮询**，比较通用。虽然使用更多内存会占用服务器资源但是人家家大业大不怕，人家追求稳定通用。

## 比较

　　长轮询和短轮询比起来，明显减少了很多不必要的http请求次数，相比之下节约了资源。长轮询的缺点在于，连接挂起也会导致资源的浪费。
# 主动推送
## 长连接（SSE）

### 简介
**Server-Sent Events（SSE）** 是一种用于实现服务器向客户端实时推送数据的Web技术。与传统的轮询和长轮询相比，**SSE**提供了更高效和实时的数据推送机制。

**SSE**基于**HTTP**协议，允许服务器将数据以事件流（**Event Stream**）的形式发送给客户端。客户端通过建立持久的**HTTP**连接，并监听事件流，可以实时接收服务器推送的数据。

SSE的主要特点包括：

- **简单易用**：**SSE**使用基于文本的数据格式，如纯文本、JSON等，使得数据的发送和解析都相对简单。
- **单向通信**：**SSE**支持服务器向客户端的单向通信，服务器可以主动推送数据给客户端，而客户端只能接收数据。
- **实时性**：**SSE**建立长时间的连接，使得服务器可以实时地将数据推送给客户端，而无需客户端频繁地发起请求。

**进行SSE实时数据推送时的注意点**
- **异步处理：** 由于**SSE**是基于**长连接**的机制，推送数据的过程是一个长时间的操作。为了**不阻塞**服务器线程，推荐使用**异步**方式处理**SSE**请求。您可以在控制器方法中使用**@Async**注解或使用**CompletableFuture**等异步编程方式。
- **超时处理：** **SSE**连接可能会因为网络中断、客户端关闭等原因而发生超时。为了避免无效的连接一直保持在服务器端，您可以设置超时时间并处理连接超时的情况。可以使用**SseEmitter**对象的**setTimeout**()方法设置超时时间，并通过**onTimeout**()方法处理连接超时的逻辑。
- **异常处理：** 在实际应用中，可能会出现一些异常情况，如网络异常、推送数据失败等。您可以使用**SseEmitter**对象的**completeWithError**()方法将异常信息发送给客户端，并在客户端通过**eventSource.onerror**事件进行处理。
- **内存管理：** 使用**SseEmitter**时需要注意内存管理，特别是在大量并发连接的情况下。当客户端断开连接时，务必及时释放**SseEmitter**对象，避免造成资源泄漏和内存溢出。
- **并发性能：** **SSE**的并发连接数可能会对服务器的性能造成影响。如果需要处理大量的并发连接，可以考虑使用线程池或其他异步处理方式，以充分利用服务器资源。
- **客户端兼容性：** 虽然大多数现代浏览器都支持**SSE**，但仍然有一些旧版本的浏览器不支持。在使用SSE时，要确保您的目标客户端支持**SSE**，或者提供备用的实时数据推送机制。
这些注意点将有助于您正确和高效地使用**SseEmitter**进行**SSE**实时数据推送。根据具体的应用需求，您可以根据实际情况进行调整和优化。



### 实现

**SSE**的优点就是实现比较简单，跟**HTTP**基本一样所以就不需要改动太多

**view.py**(直接在你原来用HTTP那个处理逻辑上修改就行了基本一样)

```python
from django.http import HttpResponse, JsonResponse, StreamingHttpResponse

def test(request):
    """ 该实现我采用两种返回结果，因为考虑到可能会有客户端不支持SSE，前端判断支持之后再使用流式传输 """
    data = json.loads(request.body.decode())
    text_list = data.get('text') # [{},{}]
    # 没有该标志采用JSON传输
    if request.GET.get('stream') is None:
    	# 这就是你原来的处理逻辑了

        # 这里是个demo就简单做一个保存数据库，你可以根据需求写你自己的逻辑
        id_list = []
        for text in text_list:
        	id = User.objects.create(**text)
        	id_list.append(id)
        	time.sleep(60) # 这里模拟一下耗时操作，你的逻辑可能就是这里导致处理特别慢，循环多少次就要多少个一分钟了
        	
         # 一次性返回，假如要一小时来执行上面的逻辑，用户傻傻等待一小时快要睡着了以为还没开始然后突然跟你说全部处理好了简直人麻了
        return JsonResponse({"code": 200, "data": id_list, "msg": '保存成功'})
        
	# 采用流式传输
    elif request.GET.get('stream') is True or request.GET.get('stream') == 'true':
    	def sse_stream():
    		""" 原来的处理逻辑 """
    		# 闭包可以获取外部数据
	        # 这里是个demo就简单做一个保存数据库，你可以根据需求写你自己的逻辑
	        for text in text_list:
	        	id = User.objects.create(**text)
	        	time.sleep(60) # 这里模拟一下耗时操作，你的逻辑可能就是这里导致处理特别慢，不管循环多少次，至少一分钟用户就能得到响应而不是循环次数*一分钟后才得到响应
	        	
	        	data_dict = {
                    'code': 200,
                    'message': '有一个成功了',
                    'data': id
                }
	        	# 返回部分数据给用户查看
	        	# 注意了，SSE传输的数据格式必须是这样
	        	# "data: 你的数据\n\n" ，必须是data开头\n\n结束
               sse_message = "data: {}\n\n".format(json.dumps(data_dict))
               yield sse_message # 这里就是不一样的地方了，先返回一段去给用户看,直接就知道是哪个好了
               # return # 你的逻辑中那些try...except，if...else之类的想要服务器主动中断链接的这里就直接return就行了
  	
  	# 这是第二个区别了，这是在闭包之外也就是真正返回响应的地方，这里把JsonResponse换成StreamingHttpResponse就可以进行流式传输
  	# 注意content_type必须是text/event-stream
	return StreamingHttpResponse(sse_stream(), content_type='text/event-stream')
    else:
    	# 带了标记但是不是true也不管他
    	return responseJson(400, None, "流式传输参数错误")
```

**客户端重新连接策略:**

```javascript
yield "retry: 5000\n\n"  # 5 seconds
```

**所有数据发送完成时：** 当服务器发送完所有数据并且生成器函数结束时，连接将自动关闭。在这种情况下，除非客户端尝试重新连接，否则连接不会再次建立。

**使用return：**在生成器函数中使用**return**将会结束该函数，从而结束**SSE**连接。如果客户端被设置为在连接断开时自动重连（**这是默认行为**），它可能会尝试重新连接。您可以通过发送一个特定的**retry**值来控制或禁止这种行为。例如，**yield "retry: 0\n\n"** 会告诉客户端在连接断开时不要重新连接。

```javascript
yield "retry: 0\n\n"
```

总的来说，当生成器函数不再产生输出时，**SSE**连接将结束，无论是由于函数自然结束还是由于某个**return**语句。

前端**js**
#### GET

```javascript
// 使用GET请求接收流式数据的类
class StreamDataFetcherGET {
    constructor(endpoint) {
        this.endpoint = endpoint;
    }

    // 初始化并监听数据
    init() {
        // 创建一个新的EventSource实例，连接到提供的endpoint
        const evtSource = new EventSource(this.endpoint);

        // 当从服务器收到新的数据时
        evtSource.onmessage = (event) => {
            const data = JSON.parse(event.data);
            this.renderData(data);
        };

        // 当与服务器的连接发生错误时
        evtSource.onerror = (error) => {
            console.error("EventSource failed:", error);
            evtSource.close();
        };
    }

    // 渲染接收到的数据
    renderData(data) {
        // 这里的代码取决于如何渲染数据到你的页面上
        console.log(data);
    }
}

// 使用GET请求的示例
const fetcherGET = new StreamDataFetcherGET("请求接口");
fetcherGET.init();
```
还可以先判断是否客户端支持是否支持**SSE**(下面POST请求同理)，示例如下

```javascript
// 我后台配置了只有带上?stream=true采用流式传输，你先判断浏览器类型支持SSE不，IE系列都不支持，然后特别老的Edge和狗都不用的浏览器也不支持，如果他不支持就不要走这个方法走你原来那个函数我就不cv了

// 检查是否支持SSE
if (typeof EventSource !== "undefined") {
    // 支持SSE，所以请求流式数据
    const evtSource = new EventSource("SSE的请求接口");

    evtSource.onmessage = function(event) {
        const data = JSON.parse(event.data);
        console.log(data);
    };

    evtSource.onerror = function(error) {
        console.error("EventSource failed:", error);
        evtSource.close(); // 断开SSE连接
    };
} else {
    // 不支持SSE，所以请求JSON数据
    fetch("JSON的请求接口")
        .then(response => response.json())
        .then(data => {
            console.log(data);
        })
        .catch(error => {
            console.error("Error fetching JSON data:", error);
        });
}

```
#### POST
对于**POST**请求，**EventSource**不适用，因为它==仅支持==**GET**请求。使用**POST**请求处理**SSE**需要一些额外的工作，如下所示：
```javascript
// 使用POST请求接收流式数据的类
class StreamDataFetcherPOST {
    constructor(endpoint, postData) {
        this.endpoint = endpoint;
        this.postData = postData;
    }

    // 初始化并监听数据
    async init() {
        // 使用fetch进行POST请求
        const response = await fetch(this.endpoint, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.postData)
        });

        const reader = response.body.getReader();
        const decoder = new TextDecoder();
        let buffer = '';

        while (true) {
            const { done, value } = await reader.read();
            if (done) break;
            
            buffer += decoder.decode(value, { stream: true });

            while (buffer.includes("\n")) {
                const lineEnd = buffer.indexOf("\n");
                const line = buffer.slice(0, lineEnd).trim();
                buffer = buffer.slice(lineEnd + 1);

                this.renderData(JSON.parse(line));
            }
        }
    }

    // 渲染接收到的数据
    renderData(data) {
        // 这里的代码取决于如何渲染数据到你的页面上
        console.log(data);
    }
}

// 使用POST请求的示例
const postData = { key: "value" };  // 替换为你的POST数据
const fetcherPOST = new StreamDataFetcherPOST("请求接口", postData);
fetcherPOST.init();
```

不用类

```javascript
async function createStreamFetcherPOST(endpoint, postData) {
    const response = await fetch(endpoint, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(postData)
    });

    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let resultString = '';

    while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        resultString += decoder.decode(value, { stream: true });

        while (resultString.includes("\n\n")) {
            const lineEnd = resultString.indexOf("\n\n");
            const line = resultString.slice(0, lineEnd).trim();
			
			// 检查是否有retry: 0消息
		    if (fullMessage === "retry: 0") {
		        console.log("Received retry: 0. Stopping connection...");
		        reader.cancel();  // 取消读取，这将导致流结束并跳出循环
		        return;  // 结束处理函数
		    }
			
			try {
                    const payload = JSON.parse(line.replace(/^data: /, ""));
                    renderData(payload);
                } catch (e) {
                    console.error("Error parsing segment:", e);
                }
            resultString = resultString.slice(lineEnd + 2);
        }
    }

    function renderData(data) {
        console.log(data);
        if (payload.code === 200) {
        // 处理数据，简单就设个变量累加就知道数量，不然渲染逻辑传参就在这
    	} else {
        // 错误的，简单就不累加咯
    	}
    }
}

// 使用POST请求的示例
const postData = { key: "value" };  // 替换为您的POST数据
createStreamFetcherPOST("your_POST_endpoint_url_here", postData);

```




### 效果
这个**EverntStream** 就是用了**SSE**的
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4a5022bbb863c9c87ff621cf0841992c.png)










## webSocket
`注意，其他都是基于HTTP协议，而websocket是基于TCP协议`
### 简介
**WebSocket**是一种在单个**TCP**连接上进行==全双工通信==的协议。**WebSocket**通信协议于2011年被**IETF**定为标准**RFC 6455**，并由**RFC7936**补充规范。**WebSocket API**也被**W3C**定为标准。
**WebSocket**使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在**WebSocket API**中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

#### WebSocket的工作原理:

1. 客户端发起WebSocket连接,通过HTTP协议发起Upgrade请求将连接升级为WebSocket连接。
2. 服务器端响应Upgrade请求,完成连接升级。
3. 建立WebSocket连接后,客户端和服务器端就可以通过这个连接通道自由地双向传输数据,无需等待对方的请求。

#### WebSocket的主要优点:

- **允许全双工通信:** 客户端和服务器端都可以主动发送数据。
- **更实时:** 服务器有新数据可以立即主动推送给客户端。
- **更轻量:** 建立连接的开销小,通信高效,减少不必要的网络请求。
- 利用HTTP协议做**升级握手**,默认端口是80和443,避免了跨域问题。
- **支持扩展**,可以扩展自定义的子协议。
#### WebSocket的主要缺点:

- 不如HTTP协议广泛应用,存在兼容性问题。
- 需要浏览器和服务器端都支持WebSocket协议,增加了开发成本。
- 有连接建立和关闭的开销,不适用于量小数据的交互。
 - 安全性需要额外考虑,通信内容是明文,需要加密。
- 处于连接状态时,会占用服务器端资源。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1ee1e43b44e27beea98eb79509fdff00.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6907b053820f750356ff82f444282d78.png)
简介就不多说了哈~大概图解一下就行了，他这个连接过程要详细知道得单独写一篇，直接进入实操。


### 实现

> 因为是基于**django**来实现的所以有些东西该配置还得配置，不同框架这个配置确实就不一样了，但是前端是一样的

**安装库**

这里可能会遇到一些问题，因为**channels**需要和**django**版本对应上，如有问题可以参考我的另一篇文章，[django使用channels实现webSocket启动失败](https://blog.csdn.net/m0_69082030/article/details/131745187?spm=1001.2014.3001.5501)，求顺手一赞嘿嘿。
```python
pip install channels
```
在**settings.py**同级目录下新增**asgi.py**文件(**django4**自带了)和**routing.py**文件，内容稍后说

目录结构如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7ffb405fd46c214b6af3eccccbfc0ae4.png)


**settings.py**(配置文件)

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'channels', # 注册这个
    'app01.apps.App01Config',
]


WSGI_APPLICATION = 'staffSystem_django.wsgi.application' # 原本只有这个东西，这个不去掉也可以
ASGI_APPLICATION = 'staffSystem_django.asgi.application' # 新增这个，就是你的asgi目录所在地

# channels 配置存在内存中
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels.layers.InMemoryChannelLayer",
    }
}
# channels 配置存在redis中
# CHANNEL_LAYERS = {
#    "default": {
#        "BACKEND": "channels_redis.core.RedisChannelLayer",  # pip install channels-redis
#        "CONFIG": {
#            "hosts": [('127.0.0.1', 6379)]
#            # "hosts": ["redis://127.0.0.1:6379/1"]
#        },
#    }
# }
```
`这里建议  channels 配置存在redis中，这只是demo才配在内存中，在setting.py修改这个配置就行了其他不用变`


**routing.py**（配置使用websocket的路由）

```python
# @Author: fbz
# @File : routing.py
from django.urls import path
from app01.views import chat

websocket_urlpatterns = [
    path(r'ws/<group>/',chat.wsChat.as_asgi()),
]
```

**asgi.py**（配置asgi）

```python
"""
ASGI config for staffSystem_django project.

It exposes the ASGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/4.0/howto/deployment/asgi/
"""

import os

from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from . import routing

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'staffSystem_django.settings')

# application = get_asgi_application() # 如果这样配置就收不到http请求了

application = ProtocolTypeRouter({
    'http': get_asgi_application(), # http请求走这个
    'websocket': URLRouter(routing.websocket_urlpatterns) # ws请求走这个
})
```

**urls.py**

```python
"""staffSystem_django URL Configuration

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/4.0/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  path('', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.urls import include, path
    2. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
"""
from django.conf import settings
from django.contrib import admin
from django.urls import path, re_path
from django.views.static import serve

from app01.views import depart, user, account, admin, pretty, task, order, chart, upload, city, chat

urlpatterns = [
	path('ws/chat/', chat.ws_chat),
]
```

配置完成启动**django**项目的效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54ca8c10996364609e3a00d5a5b30af4.png)


**ws_chat.html**（前端）

```python
{% extends 'layout.html' %}
{% block css %}
    <style>
        .message {
            height: 300px;
            border: 1px solid #dddddd;
            width: 100%;
        }
    </style>
{% endblock %}

{% block content %}
    <div class="message" id="message"></div>
    <div>
        <input type="text" placeholder="请输入" id="txt">
        <input type="button" value="发送" onclick="sendMessage();">
        <input type="button" value="关闭连接" onclick="closeConn();">
    </div>
{% endblock %}

{% block js %}
    <script>
        // socket = new WebSocket('ws://127.0.0.1:8000/ws/123/');
        socket = new WebSocket('ws://' + window.location.host + '/ws/{{group_id}}/');

        // 创建好连接之后自动触发(服务端执行self.accept())
        socket.onopen = function (event) {
            let tag = document.createElement('div');
            tag.innerText = '[连接成功]';
            document.getElementById('message').appendChild(tag);
        }

        // 当websocket接收到服务端发来的消息时，自动会触发这个函数
        socket.onmessage = function (event) {
            let tag = document.createElement('div');
            tag.innerText = event.data;
            document.getElementById('message').appendChild(tag);
        }

        // 服务端主动断开连接时，自动会触发这个方法
        socket.onclose = function (event) {
            let tag = document.createElement('div');
            tag.innerText = '[断开连接]';
            document.getElementById('message').appendChild(tag);
        }

        function sendMessage() {
            let tag = document.getElementById('txt');
            socket.send(tag.value);
        }

        function closeConn() {
            socket.close(); //向服务端发送断开连接的请求
        }

    </script>
{% endblock %}
```
#### 用法一

基本每一行都注释说明了，真是极致详细

```python
from channels.generic.websocket import WebsocketConsumer
from channels.exceptions import StopConsumer
from asgiref.sync import async_to_sync


def ws_chat(request):
    return render(request, 'ws_chat.html') # 显示界面


class wsChat(WebsocketConsumer):
    def websocket_connect(self, message):
        # 有客户端来向后端发送ws连接的请求时，自动触发。
        self.accept()  # 服务端允许和客户端创建连接
        print('连接成功')
        # 服务端允许和客户端创建连接(握手)
        self.accept()  # 同时请求WebSocket HANDSHAKING和WebSocket CONNECT，分别是握手和连接

    def websocket_receive(self, message):
        # 浏览器基于ws向后端发送数据，自动触发接收消息
        print(message)
        self.send('asdasd')
        # self.close()  # 服务端主动断开连接
        text = message['text']  # {'type': 'websocket.receive', 'text': 'asd'}
        if text == 'close':
            # 服务端主动断开连接
            self.close()  # 同时也会执行客户端断开连接方法 WebSocket DISCONNECT(websocket_disconnect())
            return  # 不再执行下面的代码，如果断开连接还发送消息会报错
            # raise StopConsumer() #如果服务端断开连接时，执行raise StopConsumer()，那么不会执行websocket_disconnect()方法

            # print('接收到消息：', text)
        self.send(f'接收到消息：{text}')  # 服务端给客户端发送消息

    def websocket_disconnect(self, message):
        # 客户端与服务端端开连接时自动触发(客户端主动端开连接)
        print('断开连接')
        raise StopConsumer()  # WebSocket DISCONNECT
```
#### 用法二
使用了组，这也是**django**特有的，像**flask**不是使用**channels**实现**websocket**的 他没有组这个概念。

修改上面的类

```python
class wsChat(WebsocketConsumer):
    def websocket_connect(self, message):
        # 有客户端来向后端发送ws连接的请求时，自动触发。
        print('连接成功')
        # 获取群号，获取路由匹配中的
        group = self.scope['url_route']['kwargs'].get('group')
        # 服务端允许和客户端创建连接(握手)
        self.accept()  # 同时请求WebSocket HANDSHAKING和WebSocket CONNECT，分别是握手和连接

        # 将这个客户端的连接对象加入到内存或redis中，取决于setting.py中CHANNEL_LAYERS
        # async_to_sync将异步转为同步
        async_to_sync(self.channel_layer.group_add)(group, self.channel_name)

    def websocket_receive(self, message):
        # 浏览器基于ws向后端发送数据，自动触发接收消息
        text = message['text']  # {'type': 'websocket.receive', 'text': 'asd'}
        if text == 'close':
            # 服务端主动断开连接
            self.close()  # 同时也会执行客户端断开连接方法 WebSocket DISCONNECT(websocket_disconnect())
            return  # 不再执行下面的代码，如果断开连接还发送消息会报错
            # raise StopConsumer() #如果服务端断开连接时，执行raise StopConsumer()，那么不会执行websocket_disconnect()方法

        # self.send(f'接收到消息：{text}')  # 服务端给客户端发送消息
        # 获取群号，获取路由匹配中的
        group = self.scope['url_route']['kwargs'].get('group')
        # 通知组内的所有客户端，执行 xx_oo 方法，在此方法中自己可以去定义任意的功能
        async_to_sync(self.channel_layer.group_send)(group, {'type': 'aa.bb', 'message': message}) # aa_bb,下划线变成点

    def aa_bb(self, event):
        text = event['message']['text']
        # 这是给组内的所有人发送消息，在websocket_receive中的self.send(text)才是给当前这个人发送消息
        self.send(text)

    def websocket_disconnect(self, message):
        print('断开连接')
        # 获取群号，获取路由匹配中的
        group = self.scope['url_route']['kwargs'].get('group')
        async_to_sync(self.channel_layer.group_discard)(group, self.channel_name)
        # 客户端与服务端端开连接时自动触发(客户端主动端开连接)
        raise StopConsumer()  # WebSocket DISCONNECT
```
该类实现了一个功能就是只有同个组的成员才可以收到消息，也就是我们的群聊功能，只有当群号(组号)一样的聊天室才可以收到同一个群的其他成员发送的消息。





### 效果

`注意！！注意看请求头，必须携带这些信息才能建立一个ws连接`

服务器需要检查**Upgrade**头信息,如果支持**WebSocket**,就返回**101**状态码和**Upgrade**头信息,表示切换协议。

 **WebSocket** 协议在建立连接时的 **Sec-WebSocket-Key** 头信息和魔法字符串的相关验证过程。这主要是为了防止恶意的 **WebSocket** 请求。
具体过程是:
1. 客户端发起 **WebSocket** 请求时,需要在请求头包含 **Sec-WebSocket-Key** 字段,内容为一个随机的字符串。
2. 服务器端收到请求后,会将客户端发来的 **Sec-WebSocket-Key** 后的值与一个特定的魔法字符串(**magic_string**) **"258EAFA5-E914-47DA-95CA-C5AB0DC85B11"** 拼接。
3. 对拼接后的字符串做 **SHA-1** 摘要,然后进行 **BASE-64** 编码,得到一个 **hash** 值。
4. 服务器端需要在响应头的 **Sec-WebSocket-Accept** 字段设置这个 **hash** 值。
5. 客户端收到响应后,也按相同的算法计算出一个 **hash** 值,与服务器返回的 **Sec-WebSocket-Accept** 值进行比较。
6. 如果两个 **hash** 值一致,则验证通过,确认服务器端支持 **WebSocket** 协议,然后建立连接。

这个验证过程可以防止普通的 HTTP 客户端与服务器建立 **WebSocket** 连接,保证了连接的安全性。所以它是 **WebSocket** 协议握手过程中的一个重要组成部分。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4512aec79cb0a08884edaa7410a64a6c.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16afec633f52d104dc0edd3da8e71347.png)





## 比较
**SSE与WebSocket的比较**
**WebSocket**是另一种用于实现实时双向通信的**Web**技术，它与**SSE**在某些方面有所不同。下面是**SSE**和**WebSocket**之间的比较：

- **数据推送方向：** **SSE**是服务器向客户端的**单向通信**，服务器可以主动推送数据给客户端。而**WebSocket**是**双向通信**，允许服务器和客户端之间进行实时的双向数据交换。
- **连接建立：** **SSE**使用基于**HTTP**的长连接，通过普通的**HTTP**请求和响应来建立连接，从而实现数据的实时推送。**WebSocket**使用==自定义的协议==，通过建立**WebSocket**连接来实现双向通信。
- **兼容性：**由于**SSE**基于**HTTP**协议，它可以在大多数现代浏览器中使用，并且不需要额外的协议升级。**WebSocket**在绝大多数现代浏览器中也得到了支持，但在某些特殊的网络环境下可能会遇到问题。
- **适用场景：** **SSE**适用于服务器向客户端实时推送数据的场景，如股票价格更新、新闻实时推送等。**WebSocket**适用于需要实时双向通信的场景，如聊天应用、多人协同编辑等。

根据具体的业务需求和场景，选择**SSE**或**WebSocket**取决于您的实际需求。如果您只需要服务器向客户端单向推送数据，并且希望保持简单易用和兼容性好，那么**SSE**是一个不错的选择。如果您需要实现双向通信，或者需要更高级的功能和控制，那么**WebSocket**可能更适合您的需求。

## 参考
[https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events#event_stream_format](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events#event_stream_format)

[https://developer.mozilla.org/en-US/docs/Web/API/EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)

[https://stackoverflow.com/questions/5195452/websockets-vs-server-sent-events-eventsource/5326159](https://stackoverflow.com/questions/5195452/websockets-vs-server-sent-events-eventsource/5326159)
