---
title: Web 漏洞分析与防御之点击劫持（三）
date: 2017-10-16 09:19:49
tags: 安全
categories: 前端
---
## 一、全称
点击劫持，顾名思义，用户点击某个按钮，却触发了不是用户真正意愿的事件。

## 二、原理
用户在登陆 A 网站的系统后，被攻击者诱惑打开第三方网站，而第三方网站通过 iframe 引入了 A 网站的页面内容，用户在第三方网站中点击某个按钮（被装饰的按钮），实际上是点击了 A 网站的按钮。

## 三、演示攻击
演示图如下：

![image](http://ow97db1io.bkt.clouddn.com/web-security-5.gif)

<!-- more -->
用户登录论坛系统后，被攻击者诱骗点击第三方网站，并在第三方网站中点击某个按钮（鼠标没法录制），结果却触发了论坛系统的发言请求。

我们xian 查看“点击劫持.html”文件的源码：

``` html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>点击劫持</title>
</head>
<body style="background: url(bg.jpg) no-repeat;">
    <iframe style="opacity: 0" src="http://localhost:8080/article/articleList" width="740" height="460"></iframe>
</body>
</html>
```

bg.jpg 就是显示美女字样的图片，页面使用 iframe 元素引用论坛系统的文章界面，将 iframe 透明化。

我们通过浏览器调试工具查看该页面的元素，修改 iframe 的 opacity 为 0.5 ，结果如下：

![image](http://ow97db1io.bkt.clouddn.com/web-security-01.jpg)

从上图可知，攻击者通过图片作为页面背景，隐藏了用户操作的真实界面，从而使用户在点击第三方网站的按钮时，实际上是触发了论坛系统的发表按钮。

## 四、防御
从上文介绍的内容可知，点击劫持攻击的前提是第三方网站使用 iframe 引入目标网站的页面，我们禁止目标网站被嵌套即可。

### 4.1 Javascript 禁止内嵌

在目标网站页面添加如下代码：

``` html
<script>
    if (top.location != window.location) {
        top.location = window.location;
    }
</script>
```

当第三方页面引用目标网站后，会自动跳回到目标网站。

缺点：iframe 使用 sandbox="allow-forms" 时防御就失效。

### 4.2 X-FRAME-OPTIONS 禁止内嵌
X-Frame-Options HTTP 响应头是用来给浏览器指示允许一个页面可否在 &lt;frame&gt; 或者 &lt;object&gt; 中展现的标记。(推荐使用)

### 4.3 辅助手段
由于点击劫持只需要用户简单的点击按钮就触发事件，我们可以增加用户的操作成本，从而让用户有所警觉。如：设置验证码等。

## 五、参考资料
* <https://developer.mozilla.org/zh-CN/docs/Web/HTTP/X-Frame-Options>
