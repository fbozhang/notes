
## 背景
在线上服务中使用时间进行数据库操作时发现异常，而在本地环境无法成功复现此问题，导致难以进行故障排查。

## 核心问题

**view.py**
```
class XxxViewSet(viewsets.ModelViewSet):

    queryset = Xxx.objects.with_status().order_by("status", "-start_time")
```

**managers.py**
```
def with_status(self):
    """添加status排序字段"""
    cur_time = datetime.now(pytz.utc)
    queryset = self.annotate(
        status=Case(
            When(end_time__lt=cur_time, then=Value(2)),
            When(start_time__gt=cur_time, then=Value(1)),
            default=Value(0),
            output_field=IntegerField(),
        )
    )
    return queryset
```


## 问题的关键点

由于`viewsets.ModelViewSet`中使用了`queryset`属性，程序不会在每次请求时都重新计算当前时间，而是使用缓存的查询集。这意味着时间过滤条件并不会随着时间实时更新，而是固定在视图集类被加载时的时间。


## 分析本地无法复现原因
本地开发经常使用热部署，即服务频繁重启，这导致定义的`cur_time`变量不断被重置。而线上环境服务不会如此频繁重启，因而很难注意到这个问题。建议在本地创建一条时间精确到秒的记录，以此来模拟线上环境并复现问题。


## 问题的根源
在**Django**中与数据库交互时，应使用**Django**数据库函数中的当前时间而非**Python**标准库中的时间：

使用Django的`Now`函数：
```python
from django.db.models.functions import Now
```

而不是使用python 的时间

```python
from datetime import datetime
```

否则会把时间设置为定值

从sql的角度理解就是
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a59304292e5596b371d4217d1b2078d1.png)



## 最终方案

修改使用数据库时间
**managers.py**
```
from django.db.models.functions import Now
def with_status(self):
    """添加status排序字段"""
    cur_time = Now()
    queryset = self.annotate(
        status=Case(
            When(end_time__lt=cur_time, then=Value(2)),
            When(start_time__gt=cur_time, then=Value(1)),
            default=Value(0),
            output_field=IntegerField(),
        )
    )
    return queryset
```

## 结论与建议
使用Django `django.db.models.functions.Now()`函数替代Python `datetime.datetime.now()`在Django应用程序的时间处理中具有几个优点与潜在的缺点：
### 优点：
1.在**Django**与**数据库**交互时，如果需要获取当前时间,应优先考虑使用`django.db.models.functions.Now()`，而不是`datetime.datetime.now()`，特别是当涉及到使用类属性`queryset`的情况。这样可以确保每次请求都能反映真实的当前时间，避免由于查询集缓存所导致的时间判断错误。确保时间数据的一致性与准确性。
2.   **数据库兼容性**：`Now()`函数自动适应不同数据库系统的时间函数，降低了数据库之间兼容性问题的风险。 
3.   **性能**：如果数据库后端支持，使用`Now()`可能会比从应用层传递时间戳到数据库更优化，因为时间运算直接在数据库层完成。
4.   **时区一致性**：`Now()`函数遵守Django的时区设置，自动处理时区转换，减少了手动处理时区问题的复杂度。
  ### 缺点：
 1. **依赖数据库时钟**：使用`Now()`函数意味着依赖数据库服务器的时钟。如果数据库服务器的时间配置不正确，可能会导致问题。
 2. **数据库执行时间**：由于`Now()`函数在数据库执行查询时生成时间，如果一个请求涉及多个查询，而查询之间有延迟，这可能会导致时间上的微小不一致。 
 3. **测试复杂性**：使用`Now()`可能会使单元测试更复杂，因为在测试环境中控制或模拟数据库返回的`Now()`值通常比使用固定的`datetime`值更困难。



## 使用Now()的坑
**django**存入数据库是会将时间是会处理为==UTC==时间，而使用`Now()`是使用数据库所在会话的时区的时间并不是==UTC==时间。**MySQL**提供了**UTC_TIMESTAMP()**函数，可以获取**UTC**时间，但是**django**并没有提供这个函数，大概写了一个`Func`如下可以参考。或者还是使用`timezone.now()`，但是要注意不能在静态属性使用，只有动态计算的才能使用`timezone.now()`

```python
from django.db.models import Func, DateTimeField


class UTCNow(Func):
    function = 'UTC_TIMESTAMP'
    template = '%(function)s()'
    output_field = DateTimeField()

    def as_postgresql(self, compiler, connection, **extra_context):
        return self.as_sql(compiler, connection, template="(CURRENT_TIMESTAMP AT TIME ZONE 'UTC')", **extra_context)

cur_time = UTCNow()
```

```python
select UTC_TIMESTAMP();
select NOW();
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9e9ee4a509160f67706533c2b2f444b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2665208c005280d6fe7dafa3ef172756.png)



## 总结
1. 在**Django**与**数据库**交互时，如果需要获取当前时间,应优先考虑使用`django.db.models.functions.Now()`，而不是`datetime.datetime.now()`，特别是当涉及到使用类属性`queryset`的情况。这样可以确保每次请求都能反映真实的当前时间，避免由于查询集缓存所导致的时间判断错误。确保时间数据的一致性与准确性。
2.  若使用**queryset**应该避免存在动态计算的情况,比如上述例子的**status**字段计算,`queryset = Announcement.objects.all()`程序不会在每次请求时都重新计算当前时间，而是使用缓存的查询集。


## 参考连接
[https://stackoverflow.com/questions/73140363/what-is-the-difference-between-timezone-now-and-db-models-functions-now](https://stackoverflow.com/questions/73140363/what-is-the-difference-between-timezone-now-and-db-models-functions-now)


