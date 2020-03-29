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

## 5.使用linear-gradient和background配合实现多种效果
线性渐变linear-gradient的用法：
linear-gradient([param1], startColor position, ..., endColor position)
param1参数可选，表示渐变的方向，默认从上往下。取值可以是to right/to top, 45deg等
startColor/endColor表示开始渐变的颜色，position表示该颜色开始的位置。

### 1.斜条纹
```
<style type="text/css">
	/* 使用linear-gradient实现 */
    .container {
    	width: 400px;
      	height: 40px;
      	/*linear-gradient 只能实现45度的斜条纹*/
      	background-image:  linear-gradient(45deg, #eee 25%, red 0, red 50%, #eee 0, #eee 75%,red 75%);
      	/* 要定义background-size成小块，才会重复 */
      	background-size: 40px 40px;
    }
    /* 或者使用repeating-linear-gradient实现 */
    .container {
    	width: 400px;
      	height: 40px;
      	/*repeating-linear-gradient 可以实现任意角度的斜条纹*/
      	background-image: repeating-linear-gradient(60deg, #eee 20px, red 0, red 40px);
    }
</style>
<div class="container"></div>

```

### 2.无限滚动的进度条
```
.container {
      width: 300px;
      height: 20px;
      background-image: linear-gradient(45deg,#fff 25%, #ccc 25%, #ccc 50%, #fff 50%, #fff 75%, #ccc 0);
      background-size: 20px 20px;
      animation: move 5s linear infinite;
    }
    @keyframes move {
      0% {
        background-position: 0 0;

      }
      100% {
        background-position: 100% 100%;
      }
    }
```

### 3.多个格子
```
<style type="text/css">
	/* 使用linear-gradient实现 */
    .container {
    	width: 400px;
      	height: 40px;
      	/*45deg 代表从左下角到右上角*/
      	background-image: linear-gradient(45deg, #eee 25%, transparent 25%, transparent 75%, #eee 75%),
      linear-gradient(45deg, #eee 25%, transparent 25%, transparent 75%, #eee 75%);
      /*指定每个背景图像的开始位置*/
      background-position: 0 0,20px 20px;
      background-size: 40px 40px;
    }
    /*background-position指定的位置开始绘制图像，对于边界来说，是会被截断的。即只会取在元素内的部分*/
</style>
<div class="container"></div>

```

### 4.切角
```
<style type="text/css">
.container {
    width: 200px;
    height: 200px;
    /*border: 1px solid #d2d2d2;*/
    background: 
    /*使用top left来定位每个背景图像的起始位置*/
    linear-gradient(135deg,transparent 15px, red 0) top left,
    linear-gradient(-135deg,transparent 15px, green 0) top right,
    linear-gradient(45deg,transparent 15px, yellow 0) bottom left,
    linear-gradient(-45deg, transparent 15px, #ccc 0) bottom right;
    /*要指定每个背景图像的大小，否者会铺满整个背景，会相互覆盖掉切角*/
    background-size: 50% 50%;
    /*指定大小之后，如果没有关掉平铺，那么每个背景都会平铺4次，也会相互覆盖掉切角*/
    background-repeat: no-repeat;
}
</style>
<div class="container"></div>
```

### 5.动态边框
```
.container {
    	display: inline-block;
    	font-size: 50px;
    	transition: all 1s;
    	/* left top/0 2px： 为定义背景图像的位置和大小*/
    	background: linear-gradient(0, red 2px, red 2px) no-repeat left top/0 2px,
    	linear-gradient(-90deg, red 2px, red 2px) no-repeat right top/ 2px 0,
    	linear-gradient(-180deg, red 2px, red 2px) no-repeat right bottom/0 2px,
    	linear-gradient(-270deg, red 2px, red 2px) no-repeat left bottom/ 2px 0;
    	
    }
<div class="container">css css css</div>
```

## 6.使用radial-gradient实现各种样式
radial-gradient表示从一个中点，向圆或者椭圆渐变拓展，用法:
radial-gradient(radius shape extentKeyword  circleCenter, startColor position, ..., endColor position);

- radius: 渐变的半径大小，可选。
- shape: 值: circle:圆，ellipse: 椭圆；可选。
- extentKeyword: 用于描述边缘轮廓的具体位置；可选。如：closest-side: 渐变的边缘形状与容器距离渐变中心最近的一边相切或者至少与距离渐变中心点最近的垂直和水平变相切。
- circleCenter：定义渐变中心的位置；如值可以为'at 40% 50%''。
- startColor，position: 开始渐变的颜色，和该颜色的位置
- endColor，position: 结束渐变的颜色，和该颜色的位置

### 1.立体球体
```
<style type="text/css">
    .container {
      width: 100px;
      height: 100px;
      border-radius: 50%;
      background-image: radial-gradient(circle, #fff 0, #000 100%);
    }
</style>
<div class="container"></div>
```
### 2.半圆切角
```
<style type="text/css">
    .container {
      width: 100px;
      height: 100px;
      width: 200px;
      height: 100px;
      background: 
      radial-gradient(circle at 0% 50%, transparent 15px, #000 0) left,
      radial-gradient(circle at 100% 50%, transparent 15px, red 0) right;
      background-size: 50% 100%;
      background-repeat: no-repeat;
    }
</style>
<div class="container"></div>
```

## 7.下划线跟随鼠标的导航栏
```
<style type="text/css">
		ul {
			list-style: none;
			display: flex;
			width: 500px;
		}
		li {
			padding: 10px;
			position: relative;
			cursor: pointer;
		}
		li::before {
			content: '';
			width: 100%;
			height: 0;
			border: 2px solid transparent;
			position: absolute;
			bottom: 0;
			/*下划线从右边开始*/
			left: 100%;
			width: 0;
			transition: all 300ms;
		}
		li:hover::before {
			border-color: blue;
			width: 100%;
			left: 0;
		}
		/* 关键样式，选中某个导航时，然它的兄弟导航下划线的left为从左边开始*/
		li:hover + li::before {
			left: 0;
		}
	</style>
<ul>
    <li>是的日日</li>
    <li>饭堂</li>
    <li>儿童为</li>
    <li>热特温柔让他</li>
    <li>问</li>
  </ul>
```

## 8.利用mix-blend-mode混合模式实现很多种效果
mix-blend-mode属性描述了元素的内容和背景与直系父元素的内容和背景的混合模式。有多个取值，如hue等。可以利用该属性属性文字故障，图片换装等效果.
图片换装：
```
<style type="text/css">
.container {
    	width: 500px;
    	height: 500px;
    	border: 1px solid #ccc;
    	position: relative;
    }
    input {
    	position: absolute;
    	width: 100%;
    	height: 100%;
    	cursor: pointer;
    	// 指定颜色选择器的颜色和直接父元素的混合模式，这样可以实现父元素中内容的颜色变换
    	mix-blend-mode: hue;
    	/*mix-blend-mode: lighten;*/
    }
    img {
		width: 100%;
    	height: 100%;
    	object-fit: cover;
    }
</style>
<div class="container">
    <input type="color" value="#ff6666">
    <img src="./123.png" alt="">
</div>
```

## 9.使用-webkit-box-reflect实现元素倒影的功能
用法:
-webkit-box-reflect: direction gap;   
direction: 倒影的方向;可以取below等值。
gap: 元素和倒影直接的间隙。

```
<style type="text/css">
    .container {
    	width: 200px;
    	height: 200px;
    	background: linear-gradient(#fff, #000);
    	-webkit-box-reflect: below 4px;   
    }
</style>
<div class="container"></div>
```

## 10.三维建模的立方体
使用transform， perspective， transform-style来进行三维建模
```
<style type="text/css">
:root {
    	--width: 150px;
    	--height: 150px;
    	--translateHeight: 75px;
    }
    .td-cube {
    	padding: 50px;
			width: var(--width);
			height: var(--height);
			perspective: 1000px;
		}
		ul {
			position: relative;
			width: 100%;
			height: 100%;
			transform: rotateX(-15deg) rotateY(15deg);
			transform-style: preserve-3d;
			animation: rotate 5s infinite linear;
		}
		li {
			display: flex;
			position: absolute;
			justify-content: center;
			align-items: center;
			width: var(--width);
			height: var(--height);
			opacity: .8;
			font-size: 50px;
			color: #fff;
		}
		.front {
			background-color: #f66;
			transform: translateZ(var(--translateHeight));
		}
		.back {
			background-color: #66f;
			transform: rotateY(180deg) translateZ(var(--translateHeight));
		}
		.top {
			background-color: #f90;
			transform: rotateX(90deg) translateZ(var(--translateHeight));
		}
		.bottom {
			background-color: #09f;
			transform: rotateX(-90deg) translateZ(var(--translateHeight));
		}
		.left {
			background-color: #9c3;
			transform: rotateY(-90deg) translateZ(var(--translateHeight));
		}
		.right {
			background-color: #3c9;
			transform: rotateY(90deg) translateZ(var(--translateHeight));
		}
		@keyframes rotate {
			from {
				transform: rotateY(0) rotateX(0);
			}
			to {
				transform: rotateY(-1turn) rotateX(-1turn);
			}
		}
</style>
<div class="td-cube">
    <ul>
        <li class="front">1</li>
        <li class="back">2</li>
        <li class="top">3</li>
        <li class="bottom">4</li>
        <li class="left">5</li>
        <li class="right">6</li>
    </ul>
</div>
```

## 11.星级评分
使用input:checked, flex-direction: row-reverse, 和~选择器实现星级评分
```
<style type="text/css">
    .container {
    	display: flex;
    	flex-direction: row-reverse;
    }
    input {
    	appearance: none;
    }
    .star-item {
    	/*appearance: 让一个元素显示成希望的样式（系统所用的主题样式）*/
    	-webkit-appearance: none;
    	cursor: pointer;
    	width: 30px;
    	height: 30px;
    	line-height: 30px;
    	font-size: 30px;
    	text-align: center;
    	outline: none;
    }
    .star-item::after {
    	content: '☆';
    	color: #66f;
    }
    .star-item:checked::after, .star-item:hover::after,
    .star-item:hover ~ .star-item::after, .star-item:checked ~ .star-item::after{
    	content: '★';
    	color: #f66;
    }
	</style>
	<div class="container">
		<input class="star-item" value="5" type="radio" name="rate">
		<input class="star-item" value="4" type="radio" name="rate">
		<input class="star-item" value="3" type="radio" name="rate">
		<input class="star-item" value="2" type="radio" name="rate">
		<input class="star-item" value="1" type="radio" name="rate">
	</div>
```

## 12.自动打字效果
利用动画animation和overflow来动态改变长度来实现可见的字符。利用动画的时间函数steps()来实现动画为跳跃式。同时通过ch长度单位设置动画字体元素的长度刚好为字体的长度。
```
.container {
	overflow: hidden;
    white-space: nowrap;
    font-size: 22px;
    border-right: 1px solid transparent;
    width: 34ch;
    animation: move 5s steps(35) 0s backwards, caret 500ms steps(1)0s 10 forwards;
}
    @keyframes move {
    	0% {
				width: 0;
    	}
    }
    @keyframes caret {
    	50% {
    		border-right-color: currentColor;
    	}
    }
<div class="container">
	sd 十多年 第三方股东 dfgsd 的分割目的是
</div>
```

## 13. 底部内容自适应
	当主内容不超出可视区域时，底部固定在可视区域的底部，当主内容超过可视区域时，底部跟随着在主内容的底部。
```
<style type="text/css">
	/* 可视区域 */
		.container {
			height: 400px;
			overflow-y: auto;
			border: 1px solid #ccc;
			position: relative;
			box-sizing: border-box;
		}
	/* 主内容 */
		.main {
			min-height: 100%;
			padding-bottom: 50px;
			box-sizing: border-box;
		}
	/* 底部 */
		.footer {
			height: 40px;
			line-height: 40px;
			background: yellow;
			margin-top: -40px;
		}
    
</style>
<div class="container">
    <div class="main">weqwe
        webr	<br>webr	<br>webr	<br>webr	<br>webr	<br>
        webr	<br>webr	<br>webr	<br>webr	<br>webr	<br>
        webr	<br>webr	<br>webr	<br>webr	<br>webr	<br>
        webr	<br>webr	<br>webr	<br>webr	<br>webr	<br>
        webr	<br>webr	<br>webr	<br>webr	<br>webr	<br>
    </div>
    <footer class="footer">button</footer>
</div>
```
## 14. 宽高固定比例变化
	宽度自适应，然后高度随着宽度固定比例进行变化。实现这种效果，利用了padding-top/bottom的百分比的基准是父元素宽度的特点，然后容器的一个子元素设置padding-top/bottom来撑起它的高度，从而实现容器的宽高固定比例。

高度随着宽度固定比例变化。
```
<style type="text/css">
    .container {
        width: 100%;
        background: yellow;
        position: relative;
    }
    /* 让container的伪类子元素撑起它的高度 */
    .container:before {
        content: '';
        width: 100%;
        /* 要设置成块级元素，才能撑起高度 */
        display: block;
        height: 0;
        /* 以container宽度为基准 */
        padding-bottom: 50%;
    }
    .content {
        position: absolute;
        left: 0;
        right: 0;
        bottom: 0;
        top: 0;
    }
</style>
<div class="container">
	<div class="content">123456789</div>
</div>
```
宽度随着高度固定比例变化
因为现阶段并没有什么属性的比例基准是元素的高度，所以不能像上面那样使用padding-bottom属性来实现，其中一种实现的方式就是通过变异的方法实现，即把高度随着宽度固定比例变化的容器，使用transform: rotate(90deg).即可实现容器的宽度随着高度固定比例变化。但是会产生一个新的问题就是，容器的子元素也会被旋转，所以文字的方向发生了改变。