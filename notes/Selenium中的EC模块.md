

## 简介
EC，全称为expected_conditions，中文翻译为：预期条件。
EC出现原因：
进行网页的自动化测试时，有很多会频繁使用到的方法。selenium就把这些方法封装起来到一个模块中。之后调用方法，得引用这个模块，为了简化代码量，大家就约定俗成的简写这个模块。

## EC中的方法：
- **title_is:**  判断当前页面的 title 是否完全等于（==）预期字符串，返回布尔值
- **title_contains:**  判断当前页面的 title 是否包含预期字符串，返回布尔值
- **presence_of_element_located:**  判断某个元素是否被加到了 dom 树里，并不代表该元素一定可见
-  **presence_of_all_elements_located:** :判断是否至少有 1 个元素存在于 dom 树中。举个例子，如果页面上有 n 个元素的 class 都是'column-md-3'，那么只要有 1 个元素存在，这个方法就返回 True
- **visibility_of_element_located:**  判断某个元素是否可见. 可见代表元素非隐藏，并且元素的宽和高都不等于 0
- **visibility_of:**  跟上面的方法做一样的事情，只是上面的方法要传入 locator，这个方法直接传定位到的 element 就好了

- **text_to_be_present_in_element:**  判断某个元素中的 text 是否 包含 了预期的字符串
- **text_to_be_present_in_element_value:**  判断某个元素中的 value 属性是否包含 了预期的字符串
- **frame_to_be_available_and_switch_to_it:** :判断该 frame 是否可以 switch进去，如果可以的话，返回 True 并且 switch 进去，否则返回 False
- **invisibility_of_element_located:**  判断某个元素中是否不存在于dom树或不可见
- **element_to_be_clickable:**  判断某个元素中是否可见并且是 enable 的，这样的话才叫 clickable
- **staleness_of:** :等某个元素从 dom 树中移除，注意，这个方法也是返回 True或 False
- **element_to_be_selected:**  判断某个元素是否被选中了,一般用在下拉列表
- **element_selection_state_to_be:** :判断某个元素的选中状态是否符合预期
- **element_located_to_be_selected:** 特定元素是否存在于DOM树并被选中
- **element_located_selection_state_to_be:**  跟上面的方法作用一样，只是上面的方法传入定位到的 element，而这个方法传入 locator
- **alert_is_present:**  判断页面上是否存在 alert


## 例子：

```python
#如果网页标题为“百度一下，你就知道”就返回True，否则返回False
from selenium.webdriver.support import expected_conditions as EC
result = EC.title_is('百度一下，你就知道')
print(result(self.driver))
```
>注意，**locator** 必须是一个元组
```python
# 判断类名‘abc’是否存在，等到他存在就进行下一步操作。
wait = WebDriverWait(self.web, 10)
locator = (By.CLASS_NAME, 'abc')
wait.until(EC.presence_of_element_located(locator))
```

```python
# 如果有彈窗就關閉
wait = WebDriverWait(self.web, 10)
wait.until(EC.alert_is_present())
self.web.switch_to.alert.accept()
```
