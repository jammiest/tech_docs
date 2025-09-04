# E2E 测试实践

端到端（E2E）测试是验证整个应用程序流程是否正常工作的重要测试类型。本指南将详细介绍 E2E 测试的工具选择、实施策略和最佳实践。

## 工具选择与比较

### 1. 主流 E2E 测试框架对比

| 工具 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Cypress** | 实时重载、时间旅行、调试友好 | 不支持多标签页、浏览器限制 | 中小型应用、开发体验优先 |
| **Playwright** | 多浏览器支持、多上下文、自动等待 | 学习曲线较陡 | 大型应用、跨浏览器测试 |
| **Selenium** | 生态丰富、语言支持多、企业级 | 配置复杂、速度较慢 | 传统企业应用、多语言团队 |
| **TestCafe** | 无依赖安装、内置等待机制 | 社区相对较小 | 简单项目、快速上手 |

### 2. 工具选择建议

```javascript
// 根据项目需求选择工具
const chooseE2ETool = (projectRequirements) => {
  const { 
    browserSupport, 
    teamSize, 
    ciIntegration, 
    debuggingNeeds 
  } = projectRequirements;

  if (debuggingNeeds && browserSupport === 'chromium-only') {
    return 'Cypress'; // 优秀的调试体验
  }

  if (browserSupport === 'multi-browser' && teamSize > 10) {
    return 'Playwright'; // 跨浏览器和企业级支持
  }

  if (ciIntegration && simplicity) {
    return 'TestCafe'; // 简单的 CI 集成
  }

  return 'Cypress'; // 默认选择
};
```

## Cypress 实践指南

### 1. 安装与配置

```bash
# 安装 Cypress
npm install cypress --save-dev

# 初始化 Cypress
npx cypress open

# 在 package.json 中添加脚本
{
  "scripts": {
    "cypress:open": "cypress open",
    "cypress:run": "cypress run",
    "cypress:ci": "cypress run --headless"
  }
}
```

### 2. 基础配置文件

```javascript
// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    setupNodeEvents(on, config) {
      // 在这里实现节点事件监听器
    },
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    supportFile: 'cypress/support/e2e.js'
  },
  
  viewportWidth: 1280,
  viewportHeight: 720,
  defaultCommandTimeout: 10000,
  requestTimeout: 10000,
  responseTimeout: 60000,
  
  // 环境变量
  env: {
    API_URL: 'http://localhost:8080/api',
    USER_EMAIL: 'test@example.com',
    USER_PASSWORD: 'password123'
  },
  
  // 重试配置
  retries: {
    runMode: 2,    // CI 运行时的重试次数
    openMode: 0    // 交互模式下的重试次数
  }
});
```

### 3. 测试文件结构

```
cypress/
├── e2e/
│   ├── auth/
│   │   ├── login.cy.js
│   │   └── registration.cy.js
│   ├── dashboard/
│   │   ├── navigation.cy.js
│   │   └── widgets.cy.js
│   └── api/
│       └── api-tests.cy.js
├── fixtures/
│   ├── users.json
│   └── products.json
├── support/
│   ├── commands.js
│   ├── e2e.js
│   └── utils.js
└── plugins/
    └── index.js
```

### 4. 基础测试示例

```javascript
// cypress/e2e/auth/login.cy.js
describe('Login Functionality', () => {
  beforeEach(() => {
    // 在每个测试前访问登录页面
    cy.visit('/login');
  });

  it('should login with valid credentials', () => {
    // 使用 fixture 数据
    cy.fixture('users').then((users) => {
      const testUser = users.validUser;
      
      cy.get('[data-testid="email-input"]').type(testUser.email);
      cy.get('[data-testid="password-input"]').type(testUser.password);
      cy.get('[data-testid="login-button"]').click();
      
      // 断言重定向到仪表板
      cy.url().should('include', '/dashboard');
      cy.get('[data-testid="welcome-message"]')
        .should('contain', `Welcome, ${testUser.name}`);
    });
  });

  it('should show error with invalid credentials', () => {
    cy.get('[data-testid="email-input"]').type('invalid@example.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="error-message"]')
      .should('be.visible')
      .and('contain', 'Invalid credentials');
  });

  it('should validate required fields', () => {
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="email-error"]')
      .should('be.visible')
      .and('contain', 'Email is required');
      
    cy.get('[data-testid="password-error"]')
      .should('be.visible')
      .and('contain', 'Password is required');
  });
});
```

### 5. 自定义命令

```javascript
// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type(email);
    cy.get('[data-testid="password-input"]').type(password);
    cy.get('[data-testid="login-button"]').click();
    cy.url().should('include', '/dashboard');
  });
});

Cypress.Commands.add('createProduct', (productData) => {
  cy.request({
    method: 'POST',
    url: `${Cypress.env('API_URL')}/products`,
    body: productData,
    headers: {
      Authorization: `Bearer ${Cypress.env('ADMIN_TOKEN')}`
    }
  }).then((response) => {
    expect(response.status).to.eq(201);
    return response.body;
  });
});

Cypress.Commands.add('assertToast', (message) => {
  cy.get('[data-testid="toast-message"]')
    .should('be.visible')
    .and('contain', message);
});

// 使用自定义命令
describe('Using custom commands', () => {
  it('should login and create product', () => {
    cy.login('admin@example.com', 'admin123');
    cy.createProduct({
      name: 'Test Product',
      price: 99.99,
      category: 'electronics'
    });
    cy.assertToast('Product created successfully');
  });
});
```

## Playwright 实践指南

### 1. 安装与配置

```bash
# 安装 Playwright
npm init playwright@latest

# 安装浏览器
npx playwright install

# 安装特定浏览器
npx playwright install chromium firefox webkit
```

### 2. 配置文件

```javascript
// playwright.config.js
const { defineConfig, devices } = require('@playwright/test');

module.exports = defineConfig({
  testDir: './tests',
  timeout: 30000,
  expect: {
    timeout: 5000
  },
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { open: 'never' }],
    ['list']
  ],
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },
  
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] }
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] }
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] }
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] }
    }
  ]
});
```

### 3. 测试示例

```javascript
// tests/auth.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Authentication Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('successful login', async ({ page }) => {
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="password-input"]', 'password123');
    await page.click('[data-testid="login-button"]');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="welcome-message"]'))
      .toContainText('Welcome, Test User');
  });

  test('failed login', async ({ page }) => {
    await page.fill('[data-testid="email-input"]', 'wrong@example.com');
    await page.fill('[data-testid="password-input"]', 'wrongpassword');
    await page.click('[data-testid="login-button"]');
    
    await expect(page.locator('[data-testid="error-message"]'))
      .toBeVisible();
    await expect(page.locator('[data-testid="error-message"]'))
      .toContainText('Invalid credentials');
  });

  test('form validation', async ({ page }) => {
    await page.click('[data-testid="login-button"]');
    
    await expect(page.locator('[data-testid="email-error"]'))
      .toContainText('Email is required');
    await expect(page.locator('[data-testid="password-error"]'))
      .toContainText('Password is required');
  });
});
```

## 测试策略与模式

### 1. 页面对象模式 (Page Object Model)

```javascript
// cypress/support/pages/LoginPage.js
class LoginPage {
  constructor() {
    this.emailInput = '[data-testid="email-input"]';
    this.passwordInput = '[data-testid="password-input"]';
    this.loginButton = '[data-testid="login-button"]';
    this.errorMessage = '[data-testid="error-message"]';
  }

  visit() {
    cy.visit('/login');
    return this;
  }

  fillEmail(email) {
    cy.get(this.emailInput).type(email);
    return this;
  }

  fillPassword(password) {
    cy.get(this.passwordInput).type(password);
    return this;
  }

  submit() {
    cy.get(this.loginButton).click();
    return this;
  }

  shouldShowError(message) {
    cy.get(this.errorMessage)
      .should('be.visible')
      .and('contain', message);
    return this;
  }

  login(email, password) {
    return this.visit()
      .fillEmail(email)
      .fillPassword(password)
      .submit();
  }
}

export default new LoginPage();

// 使用页面对象
import LoginPage from '../support/pages/LoginPage';

describe('Login with Page Object', () => {
  it('should login successfully', () => {
    LoginPage
      .login('test@example.com', 'password123')
      .shouldShowError('Invalid credentials');
  });
});
```

### 2. 数据驱动测试

```javascript
// cypress/fixtures/loginTestData.json
{
  "validLogin": {
    "email": "test@example.com",
    "password": "password123",
    "expected": "/dashboard"
  },
  "invalidLogin": {
    "email": "wrong@example.com",
    "password": "wrongpassword",
    "expected": "Invalid credentials"
  },
  "emptyCredentials": {
    "email": "",
    "password": "",
    "expected": {
      "emailError": "Email is required",
      "passwordError": "Password is required"
    }
  }
}

// 数据驱动测试
import testData from '../fixtures/loginTestData.json';

describe('Data-driven login tests', () => {
  Object.entries(testData).forEach(([testName, testCase]) => {
    it(`should handle ${testName}`, () => {
      cy.visit('/login');
      
      if (testCase.email) {
        cy.get('[data-testid="email-input"]').type(testCase.email);
      }
      
      if (testCase.password) {
        cy.get('[data-testid="password-input"]').type(testCase.password);
      }
      
      cy.get('[data-testid="login-button"]').click();

      if (typeof testCase.expected === 'string') {
        if (testCase.expected.startsWith('/')) {
          cy.url().should('include', testCase.expected);
        } else {
          cy.get('[data-testid="error-message"]')
            .should('contain', testCase.expected);
        }
      } else {
        // 处理对象类型的预期结果
        if (testCase.expected.emailError) {
          cy.get('[data-testid="email-error"]')
            .should('contain', testCase.expected.emailError);
        }
        if (testCase.expected.passwordError) {
          cy.get('[data-testid="password-error"]')
            .should('contain', testCase.expected.passwordError);
        }
      }
    });
  });
});
```

## API 测试集成

### 1. 网络请求拦截

```javascript
// Cypress 网络拦截
describe('API Integration Tests', () => {
  it('should mock API responses', () => {
    // 拦截登录请求
    cy.intercept('POST', '/api/login', {
      statusCode: 200,
      body: {
        success: true,
        user: { id: 1, name: 'Test User' }
      }
    }).as('loginRequest');

    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="login-button"]').click();

    // 等待请求完成并验证
    cy.wait('@loginRequest').then((interception) => {
      expect(interception.request.body).to.deep.equal({
        email: 'test@example.com',
        password: 'password123'
      });
    });
  });

  it('should handle network errors', () => {
    cy.intercept('POST', '/api/login', {
      statusCode: 500,
      body: { error: 'Internal Server Error' }
    });

    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="login-button"]').click();

    cy.get('[data-testid="error-message"]')
      .should('contain', 'Server error');
  });
});
```

### 2. 数据库操作

```javascript
// 测试数据库工具函数
const { query } = require('../../server/database');

Cypress.Commands.add('cleanupTestData', () => {
  cy.task('db:query', 'DELETE FROM users WHERE email LIKE "%test%"');
});

Cypress.Commands.add('createTestUser', (userData) => {
  return cy.task('db:query', `
    INSERT INTO users (email, password, name) 
    VALUES ('${userData.email}', '${userData.password}', '${userData.name}')
    RETURNING *
  `);
});

// 使用数据库操作
describe('User Registration', () => {
  beforeEach(() => {
    cy.cleanupTestData();
  });

  it('should create new user', () => {
    const testUser = {
      email: 'test@example.com',
      password: 'password123',
      name: 'Test User'
    };

    cy.visit('/register');
    cy.get('[data-testid="name-input"]').type(testUser.name);
    cy.get('[data-testid="email-input"]').type(testUser.email);
    cy.get('[data-testid="password-input"]').type(testUser.password);
    cy.get('[data-testid="register-button"]').click();

    // 验证数据库中的用户
    cy.task('db:query', 
      `SELECT * FROM users WHERE email = '${testUser.email}'`
    ).then((users) => {
      expect(users).to.have.length(1);
      expect(users[0].name).to.equal(testUser.name);
    });
  });
});
```

## 性能与可靠性

### 1. 测试性能优化

```javascript
// 并行执行测试
// cypress.config.js
module.exports = defineConfig({
  e2e: {
    experimentalRunAllSpecs: true
  }
});

// 使用 cy.session() 保持登录状态
beforeEach(() => {
  cy.session('user', () => {
    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type(Cypress.env('USER_EMAIL'));
    cy.get('[data-testid="password-input"]').type(Cypress.env('USER_PASSWORD'));
    cy.get('[data-testid="login-button"]').click();
    cy.url().should('include', '/dashboard');
  });
});

// 优化选择器性能
describe('Selector Performance', () => {
  it('should use efficient selectors', () => {
    // 不好: 使用复杂的 CSS 选择器
    cy.get('div.container > form > div.input-group > input[type="email"]');
    
    // 好: 使用数据属性
    cy.get('[data-testid="email-input"]');
    
    // 更好: 使用自定义命令
    cy.getEmailInput();
  });
});
```

### 2. 可靠性最佳实践

```javascript
// 使用重试和超时配置
describe('Reliable Tests', () => {
  it('should handle flaky elements', { retries: 3 }, () => {
    cy.get('[data-testid="loading-spinner"]', { timeout: 10000 })
      .should('not.exist');
    
    cy.get('[data-testid="dynamic-content"]', { timeout: 15000 })
      .should('be.visible');
  });

  it('should use conditional testing', () => {
    // 检查元素是否存在后再操作
    cy.get('body').then(($body) => {
      if ($body.find('[data-testid="modal"]').length) {
        cy.get('[data-testid="close-button"]').click();
      }
    });
  });

  it('should handle network delays', () => {
    cy.intercept('GET', '/api/data', (req) => {
      req.reply({ delay: 2000, fixture: 'data.json' });
    }).as('slowRequest');

    cy.visit('/data-page');
    cy.wait('@slowRequest');
    cy.get('[data-testid="data-content"]').should('be.visible');
  });
});
```

## CI/CD 集成

### 1. GitHub Actions 配置

```yaml
# .github/workflows/e2e-tests.yml
name: E2E Tests
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
        
      - name: Start development server
        run: npm run dev &
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
      
      - name: Wait for server
        run: npx wait-on http://localhost:3000
      
      - name: Run E2E tests
        run: npm run cypress:ci
        env:
          CYPRESS_BASE_URL: http://localhost:3000
          CYPRESS_API_URL: http://localhost:8080
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: cypress-screenshots
          path: cypress/screenshots
      
      - name: Upload videos
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: cypress-videos
          path: cypress/videos
```

### 2. 测试报告与监控

```javascript
// 配置测试报告
// cypress.config.js
module.exports = defineConfig({
  reporter: 'mochawesome',
  reporterOptions: {
    reportDir: 'cypress/reports',
    overwrite: false,
    html: true,
    json: true,
    charts: true
  }
});

// 监控测试性能
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    setupNodeEvents(on, config) {
      on('before:run', (details) => {
        console.log('Test run starting with:', details.totalTests, 'tests');
      });

      on('after:run', (results) => {
        console.log('Test run completed');
        console.log('Total duration:', results.totalDuration);
        console.log('Tests passed:', results.totalPassed);
        console.log('Tests failed:', results.totalFailed);
      });
    }
  }
});
```

## 最佳实践总结

### 1. 测试设计原则

1. **独立性**: 每个测试应该独立运行，不依赖其他测试的状态
2. **确定性**: 测试结果应该一致且可预测
3. **可维护性**: 使用页面对象模式和自定义命令减少重复代码
4. **可读性**: 测试名称和结构应该清晰表达测试意图
5. **可靠性**: 处理异步操作和网络延迟，避免脆弱的测试

### 2. 常见陷阱与解决方案

```javascript
// 反模式: 硬编码等待
cy.wait(5000); // 避免使用固定等待

// 解决方案: 使用条件等待
cy.get('[data-testid="loading"]', { timeout: 10000 })
  .should('not.exist');

// 反模式: 复杂的 CSS 选择器
cy.get('div > form > div > input');

// 解决方案: 使用数据属性
cy.get('[data-testid="email-input"]');

// 反模式: 测试实现细节
cy.window().then((win) => {
  expect(win.store.getState().user.name).toBe('Test');
});

// 解决方案: 测试用户可见的行为
cy.get('[data-testid="user-name"]').should('contain', 'Test');

// 反模式: 不清理测试数据
// 解决方案: 使用 beforeEach 清理
beforeEach(() => {
  cy.cleanupTestData();
});
```

### 3. 测试金字塔实践

```mermaid
graph TD
    A[E2E Tests 10%] -->|用户流程验证| B[关键业务场景]
    C[Integration Tests 20%] -->|组件交互| D[API 集成]
    E[Unit Tests 70%] -->|单个功能验证| F[工具函数/组件]
    
    style A fill:#e1f5fe
    style C fill:#f3e5f5
    style E fill:#f1f8e9
```

```javascript
// 遵循测试金字塔原则
describe('Test Pyramid Implementation', () => {
  // 70% Unit Tests (在组件/单元测试中)
  it('unit test example', () => {
    // 测试单个函数或组件
  });

  // 20% Integration Tests (在集成测试中)
  it('integration test example', () => {
    // 测试组件交互或 API 集成
  });

  // 10% E2E Tests (在 E2E 测试中)
  it('e2e test example', () => {
    // 测试完整用户流程
  });
});
```
