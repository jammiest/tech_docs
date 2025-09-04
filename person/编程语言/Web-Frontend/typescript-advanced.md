# TypeScript 高级类型指南

TypeScript 的类型系统提供了强大的静态类型检查能力，其高级类型特性能够显著提升代码的健壮性和可维护性。本文将深入探讨 TypeScript 的高级类型特性。

## 核心高级类型

### 1. 联合类型与交叉类型

#### 联合类型 (Union Types)
表示值可以是几种类型之一：

```typescript
type Status = 'success' | 'error' | 'pending';
type ID = string | number;

function handleResponse(response: { status: Status, data: any }) {
  if (response.status === 'success') {
    console.log(response.data);
  }
}
```

#### 交叉类型 (Intersection Types)
将多个类型合并为一个类型：

```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: string;
  department: string;
}

type EmployeePerson = Person & Employee;

const john: EmployeePerson = {
  name: 'John',
  age: 30,
  employeeId: 'E123',
  department: 'IT'
};
```

### 2. 类型守卫与类型收窄

TypeScript 可以通过类型守卫在特定代码块中收窄类型范围：

```typescript
// typeof 类型守卫
function padLeft(value: string, padding: string | number) {
  if (typeof padding === 'number') {
    return ' '.repeat(padding) + value; // padding 被识别为 number
  }
  return padding + value; // padding 被识别为 string
}

// instanceof 类型守卫
class Bird { fly() {} }
class Fish { swim() {} }

function move(animal: Bird | Fish) {
  if (animal instanceof Bird) {
    animal.fly(); // animal 被识别为 Bird
  } else {
    animal.swim(); // animal 被识别为 Fish
  }
}

// 自定义类型守卫
function isString(value: any): value is string {
  return typeof value === 'string';
}
```

### 3. 映射类型

可以基于现有类型创建新类型：

```typescript
// 只读映射
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// 可选映射
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// 实际应用
interface User {
  id: number;
  name: string;
  email: string;
}

type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
```

## 实用工具类型

TypeScript 提供了一系列内置工具类型：

### 1. 条件类型

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type Extract<T, U> = T extends U ? T : never;

type Exclude<T, U> = T extends U ? never : T;

// 示例
type T0 = Extract<'a' | 'b' | 'c', 'a' | 'f'>; // 'a'
type T1 = Exclude<'a' | 'b' | 'c', 'a' | 'f'>; // 'b' | 'c'
```

### 2. 模板字面量类型

```typescript
type EventName = 'click' | 'scroll' | 'mousemove';
type OnEvent = `on${Capitalize<EventName>}`;
// 结果为 'onClick' | 'onScroll' | 'onMousemove'
```

### 3. 递归类型

```typescript
type JsonValue = 
  | string 
  | number 
  | boolean 
  | null 
  | JsonValue[] 
  | { [key: string]: JsonValue };

const jsonData: JsonValue = {
  name: "John",
  age: 30,
  hobbies: ["reading", "swimming"],
  address: {
    street: "123 Main St",
    city: "New York"
  }
};
```

## 高级模式

### 1. 类型推断

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

function getUser(): { name: string; age: number } {
  return { name: 'Alice', age: 25 };
}

type UserReturnType = ReturnType<typeof getUser>;
// { name: string; age: number }
```

### 2. 可变元组类型

```typescript
function tuple<T extends any[]>(...args: T): T {
  return args;
}

const t = tuple(1, 'hello', true); // [number, string, boolean]
```

### 3. 类型安全的装饰器

```typescript
function logMethod(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Method ${propertyKey} returned:`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @logMethod
  add(a: number, b: number): number {
    return a + b;
  }
}
```

## 最佳实践

1. **优先使用类型别名**：对于复杂类型定义，使用 `type` 而非 `interface`
2. **合理使用泛型**：提高代码复用性同时保持类型安全
3. **利用类型推断**：减少冗余的类型注解
4. **谨慎使用 any**：尽量使用更精确的类型替代 any
5. **利用工具类型**：充分利用内置工具类型简化代码

## 性能考虑

1. 复杂类型运算可能会增加编译时间
2. 递归类型深度过大会导致性能问题
3. 条件类型嵌套过多会影响类型检查速度

## 版本兼容性

| 特性 | TypeScript 版本要求 |
|------|---------------------|
| 可变元组类型 | 4.0+ |
| 模板字面量类型 | 4.1+ |
| 类型导入导出 | 3.8+ |
| 断言函数 | 3.7+ |
