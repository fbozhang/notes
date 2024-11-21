## 描述
1. sort是应用在list（也就是列表）上的方法，属于列表的成员方法；而sorted是Python内置的全局方法，可以对所有可迭代对象进行排序操作

2. list的sort方法是对已存在的列表进行操作；而内建函数sorted的结果会返回一个新生成的列表，而不是在原有列表的基础上进行操作

3. sort的使用方法为list.sort()，而sorted的使用方法为sorted(list)

## sorted语法

```python
sorted(iterable=None,key=None,reverse=False)
```
**参数说明：**
**iterable**：可迭代对象
**key**：该参数的值为一个函数，此函数只有一个参数，并且返回一个值用来进行比较。这个方法是快速的，因为key指定的函数会对每个元素进行调用
## sort语法

```python
list.sort(key=None,reverse=False)
```
两个参数跟上面sorted的参数一样
在使用的时候要注意的是list.sort()没有返回值，也就是返回值为None。






## 基礎用法

```python
 x=[2,6,3,2,1,5]
 x.sort()
 print(x)
 [1, 2, 3, 4, 5]
```


```python
 y=sorted(x)
 print(y)
 [1, 2, 3, 4, 5]
```

从代码中可以看出sort()和sorted()这两个函数调用的方式就不同，sort()的调用是列表后面加小数点的方式调用，而且顺序排序无需参数，sorted()的调用方式则是将需要顺序排序的列表作为参数放进函数里面进行调用。
从调用后的结果来看，a列表调用sort()改变了a列表；sorted()函数中传入b列表，返回排好序的列表，但是b列表本身不改变。

表面上看这两个函数的效果相同，存在重复的问题。但事实上并非如此，编程里面，对原列表进行改变与否是一个很关键的操作。
如果你需要改变原列表，就说明原列表不需要保留，使用改变原列表的顺序排序函数sort()可以节省Python运行空间，因为是在原列表直接进行改变。而如果要保留原列表，就使用sorted()，这个函数会生成一个新列表，虽然占用了空间，但这是保留原列表所必须的代价。




## sort() 排序的高级用法
**sort（）可以接受两个参数sort（key,reverse）**


1.  **key参数**
key接受的是一个只有一个形参的函数
key接受的函数返回值，表示此元素的权值，sort将按照权值大小进行排序

```python
lst = ['123', '385', '674', '147']  # 依据第二位数字进行排序
lst.sort(key=lambda l: l[1])  # 利用key索引列表中每个元素的第二个位置，并依此排序
print(lst)
# result = ['123', '147', '674', '385']
```

2.  **reverse参数**
reverse接受的是一个bool类型的值 (Ture or False),Ture 為降序 False為升序，一般默认的是False


```python
x=[8,9,0,7,4,5,1,2,3,6]
x.sort(reverse=True)
print(x)
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
x.sort(reverse=False)
print(x)
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```


## 小例子
按字符串中的數字大小排序
```python
def _sort(list,b,a):
    '''
    list :待排列数组
    b:数字前一个字符
    a;数字后一个字符
    '''
    list.sort(key = lambda x:int(x.split(a)[0].split(b)[1]))
    return list
 
x = ["py1.py", "py2.py", "py100.py", "py20.py"]
y = _sort(x,'y','.')
```



按字符串中的數字大小排序
```python
def sort_key(s):
    if s:
        try:
            c = int("".join(list(filter(str.isdigit, s))))  # 提取字符串中的数字
        except:
            c = -1  # 不含数字的字符串让他最小
        return c
 
materials = ['6枝香槟玫瑰', '5枝向日葵', '搭配5枝浅绿色洋桔梗', '2枝多头白百合', '2枝浅黄色洋桔梗', '白色相思梅', '情人草']     
materials.sort(key=sort_key, reverse=True)  # 降序排序
print(materials)
# result = ['6枝香槟玫瑰', '5枝向日葵', '搭配5枝浅绿色洋桔梗', '2枝多头白百合', '2枝浅黄色洋桔梗', '白色相思梅', '情人草']
materials.sort(key=sort_key, reverse=False)  # 升序排序
print(materials)
# result = ['白色相思梅', '情人草', '2枝多头白百合', '2枝浅黄色洋桔梗', '5枝向日葵', '搭配5枝浅绿色洋桔梗', '6枝香槟玫瑰']
```

按字典中的數字大小排序
```python
dataList = [{'name': '不忘初心', 'price': '189'}, {'name': '开心久久', 'price': '299'}, {'name': '天天快乐', 'price': '259'}]

dataList = sorted(dataList, key=lambda item: item['price'], reverse=False)
print(dataList)
# result = [{'name': '不忘初心', 'price': '189'}, {'name': '开心久久', 'price': '299'}, {'name': '天天快乐', 'price': '259'}]
```
