
## 报错信息
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b5b24a824386fa0f699b530b8bd38e9f.png)

## 解决方法
网上有两种解决方案:[pycrypto安装出错的问题](https://www.cnblogs.com/liangqihui/p/10110144.html)

我选择第二种（更为简单）：

1. 在安装**Microsoft Visual Studio**的目录下(或者目录附近)，找到 `..\VC\Tools\MSVC\14.10.25017\include\stdint.h`并且拷贝到 `..\Windows Kits\10\Include\10.0.15063.0\ucrt`
> PS: 建议用==everything==(一个软件)搜索`VC\Tools\MSVC`和`Windows Kits\10\Include`，不然他在哪个盘的哪个目录下还要找半天
2. 在 `..\Windows Kits\10\Include\10.0.15063.0\ucrt\inttypes.h`中找到

```c++
#include <stdint.h>
```

修改为

```c++
#include "stdint.h"
```

解决问题
