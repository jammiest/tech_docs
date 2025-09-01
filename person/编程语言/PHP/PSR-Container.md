# PSR-Container

PSR-Container 包含两个主要部分：

1.  **PSR-11 (Container Interface)**： 定义了容器应实现的标准接口。
2.  **Container Meta Document**： 解释接口设计背后的目标和理由。

以下是文档的核心内容。

---

### 1. 官方项目地址 & 文档

*   **GitHub 项目地址**: [https://github.com/php-fig/container](https://github.com/php-fig/container)
*   **正式规范文档 (PSR-11)**: [https://www.php-fig.org/psr/psr-11/](https://www.php-fig.org/psr/psr-11/)

---

### 2. 核心目标 (为什么要用 PSR-Container？)

*   **标准化与互操作性**: 让代码（尤其是库和框架）不依赖于某个特定的依赖注入容器实现（如 Laravel 的容器、Symfony 的容器、PHP-DI 等）。只要容器实现了 PSR-11，你的代码就能使用它。
*   **解耦**: 开发者可以自由选择自己喜欢的容器，或者更换容器，而无需重写业务逻辑代码。

---

### 3. 核心接口

PSR-11 主要定义了两个接口：

#### a. `Psr\Container\ContainerInterface`

这是最主要的接口，容器类本身必须实现它。它定义了两个方法：

```php
<?php
namespace Psr\Container;

interface ContainerInterface
{
    /**
     * 通过一个字符串标识符（通常是类或接口名）来查找并返回一个实体。
     *
     * @param string $id 所要查找的实体的标识符。
     *
     * @return mixed 查找到的实体。
     *
     * @throws NotFoundExceptionInterface  如果找不到标识符对应的实体。
     * @throws ContainerExceptionInterface 如果查找过程中发生其他错误。
     */
    public function get($id);

    /**
     * 检查容器中是否拥有给定标识符的实体。
     *
     * @param string $id 所要查找的实体的标识符。
     *
     * @return bool 如果容器能提供这个实体返回 true，否则返回 false。
     */
    public function has($id);
}
```

#### b. `Psr\Container\ContainerExceptionInterface`

这是一个**异常接口**。容器在查找或创建对象过程中遇到**任何错误**时，都应该抛出实现此接口的异常。这包括但不限于：依赖不存在、循环依赖、自动装配失败等。

#### c. `Psr\Container\NotFoundExceptionInterface`

这是一个**异常接口**。当向容器的 `get` 方法请求一个**不存在**的标识符（`$id`）时，必须抛出实现此接口的异常。它是 `ContainerExceptionInterface` 的子接口。

---

### 4. 关键概念解释

*   **标识符 (`$id`)**:
    *   通常是一个**完全限定**的**类名**或**接口名**（例如 `My\App\LoggerInterface`）。
    *   也可以是一个**字符串名称**（例如 `'database'`），但 PSR-11 强烈建议使用类名/接口名以提高互操作性。
    *   它必须是字符串。

*   **`get` 方法**:
    *   它的行为取决于容器的实现。它可能只是简单地返回一个预先准备好的实例（如单例），也可能执行复杂的依赖解析和自动装配来创建一个新的实例。
    *   如果找不到 `$id`，**必须**抛出 `NotFoundExceptionInterface`。
    *   如果在创建过程中出错（例如，某个依赖无法被实例化），**必须**抛出 `ContainerExceptionInterface`。

*   **`has` 方法**:
    *   如果 `has($id)` 返回 `true`，则意味着 `get($id)` 不会抛出 `NotFoundExceptionInterface`。但它不保证 `get($id)` 不会抛出 `ContainerExceptionInterface`（例如，可能存在但创建失败）。

---

### 5. 使用方法示例

假设你有一个日志接口 `LoggerInterface` 和它的实现 `FileLogger`。

**在不了解具体容器的情况下，你的代码可以这样写：**

```php
use Psr\Container\ContainerInterface;

class SomeService
{
    private $logger;

    // 容器通过构造函数注入
    public function __construct(ContainerInterface $container)
    {
        // 使用标准化的 get 方法获取依赖
        // 你的代码只依赖于 PSR 接口，不依赖于任何具体容器
        $this->logger = $container->get(LoggerInterface::class);
    }

    public function doSomething()
    {
        $this->logger->info('Doing something...');
    }
}
```

**而容器的配置工作（如何创建 `FileLogger`）则在应用程序的根部，使用你所选容器的特定语法来完成。**

*   **使用 Laravel 容器的配置示例:**
    ```php
    // 在某个 ServiceProvider 的 register 方法中
    $this->app->bind(LoggerInterface::class, FileLogger::class);
    ```

*   **使用 PHP-DI 容器的配置示例 (配置文件):**
    ```php
    return [
        LoggerInterface::class => DI\create(FileLogger::class)
    ];
    ```

你的业务代码 `SomeService` 无需关心这些差异，它只和 `ContainerInterface` 交互。

---

### 6. 实现了 PSR-11 的常见容器

几乎所有主流的 PHP 容器都实现了 PSR-11，这使得它们可以互换使用：
*   **Laravel Framework Container**
*   **Symfony Framework DependencyInjection Component**
*   **PHP-DI**
*   **Aura.Di**
*   **League Container**

### 总结

| 方面 | 描述 |
| :--- | :--- |
| **官方名称** | PSR-11 Container |
| **核心目标** | 为依赖注入容器提供标准化接口，实现框架/库与特定容器实现的解耦。 |
| **核心接口** | `ContainerInterface` (包含 `get($id)` 和 `has($id)` 方法) |
| **核心异常** | `ContainerExceptionInterface` (任何错误), `NotFoundExceptionInterface` (未找到条目) |
| **如何使用** | 在代码中类型提示 `Psr\Container\ContainerInterface`，并使用 `get()` 方法获取依赖。 |
| **最大好处** | **可移植性**和**互操作性**。你的代码不再被绑定到某个特定的框架或容器上。 |

要使用它，你只需要在项目中通过 Composer 安装 `psr/container` 包，并在你的代码中引用这些接口即可。具体的容器实现（如 `symfony/dependency-injection` 或 `php-di/php-di`）会提供 `ContainerInterface` 的实际实现。