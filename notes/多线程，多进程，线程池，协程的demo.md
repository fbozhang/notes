


单线程太慢了所以我们尝试用多线程，多进程，线程池还是协程来提升程序速度

<mark>以下是使用的一些demo<mark/>

# 多线程
## 导包

```python
from threading import Thread  # 线程类
```


## 方法1
```python
线程
方法1
def asd():
    for i in range(1000):
        print("asd", i)


if __name__ == '__main__':
    t = Thread(target=asd)  # 创建线程并给线程安排任务
    t.start()  # 多线程状态为可以开始工作状态, 具体执行时间由CPU决定
    for i in range(1000):
        print("qwe", i)
```

## 方法2
```python
方法2
class MyThread(Thread):
    def run(self):  # 固定的 -> 当线程被执行之后, 被执行的就是run()
        for i in range(1000):
            print("子线程", i)


if __name__ == '__main__':
    t = MyThread()  # 创建一个对象
    # t.run() # 方法的调用 -> 单线程
    t.start()

    for i in range(1000):
        print("主线程", i
```

## 传递参数

```python
def asd(name):
    for i in range(1000):
        print(name, i)

if __name__ == '__main__':
    t1 = Thread(target=asd, args=("ads", ))  # 传递参数必须是元组
    t1.start()

    t2 = Thread(target=asd, args=("qwe",))  # 传递参数必须是元组
    t2.start()
```


# 多进程
## 导包

```python
from multiprocessing import Process  # 进程类
```

## 代码实现
```python
def asd():
    for i in range(1000):
        print("子进程", i)

if __name__ == '__main__':
    p = Process(target=asd)  # 创建线程并给线程安排任务
    p.start()  # 多线程状态为可以开始工作状态, 具体执行时间由CPU决定
    for i in range(1000):
        print("主进程", i)
```


# 线程池

## 导包

```python
from concurrent.futures import ThreadPoolExecutor
```
## 代码实现
```python
import requests
import csv

f = open("data.csv", mode="w", encoding="utf-8")
csvwriter = csv.writer(f)


def html(current):
    url = "http://www.xinfadi.com.cn/getPriceData.html"
    data = {
        "limit": "20",
        "current": current,
        "pubDateStartTime": "",
        "pubDateEndTime": "",
        "prodPcatid": "",
        "prodCatid": "",
        "prodName": ""
    }
    resp = requests.post(url, data=data)
    dic = resp.json()
    lists = dic['list']
    for list in lists:
        data_list = []
        prodName = list['prodName']
        lowPrice = list['lowPrice']
        avgPrice = list['avgPrice']
        highPrice = list['highPrice']
        place = list['place']
        pubDate = list['pubDate']
        data_list.append(prodName)
        data_list.append(lowPrice)
        data_list.append(avgPrice)
        data_list.append(highPrice)
        data_list.append(place)
        data_list.append(pubDate)
        # 把数据存放在文件中
        # print(data_list)
        csvwriter.writerow(data_list)
    print(url, "提取完毕")


if __name__ == '__main__':
    # for i in range(1, 14900):   # 单线程效率极低
    #     html(i)

    # 创建线程池
    with ThreadPoolExecutor(50) as t:
        for i in range(1, 100):  # 99*20个数据
            # 把下载任务提交给线程池
            t.submit(html, i)
    print("over!!")
```

> 线程池总体比较有序，多进程可能乱序了



# 协程
## 导包

```python
import asyncio
```

## demo1
```python
async def asd():    # 异步协程函数
    print("asd")

if __name__ == '__main__':
    f = asd()   # 此时的函数是异步协程函数, 执行函数得到的是一个协程对象
    print(f)
    asyncio.run(f)  # 协程程序运行需要asyncio模块的支持
```

## demo2
```python
async def asd1():  # 异步协程函数
    print("asd")
    # time.sleep(3)   # time.sleep相当于同步操作，当程序出现了同步操作的时候， 异步就中断了
    await asyncio.sleep(3)      # 异步操作的代码
    print("asd")


async def asd2():  # 异步协程函数
    print("qwe")
    # time.sleep(2)
    await asyncio.sleep(2)
    print("qwe")


async def asd3():  # 异步协程函数
    print("zxc")
    # time.sleep(4)
    await asyncio.sleep(4)
    print("zxc")


if __name__ == '__main__':
    a1 = asd1()
    a2 = asd2()
    a3 = asd3()
    tasks = [
        a1, a2, a3
    ]
    t1 = time.time()
    # 一次性启动多个任务(协程)
    asyncio.run(asyncio.wait(tasks))
    t2 = time.time()
    print(t2-t1)
```

## demo3

```python
async def asd1():  # 异步协程函数
    print("asd")
    # time.sleep(3)   # time.sleep相当于同步操作，当程序出现了同步操作的时候， 异步就中断了
    await asyncio.sleep(3)  # 异步操作的代码
    print("asd")


async def asd2():  # 异步协程函数
    print("qwe")
    # time.sleep(2)
    await asyncio.sleep(2)
    print("qwe")


async def asd3():  # 异步协程函数
    print("zxc")
    # time.sleep(4)
    await asyncio.sleep(4)
    print("zxc")


async def main():
    tasks = [
        asyncio.create_task(asd1()),    # 把协程对象变成task对象
        asyncio.create_task(asd2()),
        asyncio.create_task(asd3())
    ]
    await asyncio.wait(tasks)


if __name__ == '__main__':
    t1 = time.time()
    # 一次性启动多个任务(协程)
    asyncio.run(main())
    t2 = time.time()
    print(t2 - t1)
```

## 爬虫demo实现
### 导包

```python
import asyncio
import aiohttp
```

```python
urls = [
    "http://kr.shanghai-jiuxin.com/file/mm/20211129/cv5xpueozr5.jpg",
    "http://kr.shanghai-jiuxin.com/file/mm/20211129/wjgc4eu1rqr.jpg",
    "http://kr.shanghai-jiuxin.com/file/mm/20211129/1dl52zzfawi.jpg"
]

async def aiodownload(url):
    # s = aiohttp.ClientSession()     <==> requests
    # s.get() -> requests.get()
    name = url.rsplit("/", 1)[1]  # 从右边切，切一次，得到[1]位置的内容
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            # resp.content.read() => resp.content
            with open(name, mode="wb") as f:
                f.write(await resp.content.read())  # 读取内容是异步的, 需要await挂起
    print(name, "over")

async def main():
    tasks = []
    for url in urls:
        tasks.append(asyncio.create_task(aiodownload(url)))
    await asyncio.wait(tasks)

if __name__ == '__main__':
    asyncio.run(main())
```

