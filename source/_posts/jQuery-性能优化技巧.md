---
title: jQuery 性能优化技巧
date: 2017-09-23 08:31:21
tags: [Javascript,jQuery]
categories: 前端
---
![](http://ow97db1io.bkt.clouddn.com/jQuery.jpg)

## 一、使用最新版本 jQuery 类库

## 二、合理使用选择器

``` javascript
# 推荐使用
$("#id") 

# 可以使用
$("p"),$("span") 

# 可以使用
$(".class") 

# 尽量避免
$("[attribute=value]")  

# 尽量避免
$(":hidden") 

```

<!-- more -->

## 三、使用缓存对象

场景：修改某个按钮的文本和颜色

``` javascript
# 不好的写法

$("#btn").text("重置");

$("#btn").css("color","red");

```

``` javascript
# 优化的写法

var $btn = $("#btn");

$btn.text("重置").css("color","red");
```

## 四、循环时减少对DOM的操作

场景：往 &lt;ul&gt; 中添加 &lt;li&gt; 菜单项

``` javascript
# 不好的写法

var $ul = $("#menu");

for(var i=0; i<6; i++) {
    $ul.append("<li>菜单"+i+"</li>")
}

```

``` javascript
# 优化的写法

var $ul = $("#menu");
var html = "";

for(var i=0; i<6; i++) {
    html += "<li>菜单"+i+"</li>";
}

$ul.append(html);

```

## 五、使用事件代理

场景：给 &lt;ul&gt; 里的所有 &lt;li&gt; 绑定点击变色事件

``` javascript
# 不好的写法

$("ul li").on("click",function() {
   $(this).css("color","red"); 
});
```

``` javascript
# 优化的写法

$("ul li").on("click",function(e) {
   var $obj = $(e.target);
   $obj.css("color","red"); 
});

```


## 六、将代码转成 jQuery 插件

## 七、使用 join() 拼接字符串

第四点的案例中，代码还可以进行优化

``` javascript
var $ul = $("#menu");
var arr = [];

for(var i=0; i<6; i++) {
   arr.push("<li>菜单"+i+"</li>");
}

$ul.append(arr.join("");

```


## 八、合理利用 HTML5 的 data 属性

使用 data-* 属性来嵌入自定义数据。

``` html
<div id="user" data-age="26" data-gender="男">张三</div>

```

``` javascript

    var user = $("#user");

    var age = user.data("age");
    var gender = $("#user").data("gender");
```

## 九、尽量使用原生的 JS 方法

第五点的案例中，可以如下优化

``` javascript
$("ul li").on("click",function(e) {
   var $obj = $(e.target);
   $obj.get(0).style.color = "red";
});
```

## 十、压缩 JS 代码

***如有更多优化技巧，后续补充......***
