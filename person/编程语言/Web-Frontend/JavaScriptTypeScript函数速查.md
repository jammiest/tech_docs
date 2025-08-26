# JavaScript/TypeScript 函数速查表

## 数组方法
```javascript
// 基础操作
arr.push(element);                    // 末尾添加元素
arr.pop();                            // 删除并返回最后一个元素
arr.unshift(element);                 // 开头添加元素
arr.shift();                          // 删除并返回第一个元素
arr.slice(start, end);                // 返回数组片段（不修改原数组）
arr.splice(start, deleteCount, ...items); // 删除/替换/添加元素

// 遍历方法
arr.forEach((item, index) => {});     // 遍历数组
arr.map((item, index) => {});         // 映射新数组
arr.filter((item, index) => {});      // 过滤数组
arr.find((item, index) => {});        // 查找第一个满足条件的元素
arr.findIndex((item, index) => {});   // 查找第一个满足条件的索引
arr.some((item, index) => {});        // 是否有元素满足条件
arr.every((item, index) => {});       // 是否所有元素都满足条件

// 归并方法
arr.reduce((acc, item, index) => {}, initialValue); // 从左到右归并
arr.reduceRight((acc, item, index) => {}, initialValue); // 从右到左归并

// 其他方法
arr.includes(value);                  // 是否包含某个值
arr.indexOf(value);                   // 查找元素索引
arr.concat(...arrays);                // 合并数组
arr.sort((a, b) => a - b);            // 排序数组
arr.reverse();                        // 反转数组
Array.isArray(value);                 // 判断是否为数组
Array.from(arrayLike);                // 类数组转真实数组
```

## 字符串方法
```javascript
str.length;                           // 字符串长度
str.charAt(index);                    // 返回指定位置字符
str.indexOf(searchValue);             // 查找子串位置
str.lastIndexOf(searchValue);         // 从后往前查找子串位置
str.includes(searchValue);            // 是否包含子串
str.startsWith(searchValue);          // 是否以子串开头
str.endsWith(searchValue);            // 是否以子串结尾
str.slice(start, end);                // 提取子串
str.substring(start, end);            // 提取子串
str.split(separator);                 // 分割字符串为数组
str.replace(searchValue, newValue);   // 替换子串
str.toLowerCase();                    // 转小写
str.toUpperCase();                    // 转大写
str.trim();                           // 去除首尾空格
str.trimStart();                      // 去除开头空格
str.trimEnd();                        // 去除末尾空格
str.padStart(length, padString);      // 开头填充
str.padEnd(length, padString);        // 结尾填充
```

## 对象方法
```javascript
// 对象操作
Object.keys(obj);                     // 获取所有键名
Object.values(obj);                   // 获取所有键值
Object.entries(obj);                  // 获取所有键值对
Object.assign(target, ...sources);    // 合并对象
Object.freeze(obj);                   // 冻结对象
Object.seal(obj);                     // 密封对象
Object.hasOwnProperty(prop);          // 检查自身属性
Object.defineProperty(obj, prop, descriptor); // 定义属性

// 对象复制和比较
const copy = {...obj};                // 浅拷贝
const deepCopy = JSON.parse(JSON.stringify(obj)); // 深拷贝（简易）
Object.is(value1, value2);            // 严格相等比较

// TypeScript 接口
interface User {
  id: number;
  name: string;
  age?: number;                       // 可选属性
  readonly createdAt: Date;           // 只读属性
}
```

## 函数方法
```javascript
// 函数定义
function name() {}                    // 函数声明
const func = () => {};                // 箭头函数
const func = function() {};           // 函数表达式

// 高阶函数
func.bind(thisArg, ...args);          // 绑定this
func.call(thisArg, ...args);          // 调用函数
func.apply(thisArg, [args]);          // 应用函数

// TypeScript 函数类型
type Callback = (error: Error | null, data?: any) => void;
function fetchData(callback: Callback): void {}

// 异步函数
async function asyncFunc() {
  const result = await promise;
  return result;
}
```

## 异步处理
```javascript
// Promise
new Promise((resolve, reject) => {}); // 创建Promise
promise.then(onFulfilled, onRejected); // 处理成功/失败
promise.catch(onRejected);            // 处理失败
promise.finally(onFinally);           // 最终处理
Promise.all([promise1, promise2]);    // 所有成功
Promise.race([promise1, promise2]);   // 第一个完成
Promise.allSettled([promise1, promise2]); // 所有完成

// async/await
try {
  const result = await asyncCall();
} catch (error) {
  console.error(error);
}

// TypeScript Promise 类型
const promise: Promise<string> = fetchData();
```

## 日期方法
```javascript
const now = new Date();               // 当前时间
date.getFullYear();                   // 获取年份
date.getMonth();                      // 获取月份(0-11)
date.getDate();                       // 获取日期(1-31)
date.getDay();                        // 获取星期(0-6)
date.getHours();                      // 获取小时
date.getMinutes();                    // 获取分钟
date.getSeconds();                    // 获取秒数
date.getTime();                       // 获取时间戳
date.toISOString();                   // 转ISO字符串
date.toLocaleDateString();            // 转本地日期字符串
date.toLocaleTimeString();            // 转本地时间字符串

// TypeScript
const timestamp: number = Date.now();
```

## 数学方法
```javascript
Math.abs(x);                          // 绝对值
Math.ceil(x);                         // 向上取整
Math.floor(x);                        // 向下取整
Math.round(x);                        // 四舍五入
Math.max(...values);                  // 最大值
Math.min(...values);                  // 最小值
Math.random();                        // 随机数(0-1)
Math.pow(base, exponent);             // 幂运算
Math.sqrt(x);                         // 平方根
Math.PI;                              // 圆周率
Math.E;                               // 自然常数
```

## 数据类型转换
```javascript
// 显式转换
Number('123');                        // 转数字
String(123);                          // 转字符串
Boolean(1);                           // 转布尔值
parseInt('123px');                    // 解析整数
parseFloat('3.14px');                 // 解析浮点数

// 隐式转换
+'123';                               // 转数字
123 + '';                             // 转字符串
!!value;                              // 转布尔值

// TypeScript 类型断言
const length = (str as string).length;
const length = (<string>str).length;   // JSX中避免使用
```

## 模块化
```javascript
// ES6 模块
import { namedExport } from './module';
import defaultExport from './module';
import * as module from './module';
export const name = 'value';
export default function() {};

// CommonJS (Node.js)
const module = require('./module');
module.exports = value;
exports.name = value;

// TypeScript 模块
import type { User } from './types';
```

## 错误处理
```javascript
// 错误处理
try {
  throw new Error('message');
} catch (error) {
  console.error(error.message);
} finally {
  cleanup();
}

// 自定义错误
class CustomError extends Error {
  constructor(message) {
    super(message);
    this.name = 'CustomError';
  }
}

// TypeScript
try {
  // code
} catch (error: unknown) {
  if (error instanceof Error) {
    console.error(error.message);
  }
}
```

## 浏览器API
```javascript
// DOM 操作
document.getElementById('id');
document.querySelector('.class');
document.querySelectorAll('div');
element.addEventListener('click', handler);
element.removeEventListener('click', handler);

// 存储
localStorage.setItem('key', 'value');
localStorage.getItem('key');
sessionStorage.setItem('key', 'value');

// 网络请求
fetch(url, options)
  .then(response => response.json())
  .then(data => console.log(data));

// TypeScript Fetch
interface ApiResponse {
  data: any;
  status: number;
}
```

## 实用工具函数
```javascript
// 防抖和节流
function debounce(func, delay) {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), delay);
  };
}

function throttle(func, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      func.apply(this, args);
    }
  };
}

// 深度合并
function deepMerge(target, source) {
  for (const key in source) {
    if (source[key] instanceof Object && target[key]) {
      deepMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}
```

## TypeScript 特有
```typescript
// 泛型
function identity<T>(arg: T): T {
  return arg;
}

// 类型守卫
function isString(value: any): value is string {
  return typeof value === 'string';
}

// 实用类型
type Partial<T> = { [P in keyof T]?: T[P] };
type Readonly<T> = { readonly [P in keyof T]: T[P] };
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

// 枚举
enum Color { Red, Green, Blue }
const color: Color = Color.Red;
```