---
title: 原生 Javascript 编写贪吃蛇
date: 2017-09-18 14:45:23
tags: Javascript
categories: 前端
---
## 一、简单介绍

### 1.1 效果展示
![](http://ow97db1io.bkt.clouddn.com/snake-1.gif) 
 
### 1.2 实现思路

* 地图：创建一个 800px * 400px 的 div 元素来充当地图；地图可以看作由 40 x 20 个小 div（宽高都为20px） 元素组成。
* 食物：创建一个 宽高都为 20px 的 div 充当食物；食物通过 Math.random 分配随机位置。
* 蛇：创建一个二维数组充当蛇；二维数组存放蛇节（3个蛇身和1个蛇头），一维数组存放蛇节的位置、颜色和下一个蛇节对象。
* 蛇移动：先移动蛇尾，再移动蛇头。当前移动的蛇节位置为下一个蛇节未移动时的位置。

### 1.3 设计技术点
DOM操作、面向对象、事件操作和间隔函数setInterval
    
<!-- more -->

### 1.4 项目结构
![](http://ow97db1io.bkt.clouddn.com/snake-2.jpg)
    
## 二、实现步骤

### 2.1 绘制地图
#### 2.1.1 style.css 文件
``` css
html,body {
    margin:0;
    padding: 0;
}

.map {
    margin: 100px auto;
    text-align: center;
    background-color: pink;
    position: relative;
}

```

#### 2.1.2 map.js 文件

``` javascript
var Map = function(width,height) {
    this.width = width;
    this.height = height;
    this.id = "map";
}

Map.prototype.showMap = function(containerId) {
    // 创建地图并设置样式
    var mapDiv = document.createElement("div");
    mapDiv.style.width = this.width +"px";
    mapDiv.style.height = this.height + "px";
    mapDiv.className = this.id;
    mapDiv.id = this.id;
    // 将创建的地图添加到页面中
    document.getElementById(containerId).appendChild(mapDiv);
}
```

#### 2.1.3 game.js 文件创建地图

``` javascript
var Game = function() {

}

Game.prototype.start = function() {
    var map = new Map(800, 400); // 长宽数值必须是2的整数倍
    map.showMap("container");
}

```

#### 2.1.4 index.html 文件开始游戏

``` html
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <title>贪吃蛇</title>
    <link rel="stylesheet" type="text/css" href="css/style.css">
</head>

<body>
    <div id="container"></div>
    <script type="text/javascript" src="js/map.js"></script>
    <script type="text/javascript" src="js/food.js"></script>
    <script type="text/javascript" src="js/snake.js"></script>
    <script type="text/javascript" src="js/game.js"></script>
    <script type="text/javascript">
    window.onload = function() {
        var game = new Game();
        game.start();
    }
    </script>
</body>

</html>
```

#### 2.1.5 效果
![](http://ow97db1io.bkt.clouddn.com/snake-3.jpg)   

### 2.2 绘制食物
#### 2.2.1 food.js 文件

``` javascript
var Food = function(map) {
    this.size = 20; // 食物大小
    this.xFood = 0; // 食物x坐标
    this.yFood = 0; // 食物y坐标
    this.map = map; // 坐在地图
    this.foodDiv = null; // 地图中的食物
}

Food.prototype.showFood = function() {
    // 此处判断确保地图中只有一个食物
    if (this.foodDiv === null) {
        // 创建食物并设置样式
        this.foodDiv = document.createElement("div");
        this.foodDiv.style.width = this.foodDiv.style.height = this.size + "px";
        this.foodDiv.style.backgroundColor = "red";
        this.foodDiv.style.position = "absolute";
        // 将食物添加到地图上
        document.getElementById(this.map.id).appendChild(this.foodDiv);
    }

    // 食物步进值：20px
    // 食物权值坐标：X轴（0 - map.width/this.size）Y轴（0 - map.height/this.size）
    this.xFood = Math.floor(Math.random() * (this.map.width / this.size));
    this.yFood = Math.floor(Math.random() * (this.map.height / this.size));
    // 设置食物在地图的位置
    this.foodDiv.style.left = this.xFood * this.size + "px";
    this.foodDiv.style.top = this.yFood * this.size + "px";
}
```

#### 2.2.2 game.js 文件创建地图和食物对象

``` javascript
var Game = function() {

}

Game.prototype.start = function() {
    var map = new Map(800, 400); // 长宽数值必须是2的整数倍
    map.showMap("container");

    var food = new Food(map);
    food.showFood();
}

```

#### 2.2.3 效果
![](http://ow97db1io.bkt.clouddn.com/snake-4.jpg)

### 2.3 绘制小蛇

#### 2.3.1 snake.js 文件

``` javascript
var Snake = function(food) {
    this.size = 20;
    // 初始化蛇{x坐标，y坐标，颜色，蛇节对象}
    this.snakeBody = [
        {x:0,y:1,color:"black",obj:null},// 蛇身
        {x:1,y:1,color:"black",obj:null},// 蛇身
        {x:2,y:1,color:"black",obj:null},// 蛇身
        {x:3,y:1,color:"white",obj:null}// 蛇头
    ];
    this.direction = "right"; // 蛇移动方向
    this.food = food; //食物
}

// 显示蛇
Snake.prototype.showSnake = function() {
    //遍历蛇节，依次创建
    for (var i = 0; i < this.snakeBody.length; i++){
        //此处判断为了避免重复创建蛇节
        if (this.snakeBody[i].obj == null) {
            // 创建蛇节div，设置样式
            this.snakeBody[i].obj = document.createElement("div");
            this.snakeBody[i].obj.style.width = this.snakeBody[i].obj.style.height = this.size + "px";
            this.snakeBody[i].obj.style.backgroundColor = this.snakeBody[i].color;
            this.snakeBody[i].obj.style.position = "absolute";
            // 追加蛇节
            document.getElementById(this.food.map.id).appendChild(this.snakeBody[i].obj);
        }
        // 设置蛇在地图中的位置
        this.snakeBody[i].obj.style.left = this.snakeBody[i].x * this.size + "px";
        this.snakeBody[i].obj.style.top = this.snakeBody[i].y * this.size + "px";

    }
}
```
#### 2.3.2 game.js 文件创建地图、食物和蛇对象

``` javascript
var Game = function() {

}

Game.prototype.start = function() {
    var map = new Map(800, 400); // 长宽数值必须是2的整数倍
    map.showMap("container");

    var food = new Food(map);
    food.showFood();

    var snake = new Snake(food);
    snake.showSnake();

}
```
#### 2.3.3 效果
![](http://ow97db1io.bkt.clouddn.com/snake-5.jpg)

### 2.4 小蛇移动
#### 2.4.1 snake.js 文件，添加move方法

``` javascript
// 移动蛇
Snake.prototype.move = function() {
    // 非蛇头蛇节（当前蛇节的新坐标 为 下个蛇节的旧坐标）
    for (var i=0; i<this.snakeBody.length -1; i++) {
        this.snakeBody[i].x = this.snakeBody[i+1].x;
        this.snakeBody[i].y = this.snakeBody[i+1].y;
    }

    // 设置蛇头位置
    if (this.direction == "right") {
        // 蛇头x坐标累加
        this.snakeBody[this.snakeBody.length - 1].x += 1;
    }
    if (this.direction == "left") {
        // 蛇头x坐标累加
        this.snakeBody[this.snakeBody.length - 1].x -= 1;
    }
    if (this.direction == "up") {
        // 蛇头x坐标累加
        this.snakeBody[this.snakeBody.length - 1].y -= 1
    }
    if (this.direction == "down") {
        // 蛇头x坐标累加
        this.snakeBody[this.snakeBody.length - 1].y += 1;
    }

    this.showSnake();
}
```

#### 2.4.2 game.js 文件使用 interval 函数调用蛇对象的move方法

``` javascript
var Game = function() {

}

Game.prototype.start = function() {
    var map = new Map(800, 400); // 长宽数值必须是2的整数倍
    map.showMap("container");

    var food = new Food(map);
    food.showFood();

    var snake = new Snake(food);
    snake.showSnake();

    setInterval(function() {
        snake.move();
    }, 100);

}
```


### 2.5 控制小蛇移动方向
game.js 添加键盘按下事件

``` javascript
var Game = function() {

}

Game.prototype.start = function() {
    var map = new Map(800, 400); // 长宽数值必须是2的整数倍
    map.showMap("container");

    var food = new Food(map);
    food.showFood();

    var snake = new Snake(food);
    snake.showSnake();

    setInterval(function() {
        snake.move();
    }, 100);

    // 键盘控制
    document.onkeydown = function(e) {
        switch (e.keyCode) {
            case 37:
                snake.direction = "left";
                break;
            case 38:
                snake.direction = "up";
                break;
            case 39:
                snake.direction = "right";
                break;
            case 40:
                snake.direction = "down";
                break;
        }
        snake.showSnake();
    }
}
```

### 2.6 小蛇吃食物
snake.js 文件在 move 方法   this.showSnake() 之前添加逻辑判断

``` javascript
// 蛇头坐标
var xSnakeHead = this.snakeBody[this.snakeBody.length -1].x;
var ySnakeHead = this.snakeBody[this.snakeBody.length -1].y;

//判断蛇吃否吃到食物
if (xSnakeHead == this.food.xFood && ySnakeHead == this.food.yFood) {
    // 增加蛇长
    var newBody = {x:this.snakeBody[0].x,y:this.snakeBody[0].y,color:"black",obj:null};
    this.snakeBody.unshift(newBody);
    // 食物消失，再随机生成
    this.food.showFood();
}
```

### 2.7 小蛇移动范围
snake.js 文件在 move 方法   this.showSnake() 之前添加逻辑判断

``` javascript
// 控制小蛇移动范围
if (xSnakeHead < 0 || xSnakeHead >= this.food.map.width/this.size
    || ySnakeHead <0 || ySnakeHead >= this.food.map.height/this.size) {
    alert("游戏结束!");
    window.location.reload();
}

// 不能吃自己
for (var j=0; j<this.snakeBody.length -1; j++) {
    // 蛇头坐标 = 蛇身坐标，游戏结束
    if (this.snakeBody[j].x == xSnakeHead && this.snakeBody[j].y == ySnakeHead) {
        alert("游戏结束!");
        window.location.reload();
    }
}
```

## 三、源码下载

[贪吃蛇](https://github.com/moonlightL/snake)
