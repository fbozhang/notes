在爬取网页的过程中，经常会遇到一些弹窗的情况，有alert、confirm、prompt等三种，区别如下：

1. **alert() 弹出个提示框 （确定）**
警告消息框 alert 方法有一个参数，即希望对用户显示的文本字符串。该字符串不是 HTML 格式。该消息框提供了一个“确定”按钮让用户关闭该消息框，并且该消息框是模式对话框，也就是说，用户必须先关闭该消息框然后才能继续进行操作。
2. **confirm() 弹出个确认框 （确定，取消）**
确认消息框 使用确认消息框可向用户问一个“是-或-否”问题，并且用户可以选择单击“确定”按钮或者单击“取消”按钮。confirm 方法的返回值为 true 或 false。该消息框也是模式对话框：用户必须在响应该对话框（单击一个按钮）将其关闭后，才能进行下一步操作。
2. **prompt() 弹出个输入框（确定，取消）**
提示消息框 提供了一个文本字段，用户可以在此字段输入一个答案来响应您的提示。该消息框有一个“确定”按钮和一个“取消”按钮。如果您提供了一个辅助字符串参数，则提示消息框将在文本字段显示该辅助字符串作为默认响应。否则，默认文本为 "undefined"。

**这三种弹窗的共同点是，弹出之后你是获取不到任何网页内容的，也就是无法通过常规的辦法**

```python
driver.find_element(by=By.XPATH,value='')
```
**这种形式来获取元素。F12是没有任何内容，也无法点选的**



**selenium**另外有一套方法来把driver转换到弹窗上：

```python
driver.switch_to.alert.accept()
```
其中**driver**就是你设置好的浏览器句柄，**switch_to.alert**代表你当前的弹窗类型，**alert**就对应**alert**，**accept**的意思就是点确定，另外还有**dismiss**等用法，网上很多了，不详细说。
   
<br>

用这个语句，就可以把弹窗点掉，之后正常操作，但是我重点想说的是以下内容：
**在实际网页中，往往弹窗会有一定延时，这时候你用这个语句就会报一个no such alert的错误，意味着获取不到弹窗，此时要用如下的办法解决：**

```python
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from selenium.common.exceptions import TimeoutException

wait = WebDriverWait(driver, 10)
wait.until(EC.alert_is_present())
driver.switch_to.alert.accept()#注意之前的两步
```
引用**Webdriver**类，里面的参数第一个是句柄，第二个则是超时等待时间，这里是10秒钟。

**Webdriver**这种方法叫做**显示等待**，用一个默认频率不停的刷新（默认是0.5s），检测当前页面元素是否存在，如果超过10秒则抛出TimeOut。

很显然，这种方法比一般的sleep效率要高。
**wait.until(EC.alert_is_present())** 就是判断弹窗是否存在，如果存在，那么就不会抛出异常，继续走下一步也就是获取到弹窗点击确定。

之后就可以进行正常的操作了。


