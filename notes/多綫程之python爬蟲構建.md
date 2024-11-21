
# 多綫程
`以下定義來自百度百科，看看就好沒仔細寫`
## 定義
**多线程**（multithreading），是指从软件或者硬件上实现多个线程并发执行的技术。==具有多线程能力的计算机因有硬件支持而能够在同一时间执行多于一个线程，进而提升整体处理性能==。具有这种能力的系统包括对称多处理机、多核心处理器以及芯片级多处理或同时多线程处理器。在一个程序中，这些独立运行的程序片段叫作“**线程**”（Thread），利用它编程的概念就叫作“**多线程处理**”
## 簡介
在计算机编程中，一个基本的概念就是同时对多个任务加以控制。许多程序设计问题都要求程序能够停下手头的工作，改为处理其他一些问题，再返回主进程。可以通过多种途径达到这个目的。最开始的时候，那些掌握机器低级语言的程序员编写一些“**中断服务例程**”，主进程的暂停是通过硬件级的中断实现的。尽管这是一种有用的方法，但编出的程序很难移植，由此造成了另一类的代价高昂问题。中断对那些实时性很强的任务来说是很有必要的。但对于其他许多问题，只要求将问题划分进入独立运行的程序片断中，使整个程序能更迅速地响应用户的请求。

最开始，**线程**只是用于分配单个处理器的处理时间的一种工具。但假如操作系统本身支持多个处理器，那么每个线程都可分配给一个不同的处理器，真正进入“并行运算”状态。==从程序设计语言的角度看，多线程操作最有价值的特性之一就是程序员不必关心到底使用了多少个处理器==。程序在逻辑意义上被分割为数个线程；假如机器本身安装了多个处理器，那么程序会运行得更快，毋需作出任何特殊的调校。根据前面的论述，大家可能感觉线程处理非常简单。但必须注意一个问题：==共享资源==！如果有多个线程同时运行，而且它们试图访问相同的资源，就会遇到一个问题。举个例子来说，两个线程不能将信息同时发送给一台打印机。为解决这个问题，对那些可共享的资源来说（比如打印机），它们在使用期间必须进入**锁定状态**。所以一个线程可将资源锁定，在完成了它的任务后，再解开（释放）这个锁，使其他线程可以接着使用同样的资源 。

==线程是进程中的一部分，也是进程的的实际运作单位，它也是操作系统中的最小运算调度单位==。**进程中的一个单一顺序的控制流就是一条线程，多个线程可以在一个进程中并发。可以使用多线程技术来提高运行效率。**
**多线程是为了同步完成多项任务，不是为了提高运行效率，而是为了提高资源使用效率来提高系统的效率。线程是在同一时间需要完成多项任务的时候实现的**

## 原理
**实现多线程是采用一种并发执行机制。**

**并发执行机制原理**：简单地说就是把一个处理器划分为若干个短的时间片，**每个时间片依次轮流地执行处理各个应用程序，由于一个时间片很短，相对于一个应用程序来说，就好像是处理器在为自己单独服务一样，从而达到多个应用程序在同时进行的效果。**

**多线程**就是把操作系统中的这种并发执行机制原理运用在一个程序中，把一个程序划分为若干个子任务，多个子任务并发执行，每一个任务就是一个线程。这就是多线程程序。

**多线程技术**不但可以提高交互式，而且能够更加高效、便捷地进行控制。在对多线程应用的时候，可以使程序响应速度得到提高，从而实现速度化、高效化的特点。另外，**多线程技术存在的缺点也比较明显，需要等待比较长的时间之外，还会在一定程度上使程序运行速度降低，使工作效率受到一定的影响，从而对资源造成了浪费。**

> 具體可以查看操作系統的知識

## 优点
1. **使用线程可以把占据时间长的程序中的任务放到后台去处理。**
2. 用户界面可以更加吸引人，这样比如用户点击了一个按钮去触发某些事件的处理，可以弹出一个进度条来显示处理的进度 。
3. **程序的运行速度可能加快。**
4. ==在一些等待的任务实现上如用户输入、文件读写和网络收发数据等，线程就比较有用了==。在这种情况下可以释放一些珍贵的资源如内存占用等。
5. **多线程技术在IOS**软件开发中也有举足轻重的作用。


## 缺点
1. 如果有大量的线程，会影响性能，因为**操作系统需要在它们之间切换**
1. 更多的线程需要==更多的内存空间==
1. 线程可能会给程序带来更多“**bug**”，因此要小心使用
1. 线程的中止需要考虑其对程序运行的影响
1. 通常块模型数据是在多个线程间共享的，==需要防止线程死锁情况的发生==。


## 优势
**多进程程序结构和多线程程序结构有很大的不同，多线程程序结构相对于多进程程序结构有以下的优势：**
1. **方便的通信和数据交换**
线程间有方便的通信和数据**交换机制**。对于不同进程来说，它们具有独立的**数据空间**，要进行数据的传递只能通过通信的方式进行，这种方式不仅费时，而且很不方便。线程则不然，由于同一进程下的线程之间共享数据空间，所以一个线程的数据可以直接为其他线程所用，这不仅快捷，而且方便。
1. **更高效地利用CPU**
使用多线程可以加快应用程序的响应。这对图形界面的程序尤其有意义，当一个操作耗时很长时，整个系统都会等待这个操作，此时程序不会响应键盘、鼠标、菜单的操作，而使用多线程技术，将耗时长的操作置于一个新的线程，就可以避免这种尴尬的情况。

同时，**多线程使多CPU系统更加有效**。操作系统会保证当线程数不大于CPU数目时，不同的线程运行于不同的CPU上。







# 代碼框架實現
## 導包

```python
import random	# 隨機
import queue	# 隊列
import threading	# 綫程
import time	# 時間
import requests	# http請求
from bs4 import BeautifulSoup	# 靚湯解析
import pandas as pd	# 數據解析
```


## 打印類
讓打印的信息有顔色，具體可以看我另一篇文章：[print打印設置字體顔色](https://blog.csdn.net/m0_69082030/article/details/129153428)

```python
class BCOLOR:
    def __init__(self):
        self.HEADER = '\033[95m'
        self.OKBLUE = '\033[94m'
        self.OKGREEN = '\033[92m'
        self.ERROR = '\033[31m'
        self.WARNING = '\033[93m'
        self.FAIL = '\033[91m'
        self.ENDC = '\033[0m'
        self.BOLD = '\033[1m'
        self.UNDERLINE = '\033[4m'

    def yellow(self, s: str):
        print(self.WARNING + s + self.ENDC)

    def blue(self, s: str):
        print(self.OKBLUE + s + self.ENDC)

    def green(self, s: str):
        print(self.OKGREEN + s + self.ENDC)

    def red(self, s: str):
        print(self.ERROR + s + self.ENDC)
```

## 爬蟲類
這個案例啓用5個綫程
### 構造方法
```python
bprint = BCOLOR()
threadNum = 5   # 綫程數量
totalNum = 0


class Spider:
    def __init__(self):
        self.urlQueue = queue.Queue()
        self.proxies = self.getProxies()
```
**urlQueue = queue.Queue()** 建一個隊列來保存需要爬取的所有ur，每個綫程都有這裏取一條，利用了隊列先進先出的特點l
**self.proxies = self.getProxies()** 得到代理的ip，不用代理也行但是速度快不用代理容易被封ip，但是使用代理需謹慎


### 獲取代理
> 非必須操作，看個人選擇

```python
    def getProxies(self):
        pUrl = 'https://api.asdas.com/ip/get' # 這個url是假的，使用自己買的代理ip接口
        params = {
            'appKey': '',
            'appSecret': ''
        }
        while True:
            res = requests.get(pUrl, headers=self.getHeaders(), params=params).json()
            code = res.get('code')
            if code == 200:
                ip = res['data'][0].get('ip')
                port = res['data'][0].get('port')
                proxies = {
                    "http": f"http://{ip}:{port}",
                    "https": f"https://{ip}:{port}",
                }
                print(proxies)
                return proxies
```
也可以采用自己去爬一些免費代理建立代理詞，但是免費不太穩定可能隨時就失效了



### 設置headers

```python
    def getHeaders(self):
        user_agent_list = [
            "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36",
            "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36",
            "Mozilla/5.0 (Windows NT 10.0; WOW64) Gecko/20100101 Firefox/61.0",
            "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36",
            "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36",
            "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.101 Safari/537.36",
            "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0)",
            "Mozilla/5.0 (Macintosh; U; PPC Mac OS X 10.5; en-US; rv:1.9.2.15) Gecko/20110303 Firefox/3.6.15",
        ]

        headers = {
            'User-Agent': random.choice(user_agent_list),
        }
        return headers
```
其實隨便設一個ua就好了沒必要隨機拿一個，有ua就行了。現在可能都不咋檢查ua了，畢竟服務器檢測同一個ip發過來ua能有多不一樣。

如果不設置ua的話，原來的ua是`python-requests/2.28.1`，極有可能遇到反爬，可能會報錯，==狀態碼403或者418==，**418**就是那個**我是一個茶壺**，有興趣可以去搜一下梗。

**這裏大概説一下：**

	这个代码是在1998年作为传统的IETF April Fools‘ jokes被定义的在RFC2324，超文本咖啡罐控制协议，但是并没有被实际的HTTP服务器实现。RFC指定了这个代码应该是由茶罐返回给速溶咖啡。
> 这当然是一个玩笑，甚至连这个RFC的内容都是从歌词里引用过来的。原文如下：
The HTCPCP server is a teapot; the resulting entity body may be short and stout.
翻译成中文大概是，服务器是个茶壶，返回实体短又粗。short and stout 是一句歌词。

> “我是一个茶壶，我啥也不知道，你别找我。”
“我是一个茶壶，我啥也不知道，你找咖啡壶去。”
“我是一个茶壶，我不是咖啡壶，不能当咖啡壶用。”
“我是一个茶壶，我不是咖啡壶，禁止当咖啡壶用。”
“我是一个茶壶，我不是咖啡壶，咖啡煮坏了你自己负责。”
“我是一个茶壶，暂时还不能当咖啡壶用。”
“我是一个茶壶，虽然也能煮咖啡，但由于某些原因暂时不能煮咖啡。”

可以看一下這個寫的，[我是一個茶壺](https://blog.csdn.net/HermitSun/article/details/118942694)







### 獲取新session
```python
    def new_session(self):
        session = requests.session()
        url = 'https://www.scopus.com/'
        while True:
            try:
                session.get(url=url, headers=self.getHeaders(), proxies=self.proxies, timeout=(3.5, 7))
                bprint.yellow(s='new session')
                return session
            except:
                self.proxies = self.getProxies()
                bprint.yellow(s='network error')
```
獲取一個新的**session**，不管出現什麽**異常**就換**代理**，做的比較水，你也可以寫的細緻點


### 獲取源代碼

```python
    def get_html(self, session, url, flag):
        print(url)
        print('get_html')
        while True:
            try:
                res = session.get(url=url, headers=self.getHeaders(), proxies=self.proxies, timeout=(3.5, 7)).text
                # print(res)
                if flag in res:
                    return session, res
                session = self.new_session()
                bprint.red(s='error response')
            except:
                self.proxies = self.getProxies()
                bprint.yellow(s='network timeout')
```



這裏的**flag** 就是頁面源代碼的一段話，如果這段話在**res**裏證明確實爬到頁面源代碼了那就返回頁面頁面源代碼。這裏寫的簡單點就是不管什麽情況沒爬到就換一個**session**然後出異常就換代理


### 解析網頁

```python
 def analyze_out(self, session, res, dic: dict) -> list:
        """
        bs解析html
        :param html:
        :param excelData:
        :return:
        """
        print('start parse')
        bs = BeautifulSoup(res, "html.parser")  # 生成靓汤对象解析
        tbody = bs.find_all(class_="searchArea")  # 搜索区域
      
        data_list = []
        for tr in tbody:
            print(f'正在检查第 {tbody.index(tr) + 1} 条')
			# 這裏就是解析網頁，可以自己去補充，下面給一些示例提示
			....................
            th_input = tr.find('th').find('input')
            name = th_input.get('name')
            
            dic['name'] = name

            flag = '<span class="titleText">'
            in_url = f'https://www.百度.com/in/'	# 這裏也是假url需要換你自己的，這個就是有一個情況比如頁面内還嵌套一個鏈接你需要去解析子網頁

            session, res = self.get_html(session=session, url=in_url, flag=flag)
           
            dic = self.analyze_ins(res=res, dic=dic)	# 這裏就是解析子頁面
            # print(dic)
            data_list.append(dic)
        print('over parse')
        return data_list
```
這個案例是拿**BeautifulSoup**解析頁面舉例，也可以使用**xpath**，**re**等

### 解析子頁面

```python
    def analyze_ins(self, res, dic: dict) -> dict:
        """ 解析子頁面 """
        bs = BeautifulSoup(res, "html.parser")  # 生成靓汤对象解析
        title = bs.find(class_="titleText").text  # 搜索区域
        dic['title '] = ''.join(title )
        return dic
```
這裏使用 **''.join(title )** 主要是爲了防止爬不到報錯，整個代碼我很少用**try**捕獲異常

### 保存數據

```python
def saveMysql(self, dic: dict):
	這裏自己根據情況自己補充，我采用的是保存數據庫，想要保存csv還是什麽格式就導入相應的包
```


### 綫程任務

```python
    def thread_work(self, num):
        global totalNum
        print('start')
        session = self.new_session()
        while True:
            if self.urlQueue.empty():
                break

            url = self.urlQueue.get()
            dic = {
                'name': '',
                'name': '',
                'name': '',
                .....
                需要爬取的數據，這裏以一個字典的方式接收，先讓每一個建設爲空字符串免得後面報錯
            }

            flag = '<title>百度一下，你就知道</title>' # 這個flag根據情況自己設置，這裏以百度爲例，源代碼裏一點有這句話，沒有這句話證明是被反爬收到一個js還是假的html之類，總之不是我們要的源代碼
            session, res = self.get_html(session=session, url=url, flag=flag)
            dics = self.analyze_out(session=session, res=res, dic=dic)
            with threading.Lock():
                ''' 存数据库 '''
                for dic in dics:
                    self.saveMysql(dic=dic)
                    totalNum += 1
                    bprint.green(s=f'thread: {num} finishNum: {totalNum} data:{dic}')
```

設一個死循環，如果隊列裏還有需要爬的**url**就繼續分配給綫程去爬取，如果隊列空了就結束
- **urlQueue.empty()**，判斷隊列是否爲空，返回布爾值
- **urlQueue.get()** 在隊列裏取一個**url**，先進先出

### 得到url

```python
    def getExcelData(self):
        print('start readExcel')
        raw_data = pd.read_excel(self.file, header=0, keep_default_na=False)  # header=0表示第一行是表头，就自动去除了
        raw_list = raw_data.values.tolist()
        url_list = []
        for raw in raw_list:
            name = raw[1]

            url = f'https://www.百度.com/search/name={LastName}'	# 這裏依舊是一個假url，就是提示一下怎麽構建url，根據個人情況修改
            url_list.append(url)

        print('over readExcel')
        return url_list
```

這裏因爲是爬取**excel**裏的地址所以才這樣，根據自己情況構建**url**隊列即可
這裏用的是**pandas**來讀取**excel**，也可以使用**openpyxl**，**xlwt**或者**csv**等等庫來讀寫**excel**

==pd.read_excel(self.file, header=0, keep_default_na=False)== 
-  **header=0**表示第一行是表头，就自动去除了；
-  **keep_default_na=False**：假設表格中有一些空行還是空格什麽的不是設置這個則是為**nal**值，設置了這個為空字符串（即：‘’）

**raw_data.values.tolist()**：將**DataFrame**類型轉換為**list**列表類型


### 啓動多綫程爬蟲

```python
    def run(self):
        start = time.perf_counter()	 # 記錄開始時間

        global threadNum
        bprint.green(s=f'program starts with {threadNum} threads')
        url_list = self.getExcelData()	# 獲取url列表

        for url in url_list:
            self.urlQueue.put(url)	# 將列表裏的url加入隊列

        threadList = []
        for i in range(threadNum):
            t = threading.Thread(target=self.thread_work, args=(i + 1,))
            threadList.append(t)
            t.start()

        for t in threadList:
            t.join()

        bprint.green(s='program finishs')

        end = time.perf_counter()
        print('Running time: %s Seconds' % (end - start))	# 打印執行時間
```

調用這個方法運行這個爬蟲類，具體多綫程的使用可以看我另一篇文章：[demo](https://blog.csdn.net/m0_69082030/article/details/129153411)


# 總結
這只是寫了一個大概的框架沒有具體實現一個案例，把單綫程爬蟲的邏輯在這裏面修改就可以啓用多綫程，理論上多綫程比單綫程快了幾十倍不止，因爲少了阻塞的問題有點像操作系統的流水綫。還有之所以這個案例不用多進程就是因爲多進程會變得亂序和消耗資源，多綫程至少總體沒那麽亂序。
