# CSS 速查手册

## 📋 选择器

### 基础选择器
```css
* {}                     /* 通用选择器 */
element {}               /* 元素选择器 */
.class {}                /* 类选择器 */
#id {}                   /* ID选择器 */
selector1, selector2 {}  /* 分组选择器 */
```

### 组合选择器
```css
selector1 selector2 {}   /* 后代选择器 */
selector1 > selector2 {} /* 子选择器 */
selector1 + selector2 {} /* 相邻兄弟选择器 */
selector1 ~ selector2 {} /* 通用兄弟选择器 */
```

### 属性选择器
```css
[attribute] {}           /* 具有该属性的元素 */
[attribute=value] {}     /* 属性等于指定值 */
[attribute~=value] {}    /* 属性包含指定词 */
[attribute|=value] {}    /* 属性以指定值开头 */
[attribute^=value] {}    /* 属性以指定值开始 */
[attribute$=value] {}    /* 属性以指定值结束 */
[attribute*=value] {}    /* 属性包含指定值 */
```

### 伪类选择器
```css
:link {}                 /* 未访问的链接 */
:visited {}              /* 已访问的链接 */
:hover {}                /* 鼠标悬停 */
:active {}               /* 激活状态 */
:focus {}                /* 获得焦点 */
:first-child {}          /* 第一个子元素 */
:last-child {}           /* 最后一个子元素 */
:nth-child(n) {}         /* 第n个子元素 */
:nth-of-type(n) {}       /* 第n个同类元素 */
:not(selector) {}        /* 非指定选择器 */
:checked {}              /* 选中的表单元素 */
:disabled {}             /* 禁用的表单元素 */
```

### 伪元素选择器
```css
::before {}              /* 元素前插入内容 */
::after {}               /* 元素后插入内容 */
::first-letter {}        /* 首字母 */
::first-line {}          /* 首行 */
::selection {}           /* 用户选中的内容 */
```

## 🎨 盒模型

### 盒模型属性
```css
box-sizing: content-box; /* 标准盒模型 */
box-sizing: border-box;  /* 怪异盒模型 */

width: 100px;           /* 宽度 */
height: 100px;          /* 高度 */
min-width: 100px;       /* 最小宽度 */
max-width: 100px;       /* 最大宽度 */
min-height: 100px;      /* 最小高度 */
max-height: 100px;      /* 最大高度 */
```

### 边距与填充
```css
margin: 10px;           /* 四个方向外边距 */
margin: 10px 20px;      /* 上下 | 左右 */
margin: 10px 20px 30px; /* 上 | 左右 | 下 */
margin: 10px 20px 30px 40px; /* 上右下左 */

padding: 10px;          /* 四个方向内边距 */
/* 语法同margin */
```

### 边框
```css
border: 1px solid #000; /* 边框简写 */
border-width: 1px;      /* 边框宽度 */
border-style: solid;    /* 边框样式 */
border-color: #000;     /* 边框颜色 */

border-radius: 10px;    /* 圆角 */
border-top-left-radius: 10px; /* 左上圆角 */

outline: 1px solid red; /* 轮廓线 */
```

## 🎯 布局

### 显示模式
```css
display: block;         /* 块级元素 */
display: inline;        /* 行内元素 */
display: inline-block;  /* 行内块元素 */
display: none;          /* 隐藏元素 */
display: flex;          /* 弹性布局 */
display: grid;          /* 网格布局 */
```

### 定位
```css
position: static;       /* 默认定位 */
position: relative;     /* 相对定位 */
position: absolute;     /* 绝对定位 */
position: fixed;        /* 固定定位 */
position: sticky;       /* 粘性定位 */

top: 0;                 /* 上偏移 */
right: 0;               /* 右偏移 */
bottom: 0;              /* 下偏移 */
left: 0;                /* 左偏移 */

z-index: 10;            /* 堆叠顺序 */
```

### Flexbox 布局
```css
/* 容器属性 */
display: flex;
flex-direction: row;    /* 主轴方向: row | row-reverse | column | column-reverse */
flex-wrap: nowrap;      /* 换行: nowrap | wrap | wrap-reverse */
justify-content: flex-start; /* 主轴对齐: flex-start | flex-end | center | space-between | space-around */
align-items: stretch;   /* 交叉轴对齐: stretch | flex-start | flex-end | center | baseline */
align-content: stretch; /* 多行对齐 */

/* 项目属性 */
order: 0;               /* 排列顺序 */
flex-grow: 0;           /* 放大比例 */
flex-shrink: 1;         /* 缩小比例 */
flex-basis: auto;       /* 初始大小 */
flex: 0 1 auto;         /* 简写: grow shrink basis */
align-self: auto;       /* 单个项目对齐 */
```

### Grid 布局
```css
/* 容器属性 */
display: grid;
grid-template-columns: 100px 1fr; /* 列定义 */
grid-template-rows: 100px 1fr;    /* 行定义 */
grid-template-areas: "header header" "sidebar content"; /* 区域命名 */
gap: 10px;              /* 网格间隙 */
justify-items: stretch; /* 单元格水平对齐 */
align-items: stretch;   /* 单元格垂直对齐 */

/* 项目属性 */
grid-column: 1 / 3;     /* 列开始/结束 */
grid-row: 1 / 2;        /* 行开始/结束 */
grid-area: header;      /* 指定区域 */
justify-self: stretch;  /* 单个项目水平对齐 */
align-self: stretch;    /* 单个项目垂直对齐 */
```

## 🎨 样式属性

### 文字样式
```css
font-family: Arial, sans-serif; /* 字体 */
font-size: 16px;        /* 字体大小 */
font-weight: normal;    /* 字体粗细: normal | bold | 100-900 */
font-style: normal;     /* 字体样式: normal | italic | oblique */
line-height: 1.5;       /* 行高 */
text-align: left;       /* 文本对齐: left | right | center | justify */
text-decoration: none;  /* 文本装饰: none | underline | overline | line-through */
text-transform: none;   /* 文本转换: none | capitalize | uppercase | lowercase */
color: #000;            /* 文字颜色 */
```

### 背景
```css
background-color: #fff; /* 背景颜色 */
background-image: url('image.jpg'); /* 背景图片 */
background-repeat: no-repeat; /* 重复方式: repeat | no-repeat | repeat-x | repeat-y */
background-position: center; /* 背景位置 */
background-size: cover; /* 背景大小: cover | contain | 长度 */
background-attachment: scroll; /* 背景附着: scroll | fixed */
background: #fff url('image.jpg') no-repeat center; /* 简写 */
```

### 列表
```css
list-style-type: disc;  /* 列表标记类型: disc | circle | square | none */
list-style-position: outside; /* 标记位置: inside | outside */
list-style-image: url('bullet.png'); /* 自定义标记 */
list-style: disc outside none; /* 简写 */
```

### 表格
```css
border-collapse: collapse; /* 边框合并: collapse | separate */
border-spacing: 0;       /* 边框间距 */
empty-cells: show;       /* 空单元格: show | hide */
```

## 🎭 变换与动画

### 变换 (Transform)
```css
transform: none;         /* 变换 */
transform: translate(10px, 20px); /* 平移 */
transform: rotate(45deg); /* 旋转 */
transform: scale(1.5);   /* 缩放 */
transform: skew(10deg, 5deg); /* 倾斜 */
transform: matrix(1, 0, 0, 1, 0, 0); /* 矩阵变换 */
transform-origin: center; /* 变换原点 */
```

### 过渡 (Transition)
```css
transition-property: all; /* 过渡属性 */
transition-duration: 0.3s; /* 过渡时间 */
transition-timing-function: ease; /* 时间函数: ease | linear | ease-in | ease-out | ease-in-out */
transition-delay: 0s;    /* 过渡延迟 */
transition: all 0.3s ease; /* 简写 */
```

### 动画 (Animation)
```css
animation-name: slide;   /* 动画名称 */
animation-duration: 1s;  /* 动画时长 */
animation-timing-function: ease; /* 时间函数 */
animation-delay: 0s;     /* 动画延迟 */
animation-iteration-count: infinite; /* 迭代次数: infinite | 数字 */
animation-direction: normal; /* 方向: normal | reverse | alternate | alternate-reverse */
animation-fill-mode: none; /* 填充模式: none | forwards | backwards | both */
animation-play-state: running; /* 播放状态: running | paused */
animation: slide 1s ease infinite; /* 简写 */

@keyframes slide {
  from { transform: translateX(0); }
  to { transform: translateX(100px); }
}
```

## 📱 响应式设计

### 媒体查询
```css
/* 屏幕宽度 */
@media (max-width: 768px) {}  /* 最大宽度 */
@media (min-width: 769px) {}  /* 最小宽度 */
@media (min-width: 769px) and (max-width: 1024px) {} /* 范围 */

/* 设备特性 */
@media (orientation: portrait) {}  /* 竖屏 */
@media (orientation: landscape) {} /* 横屏 */
@media (prefers-color-scheme: dark) {} /* 深色模式 */
@media (prefers-reduced-motion: reduce) {} /* 减少动画 */

/* 打印样式 */
@media print {}
```

### 响应式单位
```css
width: 100%;            /* 百分比 */
width: 100vw;           /* 视口宽度 */
height: 100vh;          /* 视口高度 */
font-size: 1rem;        /* 根元素字体大小 */
font-size: 1em;         /* 父元素字体大小 */
font-size: clamp(1rem, 2.5vw, 2rem); /* 流体排版 */
```

## 🎯 常用技巧

### 居中方案
```css
/* 水平居中 */
text-align: center;     /* 文本和行内元素 */
margin: 0 auto;         /* 块级元素 */

/* 绝对定位居中 */
position: absolute;
top: 50%;
left: 50%;
transform: translate(-50%, -50%);

/* Flexbox 居中 */
display: flex;
justify-content: center;
align-items: center;

/* Grid 居中 */
display: grid;
place-items: center;
```

### 清除浮动
```css
.clearfix::after {
  content: "";
  display: table;
  clear: both;
}
```

### 隐藏元素
```css
display: none;          /* 完全隐藏，不占空间 */
visibility: hidden;     /* 隐藏，但占空间 */
opacity: 0;             /* 透明，但可交互 */
```

### 文本溢出
```css
white-space: nowrap;    /* 不换行 */
overflow: hidden;       /* 隐藏溢出 */
text-overflow: ellipsis; /* 显示省略号 */
```

## 🔧 CSS 变量
```css
:root {
  --primary-color: #007bff;
  --spacing: 16px;
  --font-size: 1rem;
}

.element {
  color: var(--primary-color);
  padding: var(--spacing);
  font-size: var(--font-size);
}
```

## 🎨 滤镜效果
```css
filter: blur(5px);      /* 模糊 */
filter: brightness(0.5); /* 亮度 */
filter: contrast(200%); /* 对比度 */
filter: grayscale(100%); /* 灰度 */
filter: hue-rotate(90deg); /* 色相旋转 */
filter: invert(100%);   /* 反色 */
filter: opacity(50%);   /* 透明度 */
filter: saturate(200%); /* 饱和度 */
filter: sepia(100%);    /* 深褐色 */
filter: drop-shadow(2px 2px 5px rgba(0,0,0,0.5)); /* 阴影 */
```

这个速查表涵盖了CSS的主要特性和常用技巧，适合日常开发中快速查阅使用。