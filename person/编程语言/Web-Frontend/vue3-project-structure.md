# Vue3 项目规范

本规范旨在为 Vue 3 项目提供统一的代码风格、项目结构和最佳实践指南，确保代码质量和团队协作效率。

## 项目结构规范

### 1. 标准项目结构

```
src/
├── assets/           # 静态资源
│   ├── images/      # 图片文件
│   ├── fonts/       # 字体文件
│   └── styles/      # 全局样式
├── components/       # 公共组件
│   ├── base/        # 基础组件
│   ├── ui/          # UI 组件
│   └── layout/      # 布局组件
├── composables/     # 组合式函数
│   ├── useAuth/     # 认证相关
│   ├── useApi/      # API 相关
│   └── index.ts     # 统一导出
├── stores/          # 状态管理
│   ├── modules/     # 模块化 store
│   └── index.ts     # store 入口
├── router/          # 路由配置
│   ├── guards/     # 路由守卫
│   ├── routes/      # 路由定义
│   └── index.ts     # 路由入口
├── views/           # 页面组件
│   ├── Home/        # 首页
│   ├── User/        # 用户相关页面
│   └── ...          # 其他页面
├── services/        # API 服务
│   ├── api/         # API 接口
│   ├── types/       # 类型定义
│   └── index.ts     # 服务入口
├── utils/           # 工具函数
│   ├── helpers/    # 辅助函数
│   ├── constants/   # 常量定义
│   └── index.ts     # 工具入口
├── App.vue          # 根组件
└── main.ts          # 应用入口
```

### 2. 组件命名规范

```typescript
// 文件命名：PascalCase
// components/
//   BaseButton.vue     // 基础组件
//   UserAvatar.vue     // 功能组件
//   AppHeader.vue      // 布局组件

// 组件命名：PascalCase
export default defineComponent({
  name: 'BaseButton' // 明确且有意义的名称
})

// 目录命名：kebab-case
// views/
//   user-profile/      // 用户相关页面
//   product-detail/    // 商品详情页
```

## 代码风格规范

### 1. 组合式 API 规范

```vue
<script setup lang="ts">
// 导入顺序：Vue → 第三方库 → 本地模块
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { fetchUserData } from '@/services/api'

// 响应式状态定义
const count = ref(0)
const user = ref<User | null>(null)
const loading = ref(false)

// 计算属性
const doubleCount = computed(() => count.value * 2)
const isAuthenticated = computed(() => !!user.value)

// 方法函数
const increment = () => {
  count.value++
}

const loadUserData = async () => {
  loading.value = true
  try {
    user.value = await fetchUserData()
  } catch (error) {
    console.error('加载用户数据失败:', error)
  } finally {
    loading.value = false
  }
}

// 生命周期
onMounted(() => {
  loadUserData()
})

// 模板引用
const formRef = ref<HTMLFormElement | null>(null)

// 暴露给模板
defineExpose({
  increment,
  loadUserData
})
</script>
```

### 2. Props 定义规范

```vue
<script setup lang="ts">
interface Props {
  // 必填属性
  title: string
  // 可选属性带默认值
  size?: 'small' | 'medium' | 'large'
  // 复杂对象
  user?: User
  // 函数类型
  onSubmit?: (data: FormData) => void
  // 数组类型
  items?: Array<string | number>
}

const props = withDefaults(defineProps<Props>(), {
  size: 'medium',
  items: () => []
})

// 使用类型安全的 props
const buttonClass = computed(() => `btn-${props.size}`)
</script>
```

### 3. Emits 定义规范

```vue
<script setup lang="ts">
const emit = defineEmits<{
  // 简单事件
  (e: 'click'): void
  // 带参数事件
  (e: 'update:modelValue', value: string): void
  (e: 'submit', data: FormData): void
  // 异步事件
  (e: 'loaded', payload: { data: any; status: number }): void
}>()

// 触发事件
const handleClick = () => {
  emit('click')
}

const handleSubmit = (formData: FormData) => {
  emit('submit', formData)
}
</script>
```

## 组件设计规范

### 1. 组件设计原则

```vue
<template>
  <!-- 单一职责：每个组件只负责一个明确的功能 -->
  <BaseButton 
    :type="type"
    :size="size"
    :disabled="disabled"
    @click="handleClick"
  >
    <slot></slot>
  </BaseButton>
</template>

<script setup lang="ts">
interface Props {
  type?: 'primary' | 'secondary' | 'danger'
  size?: 'small' | 'medium' | 'large'
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  type: 'primary',
  size: 'medium',
  disabled: false
})

const emit = defineEmits<{
  (e: 'click', event: MouseEvent): void
}>()

const handleClick = (event: MouseEvent) => {
  if (!props.disabled) {
    emit('click', event)
  }
}
</script>
```

### 2. 插槽使用规范

```vue
<template>
  <!-- 命名插槽 -->
  <div class="card">
    <header v-if="$slots.header" class="card-header">
      <slot name="header"></slot>
    </header>
    
    <div class="card-body">
      <!-- 默认插槽 -->
      <slot></slot>
    </div>
    
    <footer v-if="$slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

<!-- 作用域插槽 -->
<template>
  <div class="data-list">
    <slot 
      v-for="item in items" 
      :item="item" 
      :index="index"
      name="item"
    ></slot>
  </div>
</template>
```

## 状态管理规范

### 1. Pinia Store 规范

```typescript
// stores/user.ts
import { defineStore } from 'pinia'

interface UserState {
  user: User | null
  token: string | null
  isLoggedIn: boolean
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    user: null,
    token: null,
    isLoggedIn: false
  }),

  getters: {
    userName: (state) => state.user?.name || 'Guest',
    isAdmin: (state) => state.user?.role === 'admin'
  },

  actions: {
    async login(credentials: LoginCredentials) {
      try {
        const response = await api.auth.login(credentials)
        this.user = response.user
        this.token = response.token
        this.isLoggedIn = true
        
        // 持久化存储
        localStorage.setItem('token', response.token)
      } catch (error) {
        throw new Error('登录失败')
      }
    },

    logout() {
      this.$reset()
      localStorage.removeItem('token')
    }
  }
})
```

### 2. Store 使用规范

```vue
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'

// 正确使用方式
const userStore = useUserStore()
const { user, isLoggedIn } = storeToRefs(userStore)
const { login, logout } = userStore

// 错误：直接解构会失去响应性
// const { user, isLoggedIn } = useUserStore()
</script>
```

## 路由规范

### 1. 路由配置规范

```typescript
// router/routes.ts
import type { RouteRecordRaw } from 'vue-router'

export const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home/index.vue'),
    meta: {
      requiresAuth: true,
      title: '首页'
    }
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/Login/index.vue'),
    meta: {
      requiresGuest: true,
      title: '登录'
    }
  },
  {
    path: '/user/:id',
    name: 'UserProfile',
    component: () => import('@/views/User/Profile.vue'),
    props: true, // 开启 props 传参
    meta: {
      requiresAuth: true,
      title: '用户资料'
    }
  }
]
```

### 2. 路由守卫规范

```typescript
// router/guards/auth.ts
import type { NavigationGuardNext, RouteLocationNormalized } from 'vue-router'
import { useUserStore } from '@/stores/user'

export const authGuard = (
  to: RouteLocationNormalized,
  from: RouteLocationNormalized,
  next: NavigationGuardNext
) => {
  const userStore = useUserStore()
  
  if (to.meta.requiresAuth && !userStore.isLoggedIn) {
    next({ name: 'Login', query: { redirect: to.fullPath } })
  } else if (to.meta.requiresGuest && userStore.isLoggedIn) {
    next({ name: 'Home' })
  } else {
    next()
  }
}
```

## API 服务规范

### 1. API 服务层

```typescript
// services/api/client.ts
import axios from 'axios'

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// 请求拦截器
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 响应拦截器
apiClient.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      // 处理未授权错误
    }
    return Promise.reject(error)
  }
)

export default apiClient
```

### 2. 业务 API 模块

```typescript
// services/api/user.ts
import apiClient from './client'

export const userApi = {
  // 获取用户列表
  getUsers: (params?: PaginationParams) =>
    apiClient.get<User[]>('/users', { params }),

  // 获取用户详情
  getUser: (id: number) =>
    apiClient.get<User>(`/users/${id}`),

  // 创建用户
  createUser: (data: CreateUserData) =>
    apiClient.post<User>('/users', data),

  // 更新用户
  updateUser: (id: number, data: UpdateUserData) =>
    apiClient.put<User>(`/users/${id}`, data),

  // 删除用户
  deleteUser: (id: number) =>
    apiClient.delete(`/users/${id}`)
}
```

## 样式规范

### 1. CSS 命名规范

```vue
<template>
  <div class="user-card">
    <div class="user-card__header">
      <h3 class="user-card__title">{{ title }}</h3>
    </div>
    <div class="user-card__body">
      <p class="user-card__content">{{ content }}</p>
    </div>
  </div>
</template>

<style scoped>
.user-card {
  /* 组件根样式 */
}

.user-card__header {
  /* 组件头部 */
}

.user-card__title {
  /* 标题样式 */
}

.user-card__body {
  /* 内容区域 */
}

.user-card__content {
  /* 内容文本 */
}

/* 状态修饰符 */
.user-card--disabled {
  opacity: 0.5;
}

.user-card--loading {
  pointer-events: none;
}
</style>
```

### 2. CSS 变量使用

```vue
<style scoped>
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --border-radius: 4px;
  --spacing: 8px;
}

.component {
  color: var(--primary-color);
  border-radius: var(--border-radius);
  padding: calc(var(--spacing) * 2);
}

@media (prefers-color-scheme: dark) {
  :root {
    --primary-color: #0d6efd;
  }
}
</style>
```

## 测试规范

### 1. 单元测试规范

```typescript
// tests/component/BaseButton.spec.ts
import { mount } from '@vue/test-utils'
import BaseButton from '@/components/base/BaseButton.vue'

describe('BaseButton', () => {
  it('渲染正确的内容', () => {
    const wrapper = mount(BaseButton, {
      slots: {
        default: '点击我'
      }
    })
    
    expect(wrapper.text()).toContain('点击我')
  })

  it('触发点击事件', async () => {
    const wrapper = mount(BaseButton)
    
    await wrapper.trigger('click')
    
    expect(wrapper.emitted()).toHaveProperty('click')
  })

  it('禁用状态下不触发事件', async () => {
    const wrapper = mount(BaseButton, {
      props: {
        disabled: true
      }
    })
    
    await wrapper.trigger('click')
    
    expect(wrapper.emitted()).not.toHaveProperty('click')
  })
})
```

### 2. 组合式函数测试

```typescript
// tests/composables/useCounter.spec.ts
import { renderHook } from '@testing-library/vue'
import { useCounter } from '@/composables/useCounter'

describe('useCounter', () => {
  it('初始值为0', () => {
    const { result } = renderHook(() => useCounter())
    
    expect(result.value.count.value).toBe(0)
  })

  it('增加计数', async () => {
    const { result } = renderHook(() => useCounter())
    
    result.value.increment()
    
    expect(result.value.count.value).toBe(1)
  })

  it('减少计数', async () => {
    const { result } = renderHook(() => useCounter())
    
    result.value.decrement()
    
    expect(result.value.count.value).toBe(-1)
  })
})
```

## 提交规范

### 1. Git Commit 规范

```bash
# 提交类型
feat:     新增功能
fix:      修复bug
docs:     文档更新
style:    代码格式调整
refactor: 代码重构
test:     测试相关
chore:    构建过程或辅助工具变动

# 示例
git commit -m "feat: 添加用户登录功能"
git commit -m "fix: 修复按钮点击无效的问题"
git commit -m "docs: 更新项目README文档"
```

### 2. 版本发布规范

```json
{
  "version": "1.0.0",
  "scripts": {
    "release": "standard-version",
    "release:major": "standard-version --release-as major",
    "release:minor": "standard-version --release-as minor",
    "release:patch": "standard-version --release-as patch"
  }
}
```

## 部署规范

### 1. 环境配置

```env
# .env.development
VITE_API_BASE_URL=http://localhost:3000/api
VITE_APP_TITLE=开发环境

# .env.production
VITE_API_BASE_URL=https://api.example.com
VITE_APP_TITLE=生产环境
```

### 2. 构建配置

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig(({ mode }) => ({
  plugins: [vue()],
  
  build: {
    outDir: 'dist',
    sourcemap: mode !== 'production',
    minify: mode === 'production' ? 'terser' : false,
    
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['vue', 'vue-router', 'pinia'],
          ui: ['element-plus', 'vant']
        }
      }
    }
  },
  
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      }
    }
  }
}))
```

## 代码审查规范

### 1. 审查清单

- [ ] 代码符合项目规范
- [ ] 组件命名和文件结构正确
- [ ] 类型定义完整且正确
- [ ] 错误处理机制完善
- [ ] 性能优化考虑周全
- [ ] 测试覆盖充分
- [ ] 文档更新及时

### 2. 常见问题检查

```typescript
// 错误示例：直接解构 store
const { user, isLoggedIn } = useUserStore() // ❌

// 正确示例：使用 storeToRefs
const userStore = useUserStore()
const { user, isLoggedIn } = storeToRefs(userStore) // ✅

// 错误示例：未处理异步错误
const loadData = async () => {
  const data = await api.getData() // ❌ 未处理错误
}

// 正确示例：完整的错误处理
const loadData = async () => {
  try {
    const data = await api.getData()
    return data
  } catch (error) {
    console.error('加载数据失败:', error)
    throw error
  }
}
```
