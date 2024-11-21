

## 背景
如何把自己写的代码，打包成库方便其他人使用

## 安装setuptools库
正所谓想要富先修路，先把造轮子要用的库装上

```python
pip install wheel
pip install setuptools
```

## 准备要打包的代码

本文我将拿最经典的代码为例。

**hello.py**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61202fe296d4b15f13c4088b6ab8931d.png)

包名就打算叫**big_word**了。


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/35b68f0a0b123e04354f025a0417e5ae.png)


## 创建setup.py文件
在包内目录下创建一个`setup.py`文件，并按照以下格式写入一个**setup**方法(我就挑了几个写)。

```python
from setuptools import setup

setup(name='mingnanqu', # 库的名称(安装的时候用的，即pip install name)
      version='1.0',    # 版本号
      description='word很大',
      author='mingnanqu',
      author_email='xxx@gmail.com',
      py_modules=['hello'],
)
```
> setup.py参数介绍：
name : 打包起来的包的文件名
version : 版本号,添加为打包文件的后缀名
author : 作者
author_email : 作者的邮箱
py_modules : 打包的.py文件
packages: 打包的python文件夹
include_package_data : 打包非py文件的目录
license : 支持的开源协议
description : 对项目简短的一个形容
ext_modules : 是一个包含Extension实例的列表,Extension的定义也有一些参数。
ext_package : 定义extension的相对路径
requires : 定义依赖哪些模块
provides : 定义可以为哪些模块提供依赖
data_files :指定其他的一些文件(如配置文件),规定了哪些文件被安装到哪些目录中。如果目录名是相对路径,则是相对于sys.prefix或sys.exec_prefix的路径。如果没有提供模板,会被添加到MANIFEST文件中。


还是给我完整通用一点的吧 ^ _ ^

```python
import os

from setuptools import find_packages, setup

with open("requirements.txt") as f:
    requirements = f.read().splitlines()

with open(
    os.path.join(os.path.dirname(__file__), "README.md"), encoding="utf-8"
) as readme:
    README = readme.read()

setup(
    name="mingnanqu",  # 项目名称
    version="1.0.0",  # 项目版本
    description="word很大",  # 项目的简短描述
    packages=find_packages(),  # 包含项目中的所有包
    author='mingnanqu',
    author_email='xxx@gmail.com',
    # url
    include_package_data=True,  # 包含非代码文件（如模板、静态文件等）
    long_description=README,  # 项目的详细描述，从 README.md 文件获取
    long_description_content_type="text/markdown",  # 详细描述的内容类型
    install_requires=requirements,  # 项目的依赖列表
)
```

## 打包生成whl文件

进到`setup.py`的目录，打开`cmd`窗口：
输入

```python
python setup.py bdist_wheel
```
如果出现以下信息，就说明已经打包成功了。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54fc6efce115c8bd593872c579d0d165.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0cef9a2551022b3530a07675826b619.png)

`setup.py`所在的目录下会多几个文件夹。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0b8e11cc0592fd67b39041e7886aaeb9.png)

打包好的库就在**dist**中
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4a6330095d9771ca7a0d3f8967fd0b77.png)
## 把库装到电脑上

在库所在的目录下打开**cmd**，并输入

```python
pip install 打包成库的文件名（whl文件）
```
这个大家都懂就不截屏了

使用`pip list`命令查看本地是否已成功安装，验证一下就行了

## 使用这个库
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b688e69ccd702cdaca563333a1fa13b1.png)



需要注意这里是	`import 包(目录/文件)名`，而不是这个库(**mingnanqu**)的名字,库的名字只用于pip而已，当然你可以起同一个名字


这样就实现通过**whl**文件可以让别的伙伴也能使用你造的轮子啦。
