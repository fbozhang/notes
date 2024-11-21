

# 前言

爬取到的数据不能直观的知道得到了什么数据，能用什么方法将数据分析并进行可视化？

---

`提示：以下是本篇文章正文内容，下面案例可供参考`

# 一、Flask是什么？
Flask是一个使用 Python 编写的轻量级 Web 应用框架。其 WSGI 工具箱采用 Werkzeug ，模板引擎则使用 Jinja2 。Flask使用 BSD 授权。
Flask也被称为 “microframework” ，因为它使用简单的核心，用 extension 增加其他功能。Flask没有默认使用的数据库、窗体验证工具。
Flask 本身相当于一个 内核, 主要实现了路由分发和模板渲染功能, 分别集成自 Werkzeug 和 Jinja2模块包, 这两个也是Flask框架的核心。
虽然核心精简, 但flask提供了 非常好的扩展机制, 开发中的各类需求基本都有对应的官方/第三方扩展可以实现, 甚至连自己动手实现也很简单。

## - **常用扩展包**

Flask-SQLalchemy：ORM操作数据库；
Flask-RESTful：开发REST API的工具；
Flask-Migrate：管理迁移数据库；
Flask-Caching：缓存；
Flask-WTF：表单；
Flask-Mail：邮件；
Flask-Login：认证用户状态；
Flask-OpenID：认证；
Flask-Admin：简单而可扩展的管理接口的框架
Flask-Bable：提供国际化和本地化支持，翻译；
Flask-Bootstrap：集成前端Twitter Bootstrap框架；
Flask-Moment：本地化日期和时间；

## - **基本模式**
 Flask的基本模式为在程序里将一个视图函数分配给一个URL，每当用户访问这个URL时，系统就会执行给该URL分配好的视图函数，获取函数的返回值并将其显示到浏览器上，其工作过程见图：
 ![来自百度百科](https://i-blog.csdnimg.cn/blog_migrate/6014adc7f9573abf967e6c71e8e81d7c.png)
IT运维的基本点为安全、稳定、高效，运维自动化的目的就是为了提高运维效率，Flask[9]开发快捷的特点正好符合运维的高效性需求。在项目迭代开发的过程中，所需要实现的运维功能以及扩展会逐渐增多，针对这一特点更是需要使用易扩展的Flask框架。


# 二、Flask基础使用
## 1.引入库
代码如下：

```c
from flask import Flask, render_template
```

## 2.路由解析
**路由路径不能重复，用户通过唯一路径访问特定函数**
### 新建项目默认代码如下：
```c
@app.route('/')
def hello_world():
    return 'hello!'
```
将路由路径传给@app.route()方法，return返回的是在该路由网页上显示的内容
flask默认http://127.0.0.1:5000/，该处访问路由为'/'的页面在网页上显示hello

**结果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4a37ae1f3a7867fe7441ba39040f1a7.png)
### 修改Debug模式
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7e9aac20789f4be9a022481ff759c3ab.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/620ee7006d2e8c29b8b41801a3b7b340.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1615b6ed050448da17f6e3967b311b51.png)
打开Debug模式在修改代码后在网页刷新就能显示而不需要重新编译

### 通过访问路径，获取用户的参数

```python
# 获取用户的字符串参数
@app.route('/user/<name>')
def welcome(name):
    return "你好,{}".format(name)
```
**结果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b4691e74885b242c634727063ce41cef.png)

```python
# 获取用户的整型参数      此外还有float类型
@app.route('/user/<int:id>')
def welcome2(id):
    return "你好,{}号会员".format(int(id))
```
**结果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05dfcb5a91e784475a5c6b0510f89eb0.png)
### 返回给用户渲染后的网页文件
写一个简单测试index.html

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Flask_test</title>
</head>
<body>
	hello,world
</body>
</html>
```
通过路由访问index.html
```python
@app.route("/")
def index2():
    return render_template("index.html")
```

**结果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fde7995233c4716fa1f5b93633622152.png)

### 向页面传参

```python
@app.route("/")
def index2():
    time = datetime.date.today()  # 普通变量
    name = ["张三", "老王", "李四"]  # 列表类型
    task = {"任务": "打扫卫生", "时间": "3小时"}  # 字典类型
    return render_template("index.html", var=time, list=name, task=task)
```
将获取到的数据传入index.html中
**index.html内容：**

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Flask_test</title>
</head>
<body>
    今天是{{ var }},欢迎光临<br/>
    今天值班的有：<br/>
    {% for data in list %}      <!--用大括号和百分号括起来的控制结构，还有if-->
        <li>{{ data }}</li>
    {% endfor %}

    任务:<br/>        <!--了解一下如何在页面打印表格，以及如何迭代-->
        <table border="1">
            {% for key,value in task.items() %}
            <tr>
                <td>{{ key }}</td>
                <td>{{ value }}</td>
            </tr>
            {% endfor %}
        </table>

</body>
</html>
```
循环需要加上 {% endfor %}终止，判断需要 {% endif %}终止



**结果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9fb72176cb903c78a80511bab8bb2184.png)

### 表单

```python
# 表单提交
@app.route('/test/register')
def register():
    return render_template("test/register.html")


# 接受表单提交的路由，需要指定methods为post
@app.route('/result', methods=['POST', 'GET'])
def result():
    if request.method == 'POST':
        result = request.form
        return render_template("test/result.html", result=result)
```

**register.html：**


```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Flask_test</title>
</head>
<body>

<form action="{{ url_for('result') }}" method="post">
    <p>姓名：<input type="text" name = "姓名"></p>
    <p>年龄：<input type="text" name = "年龄"></p>
    <p>性别：<input type="text" name = "性别"></p>
    <p>地址：<input type="text" name = "地址"></p>
    <p><input type="submit" value="提交"></p>
</form>
</body>
</html>
```

**result.html：**


```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Flask_test</title>
</head>
<body>
        <table border="1">
            {% for key,value in result.items() %}
            <tr>
                <th>{{ key }}</th>
                <td>{{ value }}</td>
            </tr>
            {% endfor %}
        </table>
</body>
</html>
```

**结果**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a06629d177866670499e6b8012a9749f.png)
**输入各项参数点击提交**


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e8446ef28c279a4fbeda20c694ad2f0.png)






---

# 总结

以上就是今天要讲的内容，本文仅仅简单介绍了flask的简单使用。
