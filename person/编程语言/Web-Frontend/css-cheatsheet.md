# CSS 速查手册

CSS（层叠样式表）是前端开发的核心技术之一，用于定义网页的样式和布局。本手册提供CSS核心概念、常用属性和最佳实践的快速参考。

## 选择器与优先级

### 1. 基础选择器

```css
/* 元素选择器 */
div { color: blue; }

/* 类选择器 */
.class-name { color: red; }

/* ID选择器 */
#element-id { color: green; }

/* 通配符选择器 */
* { margin: 0; padding: 0; }

/* 属性选择器 */
[type="text"] { border: 1px solid #ccc; }
[href^="https"] { color: purple; } /* 以https开头 */
[href$=".pdf"]::after { content: " (PDF)"; } /* 以.pdf结尾 */
```

### 2. 组合选择器

```css
/* 后代选择器 */
div p { margin: 10px; }

/* 子选择器 */
div > p { color: red; }

/* 相邻兄弟选择器 */
h1 + p { font-size: 1.2em; }

/* 通用兄弟选择器 */
h1 ~ p { color: gray; }
```

### 3. 伪类与伪元素

```css
/* 伪类 */
a:hover { color: #ff0000; }
input:focus { border-color: blue; }
li:nth-child(odd) { background: #f0f0f0; }
tr:nth-of-type(2n) { background: #fafafa; }

/* 伪元素 */
p::first-line { font-weight: bold; }
p::first-letter { font-size: 2em; }
::selection { background: yellow; }
```

### 4. 优先级计算

优先级由A-B-C-D四个值组成：
- A: 内联样式（1或0）
- B: ID选择器数量
- C: 类、属性、伪类选择器数量
- D: 元素、伪元素选择器数量

```css
/* 优先级: 0,1,1,1 = 0111 */
#header .nav li { color: red; }

/* 优先级: 0,0,2,1 = 0021 */
.nav .item { color: blue; }

/* 优先级: 0,0,0,1 = 0001 */
li { color: green; }
```

## 盒模型与布局

### 1. 盒模型属性

```css
.box {
  /* 内容框 */
  width: 300px;
  height: 200px;
  
  /* 内边距 */
  padding: 20px;
  padding-top: 10px;
  padding-right: 15px;
  padding-bottom: 10px;
  padding-left: 15px;
  
  /* 边框 */
  border: 2px solid #333;
  border-radius: 8px;
  
  /* 外边距 */
  margin: 10px;
  margin: 10px 20px; /* 上下10px，左右20px */
  margin: 10px 20px 15px 5px; /* 上右下左 */
}

/* box-sizing 控制盒模型计算 */
.box-content { box-sizing: content-box; } /* 默认 */
.box-border { box-sizing: border-box; } /* 推荐使用 */
```

### 2. 显示模式

```css
/* 块级元素 */
.block { display: block; }

/* 行内元素 */
.inline { display: inline; }

/* 行内块元素 */
.inline-block { 
  display: inline-block;
  vertical-align: middle; /* 垂直对齐 */
}

/* Flex布局 */
.flex-container {
  display: flex;
  flex-direction: row; /* row | row-reverse | column | column-reverse */
  justify-content: center; /* 主轴对齐 */
  align-items: center; /* 交叉轴对齐 */
  flex-wrap: wrap; /* 换行 */
  gap: 10px; /* 项目间距 */
}

.flex-item {
  flex: 1; /* 弹性比例 */
  align-self: stretch; /* 单个项目对齐 */
}

/* Grid布局 */
.grid-container {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr; /* 列定义 */
  grid-template-rows: auto; /* 行定义 */
  grid-gap: 20px;
  grid-template-areas: 
    "header header header"
    "sidebar main aside"
    "footer footer footer";
}

.grid-item {
  grid-area: header; /* 指定区域 */
  grid-column: 1 / 3; /* 列范围 */
  grid-row: 1; /* 行位置 */
}
```

## 常用属性速查

### 1. 文本与字体

```css
.text {
  /* 字体 */
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  font-weight: 400; /* normal | bold | 100-900 */
  font-style: italic;
  line-height: 1.5;
  
  /* 文本 */
  color: #333;
  text-align: center; /* left | right | center | justify */
  text-decoration: none; /* underline | overline | line-through */
  text-transform: uppercase; /* lowercase | capitalize */
  letter-spacing: 1px;
  word-spacing: 2px;
}
```

### 2. 背景与颜色

```css
.background {
  /* 颜色 */
  background-color: #ffffff;
  background-image: url('image.jpg');
  background-repeat: no-repeat; /* repeat | repeat-x | repeat-y */
  background-position: center center;
  background-size: cover; /* contain | 100% 100% */
  background-attachment: fixed; /* scroll */
  
  /* 渐变 */
  background: linear-gradient(45deg, #ff0000, #0000ff);
  background: radial-gradient(circle, #ff0000, #0000ff);
  
  /* 多重背景 */
  background: 
    url('image1.jpg') center/cover no-repeat,
    url('image2.png') right bottom/100px auto no-repeat;
}
```

### 3. 定位与浮动

```css
.position {
  /* 定位 */
  position: static; /* 默认 */
  position: relative; /* 相对定位 */
  position: absolute; /* 绝对定位 */
  position: fixed; /* 固定定位 */
  position: sticky; /* 粘性定位 */
  
  /* 定位偏移 */
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  
  /* z轴控制 */
  z-index: 10;
}

.float {
  /* 浮动 */
  float: left; /* left | right | none */
  
  /* 清除浮动 */
  clear: both; /* left | right | both */
}

/* 清除浮动技巧 */
.clearfix::after {
  content: "";
  display: table;
  clear: both;
}
```

### 4. 动画与过渡

```css
.transition {
  /* 过渡 */
  transition: all 0.3s ease-in-out;
  transition-property: opacity, transform;
  transition-duration: 0.5s;
  transition-timing-function: ease; /* linear | ease-in | ease-out */
  transition-delay: 0.1s;
}

.animation {
  /* 动画 */
  animation: slideIn 1s ease-in-out;
  animation-name: slideIn;
  animation-duration: 1s;
  animation-timing-function: ease;
  animation-delay: 0s;
  animation-iteration-count: infinite; /* 1 | infinite */
  animation-direction: alternate; /* normal | reverse | alternate */
  animation-fill-mode: forwards; /* none | forwards | backwards | both */
}

@keyframes slideIn {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}
```

## 响应式设计

### 1. 媒体查询

```css
/* 移动设备优先 */
.container {
  width: 100%;
  padding: 10px;
}

/* 平板设备 */
@media (min-width: 768px) {
  .container {
    width: 750px;
    margin: 0 auto;
  }
}

/* 桌面设备 */
@media (min-width: 992px) {
  .container {
    width: 970px;
  }
}

/* 大屏设备 */
@media (min-width: 1200px) {
  .container {
    width: 1170px;
  }
}

/* 特定设备特性 */
@media (hover: hover) {
  /* 支持悬停的设备 */
  .button:hover {
    background-color: #007bff;
  }
}

@media (prefers-color-scheme: dark) {
  /* 暗色模式 */
  body {
    background-color: #1a1a1a;
    color: #ffffff;
  }
}
```

### 2. 响应式单位

```css
.container {
  /* 相对单位 */
  font-size: 1rem; /* 相对于根元素 */
  font-size: 1em; /* 相对于父元素 */
  
  /* 视口单位 */
  width: 100vw; /* 视口宽度 */
  height: 100vh; /* 视口高度 */
  font-size: 2vmin; /* 视口较小尺寸的2% */
  
  /* 百分比 */
  width: 50%; /* 父元素宽度的50% */
  
  /* 函数 */
  width: calc(100% - 20px); /* 计算值 */
  width: min(100%, 1200px); /* 取较小值 */
  width: max(300px, 50%); /* 取较大值 */
  width: clamp(300px, 50%, 800px); /* 范围限制 */
}
```

## 实用技巧与最佳实践

### 1. CSS变量

```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --font-size: 16px;
  --spacing: 8px;
}

.element {
  color: var(--primary-color);
  font-size: var(--font-size);
  margin: calc(var(--spacing) * 2);
}

@media (prefers-color-scheme: dark) {
  :root {
    --primary-color: #0d6efd;
  }
}
```

### 2. 性能优化

```css
/* 避免过度使用昂贵属性 */
.expensive {
  /* 慎用这些属性 */
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
  border-radius: 10px;
  filter: blur(5px);
  transform: translateZ(0); /* 触发GPU加速 */
}

/* 使用will-change提示浏览器 */
.animate {
  will-change: transform, opacity;
}

/* 减少重绘和回流 */
.optimized {
  /* 批量修改样式 */
  transform: translateX(100px) scale(1.2);
}
```

### 3. 浏览器前缀

```css
.example {
  /* 标准属性 */
  display: flex;
  
  /* 浏览器前缀（根据需要） */
  display: -webkit-flex;
  display: -ms-flexbox;
  
  /* 使用PostCSS等工具自动添加 */
}
```

## 常见问题解决方案

### 1. 垂直居中

```css
/* Flex方案 */
.parent {
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Grid方案 */
.parent {
  display: grid;
  place-items: center;
}

/* 传统方案 */
.parent {
  position: relative;
}

.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

### 2. 等高列

```css
/* Flex方案 */
.container {
  display: flex;
}

.column {
  flex: 1;
}

/* Grid方案 */
.container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  align-items: stretch;
}
```

### 3. 隐藏元素

```css
.hidden {
  /* 完全隐藏且不占空间 */
  display: none;
}

.invisible {
  /* 隐藏但占空间 */
  visibility: hidden;
}

.opacity-zero {
  /* 透明但可交互 */
  opacity: 0;
  pointer-events: none; /* 禁止交互 */
}

.sr-only {
  /* 屏幕阅读器可见 */
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

## 浏览器兼容性参考

| 特性 | Chrome | Firefox | Safari | Edge |
|------|--------|---------|--------|------|
| CSS Grid | 57+ | 52+ | 10.1+ | 16+ |
| Flexbox | 29+ | 28+ | 9+ | 12+ |
| CSS Variables | 49+ | 31+ | 9.1+ | 15+ |
| gap (Flex/Grid) | 84+ | 75+ | 14.1+ | 84+ |
| backdrop-filter | 76+ | 103+ | 9+ | 79+ |

---