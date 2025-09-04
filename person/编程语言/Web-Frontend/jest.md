# Jest 使用指南

Jest 是一个功能丰富的 JavaScript 测试框架，专注于简洁性和强大的功能。本指南将全面介绍 Jest 的使用方法、高级功能和最佳实践。

## 基础配置

### 1. 安装和初始化

```bash
# 使用 npm 安装
npm install --save-dev jest @types/jest

# 使用 yarn 安装
yarn add --dev jest @types/jest

# 初始化 Jest 配置
npx jest --init

# 在 package.json 中添加脚本
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### 2. 基础配置文件

```javascript
// jest.config.js
module.exports = {
  // 测试环境
  testEnvironment: 'jsdom',
  
  // 测试文件匹配模式
  testMatch: [
    '**/__tests__/**/*.[jt]s?(x)',
    '**/?(*.)+(spec|test).[tj]s?(x)'
  ],
  
  // 模块路径映射
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|svg)$': '<rootDir>/__mocks__/fileMock.js'
  },
  
  // 设置测试文件
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  
  // 覆盖率配置
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js'
  ],
  
  // 变换配置
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest'
  }
};
```

### 3. 测试环境设置

```javascript
// jest.setup.js
import '@testing-library/jest-dom';

// 全局测试配置
beforeAll(() => {
  // 所有测试之前执行
  console.log('Starting test suite...');
});

afterAll(() => {
  // 所有测试之后执行
  console.log('Test suite completed');
});

// 全局变量设置
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

## 基础测试语法

### 1. 测试结构

```javascript
// 描述测试套件
describe('Math operations', () => {
  // 前置钩子
  beforeEach(() => {
    // 每个测试前执行
  });
  
  afterEach(() => {
    // 每个测试后执行
  });
  
  // 测试用例
  it('adds two numbers correctly', () => {
    expect(1 + 2).toBe(3);
  });
  
  // 使用 test 别名
  test('subtracts two numbers correctly', () => {
    expect(5 - 3).toBe(2);
  });
  
  // 异步测试
  it('handles async operations', async () => {
    const result = await Promise.resolve(42);
    expect(result).toBe(42);
  });
});
```

### 2. 断言方法

```javascript
// 基本断言
expect(value).toBe(expected);        // 严格相等
expect(value).toEqual(expected);     // 深度相等
expect(value).not.toBe(expected);    // 取反断言

// 真值判断
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// 数字比较
expect(value).toBeGreaterThan(3);
expect(value).toBeGreaterThanOrEqual(4);
expect(value).toBeLessThan(10);
expect(value).toBeLessThanOrEqual(9);
expect(value).toBeCloseTo(0.3);      // 浮点数近似相等

// 字符串匹配
expect(string).toMatch(/regex/);
expect(string).toContain('substring');

// 数组检查
expect(array).toContain(item);
expect(array).toHaveLength(5);

// 对象检查
expect(object).toHaveProperty('key');
expect(object).toHaveProperty('key', 'value');

// 错误抛出
expect(() => { throw new Error() }).toThrow();
expect(() => { throw new Error() }).toThrow('error message');

// 异步断言
await expect(Promise.resolve(42)).resolves.toBe(42);
await expect(Promise.reject('error')).rejects.toMatch('error');
```

## Mock 功能

### 1. 函数 Mock

```javascript
// 创建 Mock 函数
const mockFn = jest.fn();

// 基本使用
mockFn('hello');
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith('hello');
expect(mockFn).toHaveBeenCalledTimes(1);

// Mock 返回值
mockFn.mockReturnValue('mocked value');
mockFn.mockReturnValueOnce('first call');
mockFn.mockReturnValueOnce('second call');

// Mock 实现
mockFn.mockImplementation((arg) => arg.toUpperCase());
mockFn.mockImplementationOnce(() => 'special');

// Mock 异步函数
mockFn.mockResolvedValue('async value');
mockFn.mockResolvedValueOnce('first async');
mockFn.mockRejectedValue(new Error('async error'));

// 获取调用信息
const calls = mockFn.mock.calls;
const results = mockFn.mock.results;
const instances = mockFn.mock.instances;

// 清除 Mock
mockFn.mockClear();        // 清除调用记录
mockFn.mockReset();        // 清除所有 Mock 信息
mockFn.mockRestore();     // 恢复原始实现
```

### 2. 模块 Mock

```javascript
// 完整模块 Mock
jest.mock('axios', () => ({
  get: jest.fn().mockResolvedValue({ data: {} }),
  post: jest.fn()
}));

// 部分 Mock
jest.mock('./module', () => {
  const original = jest.requireActual('./module');
  return {
    ...original,
    specificFunction: jest.fn()
  };
});

// 自动 Mock
jest.mock('some-module');

// 在测试文件中使用 Mock
import { mockedFunction } from './module';
mockedFunction.mockReturnValue('mocked');

// 手动 Mock 文件
// __mocks__/axios.js
module.exports = {
  get: jest.fn(() => Promise.resolve({ data: {} }))
};
```

### 3. Timer Mock

```javascript
// Mock 定时器
jest.useFakeTimers();

// 控制时间
jest.advanceTimersByTime(1000);      // 前进 1 秒
jest.runOnlyPendingTimers();         // 运行等待中的定时器
jest.runAllTimers();                 // 运行所有定时器
jest.clearAllTimers();               // 清除所有定时器

// 测试示例
test('debounce function', () => {
  const mockFn = jest.fn();
  const debounced = debounce(mockFn, 1000);
  
  debounced();
  jest.advanceTimersByTime(500);
  expect(mockFn).not.toHaveBeenCalled();
  
  jest.advanceTimersByTime(500);
  expect(mockFn).toHaveBeenCalledTimes(1);
});

// 恢复真实定时器
jest.useRealTimers();
```

## 异步测试

### 1. Promise 测试

```javascript
// Resolves/Rejects 断言
test('resolves to lemon', async () => {
  await expect(Promise.resolve('lemon')).resolves.toBe('lemon');
  await expect(Promise.reject(new Error('error'))).rejects.toThrow('error');
});

// Async/Await 测试
test('fetches data', async () => {
  const data = await fetchData();
  expect(data).toEqual({ id: 1, name: 'John' });
});

// 错误处理测试
test('throws error on failure', async () => {
  await expect(asyncFunction()).rejects.toThrow('Specific error message');
});
```

### 2. 回调函数测试

```javascript
// 使用 done 回调
test('calls callback with data', (done) => {
  function callback(data) {
    try {
      expect(data).toBe('expected data');
      done();
    } catch (error) {
      done(error);
    }
  }
  
  asyncFunction(callback);
});

// 使用 Promise
test('calls callback with data', () => {
  return new Promise((resolve, reject) => {
    function callback(data) {
      try {
        expect(data).toBe('expected data');
        resolve();
      } catch (error) {
        reject(error);
      }
    }
    
    asyncFunction(callback);
  });
});
```

## 快照测试

### 1. 组件快照

```javascript
import renderer from 'react-test-renderer';

test('component snapshot', () => {
  const tree = renderer.create(<Component prop="value" />).toJSON();
  expect(tree).toMatchSnapshot();
});

// 内联快照
test('inline snapshot', () => {
  const result = processData(input);
  expect(result).toMatchInlineSnapshot(`
    {
      "id": 1,
      "name": "John"
    }
  `);
});

// 属性匹配器
test('snapshot with property matchers', () => {
  const obj = {
    id: 1,
    name: 'John',
    date: new Date(),
    count: Math.random()
  };
  
  expect(obj).toMatchSnapshot({
    date: expect.any(Date),
    count: expect.any(Number)
  });
});
```

### 2. 快照更新

```bash
# 更新失败的快照
npm test -- --updateSnapshot

# 或使用 u 标志
npm test -- -u

# 交互式更新模式
npm test -- --watch
# 然后按 'u' 更新快照
```

## 高级功能

### 1. 参数化测试

```javascript
// test.each 数组格式
test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3]
])('adds %i + %i to equal %i', (a, b, expected) => {
  expect(a + b).toBe(expected);
});

// test.each 模板字符串格式
test.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added to $b', ({ a, b, expected }) => {
  expect(a + b).toBe(expected);
});

// describe.each 用于测试套件
describe.each([
  { a: 1, b: 1, expected: 2 },
  { a: 1, b: 2, expected: 3 },
  { a: 2, b: 1, expected: 3 }
])('describe each test', ({ a, b, expected }) => {
  test(`${a} + ${b} = ${expected}`, () => {
    expect(a + b).toBe(expected);
  });
});
```

### 2. 自定义匹配器

```javascript
// 创建自定义匹配器
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be within range ${floor} - ${ceiling}`,
        pass: false
      };
    }
  }
});

// 使用自定义匹配器
test('numeric ranges', () => {
  expect(100).toBeWithinRange(90, 110);
  expect(101).not.toBeWithinRange(0, 100);
});

// 异步自定义匹配器
expect.extend({
  async toBeDivisibleBy(received, argument) {
    const pass = received % argument === 0;
    return {
      message: () => `expected ${received} to be divisible by ${argument}`,
      pass
    };
  }
});
```

## 性能优化

### 1. 测试性能配置

```javascript
// jest.config.js
module.exports = {
  // 最大工作进程数
  maxWorkers: '50%',
  
  // 最大并发数
  maxConcurrent: 5,
  
  // 测试超时时间
  testTimeout: 5000,
  
  // 缓存配置
  cache: true,
  cacheDirectory: '/tmp/jest',
  
  // 隔离模式
  isolate: false
};
```

### 2. 测试套件优化

```javascript
// 使用 test.concurrent 进行并发测试
test.concurrent('async test 1', async () => {
  await expect(Promise.resolve(1)).resolves.toBe(1);
});

test.concurrent('async test 2', async () => {
  await expect(Promise.resolve(2)).resolves.toBe(2);
});

// 跳过耗时测试
test.skip('slow test', () => {
  // 这个测试会被跳过
});

// 只运行特定测试
test.only('critical test', () => {
  // 只运行这个测试
});

// 条件测试
test.runIf(process.env.CI)('CI only test', () => {
  // 只在 CI 环境运行
});
```

## 调试技巧

### 1. 调试配置

```javascript
// 在 VS Code 中调试 Jest
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": ["--runInBand", "--watchAll=false"],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "disableOptimizations": true
    }
  ]
}

// 使用 --inspect-brk 调试
node --inspect-brk ./node_modules/.bin/jest --runInBand
```

### 2. 调试技巧

```javascript
// 使用 console.log 调试
test('debug test', () => {
  console.log('Debug information');
  // 测试逻辑
});

// 使用 debugger 语句
test('debug with breakpoint', () => {
  debugger; // 会在调试时暂停
  // 测试逻辑
});

// 使用 Jest 的 verbose 模式
npm test -- --verbose

// 查看测试时间
npm test -- --showConfig
```

## 常见问题解决

### 1. 配置问题

```javascript
// 解决模块导入问题
// jest.config.js
module.exports = {
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less)$': 'identity-obj-proxy'
  },
  
  // 处理 ES 模块
  transformIgnorePatterns: [
    'node_modules/(?!(module-to-transform)/)'
  ]
};

// 解决 TypeScript 问题
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  globals: {
    'ts-jest': {
      tsconfig: 'tsconfig.test.json'
    }
  }
};
```

### 2. 性能问题

```javascript
// 优化大型测试套件
module.exports = {
  // 只测试变化的文件
  onlyChanged: true,
  
  // 测试文件隔离
  isolate: false,
  
  // 使用 worker 线程
  maxWorkers: 4,
  
  // 缓存策略
  cache: true
};

// 使用 jest-runner-groups 分组测试
module.exports = {
  runner: 'jest-runner-groups',
  testMatch: ['**/*.test.js'],
  groups: ['unit', 'integration']
};
```

## 最佳实践

### 1. 测试组织

```javascript
// 测试文件结构
describe('Component', () => {
  // 生命周期钩子
  beforeAll(() => {});
  afterAll(() => {});
  beforeEach(() => {});
  afterEach(() => {});
  
  // 测试分组
  describe('when prop is true', () => {
    it('should render correctly', () => {});
  });
  
  describe('when prop is false', () => {
    it('should not render', () => {});
  });
  
  // 边缘用例
  describe('edge cases', () => {
    it('should handle null values', () => {});
    it('should handle undefined values', () => {});
  });
});
```

### 2. Mock 最佳实践

```javascript
// 在 beforeEach 中重置 Mock
let mockFunction;

beforeEach(() => {
  mockFunction = jest.fn();
});

// 使用 jest.spyOn 进行部分 Mock
test('spy on function', () => {
  const spy = jest.spyOn(console, 'log');
  functionUnderTest();
  expect(spy).toHaveBeenCalledWith('expected message');
  spy.mockRestore();
});

// 避免过度 Mock
// 不好：Mock 所有依赖
// 好：只 Mock 外部依赖，测试业务逻辑
```

### 3. 测试数据管理

```javascript
// 使用工厂函数创建测试数据
const createUser = (overrides = {}) => ({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  ...overrides
});

// 在测试中使用
test('should display user name', () => {
  const user = createUser({ name: 'Jane Smith' });
  render(<UserProfile user={user} />);
  expect(screen.getByText('Jane Smith')).toBeInTheDocument();
});

// 使用 fixtures 文件夹管理复杂数据
// __fixtures__/complexData.js
export const complexData = { /* ... */ };
```
