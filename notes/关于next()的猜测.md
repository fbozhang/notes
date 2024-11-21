`python`的`next`函数是用来获取`iterator`对象的下一个值，当获取不到就会报错`StopIteration`，我们通常可以通过捕获整个异常得知这个迭代完了没，当我们给他设置一个默认值那么迭代完则会使用默认值，示例代码如下


```python
In [1]: a = (i for i in [1,2])

In [2]: next(a)
Out[2]: 1

In [3]: next(a)
Out[3]: 2

In [4]: next(a)
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
Cell In[4], line 1
----> 1 next(a)

StopIteration:

In [5]: next(a, None)

In [6]: print(next(a, None))
None

In [8]: next(a, 1)
Out[8]: 1
```


但是当我们使用`pycharm`点进去查看他的源码可以获得一段伪代码如下

```python
def next(iterator, default=None): # real signature unknown; restored from __doc__
    """
    next(iterator[, default])
    
    Return the next item from the iterator. If default is given and the iterator
    is exhausted, it is returned instead of raising StopIteration.
    """
    pass
```
从注释中可以得知，如果没有传递默认值则会抛出异常，如果传递默认值则会返回默认值，但是从这个函数签名可以得知`default`是有默认值的，他的默认值是`None`，那么他是怎么判断这个`default`参数的值是传递进来的还是函数签名的默认值？
> 其实他签名旁边的注释也说了真正的签名未知，所以其实这压根就是个假签名

我们可以通过`cpython`的代码进行猜测

[官网链接](https://github.com/python/cpython/blob/b9e10d1a0fc4d8428d4b36eb127570a832c26b6f/Python/clinic/bltinmodule.c.h#L727)

相关源码如下
> 所以上面那个签名大概率就是根据下面这个doc生成出来的，有点离谱

```python
PyDoc_STRVAR(builtin_anext__doc__,
"anext($module, aiterator, default=<unrepresentable>, /)\n"
"--\n"
"\n"
"Return the next item from the async iterator.\n"
"\n"
"If default is given and the async iterator is exhausted,\n"
"it is returned instead of raising StopAsyncIteration.");

#define BUILTIN_ANEXT_METHODDEF    \
    {"anext", _PyCFunction_CAST(builtin_anext), METH_FASTCALL, builtin_anext__doc__},

static PyObject *
builtin_anext_impl(PyObject *module, PyObject *aiterator,
                   PyObject *default_value);

static PyObject *
builtin_anext(PyObject *module, PyObject *const *args, Py_ssize_t nargs)
{
    PyObject *return_value = NULL;
    PyObject *aiterator;
    PyObject *default_value = NULL;

    if (!_PyArg_CheckPositional("anext", nargs, 1, 2)) {
        goto exit;
    }
    aiterator = args[0];
    if (nargs < 2) {
        goto skip_optional;
    }
    default_value = args[1];
skip_optional:
    return_value = builtin_anext_impl(module, aiterator, default_value);

exit:
    return return_value;
}
```
从上面源码可以得知`default_value = args[1];` 猜测没传递就设置默认值是`NULL`吧`PyObject *default_value = NULL;`
然后就走`builtin_anext_impl`源码如下


```python
/*[clinic input]
anext as builtin_anext

    aiterator: object
    default: object = NULL
    /

Return the next item from the async iterator.

If default is given and the async iterator is exhausted,
it is returned instead of raising StopAsyncIteration.
[clinic start generated code]*/

static PyObject *
builtin_anext_impl(PyObject *module, PyObject *aiterator,
                   PyObject *default_value)
/*[clinic end generated code: output=f02c060c163a81fa input=2900e4a370d39550]*/
{
    PyTypeObject *t;
    PyObject *awaitable;

    t = Py_TYPE(aiterator);
    if (t->tp_as_async == NULL || t->tp_as_async->am_anext == NULL) {
        PyErr_Format(PyExc_TypeError,
            "'%.200s' object is not an async iterator",
            t->tp_name);
        return NULL;
    }

    awaitable = (*t->tp_as_async->am_anext)(aiterator);
    if (default_value == NULL) {
        return awaitable;
    }

    PyObject* new_awaitable = PyAnextAwaitable_New(
            awaitable, default_value);
    Py_DECREF(awaitable);
    return new_awaitable;
}
```


这里有个判断`default_value == NULL`，猜测可能这里的`NULL`和`python`的`None`不是一个东西，所以从这判断出了有没有传递`python`的对象吧，总之那个`python`伪代码的签名假的离谱
