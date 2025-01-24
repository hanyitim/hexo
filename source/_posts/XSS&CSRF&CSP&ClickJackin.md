## XSS（跨网站指令码 Cross-site scription）

#### 概述

利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令到网页，使用户加载并执行攻击装恶意制造的网页程序；攻击成功后，可能得到更高的权限、私密网页内容、会话和 cookie 等各种内容

#### 攻击方式

- 反射型：攻击者构成一个带有恶意代码的 url 链接诱导正常用户点击，服务器接收到这个 url 对应的请求读取出其中的参数然后没有做过滤就凭借到 html 页面发送给浏览器，浏览器解析。
- 存储型：攻击者将带有恶意代码的内容发送给服务器，服务器没做任何过滤将数据存储到数据库中，下次再请求这个页面的时候服务器直接从数据库取出相关内容拼接到 html 上面，浏览器解析。
- dom 型：dom 型 xss 工具是 js 获取到攻击者输入的内容并插入到 html 中。

#### 分析

方式：通过注入恶意内容进行攻击
目的：获取用户隐私内容或者执行恶意操作

#### 案例

一、通过 url 注入

```
http://test.com/static/shareLink/index.html?shareUserName=<script>alert(document.cookie)</script>
```

```html
<html>
  <head>
    <script>
      let $referer = document.querySelectorAll(".referer");
      referer.innerHtml = shareUserName;
    </script>
  </head>
  <body>
    <p><span class="referer"></span>分享给你</p>
  </body>
</html>
```

二、通过 npm 包注入
例如：《高达 800 万次下载量的 npm 包被黑客篡改了代码，你的设备或正成为挖矿机》；

event-stream，是一个用于处理 Node.js 流数据的 JavaScript npm 包， event-stream 突然被发现包含一个名为 flatmap-stream 的依赖项，而这个依赖项被植入了窃取比特币的后门，这意味着使用到该模块的开发者们，你们的设备或许早已在自己不知情的情况下变成了挖矿机。

三、通过表单注入
通过表单提交永久注入，在其他用户,在加载到该内容直接执行了恶意脚本

```html
//Form
<input userName value="<script>alert(documemt.cookie)</sctipt>" />

//Html
<p>《安全防御》---作者:<script>alert(documemt.cookie)</sctipt></p>
```

四、SQL 注入
当系统的用户登录校验是通过“SELECT \* FROM accounts WHERE username='admin' and pasword='password' ”这类显式的 sql 进行校验;

```html
//用户名
<input name="username" />

//密码
<input name="psw" type="password" />
```

当用户在以上 username 输入框输入 `"admin' and 1=1 /*"` , 系统的校验 SQL 语句是这样的

```mysql
SELECT * FROM accounts WHERE username='admin' and 1=1 /*' and password = ''
```

因为 /\*后的语句直接被当成注释忽略，用户直接登录成功了

#### 防御手段

一、特殊符号转义

```javascript
const escapeHTML = (str) => {
  return str.replace(
    /[&<>'"]/g,
    (tag) =>
      ({
        "&": "&amp;",
        "<": "&lt;",
        ">": "&gt;",
        "'": "&#39;",
        '"': "&quot;",
      }[tag] || tag)
  );
};
```

二、在引用第三方包之前，提前评估风险，尽量使用比较多人使用的 npm 包，npm 包的版本加上 lock

三、在设计应用程序时，完全使用完全参数化（Parameterized Query）来设计资料存取功能

```mysql
set @userName := xxx;
set @passowrd := xxx;

UPDATE myTable SET c1 = @c1, c2 = @c2, c3 = @c3 WHERE c4 = @c4

SELECT * FROM accounts WHERE username=@userName and password = @passowrd
```
