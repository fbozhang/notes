# 問題
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ddc0af040d6cb9aec08e4c85c64abc93.png)
在**anaconda**打開**jupyter notebook**無法通過`CTRL+C`來終結服務，那每**launch**一下就啓動一個服務，導致啓動很多服務看著就無語，占用一堆端口			
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2cef98ad93cb82785bf7f9cad5d1effb.png)


# 解決方案

## 手動
 **1. 在終端輸入命令看啓動了多少個服務**

```python
jupyter-notebook list # 有沒有-都行(jupyter notebook list也可以)
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/026a29f56fb83d75c13defefe553368d.png)



 **2. 通過端口號看PID**

```python
netstat -o -n -a | findstr :8889 # 寫自己的端口
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d725a27551e818361ce36ee265edfa9a.png)


**3. 殺死進程**

```python
taskkill /F /PID 17376 # 寫自己的PID
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/60529e323b62b234afdc0f245f655580.png)

## 自動

多開一個那敲命令還好，十幾個那就受不了了，所以寫了個程序解決

```python
port_lines = !jupyter notebook list
# print(port_lines)

ports = []
for line in port_lines:
    if port_lines.index(line) == 0:
        continue
    port = line.split('/?')[0].split('http://localhost:')[1]
    # print(port)
    ports.append(port)
# print(ports)

pids = []
for port in ports:
    if port == '8888':  # 這裏因爲我jupyter notebook在用著8888，我還想接著用所以讓他跳過這個
        print('continue')
        continue
    info= !netstat -o -n -a | findstr :$port
    if len(info) != 0:
        pid = info[0].split('LISTENING       ')[1]
        !taskkill /F /PID $pid

```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6cc1229d2b085aef9783479a3f62701c.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0d31bcb3b399038e4fc4908c93f3dad.png)
# 總結

遇到問題記錄一下
