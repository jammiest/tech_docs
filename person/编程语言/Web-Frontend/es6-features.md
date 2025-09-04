# ES6+ 新特性

ES6（ECMAScript 2015）是 JavaScript 语言的一次重大更新，引入了许多现代编程特性。后续的 ES7-ES13 版本持续增强了语言能力。

## 核心特性概览

### 1. 块级作用域与变量声明

#### let 与 const
- `let` 声明块级作用域变量
- `const` 声明块级作用域常量

```javascript
// 块级作用域示例
{
  let x = 10;
  const y = 20;
  console.log(x + y); // 30
}
// console.log(x); // ReferenceError
```

### 2. 箭头函数

提供更简洁的函数语法和词法 `this` 绑定：

```javascript
// 传统函数
function add(a, b) {
  return a + b;
}

// 箭头函数
const add = (a, b) => a + b;

// this 绑定差异
const obj = {
  value: 42,
  traditional: function() {
    console.log(this.value); // 42
  },
  arrow: () => {
    console.log(this.value); // undefined
  }
};
```

### 3. 模板字符串

支持多行字符串和表达式插值：

```javascript
const name = "John";
const age = 30;

const message = `Hello ${name},
You are ${age} years old.
Next year you'll be ${age + 1}.`;
```

### 4. 解构赋值

从数组或对象中提取值：

```javascript
// 数组解构
const [first, second] = [1, 2, 3];

// 对象解构
const { name, age } = person;

// 函数参数解构
function greet({ name, age }) {
  return `Hello ${name}, age ${age}`;
}
```

### 5. 默认参数

```javascript
function multiply(a, b = 1) {
  return a * b;
}

multiply(5);    // 5
multiply(5, 2); // 10
```

### 6. 扩展运算符

```javascript
// 数组展开
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

// 对象展开
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 }; // { a: 1, b: 2, c: 3 }
```

## 高级特性

### Promise 与异步编程

```javascript
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("Data received");
    }, 1000);
  });
};

fetchData()
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

### Class 语法糖

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello, ${this.name}`;
  }

  // 静态方法
  static createAnonymous() {
    return new Person("Anonymous");
  }
}
```

### 模块系统

```javascript
// math.js
export const PI = 3.14159;
export function square(x) {
  return x * x;
}

// app.js
import { PI, square } from './math.js';
```

## ES7+ 新增特性

### 指数运算符

```javascript
const result = 2 ** 3; // 8
```

### Array.includes()

```javascript
const arr = [1, 2, 3];
arr.includes(2); // true
```

### Async/Await (ES8)

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### 对象扩展方法

```javascript
const obj = { a: 1, b: 2 };

// Object.values()
Object.values(obj); // [1, 2]

// Object.entries()
Object.entries(obj); // [['a', 1], ['b', 2]]
```

## 性能考量

ES6+ 特性在现代 JavaScript 引擎中通常有良好的性能表现，但需要注意：

1. **箭头函数**：在大多数情况下性能优于传统函数，但不适合作为构造函数
2. **Promise**：比回调更易读，但会产生微任务队列开销
3. **解构赋值**：在复杂嵌套结构中可能产生性能开销

## 浏览器兼容性

建议使用 Babel 进行转译以确保跨浏览器兼容性。主要特性支持情况：

| 特性 | Chrome | Firefox | Safari | Edge |
|------|--------|---------|--------|------|
| let/const | 49+ | 44+ | 10+ | 14+ |
| 箭头函数 | 45+ | 22+ | 10+ | 14+ |
| Promise | 32+ | 29+ | 8+ | 14+ |
| async/await | 55+ | 52+ | 10.1+ | 15+ |

## 最佳实践

1. 优先使用 `const`，必要时使用 `let`，避免 `var`
2. 使用箭头函数保持 `this` 上下文一致性
3. 利用解构赋值简化代码结构
4. 使用 Promise 和 async/await 处理异步操作
5. 合理使用模块化组织代码结构
