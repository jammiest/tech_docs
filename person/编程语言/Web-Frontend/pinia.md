# Pinia 状态管理指南

Pinia 是 Vue.js 的下一代状态管理库，提供更简洁的 API、更好的 TypeScript 支持和优秀的开发体验。本指南全面介绍 Pinia 的核心概念和最佳实践。

## 核心概念

### 1. Store 基础定义

```typescript
// stores/counter.ts
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  // 状态定义
  state: () => ({
    count: 0,
    name: 'Counter Store'
  }),

  // 计算属性
  getters: {
    doubleCount: (state) => state.count * 2,
    doubleCountPlusOne(): number {
      return this.doubleCount + 1
    },
    // 带参数的计算属性
    getCountBy: (state) => {
      return (multiplier: number) => state.count * multiplier
    }
  },

  // 操作方法
  actions: {
    increment() {
      this.count++
    },
    decrement() {
      this.count--
    },
    async incrementAsync() {
      await new Promise(resolve => setTimeout(resolve, 1000))
      this.increment()
    },
    reset() {
      this.$reset() // 内置重置方法
    }
  }
})
```

### 2. 组合式 Store 定义

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User } from '@/types/user'

export const useUserStore = defineStore('user', () => {
  // 状态
  const user = ref<User | null>(null)
  const token = ref<string | null>(null)
  const isLoggedIn = computed(() => !!user.value)

  // 计算属性
  const userName = computed(() => user.value?.name || 'Guest')
  const userRole = computed(() => user.value?.role || 'user')

  // 操作方法
  const login = async (credentials: LoginCredentials) => {
    try {
      const response = await api.auth.login(credentials)
      user.value = response.user
      token.value = response.token
      localStorage.setItem('token', response.token)
    } catch (error) {
      throw new Error('登录失败')
    }
  }

  const logout = () => {
    user.value = null
    token.value = null
    localStorage.removeItem('token')
  }

  const updateProfile = async (updates: Partial<User>) => {
    if (!user.value) throw new Error('用户未登录')
    user.value = { ...user.value, ...updates }
    await api.user.update(user.value.id, updates)
  }

  return {
    user,
    token,
    isLoggedIn,
    userName,
    userRole,
    login,
    logout,
    updateProfile
  }
})
```

## Store 使用模式

### 1. 组件中使用 Store

```vue
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useCounterStore } from '@/stores/counter'
import { useUserStore } from '@/stores/user'

// 正确使用方式
const counterStore = useCounterStore()
const userStore = useUserStore()

// 保持响应性：使用 storeToRefs
const { count, doubleCount } = storeToRefs(counterStore)
const { user, isLoggedIn } = storeToRefs(userStore)

// 直接解构会失去响应性（错误示例）
// const { count, name } = counterStore // ❌

// 操作方法直接使用
const increment = () => {
  counterStore.increment()
}

const loginUser = async () => {
  try {
    await userStore.login({ username: 'test', password: 'test' })
  } catch (error) {
    console.error('登录失败:', error)
  }
}

// 监听状态变化
counterStore.$subscribe((mutation, state) => {
  console.log('状态变化:', mutation.type, mutation.storeId, state)
})

// 监听 action 调用
counterStore.$onAction(({ name, store, args, after, onError }) => {
  console.log(`Action ${name} 被调用`, args)
  
  after((result) => {
    console.log(`Action ${name} 完成`, result)
  })
  
  onError((error) => {
    console.error(`Action ${name} 失败`, error)
  })
})
</script>

<template>
  <div>
    <h2>计数器: {{ count }}</h2>
    <p>双倍计数: {{ doubleCount }}</p>
    <button @click="increment">增加</button>
    
    <div v-if="isLoggedIn">
      <h3>欢迎, {{ user?.name }}</h3>
    </div>
    <div v-else>
      <button @click="loginUser">登录</button>
    </div>
  </div>
</template>
```

### 2. Store 间通信

```typescript
// stores/notification.ts
import { defineStore } from 'pinia'
import { useUserStore } from './user'

export const useNotificationStore = defineStore('notification', {
  state: () => ({
    messages: [] as string[],
    unreadCount: 0
  }),

  actions: {
    addMessage(message: string) {
      this.messages.push(message)
      this.unreadCount++
    },
    
    clearMessages() {
      this.messages = []
      this.unreadCount = 0
    },
    
    // 跨 store 调用
    async fetchUserNotifications() {
      const userStore = useUserStore()
      if (!userStore.isLoggedIn) return
      
      const notifications = await api.notifications.get(userStore.user!.id)
      this.messages = notifications
      this.unreadCount = notifications.length
    }
  }
})
```

## 高级功能

### 1. 状态持久化

```typescript
// stores/persist.ts
import { defineStore } from 'pinia'
import { ref, watch } from 'vue'

export const usePersistentStore = defineStore('persistent', () => {
  // 从 localStorage 初始化
  const savedState = localStorage.getItem('app-state')
  const state = ref(savedState ? JSON.parse(savedState) : {
    theme: 'light',
    language: 'zh-CN',
    preferences: {}
  })

  // 自动保存到 localStorage
  watch(state, (newState) => {
    localStorage.setItem('app-state', JSON.stringify(newState))
  }, { deep: true })

  return { state }
})

// 使用插件进行持久化
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

// Store 中使用持久化
export const usePersistentCounter = defineStore('persistent-counter', {
  state: () => ({ count: 0 }),
  persist: {
    enabled: true,
    strategies: [
      {
        key: 'counter',
        storage: localStorage,
        paths: ['count'] // 只持久化 count
      }
    ]
  }
})
```

### 2. 状态序列化与 hydration

```typescript
// stores/serialization.ts
import { defineStore } from 'pinia'

export const useComplexStore = defineStore('complex', {
  state: () => ({
    date: new Date(),
    map: new Map([['key', 'value']]),
    set: new Set([1, 2, 3]),
    regex: /pattern/g
  }),

  // 自定义序列化
  hydrate(state, initialState) {
    state.date = new Date(state.date)
    state.map = new Map(Object.entries(state.map))
    state.set = new Set(state.set)
    state.regex = new RegExp(state.regex.source, state.regex.flags)
  }
})
```

## 类型安全

### 1. 完整的 TypeScript 支持

```typescript
// types/user.ts
export interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user' | 'guest'
  createdAt: Date
}

export interface LoginCredentials {
  username: string
  password: string
  rememberMe?: boolean
}

// stores/user.ts
import { defineStore } from 'pinia'
import type { User, LoginCredentials } from '@/types/user'

interface UserState {
  user: User | null
  token: string | null
  loading: boolean
  error: string | null
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    user: null,
    token: null,
    loading: false,
    error: null
  }),

  getters: {
    isLoggedIn: (state): boolean => !!state.user,
    isAdmin: (state): boolean => state.user?.role === 'admin',
    userName: (state): string => state.user?.name || 'Guest'
  },

  actions: {
    async login(credentials: LoginCredentials): Promise<void> {
      this.loading = true
      this.error = null
      
      try {
        const response = await api.auth.login(credentials)
        this.user = response.user
        this.token = response.token
      } catch (error: any) {
        this.error = error.message
        throw error
      } finally {
        this.loading = false
      }
    }
  }
})
```

### 2. Store 组合与复用

```typescript
// stores/base-store.ts
import { defineStore } from 'pinia'

export const createBaseStore = <T extends string>(storeId: T) => {
  return defineStore(storeId, {
    state: () => ({
      loading: false,
      error: null as string | null
    }),

    actions: {
      setLoading(loading: boolean) {
        this.loading = loading
      },
      
      setError(error: string | null) {
        this.error = error
      },
      
      clearError() {
        this.error = null
      }
    }
  })
}

// 使用基础 Store
export const useProductsStore = defineStore('products', () => {
  const baseStore = createBaseStore('products-base')()
  
  const products = ref<Product[]>([])
  const selectedProduct = ref<Product | null>(null)
  
  const fetchProducts = async () => {
    baseStore.setLoading(true)
    try {
      products.value = await api.products.getAll()
    } catch (error: any) {
      baseStore.setError(error.message)
    } finally {
      baseStore.setLoading(false)
    }
  }
  
  return {
    ...baseStore,
    products,
    selectedProduct,
    fetchProducts
  }
})
```

## 性能优化

### 1. 状态更新优化

```typescript
// stores/optimized.ts
import { defineStore } from 'pinia'

export const useOptimizedStore = defineStore('optimized', {
  state: () => ({
    largeArray: [] as any[],
    complexObject: {} as Record<string, any>
  }),

  actions: {
    // 批量更新
    updateLargeArray(newItems: any[]) {
      // 避免直接赋值大数组
      this.largeArray = [...this.largeArray, ...newItems]
    },
    
    // 使用 patch 进行部分更新
    partialUpdate(updates: Partial<typeof this.$state>) {
      this.$patch(updates)
    },
    
    // 函数式更新
    functionalUpdate() {
      this.$patch((state) => {
        state.largeArray.push({ id: Date.now() })
        state.complexObject.lastUpdated = new Date()
      })
    },
    
    // 防抖操作
    debouncedUpdate: debounce(function (this: any, data: any) {
      this.complexObject = { ...this.complexObject, ...data }
    }, 300)
  }
})
```

### 2. 内存管理

```typescript
// stores/memory-management.ts
import { defineStore } from 'pinia'

export const useMemoryStore = defineStore('memory', {
  state: () => ({
    largeData: null as any,
    subscriptions: new Set<() => void>()
  }),

  actions: {
    // 清理大对象
    clearLargeData() {
      this.largeData = null
    },
    
    // 添加订阅
    subscribe(callback: () => void) {
      this.subscriptions.add(callback)
      return () => this.subscriptions.delete(callback)
    },
    
    // Store 清理
    cleanup() {
      this.clearLargeData()
      this.subscriptions.clear()
      this.$dispose() // 销毁 store 实例
    }
  }
})

// 组件中使用清理
const memoryStore = useMemoryStore()
onUnmounted(() => {
  memoryStore.cleanup()
})
```

## 测试策略

### 1. Store 单元测试

```typescript
// tests/unit/stores/counter.spec.ts
import { setActivePinia, createPinia } from 'pinia'
import { useCounterStore } from '@/stores/counter'

describe('Counter Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('应该初始化状态', () => {
    const store = useCounterStore()
    expect(store.count).toBe(0)
    expect(store.name).toBe('Counter Store')
  })

  it('应该增加计数', () => {
    const store = useCounterStore()
    store.increment()
    expect(store.count).toBe(1)
  })

  it('应该计算双倍计数', () => {
    const store = useCounterStore()
    store.increment()
    expect(store.doubleCount).toBe(2)
  })

  it('异步增加应该工作', async () => {
    const store = useCounterStore()
    await store.incrementAsync()
    expect(store.count).toBe(1)
  })
})

// tests/unit/stores/user.spec.ts
import { useUserStore } from '@/stores/user'
import { api } from '@/services/api'

vi.mock('@/services/api')

describe('User Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('应该处理登录成功', async () => {
    const mockUser = { id: 1, name: 'Test User' }
    api.auth.login.mockResolvedValue({ user: mockUser, token: 'test-token' })

    const store = useUserStore()
    await store.login({ username: 'test', password: 'test' })

    expect(store.user).toEqual(mockUser)
    expect(store.isLoggedIn).toBe(true)
  })

  it('应该处理登录失败', async () => {
    api.auth.login.mockRejectedValue(new Error('登录失败'))

    const store = useUserStore()
    await expect(store.login({ username: 'test', password: 'test' }))
      .rejects.toThrow('登录失败')

    expect(store.error).toBe('登录失败')
  })
})
```

### 2. 组件集成测试

```typescript
// tests/components/UserComponent.spec.ts
import { mount } from '@vue/test-utils'
import { createPinia, setActivePinia } from 'pinia'
import UserComponent from '@/components/UserComponent.vue'
import { useUserStore } from '@/stores/user'

describe('UserComponent', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('应该显示用户信息', async () => {
    const userStore = useUserStore()
    userStore.user = { id: 1, name: 'Test User', email: 'test@example.com' }

    const wrapper = mount(UserComponent)
    expect(wrapper.text()).toContain('Test User')
  })

  it('应该处理登录操作', async () => {
    const userStore = useUserStore()
    const loginSpy = vi.spyOn(userStore, 'login')

    const wrapper = mount(UserComponent)
    await wrapper.find('button').trigger('click')

    expect(loginSpy).toHaveBeenCalled()
  })
})
```

## 最佳实践

### 1. Store 组织架构

```
src/
├── stores/
│   ├── index.ts              # Store 导出入口
│   ├── modules/              # 业务模块
│   │   ├── user/            # 用户相关
│   │   │   ├── index.ts     # 主 store
│   │   │   ├── types.ts     # 类型定义
│   │   │   └── api.ts       # API 调用
│   │   ├── products/        # 商品相关
│   │   └── cart/            # 购物车
│   ├── system/              # 系统状态
│   │   ├── app.ts          # 应用状态
│   │   ├── theme.ts        # 主题设置
│   │   └── notification.ts # 通知系统
│   └── shared/             # 共享工具
│       ├── base-store.ts   # 基础 Store
│       ├── persist.ts      # 持久化工具
│       └── types.ts        # 共享类型
```

### 2. 代码组织规范

```typescript
// stores/modules/user/index.ts
import { defineStore } from 'pinia'
import type { User, LoginCredentials } from './types'
import * as userApi from './api'

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null as User | null,
    token: null as string | null,
    loading: false,
    error: null as string | null
  }),

  getters: {
    isLoggedIn: (state) => !!state.user,
    userName: (state) => state.user?.name || 'Guest'
  },

  actions: {
    async login(credentials: LoginCredentials) {
      this.loading = true
      this.error = null
      
      try {
        const response = await userApi.login(credentials)
        this.user = response.user
        this.token = response.token
      } catch (error: any) {
        this.error = error.message
        throw error
      } finally {
        this.loading = false
      }
    }
  }
})

// stores/modules/user/types.ts
export interface User {
  id: number
  name: string
  email: string
  role: string
}

export interface LoginCredentials {
  username: string
  password: string
  rememberMe?: boolean
}

// stores/modules/user/api.ts
import { api } from '@/services/api'
import type { User, LoginCredentials } from './types'

export const login = async (credentials: LoginCredentials) => {
  const response = await api.post('/auth/login', credentials)
  return response.data
}

export const logout = async () => {
  await api.post('/auth/logout')
}

export const getProfile = async (userId: number) => {
  const response = await api.get(`/users/${userId}`)
  return response.data
}
```

### 3. 错误处理模式

```typescript
// stores/error-handling.ts
import { defineStore } from 'pinia'

export const useErrorHandlingStore = defineStore('error-handling', {
  state: () => ({
    errors: new Map<string, Error>()
  }),

  actions: {
    setError(key: string, error: Error) {
      this.errors.set(key, error)
      // 自动清理错误
      setTimeout(() => {
        this.clearError(key)
      }, 5000)
    },
    
    clearError(key: string) {
      this.errors.delete(key)
    },
    
    clearAllErrors() {
      this.errors.clear()
    },
    
    getError(key: string): Error | undefined {
      return this.errors.get(key)
    },
    
    hasError(key: string): boolean {
      return this.errors.has(key)
    }
  }
})

// 在业务 Store 中使用错误处理
export const useProductsStore = defineStore('products', {
  actions: {
    async fetchProducts() {
      try {
        // ... 业务逻辑
      } catch (error: any) {
        const errorStore = useErrorHandlingStore()
        errorStore.setError('products-fetch', error)
        throw error
      }
    }
  }
})
```

## 插件开发

### 1. 自定义 Pinia 插件

```typescript
// plugins/pinia-logger.ts
import { PiniaPluginContext } from 'pinia'

interface LoggerOptions {
  logActions?: boolean
  logStateChanges?: boolean
}

export function piniaLogger({ logActions = true, logStateChanges = true }: LoggerOptions = {}) {
  return ({ store }: PiniaPluginContext) => {
    // 记录状态变化
    if (logStateChanges) {
      store.$subscribe((mutation, state) => {
        console.log(`[Pinia] ${mutation.storeId} 状态变化:`, mutation.type, state)
      })
    }

    // 记录 Action 调用
    if (logActions) {
      store.$onAction(({ name, store, args, after, onError }) => {
        console.log(`[Pinia] ${store.$id} Action ${name} 调用:`, args)
        
        after((result) => {
          console.log(`[Pinia] ${store.$id} Action ${name} 完成:`, result)
        })
        
        onError((error) => {
          console.error(`[Pinia] ${store.$id} Action ${name} 失败:`, error)
        })
      })
    }
  }
}

// 使用插件
import { createPinia } from 'pinia'
import { piniaLogger } from '@/plugins/pinia-logger'

const pinia = createPinia()
pinia.use(piniaLogger({ logActions: true, logStateChanges: false }))
```

### 2. 状态同步插件

```typescript
// plugins/pinia-sync.ts
export function piniaSync() {
  return ({ store }: PiniaPluginContext) => {
    // 监听状态变化并同步到其他标签页
    store.$subscribe((mutation, state) => {
      if (mutation.type === 'direct') {
        localStorage.setItem(`pinia-${store.$id}`, JSON.stringify(state))
        
        // 广播到其他标签页
        window.dispatchEvent(new CustomEvent('pinia-state-change', {
          detail: { storeId: store.$id, state }
        }))
      }
    })

    // 监听其他标签页的状态变化
    window.addEventListener('pinia-state-change', ((event: CustomEvent) => {
      const { storeId, state } = event.detail
      if (storeId === store.$id) {
        store.$patch(state)
      }
    }) as EventListener)

    // 初始化时从 localStorage 恢复状态
    const savedState = localStorage.getItem(`pinia-${store.$id}`)
    if (savedState) {
      store.$patch(JSON.parse(savedState))
    }
  }
}
```