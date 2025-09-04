# 单元测试策略

单元测试是软件开发中确保代码质量的重要手段。本文档将详细介绍前端单元测试的策略、工具和最佳实践。

## 测试框架选择

### 1. 主流测试框架对比

| 框架 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| Jest | 开箱即用、快照测试、Mock功能强大 | 配置灵活性较低 | React、大型项目 |
| Vitest | 速度快、Vite生态、TypeScript友好 | 生态相对较新 | Vite项目、中小型应用 |
| Mocha | 灵活、可扩展、生态丰富 | 需要额外配置断言库 | Node.js、需要高度定制 |
| Jasmine | 内置断言、BDD风格 | 功能相对简单 | 简单项目、BDD爱好者 |

### 2. 测试框架配置示例

```javascript
// Jest 配置 (jest.config.js)
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/test/setup.js'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss)$': 'identity-obj-proxy'
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};

// Vitest 配置 (vite.config.js)
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.js',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'src/test/']
    }
  }
});
```

## 测试结构设计

### 1. 测试文件组织

```
src/
├── components/
│   ├── Button/
│   │   ├── index.jsx
│   │   ├── Button.test.jsx
│   │   └── __snapshots__/
│   └── Form/
│       ├── index.jsx
│       └── Form.test.jsx
├── hooks/
│   ├── useApi.js
│   └── useApi.test.js
├── utils/
│   ├── helpers.js
│   └── helpers.test.js
└── test/
    ├── setup.js
    ├── mocks/
    └── __fixtures__/
```

### 2. 测试用例结构

```javascript
// 标准的测试用例结构
describe('Component/Function Name', () => {
  // 前置准备
  beforeEach(() => {
    // 初始化操作
  });

  afterEach(() => {
    // 清理操作
  });

  // 测试用例分组
  describe('when condition A', () => {
    it('should behave correctly', () => {
      // 测试逻辑
    });
  });

  describe('when condition B', () => {
    it('should handle error', () => {
      // 错误处理测试
    });
  });
});

// BDD 风格测试
describe('User authentication', () => {
  context('with valid credentials', () => {
    it('should return user token', async () => {
      // 测试逻辑
    });
  });

  context('with invalid credentials', () => {
    it('should throw authentication error', async () => {
      // 错误测试
    });
  });
});
```

## 组件测试策略

### 1. React 组件测试

```javascript
// Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Button from './Button';

describe('Button', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('calls onClick handler when clicked', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('displays loading state', () => {
    render(<Button loading>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });

  it('matches snapshot', () => {
    const { container } = render(<Button variant="primary">Click me</Button>);
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

### 2. Vue 组件测试

```javascript
// Button.spec.js
import { mount } from '@vue/test-utils';
import Button from './Button.vue';

describe('Button', () => {
  it('renders with correct text', () => {
    const wrapper = mount(Button, {
      slots: {
        default: 'Click me'
      }
    });
    expect(wrapper.text()).toContain('Click me');
  });

  it('emits click event', async () => {
    const wrapper = mount(Button);
    await wrapper.trigger('click');
    expect(wrapper.emitted().click).toHaveLength(1);
  });

  it('applies correct classes based on props', () => {
    const wrapper = mount(Button, {
      props: {
        variant: 'primary',
        size: 'large'
      }
    });
    expect(wrapper.classes()).toContain('button--primary');
    expect(wrapper.classes()).toContain('button--large');
  });
});
```

## 工具函数测试

### 1. 纯函数测试

```javascript
// utils.test.js
import { formatDate, debounce, deepClone } from './utils';

describe('formatDate', () => {
  it('formats date correctly', () => {
    const date = new Date('2023-01-01');
    expect(formatDate(date)).toBe('2023-01-01');
  });

  it('handles invalid date', () => {
    expect(formatDate('invalid')).toBe('Invalid Date');
  });
});

describe('debounce', () => {
  jest.useFakeTimers();

  it('delays function execution', () => {
    const mockFn = jest.fn();
    const debounced = debounce(mockFn, 1000);

    debounced();
    expect(mockFn).not.toHaveBeenCalled();

    jest.advanceTimersByTime(1000);
    expect(mockFn).toHaveBeenCalledTimes(1);
  });

  it('cancels previous call', () => {
    const mockFn = jest.fn();
    const debounced = debounce(mockFn, 1000);

    debounced();
    debounced();
    debounced();

    jest.advanceTimersByTime(1000);
    expect(mockFn).toHaveBeenCalledTimes(1);
  });
});
```

### 2. 异步函数测试

```javascript
// api.test.js
import { fetchUser, retry } from './api';

describe('fetchUser', () => {
  beforeEach(() => {
    global.fetch = jest.fn();
  });

  it('fetches user data successfully', async () => {
    const mockUser = { id: 1, name: 'John' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve(mockUser)
    });

    const user = await fetchUser(1);
    expect(user).toEqual(mockUser);
    expect(fetch).toHaveBeenCalledWith('/api/users/1');
  });

  it('throws error on failed request', async () => {
    fetch.mockResolvedValueOnce({
      ok: false,
      status: 404
    });

    await expect(fetchUser(999)).rejects.toThrow('User not found');
  });
});

describe('retry', () => {
  it('retries failed operations', async () => {
    const mockFn = jest.fn()
      .mockRejectedValueOnce(new Error('Failed'))
      .mockResolvedValueOnce('Success');

    const result = await retry(mockFn, 2);
    expect(result).toBe('Success');
    expect(mockFn).toHaveBeenCalledTimes(2);
  });

  it('throws error after max retries', async () => {
    const mockFn = jest.fn().mockRejectedValue(new Error('Failed'));

    await expect(retry(mockFn, 3)).rejects.toThrow('Failed');
    expect(mockFn).toHaveBeenCalledTimes(3);
  });
});
```

## 自定义 Hooks 测试

### 1. React Hooks 测试

```javascript
// useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('resets count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(5);
  });
});
```

### 2. 复杂 Hooks 测试

```javascript
// useApi.test.js
import { renderHook, waitFor } from '@testing-library/react';
import useApi from './useApi';

describe('useApi', () => {
  beforeEach(() => {
    global.fetch = jest.fn();
  });

  it('fetches data successfully', async () => {
    const mockData = { id: 1, name: 'Test' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve(mockData)
    });

    const { result } = renderHook(() => useApi('/api/test'));

    expect(result.current.loading).toBe(true);
    expect(result.current.data).toBeNull();

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.data).toEqual(mockData);
      expect(result.current.error).toBeNull();
    });
  });

  it('handles fetch errors', async () => {
    fetch.mockRejectedValueOnce(new Error('Network error'));

    const { result } = renderHook(() => useApi('/api/test'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.error).toBe('Network error');
      expect(result.current.data).toBeNull();
    });
  });
});
```

## Mock 策略

### 1. Jest Mock 技术

```javascript
// 函数 Mock
jest.mock('./api', () => ({
  fetchUser: jest.fn().mockResolvedValue({ id: 1, name: 'John' })
}));

// 模块 Mock
jest.mock('axios', () => ({
  get: jest.fn(),
  post: jest.fn()
}));

// 部分 Mock
jest.mock('./utils', () => {
  const original = jest.requireActual('./utils');
  return {
    ...original,
    expensiveOperation: jest.fn()
  };
});

// Mock 实现控制
const mockFn = jest.fn()
  .mockReturnValueOnce('first call')
  .mockReturnValueOnce('second call')
  .mockReturnValue('default value');

// Mock 定时器
jest.useFakeTimers();
jest.advanceTimersByTime(1000);
```

### 2. 依赖注入 Mock

```javascript
// 使用依赖注入进行测试
const createUserService = (apiClient = defaultApiClient) => {
  return {
    getUser: (id) => apiClient.get(`/users/${id}`)
  };
};

// 测试时注入 Mock
test('getUser uses api client', () => {
  const mockApiClient = {
    get: jest.fn().mockResolvedValue({})
  };
  
  const userService = createUserService(mockApiClient);
  userService.getUser(1);
  
  expect(mockApiClient.get).toHaveBeenCalledWith('/users/1');
});
```

## 测试覆盖率

### 1. 覆盖率配置

```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
    '!src/**/__tests__/**',
    '!src/**/__mocks__/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/components/': {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  },
  coverageReporters: ['text', 'lcov', 'html']
};

// package.json 脚本
{
  "scripts": {
    "test:coverage": "jest --coverage",
    "test:watch": "jest --watch",
    "test:ci": "jest --ci --coverage"
  }
}
```

### 2. 覆盖率优化策略

```javascript
// 忽略不必要的测试覆盖
/* istanbul ignore next */
const fallbackFunction = () => {
  // 这个函数很少被执行，忽略测试覆盖
};

// 强制覆盖特定分支
test('covers edge case', () => {
  const result = processInput(null); // 测试 null 输入
  expect(result).toBe('default');
});

// 使用 coverage 注释
/* coverage ignore start */
if (process.env.NODE_ENV === 'development') {
  // 开发环境代码，忽略测试覆盖
}
/* coverage ignore end */
```

## 持续集成配置

### 1. GitHub Actions 配置

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage --ci
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
```

### 2. 质量门禁配置

```javascript
// package.json
{
  "scripts": {
    "test:quality": "npm test && npm run lint",
    "prepush": "npm run test:quality",
    "precommit": "npm run test:unit"
  }
}

// 使用 Husky 进行 Git Hooks 验证
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run test:unit && npm run lint",
      "pre-push": "npm run test:quality"
    }
  }
}
```

## 性能优化

### 1. 测试性能优化

```javascript
// jest.config.js
module.exports = {
  // 并行执行测试
  maxWorkers: '50%',
  maxConcurrent: 5,
  
  // 测试文件隔离
  isolate: false,
  
  // 缓存配置
  cache: true,
  cacheDirectory: '/tmp/jest',
  
  // 测试超时设置
  testTimeout: 10000
};

// 优化大型测试套件
describe('Large test suite', () => {
  // 共享设置
  beforeAll(() => {
    // 一次性初始化
  });
  
  // 使用 test.each 进行参数化测试
  test.each([
    [1, 2, 3],
    [4, 5, 9],
    [10, 20, 30]
  ])('adds %i + %i to equal %i', (a, b, expected) => {
    expect(a + b).toBe(expected);
  });
});
```

### 2. 虚拟 DOM 优化

```javascript
// 使用适当的渲染方法
import { render, screen } from '@testing-library/react';

// 对于简单组件测试
test('renders button', () => {
  render(<Button>Click me</Button>);
  expect(screen.getByRole('button')).toBeInTheDocument();
});

// 避免不必要的渲染
test('does not re-render unnecessarily', () => {
  const { rerender } = render(<Component prop="value" />);
  
  // 使用相同的 props 重新渲染
  rerender(<Component prop="value" />);
  
  // 验证没有不必要的渲染
  expect(renderCount).toBe(1);
});
```

## 最佳实践总结

### 1. 测试编写原则

1. **FIRST 原则**:
   - Fast: 测试要快速运行
   - Independent: 测试之间要独立
   - Repeatable: 测试要可重复
   - Self-validating: 测试要自验证
   - Timely: 及时编写测试

2. **3A 模式**:
   - Arrange: 准备测试数据
   - Act: 执行被测代码
   - Assert: 验证结果

3. **测试优先级**:
   - 核心业务逻辑 → 高优先级
   - 工具函数 → 中优先级
   - UI 组件 → 低优先级

### 2. 常见反模式

```javascript
// 反模式: 测试实现细节
test('updates state correctly', () => {
  const component = render(<Component />);
  // 不要测试内部状态
  expect(component.instance().state.value).toBe('test');
});

// 正确: 测试行为
test('displays correct value', () => {
  render(<Component />);
  expect(screen.getByText('test')).toBeInTheDocument();
});

// 反模式: 过于复杂的测试
test('does everything', () => {
  // 一个测试验证太多东西
});

// 正确: 单一职责测试
test('handles user input', () => { /* ... */ });
test('validates data', () => { /* ... */ });
test('submits form', () => { /* ... */ });
```

### 3. 测试维护策略

```javascript
// 使用自定义匹配器
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    return {
      message: () => `expected ${received} to be within range ${floor}-${ceiling}`,
      pass
    };
  }
});

// 创建测试工具函数
const renderWithProviders = (ui, options = {}) => {
  const store = createTestStore(options.initialState);
  return {
    ...render(ui, { wrapper: Provider, props: { store } }),
    store
  };
};

// 使用测试数据工厂
const createUser = (overrides = {}) => ({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  ...overrides
});

test('displays user name', () => {
  const user = createUser({ name: 'Jane Smith' });
  render(<UserProfile user={user} />);
  expect(screen.getByText('Jane Smith')).toBeInTheDocument();
});
```
