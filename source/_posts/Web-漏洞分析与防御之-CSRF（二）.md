---
title: Web 漏洞分析与防御之 CSRF（二）
date: 2017-10-11 21:45:52
tags: 安全
categories: 前端
---

## 一、全称
跨站请求伪造（Cross-site Request Forgery）

## 二、原理

在用户登陆目标网站后，后端会返回用户登陆的凭证到前端（浏览器的 cookie）。攻击者诱使用户点击某个超链接，该超链接会发送恶意请求（会携带用户的 cookie），从而冒充用户完成业务请求（发帖、盗取用户资金等）。

<!-- more -->

## 三、攻击方式
笔者以网站的发帖功能为案例对 CSRF 攻击进行简单的讲解。

### 3.1 提交 form 表单

下图为模拟攻击演示图：

![image](http://ow97db1io.bkt.clouddn.com/web-security-1.gif)

以下是演示图中，"CSRF 攻击.html" 文件的源码。

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
</head>
<body>
<script type="text/javascript">
        document.write(`
                <form id="myForm" name="myForm" target="csrf"
                        method="post" action="http://localhost:8080/article/save">
                    <input type="hidden" name="userName" value="李四" />
                    <textarea name="content">表单攻击</textarea>
                </form>
            `);

        var iframe = document.createElement("iframe");
        iframe.name = "csrf";
        iframe.style.display = "none";
        document.body.appendChild(iframe);

        setTimeout(function() {
            document.getElementById("myForm").submit();
        },1000);
</script>
</body>
</html>
```

**实际案例中，攻击者会先了解目标网站请求的 url，通过诱惑登陆用户点击某个超链接，该链接指向类似上文的 html 页面（第三方网站）。因为是用户在自己浏览器点击的超链接，因此会自动携带用户浏览器中的  cookie，这样在第三方网站就可以冒充用户发起请求。**

### 3.2 点击超链接

如果目标网站允许 get 请求，可通过超链接攻击。如下图：

![image](http://ow97db1io.bkt.clouddn.com/web-security-2.gif)

超链接源码如下：

``` html
<a href="http://localhost:8080/article/save?userName=李四&content=超链接攻击">点击获取现金大奖</a>
```

### 3.3 加载图片方式

更加粗暴的方式就是通过图片发送请求，如下图：

![image](http://ow97db1io.bkt.clouddn.com/web-security-4.gif)

图片源码如下：

``` html
<img src="http://localhost:8080/article/save?userName=李四&content=图片攻击" />
```

## 四、防御

CSRF 攻击的一个特点是绕过目标网站的前端页面（无法获知页面的信息，如：验证码，token），在第三方网站发送请求到目标网站后端。

### 4.1 禁止第三方网站携带 cookie
在网站后端，用户登陆功能的代码中，设置 cookie 的 SameSite 的值为 Strict 返回给浏览器。

缺点：目前只有 chrome 和 opera 支持该属性。

### 4.2 使用验证码

以论坛发帖为例，在发帖时，设置一个验证码，发帖后在后端进行校验。CSRF 攻击没法获取到验证码，从而目标网站得到了防御。

### 4.3 使用 token

继续以论坛发帖为例，在论坛后端跳转发帖页面的代码中随机生成一个 N 位数的数字/字符串作为 token，设置到 cookie 和任意变量名的变量中（最终放入到前端的 form 表单中）。当用户在页面发帖后，后端接收和校验 cookie 中的 token 和 页面提交的 token 是否一致。如果一致，则说明是用户合法操作。

### 4.4 判断 referer
referer 是 http 的请求头，用户发送请求后，在后端获取该请求头，判断它的值是否包含目标网站的域名。如果包含说明操作合法。

## 五、参考资料
* <http://netsecurity.51cto.com/art/200811/97281.htm>
* <https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Referer>
