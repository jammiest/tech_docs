# 常见的设计模式

PHP 中常见的设计模式可以分为 **创建型**、**结构型** 和 **行为型** 三大类

## 创建型模式（Creational Patterns）

### 单例模式（Singleton）

作用：确保一个类只有一个实例，并提供全局访问点。

适用场景：数据库连接、日志管理、配置加载等。

```php
class Database {
    private static $instance;
    private function __construct() {} // 防止外部实例化
    public static function getInstance() {
        if (!self::$instance) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}
$db = Database::getInstance();

```


### 工厂模式（Factory）

作用：通过工厂类统一创建对象，隐藏具体实现。

变体：

简单工厂：一个工厂类直接创建不同对象。

工厂方法：子类决定实例化哪个类。

```php
interface Logger {
    public function log($message);
}
class FileLogger implements Logger {
    public function log($message) { /* 写入文件 */ }
}
class DatabaseLogger implements Logger {
    public function log($message) { /* 写入数据库 */ }
}
class LoggerFactory {
    public static function createLogger($type) {
        return match ($type) {
            'file' => new FileLogger(),
            'db' => new DatabaseLogger(),
        };
    }
}
$logger = LoggerFactory::createLogger('file');
```

### 抽象工厂模式（Abstract Factory）

作用：创建一组相关或依赖的对象（家族）。

示例：创建不同风格的 UI 组件（按钮、输入框等）。

### 建造者模式（Builder）

作用：分步骤构建复杂对象，分离构造与表示。

适用场景：SQL 查询构造器、HTTP 请求构建等。

### 原型模式（Prototype）

作用：通过克隆现有对象来创建新对象。

示例：clone 关键字实现深拷贝/浅拷贝。

### 结构型模式（Structural Patterns）
适配器模式（Adapter）

作用：将不兼容的接口转换为可兼容的接口。

示例：将第三方支付接口适配为统一支付接口。

### 装饰器模式（Decorator）

作用：动态扩展对象功能，避免继承爆炸。

示例：为日志添加时间戳、加密等功能。

```php
interface Logger { /* ... */ }
class BasicLogger implements Logger { /* ... */ }
class TimestampDecorator implements Logger {
    public function __construct(private Logger $logger) {}
    public function log($message) {
        $this->logger->log("[".date('Y-m-d')."] $message");
    }
}
$logger = new TimestampDecorator(new BasicLogger());
```

### 代理模式（Proxy）

作用：控制对对象的访问（如延迟加载、权限校验）。

示例：数据库查询的懒加载代理。

### 组合模式（Composite）

作用：以树形结构处理部分-整体关系（如菜单嵌套）。

### 外观模式（Facade）

作用：为复杂子系统提供统一入口。

示例：封装支付、物流等服务的订单处理门面。

## 行为型模式（Behavioral Patterns）

### 策略模式（Strategy）

作用：定义算法族，使其可互换。

示例：支付方式（支付宝、微信）的动态切换。

```php
interface PaymentStrategy {
    public function pay($amount);
}
class AlipayStrategy implements PaymentStrategy { /* ... */ }
class WechatPayStrategy implements PaymentStrategy { /* ... */ }
class PaymentContext {
    public function __construct(private PaymentStrategy $strategy) {}
    public function execute($amount) {
        $this->strategy->pay($amount);
    }
}
$context = new PaymentContext(new AlipayStrategy());
$context->execute(100);
```

### 观察者模式（Observer）

作用：一对多的依赖关系，主题变更时通知观察者。

示例：事件监听、消息队列。

### 责任链模式（Chain of Responsibility）

作用：将请求沿处理链传递，直到被处理。

示例：中间件管道（如 Laravel 的 Middleware）。

### 命令模式（Command）

作用：将请求封装为对象，支持撤销/重做。

示例：任务队列、事务管理。

### 模板方法模式（Template Method）

作用：定义算法骨架，子类重写特定步骤。

示例：抽象类中的固定流程（如数据库迁移）。

## 其他常见模式

### 依赖注入（DI）与控制反转（IoC）

作用：解耦依赖关系，通过容器管理对象创建。

框架应用：Laravel 的服务容器。

### 仓库模式（Repository）

作用：抽象数据访问层，隔离业务逻辑与数据库。

### 数据映射模式（Data Mapper）

作用：对象与数据库表之间的映射（如 ORM）。

## 设计模式选择原则

优先组合而非继承（Favor Composition Over Inheritance）。

针对接口编程，而非实现。

开闭原则：对扩展开放，对修改关闭。

## 实际应用建议

框架集成：如 Laravel 中的 Container（DI）、Observer（事件）、Pipeline（责任链）。

避免过度设计：简单场景直接编码，复杂场景再引入模式。

## 常用框架

| **设计模式**           | **Laravel**                          | **Symfony**                        | **ThinkPHP**                | **CodeIgniter**            | **Yii**                     |
|------------------------|--------------------------------------|------------------------------------|-----------------------------|----------------------------|-----------------------------|
| **依赖注入 (DI)**       | 服务容器 (`app->make()`)             | 依赖注入组件 (`autowire`)          | 容器管理 (`app()`)           | 手动注入（较少内置支持）     | 依赖注入容器 (`Yii::$container`) |
| **工厂模式**            | 数据库工厂、日志工厂                 | 服务工厂 (`FactoryBundle`)          | 模型/门面静态调用           | 模型/库加载机制            | 对象工厂 (`Yii::createObject()`) |
| **单例模式**            | 服务容器绑定的单例 (`singleton()`)   | 服务配置中的单例 (`shared: true`)   | 单例 Trait (`instance()`)    | 无严格单例，需手动实现      | 单例组件（如 `Yii::$app`）      |
| **观察者模式**          | 事件系统 (`Event` + `Listener`)      | EventDispatcher 组件               | 事件监听 (`Hook`)           | 第三方库扩展实现           | 事件机制 (`on()` + `trigger()`) |
| **策略模式**            | 支付网关、缓存驱动切换               | 安全认证策略 (`Firewall`)           | 多应用路由策略              | 无内置支持                 | 行为（Behavior）组件          |
| **装饰器模式**          | 中间件（请求/响应装饰）              | HTTP Kernel 的装饰层               | 中间件机制                  | 无内置支持                 | 过滤器（Filter）机制          |
| **代理模式**            | 延迟服务加载 (`defer` 属性)          | 懒加载代理（ORM Proxy）             | 模型延迟关联                | 无内置支持                 | ActiveRecord 的关联代理       |
| **责任链模式**          | 中间件管道 (`Middleware Pipeline`)   | HTTP Kernel 的中间件栈             | 中间件队列                  | 无内置支持                 | 行为（Behavior）链           |
| **仓库模式**            | Eloquent Repository（需手动实现）    | Doctrine Repository                | 模型基类 (`Model`)           | 无内置支持                 | ActiveRecord 的 Query        |
| **模板方法模式**        | 数据库迁移抽象类                     | AbstractController 的模板方法      | 控制器基类 (`Controller`)    | 无内置支持                 | 控制器基类 (`Controller`)     |
| **外观模式 (Facade)**   | `Facade` 类（静态代理）              | 无直接等价，但可用 Service Locator | 门面类 (`Facade`)            | 无内置支持                 | 无内置支持                  |
| **适配器模式**          | 缓存/队列驱动适配（Redis, SQS等）    | 缓存适配器 (`CacheAdapter`)        | 驱动适配（数据库/缓存）      | 数据库驱动适配             | 缓存/日志适配器              |
| **数据映射模式**        | Eloquent ORM                         | Doctrine ORM                       | 模型 ORM                    | 无原生 ORM                 | ActiveRecord ORM            |





## 常用设计模式对比表

### 一、创建型模式

| 模式名称       | 定义                          | 解决的问题                  | PHP 框架实现示例                          | 典型使用场景               |
|----------------|-------------------------------|---------------------------|------------------------------------------|--------------------------|
| **单例模式**   | 确保类只有一个全局实例          | 避免重复创建消耗资源的对象  | Laravel 服务容器绑定 `singleton()`       | 数据库连接、日志系统       |
| **工厂模式**   | 通过工厂类创建对象，隐藏实现细节 | 对象创建与使用的解耦        | ThinkPHP 模型静态调用 `User::find(1)`    | 多数据库驱动、支付网关     |
| **建造者模式** | 分步骤构建复杂对象              | 简化复杂对象的创建过程      | PDO 查询构造器、Guzzle HTTP 请求构建     | 复杂参数对象的创建         |

### 二、结构型模式

| 模式名称       | 定义                          | 解决的问题                  | PHP 框架实现示例                          | 典型使用场景               |
|----------------|-------------------------------|---------------------------|------------------------------------------|--------------------------|
| **适配器模式** | 转换接口使其兼容               | 接口不兼容问题              | Laravel 文件系统驱动（本地/S3适配）       | 第三方API接入、旧系统改造  |
| **装饰器模式** | 动态添加功能而不改变原类        | 扩展功能时避免继承爆炸      | Laravel 中间件对请求/响应的装饰           | 数据加密、日志增强         |
| **外观模式**   | 为复杂系统提供统一入口          | 简化子系统调用复杂度        | ThinkPHP 门面类 `Cache::get()`           | SDK封装、复杂流程简化      |

### 三、行为型模式

| 模式名称          | 定义                          | 解决的问题                  | PHP 框架实现示例                          | 典型使用场景               |
|-------------------|-------------------------------|---------------------------|------------------------------------------|--------------------------|
| **观察者模式**    | 一对多的状态通知机制            | 组件间松耦合通信            | Laravel 事件系统 `Event::dispatch()`      | 订单状态变更通知           |
| **策略模式**      | 封装算法族使其可互换            | 避免条件语句的硬编码        | Symfony 安全认证策略切换                 | 支付方式选择、排序算法切换 |
| **责任链模式**    | 将请求沿处理链传递直到被处理      | 动态指定请求处理器          | Laravel 中间件管道                       | 权限校验、请求过滤         |

### 四、特别补充

| 模式名称          | 定义                          | 解决的问题                  | PHP 特殊实现                          | 最佳实践建议               |
|-------------------|-------------------------------|---------------------------|--------------------------------------|--------------------------|
| **依赖注入**      | 通过构造函数/方法注入依赖        | 降低类之间的耦合度          | Laravel 服务容器自动解析              | 控制器构造函数注入         |
| **仓库模式**      | 抽象数据访问层                 | 分离业务逻辑与数据操作      | Eloquent 的 `UserRepository` 封装     | 复杂查询逻辑集中管理       |
| **空对象模式**    | 用空对象替代null检查            | 避免null判断污染代码        | 返回空模型对象 `new NullUser()`       | 未找到数据的默认处理       |

### 框架实现速查

| 模式          | Laravel              | ThinkPHP           | Symfony            | Yii2                |
|--------------|----------------------|--------------------|--------------------|---------------------|
| **DI容器**   | 服务容器              | 容器管理           | 依赖注入组件        | 依赖注入容器         |
| **ORM**      | Eloquent             | 模型基类           | Doctrine           | ActiveRecord        |
| **事件**     | Event/Listener       | Hook系统           | EventDispatcher    | on()/trigger()      |
| **中间件**   | Middleware Pipeline  | 中间件队列         | HTTP Kernel        | 过滤器              |