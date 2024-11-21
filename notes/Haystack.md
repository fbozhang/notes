# Haystack介绍

 - Haystack 是Django中对接搜索引擎的框架，搭建了用户和搜索引擎之间的沟通桥梁。

- 我们在Django中可以通过使用 Haystack 来调用 Elasticsearch 搜索引擎。
- Haystack 可以在不修改代码的情况下使用不同的搜索后端（比如 Elasticsearch 、 Whoosh 、Solr 等等）
# 安装配置 Haystack
## 1、安装


```python
pip install django-haystack
pip install elasticsearch==5.20	
```
elasticsearch和下載的elasticsearch.zip	的版本必須要聯係起來


## 2、配置
**以下以django爲例：**
- 添加应用和路由

```python
# setting.py
INSTALLED_APPS = [
    'haystack',
]
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/121591a45fee7dc6c61dd434b9f2166f.png)

- 添加配置


```python
# Haystack
HAYSTACK_CONNECTIONS = {
    'default': {
    'ENGINE': 'haystack.backends.elasticsearch_backend.ElasticsearchSearchEngine',# 要一定和es版本對應
    'URL': 'http://ip地址:9200/', # Elasticsearch服务器ip地址，端口号固定为9200
    'INDEX_NAME': '数据库名称', # Elasticsearch建立的索引库的名称
    },
} 
# 当添加、修改、删除数据时，自动生成索引
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
```


- 添加索引类


0. 我们需要在 模型所对应的 子应用中 创建 search_indexes.py 文件。以方便haystack来检索数据
1. 索引类必须继承自  indexes.SearchIndex, indexes.Indexable
2. 必须定义一个字段 document=True
    字段名 起什么都可以。 text只是惯例（大家习惯都这么做） 。
    所有的索引的 这个字段 都一致就行
3. use_template=True
  表示后续通过模板来指明
    允许我们来单独设置一个文件，来指定哪些字段进行检索
    
    这个单独的文件创建在哪里呢？？？
    模板文件夹下/search/indexes/子应用名目录/模型类名小写_text.txt
    
    
 数据         <----------Haystack--------->             elasticsearch 
 
 运作： 我们应该让 haystack 将数据获取到 给es 来生成索引
 
    在終端下  python manage.py rebuild_index
   

```python
class SKUIndex(indexes.SearchIndex, indexes.Indexable):
	# 接收索引字段：使用文档定义索引字段，并且使用模板语法渲染
    text = indexes.CharField(document=True, use_template=True)

    def get_model(self):
        """返回建立索引的模型类"""
        return SKU

    def index_queryset(self, using=None):
        """返回要建立索引的数据查询集"""
        return self.get_model().objects.all()
        # return SKU.objects.all()

```



- 创建 text 字段索引值 模板文件
 

```python
# 位置  templates/search/indexes/appname/sku_text.txt
# 添加要索引的模型类字段
# 这个注释要删掉
{{ object.id }}
{{ object.name }}
{{ object.detail}}

```

- 手动生成初始索引


```python
python manage.py rebuild_index
```




