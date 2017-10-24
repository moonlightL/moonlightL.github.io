---
title: CSS3 动画简单入门
date: 2017-10-13 09:46:40
tags: CSS3
categories: 前端
---
## 一、基本介绍

### 1.1 动画属性

``` html
transform #变形

transition #过度

animation #自定义动画
```


### 1.2 案例模板如下：

``` html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>transform</title>
    <style type="text/css">
        body {
            background-color: #eee;
            margin: 0;
            padding:0;
        }
        img {
            position: absolute;
            top:200px;
            left:200px;
        }
    </style>
</head>
<body>
    <img src="test.jpg">
    <img src="test.jpg" class="test">
</body>
</html>

```

初始效果：

![](http://ow97db1io.bkt.clouddn.com/css3-1.jpg)

两张图片重合

<!-- more -->

## 二、transform

### 2.1 移动（translate）

功能：让元素进行位移

语法：
   
``` html
translate(x,y) #水平和垂直方向位移

translateX(x) #水平方向位移

translateY(y) #垂直方向位移
    
```

单位：px，em，rem，百分比

方向：水平向右为正，垂直向下为正

测试：

``` html
.test {
    transform: translate(100px,50px); #水平向右位移100px，垂直向下位移50px
}
```
![](http://ow97db1io.bkt.clouddn.com/css3-2.jpg)


    
### 2.2 旋转（rotate）

功能：让元素绕中心点旋转

语法：rotate(num)
    
单位：deg
    
方向：顺时为正，逆时为负。旋转中心点位置有transform-origin属性决定，默认以元素中心为旋转点。

测试：

```  html
.test {
    transform: rotate(45deg); #顺时旋转45度
}
```
![](http://ow97db1io.bkt.clouddn.com/css3-3.jpg)



### 2.3 缩放（scale）

功能：让元素以中心点进行缩放

语法：
    
``` html
scale(x,y) #水平和垂直方向缩放

scaleX(x) #水平方向缩放

scaleY(y) #垂直方向缩放
```


单位：无

测试：

``` css
.test {
    transform: scale(.5); #水平和垂直方向缩小0.5倍
}
```
![](http://ow97db1io.bkt.clouddn.com/css3-4.jpg)

    

### 2.4 扭曲（skew）

功能：让元素绕中心点进行扭曲

语法：
    
``` html
skew(x,y) #水平和垂直方向扭曲

skewX(x) #水平方向扭曲

skewY(y) #垂直方向扭曲
```


 单位：deg

方向：顺时为正，逆时为负

测试：

``` css
.test {
    transform: skew(20deg,20deg); #水平方向扭曲20度，垂直方向扭曲20度
}
```
![](http://ow97db1io.bkt.clouddn.com/css3-5.jpg)


### 2.5 矩阵变形：matrix()

上述 4 种动画方式功能的底层都是通过 matrix() 实现的
    
``` html
translate(tx,ty)==matrix(1,0,0,1,tx,ty)

scale(sx,sy)==matrix(sx,0,0,sy,0,0)

rotate(θ)==matrix(cosθ,sinθ,-sinθ,cosθ,0,0)

skew(θx，θy)==matrix(1,tan(θy),tan(θx),1,0,0)
```


掌握上述4种即可。

## 三、transition

功能：在一定的时间内平滑地把属性值从起始状态改到结束状态

语法：transition 属性 耗时 速度曲线 延时

默认值：all 0s ease 0s;

测试：

``` html
test {
   transition: all 3s ease
}
img:hover {
     transform: scale(1.5); #鼠标触碰图片时，3秒时间内平滑地将图片放大到1.5倍大小
}
```
![](http://ow97db1io.bkt.clouddn.com/css3-1.gif)

## 四、animation

功能：让元素实现动画效果，效果比前两个属性更强大。

语法:animation 动画名称 持续时间 速度曲线 延时 循环次数 方向 动画结束状态 动画状态

测试：

``` html
.test {
   animation: jump 1s; #1秒时间内跳动
}

@keyframes jump {
    0% {
        transform: translateY(0px);
    }

    100% {
        transform: translateY(100px);
    }
}
```
![](http://ow97db1io.bkt.clouddn.com/css3-2.gif)

## 五、参考资料
* <http://css.doyoe.com/>
