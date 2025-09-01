# PHP-DI

好的，这是一份详细且实用的 PHP-DI 使用指南，从基础概念到高级用法，旨在帮助您快速上手并有效地在项目中使用这个强大的依赖注入容器。

---

### 1. 什么是 PHP-DI？

**PHP-DI** 是一个适用于 PHP 的、强大且易于使用的**依赖注入容器**。它实现了 **PSR-11** 标准，意味着它可以与其他遵循该标准的框架和库无缝协作。

**核心思想：**
- **控制反转 (IoC)** ：将对象的创建和依赖组装的控制权从类内部转移到外部容器。
- **依赖注入 (DI)** ：通过构造函数、方法或属性等方式，将一个对象所依赖的其他对象“注入”给它，而不是让它自己创建。

**主要优点：**
*   **解耦代码**：类不再负责创建自己的依赖项，只关注自身的职责。
*   **提高可测试性**：可以轻松地将依赖项替换为 Mock 或 Stub，方便进行单元测试。
*   **增强可维护性**：依赖关系在容器中集中配置，更改实现时只需修改配置，无需改动大量业务代码。
*   **自动装配**：容器可以自动解析并注入依赖，大大减少了手动配置的工作量。

---

### 2. 安装

使用 Composer 进行安装：

```bash
composer require php-di/php-di
```

---

### 3. 核心概念与基本用法

#### 创建容器

最简单的方式是使用 `ContainerBuilder`：

```php
<?php
use DI\ContainerBuilder;

$builder = new ContainerBuilder();
$container = $builder->build();
// 现在可以使用 $container 了
```

#### 手动定义依赖（配置）

你可以通过数组或 PHP 代码定义依赖关系。推荐创建一个独立的配置文件（如 `dependencies.php`）。

**使用数组定义：**

```php
// dependencies.php
return [
    // 定义一个接口到实现的映射
    Psr\Logger\LoggerInterface::class => DI\create(MyLogger::class),

    // 定义一个字符串别名到类的映射
    'db.host' => 'localhost',
    'db.port' => 3306,

    // 定义一个复杂的对象，并注入其依赖
    Database::class => DI\autowire()
        ->constructorParameter('host', DI\get('db.host'))
        ->constructorParameter('port', DI\get('db.port')),

    // 使用工厂函数来创建实例
    Cache::class => function (\Psr\Container\ContainerInterface $c) {
        $redis = new Redis();
        $redis->connect($c->get('db.host'), $c->get('db.port'));
        return new RedisCache($redis);
    },
];
```

然后在构建容器时加载这个配置：

```php
$builder = new ContainerBuilder();
$builder->addDefinitions('dependencies.php');
$container = $builder->build();
```

#### 使用容器获取对象

```php
// 获取一个已定义的对象
$database = $container->get(Database::class);

// 获取一个定义的值
$dbHost = $container->get('db.host');

// 即使没有明确定义，PHP-DI 也会尝试通过自动装配创建它
$someService = $container->get(SomeService::class);
```

#### 自动装配 (Autowiring)

这是 PHP-DI 最强大的功能之一。如果容器遇到一个未定义的类，它会尝试：
1.  通过反射检查该类的构造函数参数。
2.  递归地解析这些参数的类型提示。
3.  自动创建所有需要的依赖项并注入。

**例如：**

```php
class Mailer {
    public function __construct() {}
}

class UserManager {
    private $mailer;
    public function __construct(Mailer $mailer) {
        $this->mailer = $mailer;
    }
}

// 无需任何配置！
$userManager = $container->get(UserManager::class);
// PHP-DI 会自动创建 Mailer 并注入到 UserManager 中
```

要启用或禁用自动装配，可以在构建时设置：

```php
$builder->useAutowiring(true); // 默认就是 true
```

---

### 4. 定义依赖的几种方式

#### 1. 使用 `DI\create()`
明确指定如何创建一个类的实例。

```php
return [
    MyClass::class => DI\create()
        ->constructor('arg1', 'arg2') // 传递构造函数参数
        ->method('setLogger', DI\get(LoggerInterface::class)) // 调用方法注入
];
```

#### 2. 使用 `DI\autowire()`
在自动装配的基础上进行微调。

```php
return [
    MyClass::class => DI\autowire()
        // 覆盖自动装配，为特定参数指定值
        ->constructorParameter('paramName', 'specificValue')
        ->method('setSomething', DI\get('some.definition'))
];
```

#### 3. 使用工厂函数 `function (ContainerInterface $c) {}`
最灵活的方式，完全控制对象的创建过程。

```php
return [
    ComplexService::class => function (ContainerInterface $c) {
        $dependency = $c->get(SomeDependency::class);
        $config = $c->get('config');
        return new ComplexService($dependency, $config['api_key']);
    }
];
```

#### 4. 使用现有对象或值
直接返回一个实例或标量值。

```php
return [
    'config.api_key' => 'abc123xyz',
    ExistingObject::class => new ExistingObject(),
];
```

---

### 5. 注入方式

PHP-DI 支持三种主要的注入方式：

#### 1. 构造函数注入 (最推荐)
依赖通过类的构造函数传入。

```php
class UserController {
    private $userRepository;
    public function __construct(UserRepository $userRepository) {
        $this->userRepository = $userRepository;
    }
}
// 容器会自动注入
```

#### 2. Setter 方法注入
通过调用 setter 方法注入。

```php
class OrderService {
    private $logger;
    public function setLogger(LoggerInterface $logger) {
        $this->logger = $logger;
    }
}

// 在配置中定义
return [
    OrderService::class => DI\autowire()
        ->method('setLogger', DI\get(LoggerInterface::class))
];
```

#### 3. 属性注入 (不推荐)
直接给公共属性赋值。这会破坏封装性，应谨慎使用。

```php
class ReportGenerator {
    /**
     * @Inject
     * @var LoggerInterface
     */
    public $logger;
}

// 或者在配置中指定
return [
    ReportGenerator::class => DI\autowire()
        ->property('logger', DI\get(LoggerInterface::class))
];
```

---

### 6. 高级特性

#### 环境特定配置

你可以根据不同环境（开发、生产）加载不同的配置。

```php
$builder = new ContainerBuilder();

if ($isProduction) {
    $builder->addDefinitions('prod.config.php');
    // 生产环境可能启用缓存以提升性能
    $builder->enableCompilation(__DIR__ . '/var/cache');
    $builder->writeProxiesToFile(true, __DIR__ . '/var/proxies');
} else {
    $builder->addDefinitions('dev.config.php');
}

$container = $builder->build();
```

#### 注解支持 (已弃用，推荐使用属性)

PHP-DI 6.0 开始，注解 (`@Inject`, `@Named`) 已被弃用，推荐使用 PHP 原生**属性**。

**首先需要安装注解/属性支持：**

```bash
composer require doctrine/annotations # 如果你仍想用注解（不推荐新项目使用）
composer require php-di/php-di # 6.0+ 已内置属性支持
```

**使用属性：**

```php
use DI\Attribute\Inject;

class UserController {
    public function __construct(
        #[Inject('app.db_host')] // 注入一个名为 'app.db_host' 的条目
        private string $dbHost,
        #[Inject] // 自动注入类型提示的类
        private UserRepository $userRepository
    ) {}
}
```

#### 延迟加载与代理

对于繁重或很少使用的依赖，可以使用**代理**来实现延迟加载。

```php
// 在配置中定义
return [
    HeavyService::class => DI\create(LazyProxy::class) // 示例，具体用法请查阅文档
    // 或者使用 `DI\lazy(fn () => new HeavyService())`
];
// 实际对象只在第一次被调用时才会创建
```

---

### 7. 与框架集成

PHP-DI 可以轻松集成到各种框架中，为其提供更强大的 DI 功能。

#### 与 Slim 框架集成

Slim 官方推荐使用 PHP-DI。

1.  **安装桥接包：**
    ```bash
    composer require slim/slim php-di/php-di slim/psr7
    ```

2.  **创建容器并启动应用：**
    ```php
    use DI\Container;
    use Slim\Factory\AppFactory;

    $container = new Container();
    // ... 配置你的容器定义 ...

    // 将容器实例传递给 AppFactory
    AppFactory::setContainer($container);
    $app = AppFactory::create();

    // 在路由中，现在可以通过类型提示注入依赖
    $app->get('/users', function (Request $request, Response $response, UserRepository $userRepo) {
        // $userRepo 被自动注入！
        $users = $userRepo->findAll();
        // ... 返回响应
    });

    $app->run();
    ```

#### 与其他框架（如 Laravel, Symfony）
这些框架有自己的容器，但你可以通过适配器或特定包将 PHP-DI 融入其生态，不过通常直接使用框架自带的容器是更自然的选择。

---

### 8. 最佳实践

1.  **面向接口编程**：在配置中绑定接口到具体实现，这样更容易切换实现。
    ```php
    LoggerInterface::class => DI\create(FileLogger::class),
    // 想换日志系统？只需改这一行
    // LoggerInterface::class => DI\create(ElasticsearchLogger::class),
    ```

2.  **将配置集中在容器中**：避免在代码中直接使用 `new` 创建依赖，让容器管理所有对象生命周期。

3.  **优先使用构造函数注入**：它明确声明了类的必需依赖，并且保证了对象在创建后即处于完整状态。

4.  **为生产环境启用缓存**：使用 `enableCompilation` 可以显著提升性能。

5.  **保持容器配置的简洁性**：充分利用**自动装配**，只为那些需要特殊处理的类（如接口、带参数的类）编写显式配置。

### 总结

PHP-DI 通过其强大的自动装配和灵活的配置方式，极大地简化了 PHP 应用程序中的依赖管理。
它遵循标准，易于集成，并能显著提高代码的质量和可测试性。
从简单的数组配置到复杂的工厂函数，它提供了各种工具来满足不同场景的需求。