# PHP主流框架介绍

## 主流框架分析

好的，我们来对 PHP 三大主流框架（Laravel, ThinkPHP, Symfony）以及一个高性能框架（Hyperf）的目录结构进行简要的对比分析。这将帮助你快速理解它们的异同和设计哲学。

### 核心设计哲学对比

| 框架         | 设计哲学               | 特点                                       |
| :----------- | :--------------------- | :----------------------------------------- |
| **Laravel**  | **Web 艺术家框架**     | 优雅、表达性强、功能全家桶、开箱即用       |
| **ThinkPHP** | **国内流行，易上手**   | 简单实用、中文友好、文档丰富、符合国习惯   |
| **Symfony**  | **企业级模块化工具箱** | 高度解耦、可复用组件、灵活性极高、稳定性强 |
| **Hyperf**   | **高性能协程框架**     | 基于 Swoole、常驻内存、依赖注入、组件化    |

---

### 目录结构详细对比

为了更直观地对比，我们假设一个典型的 MVC 应用在四个框架中的代码分布：

| 功能/文件      | Laravel                     | ThinkPHP               | Symfony                           | Hyperf                             |
| :------------- | :-------------------------- | :--------------------- | :-------------------------------- | :--------------------------------- |
| **入口文件**   | `public/index.php`          | `public/index.php`     | `public/index.php`                | `bin/hyperf.php`                   |
| **应用配置**   | `config/`                   | `config/`              | `config/`                         | `config/autoload/`                 |
| **路由定义**   | `routes/web.php`            | `route/app.php`        | `config/routes.yaml`              | `config/routes.php`                |
| **控制器**     | `app/Http/Controllers/`     | `app/controller/`      | `src/Controller/`                 | `app/Controller/`                  |
| **模型**       | `app/Models/`               | `app/model/`           | `src/Entity/` + `src/Repository/` | `app/Model/`                       |
| **业务逻辑**   | `app/Services/` (约定)      | `app/service/` (约定)  | `src/Service/` (约定)             | `app/Service/`                     |
| **视图模板**   | `resources/views/`          | `view/`                | `templates/`                      | (API项目通常无)                    |
| **数据迁移**   | `database/migrations/`      | `database/migrations/` | `migrations/`                     | `migrations/`                      |
| **命令行工具** | `app/Console/Commands/`     | `app/command/`         | `src/Command/`                    | `app/Command/`                     |
| **事件监听**   | `app/Listeners/`            | `app/listener/`        | `src/EventListener/`              | `app/Listener/`                    |
| **中间件**     | `app/Http/Middleware/`      | `app/middleware/`      | `src/Middleware/`                 | `app/Middleware/`                  |
| **依赖注入**   | `app/Providers/`            | (容器自动)             | `config/services.yaml`            | `config/autoload/dependencies.php` |
| **静态资源**   | `public/css/`, `public/js/` | `public/static/`       | `public/`                         | `public/`                          |

---

### 各框架目录结构简要分析

#### 1. Laravel (v8+)

```
laravel-app/
├── app/                 # 应用核心代码
│   ├── Models/          # 数据模型
│   ├── Http/            # HTTP相关
│   │   ├── Controllers/ # 控制器
│   │   ├── Middleware/  # 中间件
│   │   └── ...          # 请求、响应等
│   ├── Providers/       # 服务提供者（注册服务、依赖注入）
│   └── Console/         #  artisan命令
├── bootstrap/           # 框架启动引导文件
├── config/              # 配置文件
├── database/            # 数据库相关（迁移、种子、工厂）
├── public/              # Web入口，静态资源
├── resources/           # 原始资源（视图、未编译的JS/CSS）
├── routes/              # 路由定义（web, api, console等）
├── storage/             # 运行时文件（日志、缓存、编译后的视图等）
├── vendor/              # Composer依赖
└── tests/               # 测试用例
```
**特点**：结构清晰但略重，`app/` 目录下子目录较多，体现了其“全家桶”式的设计。

#### 2. ThinkPHP (v6+)

```
thinkphp-app/
├── app/                 # 应用目录（可以有多应用）
│   ├── controller/      # 控制器
│   ├── model/           # 模型
│   ├── service/         # 服务层（约定）
│   ├── middleware/      # 中间件
│   └── ...              # 其他如event, listener等
├── config/              # 配置文件
├── public/              # Web入口，静态资源
├── route/               # 路由定义
├── runtime/             # 运行时目录（日志、缓存）
├── vendor/              # Composer依赖
└── extend/              # 扩展类库目录
```
**特点**：结构简单直观，符合国内开发者习惯。**多应用模式**是其一大特色，可以在一个`app/`目录下创建多个独立应用（如 `app/admin/`, `app/index/`）。

#### 3. Symfony (v5+)

```
symfony-app/
├── bin/                 # 可执行文件，如console
├── config/              # 配置（路由、服务、包等）
├── public/              # Web入口
├── src/                 # **你的所有PHP代码都在这里**
│   ├── Controller/      # 控制器
│   ├── Entity/          #  Doctrine实体（≈模型）
│   ├── Repository/      #  数据仓库层
│   ├── Service/         #  业务服务
│   └── ...              #  其他如Event, Command等
├── templates/           #  Twig模板
├── translations/        #  翻译文件
├── var/                 #  运行时文件（缓存、日志）
├── vendor/              #  Composer依赖
└── migrations/          #  数据库迁移
```
**特点**：**极其规范**。所有自定义PHP代码严格放在 `src/` 目录下，与 `vendor/` 的第三方代码完全分离。配置通常使用YAML、XML格式，严谨且强大。

#### 4. Hyperf (v2+)

```
hyperf-app/
├── app/                 # 应用核心代码（PSR-4）
│   ├── Controller/      # 控制器
│   ├── Model/           # 模型
│   ├── Service/         # 服务层
│   ├── Listener/        # 事件监听器
│   └── ...              # Process, Task, Job等
├── bin/                 # 入口文件，CLI模式启动
│   └── hyperf.php       # Swoole服务器入口
├── config/              # 配置中心
│   └── autoload/        # 所有自动加载的配置
├── runtime/             # 运行时文件（AOP代理类、日志）
├── test/                # 测试用例
└── vendor/              # Composer依赖
```
**特点**：**为高性能和CLI设计**。`bin/hyperf.php` 是Swoole服务器入口而非传统的 `public/index.php`。配置集中在 `config/autoload/` 下。目录结构鼓励清晰的分层和解耦，并包含了大量面向常驻内存和异步编程的特性目录（如 `Process`, `Task`）。

### 总结与选择建议

| 框架         | 适合场景                                  | 学习曲线 | 结构特点                 |
| :----------- | :---------------------------------------- | :------- | :----------------------- |
| **Laravel**  | 快速开发全功能Web应用（前后端）           | 中等     | 功能全面，结构略重但规范 |
| **ThinkPHP** | 需要快速上手或国内政府、企业项目          | 简单     | 简单直观，支持多应用     |
| **Symfony**  | 大型复杂企业级系统，需要高度定制和稳定性  | 较陡峭   | 极其规范，模块化，灵活   |
| **Hyperf**   | API中间件、微服务、游戏服务器等高性能场景 | 中等偏上 | 为性能和异步而生，组件化 |

**核心共通点**：
所有现代框架都遵循类似的 **MVC（模型-视图-控制器）** 模式思想，因此它们的核心目录（控制器、模型、配置、路由）的功能是相似的，只是命名和组织方式略有不同。理解了其中一个，再学习其他的会非常容易。选择时，应根据项目需求、团队背景和性能要求来决定。

## 简要总结


### 一、核心设计哲学对比

| 框架         | 核心哲学               | 一句话形容                             |
| :----------- | :--------------------- | :------------------------------------- |
| **Laravel**  | **Web 艺术家框架**     | “为开发者提供极致愉悦体验的全家桶”     |
| **ThinkPHP** | **简单实用，中文友好** | “最懂中国开发者的快速开发利器”         |
| **Symfony**  | **企业级模块化工具箱** | “构建复杂、稳定、可复用系统的瑞士军刀” |
| **Hyperf**   | **高性能协程框架**     | “为 Swoole 和高并发场景而生的现代引擎” |

---

### 二、结构共通点：MVC 模式

所有现代 PHP 框架都遵循 **MVC (Model-View-Controller)** 或其变体的设计模式，这是它们最根本的共通点。

| 组件                    | 职责                         | 在各框架中的常见位置                                          |
| :---------------------- | :--------------------------- | :------------------------------------------------------------ |
| **模型 (Model)**        | 处理数据逻辑，与数据库交互   | `app/Models/`, `app/model/`, `src/Entity/`                    |
| **视图 (View)**         | 展示数据，用户界面           | `resources/views/`, `view/`, `templates/`                     |
| **控制器 (Controller)** | 处理用户请求，协调模型和视图 | `app/Http/Controllers/`, `app/controller/`, `src/Controller/` |

---

### 三、核心组件与目录结构总结

尽管目录命名不同，但所有框架都包含以下核心组成部分：

| 功能         | 描述                                      | 典型目录/文件                                                                               |
| :----------- | :---------------------------------------- | :------------------------------------------------------------------------------------------ |
| **入口文件** | 所有请求的起点，引导框架启动              | `public/index.php` (Hyperf 是 `bin/hyperf.php`)                                             |
| **配置中心** | 集中管理所有设置                          | `config/` 目录                                                                              |
| **路由定义** | 定义 URL 与控制器方法的映射               | `routes/`, `route/`, `config/routes.yaml`                                                   |
| **业务逻辑** | 存放核心业务代码，解耦控制器              | `app/Services/`, `app/service/` (通常为约定)                                                |
| **依赖容器** | 管理类依赖，实现控制反转 (IoC)            | Laravel: `Providers/`, ThinkPHP: 自动, Symfony: `services.yaml`, Hyperf: `dependencies.php` |
| **数据迁移** | 版本化管理数据库结构                      | `database/migrations/`                                                                      |
| **公共资源** | 存放可公开访问的静态文件（CSS, JS, 图片） | `public/` 目录                                                                              |
| **扩展包**   | 第三方库和组件                            | `vendor/` 目录 (由 Composer 管理)                                                           |

---

### 四、各框架最显著的结构特点

1.  **Laravel**
    *   **特点：** “全家桶”式设计，开箱即用功能极多。
    *   **标志目录：** `resources/`（原始前端资源），`app/Providers/`（服务提供者）。
    *   **感受：** 结构全面但稍重，为开发者考虑好了所有事情。

2.  **ThinkPHP**
    *   **特点：** 简单直观，符合中文思维习惯。
    *   **标志特性：** **多应用模式**（在 `app/` 下可创建 `admin/`, `index/` 等子应用）。
    *   **感受：** 学习成本最低，能快速上手并产出项目。

3.  **Symfony**
    *   **特点：** 高度解耦，由独立组件构成，极其规范。
    *   **标志目录：** `src/`（**所有**自定义PHP代码必须放在这里），`var/`（缓存和日志）。
    *   **感受：** 严谨、稳定、灵活，是构建大型复杂项目的首选。

4.  **Hyperf**
    *   **特点：** 为高性能和命令行（CLI）设计，常驻内存。
    *   **标志目录：** `bin/hyperf.php`（Swoole入口），`config/autoload/`（配置分区）。
    *   **标志特性：** 大量使用 **注解 (Annotation)** 和 **AOP（面向切面编程）**。
    *   **感受：** 现代、高效，专为API、微服务等高性能场景优化。

### 五、如何选择？

*   **追求开发速度和优雅体验？** -> **Laravel**
*   **需要快速上手或承接国内项目？** -> **ThinkPHP**
*   **构建大型、复杂、需要长期维护的企业系统？** -> **Symfony**
*   **需要极致性能，开发API、微服务或常驻内存应用？** -> **Hyperf**

### 终极总结

无论框架如何变化，其核心思想都是 **“约定大于配置”** 和 **“分离关注点”**。它们通过不同的目录结构来践行这些约定，最终目的都是为了让代码更**有序、可维护、可扩展**。

理解了一个框架的 MVC 结构和设计思想，再学习其他框架都会触类旁通。