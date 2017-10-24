---
title: UML 类图介绍
date: 2017-09-20 15:05:07
tags: UML
categories: 其他
---
## 一、基本概念
类图是描述类、接口以及类之间关系的图。

### 1.1 作用
类图常用来描述业务或软件系统的组成、结构和关系

### 1.2 类描述
类在 UML 中通常以实线矩形框表示。
矩形框中有若干分隔框，分别包含类名、属性、行为等元素。如下图：
![](http://ow97db1io.bkt.clouddn.com/uml-1.jpg)

**类名：图中蓝色背景的字，如果字体为斜体，表名该类为抽象类**
**属性：类名下边的区域**
**行为：属性下边的区域**
**可见性：属性和行为前边的 "+"、"-" 和 "#"（图中未标注） 分别表示 public、private 和 protected**

<!-- more -->

### 1.3 接口描述
接口的类图表述与类大致相同，不同的是接口名要添加 Interface 标识，且行为的可见性必须用 "+" 表示。如下图：
![](http://ow97db1io.bkt.clouddn.com/uml-2.jpg)

## 二、类图中的六种关系
* 继承
* 实现
* 关联
* 依赖
* 组合
* 聚合

### 2.1 继承（Inherit）
继承是面向对象语言的三个特性之一。子类继承父类，子类可以使用父类所有非私有的属性和方法，其UML类图表示如下：
![](http://ow97db1io.bkt.clouddn.com/uml-03.jpg)

UML 类图中继承关系使用**空心三角形+实线**表示。

### 2.2 实现（Realization）
实现与继承类似，实现类继承接口中的方法，但是方法必须由实现类自己实现，其UML类图表示如下：
![](http://ow97db1io.bkt.clouddn.com/uml-4.jpg)

UML 类图中实现关系使用**空心三角形+虚线**表示。

### 2.3 关联（Association）
指类与类之间的关系，它使得一个类知道另一个类的属性和方法。关联可以是双向的，也可以是单向的。

用 Java 代码表示企鹅只存在在南极，与气候有关系：
``` java
public class Penguin {
    private Climate climate;
}
```

其UML类图表示如下：
![](http://ow97db1io.bkt.clouddn.com/uml-5.jpg)

UML 类图中关联关系使用**实线箭头**表示。

### 2.4 依赖（Dependency）
指类与类之间的联接，依赖关系表示一个类依赖于另一个类的定义。一般而言，依赖关系在Java语言中体现为局域变量、方法的形参，或者对静态方法的调用。

用 Java 代码表示程序员工作需要用到电脑：
``` java
public class Programmer{
    public void work(Computer computer) {
        
    }
}
```

其UML类图表示如下：
![](http://ow97db1io.bkt.clouddn.com/uml-6.jpg)

UML 类图中依赖关系使用**虚线箭头**表示。

### 2.5 组合（Composition） 
关联关系的一种，表示一种强的“拥有”关系，体现了严格的部分和整体的关系，部分和整体的生命周期一样。

用 Java 表示每只鸟都有翅膀：
``` java
public class Bird {
    private Wing wing;
    public Bird() {
        wing = new Wing();
    }
}
```

其UML类图表示如下：
![](http://ow97db1io.bkt.clouddn.com/uml-7.jpg)

UML 类图中组合关系使用**实心菱形实线**表示。

### 2.6 聚合（Aggressgation）
关联关系的一种，表示一种弱的“拥有”关系，体现的是A对象可以包含B对象，但是B对象不是A对象的一部分。

用 Java 代码表示大雁是群居动物，每只大雁都属于一个雁群，一个雁群可以有多只大雁：
``` java
public class WildGooseAggregate {
    private List<WildGoose> wideGooses;
}
```

其UML类图表示如下：
![](http://ow97db1io.bkt.clouddn.com/uml-8.jpg)

UML 类图中聚合关系使用**空心菱形实线**表示。

## 三、参考资料
* <http://www.cnblogs.com/javawebsoa/archive/2013/08/01/3230737.html>
