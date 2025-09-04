# CSS 布局系统

CSS 布局系统是现代前端开发的核心，提供了多种强大的布局方案。本文将全面介绍各种CSS布局技术及其应用场景。

## 传统布局方案

### 1. 浮动布局 (Float Layout)

```css
/* 基本浮动布局 */
.container {
  width: 100%;
  overflow: hidden; /* 清除浮动 */
}

.column {
  float: left;
  width: 33.33%;
  box-sizing: border-box;
  padding: 20px;
}

/* 清除浮动技巧 */
.clearfix::after {
  content: "";
  display: table;
  clear: both;
}

/* 媒体查询适配 */
@media (max-width: 768px) {
  .column {
    float: none;
    width: 100%;
  }
}
```

### 2. 定位布局 (Position Layout)

```css
/* 相对定位 */
.relative-box {
  position: relative;
  top: 10px;
  left: 20px;
}

/* 绝对定位 */
.absolute-box {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

/* 固定定位 */
.fixed-header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 1000;
}

/* 粘性定位 */
.sticky-nav {
  position: sticky;
  top: 0;
  z-index: 100;
}
```

## 现代布局方案

### 1. Flexbox 弹性布局

#### 容器属性
```css
.flex-container {
  display: flex;
  
  /* 主轴方向 */
  flex-direction: row; /* row | row-reverse | column | column-reverse */
  
  /* 换行控制 */
  flex-wrap: nowrap; /* nowrap | wrap | wrap-reverse */
  
  /* 主轴对齐 */
  justify-content: flex-start; /* flex-start | flex-end | center | space-between | space-around | space-evenly */
  
  /* 交叉轴对齐 */
  align-items: stretch; /* stretch | flex-start | flex-end | center | baseline */
  
  /* 多行对齐 */
  align-content: stretch; /* flex-start | flex-end | center | space-between | space-around | stretch */
  
  /* 间距 */
  gap: 10px;
  row-gap: 15px;
  column-gap: 20px;
}
```

#### 项目属性
```css
.flex-item {
  /* 弹性比例 */
  flex: 0 1 auto; /* flex-grow | flex-shrink | flex-basis */
  
  /* 单独对齐 */
  align-self: auto; /* auto | flex-start | flex-end | center | baseline | stretch */
  
  /* 顺序控制 */
  order: 0;
  
  /* 最小最大尺寸 */
  min-width: 100px;
  max-width: 300px;
}
```

#### 实用布局模式
```css
/* 水平居中 */
.center-horizontal {
  display: flex;
  justify-content: center;
}

/* 垂直居中 */
.center-vertical {
  display: flex;
  align-items: center;
}

/* 完全居中 */
.center-both {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 等分布局 */
.equal-columns {
  display: flex;
}

.equal-columns > * {
  flex: 1;
}

/* 圣杯布局 */
.holy-grail {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.holy-grail main {
  flex: 1;
  display: flex;
}

.holy-grail .sidebar {
  width: 250px;
}

.holy-grail .content {
  flex: 1;
}
```

### 2. CSS Grid 网格布局

#### 容器属性
```css
.grid-container {
  display: grid;
  
  /* 列定义 */
  grid-template-columns: 100px 1fr 2fr;
  grid-template-columns: repeat(3, 1fr);
  grid-template-columns: minmax(100px, 1fr) 2fr;
  grid-template-columns: [col-start] 1fr [col-2] 1fr [col-end];
  
  /* 行定义 */
  grid-template-rows: auto 1fr auto;
  grid-template-rows: repeat(2, minmax(100px, auto));
  
  /* 区域定义 */
  grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";
  
  /* 间距 */
  gap: 20px;
  grid-gap: 20px; /* 旧语法 */
  row-gap: 15px;
  column-gap: 25px;
  
  /* 对齐方式 */
  justify-items: stretch; /* start | end | center | stretch */
  align-items: stretch;
  justify-content: start; /* start | end | center | stretch | space-around | space-between | space-evenly */
  align-content: start;
  
  /* 自动布局 */
  grid-auto-flow: row; /* row | column | row dense | column dense */
  grid-auto-columns: auto;
  grid-auto-rows: minmax(100px, auto);
}
```

#### 项目属性
```css
.grid-item {
  /* 位置指定 */
  grid-column: 1 / 3; /* 开始线 / 结束线 */
  grid-column: 1 / span 2; /* 开始线 / 跨越数量 */
  grid-column: col-start / col-end; /* 命名线 */
  
  grid-row: 2 / 4;
  grid-row: 2 / span 2;
  
  /* 区域指定 */
  grid-area: header; /* 命名区域 */
  grid-area: 1 / 1 / 3 / 3; /* row-start / col-start / row-end / col-end */
  
  /* 单独对齐 */
  justify-self: stretch; /* start | end | center | stretch */
  align-self: stretch;
}
```

#### 实用网格模式
```css
/* 12列网格系统 */
.grid-12 {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 20px;
}

.col-1 { grid-column: span 1; }
.col-2 { grid-column: span 2; }
.col-3 { grid-column: span 3; }
.col-4 { grid-column: span 4; }
.col-6 { grid-column: span 6; }
.col-12 { grid-column: span 12; }

/* 响应式网格 */
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

/* 杂志布局 */
.magazine-layout {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: masonry; /* 实验性特性 */
  gap: 20px;
}

.magazine-layout .featured {
  grid-column: span 2;
  grid-row: span 2;
}

/* 表单网格 */
.form-grid {
  display: grid;
  grid-template-columns: [labels] auto [controls] 1fr [messages] auto;
  gap: 10px;
  align-items: center;
}

.form-grid label {
  grid-column: labels;
}

.form-grid input {
  grid-column: controls;
}

.form-grid .error {
  grid-column: messages;
  color: red;
}
```

## 响应式布局策略

### 1. 媒体查询断点
```css
/* 移动设备优先 */
.container {
  width: 100%;
  padding: 10px;
}

/* 小屏设备 (≥576px) */
@media (min-width: 576px) {
  .container {
    max-width: 540px;
    margin: 0 auto;
  }
}

/* 中等设备 (≥768px) */
@media (min-width: 768px) {
  .container {
    max-width: 720px;
  }
  
  .grid-layout {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* 大设备 (≥992px) */
@media (min-width: 992px) {
  .container {
    max-width: 960px;
  }
  
  .grid-layout {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* 超大设备 (≥1200px) */
@media (min-width: 1200px) {
  .container {
    max-width: 1140px;
  }
  
  .grid-layout {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

### 2. 响应式实用类
```css
/* 显示控制 */
.hidden { display: none; }
.block { display: block; }
.flex { display: flex; }
.grid { display: grid; }

/* 响应式显示 */
@media (max-width: 767px) {
  .hidden-mobile { display: none; }
  .block-mobile { display: block; }
}

@media (min-width: 768px) {
  .hidden-tablet { display: none; }
  .flex-tablet { display: flex; }
}

/* 间距响应式 */
.spacing {
  padding: 1rem;
}

@media (min-width: 768px) {
  .spacing {
    padding: 2rem;
  }
}
```

## 布局性能优化

### 1. 渲染性能
```css
/* 减少重排重绘 */
.optimized-layout {
  /* 使用transform代替top/left */
  transform: translateX(100px);
  
  /* 使用opacity代替visibility */
  opacity: 0;
  
  /* 避免频繁修改布局属性 */
  will-change: transform;
}

/* 硬件加速 */
.gpu-accelerated {
  transform: translateZ(0);
  backface-visibility: hidden;
}
```

### 2. 内存优化
```css
/* 避免过多嵌套 */
.flat-structure {
  /* 减少选择器复杂度 */
}

/* 使用CSS变量 */
:root {
  --spacing: 8px;
  --breakpoint-md: 768px;
}

.layout {
  padding: var(--spacing);
}

@media (min-width: var(--breakpoint-md)) {
  .layout {
    padding: calc(var(--spacing) * 2);
  }
}
```

## 浏览器兼容性方案

### 1. 渐进增强
```css
/* 基础布局 */
.layout {
  display: block;
}

/* Flexbox增强 */
@supports (display: flex) {
  .layout {
    display: flex;
  }
}

/* Grid增强 */
@supports (display: grid) {
  .layout {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
```

### 2. 回退方案
```css
.grid-layout {
  /* 浮动回退 */
  float: left;
  width: 50%;
}

@supports (display: grid) {
  .grid-layout {
    float: none;
    width: auto;
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
```

## 实用布局示例

### 1. 三栏布局
```css
/* Flexbox实现 */
.three-columns {
  display: flex;
  gap: 20px;
}

.sidebar {
  width: 250px;
  flex-shrink: 0;
}

.main {
  flex: 1;
}

/* Grid实现 */
.three-columns-grid {
  display: grid;
  grid-template-columns: 250px 1fr 200px;
  gap: 20px;
}
```

### 2. 卡片网格
```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.card {
  display: flex;
  flex-direction: column;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

.card-image {
  height: 200px;
  object-fit: cover;
}

.card-content {
  padding: 20px;
  flex: 1;
}

.card-footer {
  padding: 15px 20px;
  border-top: 1px solid #eee;
}
```

### 3. 粘性侧边栏
```css
.layout {
  display: grid;
  grid-template-columns: 300px 1fr;
  min-height: 100vh;
}

.sidebar {
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
}

.main {
  padding: 20px;
}
```

## 布局调试技巧

### 1. 开发工具辅助
```css
/* 调试边框 */
.debug * {
  outline: 1px solid #ff0000 !important;
}

/* 网格调试 */
.grid-debug {
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.grid-debug > * {
  background-color: rgba(0, 100, 255, 0.1);
  border: 1px dashed #0064ff;
  min-height: 50px;
}
```

### 2. 响应式测试
```css
/* 断点指示器 */
body::before {
  content: "Mobile";
  position: fixed;
  top: 10px;
  right: 10px;
  background: red;
  color: white;
  padding: 5px;
  z-index: 9999;
}

@media (min-width: 768px) {
  body::before {
    content: "Tablet";
    background: blue;
  }
}

@media (min-width: 1024px) {
  body::before {
    content: "Desktop";
    background: green;
  }
}
```
