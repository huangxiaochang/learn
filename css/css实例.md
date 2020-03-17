css灵活使用实例
=============

## 1.使用:valid和invalid实现表单验证样式
可以使用伪类valid和invalid,配合pattern来实现表单验证的样式提示
```
<style type="text/css">
    .form-input:invalid {
     /*定义验证不通过时的样式*/
      outline-color: red;
    }
    .form-input:valid {
     /*定义验证通过时的样式*/
      outline-color: blue;
    }
  </style>
  <!-- 在pattern属性中编写验证的正则表达式 -->
  <input class="form-input" pattern="\d+" type="text" required>
```
## 2.:focus-within捕获焦点冒泡事件设置样式
元素触发focus和blur事件后会往父元素进行冒泡，可以在父元素上通过:focus-within捕获该冒泡事件来设置样式
```
  <style type="text/css">
    div.form-item:focus-within + p {
      color: red;
    }
  </style>
  
  <div class="form-item">
  	<!-- 当input触发focus事件后，父元素可以捕获到，然后进行相应的样式设置 -->
    <input type="text">
  </div>
  <p>123</p>
```
:placeholder-shown伪类可以获取设置了placeholder的input,textarea元素在进行输入时的事件，然后进行设置相应的样式
```
<style type="text/css">
    input:placeholder-shown {
      color: red;
    }
</style>
<input type="text">
```
## 3.使用max-height实现不定内容的手风琴效果
```
<style type="text/css">
	ul {
      list-style: none;
      border: 1px solid #ccc;
    }
    li {
      max-height: 0;
      transition: all 1s;
      overflow-y: auto;
    }
    li:hover {
      max-height: 100px;
    }
    h3 {
      cursor: pointer;
    }
    h3:hover + li {
      max-height: 100px;
    }
</style>

<ul> 
    <h3>rrr</h3>
    <li>
        1
        <br>
        2
        <br>
        3
        <br>
        4
        <br>
        5
        <br>
        6
    </li>
    <h3>ttt</h3>
        <li>
        1
        <br>
        2
        <br>
        3
    </li>
</ul>
```

## 4.利用background-attachment或者transform和perspective实现视差滚动效果
### 使用background-attachment实现
css的background-attachment属性决定了背景图像滚动模式，取值有：
- scroll: 背景图像相对元素本身固定，而不是随着元素的内容固定。
- local: 背景图像相对于内容是固定的，即会随着元素的内容进行滚动，如果内容超出元素的视口时。
- fixed: 背景图像相对于浏览器视口固定。

```
<style type="text/css">
    .container {
      width: 200px;
      height: 100px;
      overflow-y: auto;
      background-image: url('./123.png');
      color: red;
      background-attachment: local;
    }
  </style>
  <div class="container">
    111<br>222<br>333<br>
    444<br>555<br>666<br>
    777<br>888<br>999
  </div>
```
使用background-attachment:fixed来实现滚动视差。background-attachment:fixed表示背景图像相对于视口固定，即一个元素即使拥有滚动机制，背景图像有不会随着元素内容进行滚动，该背景图片从一开始就固定在**初始所在的位置**。

```
<style type="text/css">
    .content {
      height: 100vh;
      color: red;
      background: rgba(0, 0, 0, .7);
    }
    .fix_bg {
      height: 100vh;
      background-image: url('./123.png');
      background-attachment: fixed;
      background-size: cover;
      background-position: center center;
    }
  </style>
<section class="content">11</section>
  <section class=" fix_bg">22</section>
  <section class="content">33</section>
  <section class=" fix_bg">44</section>
  <section class="content">55</section>
  <section class=" fix_bg">66</section>
  <section class="content">77</section>
  <section class=" fix_bg">88</section>
  <section class="content">99</section>
```

### 使用transform和perspective实现
1. 给容器加上transform-style: preserve-3d和perspective属性。这样它的子元素就会处于3D空间中。
2. 给不同的子元素设置不同transform: translateZ()，这样不同子元素在z轴距离视口的距离就不一样。
3. 容器滚动条滚动时，由于子元素设置了不同的translateZ，所以他们滚动的上下距离(translateY)相对于视口是不同的。所以会达到了滚动视差的效果。

```
<style type="text/css">
    html {
      height: 100%;
      overflow: hidden;
    }
    body {
      transform-style: preserve-3d;
      perspective: 2px;
      height: 100vh;
      overflow-y: auto;
      transform-origin: center center;
    }
    .container {
      height: 200%;
      position: relative;
      font-size: 50px;
    }
    .child1 {
      transform: translateZ(-2px);
    }
    .child2 {
      transform: translateZ(-4px);
    }
    .child3 {
      transform: translateZ(-6px);
    }
    .child4 {
      transform: translateZ(-8px);
    }
  </style>
<div class="container">
    <div class="child1">11</div>
    <div class="child2">22</div>
    <div class="child3">33</div>
    <div class="child4">44</div>
</div>
```
