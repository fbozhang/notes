

# 背景

居然还有万能的**pycharm**解决不了的**python**程序？？？

# 创建scrapy
由于**PyCharm**中没有直接创建**Scrapy**项目的选项,所以使用命令行创建一个项目

**安装scrapy**

```python
pip install scrapy
```
**查看版本**
能看版本就是安装成功
```python
scrapy version
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6bfbf6b323a9e8d4eae20b3cf5eb4213.png)

**创建一个scrapy项目**

```python
scrapy startproject yourprojectname
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec641f489e50cb90882395636468725b.png)

根据提示创建爬虫

```python
cd asd
scrapy genspider example example.com
```
这样就创建成功了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3693e4684ca4827ce36df1d33eb3db49.png)

# 难受的开始
使用**PyCharm**打开这个项目,却发现爬虫中的**parse**函数是灰色的框
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c2157fa8146e99b7e6701142ab722973.png)


```python
Signature of method 'ExampleSpider.parse()' does not match signature of the base method in class 'Spider'
```

看一下父类中的**parse**函数是如何定义的，因为我们是重写父类的方法，在**pycharm**点这个就行了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ded197389a1a24d3528336ebd52aa433.png)

进到父类可以看到有一个`**kwargs`参数
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ebbc4f3b0ff3de26e781e616ab594a38.png)

把`**kwargs`参数加入到自己的爬虫的**parse**上面去，灰框就不见了，总算看着不难受了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b7ea23ef08cba0b6fa93604bda2e945.png)
# 指定类型

但是还有一个问题，灰框只是看着难受，没有代码提示才是真的难受，`.`不出来谁懂啊
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/01e398b595191103fcfd5131b6048ab3.png)

运行一下打印他的类型看看，可以看到是`scrapy.http.response.html.HtmlResponse`类型

```python
scrapy crawl example # example是你的爬虫名字，也就是类里面的name属性
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25ed277e92e8787504203e7cc99206c2.png)
如果不想看到那么多烦人的日志信息就在**settings.py**中加上日志等级

```python
LOG_LEVEL = 'WARNING'
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e289eac7dc75b4b8fbea11d0d3846523.png)

重新运行看看效果，世界安静
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/926e0498c43cdfd1cb3085d2fe053a57.png)

既然知道是什么类型那就给他指定类型就行了

```python
from scrapy.http.response import Response

def parse(self, response: Response, **kwargs):
```
`.`出来了，舒服了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc0a0b43cc920a87c58a90422dae1d67.png)

# 修改模板并指定使用
终于可以看到有了正常的代码提示了，但是总不能每次都这样写吧，查看**genspider**命令,发现`-t`参数可以使用自定义模板
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8fbf582d8a076a6617e605febe0454c.png)

可以看到正常创建爬虫是使用**basic**这个模板
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c74c8ca7ca70610eba241a4bc6db1ab7.png)

在你的解释器路径下找到该文件夹
```python
\Python39\Lib\site-packages\scrapy\templates\spiders
```
可以在该路径下找到这个模板文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/77abbb86ea131666f433b42a12a851a4.png)

如果是用**pycharm**还可以这样操作更快一点，而且直接定位到解释器位置以免你用的是虚拟环境路径不一样

导入**scrapy**包然后按住键盘的**ctrl**键然后用鼠标左键点一下就可以跳过去他的源代码
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7f21f6feb8f1264192ddf612fe1d4d99.png)

点击后跳转到`__init__.py`，其实跳哪个不重要，只要属于**scrapy**这个包就好了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1b7677ae31bc44b603fa33cc5ced17ec.png)

然后就可以找到模板文件了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44be4f8a17933148aec28902e6a39b8e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71d0f304a80a26125418c822c56b8dad.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4569a72df265350857c9cdb6313e1a5.png)

这样就可以看见他的内容了，也可以右键打开他的文件夹
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9ce078638b877f49c41e451126cb710d.png)

我选择复制一份出来(你也可以在上面直接改那就不用使用`genspider -t`指定模板了)
**mytemplate.tmpl** 你想叫什么名字都可以
```python
import scrapy
from scrapy.http.response import Response


class $classname(scrapy.Spider):
    name = "$name"
    allowed_domains = ["$domain"]
    start_urls = ["$url"]

    def parse(self, response: Response, **kwargs):
        pass

```

然后就可以使用自定义的模板创建爬虫

```python
# scrapy genspider -t 模板名称 爬虫名称 域名
 scrapy genspider -t mytemplate test test.com
```
可以看见新建的爬虫没有一点问题，舒服的代码提示
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/009a1737d102d13bb88778e11a9e9113.png)

注意模板名称必须写对，不然就会报错
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d60cd1e8a9b00affa094fa113366347.png)

如果你忘了你的模板名称可以安装提示查看

```python
scrapy genspider --list
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/68a39ea9a337074ed7f19f406fa7794e.png)

# 运行scrapy
前面提到的启动**scrapy**需要在终端使用以下命令
```python
scrapy crawl example # example是你的爬虫名字，也就是类里面的name属性
```

这么麻烦？我都用万能的**pycharm**了就没有简单的办法让我运行和断点调试吗？
我就能不能用右键运行或调试？万能的`ctrl + shift + F10`吗？
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c96ad6185f3d2f3420a56183dfdbf11.png)
那肯定是可以的啊

在项目根目录也就**scrapy.cfg**同一层，**settings.py**的上一层创建**main.py**(你想叫啥都行)文件写入以下代码

**main.py**
```python
from scrapy.cmdline import execute
import os
import sys

if __name__ == '__main__':
    sys.path.append(os.path.dirname(os.path.abspath(__file__)))
    execute(['scrapy', 'crawl', 'example'])  # 把最后的参数换成你自己的爬虫名字
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0def72bed7958b6d6c15793ed5a534c6.png)

直接运行该文件就可以运行**scrapy**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3300b86d7c4e4772fb30078cc13a759a.png)

运行过一次后面就可以用`CTRL + F5`了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2053120f0113d6b3bf6b683d82c61ea2.png)

也可以使用**DEBUG**调试，可以看到已经停在断点处了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df1081204b0c5b9a25c9f5f1c6e10a36.png)
