# CSS 面试题

## 基础概念

### 1. 盒模型
**问题**: 解释 CSS 盒模型，标准盒模型和 IE 盒模型的区别

**答案**:
```css
/* 标准盒模型 (content-box) */
.standard-box {
  box-sizing: content-box; /* 默认值 */
  width: 200px;
  height: 100px;
  padding: 20px;
  border: 5px solid #000;
  margin: 10px;
  /* 总宽度 = 200 + 20*2 + 5*2 = 250px */
}

/* IE 盒模型 (border-box) */
.ie-box {
  box-sizing: border-box;
  width: 200px;
  height: 100px;
  padding: 20px;
  border: 5px solid #000;
  margin: 10px;
  /* 总宽度 = 200px (包含 padding 和 border) */
  /* 内容宽度 = 200 - 20*2 - 5*2 = 150px */
}

/* 全局设置 border-box */
* {
  box-sizing: border-box;
}
```

### 2. 选择器优先级
**问题**: CSS 选择器的优先级规则

**答案**:
```css
/* 优先级计算规则：
   内联样式：1000
   ID选择器：100
   类选择器、属性选择器、伪类：10
   元素选择器、伪元素：1
*/

/* 优先级 = 0001 */
p {
  color: red;
}

/* 优先级 = 0010 */
.text {
  color: blue;
}

/* 优先级 = 0100 */
#title {
  color: green;
}

/* 优先级 = 0111 */
#title.text p {
  color: orange;
}

/* 优先级 = 1000 */
/* <p style="color: purple;">内联样式</p> */

/* !important 最高优先级 */
p {
  color: black !important;
}

/* 相同优先级，后面的覆盖前面的 */
p { color: red; }
p { color: blue; } /* 生效 */
```

### 3. 选择器类型
**问题**: 常用的 CSS 选择器有哪些？

**答案**:
```css
/* 基础选择器 */
* { } /* 通配符选择器 */
p { } /* 元素选择器 */
.class { } /* 类选择器 */
#id { } /* ID选择器 */

/* 组合选择器 */
div p { } /* 后代选择器 */
div > p { } /* 子元素选择器 */
div + p { } /* 相邻兄弟选择器 */
div ~ p { } /* 通用兄弟选择器 */

/* 属性选择器 */
[attr] { } /* 有 attr 属性 */
[attr="value"] { } /* attr 等于 value */
[attr^="value"] { } /* attr 以 value 开头 */
[attr$="value"] { } /* attr 以 value 结尾 */
[attr*="value"] { } /* attr 包含 value */
[attr~="value"] { } /* attr 包含独立的 value 单词 */

/* 伪类选择器 */
:hover { } /* 鼠标悬停 */
:active { } /* 激活状态 */
:focus { } /* 焦点状态 */
:first-child { } /* 第一个子元素 */
:last-child { } /* 最后一个子元素 */
:nth-child(n) { } /* 第 n 个子元素 */
:nth-of-type(n) { } /* 第 n 个同类型元素 */
:not(selector) { } /* 不匹配选择器的元素 */

/* 伪元素选择器 */
::before { } /* 元素前插入内容 */
::after { } /* 元素后插入内容 */
::first-line { } /* 第一行 */
::first-letter { } /* 第一个字母 */
::selection { } /* 选中的文本 */
```

## 布局

### 4. Flexbox 布局
**问题**: Flexbox 的主要属性和使用场景

**答案**:
```css
/* 容器属性 */
.flex-container {
  display: flex; /* 或 inline-flex */
  
  /* 主轴方向 */
  flex-direction: row; /* row | row-reverse | column | column-reverse */
  
  /* 换行 */
  flex-wrap: nowrap; /* nowrap | wrap | wrap-reverse */
  
  /* 主轴对齐 */
  justify-content: flex-start; /* flex-start | flex-end | center | space-between | space-around | space-evenly */
  
  /* 交叉轴对齐 */
  align-items: stretch; /* stretch | flex-start | flex-end | center | baseline */
  
  /* 多行对齐 */
  align-content: stretch; /* stretch | flex-start | flex-end | center | space-between | space-around */
  
  /* 简写 */
  flex-flow: row nowrap;
}

/* 项目属性 */
.flex-item {
  /* 放大比例 */
  flex-grow: 0; /* 默认 0，不放大 */
  
  /* 缩小比例 */
  flex-shrink: 1; /* 默认 1，空间不足时缩小 */
  
  /* 基础大小 */
  flex-basis: auto; /* 默认 auto */
  
  /* 简写 */
  flex: 0 1 auto; /* grow shrink basis */
  flex: 1; /* 等价于 1 1 0% */
  flex: auto; /* 等价于 1 1 auto */
  flex: none; /* 等价于 0 0 auto */
  
  /* 单独对齐 */
  align-self: auto; /* auto | flex-start | flex-end | center | baseline | stretch */
  
  /* 顺序 */
  order: 0; /* 默认 0，数值越小越靠前 */
}

/* 常用布局示例 */
/* 水平垂直居中 */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 两端对齐 */
.space-between {
  display: flex;
  justify-content: space-between;
}

/* 等宽布局 */
.equal-width .item {
  flex: 1;
}
```

### 5. Grid 布局
**问题**: CSS Grid 的基本使用和属性

**答案**:
```css
/* 容器属性 */
.grid-container {
  display: grid; /* 或 inline-grid */
  
  /* 定义行和列 */
  grid-template-columns: 100px 200px 1fr; /* 固定 固定 自适应 */
  grid-template-rows: 100px auto; /* 固定 自适应 */
  
  /* 使用 repeat */
  grid-template-columns: repeat(3, 1fr); /* 三等分 */
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); /* 响应式 */
  
  /* 命名网格线 */
  grid-template-columns: [start] 100px [line2] 200px [end];
  
  /* 间距 */
  grid-gap: 10px; /* 行列间距 */
  grid-row-gap: 10px; /* 行间距 */
  grid-column-gap: 20px; /* 列间距 */
  gap: 10px 20px; /* 新语法 */
  
  /* 对齐 */
  justify-items: start; /* start | end | center | stretch */
  align-items: start; /* start | end | center | stretch */
  justify-content: start; /* start | end | center | stretch | space-around | space-between | space-evenly */
  align-content: start;
  
  /* 自动网格 */
  grid-auto-columns: 100px; /* 隐式列大小 */
  grid-auto-rows: 100px; /* 隐式行大小 */
  grid-auto-flow: row; /* row | column | dense */
}

/* 项目属性 */
.grid-item {
  /* 指定位置 */
  grid-column-start: 1;
  grid-column-end: 3;
  grid-row-start: 1;
  grid-row-end: 2;
  
  /* 简写 */
  grid-column: 1 / 3; /* start / end */
  grid-row: 1 / 2;
  grid-area: 1 / 1 / 2 / 3; /* row-start / col-start / row-end / col-end */
  
  /* 跨越 */
  grid-column: span 2; /* 跨越 2 列 */
  grid-row: span 3; /* 跨越 3 行 */
  
  /* 对齐 */
  justify-self: center; /* start | end | center | stretch */
  align-self: center;
}

/* 网格区域命名 */
.grid-template {
  display: grid;
  grid-template-areas: 
    "header header header"
    "sidebar main main"
    "footer footer footer";
  grid-template-columns: 200px 1fr 1fr;
  grid-template-rows: 80px 1fr 60px;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

### 6. 定位
**问题**: CSS 定位的几种方式和区别

**答案**:
```css
/* static: 默认定位，按正常文档流 */
.static {
  position: static;
  /* top, right, bottom, left 无效 */
}

/* relative: 相对定位，相对于自身原始位置 */
.relative {
  position: relative;
  top: 10px; /* 相对原位置向下 10px */
  left: 20px; /* 相对原位置向右 20px */
  /* 仍占据原来的空间 */
}

/* absolute: 绝对定位，相对于最近的非 static 定位祖先元素 */
.absolute {
  position: absolute;
  top: 0;
  right: 0;
  /* 脱离文档流，不占据空间 */
}

/* fixed: 固定定位，相对于视口 */
.fixed {
  position: fixed;
  bottom: 20px;
  right: 20px;
  /* 脱离文档流，滚动时位置不变 */
}

/* sticky: 粘性定位，在特定条件下表现为 relative 或 fixed */
.sticky {
  position: sticky;
  top: 0; /* 距离顶部 0px 时开始粘性定位 */
  /* 在容器内滚动时"粘"在指定位置 */
}

/* z-index 层级控制 */
.layer1 { z-index: 1; }
.layer2 { z-index: 2; } /* 显示在 layer1 上方 */
.layer-negative { z-index: -1; } /* 显示在默认层下方 */

/* 居中定位 */
.center-absolute {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.center-fixed {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  width: 300px;
  height: 200px;
}
```

### 7. 浮动和清除浮动
**问题**: CSS 浮动的原理和清除浮动的方法

**答案**:
```css
/* 浮动元素 */
.float-left {
  float: left;
  width: 200px;
  height: 100px;
  background: red;
}

.float-right {
  float: right;
  width: 200px;
  height: 100px;
  background: blue;
}

/* 清除浮动的方法 */

/* 1. 使用 clear 属性 */
.clear {
  clear: both; /* left | right | both */
}

/* 2. 父元素设置 overflow */
.clearfix-overflow {
  overflow: hidden; /* 或 auto */
}

/* 3. 使用伪元素清除浮动 */
.clearfix::after {
  content: "";
  display: table;
  clear: both;
}

/* 4. 父元素也浮动 */
.parent-float {
  float: left;
  width: 100%;
}

/* 5. 使用 display: flow-root */
.clearfix-modern {
  display: flow-root;
}

/* 浮动布局示例 */
.container {
  width: 100%;
}

.sidebar {
  float: left;
  width: 25%;
  background: #f0f0f0;
}

.main {
  float: right;
  width: 75%;
  background: #fff;
}

.container::after {
  content: "";
  display: table;
  clear: both;
}
```

## 样式和效果

### 8. 背景和边框
**问题**: CSS 背景和边框的高级用法

**答案**:
```css
/* 背景属性 */
.background {
  /* 背景颜色 */
  background-color: #ff0000;
  
  /* 背景图片 */
  background-image: url('image.jpg');
  
  /* 背景重复 */
  background-repeat: no-repeat; /* repeat | repeat-x | repeat-y | no-repeat */
  
  /* 背景位置 */
  background-position: center top; /* 关键字 */
  background-position: 50% 0; /* 百分比 */
  background-position: 100px 50px; /* 像素值 */
  
  /* 背景大小 */
  background-size: cover; /* contain | cover | 100px 200px | 50% 100% */
  
  /* 背景固定 */
  background-attachment: fixed; /* scroll | fixed | local */
  
  /* 背景裁剪 */
  background-clip: padding-box; /* border-box | padding-box | content-box */
  
  /* 背景原点 */
  background-origin: padding-box; /* border-box | padding-box | content-box */
  
  /* 简写 */
  background: #fff url('bg.jpg') no-repeat center / cover fixed;
}

/* 多背景 */
.multiple-bg {
  background-image: 
    url('front.png'),
    url('back.jpg');
  background-position: 
    top right,
    center;
  background-repeat: 
    no-repeat,
    repeat;
}

/* 渐变背景 */
.gradient {
  /* 线性渐变 */
  background: linear-gradient(45deg, #ff0000, #0000ff);
  background: linear-gradient(to right, red, yellow, green);
  
  /* 径向渐变 */
  background: radial-gradient(circle, red, blue);
  background: radial-gradient(ellipse at center, red 0%, blue 100%);
  
  /* 重复渐变 */
  background: repeating-linear-gradient(
    45deg,
    red 0px,
    red 10px,
    blue 10px,
    blue 20px
  );
}

/* 边框属性 */
.border {
  /* 基本边框 */
  border: 2px solid #000;
  border-width: 1px 2px 3px 4px; /* 上 右 下 左 */
  border-style: solid; /* solid | dashed | dotted | double | groove | ridge | inset | outset */
  border-color: red;
  
  /* 单边边框 */
  border-top: 1px solid red;
  border-right: 2px dashed blue;
  
  /* 圆角 */
  border-radius: 10px;
  border-radius: 10px 20px; /* 左上右下 | 右上左下 */
  border-radius: 10px 20px 30px 40px; /* 左上 | 右上 | 右下 | 左下 */
  
  /* 椭圆圆角 */
  border-radius: 50px / 25px; /* 水平半径 / 垂直半径 */
  
  /* 边框图片 */
  border-image: url('border.png') 30 repeat;
  border-image-source: url('border.png');
  border-image-slice: 30; /* 切片大小 */
  border-image-repeat: repeat; /* stretch | repeat | round | space */
}

/* 阴影效果 */
.shadow {
  /* 盒阴影 */
  box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.3);
  box-shadow: 
    0 0 10px red,
    inset 0 0 10px blue; /* 多重阴影 */
  
  /* 文字阴影 */
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  text-shadow: 
    1px 1px 2px red,
    -1px -1px 2px blue; /* 多重文字阴影 */
}
```

### 9. 动画和过渡
**问题**: CSS 动画和过渡的实现方法

**答案**:
```css
/* CSS 过渡 */
.transition {
  width: 100px;
  height: 100px;
  background: red;
  
  /* 过渡属性 */
  transition-property: width, height, background-color;
  transition-duration: 0.3s; /* 持续时间 */
  transition-timing-function: ease; /* 时间函数 */
  transition-delay: 0.1s; /* 延迟 */
  
  /* 简写 */
  transition: all 0.3s ease 0.1s;
  transition: width 0.3s, height 0.5s; /* 多个属性 */
}

.transition:hover {
  width: 200px;
  height: 200px;
  background: blue;
}

/* 时间函数 */
.timing-functions {
  transition-timing-function: ease; /* 默认 */
  transition-timing-function: linear; /* 匀速 */
  transition-timing-function: ease-in; /* 加速 */
  transition-timing-function: ease-out; /* 减速 */
  transition-timing-function: ease-in-out; /* 先加速后减速 */
  transition-timing-function: cubic-bezier(0.1, 0.7, 1.0, 0.1); /* 自定义 */
}

/* CSS 动画 */
@keyframes slideIn {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.animation {
  /* 动画属性 */
  animation-name: slideIn; /* 动画名称 */
  animation-duration: 1s; /* 持续时间 */
  animation-timing-function: ease-in-out; /* 时间函数 */
  animation-delay: 0.5s; /* 延迟 */
  animation-iteration-count: infinite; /* 重复次数 */
  animation-direction: alternate; /* 方向 */
  animation-fill-mode: forwards; /* 填充模式 */
  animation-play-state: running; /* 播放状态 */
  
  /* 简写 */
  animation: slideIn 1s ease-in-out 0.5s infinite alternate forwards;
}

/* 常用动画效果 */
.fade-in {
  animation: fadeIn 0.5s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.bounce {
  animation: bounce 1s ease-in-out infinite;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-30px);
  }
  60% {
    transform: translateY(-15px);
  }
}

.spin {
  animation: spin 2s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
```

### 10. Transform 变换
**问题**: CSS Transform 的各种变换函数

**答案**:
```css
/* 2D 变换 */
.transform-2d {
  /* 平移 */
  transform: translate(50px, 100px); /* X轴, Y轴 */
  transform: translateX(50px); /* 仅X轴 */
  transform: translateY(100px); /* 仅Y轴 */
  
  /* 缩放 */
  transform: scale(1.5); /* 等比缩放 */
  transform: scale(2, 0.5); /* X轴2倍, Y轴0.5倍 */
  transform: scaleX(2); /* 仅X轴 */
  transform: scaleY(0.5); /* 仅Y轴 */
  
  /* 旋转 */
  transform: rotate(45deg); /* 顺时针45度 */
  transform: rotate(-90deg); /* 逆时针90度 */
  
  /* 倾斜 */
  transform: skew(30deg, 20deg); /* X轴, Y轴 */
  transform: skewX(30deg); /* 仅X轴 */
  transform: skewY(20deg); /* 仅Y轴 */
  
  /* 组合变换 */
  transform: translate(50px, 100px) rotate(45deg) scale(1.5);
}

/* 3D 变换 */
.transform-3d {
  /* 3D 平移 */
  transform: translate3d(50px, 100px, 25px); /* X, Y, Z轴 */
  transform: translateZ(25px); /* 仅Z轴 */
  
  /* 3D 缩放 */
  transform: scale3d(2, 1.5, 0.5); /* X, Y, Z轴 */
  transform: scaleZ(0.5); /* 仅Z轴 */
  
  /* 3D 旋转 */
  transform: rotate3d(1, 1, 0, 45deg); /* 绕向量旋转 */
  transform: rotateX(45deg); /* 绕X轴旋转 */
  transform: rotateY(45deg); /* 绕Y轴旋转 */
  transform: rotateZ(45deg); /* 绕Z轴旋转 */
  
  /* 透视 */
  transform: perspective(1000px) rotateY(45deg);
}

/* 变换原点 */
.transform-origin {
  transform-origin: center center; /* 默认中心点 */
  transform-origin: top left; /* 左上角 */
  transform-origin: 50% 50%; /* 百分比 */
  transform-origin: 100px 50px; /* 像素值 */
  transform-origin: center center 100px; /* 3D原点 */
}

/* 3D 变换上下文 */
.transform-style {
  transform-style: preserve-3d; /* 保持3D */
  transform-style: flat; /* 平面化 */
}

.perspective {
  perspective: 1000px; /* 透视距离 */
  perspective-origin: center center; /* 透视原点 */
}

/* 背面可见性 */
.backface-visibility {
  backface-visibility: visible; /* 默认可见 */
  backface-visibility: hidden; /* 隐藏背面 */
}

/* 实用示例 */
.flip-card {
  width: 300px;
  height: 200px;
  perspective: 1000px;
}

.flip-card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  text-align: center;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.flip-card:hover .flip-card-inner {
  transform: rotateY(180deg);
}

.flip-card-front, .flip-card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
}

.flip-card-back {
  transform: rotateY(180deg);
}
```

## 响应式设计

### 11. 媒体查询
**问题**: CSS 媒体查询的使用方法

**答案**:
```css
/* 基本媒体查询 */
@media screen and (max-width: 768px) {
  .container {
    width: 100%;
    padding: 10px;
  }
}

/* 多种条件 */
@media screen and (min-width: 768px) and (max-width: 1024px) {
  .container {
    width: 90%;
  }
}

/* 或条件 */
@media screen and (max-width: 480px), print {
  .hide-mobile {
    display: none;
  }
}

/* 否定条件 */
@media not screen and (max-width: 768px) {
  .desktop-only {
    display: block;
  }
}

/* 常用断点 */
/* 超小屏幕 */
@media (max-width: 575.98px) {
  .container { width: 100%; }
}

/* 小屏幕 */
@media (min-width: 576px) and (max-width: 767.98px) {
  .container { width: 540px; }
}

/* 中等屏幕 */
@media (min-width: 768px) and (max-width: 991.98px) {
  .container { width: 720px; }
}

/* 大屏幕 */
@media (min-width: 992px) and (max-width: 1199.98px) {
  .container { width: 960px; }
}

/* 超大屏幕 */
@media (min-width: 1200px) {
  .container { width: 1140px; }
}

/* 高分辨率屏幕 */
@media (-webkit-min-device-pixel-ratio: 2),
       (min-resolution: 192dpi) {
  .logo {
    background-image: url('logo@2x.png');
    background-size: 100px 50px;
  }
}

/* 横屏竖屏 */
@media (orientation: landscape) {
  .landscape-only { display: block; }
}

@media (orientation: portrait) {
  .portrait-only { display: block; }
}

/* 打印样式 */
@media print {
  .no-print { display: none; }
  .container { 
    width: 100%; 
    margin: 0;
    box-shadow: none;
  }
}

/* 暗色模式 */
@media (prefers-color-scheme: dark) {
  body {
    background-color: #121212;
    color: #ffffff;
  }
}

/* 用户偏好 */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 12. 响应式单位
**问题**: CSS 中的相对单位和响应式单位

**答案**:
```css
/* 绝对单位 */
.absolute-units {
  width: 300px; /* 像素 */
  height: 2in; /* 英寸 */
  margin: 1cm; /* 厘米 */
  padding: 10mm; /* 毫米 */
  border-width: 2pt; /* 点 */
  font-size: 12pc; /* 派卡 */
}

/* 相对单位 */
.relative-units {
  /* 相对于父元素字体大小 */
  font-size: 1.2em;
  margin: 2em;
  
  /* 相对于根元素字体大小 */
  padding: 1rem;
  font-size: 1.5rem;
  
  /* 相对于元素自身字体大小 */
  width: 10ch; /* 字符宽度 */
  height: 2ex; /* x高度 */
  
  /* 相对于视口 */
  width: 100vw; /* 视口宽度 */
  height: 100vh; /* 视口高度 */
  font-size: 4vmin; /* 视口最小尺寸 */
  padding: 2vmax; /* 视口最大尺寸 */
  
  /* 百分比 */
  width: 50%; /* 相对于父元素 */
  line-height: 150%; /* 相对于字体大小 */
}

/* 新的视口单位 */
.new-viewport-units {
  /* 小视口单位（排除动态UI） */
  height: 100svh; /* Small Viewport Height */
  width: 100svw; /* Small Viewport Width */
  
  /* 大视口单位（包括动态UI） */
  height: 100lvh; /* Large Viewport Height */
  width: 100lvw; /* Large Viewport Width */
  
  /* 动态视口单位 */
  height: 100dvh; /* Dynamic Viewport Height */
  width: 100dvw; /* Dynamic Viewport Width */
}

/* 容器查询单位 */
.container-query-units {
  /* 相对于容器 */
  width: 50cqw; /* Container Query Width */
  height: 30cqh; /* Container Query Height */
  font-size: 4cqi; /* Container Query Inline */
  padding: 2cqb; /* Container Query Block */
  margin: 1cqmin; /* Container Query Min */
  border-width: 1cqmax; /* Container Query Max */
}

/* 实用示例 */
.responsive-text {
  /* 基于视口的响应式字体 */
  font-size: calc(16px + 2vw);
  font-size: clamp(16px, 4vw, 32px); /* 最小值 期望值 最大值 */
}

.responsive-spacing {
  /* 响应式间距 */
  padding: clamp(1rem, 5vw, 3rem);
  margin: clamp(0.5rem, 2.5vw, 2rem);
}

.fluid-layout {
  /* 流式布局 */
  width: clamp(300px, 90vw, 1200px);
  max-width: 100%;
}

/* CSS 函数 */
.css-functions {
  /* calc 计算 */
  width: calc(100% - 20px);
  height: calc(100vh - 60px);
  font-size: calc(1rem + 0.5vw);
  
  /* min/max */
  width: min(90vw, 1200px);
  height: max(200px, 20vh);
  
  /* clamp */
  font-size: clamp(1rem, 2.5vw, 2rem);
  padding: clamp(1rem, 5%, 3rem);
}
```

## 高级特性

### 13. CSS 变量
**问题**: CSS 自定义属性（变量）的使用

**答案**:
```css
/* 全局变量定义 */
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --font-size-base: 16px;
  --font-size-lg: calc(var(--font-size-base) * 1.25);
  --spacing-unit: 8px;
  --spacing-sm: calc(var(--spacing-unit) * 2);
  --spacing-md: calc(var(--spacing-unit) * 3);
  --spacing-lg: calc(var(--spacing-unit) * 4);
  --border-radius: 4px;
  --box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  --transition: all 0.3s ease;
}

/* 暗色主题 */
[data-theme="dark"] {
  --primary-color: #0d6efd;
  --background-color: #121212;
  --text-color: #ffffff;
  --border-color: #333333;
}

/* 浅色主题 */
[data-theme="light"] {
  --background-color: #ffffff;
  --text-color: #333333;
  --border-color: #e0e0e0;
}

/* 使用变量 */
.button {
  background-color: var(--primary-color);
  color: white;
  font-size: var(--font-size-base);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--border-radius);
  border: 1px solid var(--primary-color);
  transition: var(--transition);
  box-shadow: var(--box-shadow);
}

.button:hover {
  background-color: var(--secondary-color, #6c757d); /* 备用值 */
}

/* 局部变量 */
.card {
  --card-padding: 20px;
  --card-background: #f8f9fa;
  
  padding: var(--card-padding);
  background-color: var(--card-background);
  border-radius: var(--border-radius);
}

.card .title {
  color: var(--primary-color);
  margin-bottom: var(--spacing-sm);
}

/* JavaScript 操作变量 */
/* 
document.documentElement.style.setProperty('--primary-color', '#ff0000');
const primaryColor = getComputedStyle(document.documentElement)
  .getPropertyValue('--primary-color');
*/

/* 响应式变量 */
.responsive-component {
  --columns: 1;
  --gap: var(--spacing-sm);
}

@media (min-width: 768px) {
  .responsive-component {
    --columns: 2;
    --gap: var(--spacing-md);
  }
}

@media (min-width: 1024px) {
  .responsive-component {
    --columns: 3;
    --gap: var(--spacing-lg);
  }
}

.grid {
  display: grid;
  grid-template-columns: repeat(var(--columns), 1fr);
  gap: var(--gap);
}

/* 变量计算 */
.progressive-enhancement {
  --base-size: 16px;
  --scale-factor: 1.2;
  --level-1: calc(var(--base-size) * var(--scale-factor));
  --level-2: calc(var(--level-1) * var(--scale-factor));
  --level-3: calc(var(--level-2) * var(--scale-factor));
}

.h1 { font-size: var(--level-3); }
.h2 { font-size: var(--level-2); }
.h3 { font-size: var(--level-1); }
```

### 14. 预处理器特性
**问题**: SCSS/SASS 的主要特性和使用

**答案**:
```scss
// 变量
$primary-color: #007bff;
$secondary-color: #6c757d;
$font-size-base: 16px;
$border-radius: 4px;

// 嵌套
.navigation {
  background: $primary-color;
  
  ul {
    list-style: none;
    padding: 0;
    
    li {
      display: inline-block;
      
      a {
        color: white;
        text-decoration: none;
        padding: 10px 15px;
        
        &:hover {
          background: rgba(white, 0.1);
        }
        
        &.active {
          background: rgba(white, 0.2);
        }
      }
    }
  }
}

// 混合器 (Mixins)
@mixin button-style($bg-color, $text-color: white) {
  background-color: $bg-color;
  color: $text-color;
  border: none;
  padding: 10px 20px;
  border-radius: $border-radius;
  cursor: pointer;
  transition: all 0.3s ease;
  
  &:hover {
    background-color: darken($bg-color, 10%);
  }
}

@mixin responsive($breakpoint) {
  @if $breakpoint == mobile {
    @media (max-width: 767px) { @content; }
  }
  @if $breakpoint == tablet {
    @media (min-width: 768px) and (max-width: 1023px) { @content; }
  }
  @if $breakpoint == desktop {
    @media (min-width: 1024px) { @content; }
  }
}

// 使用混合器
.primary-button {
  @include button-style($primary-color);
}

.secondary-button {
  @include button-style($secondary-color);
}

// 继承
.message {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.success {
  @extend .message;
  border-color: green;
  color: green;
}

.error {
  @extend .message;
  border-color: red;
  color: red;
}

// 函数
@function calculate-rem($size) {
  @return $size / $font-size-base * 1rem;
}

@function theme-color($key) {
  @return map-get($theme-colors, $key);
}

// 映射
$theme-colors: (
  primary: #007bff,
  secondary: #6c757d,
  success: #28a745,
  danger: #dc3545,
  warning: #ffc107,
  info: #17a2b8
);

// 循环
@each $name, $color in $theme-colors {
  .btn-#{$name} {
    @include button-style($color);
  }
  
  .text-#{$name} {
    color: $color !important;
  }
}

// 条件语句
@mixin triangle($direction, $size, $color) {
  width: 0;
  height: 0;
  
  @if $direction == up {
    border-left: $size solid transparent;
    border-right: $size solid transparent;
    border-bottom: $size solid $color;
  } @else if $direction == down {
    border-left: $size solid transparent;
    border-right: $size solid transparent;
    border-top: $size solid $color;
  } @else if $direction == left {
    border-top: $size solid transparent;
    border-bottom: $size solid transparent;
    border-right: $size solid $color;
  } @else if $direction == right {
    border-top: $size solid transparent;
    border-bottom: $size solid transparent;
    border-left: $size solid $color;
  }
}

// 响应式布局
.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 15px;
  
  @include responsive(mobile) {
    padding: 0 10px;
  }
  
  @include responsive(tablet) {
    max-width: 900px;
  }
  
  @include responsive(desktop) {
    max-width: 1200px;
  }
}

// 颜色函数
.color-functions {
  background: lighten($primary-color, 20%);
  border: 1px solid darken($primary-color, 15%);
  color: complement($primary-color);
  box-shadow: 0 2px 4px rgba($primary-color, 0.3);
}

// 数学运算
$grid-columns: 12;
$grid-gutter: 30px;

@for $i from 1 through $grid-columns {
  .col-#{$i} {
    width: percentage($i / $grid-columns);
  }
}

.container {
  max-width: 1200px - $grid-gutter;
  margin: 0 auto;
  padding: 0 $grid-gutter / 2;
}

// 导入和模块化
@import 'variables';
@import 'mixins';
@import 'base';
@import 'components/buttons';
@import 'components/forms';
@import 'layout/grid';
@import 'pages/home';
```

### 15. 现代 CSS 特性
**问题**: 最新的 CSS 特性有哪些？

**答案**:
```css
/* 容器查询 */
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 300px) {
  .card {
    display: flex;
    flex-direction: row;
  }
}

@container (max-width: 250px) {
  .card h2 {
    font-size: 1rem;
  }
}

/* CSS 嵌套 */
.card {
  background: white;
  border-radius: 8px;
  padding: 1rem;
  
  & .title {
    font-size: 1.5rem;
    margin-bottom: 0.5rem;
    
    &:hover {
      color: blue;
    }
  }
  
  & .content {
    line-height: 1.6;
    
    & p {
      margin-bottom: 1rem;
      
      &:last-child {
        margin-bottom: 0;
      }
    }
  }
}

/* :has() 选择器 */
.card:has(.image) {
  display: grid;
  grid-template-columns: 200px 1fr;
  gap: 1rem;
}

.form:has(input:invalid) {
  border: 2px solid red;
}

.article:has(h1) .sidebar {
  display: block;
}

/* CSS 层叠层 */
@layer reset, base, components, utilities;

@layer reset {
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
}

@layer base {
  body {
    font-family: system-ui;
    line-height: 1.6;
  }
}

@layer components {
  .button {
    padding: 0.5rem 1rem;
    border-radius: 4px;
    border: none;
  }
}

@layer utilities {
  .sr-only {
    position: absolute !important;
    width: 1px !important;
    height: 1px !important;
    overflow: hidden !important;
  }
}

/* 子网格 */
.grid-container {
  display: grid;
  grid-template-columns: 1fr 3fr 1fr;
  gap: 1rem;
}

.grid-item {
  display: grid;
  grid-template-columns: subgrid; /* 继承父网格的列定义 */
  gap: 0.5rem;
}

/* :is() 和 :where() */
/* :is() 有特异性 */
:is(h1, h2, h3) {
  margin-bottom: 1rem;
}

:is(.error, .warning) .message {
  font-weight: bold;
}

/* :where() 无特异性 */
:where(h1, h2, h3) {
  margin-bottom: 1rem; /* 特异性为 0，容易被覆盖 */
}

/* 逻辑属性 */
.logical-properties {
  /* 块方向（垂直） */
  margin-block-start: 1rem; /* 块开始边距 */
  margin-block-end: 1rem; /* 块结束边距 */
  margin-block: 1rem 2rem; /* 块方向边距 */
  
  /* 行方向（水平） */
  margin-inline-start: 1rem; /* 行开始边距 */
  margin-inline-end: 1rem; /* 行结束边距 */
  margin-inline: 1rem 2rem; /* 行方向边距 */
  
  /* 尺寸 */
  inline-size: 300px; /* 行方向尺寸（宽度） */
  block-size: 200px; /* 块方向尺寸（高度） */
  
  /* 定位 */
  inset-block-start: 10px; /* 块开始位置 */
  inset-inline-end: 20px; /* 行结束位置 */
}

/* 滚动驱动动画 */
@keyframes slideProgress {
  to {
    transform: translateX(100%);
  }
}

.progress-bar {
  animation: slideProgress linear;
  animation-timeline: scroll(root); /* 滚动驱动 */
}

/* 视图过渡 */
.fade-transition {
  view-transition-name: fade;
}

::view-transition-old(fade) {
  animation: fadeOut 0.3s ease-out;
}

::view-transition-new(fade) {
  animation: fadeIn 0.3s ease-in;
}

/* 颜色混合 */
.color-mixing {
  background: color-mix(in srgb, red 50%, blue);
  background: color-mix(in oklab, red 30%, transparent);
}

/* 相对颜色语法 */
.relative-colors {
  --base-color: #007bff;
  background: oklch(from var(--base-color) calc(l * 0.8) c h);
  border: 1px solid hsl(from var(--base-color) h s calc(l * 0.7));
}

/* 锚点定位 */
.anchor {
  anchor-name: --my-anchor;
}

.tooltip {
  position: absolute;
  position-anchor: --my-anchor;
  top: anchor(bottom);
  left: anchor(center);
}
```