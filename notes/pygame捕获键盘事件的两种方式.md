# pygame捕获键盘事件的两种方式

方式1：在pygame中使用pygame.event.get()方法捕获键盘事件，使用这个方式捕获的键盘事件必须要是按下再弹起才算一次。
示例示例：

```python
eventList = pygame.event.get()
# 2.对事件进行判断处理(1. 点击关闭按钮  2.按下键盘上的某个键)
for event in eventList:
    # 判断event.type  是否QUIT
    if event.type == pygame.QUIT:
        print("点击了×")
    # 判断事件类型是否为按键按下，如果是:继续判断按键是哪一个按键来进行对应的处理
    if event.type == pygame.KEYDOWN:
        # 具体是哪一个按键的处理
        if event.key == pygame.K_LEFT:
            print("按下左键")
```
方式2：在pygame中可以使用pygame.key.get_pressed()来返回所有按键元组，通过判断键盘常量，可以在元组中判断出那个键被按下，如果被按下则元组中就会存在这个按键信息。通过这样的方式也可以捕获到键盘的事件，并且不需要按下再弹起的操作，一按下就会有响应，灵活性比较高。
示例代码：

```python
mykeyslist = pygame.key.get_pressed()  # 获取按键元组信息
if mykeyslist[pygame.K_RIGHT]:  # 如果按键按下，这个值为1
    print("按下了方向右键")

```
总结：
两种方式的比较：
方式1按下一次必须弹起按键再点击一次才有操作，方式2可以按住这个键则会重复操作
方式1的灵活性没有方式2的好，如果对灵活性要求高的游戏，一般建议使用方式2。
