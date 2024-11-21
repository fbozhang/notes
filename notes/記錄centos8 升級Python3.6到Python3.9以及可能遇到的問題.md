
Centos8-默认的Python、python2版本为2.7，python3版本為3.6，不符合项目的需求。现在升级到3.9

## 下载Python3.9.9

```python
wget https://www.python.org/ftp/python/3.9.9/Python-3.9.9.tgz
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a19f761ac8cacee26526733a2213e945.png)
## 可能出現的問題及解決方案
注意這個需要在**root權限**下,不然會報**權限不夠**的錯誤，如下：

```python
Resolving www.python.org (www.python.org)... 151.101.108.223, 2a04:4e42:8c::223
Connecting to www.python.org (www.python.org)|151.101.108.223|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 25787134 (25M) [application/octet-stream]
Python-3.9.9.tgz.1: Permission denied

Cannot write to ‘Python-3.9.9.tgz.1’ (Success).
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dee9357bc1b8119adc8f2f11c942e1f0.png)

**切換root權限**

```python
su root 
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d8609bcdebcb27728ba68761e4c06ac4.png)
默認是不顯示密碼的，打完回車就行了
## 安裝python3.9

**解壓**
```python
tar zxvf Python-3.9.9.tgz	# 解壓
```

在Linux中紅色是壓縮文件；白色文件是一般性文件，如文本文件，配置文件，源码文件等。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c26aef87f7f82d4e20e31440108f6285.png)


>蓝色代表目录
>绿色代表可执行文件
>红色表示压缩文件
>浅蓝色表示链接文件
>灰色表示其他文件
>红色闪烁表示链接的文件有问题了
>黄色表示设备文件。

3.7版本之后需要一个新的包libffi-devel(我沒安裝好像也不影響，暫時沒發現什麽問題)

```python
yum install libffi-devel -y
```
 进入python文件夹，生成编译脚本(指定安装目录)。**如果這裏make的時候報錯往下面看解決方案**
```python
cd Python-3.9.9/	# 去到Python-3.9.9目錄
./configure --prefix=/usr/local/python3 --enable-optimizations	# 进入python文件夹，生成编译脚本(指定安装目录)：
make # 编译
make install	# 安裝
```
> **编译：make**
实际就是编译源代码，按照上一步生成makefile文件进行编译，并生成执行文件。
在 Linux环境下使用 GNU 的 make工具能够比较容易的构建一个属于你自己的工程，整个工程的编译只需要一个命令就可以完成编译、连接以至于最后的执行。不过这需要我们投入一些时间去完成一个或者多个称之为 Makefile 文件的编写。此文件正是 make 正常工作的基础。make 是一个命令工具，它解释 Makefile 中的指令（应该说是规则）。在 Makefile文件中描述了整个工程所有文件的编译顺序、编译规则。所以，首先我们要将所有要编译的文件信息写到makefile中，然后执行make，make就会按照makefile进行编译，这避免了程序在调试过程中频繁输入命令进行编译。但是makefile写起来比较复杂，规则很多。

## 可能出現的問題及解決方案
**注意！！編譯(make)的時候可能會報以下錯誤**

```python
make build_all CFLAGS_NODIST=" -fprofile-use -fprofile-correction" LDFLAGS_NODIST=""
make[1]: Entering directory `/usr/local/src/Python-3.8.0'
./python -E -S -m sysconfig --generate-posix-vars ;\
if test $? -ne 0 ; then \
	echo "generate-posix-vars failed" ; \
	rm -f ./pybuilddir.txt ; \
	exit 1 ; \
fi
Could not import runpy module
Traceback (most recent call last):
  File "/usr/local/src/Python-3.8.0/Lib/runpy.py", line 15, in <module>
    import importlib.util
  File "/usr/local/src/Python-3.8.0/Lib/importlib/util.py", line 14, in <module>
    from contextlib import contextmanager
  File "/usr/local/src/Python-3.8.0/Lib/contextlib.py", line 4, in <module>
    import _collections_abc
SystemError: <built-in function compile> returned NULL without setting an error
generate-posix-vars failed
make[1]: *** [pybuilddir.txt] Error 1
make[1]: Leaving directory `/usr/local/src/Python-3.8.0'
make: *** [profile-opt] Error 2
```

**导致原因：**
- 在低版本的gcc版本中带有 --enable-optimizations 参数时会出现上面问题
- gcc 8.1.0修复此问题


**解决方法如下**

- 升级gcc至8.1.0【不推荐】
- ./configure参数中去掉 --enable-optimizations
 

**即上一個步驟執行的命令修改為：**

```python
cd Python-3.9.9/
./configure --prefix=/usr/local/python3
make
make install
```

安裝成功可以檢查一下**python3.9**編譯器

```python
cd /usr/local/python3/bin
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed72094766a77c133cf448ea2826b712.png)

## 建立Python3和pip3的软鏈接:

```python
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

**注意！！因爲我們不是安裝而是升級，原來其實是有一個python3但是是指向python3.6的，我們需要把原來的python刪掉先不然會報錯，如下**

```python
ln: failed to create symbolic link ‘/usr/bin/python3’: File exists
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/14fe8cd217d282e8092e8fad6a37a52d.png)
**即上一個步驟執行的命令修改為：**

```python
# 删除原先的Python3和pip3
rm -rf /usr/bin/python3
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
rm -rf /usr/bin/pip3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

## 查看Python3和Pip3是否正确的被安装：

```python
python3 --version 
pip3 --version
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/35fa713caa524039de9af063ed539f4a.png)
**或者**

```python
# 注意是大寫V
python3 -V	
pip3 -V
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b1c08db7ce6e0d73d80430ff034c88d9.png)

`到此就完成升級版本了，可能運行會有個警告說沒有添加環境變量(PATH)，但是我記得不影響使用`


## 将/usr/local/python3/bin加入PATH

```python
# 一定要在root權限下面
vim /etc/profile	
```

```python
# 不在root權限執行這一條
sudo vim /etc/profile	
```

**按“I”，然后贴上下面内容：**

```python
  export PYTHON_HOME=/usr/local/python3
  export PATH=${PYTHON_HOME}/bin:$PATH
```

按ESC，输入 **:wq** (注意有個冒號)回车退出。


修改完执行行下面的命令，让配置生效：

```python
source /etc/profile
```

因爲不配環境變量也能python -V測試所以這個也不知道怎麽證明配好了，不過之前運行pip命令下載東西要是有警告沒有配PATH應該這樣配置后就不會有警告了




