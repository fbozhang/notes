
## 背景
换了台服务器按照之前[centos升级python版本](https://blog.csdn.net/m0_69082030/article/details/128639404)升级**python**正常编译安装成功，但是当使用时又出现了奇怪的报错，估计是机器太老了


## 具体报错
这个报错也会导致无法`pip`安装库
```python
>>> import ssl
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/PythonDir/lib/ssl.py", line 98, in <module>
    import _ssl             # if we can't import it, let the error propagate
ModuleNotFoundError: No module named '_ssl'
```

## 检验
 通过命令：`openssl version`查看`centos`上`openssl`版本是`1.0`，版本过低，导致失败了


## 升级过程
### 步骤一：升级ssl。

1. 分别执行以下命令，下载安装包并编译安装
	```bash
	wget https://www.openssl.org/source/openssl-1.1.1d.tar.gz
	tar -zxvf openssl-1.1.1d.tar.gz
	cd openssl-1.1.1d
	./config --prefix=/usr/local/openssl
	make && make install
	```
	
	说明：`./config --prefix=/usr/local/openssl`  此命令，可以直接`./config` ，这样默认安装路径就是`/usr/local`。建议增加--`prefix=/usr/local/openssl`  ，表示安装路径是在`/usr/local/openssl`	


2. 修改链接文件

	备份原有链接
	```bash
	mv /usr/bin/openssl /usr/bin/openssl.bak
	```
	
	创建软链接
	```bash
	ln -sf /usr/local/openssl/bin/openssl /usr/bin/openssl
	```

3. 添加路径至ld.so.conf
	>  注意：`lib`路径最后不带`/`，否则报错
	
	```bash
	echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
	```
4. 设置生效

	```bash
	ldconfig -v
	```
5.  校验版本
	通过`openssl version`返回如下
	```bash
	OpenSSL 1.1.1d 10 Sep 2019
	```


### 步骤二：重新编译安装python

前面跟之前[centos升级python版本](https://blog.csdn.net/m0_69082030/article/details/128639404)基本一样

1. 执行命令清除临时文件：

	```bash
	make clean
	```

2. 进行配置：

	```bash
	 ./configure --prefix=/usr/python3  --with-openssl=/usr/local/openssl --with-openssl-rpath=auto
	```
	
	> 有2个地方注意： 
	> - `--prefix=/usr/python3`  是python安装路径。详细可看[centos升级python版本](https://blog.csdn.net/m0_69082030/article/details/128639404) 
	> - `--with-openssl=/usr/local/openssl` 是`openssl` 安装路径。我上面安装时指明了这个路径

3. 执行命令：
	
	```bash
	make
	```

	  此时需要注意是否有报错，有报错就是上一步 `./configure`配置有问题。

	```bash
	make install 
	```

4. 安装后，执行`python3` 后，通过`import  ssl`没有报错，就证明`openssl`安装成功了。
