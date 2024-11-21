`celery`注册异步函数是模块级别的，也就是同个模块不能有同名函数，比如搞个骚操作，将`celery`任务写在类中如下(注意这个静态方法是个特殊的装饰器，他实际是个描述器，他必须写在最上面)
	
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a9dbfb78e0c1877f73d01d945ce29433.png)

实际注册的任务是`apps.business.tasks.asd`而不是`apps.business.tasks.A.asd`或者`apps.business.tasks.B.asd`，截图如下
	
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d2c3dcceeeaea0aff998359408e5eaa.png)

	
那么当我们有两个同名函数生效哪个？可以发现被`task`装饰后他们的`id`是一样的也就是实际是同一个
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/17e43f209033c4bb9445fb7ad347a6ce.png)

	
同步运行结果如下，可以发现这有点抽象，当先调用`A.asd()`那么无论之后调用`A.asd()`或`B.asd()`都是打印`asd`，当先调用`B.asd()`那么无论之后调用`A.asd()`或`B.asd()`都是打印`qwe`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b55ac80fee1b6604d7d360a1974b9aa.png)

有点抽象的是当我们异步去调用他是调用第二个函数的打印，也就是后面的覆盖前面的
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/703f6131a59abe08200f37579a64bafa.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad40607f2f07b0233a13b1e7d3af900d.png)
猜测可能是这样实现的导致同步调用时会发生谁先调用就变成谁(异步注册任务是另一套逻辑，这里给出的是可能造成同步调用时那种效果的示例demo)
```python
	lis = {}
	
	def task(func):
	    def w():
	        if func.__name__ in lis:
	            return lis[func.__name__]()
	        else:
	            lis[func.__name__] = func
	            func()
	    return w
	
	class B:
	    @staticmethod
	    @task
	    def asd():
	        print("asd")
	class A:
	    @staticmethod
	    @task
	    def asd():
	        print("qwe")
	
	
	B.asd()
	A.asd()
```

实际源码如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/55d0fb78c110476aa326acb73bfe3368.png)
