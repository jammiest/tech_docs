# CSS 动画与过渡

CSS 动画与过渡是现代前端开发中创建流畅用户体验的关键技术。本文将深入探讨 CSS 动画的各种技术、性能优化和最佳实践。

## 基础概念

### 1. CSS 过渡 (Transitions)

```css
/* 基本过渡 */
.element {
  transition: all 0.3s ease-in-out;
}

.element:hover {
  transform: scale(1.1);
  opacity: 0.8;
}

/* 多属性过渡 */
.multi-transition {
  transition: 
    opacity 0.3s ease,
    transform 0.5s cubic-bezier(0.4, 0, 0.2, 1),
    background-color 0.2s linear;
}

/* 具体属性控制 */
.detailed-transition {
  transition-property: transform, opacity;
  transition-duration: 0.3s, 0.5s;
  transition-timing-function: ease-out, linear;
  transition-delay: 0s, 0.1s;
}
```

### 2. CSS 动画 (Animations)

```css
/* 关键帧定义 */
@keyframes slideIn {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  50% {
    transform: translateX(-50%);
    opacity: 0.5;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

/* 动画应用 */
.animated-element {
  animation: slideIn 1s ease-in-out;
}

/* 详细属性控制 */
.detailed-animation {
  animation-name: slideIn;
  animation-duration: 1s;
  animation-timing-function: ease-in-out;
  animation-delay: 0.5s;
  animation-iteration-count: infinite;
  animation-direction: alternate;
  animation-fill-mode: both;
  animation-play-state: running;
}
```

## 动画属性详解

### 1. 变换 (Transform)

```css
.transform-examples {
  /* 2D 变换 */
  transform: translate(100px, 50px);
  transform: rotate(45deg);
  transform: scale(1.5);
  transform: skew(30deg, 20deg);
  
  /* 3D 变换 */
  transform: translate3d(100px, 50px, 0);
  transform: rotate3d(1, 1, 1, 45deg);
  transform: scale3d(1.5, 1.5, 1.5);
  
  /* 变换原点 */
  transform-origin: center center;
  transform-origin: 0 0;
  transform-origin: 100% 100%;
  
  /* 变换组合 */
  transform: translateX(100px) rotate(45deg) scale(1.2);
}
```

### 2. 过渡函数 (Timing Functions)

```css
.timing-examples {
  /* 预定义函数 */
  transition-timing-function: ease;
  transition-timing-function: ease-in;
  transition-timing-function: ease-out;
  transition-timing-function: ease-in-out;
  transition-timing-function: linear;
  transition-timing-function: step-start;
  transition-timing-function: step-end;
  transition-timing-function: steps(4, jump-start);
  
  /* 贝塞尔曲线 */
  transition-timing-function: cubic-bezier(0.1, 0.7, 1.0, 0.1);
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.27, 1.55);
  
  /* 阶梯函数 */
  transition-timing-function: steps(5);
  transition-timing-function: steps(5, jump-start);
  transition-timing-function: steps(5, jump-end);
  transition-timing-function: steps(5, jump-none);
  transition-timing-function: steps(5, jump-both);
}
```

### 3. 动画控制属性

```css
.animation-control {
  /* 迭代次数 */
  animation-iteration-count: 1;
  animation-iteration-count: infinite;
  animation-iteration-count: 2.5;
  
  /* 播放方向 */
  animation-direction: normal;
  animation-direction: reverse;
  animation-direction: alternate;
  animation-direction: alternate-reverse;
  
  /* 填充模式 */
  animation-fill-mode: none;
  animation-fill-mode: forwards;
  animation-fill-mode: backwards;
  animation-fill-mode: both;
  
  /* 播放状态 */
  animation-play-state: running;
  animation-play-state: paused;
}
```

## 实用动画模式

### 1. 加载动画

```css
/* 旋转加载器 */
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loader {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

/* 骨架屏动画 */
@keyframes shimmer {
  0% { background-position: -200px 0; }
  100% { background-position: calc(200px + 100%) 0; }
}

.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200px 100%;
  animation: shimmer 1.5s infinite;
}
```

### 2. 微交互动画

```css
/* 按钮点击效果 */
@keyframes buttonPress {
  0% { transform: scale(1); }
  50% { transform: scale(0.95); }
  100% { transform: scale(1); }
}

.button:active {
  animation: buttonPress 0.2s ease;
}

/* 悬停效果 */
.card {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
}

/* 聚焦效果 */
.input-field {
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.input-field:focus {
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25);
}
```

### 3. 页面过渡动画

```css
/* 页面进入动画 */
@keyframes pageEnter {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.page-content {
  animation: pageEnter 0.6s ease-out;
}

/* 淡入淡出 */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to { opacity: 0; }
}

.fade-in {
  animation: fadeIn 0.5s ease-in;
}

.fade-out {
  animation: fadeOut 0.5s ease-out;
}
```

## 高级动画技术

### 1. 关键帧控制

```css
/* 复杂关键帧 */
@keyframes complexAnimation {
  0% {
    transform: translateX(0) rotate(0deg);
    opacity: 1;
    background-color: red;
  }
  25% {
    transform: translateX(100px) rotate(90deg);
    opacity: 0.8;
    background-color: blue;
  }
  50% {
    transform: translateX(200px) rotate(180deg);
    opacity: 0.6;
    background-color: green;
  }
  75% {
    transform: translateX(100px) rotate(270deg);
    opacity: 0.8;
    background-color: yellow;
  }
  100% {
    transform: translateX(0) rotate(360deg);
    opacity: 1;
    background-color: red;
  }
}

/* 使用百分比和from/to */
@keyframes slide {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

/* 使用多个相同百分比 */
@keyframes bounce {
  0%, 20%, 53%, 80%, 100% {
    transform: translate3d(0, 0, 0);
  }
  40%, 43% {
    transform: translate3d(0, -30px, 0);
  }
  70% {
    transform: translate3d(0, -15px, 0);
  }
  90% {
    transform: translate3d(0, -4px, 0);
  }
}
```

### 2. 动画组合

```css
/* 多个动画同时运行 */
.multiple-animations {
  animation: 
    slideIn 1s ease-out,
    fadeIn 0.8s ease-in,
    bounce 2s infinite;
}

/* 顺序动画 */
.sequence-animations {
  animation: 
    slideIn 0.5s ease-out forwards,
    fadeIn 0.3s ease-in 0.5s forwards;
}

/* 链式动画控制 */
.chain-animation {
  animation: firstAnimation 0.5s ease-out;
}

.chain-animation:hover {
  animation: secondAnimation 0.3s ease-in;
}
```

### 3. 3D 动画

```css
/* 3D 变换动画 */
@keyframes flip3d {
  0% {
    transform: perspective(1000px) rotateY(0deg);
  }
  50% {
    transform: perspective(1000px) rotateY(90deg);
  }
  100% {
    transform: perspective(1000px) rotateY(180deg);
  }
}

.flip-card {
  transform-style: preserve-3d;
  animation: flip3d 1s ease-in-out;
}

/* 3D 透视效果 */
.perspective-container {
  perspective: 1000px;
  transform-style: preserve-3d;
}

.rotating-element {
  transform: rotateY(45deg);
  transition: transform 0.5s ease;
}

.rotating-element:hover {
  transform: rotateY(180deg);
}
```

## 性能优化

### 1. 硬件加速

```css
/* 触发GPU加速 */
.gpu-accelerated {
  transform: translateZ(0);
  backface-visibility: hidden;
  perspective: 1000px;
}

/* 性能友好的属性 */
.performance-friendly {
  /* 这些属性性能较好 */
  transform: translateX(100px);
  opacity: 0.5;
  filter: blur(5px);
}

/* 性能消耗大的属性 */
.performance-heavy {
  /* 慎用这些属性 */
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
  border-radius: 20px;
  background-position: 0 0;
}
```

### 2. 减少重绘回流

```css
/* 批量修改 */
.optimized {
  /* 不好的做法 */
  /* element.style.left = '100px';
     element.style.top = '50px'; */
  
  /* 好的做法 */
  transform: translate(100px, 50px);
}

/* 使用will-change */
.will-change {
  will-change: transform, opacity;
}

/* 避免布局抖动 */
.no-layout-thrash {
  /* 使用transform代替top/left */
  transform: translate(var(--x), var(--y));
}
```

### 3. 动画性能监测

```css
/* 性能调试类 */
.performance-debug {
  outline: 2px solid red;
}

.performance-debug:hover {
  outline-color: green;
}

/* 帧率监测 */
@keyframes frameRateTest {
  0% { opacity: 1; }
  100% { opacity: 0.99; }
}

.frame-rate-indicator {
  animation: frameRateTest 1s infinite;
}
```

## 响应式动画

### 1. 媒体查询中的动画

```css
/* 基础动画 */
.mobile-animation {
  animation: slideIn 0.5s ease;
}

/* 大屏设备增强 */
@media (min-width: 1024px) {
  .mobile-animation {
    animation: slideIn 1s ease, fadeIn 0.8s ease;
  }
}

/* 减少动画的媒体查询 */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 2. 自适应动画参数

```css
:root {
  --animation-duration: 0.3s;
  --animation-timing: ease-in-out;
}

.responsive-animation {
  transition: all var(--animation-duration) var(--animation-timing);
}

@media (min-width: 768px) {
  :root {
    --animation-duration: 0.5s;
  }
}

@media (min-width: 1200px) {
  :root {
    --animation-duration: 0.8s;
  }
}
```

## 浏览器兼容性

### 1. 前缀处理

```css
.animated-element {
  /* 标准属性 */
  animation: slideIn 1s ease;
  
  /* 浏览器前缀 */
  -webkit-animation: slideIn 1s ease;
  -moz-animation: slideIn 1s ease;
  -o-animation: slideIn 1s ease;
}

@keyframes slideIn {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(0); }
}

@-webkit-keyframes slideIn {
  0% { -webkit-transform: translateX(-100%); }
  100% { -webkit-transform: translateX(0); }
}
```

### 2. 特性检测

```css
/* 使用@supports检测支持情况 */
@supports (animation: slideIn 1s ease) {
  .modern-animation {
    animation: slideIn 1s ease;
  }
}

@supports not (animation: slideIn 1s ease) {
  .fallback-animation {
    transition: transform 1s ease;
  }
}
```

## 实用工具类

### 1. 动画工具类

```css
/* 淡入淡出 */
.fade-in { animation: fadeIn 0.5s ease; }
.fade-out { animation: fadeOut 0.5s ease; }

/* 滑动 */
.slide-in-left { animation: slideInLeft 0.5s ease; }
.slide-in-right { animation: slideInRight 0.5s ease; }
.slide-in-up { animation: slideInUp 0.5s ease; }
.slide-in-down { animation: slideInDown 0.5s ease; }

/* 缩放 */
.zoom-in { animation: zoomIn 0.3s ease; }
.zoom-out { animation: zoomOut 0.3s ease; }

/* 旋转 */
.rotate-in { animation: rotateIn 0.6s ease; }
.rotate-out { animation: rotateOut 0.6s ease; }

/* 弹跳 */
.bounce { animation: bounce 0.5s ease; }

/* 闪烁 */
.pulse { animation: pulse 2s infinite; }

/* 摇摆 */
.shake { animation: shake 0.5s ease; }
```

### 2. 过渡工具类

```css
/* 过渡速度 */
.transition-fast { transition: all 0.15s ease; }
.transition-normal { transition: all 0.3s ease; }
.transition-slow { transition: all 0.5s ease; }

/* 特定属性过渡 */
.transition-opacity { transition: opacity 0.3s ease; }
.transition-transform { transition: transform 0.3s ease; }
.transition-colors { transition: color 0.3s ease, background-color 0.3s ease; }

/* 延迟过渡 */
.transition-delay-100 { transition-delay: 0.1s; }
.transition-delay-200 { transition-delay: 0.2s; }
.transition-delay-300 { transition-delay: 0.3s; }
```

## 调试与测试

### 1. 动画调试

```css
/* 调试模式 */
.debug-animation * {
  animation-duration: 3s !important;
  animation-timing-function: linear !important;
}

/* 步进调试 */
.step-debug {
  animation-timing-function: steps(1, jump-end) !important;
}

/* 高亮动画元素 */
.animation-debug {
  outline: 2px solid #ff0000;
  background-color: rgba(255, 0, 0, 0.1);
}
```

### 2. 性能测试

```css
/* 性能测试类 */
.performance-test {
  will-change: transform;
  backface-visibility: hidden;
}

/* 帧率指示器 */
.fps-indicator::after {
  content: "60fps";
  position: fixed;
  top: 10px;
  right: 10px;
  background: green;
  color: white;
  padding: 5px;
  border-radius: 3px;
  font-size: 12px;
}

.low-fps::after {
  content: "30fps";
  background: orange;
}

.very-low-fps::after {
  content: "<30fps";
  background: red;
}
```
