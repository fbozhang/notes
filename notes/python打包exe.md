

# 前言
python写的程序要怎么办才能使没有没有python环境的人使用

---



# 一、pyinstaller是什么？
pyinstaller是一个非常简单的打包python的py文件的库。用起来就几条命令就够了

PyInstaller 是一个用来将 Python 程序打包成一个独立可执行软件包，支持 Windows、Linux 和 Mac OS X。
PyInstaller 可以读取您编写的 Python 脚本。它分析您的代码以发现脚本执行所需的所有其他模块和库。然后，它将收集所有这些文件的副本 - 包括活动的 Python 解释器！- 并将其与脚本一起放在单个文件夹中，或者可选地在单个可执行文件中。
PyInstaller 已针对 Windows，Mac OS X 和 GNU / Linux 进行了测试。但是，它不是交叉编译器：要制作 Windows 应用程序，请在 Windows 中运行 PyInstaller。要创建 GNU / Linux 应用程序，请在 GNU / Linux 等环境中运行它。PyInstaller 已成功与 AIX，Solaris，FreeBSD 和 OpenBSD 结合使用，但未在持续集成测试中针对它们进行测试。

# 二、使用步骤
## 1.安装库

```python
pip install pyinstaller
```
**运行结果：**

```python
Successfully installed pyinstaller-x.x.x
```
其中的 x.x.x 代表 PyInstaller 的版本

## 2.pyInstaller生成可执行程序
**PyInstaller 工具的命令语法如下：**

```python
 pyinstaller 选项 Python 源文件
```
在cmd执行上面命令，不管这个 Python 应用是单文件的应用，还是多文件的应用，只要在使用 pyinstaller 命令时编译作为程序入口的 Python 程序即可

> PyInstaller工具是跨平台的，它既可以在 Windows平台上使用，也可以在 Mac OS X 平台上运行。在不同的平台上使用 PyInstaller 工具的方法是一样的，它们支持的选项也是一样的。
### 2.1带命令行的打包
**举例**
新建一个asd.py文件，文件代码如下：

```python
def main():
    print('程序开始执行')
    print("哈哈哈哈哈")
# 增加调用main()函数
if __name__ == '__main__':
    main()
```
接下来在该目录下打开cmd执行命令

> 必须注意这里的F是大写的

```python
pyinstaller -F asd.py
```
执行上面命令，将看到详细的生成过程，提示Successfully即成功。当生成完成后，将会在当前 目录下看到多了一个 dist 目录，并在该目录下看到有一个 app.exe 文件，这就是使用 PyInstaller 工具生成的 exe 程序
在命令行窗口中进入 dist 目录下，在该目录执行asd.exe，会看到程序输出以下结果：

```python
程序开始执行
哈哈哈哈哈
```
由于该程序没有图形用户界面，因此如果读者试图通过双击来运行该程序，则只能看到程序窗口一闪就消失了，这样将无法看到该程序的输出结果。
可以修改以上程序加一个I/O操作使得程序没有结束即可双击打开程序不会一闪而过
**修改例子**

```python
def main():
    print('程序开始执行')
    print("哈哈哈哈哈")
    s = input()
# 增加调用main()函数
if __name__ == '__main__':
    main()
```
在上面命令中使用了-F 选项，该选项指定生成单独的 EXE 文件，因此，在 dist 目录下生成了一个单独的大约为 6MB 的asd.exe 文件（在 Mac OS X 平台上生成的文件就叫 asd，没有后缀）；与 -F 选项对应的是 -D 选项（默认选项），该选项指定生成一个目录（包含多个文件）来作为程序。

下面先将 PyInstaller 工具在当前目录下生成的 build、dist 目录删除，并将asd.spec 文件也删除，然后使用如下命令来生成 exe文件。

```python
pyinstaller -D asd.py
```
执行上面命令，将看到详细的生成过程。当生成完成后，将会在当前目录下看到多了一个 dist 目录，并在该目录下看到有一个 asd 子目录，在该子目录下包含了大量 .dll 文件和 .pyz 文件，它们都是 asd.exe 程序的支撑文件。在命令行窗口中运行该 asd.exe 程序，同样可以看到与前一个asd.exe 程序相同的输出结果。
### 2.2pyinstaller支持的选项
pyInstaller 不仅支持 -F、-D 选项还支持以下选项


 
|-h，--help| 查看该模块的帮助信息 |
|--|--|
-F，-onefile | 产生单个的可执行文件
-D，--onedir |产生一个目录（包含多个文件）作为可执行程序
-a，--ascii |不包含 Unicode 字符集支持
-d，--debug |产生 debug 版本的可执行文件
-w，--windowed，--noconsolc |指定程序运行时不显示命令行窗口（仅对 Windows 有效）
-c，--nowindowed，--console |指定使用命令行窗口运行程序（仅对 Windows 有效）
-o DIR，--out=DIR |指定 spec 文件的生成目录。如果没有指定，则默认使用当前目录来生成 spec 文件
-p DIR，--path=DIR |设置 Python 导入模块的路径（和设置 PYTHONPATH 环境变量的作用相似）。也可使用路径分隔符（Windows 使用分号，Linux 使用冒号）来分隔多个路径
-n NAME，--name=NAME |指定项目（产生的 spec）名字。如果省略该选项，那么第一个脚本的主文件名将作为 spec 的名字

> 上面列出的只是 PyInstaller 模块所支持的常用选项，如果需要了解 PyInstaller 选项的详细信息，则可通过 pyinstaller -h 来查看。

### 2.3带图形化程序的打包（例如pygame）
下面再举例一个带图形用户界面的程序
**例子**
创建qwe.py

```python
import pygame

version = "v1.02"
COLOR_GOLDEN = pygame.Color(255, 215, 0)

class MainGame():
    # 游戏主窗口
    window = None
    SCREEN_WIDTH = 700
    SCREEN_HEIGHT = 400

    def __init__(self):
        pass

    # 开始游戏方法
    def startGame(self):
        pass
        # 创建窗口加载窗口
        MainGame.window = pygame.display.set_mode([MainGame.SCREEN_WIDTH, MainGame.SCREEN_HEIGHT])
        # 设置游戏标题
        pygame.display.set_caption("坦克大战" + version)
        # 让窗口持续刷新操作
        while True:
            # 给窗口填充颜色
            MainGame.window.fill(COLOR_GOLDEN )
            # 窗口的刷新
            pygame.display.update()


if __name__ == '__main__':
	MainGame().startGame()
```
在当前目录用命令行执行以下命令
```python
pyinstaller -F -w qwe.py
```
上面命令中的 -F 选项指定生成单个的可执行程序，-w 选项指定生成图形用户界面程序（不需要命令行界面）。运行上面命令，该工具同样在当前目录下生成了一个 dist 子目录，并在该子目录下生成了一个 qwe.exe 文件。
直接双击运行 qwe.exe 程序（该程序有图形用户界面，因此可以双击运行），可自行查看运行结果。

---
因为没有做退出事件处理所以运行上面例子会报错或卡住
**修改例子**

```python
import pygame

version = "v1.02"
COLOR_GOLDEN = pygame.Color(255, 215, 0)


class MainGame():
    # 游戏主窗口
    window = None
    SCREEN_WIDTH = 700
    SCREEN_HEIGHT = 400

    def __init__(self):
        pass

    # 开始游戏方法
    def startGame(self):
        # 创建窗口加载窗口
        MainGame.window = pygame.display.set_mode([MainGame.SCREEN_WIDTH, MainGame.SCREEN_HEIGHT])
        # 设置游戏标题
        pygame.display.set_caption("坦克大战" + version)
        # 让窗口持续刷新操作
        while True:
            # 给窗口填充颜色
            MainGame.window.fill(COLOR_GOLDEN )
            # 在循环中持续完成事件的获取
            self.getEvent()
            # 窗口的刷新
            pygame.display.update()

    # 获取程序运行期间所有事件(鼠标事件，键盘事件)
    def getEvent(self):
        # 1.获取所有事件
        eventList = pygame.event.get()
        # 2.对事件进行判断处理(1. 点击关闭按钮  2.按下键盘上的某个键)
        for event in eventList:
            # 判断event.type  是否QUIT，如果是退出的话，直接调用程序结束方法
            if event.type == pygame.QUIT:
                sys.exit()
```

> 注意上面退出不能使用exit()关闭python解释器操作，不然在python解释器运行正常但是打包exe运行会报错

这个简单例子仅演示了如何打包exe文件，例子内容没有实际作用
### 2.4打包程序加logo
使用下方指令指令

```python
pyinstaller -F -w -i icofile filename
```
icofile表示图标的位置，建议直接放在程序文件夹里面，这样子打包的时候直接写文件名就好
这里我的.ico文件名为logo.ico

```python
pyinstaller -F -w -i logo.ico qwe.py
```

> 注意如果有用到图片视频音频等资源文件，需要将用到的所有资源包放入dist目录中，不然会报错。



## 3.Mac系统下打包生成.app应用程序

```python
1安装py2app 

pip3 install  py2app 

2生成setup.py 文件

py2applet  --make-setup xx.py (需要打包的文件)

3打包文件

python3 setup.py py2app 

遇到的问题，打包gui文件时，按钮展示不了 （不知道啥原因，现在还未解决）
```

以上就是mac和windows 环境下打包python文件的方式和命令

## 如果想将pyinstaller打包的exe文件制作成安装包或者反编译可以看下下面两篇博文

```python
https://blog.csdn.net/qq_42004597/article/details/89087465?spm=1001.2014.3001.5502
https://blog.csdn.net/qq_42004597/article/details/105532503?spm=1001.2014.3001.5502
```



---

# 总结
个人使用发现把exe文件移动到原目录与资源文件目录同一级也可以使用，打包后产生的build和dist目录删掉不影响使用
