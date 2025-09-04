# 设计模式与应用

设计模式是软件工程中解决常见问题的可复用解决方案。在前端开发中，合理运用设计模式能够提升代码的可维护性、可扩展性和可读性。

## 创建型模式

### 1. 工厂模式 (Factory Pattern)

```javascript
// 简单工厂
class ButtonFactory {
  static createButton(type) {
    switch (type) {
      case 'primary':
        return new PrimaryButton();
      case 'secondary':
        return new SecondaryButton();
      default:
        throw new Error('Unknown button type');
    }
  }
}

// 使用
const button = ButtonFactory.createButton('primary');
```

### 2. 单例模式 (Singleton Pattern)

```javascript
class AppConfig {
  static instance = null;
  
  constructor() {
    if (AppConfig.instance) {
      return AppConfig.instance;
    }
    this.config = {};
    AppConfig.instance = this;
  }
  
  static getInstance() {
    if (!AppConfig.instance) {
      AppConfig.instance = new AppConfig();
    }
    return AppConfig.instance;
  }
}

// 使用
const config1 = AppConfig.getInstance();
const config2 = AppConfig.getInstance();
console.log(config1 === config2); // true
```

### 3. 建造者模式 (Builder Pattern)

```javascript
class QueryBuilder {
  constructor() {
    this.query = {
      select: [],
      where: [],
      orderBy: [],
      limit: null
    };
  }
  
  select(fields) {
    this.query.select = fields;
    return this;
  }
  
  where(condition) {
    this.query.where.push(condition);
    return this;
  }
  
  build() {
    return this.query;
  }
}

// 使用
const query = new QueryBuilder()
  .select(['id', 'name'])
  .where('age > 18')
  .build();
```

## 结构型模式

### 1. 适配器模式 (Adapter Pattern)

```javascript
// 旧接口
class OldAPI {
  request() {
    return { data: 'old format' };
  }
}

// 适配器
class APIAdapter {
  constructor(oldAPI) {
    this.oldAPI = oldAPI;
  }
  
  fetch() {
    const oldData = this.oldAPI.request();
    return this.transformData(oldData);
  }
  
  transformData(data) {
    return {
      result: data.data,
      timestamp: Date.now()
    };
  }
}

// 使用
const adapter = new APIAdapter(new OldAPI());
const newData = adapter.fetch();
```

### 2. 装饰器模式 (Decorator Pattern)

```javascript
// 基础组件
class Component {
  operation() {
    return 'Component';
  }
}

// 装饰器基类
class Decorator {
  constructor(component) {
    this.component = component;
  }
  
  operation() {
    return this.component.operation();
  }
}

// 具体装饰器
class LogDecorator extends Decorator {
  operation() {
    const result = super.operation();
    console.log('Operation result:', result);
    return result;
  }
}

// 使用
const component = new Component();
const decorated = new LogDecorator(component);
decorated.operation();
```

### 3. 代理模式 (Proxy Pattern)

```javascript
// 真实对象
class ImageLoader {
  load(url) {
    console.log('Loading image from:', url);
    return `Image data from ${url}`;
  }
}

// 代理
class ImageProxy {
  constructor() {
    this.cache = new Map();
    this.realLoader = new ImageLoader();
  }
  
  load(url) {
    if (this.cache.has(url)) {
      console.log('Returning cached image');
      return this.cache.get(url);
    }
    
    const imageData = this.realLoader.load(url);
    this.cache.set(url, imageData);
    return imageData;
  }
}

// 使用
const proxy = new ImageProxy();
proxy.load('image1.jpg'); // 真实加载
proxy.load('image1.jpg'); // 从缓存返回
```

## 行为型模式

### 1. 观察者模式 (Observer Pattern)

```javascript
class Observable {
  constructor() {
    this.observers = [];
  }
  
  subscribe(observer) {
    this.observers.push(observer);
  }
  
  unsubscribe(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }
  
  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Observer {
  update(data) {
    console.log('Received data:', data);
  }
}

// 使用
const observable = new Observable();
const observer1 = new Observer();
const observer2 = new Observer();

observable.subscribe(observer1);
observable.subscribe(observer2);
observable.notify('Hello World!');
```

### 2. 策略模式 (Strategy Pattern)

```javascript
// 策略接口
class PaymentStrategy {
  pay(amount) {
    throw new Error('Method not implemented');
  }
}

// 具体策略
class CreditCardPayment extends PaymentStrategy {
  pay(amount) {
    console.log(`Paid $${amount} with Credit Card`);
  }
}

class PayPalPayment extends PaymentStrategy {
  pay(amount) {
    console.log(`Paid $${amount} with PayPal`);
  }
}

// 上下文
class PaymentContext {
  constructor(strategy) {
    this.strategy = strategy;
  }
  
  setStrategy(strategy) {
    this.strategy = strategy;
  }
  
  executePayment(amount) {
    this.strategy.pay(amount);
  }
}

// 使用
const context = new PaymentContext(new CreditCardPayment());
context.executePayment(100);
context.setStrategy(new PayPalPayment());
context.executePayment(200);
```

### 3. 状态模式 (State Pattern)

```javascript
class TrafficLight {
  constructor() {
    this.states = {
      red: new RedState(this),
      yellow: new YellowState(this),
      green: new GreenState(this)
    };
    this.currentState = this.states.red;
  }
  
  changeState(state) {
    this.currentState = this.states[state];
  }
  
  request() {
    this.currentState.handle();
  }
}

class State {
  constructor(light) {
    this.light = light;
  }
  
  handle() {
    throw new Error('Method not implemented');
  }
}

class RedState extends State {
  handle() {
    console.log('Red light - Stop');
    setTimeout(() => this.light.changeState('green'), 3000);
  }
}

class GreenState extends State {
  handle() {
    console.log('Green light - Go');
    setTimeout(() => this.light.changeState('yellow'), 5000);
  }
}

// 使用
const trafficLight = new TrafficLight();
trafficLight.request(); // 开始状态流转
```

## 前端特定模式

### 1. 组件组合模式

```javascript
// React示例
const Card = ({ children }) => (
  <div className="card">{children}</div>
);

const CardHeader = ({ title }) => (
  <div className="card-header">{title}</div>
);

const CardBody = ({ content }) => (
  <div className="card-body">{content}</div>
);

// 组合使用
const App = () => (
  <Card>
    <CardHeader title="My Card" />
    <CardBody content="Card content" />
  </Card>
);
```

### 2. 渲染属性模式 (Render Props)

```javascript
class DataFetcher extends React.Component {
  state = { data: null, loading: true, error: null };
  
  async componentDidMount() {
    try {
      const response = await fetch(this.props.url);
      const data = await response.json();
      this.setState({ data, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  }
  
  render() {
    return this.props.children(this.state);
  }
}

// 使用
<DataFetcher url="/api/data">
  {({ data, loading, error }) => (
    loading ? <Spinner /> : error ? <Error /> : <DataDisplay data={data} />
  )}
</DataFetcher>
```

### 3. 高阶组件模式 (HOC)

```javascript
const withLoading = (WrappedComponent) => {
  return class extends React.Component {
    state = { loading: true };
    
    async componentDidMount() {
      // 模拟数据加载
      await new Promise(resolve => setTimeout(resolve, 1000));
      this.setState({ loading: false });
    }
    
    render() {
      return this.state.loading ? 
        <div>Loading...</div> : 
        <WrappedComponent {...this.props} />;
    }
  };
};

// 使用
const EnhancedComponent = withLoading(MyComponent);
```

## 性能优化模式

### 1. 记忆化模式 (Memoization)

```javascript
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// 使用
const expensiveCalculation = memoize((a, b) => {
  console.log('Calculating...');
  return a * b;
});

expensiveCalculation(2, 3); // 计算并缓存
expensiveCalculation(2, 3); // 从缓存返回
```

### 2. 懒加载模式

```javascript
class LazyLoader {
  constructor(loader) {
    this.loader = loader;
    this.value = null;
    this.loaded = false;
  }
  
  get() {
    if (!this.loaded) {
      this.value = this.loader();
      this.loaded = true;
    }
    return this.value;
  }
}

// 使用
const heavyModule = new LazyLoader(() => import('./heavy-module'));
// 只有在调用 heavyModule.get() 时才会加载
```

## 最佳实践指南

### 1. 模式选择原则

| 场景 | 推荐模式 | 原因 |
|------|----------|------|
| 对象创建复杂 | 建造者模式 | 简化对象构建过程 |
| 需要全局访问点 | 单例模式 | 确保唯一实例 |
| 系统状态变化 | 状态模式 | 清晰的状态转换逻辑 |
| 算法可替换 | 策略模式 | 灵活切换算法实现 |

### 2. 前端应用建议

1. **组件开发**：优先使用组合模式而非继承
2. **状态管理**：观察者模式适合事件驱动的UI更新
3. **性能优化**：合理使用代理模式和记忆化模式
4. **代码复用**：装饰器模式和高阶组件提升代码复用性

### 3. 反模式警示

1. **过度设计**：避免在不必要的场景使用复杂模式
2. **模式滥用**：不要为了使用模式而使用模式
3. **性能考虑**：某些模式可能带来性能开销，需权衡利弊
