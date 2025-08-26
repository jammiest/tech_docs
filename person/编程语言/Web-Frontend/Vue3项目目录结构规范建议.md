# Vue 3 项目目录结构规范建议

一个经过实践验证的Vue 3项目目录结构规范，这种结构适用于中大型项目，具有良好的可扩展性和维护性。

## 核心目录结构

```
project-root/
├── public/                   # 静态资源，不会被webpack处理
├── src/
│   ├── assets/               # 静态资源，会被webpack处理
│   │   ├── images/           # 图片资源
│   │   ├── fonts/            # 字体文件
│   │   └── styles/           # 全局样式
│   │       ├── variables.scss  # SCSS变量
│   │       ├── mixins.scss     # SCSS混合
│   │       └── global.scss     # 全局样式
│   ├── components/           # 公共组件
│   │   ├── common/           # 全局通用组件
│   │   ├── business/         # 业务通用组件
│   │   └── ui/               # UI组件库封装
│   ├── composables/          # Vue组合式函数
│   ├── directives/           # 自定义指令
│   ├── hooks/                # 自定义hooks
│   ├── layouts/              # 布局组件
│   ├── plugins/              # 插件
│   ├── router/               # 路由配置
│   │   ├── index.ts          # 路由入口
│   │   ├── routes/           # 路由模块
│   │   └── guards/           # 路由守卫
│   ├── stores/               # 状态管理(Pinia)
│   │   ├── modules/          # 模块化store
│   │   └── index.ts          # store入口
│   ├── utils/                # 工具函数
│   │   ├── request.ts        # 请求封装
│   │   ├── validate.ts       # 验证工具
│   │   └── ...               # 其他工具
│   ├── views/                # 页面组件
│   │   ├── moduleA/          # 模块A页面
│   │   ├── moduleB/          # 模块B页面
│   │   └── ...               # 其他模块
│   ├── App.vue               # 根组件
│   └── main.ts               # 应用入口
├── tests/                    # 测试代码
│   ├── unit/                 # 单元测试
│   └── e2e/                  # 端到端测试
├── types/                    # 类型定义
├── .env                      # 环境变量
├── .env.development          # 开发环境变量
├── .env.production           # 生产环境变量
└── vite.config.ts            # Vite配置
```

## 关键目录详细说明

### 1. 组件组织规范

组件分为三个层级：

1. **基础组件** (`components/common/`)：与业务无关的纯UI组件
2. **业务组件** (`components/business/`)：与业务相关但可复用的组件
3. **页面组件** (`views/`)：与路由对应的页面级组件

组件命名遵循 `PascalCase` 命名法，例如 `BaseButton.vue`

### 2. 状态管理规范

使用Pinia作为状态管理工具，采用模块化组织：

```typescript
// stores/modules/user.ts
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    name: '',
    age: 0
  }),
  getters: {
    isAdult: (state) => state.age >= 18
  },
  actions: {
    async fetchUser() {
      // API调用
    }
  }
})
```

### 3. 路由规范

采用动态路由和路由懒加载：

```typescript
// router/routes/auth.ts
const authRoutes: RouteRecordRaw[] = [
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/auth/Login.vue'),
    meta: { requiresAuth: false }
  }
]

export default authRoutes
```

### 4. API请求规范

建议采用分层架构：

```
src/
├── api/
│   ├── modules/          # 按模块划分API
│   │   ├── user.api.ts   # 用户相关API
│   │   └── product.api.ts # 产品相关API
│   └── index.ts          # API统一出口
```

请求封装示例：

```typescript
// utils/request.ts
import axios from 'axios'

const service = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000
})

// 请求拦截器
service.interceptors.request.use(
  (config) => {
    // 添加token等逻辑
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)

// 响应拦截器
service.interceptors.response.use(
  (response) => {
    // 统一处理响应
    return response.data
  },
  (error) => {
    // 统一错误处理
    return Promise.reject(error)
  }
)

export default service
```

### 5. 样式管理规范

推荐使用SCSS和CSS变量，通过Vite配置全局样式：

```javascript
// vite.config.ts
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/assets/styles/variables.scss";`
      }
    }
  }
})
```

## 架构设计原则

1. **模块化**：按功能划分模块，高内聚低耦合
2. **可扩展性**：目录结构支持水平扩展
3. **类型安全**：全面使用TypeScript
4. **性能优化**：路由懒加载、组件按需引入
5. **一致性**：统一的代码风格和命名规范

## 数学表达示例

在架构设计中，我们常用一些数学模型来衡量系统质量。例如，模块间的耦合度可以用以下公式表示：

$$
\text{耦合度} = \frac{\text{模块间交互点数}}{\text{总模块数} \times (\text{总模块数} - 1)}
$$

理想情况下，我们希望这个值趋近于0，表示模块间独立性高。

## 总结

这种目录结构规范结合了Vue 3的最佳实践和架构设计原则，适用于大多数中大型项目。根据项目实际情况，可以适当调整某些目录结构，但保持核心设计理念不变。