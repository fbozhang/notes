

# 前言
在[上一篇文章](https://blog.csdn.net/m0_69082030/article/details/129570233?spm=1001.2014.3001.5502)中用**mlp**解决了一个好壞質檢二分類问题，这次我们依然用**多層感知器mlp**来解决**經典mnist數字識別**


---
> 回顧一下前文，但是具體理論還是看前文[深度學習之多層感知器（MLP）](https://blog.csdn.net/m0_69082030/article/details/129570233?spm=1001.2014.3001.5502)


# 簡介
**多层感知器（MLP，Multilayer Perceptron）是一种前馈人工神经网络模型，其将输入的多个数据集映射到单一的输出的数据集上。**




![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/274c631eaa64b8bd5e3719b02834b463.png)

# 思考與推導

如果学过生物的话，应该都学过人的神经元结构，其中有两个很重要的部分（树突和轴突末端）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a2106542270f9dee02ad748bd1afda4a.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e3f24b1b4a916d1c7774e3624ab4dcd2.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5f54fd7ebfd9de6b0c7b4c59aa99e971.png)
当人看到图片时，神经元间就会进行一些信息传递进而判断出图片类别。所以当我们模仿人体神经元时，可以想象将神经元信号数值化

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5fd7409ce891a298879ba85a5b368eed.png)
即是说给通过输入数据，再经过相关公式，最后生成一个新的数值信号。看到这儿，是不是感觉和逻辑回归有一些相似之处，那就再回顾一下逻辑回归模型框架

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/83bee49c057186718cacfbd4c3a80275.png)


可以看到左边是我们的输入数据，中间是我们的公式，右边就是我们求得的概率了。
虽然有些相似，但人的神经元有一个不同的点就是它并不是一个单一的神经元，也可以理解成它并不是一个单一的逻辑回归结构，而是由很多个神经元组成的神经网络。那么我们能把逻辑回归组成一个网络吗？

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2272befb87b1b7e428eba4d48b03abf3.png)


这里我们可以这样理解，从输入神经元开始到第一个隐含神经元这样一个计算过程，是通过逻辑回归计算的，然后第一个隐含神经元和第二个隐含神经元间也是通过逻辑回归计算的（这里第一个隐含神经元的相关数据可以看作是新的输入），同理，第二个隐含神经元和输出神经元间也是通过逻辑回归计算的。

还是一样，来看一个简化的MLP模型结构

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88f0b86f360faebced55e6b016dffaba.png)

来回忆一下**逻辑回归**模型的一些数学公式

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3744d48fb3af00fcab6ecc2b81d4b744.png)


这里z可以是x乘以θ

那这个简化的结构的y怎么计算呢？

我们是先计算a的值然后再计算y的值（这里x0是常数项）
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/491a49bb17acc67525f8ef4737445c7f.png)
这里就和前面的z=xθ相同了

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e56a9809160fee21aa5ab539cb69af87.png)


>話不多説直接進入實戰

# 實戰

**基於mnist數據集，建立mlp模型，實現0-9數字的十分類task:**
1. 實現mnist數據載入，可視化圖形數字
1. 完成數據預處理：圖像數據維度轉換與歸一化、輸出結果格式轉換
1. 計算模型在預測數據集的準確率
1. 模型結構：兩層隱藏層，每層有392個神經元

```python
# load the mnist data
from keras.datasets import mnist
(X_train, y_train), (X_test, y_test) = mnist.load_data()
```
加載數據，**keras**為我們提供了這個經典數據集

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/70859102ecc26c3f1afca26c255b5d27.png)

```python
# visualize the data
img1 = X_train[0]
from matplotlib import pyplot as plt
fig1 = plt.figure(figsize=(3,3))
plt.imshow(img1)
plt.title(y_train[0])
plt.show()
```
隨便找張圖可視化看一下是什麽
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6cc7adff63f53d5b3ae20733a05c9d4.png)
可以看的出來是數字5

```python
#fromat the input data
feature_size = img1.shape[0] * img1.shape[1]
X_train_format = X_train.reshape(X_train.shape[0],feature_size)
X_test_format = X_test.reshape(X_test.shape[0],feature_size)

print(X_train_format.shape)
```
格式化一下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b85aa96180026a159e7ccd7fffe3b05.png)

```python
# normalize the input data
X_train_normal = X_train_format/255
X_test_normal = X_test_format/255

print(X_train_normal[0])
```
歸一化
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8d636187f52838613366fe5cd47f61d.png)


```python
# format the output data(labels)
from keras.utils import to_categorical
y_train_format = to_categorical(y_train)
y_test_format = to_categorical(y_test)

print(y_train[0]) # 5
print(y_train_format[0]) # 在下標為5 的地方為1
```
轉爲**one-hot編碼(獨熱編碼)** 
> 独热编码即 One-Hot 编码，又称一位有效编码，其方法是使用N位状态寄存器来对N个状态进行编码，每个状态都有它独立的寄存器位，并且在任意时候，其中只有一位有效。
> 這裏不詳細説明，看注釋應該也看得懂這個編碼是什麽效果
> 
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2c75255d663b1be08e5e2da4ea079dc4.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8d9586bc1ea8ed7bf51f4ef60bf71d28.png)
建一個這樣的模型

```python
# set up the model
from keras.models import Sequential
from keras.layers import Dense,Activation

mlp = Sequential()
mlp.add(Dense(units=392,activation='sigmoid',input_dim=feature_size)) # 第一層
mlp.add(Dense(units=392,activation='sigmoid')) # 第二層
mlp.add(Dense(units=10,activation='softmax')) # 輸出層
mlp.summary()
```

設置模型並**summary()**看一下模型結構，第一層是需要告訴**input_dim**的，後面幾層是根據上一層的所以就不用，激活函數用的**sigmoid**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44a367ab08a83a81c02d1bdad06770b6.png)
可以看出有兩個隱藏層，而且訓練數據量也挺大的，會挺耗時


```python
# comfigure the model
mlp.compile(loss='categorical_crossentropy',optimizer='adam')
```
配置模型

```python
mlp.fit(X_train_normal,y_train_format,epochs=10)
```
模型訓練

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b8e76350dbe0eba89fd91349d22c6f07.png)
迭代10次大概用了一分鐘


评估模型

```python
y_train_predict = np.argmax(mlp.predict(X_train_normal), 1) # np.argmax()獲取對應整數, [5 0 4 ... 5 6 8]
print(y_train_predict)
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0b3eeefbc1fdfff39203164cc8a034a2.png)

```python
from sklearn.metrics import accuracy_score
accuracy_train = accuracy_score(y_train,y_train_predict)
print(accuracy_train)
```
看一下準確率，發現還不錯

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2f56f19b46e35ac7612c96b6c8e36f10.png)

```python
y_test_predict = np.argmax(mlp.predict(X_test_normal), 1)
accuracy_test = accuracy_score(y_test,y_test_predict)
print(accuracy_test)
```
看一下測試數據準確率，也是還可以，沒有過擬合問題

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/38b8f3cfe766cae96ee5dd1078486d23.png)

```python
img2 = X_test[123]
fig1 = plt.figure(figsize=(3,3))
plt.imshow(img2)
plt.title(y_test_predict[123])
plt.show()
```
可視化看一下，標題是預測數字，圖片是本來的圖片

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8dc0e5398a6d961e830fe430ad683cc0.png)
可以看出準確預測出是6了，也可以用別的數字試試看結果都預測的還可以





# 總結
**圖像數字多分類實戰summary:**
1. 通過mlp模型，實現了基於圖像數據的數字自動識別分類
2. 完成了圖像的數字化處理與可視化
1. 對mlp模型的輸入、輸出數據格式有了更深的認識，完成了數據預處理與格式轉換
1. 建立了結構更爲複雜的mlp模型
1. mnist數據集地址：[http://yann.lecun.com/exdb/mnist/](http://yann.lecun.com/exdb/mnist/)




這就是本次學習深度學習之多層感知器(MLP)的筆記
附上本次實戰的數據集和源碼：
鏈接：[https://github.com/fbozhang/python/tree/master/jupyter](https://github.com/fbozhang/python/tree/master/jupyter)

