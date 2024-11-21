# 问题背景：
用python获取图片分辨率

---

# 问题描述


导入PIL库报错，安装不了
**报错信息：**

```c
ERROR: Could not find a version that satisfies the requirement PIL (from versions: none)
ERROR: No matching distribution found for PIL '
```


---

# 原因分析：
由报错信息知道找不到这个版本，一开始想去清华园找这个版本，然后百度得知PIL较多用于2.7版本的Python中，到python3版本已经用Pillow代替PIL了，所以应该安装的是Pillow包



---

# 解决方案：

**安装Pillow包**
<img width="546" alt="image" src="https://github.com/user-attachments/assets/dcc7d429-616e-4511-b629-5d0decaab948">



**导入Image包**
```python
from PIL import Image
```

**成功安装Pillow包后PIL库没有再报错了**





# 获取图片分辨率

```python
from PIL import Image 

filename = "asd.jpg"
img = Image.open(filename)
imgSize = img.size	# 获取图片的长和宽，返回元组类型
imgheight = img.height	# 获取图片长度
imgwidth = img.width	# 获取图片宽度
format = img.format		# 图片格式
img.close()		#关闭
print("图片尺寸是：{},宽：{}，长：{}".format(imgSize, imgSize[0], imgSize[1]))
print("图片宽：{}，长：{}，格式：{}".format(imgwidth , imgheight , format)
```
# 删除目录中不符合分辨率要求的图片


```python
import os
from PIL import Image	# 导包

path = r'.\img'		#图片路径
file_list = os.listdir(path)	# 返回指定路径下的文件和文件夹列表

for file in file_list:
    imgPath= path + '\\' + file		# 目录路径和图片名拼接得到图片路径
    # print(imgPath)	# 测试图片路径
    img = Image.open(imgPath)
    imgSize = img.size
    imgheight = img.height
    imgwidth = img.width
    img.close()
    # print(imgSize)
    if imgwidth  != 1920 and imgheight  != 1080:
        os.remove(imgPath)  # 删除文件
        print("图片{}尺寸为{},已删除".format(file, imgSize))
```
