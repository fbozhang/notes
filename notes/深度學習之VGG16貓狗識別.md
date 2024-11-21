

# 前言

在[上一篇文章](https://blog.csdn.net/m0_69082030/article/details/129645624?spm=1001.2014.3001.5502)中用**捲積神經網絡(CNN)** 解决了一个貓狗識別問題，这次我们依然解决了貓狗識別問題，不過這次用在巨人的肩膀上建立模型，采用**VGG16**來做**遷移學習**


# 回顧CNN
[深度學習之捲積神經網絡(CNN)之貓狗識別](https://blog.csdn.net/m0_69082030/article/details/129645624?spm=1001.2014.3001.5502)


# VGG16

## 簡介

**VGG**是Visual Geometry Group Network的缩写，视觉几何群网络；**16**是**VGG**结构中有13个**卷积层**和3个**全链接层**。


**VGG16**是由牛津大学的K. Simonyan和A. Zisserman在“用于大规模图像识别的非常深卷积网络”的论文中提出的卷积神经网络模型。 该模型在**ImageNet**中实现了92.7％的前5个测试精度，这是属于1000个类的超过1400万张图像的数据集。它是**ILSVRC-2014**提交的着名模型之一。它通过一个接一个地用多个3×3内核大小的过滤器替换大型内核大小的过滤器（分别在第一个和第二个卷积层中为11和5）来改进**AlexNet**。**VGG16**训练了几周，并使用**NVIDIA Titan Black GPU**。==VGGNet突出的贡献是证明了很小的卷积，通过增加网络深度可以有效提高性能==。VGG很好的继承了Alexnet的衣钵同时拥有着鲜明的特点。即网络层次较深。



## 結構

### 模塊劃分
**vgg16**网络结构可以划分为6个模块层次加1个输入模块，分别如下

   模块| 操作
-------- | -----
输入模块| 224*224*3(一定是這個維度)
第一个模块| conv3-64<br>conv3-64<br>maxpool
  第二个模块| conv3-128<br> conv3-128<br>maxpool
   第三个模块| conv3-256<br>conv3-256<br>conv3-256<br>maxpool
  第四个模块| conv3-512<br>conv3-512<br>conv3-512<br>maxpool
第五个模块| conv3-512<br>conv3-512<br>conv3-512<br>maxpool
 第六个模块（全连接层和输出层） |     FC-4096 （实际上前面需要加一个Flatten层）<br> FC-4096<br>FC-1000 (负责分类)<br>softmax(输出层函数)

### 結構圖
- **conv3-64** ：是指第三层**卷积**后维度变成64，**conv3-128**指的是第三层**卷积**后维度变成128；
- **input**（224x224 RGB image） ：指的是输入图片大小为**224*244**的彩色图像，通道为3，即224*224*3；
- **maxpool** ：是指**最大池化**，在**vgg16**中，**pooling**采用的是2*2的最大池化方法
- **FC-4096** :指的是全连接层中有4096个节点，**FC-1000**为该层全连接层有1000个节点；
- **padding**：指的是对矩阵**在外边填充**n圈，**padding**=1即填充1圈，5X5大小的矩阵，填充一圈后变成7X7大小；
- **vgg16**每层卷积的滑动步长**stride**=1，**padding**=1，**卷积核**大小为333；

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e253ba2d8544666281244210aca896bb.png)

如上图**VGG16**的网络结构为，**VGG**由5层**卷积层**、3层**全连接层**、softmax**输出层**构成，层与层之间使用**max-pooling**（最大化池）分开，所有隐层的激活单元都采用**ReLU**函数。具体信息如下：

- 卷积-卷积-池化-卷积-卷积-池化-卷积-卷积-卷积-池化-卷积-卷积-卷积-池化-卷积-卷积-卷积-池化-全连接-全连接-全连接
-  通道数分别为64，128，512，512，512，4096，4096，1000。卷积层通道数翻倍，直到512时不再增加。通道数的增加，使更多的信息被提取出来。全连接的4096是经验值，当然也可以是别的数，但是不要小于最后的类别。1000表示要分类的类别数。
- 用池化层作为分界，VGG16共有6个块结构，每个块结构中的通道数相同。因为卷积层和全连接层都有- 权重系数，也被称为权重层，其中卷积层13层，全连接3层，池化层不涉及权重。所以共有13+3=16层。
- 对于VGG16卷积神经网络而言，其13层卷积层和5层池化层负责进行特征的提取，最后的3层全连接层负责完成分类任务。

### 卷积核
- VGG使用多个较小卷积核（3x3）的卷积层代替一个卷积核较大的卷积层，一方面可以减少参数，另一方面相当于进行了更多的非线性映射，可以增加网络的拟合/表达能力。
- 卷积层全部都是3*3的卷积核，用上图中conv3-xxx表示，xxx表示通道数。其步长为1，用**padding=same**填充。
- 池化层的池化核为2*2

### 卷积计算

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9dca3fcf7b715345cde5f9544d7c70e8.png)

1. 输入图像尺寸为**224x224x3**，经64个通道为3的3x3的卷积核，步长为1，**padding=same**填充，卷积两次，再经**ReLU**激活，输出的尺寸大小为**224x224x64**
1. 经**max pooling**（最大化池化），**滤波器**为2x2，步长为2，==图像尺寸减半==，池化后的尺寸变为**112x112x64**
1. 经**128**个3x3的卷积核，两次卷积，**ReLU**激活，尺寸变为**112x112x128**
1. **max pooling**池化，尺寸变为**56x56x128**
1. 经**256**个3x3的卷积核，三次卷积，**ReLU**激活，尺寸变为**56x56x256**
1. **max pooling**池化，尺寸变为**28x28x256**
1. 经**512**个3x3的卷积核，三次卷积，**ReLU**激活，尺寸变为**28x28x512**
1. **max pooling**池化，尺寸变为**14x14x512**
1. 经**512**个3x3的卷积核，三次卷积，**ReLU**激活，尺寸变为**14x14x512**
1. **max pooling**池化，尺寸变为**7x7x512**
1. 然后**Flatten()**，==将数据拉平成向量==，变成一维**7x7x512**=25088。
1. 再经过两层**1x1x4096**，一层**1x1x1000**的**全连接层**（共三层），经**ReLU**激活
1. 最后通过**softmax**输出1000个预测结果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/55b3a940d7f007b310884acad09877a6.png)

### 权重参数
1. 输入层有0个参数，所需存储容量为224x224x3=150k
1. 对于第一层卷积，由于输入图的通道数是3，网络必须要有通道数为3的的卷积核，这样的卷积核有64个，因此总共有（3x3x3）x64 = 1728个参数。
所需存储容量为224x224x64=3.2M
计算量为：输入图像224×224×3，输出224×224×64，卷积核大小3×3。
所以Times=224×224×3x3×3×64=8.7×107
1. 池化层有0个参数，所需存储容量为 图像尺寸x图像尺寸x通道数=xxx k
1. 全连接层的权重参数数目的计算方法为：前一层节点数×本层的节点数。因此，全连接层的参数分别为：
7x7x512x4096 = 1027,645,444
4096x4096 = 16,781,321
4096x1000 = 4096000

按上述步骤计算的VGG16整个网络总共所占的存储容量为24M*4bytes=96MB/image 。
所有参数为138M
VGG16具有如此之大的参数数目，可以预期它具有很高的拟合能力；

但同时缺点也很明显：
即训练时间过长，调参难度大。
需要的存储容量大，不利于部署。

### VGG模型所需要的内存容量

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b23799c614df95f6e1f8cec7e2dff69.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a484eadcc823e77c15367f54a743752.png)
# 實戰

**使用VGG16的結構提取圖像特徵，再根據特徵建立mlp模型，實現貓狗圖像識別。訓練/測試數據: dataset/data_vgg:**
1. 對數據進行分離、計算測試數據預測準確率
2. 從網站下載貓/狗圖片，對其進行預測

mlp模型一個隱藏層，10個神經元

```python
# load the data
from keras.utils.image_utils import load_img,img_to_array

img_path = 'dataset/test_set/dog/dog.56.jpg'
img = load_img(img_path,target_size=(224,224))
img = img_to_array(img)
type(img)
```

加載數據，**target_size**：設置圖片尺寸，必須是**224*224**因爲這是**VGG16**的輸入維度

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb3aee07c383002e1457924fddf9ade1.png)

```python
from keras.applications.vgg16 import VGG16,preprocess_input
import numpy as np
model_vgg = VGG16(weights='imagenet',include_top=False)
x = np.expand_dims(img,axis=0)
x = preprocess_input(x)
print(x.shape)
```
**include_top**： 去掉全連接層
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/864743c89894704b9b276e093c7ffc5e.png)
看一下維度，**224*224*3**就沒錯了,原理在上面的**卷积计算**說了


```python
# 特徵提取
features = model_vgg.predict(x)
print(features.shape)
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a163bc7942cd9bc31b5b5a7bc6eecf13.png)
**7*7*512**就沒錯了,原理在上面的**卷积计算**說了

```python
# flatten
features = features.reshape(1,25088)
print(features.shape)
```
展開一下,原理在上面的**卷积计算**說了

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/377e6c9790b12e8110410c62f9eee96e.png)

```python
# visualize the data
from matplotlib import pyplot as plt
fig = plt.figure(figsize=(5,5))
img = load_img(img_path,target_size=(224,224))
plt.imshow(img)
```
可視化看一下

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/69317e71ff68068c2c6e4af5a5158662.png)

```python
# load image and preprocess it with vgg16 structure
from keras.utils.image_utils import load_img,img_to_array
from keras.applications.vgg16 import VGG16,preprocess_input
import numpy as np

model_vgg = VGG16(weights='imagenet',include_top=False)
# define a method to load and preprocess the image
def modelProcess(img_path, model):
    img = load_img(img_path,target_size=(224,224))
    img = img_to_array(img)
    x = np.expand_dims(img,axis=0)
    x = preprocess_input(x)
    x_vgg = model.predict(x)
    x_vgg = x_vgg.reshape(1,25088)
    
    return x_vgg

def flatten(name):
    # list file names if the training datasets
    import os
    folder = f'dataset/data_vgg/{name}'
    dirs = os.listdir(folder)
    # generate path for the images
    img_path = []
    for i in dirs:
        if os.path.splitext(i)[1] == '.jpg':
            img_path.append(i)
    img_path = [folder + '//' + i for i in img_path]

    # preprocess multiple images
    features = np.zeros([len(img_path),25088])
    for i in range(len(img_path)):
        features_i = modelProcess(img_path[i],model_vgg)
        print('preprocessed:',img_path[i])
        features[i] = features_i
    
    return features

features1 = flatten('cat')
features2 = flatten('dog')
# label the results
print(features1.shape,features2.shape)
y1 = np.zeros(300)
y2 = np.ones(300)

# generate the training data
X = np.concatenate((features1,features2), axis=0)
y = np.concatenate((y1,y2), axis=0)
y = y.reshape(-1,1)
print(X.shape, y.shape)
```
加载图像并使用vgg16结构对其进行预处理

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e0ac3e949e75627b4ee39df7370d6b7e.png)

```python
# split the training and test data
from sklearn.model_selection import train_test_split
X_train,X_test,y_train,y_test = train_test_split(X,y,test_size=0.3,random_state=50)
print(X_train.shape,X_test.shape,X.shape)
```

分割測試集和訓練集

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21f7dabc2700c3b0c9dbf7bc86062f19.png)

```python
# set up the mlp model
from keras.models import Sequential
from keras.layers  import Dense
model = Sequential()
model.add(Dense(units=10,activation='relu',input_dim=25088))
model.add(Dense(units=1,activation='sigmoid'))
model.summary()
```
**input_dim** VGG16輸入必須是**25088**
看一下模型結構
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4120701c7e3a004904912a3a5bac25ac.png)

很簡單的**MLP**，就一個隱藏層裏面有10個神經元

```python
#cinfigure the model
model.compile(optimizer='adam',loss='binary_crossentropy',metrics=['accuracy'])
# train the model
model.fit(X_train,y_train,epochs=50)
```
配置一下模型然後訓練，迭代50次，二分類所以損失函數用**binary_crossentropy**，**metrics=['accuracy']**看準確率

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/116b16ca044ad79a16f2e6b8fee58714.png)
很快就好了，上個實戰：[深度學習之捲積神經網絡(CNN)之貓狗識別](https://blog.csdn.net/m0_69082030/article/details/129645624?spm=1001.2014.3001.5501)迭代3000次得十幾分鐘

```python
from sklearn.metrics import accuracy_score
y_train_predict = np.around(model.predict(X_train))
accuracy_train = accuracy_score(y_train,y_train_predict)
print(accuracy_train)
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/90639a6f72a45a3932cf9455570cdff0.png)
準確率有97%算不錯了


```python
# 測試準確率
y_test_predict = np.around(model.predict(X_test))
accuracy_test = accuracy_score(y_test,y_test_predict)
print(accuracy_test)
```
看一下測試準確率

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb91c0862b7dd877fdee0d58037663da.png)

也還🆗，沒有過擬合

```python
img_path = 'dataset/test_set/dog/dog.56.jpg'
img = load_img(img_path,target_size=(224,224))
img = img_to_array(img)
model_vgg = VGG16(weights='imagenet',include_top=False)
x = np.expand_dims(img,axis=0)
x = preprocess_input(x)
features = model_vgg.predict(x)
features = features.reshape(1,7*7*512)
result = np.around(model.predict(features))
print(result)
```

加載一張看一下，0是貓、1是狗

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5b912f78fcb883310f4c3112813e4be7.png)

預測中了

```python
training_set.class_indices 
```
可以用這個看一下類別
 ![Alt](https://i-blog.csdnimg.cn/blog_migrate/7ae5df80f8d639edce0bc95ec5ac5837.png)

```python
# make prediction on multiple images
# 多張圖片的預測
import matplotlib as mlp
font2 = {
    'family': 'SimHei',
    'weiight': 'normal',
    'size': 20
}
mlp.rcParams['font.family'] = 'SimHei'
mlp.rcParams['axes.unicode_minus'] = False
from matplotlib import pyplot as plt
from matplotlib.image import imread
from keras.utils.image_utils import load_img, img_to_array
from keras.models import load_model

fig = plt.figure(figsize=(10,10))
a = [i for i in range(1,10)]
for i in a:
    img_name = './dataset/test/{i}.jpg'.format(i=i)
    img_ori = load_img(img_name,target_size=(224,224))
    img = img_to_array(img_ori)
    x = np.expand_dims(img,axis=0)
    x = preprocess_input(x)
    x_vgg = model_vgg.predict(x)
    x_vgg = x_vgg.reshape(1,7*7*512)
    result = np.around(model.predict(x_vgg))
    img_ori = load_img(img_name,target_size=(250, 250))
    plt.subplot(3,3,i)
    plt.imshow(img_ori)
    plt.title('預測為: 狗狗' if result[0][0] == 1 else '預測為: 貓咪')
plt.show()
```
都可視化出來看一下

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a733771e0578ebed033bb62e45a8ad72.png)
全對了，而且挺快的，不愧是巨人的肩膀上


# 總結
**基於VGG16、結合mlp實現貓狗識別圖像實戰summary:**
1. 基於經典的VGG16結構，實現了圖像識別模型的快速搭建與訓練，並完成貓狗識別任務
1. 掌握了拆分已經訓練好的模型結構的方法，實現對其靈活應用
1. 更熟練的運用mlp模型，並將其與其他模型相結合，實現更複雜的任務
1. 通過VGG16+mlp的模型，實現了在小數據集情況下的模型快速訓練並獲得較高的準確率

這就是本次學習深度學習之VGG16貓狗識別的筆記
附上本次實戰的數據集和源碼：
鏈接：[https://github.com/fbozhang/python/tree/master/jupyter](https://github.com/fbozhang/python/tree/master/jupyter)


