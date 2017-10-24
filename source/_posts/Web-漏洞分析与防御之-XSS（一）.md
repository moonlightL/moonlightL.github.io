---
title: ' Web 漏洞分析与防御之 XSS（一）'
date: 2017-10-09 23:06:01
tags: 安全
categories: 前端
---
## 一、全称

跨站脚本攻击（Cross Site Scripting）

## 二、原理
通过在网站中的输入框写入 script 脚本或引入 script 文件，如果网站未过滤输入内容，将会解析该脚本。

如果脚本的功能是获取网站的 cookie，cookie 中又保留一些敏感信息，则后果有可能很严重。

## 三、类型

* 反射型攻击：脚本当作 url 的参数进行注入执行

* 存储型攻击：脚本被存储到 DB 后，读取时被解析执行

<!-- more -->

## 四、注入点

### 4.1 HTML 节点
页面代码：

``` html
<div>${content}</div>
```

content 的内容为 &lt;script&gt;alert(1)&lt;/script&gt;，脚本攻击后，会变成：

``` html
<div><script>alert(1)</script></div>
```

页面将会执行 alert(1)。

### 4.2 HTML 属性
页面代码：

``` html
<img src="${imgSrc}" />
```

imgSrc 的内容为 2" onerror="alert(2)，脚本攻击后，会变成：

``` html
<img src="2" onerror="alert(2)" />
```

页面将会执行 alert(2)。

### 4.3 Javascript 代码

``` html
<script>
    var mydata = "${data}";
</script>
```

data 的内容为 hello";alert(3);"，脚本攻击后，会变成：

``` html
<script>
    var mydata = "hello";alert(3);"";
</script>
```

页面将会执行 alert(3)。

### 4.4 富文本

富文本需要保留 HTML 文本，HTML 文本中就有 XSS 攻击的风险。


## 五、防御
浏览器自带一些防御能力，但只能防御 XSS 反射类型攻击，且只能防御上文描述的前二个注入点。

防御手段原理也很简单，就是将可能会执行脚本的标签或属性进行转义和过滤。

### 5.1 HTML 节点的防御
将 < 和 > 转义成 &lt； 和 &gt；。

### 5.2 HTML 属性的防御
将 " 转义成 &quto；。

### 5.3 Javascript 代码的防御
将 " 转义成 \“ 。


### 5.3 富文本的防御
使用白名单保留部分标签和属性。

需要前端第三方工具：[cheerio](https://github.com/cheeriojs/cheerio)。

案例：

``` javascript

function xssFilter(html) {
    
    var cheerio = require("cheerio");
    var $ = cheerio.load(html);
    
    // 白名单列表，key:标签，value:属性
    var whiteList = {
        "img":["src"],
        "a":["href"],
        "font":["color","size"]
    };
    
    
    // html 的遍历所有元素
    $("*").each(function(index,elem) {
        
        // 删除不在白名单的标签
        if (!whiteList[elem.name]) {
            $(elem).remove();
            return;
        }
        
        // 删除不在白名单的标签的属性
        for (var attr in elem.attribs) {
            if (whiteList[elem.name].indexOf(attr) == -1) {
                $(elem).attr(attr,null);
                return;
            }
        }
        
    });
    
    return $.html();
    
}

```

还有另一种第三方工具，名字就叫 [xss](https://github.com/leizongmin/js-xss)
