---
title: 原生 Javascript 编写俄罗斯方块
date: 2017-10-08 10:32:42
tags: Javascript
categories: 前端
---
## 一、背景
在网上能查找到通过 Javascript 编写俄罗斯方块的资料，而且更牛的是只需要不到百行的代码就实现了，笔者非常佩服这些牛人。佩服之余，笔者也想尝试通过 Javascript 编写俄罗斯方块。在翻阅网上的资料和代码中，发现这些代码的可读性不高，因此不能让读者很好地去理解和学习代码。因此，笔者通过本文介绍自己如何通过面向对象的思想实现该游戏。

## 二、项目介绍

### 2.1 效果展示
![](http://ow97db1io.bkt.clouddn.com/tetris-1.gif)

<!-- more -->
 
### 2.2 实现思路

1) 地图：大小已经通过 css 样式确定（300px x 600px）。

2) 堆积方块：创建 200 个小方块（30px x 30px 的div），填充（通过二维数组存放）到地图中。通过 css 样式区分堆积的方块（done）和可活动的区域（none）。

3) 下落方块：方块有7个种类，都是通过4个小方块（30px x 30px 的div）构成。其横向活动区域为[0,9]，纵向活动区域为[0,19]，通过 css 样式设置颜色，如下表示：

``` javascript
[
    { x: 4, y: 0, className: "current_0" },
    { x: 3, y: 0, className: "current_0" },
    { x: 5, y: 0, className: "current_0" },
    { x: 6, y: 0, className: "current_0" }
]
```

坐标图表示如下：

![image](http://ow97db1io.bkt.clouddn.com/tetris-3.jpg)


4) 移动方块：通过设置 position: absolute ,再动态设置 top 和 left 即可。

5) 旋转方块：通过公式旋转，以上文的坐标案例演示：

``` javascript
this.current = [
    { x: 4, y: 0, className: "current_0" },
    { x: 3, y: 0, className: "current_0" },
    { x: 5, y: 0, className: "current_0" },
    { x: 6, y: 0, className: "current_0" }
]

for (var j = 1; j < this.current.length; j++) {
    var newX = this.current[0].y + this.current[0].x - this.current[j].y;
    var newY = this.current[0].y - this.current[0].x + this.current[j].x;
    this.current[j].x = newX;
    this.current[j].y = newY;
}

```

6) 消行：通过切换样式实现，具体内容下文介绍。

7) 动态效果：通过 setInterval 不断刷新页面（调用 map 对象的 _refreshMap 方法改变方块位置）。

### 2.3 设计技术点
DOM操作、面向对象、事件操作和间隔函数 setInterval

### 2.4 项目结构
![](http://ow97db1io.bkt.clouddn.com/tetris-1.jpg)

## 三、实现步骤
由于逻辑较为复杂，代码编写较长，因此只演示关键代码。

### 3.1 css 样式介绍

none 表示地图中的活动区域样式

current 开头的表示当前活动的方块样式

done 表示堆积的方块样式

游戏开始时，地图（300px x 600px）被 200 个小方块（30px x 30px 的div）填充，其 class 为 none。

当前方块在地图中下落时，设置 class 为 current 开头的样式。

当方块不能下落要堆积时，将其在地图当前区域的 div 样式由 none 改成 done。同时，将当前下落方块坐标设置为下一个方块坐标。

当方块消行时，遍历所有行，设置当前行的样式为上一行的样式，即修改坐标，让堆积的方块下落到消行的位置。


### 3.2 初始化地图
map.js 文件

``` javascript
var Map = function(square) {
    // 边界
    this.minX = this.minY = 0;
    this.maxX = 9;
    this.maxY = 19;
    // 当前移动的方块
    this.square = square;
    // 用于记录完成下落动作的方块
    this.mapDivs = [];
    // 定时器id
    this.timeId = null;
    // 暂停标记
    this.pauseFlag = false;
    // 关闭标记
    this.closeFlag = false;
    // 阴影
    this.shadow = null;
    this.shadowFlag = false;
}

// 初始化
Map.prototype.init = function(domObjs) {
    this.shadow = domObjs.shadow;

    // 绘制地图，即填充小方块
    for (var i = 0; i <= this.maxY; i++) {
        var arr = [];
        for (var s = 0; s <= this.maxX; s++) {
            var mapDiv = document.createElement("div");
            mapDiv.className = "none";
            mapDiv.style.top = (i * this.square.size) + "px";
            mapDiv.style.left = (s * this.square.size) + "px";
            domObjs.map.appendChild(mapDiv);
            arr.push(mapDiv);
        }
        this.mapDivs.push(arr);
    }

    // 当前下落方块
    for (var j = 0; j < this.square.current.length; j++) {
        var cdiv = document.createElement("div");
        cdiv.className = this.square.current[j].className;
        cdiv.style.left = this.square.current[j].x * this.square.size + "px";
        cdiv.style.top = this.square.current[j].y * this.square.size + "px";
        domObjs.map.appendChild(cdiv);
        this.square.currentDivs.push(cdiv);
    }

    if (this.shadowFlag) {
        this._showShadow();
    }

    // 下一个方块
    for (var k = 0; k < this.square.next.length; k++) {
        var ndiv = document.createElement("div");
        ndiv.className = this.square.next[k].className;
        ndiv.style.left = (this.square.next[k].x - 2) * this.square.size + "px";
        ndiv.style.top = this.square.next[k].y * this.square.size + "px";
        domObjs.next.appendChild(ndiv);
        this.square.nextDivs.push(ndiv);
    }

    var that = this;

    // 启动定时器
    this.timeId = setInterval(function() {
        if (!that.pauseFlag) {
            that.square.moveDown(that);
            that._refreshMap();
        }
    }, 300);

    // 添加键盘监听器
    this.addEventListener();

}

// 刷新地图
Map.prototype._refreshMap = function() {
    var squareDivs = this.square.currentDivs;
    for (var j = 0; j < squareDivs.length; j++) {
        squareDivs[j].className = this.square.current[j].className;
        squareDivs[j].style.top = this.square.current[j].y * this.square.size + "px";
        squareDivs[j].style.left = this.square.current[j].x * this.square.size + "px";
    }

    // 阴影
    this._showShadow();

    var nextDivs = this.square.nextDivs;
    for (var k = 0; k < nextDivs.length; k++) {
        nextDivs[k].className = this.square.next[k].className;
        nextDivs[k].style.left = (this.square.next[k].x - 2) * this.square.size + "px";
        nextDivs[k].style.top = this.square.next[k].y * this.square.size + "px";
    }
}
```


### 3.3 创建方块
square.js 文件

``` javascript
// 方块由4个小方块组成
var Square = function(info) {
    this.info = info;
    // 小方块大小
    this.size = 30;
    // 当前下落方块div
    this.currentDivs = [];
    // 下一个方块div
    this.nextDivs = [];
    // 当前方块坐标对象
    this.current = null;
    // 下一个方块坐标对象
    this.next = null;
    this._init();
}

// 初始化
Square.prototype._init = function() {
    if (this.next == null) {
        this.current = this._getSquareType();
    } else {
        this.current = this.next;
    }
    this.next = this._getSquareType();
}

// 随机获取方块，7种方块类型
Square.prototype._getSquareType = function() {
    // 坐标顺序决定旋转点
    var data = [
        [
            { x: 4, y: 0, className: "current_0" },
            { x: 3, y: 0, className: "current_0" },
            { x: 5, y: 0, className: "current_0" },
            { x: 6, y: 0, className: "current_0" }
        ],
        [
            { x: 4, y: 0, className: "current_1" },
            { x: 3, y: 0, className: "current_1" },
            { x: 4, y: 1, className: "current_1" },
            { x: 5, y: 0, className: "current_1" }
        ],
        [
            { x: 4, y: 0, className: "current_2" },
            { x: 3, y: 0, className: "current_2" },
            { x: 3, y: 1, className: "current_2" },
            { x: 5, y: 0, className: "current_2" }
        ],
        [
            { x: 4, y: 0, className: "current_3" },
            { x: 3, y: 1, className: "current_3" },
            { x: 4, y: 1, className: "current_3" },
            { x: 5, y: 0, className: "current_3" }
        ],
        [
            { x: 5, y: 0, className: "current_4" },
            { x: 4, y: 0, className: "current_4" },
            { x: 4, y: 1, className: "current_4" },
            { x: 5, y: 1, className: "current_4" }
        ],
        [
            { x: 4, y: 0, className: "current_5" },
            { x: 3, y: 0, className: "current_5" },
            { x: 5, y: 0, className: "current_5" },
            { x: 5, y: 1, className: "current_5" }
        ],
        [
            { x: 4, y: 0, className: "current_6" },
            { x: 3, y: 0, className: "current_6" },
            { x: 4, y: 1, className: "current_6" },
            { x: 5, y: 1, className: "current_6" }
        ]
    ];
    return data[Math.floor(Math.random() * data.length)];
}

```


### 3.4 移动方块
square.js 文件

``` javascript
// 移动方块
Square.prototype._move = function(moveX, moveY, map) {
    for (var i = 0; i < this.current.length; i++) {
        var newX = this.current[i].x + moveX;
        var newY = this.current[i].y + moveY;
        if (this._isOverZone(newX, newY, map)) {
            return false;
        }
    }

    for (var j = 0; j < this.current.length; j++) {
        this.current[j].x += moveX;
        this.current[j].y += moveY;
    }
    return true;
}

// 向左移动
Square.prototype.moveLeft = function(map) {
    this._move(-1, 0, map);
}

// 向右移动
Square.prototype.moveRight = function(map) {
    this._move(1, 0, map);
}

// 坠落
Square.prototype.fastDown = function(map) {
    while (this._move(0, 1, map));
}
```

### 3.5 旋转方块
square.js 文件

``` javascript
// 旋转
Square.prototype.round = function(map) {
    // 田字方块不用旋转
    if (this.current[0].className == "current_4") {
        return;
    }

    for (var i = 1; i < this.current.length; i++) {
        var newX = this.current[0].y + this.current[0].x - this.current[i].y;
        this._modify(newX, map);
    }

    for (var j = 1; j < this.current.length; j++) {
        var newX = this.current[0].y + this.current[0].x - this.current[j].y;
        var newY = this.current[0].y - this.current[0].x + this.current[j].x;
        this.current[j].x = newX;
        this.current[j].y = newY;
    }
}
```

### 3.6 消行
square.js 文件

``` javascript
// 向下移动
Square.prototype.moveDown = function(map) {
    if (this._move(0, 1, map)) {
        return false;
    }

    // 堆积
    for (var i = 0; i < this.current.length; i++) {
        map.mapDivs[this.current[i].y][this.current[i].x].className = "done";
    }

    // 消行
    for (var j = 0; j <= map.maxY; j++) {
        if (this._isCanRemoveLine(j, map)) {
            this._removeLine(j, map);
            // 加分
            this.info.plusScore(10);
        }
    }

    // 重新初始化
    this._init();
}

// 消行
Square.prototype._removeLine = function(row, map) {
    for (var col = 0; col <= map.maxX; col++) {
        for (var y = row; y > 0; y--) {
            // 当前行样式 = 上一行样式
            map.mapDivs[y][col].className = map.mapDivs[y - 1][col].className;
        }
        map.mapDivs[0][col].className = "none";
    }
}

// 是否可以消行
Square.prototype._isCanRemoveLine = function(row, map) {
    var flag = [];
    for (var col = 0; col <= map.maxX; col++) {
        flag.push(map.mapDivs[row][col].className);
    }
    return !(flag.join(",").indexOf("none") > -1);
}
```

### 3.7 启动游戏
game.js 文件

``` javascript
var Game = function() {
    this.info = null;
    this.square = null;
    this.map = null;
}

// 开始游戏
Game.prototype.start = function() {
    // 初始化数据
    this.info = new Info();
    this.square = new Square(this.info);
    this.map = new Map(this.square);
    this.map.init({
        "map":document.getElementById("map"),
        "shadow":document.getElementById("shadow"),
        "next":document.getElementById("next")
    });
    // 监听游戏状态
    var that = this;
    var timeId = setInterval(function() {
        if (that.map.closeFlag) {
            that.map.close();
            clearInterval(that.map.timeId);
            clearInterval(that.info.timeId);
            var startBtn = document.getElementById("startBtn");
            startBtn.removeAttribute("disabled");
            startBtn.innerHTML = "重新开始";
            document.getElementById("pauseBtn").setAttribute("disabled",true);
            clearInterval(timeId);
        }
    },10);
}

window.onload = function() {
    var game = new Game();
    var startBtn = document.getElementById("startBtn");
    var pauseBtn = document.getElementById("pauseBtn");
    startBtn.addEventListener("click", function() {
         if (this.innerHTML == "开始游戏") {
            this.setAttribute("disabled",true);
            pauseBtn.style.display = "inline-block";
            pauseBtn.removeAttribute("disabled");
            game.start();
         } else {
            this.setAttribute("disabled",true);
            pauseBtn.style.display = "inline-block";
            pauseBtn.removeAttribute("disabled");
            game.restart();
         }
    });

    pauseBtn.addEventListener("click",function() {
        if (this.innerHTML == "暂停游戏") {
            this.innerHTML = "恢复游戏";
            game.pause();
        } else {
            this.innerHTML = "暂停游戏";
            game.recover();
        }
    });

}
```


## 四、源码

[俄罗斯方块下载](https://github.com/moonlightL/tetris)
