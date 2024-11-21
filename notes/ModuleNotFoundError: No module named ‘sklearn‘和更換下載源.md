
# 项目场景：

ModuleNotFoundError: No module named 'sklearn'
---

# 问题描述

```python
ModuleNotFoundError: No module named 'sklearn'
```

---

# 原因分析：
沒有安裝sklearn，**注意**要裝的不是sklearn而是scikit-learn

---

# 解决方案：
>這裏提供兩種解決方案


1. pip 安装

```python
pip install scikit-learn
```

2. 如果使用的是anaconda，建议使用conda命令安装


```python
conda install scikit-learn
```

後面會有個問你確定不的操作，輸入y回車就好了



# 下載源
如果嫌太慢可以用下載源


**常見國内源如下：**

```python
清华：https://pypi.tuna.tsinghua.edu.cn/simple
中国科学技术大学 https://pypi.mirrors.ustc.edu.cn/simple/
华中理工大学：http://pypi.hustunique.com/
阿里云：http://mirrors.aliyun.com/pypi/simple/
豆瓣：http://pypi.douban.com/simple/
百度：https://mirror.baidu.com/pypi/simple
```

## 臨時使用的命令是


```python
pip install 包名 -i 下載源地址
```

**例子：**

```python
pip install pandas -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 永久修改
### 方法一：修改配置文件
#### Mac/Linux
在Mac/Linux系统下：配置文件位置在 ~/.pip/pip.conf

如果是新安装的就没有这个文件，需要自己创建.pip目录：

```python
mkdir ~/.pip
```

完成后，打开pip.conf修改为（以清华源为例）：

```python
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host = https://pypi.tuna.tsinghua.edu.cn
```

最后使用查看是否修改了：

```python
pip config list   
```

如果出现下面文字就说明修改成功了：


```python
global.index-url='https://pypi.tuna.tsinghua.edu.cn/simple'
install.trusted-host='https://pypi.tuna.tsinghua.edu.cn'
```

#### Windows
 win环境pip的配置文件在C:\Users\xxx\AppData\Roaming\pip\pip.ini里面（主要也是看你python安裝在哪），可以打开此文件（没有就自己创建一个）直接修改，同样以清华源为例：

```python
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host = pypi.tuna.tsinghua.edu.cn
```

同样可用pip config list 查看。出现下面的文字就表示成功了

```python
global.index-url='https://pypi.tuna.tsinghua.edu.cn/simple'
install.trusted-host='https://pypi.tuna.tsinghua.edu.cn'
```

### 方法二：使用命令行（所有系统都适用）

输入下面的命令可直接永久设置以清华源爲例

```python
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
同样可以使用pip config list查看是否设置成功：出现下面的文字就表示成功了

```python
global.index-url='https://pypi.tuna.tsinghua.edu.cn/simple'
install.trusted-host='https://pypi.tuna.tsinghua.edu.cn'
```

`上面都是以清華源爲例，也可以使用別的`










