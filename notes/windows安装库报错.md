

## 报错信息

```python
ERROR: Command errored out with exit status 1: ‘D:\test\venv\Scripts\python.exe’ -u -c ‘import io, os, sys, setuptools, tokenize; sys.argv[0] = ‘"’"‘C:\Users\aaa\AppData\Local\Temp\pip-install-j
oni55ju\xxx_350c8d1094f749eb97d8f0496606cd1c\setup.py’"’"’; file=’"’"‘C:\Users\aaa\AppData\Local\Temp\pip-install-joni55ju\xxx_350c8d1094f749eb97d8f0496606cd1c\setup.py’"’"’;f = ge
tattr(tokenize, ‘"’"‘open’"’"’, open)(file) if os.path.exists(file) else io.StringIO(’"’"‘from setuptools import setup; setup()’"’"’);code = f.read().replace(’"’"’\r\n’"’"’, ‘"’"’\n’"’"’);f.close();exec
(compile(code, file, ‘"’"‘exec’"’"’))’ install --record ‘C:\Users\aaa\AppData\Local\Temp\pip-record-6jnkeqfs\install-record.txt’ --single-version-externally-managed --compile --install-headers ‘D:\test\venv\include\site\python3.6\xxx’ Check the logs for full command output.
```

上面的`xxx`是我要安装的库


## 原因
这是由于 Windows 中有一个最大路径长度限制，默认值为 260 个字符。当文件的完整路径超过这个长度时，就会遇到这种错误。


## 解决方案
### 启用长路径支持
- 对于 Windows 10 (版本 1607 及更高版本)，可以通过组策略编辑器来启用长路径。方法是：

	1. `win + R`运行 `gpedit.msc` 打开**本地组策略编辑器**
	1. 导航到“**本地计算机策略**” → “**计算机配置**” → “**管理模板**” → “**系统**” → “**文件系统**”。
	1. 找到“**启用 Win32 长路径**”策略并将其设置为“**已启用**”。
	`ps:我是用的上面的方法解决了问题`

- 对于 Windows 10 Pro 和 Enterprise，还可以通过**注册表**来启用。需要添加或修改以下注册表键值：

 	- **路径**：`HKLM\SYSTEM\CurrentControlSet\Control\FileSystem`
**名称**：`LongPathsEnabled`
**类型**：`REG_DWORD`
**值**：`1`

	修改注册表前请注意备份注册表，错误的操作可能会导致系统不稳定。

尽量减少文件夹和文件名的长度。如果是虚拟环境的路径问题，考虑创建一个路径更短的虚拟环境。
如果错误是由于安装包中静态文件的路径太长而导致的，您可以手动将包的内容复制到项目中并尝试修改路径。

## 总结
不使用**Windows**可以省下一堆奇奇怪怪的专属**bug**
