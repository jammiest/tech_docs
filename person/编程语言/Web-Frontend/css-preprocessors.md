# SASS/LESS 指南

SASS（Syntactically Awesome Style Sheets）和 LESS（Leaner Style Sheets）是两种最流行的 CSS 预处理器，它们扩展了 CSS 的功能，提供了变量、嵌套、混合等强大特性。

## 核心概念对比

### 1. 变量定义

#### SASS (SCSS语法)
```scss
// 变量定义
$primary-color: #007bff;
$font-size-base: 16px;
$spacing-unit: 8px;

// 使用变量
.button {
  background-color: $primary-color;
  font-size: $font-size-base;
  padding: $spacing-unit * 2;
}

// 映射（Map）变量
$theme-colors: (
  "primary": #007bff,
  "secondary": #6c757d,
  "success": #28a745
);
```

#### LESS
```less
// 变量定义
@primary-color: #007bff;
@font-size-base: 16px;
@spacing-unit: 8px;

// 使用变量
.button {
  background-color: @primary-color;
  font-size: @font-size-base;
  padding: @spacing-unit * 2;
}

// 映射变量
@theme-colors: {
  primary: #007bff;
  secondary: #6c757d;
  success: #28a745;
};
```

### 2. 嵌套语法

#### SASS
```scss
// 选择器嵌套
.nav {
  background: #333;
  
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
    
    li {
      display: inline-block;
      
      a {
        color: white;
        text-decoration: none;
        
        &:hover {
          text-decoration: underline;
        }
      }
    }
  }
}

// 属性嵌套
.box {
  border: {
    width: 1px;
    style: solid;
    color: #ddd;
    radius: 4px;
  }
  
  margin: {
    top: 10px;
    bottom: 10px;
  }
}
```

#### LESS
```less
// 选择器嵌套
.nav {
  background: #333;
  
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
    
    li {
      display: inline-block;
      
      a {
        color: white;
        text-decoration: none;
        
        &:hover {
          text-decoration: underline;
        }
      }
    }
  }
}

// 属性嵌套（LESS不支持属性嵌套）
.box {
  border-width: 1px;
  border-style: solid;
  border-color: #ddd;
  border-radius: 4px;
}
```

## 高级特性

### 1. 混合（Mixins）

#### SASS Mixins
```scss
// 定义混合
@mixin button-style($bg-color, $text-color: white) {
  background-color: $bg-color;
  color: $text-color;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    background-color: darken($bg-color, 10%);
  }
}

// 使用混合
.primary-button {
  @include button-style(#007bff, white);
}

.secondary-button {
  @include button-style(#6c757d, white);
}

// 内容块混合
@mixin responsive($breakpoint) {
  @media (min-width: $breakpoint) {
    @content;
  }
}

.container {
  width: 100%;
  
  @include responsive(768px) {
    width: 750px;
    margin: 0 auto;
  }
}
```

#### LESS Mixins
```less
// 定义混合
.button-style(@bg-color, @text-color: white) {
  background-color: @bg-color;
  color: @text-color;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    background-color: darken(@bg-color, 10%);
  }
}

// 使用混合
.primary-button {
  .button-style(#007bff, white);
}

.secondary-button {
  .button-style(#6c757d, white);
}

// 带条件的混合
.text-overflow(@line: 1) {
  overflow: hidden;
  text-overflow: ellipsis;
  
  & when (@line = 1) {
    white-space: nowrap;
  }
  
  & when (@line > 1) {
    display: -webkit-box;
    -webkit-line-clamp: @line;
    -webkit-box-orient: vertical;
  }
}
```

### 2. 函数（Functions）

#### SASS 函数
```scss
// 内置函数
$primary: #007bff;
$dark-primary: darken($primary, 20%);
$light-primary: lighten($primary, 20%);

// 自定义函数
@function spacing($multiplier: 1) {
  @return $spacing-unit * $multiplier;
}

@function color-shade($color, $percentage: 10%) {
  @return mix(black, $color, $percentage);
}

// 使用函数
.container {
  padding: spacing(2);
  background-color: color-shade($primary, 20%);
}

// 列表和映射函数
$colors: #ff0000, #00ff00, #0000ff;
$first-color: nth($colors, 1);

$theme: (
  primary: #007bff,
  secondary: #6c757d
);

$primary-color: map-get($theme, primary);
```

#### LESS 函数
```less
// 内置函数
@primary: #007bff;
@dark-primary: darken(@primary, 20%);
@light-primary: lighten(@primary, 20%);

// 自定义函数（LESS 4.0+）
.spacing(@multiplier: 1) {
  return @spacing-unit * @multiplier;
}

.color-shade(@color, @percentage: 10%) {
  return mix(black, @color, @percentage);
}

// 使用函数
.container {
  padding: .spacing(2)[];
  background-color: .color-shade(@primary, 20%)[];
}

// 列表函数
@colors: #ff0000, #00ff00, #0000ff;
@first-color: extract(@colors, 1);
```

### 3. 继承（Extend）

#### SASS 继承
```scss
// 基础样式
%button-base {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

// 继承使用
.primary-button {
  @extend %button-base;
  background-color: #007bff;
  color: white;
}

.secondary-button {
  @extend %button-base;
  background-color: #6c757d;
  color: white;
}
```

#### LESS 继承
```less
// LESS 使用混合模拟继承
.button-base() {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.primary-button {
  .button-base();
  background-color: #007bff;
  color: white;
}

.secondary-button {
  .button-base();
  background-color: #6c757d;
  color: white;
}
```

## 项目组织架构

### 1. 文件结构
```
styles/
├── abstracts/
│   ├── _variables.scss
│   ├── _mixins.scss
│   ├── _functions.scss
│   └── _placeholders.scss
├── base/
│   ├── _reset.scss
│   ├── _typography.scss
│   └── _utilities.scss
├── components/
│   ├── _buttons.scss
│   ├── _forms.scss
│   └── _cards.scss
├── layout/
│   ├── _header.scss
│   ├── _footer.scss
│   └── _grid.scss
├── pages/
│   ├── _home.scss
│   ├── _about.scss
│   └── _contact.scss
└── main.scss
```

### 2. 主文件导入

#### SASS (main.scss)
```scss
// 抽象层
@import 'abstracts/variables';
@import 'abstracts/mixins';
@import 'abstracts/functions';
@import 'abstracts/placeholders';

// 基础层
@import 'base/reset';
@import 'base/typography';
@import 'base/utilities';

// 组件层
@import 'components/buttons';
@import 'components/forms';
@import 'components/cards';

// 布局层
@import 'layout/header';
@import 'layout/footer';
@import 'layout/grid';

// 页面层
@import 'pages/home';
@import 'pages/about';
@import 'pages/contact';
```

#### LESS (main.less)
```less
// 抽象层
@import 'abstracts/variables.less';
@import 'abstracts/mixins.less';
@import 'abstracts/functions.less';

// 基础层
@import 'base/reset.less';
@import 'base/typography.less';
@import 'base/utilities.less';

// 组件层
@import 'components/buttons.less';
@import 'components/forms.less';
@import 'components/cards.less';

// 布局层
@import 'layout/header.less';
@import 'layout/footer.less';
@import 'layout/grid.less';

// 页面层
@import 'pages/home.less';
@import 'pages/about.less';
@import 'pages/contact.less';
```

## 实用模式与技巧

### 1. 响应式设计模式

#### SASS
```scss
// 断点映射
$breakpoints: (
  'xs': 0,
  'sm': 576px,
  'md': 768px,
  'lg': 992px,
  'xl': 1200px
);

// 响应式混合
@mixin respond-to($breakpoint) {
  $value: map-get($breakpoints, $breakpoint);
  
  @if $value != null {
    @media (min-width: $value) {
      @content;
    }
  } @else {
    @warn "Breakpoint #{$breakpoint} is not defined in $breakpoints";
  }
}

// 使用
.container {
  width: 100%;
  
  @include respond-to('md') {
    width: 750px;
    margin: 0 auto;
  }
  
  @include respond-to('lg') {
    width: 970px;
  }
}
```

#### LESS
```less
// 断点变量
@breakpoint-xs: 0;
@breakpoint-sm: 576px;
@breakpoint-md: 768px;
@breakpoint-lg: 992px;
@breakpoint-xl: 1200px;

// 响应式混合
.respond-to(@breakpoint) {
  @media (min-width: @breakpoint) {
    @content();
  }
}

// 使用
.container {
  width: 100%;
  
  .respond-to(@breakpoint-md, {
    width: 750px;
    margin: 0 auto;
  });
  
  .respond-to(@breakpoint-lg, {
    width: 970px;
  });
}
```

### 2. 主题系统

#### SASS 主题
```scss
// 主题映射
$themes: (
  light: (
    background: #ffffff,
    text: #333333,
    primary: #007bff
  ),
  dark: (
    background: #1a1a1a,
    text: #ffffff,
    primary: #0d6efd
  )
);

// 主题混合
@mixin theme($property, $key) {
  @each $theme-name, $theme-map in $themes {
    .theme-#{$theme-name} & {
      #{$property}: map-get($theme-map, $key);
    }
  }
}

// 使用
.body {
  @include theme(background-color, background);
  @include theme(color, text);
}

.button {
  @include theme(background-color, primary);
}
```

#### LESS 主题
```less
// 主题定义
.theme-light() {
  @background: #ffffff;
  @text: #333333;
  @primary: #007bff;
}

.theme-dark() {
  @background: #1a1a1a;
  @text: #ffffff;
  @primary: #0d6efd;
}

// 主题应用
.theme-applier(@property, @value) {
  .theme-light & {
    @{property}: @value;
  }
  .theme-dark & {
    @{property}: @value;
  }
}

// 使用
.body {
  .theme-applier(background-color, @background);
  .theme-applier(color, @text);
}
```

## 构建与编译

### 1. 编译配置示例

#### Node.js 环境 (使用 node-sass/dart-sass)
```javascript
// package.json scripts
{
  "scripts": {
    "sass:watch": "sass --watch src/scss:dist/css",
    "sass:build": "sass src/scss/main.scss dist/css/main.css --style=compressed",
    "less:watch": "less-watch-compiler src/less dist/css",
    "less:build": "lessc src/less/main.less dist/css/main.css --clean-css"
  }
}
```

#### Webpack 配置
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [
          'style-loader',
          'css-loader',
          'less-loader'
        ]
      }
    ]
  }
};
```

### 2. 生产环境优化

```scss
// 开发环境：展开格式
// sass --style=expanded

// 生产环境：压缩格式
// sass --style=compressed

// 源映射支持
// sass --source-map

// 监听模式
// sass --watch

// 常用编译选项
$ sass input.scss output.css
$ sass src/scss:dist/css
$ sass --no-source-map --style=compressed src/scss:dist/css
```

## 最佳实践

### 1. 代码组织原则

1. **单一职责**：每个文件只负责一个明确的领域
2. **命名约定**：使用有意义的命名，遵循BEM或其他命名规范
3. **变量管理**：集中管理颜色、间距、字体等变量
4. **混合复用**：将重复模式抽象为混合
5. **注释文档**：为混合、函数和复杂逻辑添加注释

### 2. 性能考虑

1. **选择器深度**：避免过度嵌套（建议不超过4层）
2. **继承使用**：谨慎使用@extend，避免选择器爆炸
3. **混合复杂度**：避免创建过于复杂的混合
4. **编译优化**：生产环境使用压缩输出

### 3. 团队协作规范

```scss
// 变量命名规范
$color-primary: #007bff;
$spacing-small: 8px;
$font-size-base: 16px;

// 混合命名规范
@mixin text-truncate { /* ... */ }
@mixin responsive($breakpoint) { /* ... */ }

// 文件命名规范
// _variables.scss
// _mixins.scss
// _buttons.scss
```

## 常见问题解决

### 1. 浏览器兼容性

```scss
// 自动添加浏览器前缀
@mixin prefix($property, $value) {
  -webkit-#{$property}: $value;
  -moz-#{$property}: $value;
  -ms-#{$property}: $value;
  #{$property}: $value;
}

.transform($value) {
  @include prefix(transform, $value);
}

// 使用
.element {
  @include transform(translateX(100px));
}
```

### 2. 调试技巧

```scss
// 调试混合
@mixin debug($color: red) {
  outline: 2px solid $color;
  background-color: rgba($color, 0.1);
}

// 条件调试
$debug: false;

@if $debug {
  * {
    @include debug();
  }
}
```
