# 背景

当你是一台**Linux**，想要给文件加权限很简单，只需要执行以下命令

```python
chmod +x filename
```
就可以给文件添加执行权限，但是如果你是**Windows**那就很麻烦了

# 解决方案


假设这里有一个名为 `file.sh` 的文件，内容如下：
```python
#!/bin/sh
echo Hello, World!
```

要让此文件在上传到 **Git** 仓库后保留执行权限，您可以：

1. 首先，将 `file.sh` 添加到本地 Git 仓库：

	```python
	git add file.sh
	```


2. 然后，使用命令 `git ls-files` 的 `-s` 选项查看文件权限：

	```python
	$ git ls-files -s
	100644 131b6b8bb46c8286541c6503f94b21a1fd25b200 0	file.sh
	```
	![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aba6341528be5d0d348d667b06f32c65.png)
	现在的权限是 `644`，没有执行权限

3. 使用命令	 `git update-index`	 的` --chmod=+x `选项为文件添加执行权限：
	

	```python
	git update-index --chmod=+x file.sh
	```
	
4. 再次查看文件权限：

	```python
	$ git ls-files -s
	100755 131b6b8bb46c8286541c6503f94b21a1fd25b200 0	file.sh
	```
	现在的权限是 `755`，拥有执行权限


5. 将 **commit** 提交到本地 **Git** 仓库：

	```python
	git commit -m "Add file.sh"
	```

6. 最后，推送到远程 **Git** 仓库：

	```python
	git push
	```


**完成！**


