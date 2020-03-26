---
title: win10-calendar-follow-light-effect
comments: true
tags:
  - win10
  - calendar
  - style
categories: Code
abbrlink: 62749
date: 2020-03-25 16:34:54
---

今天无意间发现win10日历的一个样式，类似于追光灯的效果，大家可以打开win10的日历，用鼠标移动看一下。感觉挺有意思的，想想是否能用js来实现这种效果。

![image-20200325164248791](https://raw.githubusercontent.com/ChaosySama/ChaosySama.github.io/img/image-20200325164248791.png)

最终进行了一些调查和调整，做出了一个完整的demo，有兴趣的可以去看看： [戳我](https://codepen.io/chaosy/pen/gOpBqBx)

下面介绍这个demo的思考和开发流程。

<!-- more -->

### 观察效果

首先我们需要先观察整个交互逻辑和样式效果，来确定一个大体实现思路。

这里我们不考虑日历其他一些点击等交互，只看根据鼠标移动，导致渐进渲染的这个光源效果。

这个效果是跟随鼠标的，也就是说鼠标停留在同一个日期格内的不同位置，光源也是有位移变化的。

那么进而想到是监听`mousemove`方法来改变css。

并且鼠标停留的日期格是需要高亮边框的，所以这个需要用`:hover`伪类来实现。

那么大体的效果先考虑到这里，我们先把页面框架和元素构建起来，再来摸索实现方式。

### 页面构建

这里的页面我们采取最简便的方式，画一个4 x 7的横排固定日历。实现起来很简单，相关代码如下：

```html
<div class="calendar">
    <div class="row">
        <li>1</li>
        <li>2</li>
        <li>3</li>
        <li>4</li>
        <li>5</li>
        <li>6</li>
        <li>7</li>
    </div>
    <div class="row">
        <li>8</li>
        <li>9</li>
        <li>10</li>
        <li>11</li>
        <li>12</li>
        <li>13</li>
        <li>14</li>
    </div>
    <div class="row">
        <li>15</li>
        <li>16</li>
        <li>17</li>
        <li>18</li>
        <li>19</li>
        <li>20</li>
        <li>21</li>
    </div>
    <div class="row">
        <li>22</li>
        <li>23</li>
        <li>24</li>
        <li>25</li>
        <li>26</li>
        <li>27</li>
        <li>28</li>
    </div>
</div>
```

因为使用原生开发，这里显得代码比较多，不过还好比较简单。

接着我们加一些css样式，让他看起来比较像粗糙的日历。

```css
body {
  display: flex;
  justify-content: center;
  align-items: center;
}
.calendar {
  display: flex;
  flex-wrap: wrap;
  flex-direction: column;
  padding: 20px;
  background-color: #262828;
}
.row {
  display: flex;
  flex-direction: row;
}
.row li {
  width: 30px;
  height: 25px;
  color: #efefed;
  text-align: center;
  list-style: none;
  padding: 10px;
  margin: 3px;
  background-color: #262828;
  user-select: none;
}
.row li:hover {
  outline: solid 3px rgb(150, 153, 155);
}
```

效果如下：

![image-20200325170407342](https://raw.githubusercontent.com/ChaosySama/ChaosySama.github.io/img/image-20200325170407342.png)

这里使用outline而不使用border是为了不影响布局，并且宽度和margin设置一样，这样可以无缝覆盖。

### 追光灯效果

基础页面和样式构建好了，我们来试试加上追光灯效果。这里需要用到的样式就是`radial-gradient`径向渐变。

实现思路也比较简单，我们可以在日期元素外层套一个背景层，然后使用径向渐变，根据鼠标的位置来实现一个光圈，由于日期元素在背景层上方，可以不被光圈遮挡。

于是`html`里加上背景层：

```html
<div class="calendar">
    <div class="back">
        <div class="row">
            <li>1</li>
            <li>2</li>
            ...
        </div>
        <div class="row">...</div>
        ...
    </div>
</div>
```

js增加监听事件，获取鼠标坐标，使用径向渐变渲染：

```javascript
addEventListener("mousemove", function(ev) {
  var radius = 80;
  var back = document.getElementsByClassName("back")[0];
  var bx = back.offsetLeft;
  var by = back.offsetTop;
  var dx = ev.clientX - bx;
  var dy = ev.clientY - by;
  back.style.background = `radial-gradient(${radius}px at ${dx}px ${dy}px, #898B8D, #262828)`;
});
```

这里尽量不适用es6的语法，保证兼容性。

看一下效果：

![image-20200325175312534](https://raw.githubusercontent.com/ChaosySama/ChaosySama.github.io/img/image-20200325175312534.png)

可以看到，我们已经实现了这个效果了，不过有点不够完美，我们继续做一些样式优化。

### 样式优化

每个元素直接需要有空隙，但是如果单纯的用margin来设置，背景层的光圈不会被覆盖，所以这里我们投机取巧，在html中加入一些遮挡用元素在日期元素之间，分别为每行之间的水平遮挡条，和各日期元素之间的竖直遮挡条：

```html
<div class="calendar">
  <div class="back">
    <div class="row">
      <li>1</li>
      <div class="vlider"></div>
      <li>2</li>
      <div class="vlider"></div>
      <li>3</li>
      <div class="vlider"></div>
      <li>4</li>
      <div class="vlider"></div>
      <li>5</li>
      <div class="vlider"></div>
      <li>6</li>
      <div class="vlider"></div>
      <li>7</li>
    </div>
    <div class="slider"></div>
    <div class="row">
      <li>8</li>
      <div class="vlider"></div>
      <li>9</li>
      <div class="vlider"></div>
      <li>10</li>
      <div class="vlider"></div>
      <li>11</li>
      <div class="vlider"></div>
      <li>12</li>
      <div class="vlider"></div>
      <li>13</li>
      <div class="vlider"></div>
      <li>14</li>
    </div>
    <div class="slider"></div>
    <div class="row">
      <li>15</li>
      <div class="vlider"></div>
      <li>16</li>
      <div class="vlider"></div>
      <li>17</li>
      <div class="vlider"></div>
      <li>18</li>
      <div class="vlider"></div>
      <li>19</li>
      <div class="vlider"></div>
      <li>20</li>
      <div class="vlider"></div>
      <li>21</li>
    </div>
    <div class="slider"></div>
    <div class="row">
      <li>22</li>
      <div class="vlider"></div>
      <li>23</li>
      <div class="vlider"></div>
      <li>24</li>
      <div class="vlider"></div>
      <li>25</li>
      <div class="vlider"></div>
      <li>26</li>
      <div class="vlider"></div>
      <li>27</li>
      <div class="vlider"></div>
      <li>28</li>
    </div>
  </div>
</div>
```

然后加上对应的样式：

```css
.slider {
  width: 100%;
  height: 3px;
  background-color: #262828;
}
.vlider {
  width: 3px;
  background-color: #262828;
}
```

再来看一下效果：

![image-20200326091945305](https://raw.githubusercontent.com/ChaosySama/ChaosySama.github.io/img/image-20200326091945305.png)

是不是已经成功实现了！这里各种边框和遮挡的宽度，还有光源的直径，都是可以调整的，为了看起来方便，就调的大了一些。

### 代码优化

这样就够了嘛？从效果上来说是的，不过对于有代码洁癖的人来说，上面一坨的html标签，尤其是遮挡元素，十分难看，并且修改起来也十分麻烦。如果是vue等等框架模板的话还好解决，这边只能用js来动态生成了。

html我们只保留一个最外层元素：

```html
<div class="calendar"></div>
```

css不做改变。

js中加入生成元素的相关代码，这里我们封装成方法：

```javascript
function createElementWithClass(className, type) {
  type = type || "div";
  var el = document.createElement(type);
  el.setAttribute("class", className);
  return el;
}

function addCalendarEl(startDate) {
  startDate = startDate || 1;
  var calendar = document.getElementsByClassName("calendar")[0];
  var back = createElementWithClass("back");
  for(var i = 0; i < 4; i++) {
    var row = createElementWithClass("row");
    for(var j = 0; j < 7; j++) {
      var li = document.createElement("li");
      li.innerHTML = startDate++;
      if(startDate > 31) startDate = 1;
      row.appendChild(li);
      j < 6 && row.appendChild(createElementWithClass("vlider"));
    }
    back.appendChild(row);
    i < 3 && back.appendChild(createElementWithClass("slider"));
  }
  calendar.appendChild(back);
}

addCalendarEl(8);
```

大功告成了。

想直接看demo的可以： [戳我](https://codepen.io/chaosy/pen/gOpBqBx)

### 思考

这边虽然只是做了个简单的demo，不过发现->思考->动手->优化，整个过程还是比较完整的。不论什么问题，应该都是简化成小问题去做，这样会得心应手很多。

这边的demo本意不是做一个日历，而是实现追光灯和蒙版效果。

这种效果以后也可以用在元素边框颜色渐变高亮等地方。

那么，这次的练手和分享就到这里啦。