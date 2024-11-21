
## 前言

[Postman Pre-request Script](https://blog.csdn.net/m0_69082030/article/details/141070240?spm=1001.2014.3001.5501)

## 示例






![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ff07d60f792748f3a254d45ab8daf250.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1e58ee30bed94fd0b1528c69299c5795.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2cdad0d7bd274bec851151a4cb4139ca.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0a11e07b0bad4dc3ade25dd7ea6b5f5b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/89eb299c7d114d01b74c75867da1b167.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2eda65d774494aa7b5a231a3c7ddec96.png)





## 示例脚本

### 参数配置位置
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6dc3bf47852b4d34b1bf493e2fc4944c.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/272e3daf0b6442d8aee44d4194558491.png)



### 必要参数
`django`项目仅需要设置`domain`即可，比如`www.baidu.com`,`baidu.com`尽量域名精确避免修改到其他域的参数
必须把这个`domain`添加到 `cookies->Manage cookies ->Domains Allowlist` 中，否则cookie的注入失败

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e9674625025a4294a19f45523903940b.png)


### 代码

```javascript
// 必须把这个domain添加到 cookies->Manage cookies ->Domains Allowlist 中，否则cookie的注入失败
var domain = pm.environment.get("domain") || pm.globals.get("domain");
// console.log("domain:",domain)

// 获取全局变量的csrf, csrf为任意64位字符串若不设置则使用默认
var csrf = pm.globals.get("csrf") || "0123456789012345678901234567890123456789012345678901234567891234";

// 若没有设置环境变量则取全局变量
var csrf_cookie_name = pm.environment.get("csrf_cookie_name") || pm.globals.get("csrf_cookie_name")
// 默认为django设置的名字为csrftoken
var default_csrf_cookie_name = "csrftoken"

if (csrf_cookie_name) {
    // 如果有csrf_cookie_name
    csrf_cookie_name = csrf_cookie_name;
} else {
    // 如果没有
    csrf_cookie_name = default_csrf_cookie_name;
}
// console.log("csrf_cookie_name:", csrf_cookie_name);

// 删除原来的csrf头
pm.request.removeHeader("X-CSRFToken");
// 添加header
pm.request.addHeader(`X-CSRFToken:${csrf}`);


//创建一个cookie容器
const cookieJar =pm.cookies.jar();

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

function deleteCookie(domain, cookieName) {
    return new Promise((resolve, reject) => {
        //删除cookie中的cookieName,参数: url,cookieName,回调函数function(异常)
        cookieJar.unset(domain, cookieName, function (error) {
            // console.log(`删除 ${cookieName} cookies`);
            // console.log("error", error);
            if (error) {
                reject(error);
            } else {
                resolve();
            }
        });
    });
}

async function checkAndSetCookie(domain, cookieName, csrf) {
    // 睡眠10毫秒
    await sleep(10);

    // 查看 Cookie 是否存在
    cookieJar.get(domain, cookieName, function (error, cookie) {
        console.log("查看");
        console.log("cookie", cookie);
        // console.log("error", error);

        if (!cookie) {
            // 如果没有找到 Cookie，设置它
            console.log("未找到 Cookie，开始设置");
            cookieJar.set(domain, cookieName, csrf, function (error, cookie) {
                console.log(`设置 ${cookieName} Cookie 为 ${cookie}`);
                // console.log("cookie", cookie);
                // console.log("error", error);

                // 设置完成后再次调用检查函数
                checkAndSetCookie(domain, cookieName, csrf);
            });
        } else {
            // 如果找到 Cookie，输出成功信息
            console.log("Cookie 已存在，结束");
        }
    });
}

(async function() {
    try {
        // 首先删除 Cookie
        await deleteCookie(domain, csrf_cookie_name);
        // 然后开始检查和设置 Cookie
        checkAndSetCookie(domain, csrf_cookie_name, csrf);
    } catch (error) {
        console.error("删除 Cookie 失败:", error);
    }
})();
```
