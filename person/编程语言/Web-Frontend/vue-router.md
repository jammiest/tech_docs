# Vue Router 指南

Vue Router 是 Vue.js 的官方路由管理器，用于构建单页面应用程序（SPA）。本指南涵盖 Vue Router 4 的核心概念、高级功能和最佳实践。

## 核心概念

### 1. 路由基础配置

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/views/Home.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home,
    meta: {
      requiresAuth: true,
      title: '首页'
    }
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue'), // 懒加载
    meta: {
      title: '关于我们'
    }
  },
  {
    path: '/user/:id',
    name: 'UserProfile',
    component: () => import('@/views/UserProfile.vue'),
    props: true, // 将路由参数作为 props 传递
    meta: {
      requiresAuth: true
    }
  },
  {
    path: '/:pathMatch(.*)*', // 404 页面
    name: 'NotFound',
    component: () => import('@/views/NotFound.vue')
  }
]

const router = createRouter({
  history: createWebHistory(), // 使用 HTML5 History 模式
  // history: createWebHashHistory(), // 使用 Hash 模式
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else if (to.hash) {
      return {
        el: to.hash,
        behavior: 'smooth'
      }
    } else {
      return { top: 0 }
    }
  }
})

export default router
```

### 2. 路由安装与使用

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(router)
app.mount('#app')
```

## 路由导航

### 1. 声明式导航

```vue
<template>
  <!-- 基本导航 -->
  <router-link to="/">首页</router-link>
  
  <!-- 命名路由 -->
  <router-link :to="{ name: 'About' }">关于</router-link>
  
  <!-- 带参数路由 -->
  <router-link :to="{ name: 'UserProfile', params: { id: 1 } }">
    用户资料
  </router-link>
  
  <!-- 带查询参数 -->
  <router-link :to="{ path: '/search', query: { q: 'vue' } }">
    搜索
  </router-link>
  
  <!-- 自定义样式 -->
  <router-link 
    to="/contact" 
    class="nav-link"
    active-class="active-link"
    exact-active-class="exact-active-link"
  >
    联系我们
  </router-link>
  
  <!-- 替换当前历史记录 -->
  <router-link :to="{ name: 'Home' }" replace>
    首页（替换）
  </router-link>
</template>

<style>
.nav-link {
  color: #333;
  text-decoration: none;
}

.active-link {
  color: #007bff;
  font-weight: bold;
}

.exact-active-link {
  border-bottom: 2px solid #007bff;
}
</style>
```

### 2. 编程式导航

```vue
<script setup>
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// 基本导航
const goToHome = () => {
  router.push('/')
}

// 命名路由导航
const goToAbout = () => {
  router.push({ name: 'About' })
}

// 带参数导航
const goToUserProfile = (userId) => {
  router.push({ 
    name: 'UserProfile', 
    params: { id: userId } 
  })
}

// 带查询参数导航
const search = (query) => {
  router.push({
    path: '/search',
    query: { q: query }
  })
}

// 替换当前历史记录
const replaceToHome = () => {
  router.replace({ name: 'Home' })
}

// 前进后退
const goBack = () => {
  router.go(-1) // 后退一步
}

const goForward = () => {
  router.go(1) // 前进一步
}

// 获取当前路由信息
console.log('当前路径:', route.path)
console.log('路由参数:', route.params)
console.log('查询参数:', route.query)
console.log('路由元信息:', route.meta)
</script>
```

## 路由守卫

### 1. 全局守卫

```javascript
// router/guards.js
import { useUserStore } from '@/stores/user'

// 全局前置守卫
router.beforeEach(async (to, from, next) => {
  const userStore = useUserStore()
  
  // 设置页面标题
  if (to.meta.title) {
    document.title = `${to.meta.title} - 我的应用`
  }
  
  // 认证检查
  if (to.meta.requiresAuth && !userStore.isAuthenticated) {
    next({
      name: 'Login',
      query: { redirect: to.fullPath }
    })
    return
  }
  
  // 游客页面检查
  if (to.meta.requiresGuest && userStore.isAuthenticated) {
    next({ name: 'Home' })
    return
  }
  
  // 权限检查
  if (to.meta.requiredPermissions) {
    const hasPermission = userStore.hasPermissions(to.meta.requiredPermissions)
    if (!hasPermission) {
      next({ name: 'Forbidden' })
      return
    }
  }
  
  next()
})

// 全局解析守卫
router.beforeResolve(async (to, from, next) => {
  // 确保异步组件加载完成
  if (to.meta.requiresAsyncData) {
    try {
      await loadAsyncData()
      next()
    } catch (error) {
      next({ name: 'Error', params: { error: '数据加载失败' } })
    }
  } else {
    next()
  }
})

// 全局后置钩子
router.afterEach((to, from) => {
  // 页面访问统计
  trackPageView(to.fullPath)
  
  // 滚动到顶部
  window.scrollTo(0, 0)
})
```

### 2. 路由独享守卫

```javascript
const routes = [
  {
    path: '/admin',
    name: 'Admin',
    component: () => import('@/views/Admin.vue'),
    beforeEnter: (to, from, next) => {
      const userStore = useUserStore()
      
      if (!userStore.isAdmin) {
        next({ name: 'Forbidden' })
      } else {
        next()
      }
    }
  }
]
```

### 3. 组件内守卫

```vue
<script setup>
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

// 路由更新守卫
onBeforeRouteUpdate(async (to, from, next) => {
  // 仅当 id 发生变化时重新加载数据
  if (to.params.id !== from.params.id) {
    await loadUserData(to.params.id)
  }
  next()
})

// 路由离开守卫
onBeforeRouteLeave((to, from, next) => {
  if (hasUnsavedChanges.value) {
    const confirmLeave = window.confirm('确定要离开吗？未保存的更改将会丢失。')
    if (confirmLeave) {
      next()
    } else {
      next(false)
    }
  } else {
    next()
  }
})
</script>
```

## 高级路由功能

### 1. 动态路由

```javascript
// 动态添加路由
const addDynamicRoutes = async () => {
  const modules = await fetchUserModules()
  
  modules.forEach(module => {
    router.addRoute({
      path: module.path,
      name: module.name,
      component: () => import(`@/views/${module.component}.vue`)
    })
  })
}

// 删除路由
const removeRoute = (routeName) => {
  if (router.hasRoute(routeName)) {
    router.removeRoute(routeName)
  }
}

// 检查路由是否存在
const checkRouteExists = (routeName) => {
  return router.hasRoute(routeName)
}
```

### 2. 路由元信息

```javascript
const routes = [
  {
    path: '/dashboard',
    meta: {
      requiresAuth: true,
      requiredPermissions: ['dashboard:read'],
      breadcrumb: '控制面板',
      transition: 'fade',
      keepAlive: true
    }
  }
]

// 使用元信息
router.beforeEach((to, from, next) => {
  const matched = to.matched
  const meta = matched.reduce((acc, record) => {
    return { ...acc, ...record.meta }
  }, {})
  
  // 使用合并后的 meta
  if (meta.requiresAuth) {
    // 认证检查
  }
  
  next()
})
```

### 3. 过渡动画

```vue
<template>
  <router-view v-slot="{ Component, route }">
    <transition 
      :name="route.meta.transition || 'fade'"
      mode="out-in"
    >
      <component :is="Component" :key="route.fullPath" />
    </transition>
  </router-view>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter-from {
  transform: translateX(100%);
}

.slide-leave-to {
  transform: translateX(-100%);
}
</style>
```

### 4. 滚动行为

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    // 返回保存的位置
    if (savedPosition) {
      return savedPosition
    }
    
    // 滚动到锚点
    if (to.hash) {
      return {
        el: to.hash,
        behavior: 'smooth',
        top: 80 // 偏移量
      }
    }
    
    // 特定路由的滚动位置
    if (to.meta.scrollToTop !== false) {
      return { top: 0, left: 0 }
    }
    
    // 保持原位置
    return false
  }
})
```

## 嵌套路由

### 1. 嵌套路由配置

```javascript
const routes = [
  {
    path: '/user/:id',
    component: () => import('@/layouts/UserLayout.vue'),
    children: [
      {
        path: '', // 默认子路由
        name: 'UserProfile',
        component: () => import('@/views/user/Profile.vue')
      },
      {
        path: 'posts',
        name: 'UserPosts',
        component: () => import('@/views/user/Posts.vue')
      },
      {
        path: 'settings',
        name: 'UserSettings',
        component: () => import('@/views/user/Settings.vue')
      }
    ]
  }
]
```

### 2. 嵌套路由视图

```vue
<!-- UserLayout.vue -->
<template>
  <div class="user-layout">
    <header class="user-header">
      <h2>用户页面</h2>
      <nav class="user-nav">
        <router-link :to="{ name: 'UserProfile' }">资料</router-link>
        <router-link :to="{ name: 'UserPosts' }">文章</router-link>
        <router-link :to="{ name: 'UserSettings' }">设置</router-link>
      </nav>
    </header>
    
    <main class="user-main">
      <router-view></router-view> <!-- 嵌套路由出口 -->
    </main>
  </div>
</template>
```

## 路由数据获取

### 1. 导航前数据获取

```vue
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()
const user = ref(null)
const loading = ref(false)

// 根据路由参数获取数据
const fetchUserData = async () => {
  loading.value = true
  try {
    user.value = await userApi.getUser(route.params.id)
  } catch (error) {
    console.error('获取用户数据失败:', error)
  } finally {
    loading.value = false
  }
}

// 监听路由参数变化
watch(() => route.params.id, fetchUserData, { immediate: true })
</script>
```

### 2. 导航后数据获取

```javascript
// 路由配置中使用 props 传递数据
const routes = [
  {
    path: '/user/:id',
    name: 'UserProfile',
    component: () => import('@/views/UserProfile.vue'),
    props: (route) => ({
      id: route.params.id,
      preview: route.query.preview === 'true'
    })
  }
]
```

## 错误处理

### 1. 路由错误处理

```javascript
// 全局错误处理
router.onError((error) => {
  console.error('路由错误:', error)
  
  if (error.message.includes('Failed to fetch dynamically imported module')) {
    // 处理组件加载失败
    router.push({ name: 'ComponentLoadError' })
  }
})

// 导航失败处理
const navigate = async (to) => {
  try {
    await router.push(to)
  } catch (error) {
    if (error.name === 'NavigationDuplicated') {
      // 处理重复导航
      console.warn('重复导航尝试')
    } else {
      // 其他导航错误
      console.error('导航失败:', error)
    }
  }
}
```

### 2. 加载状态处理

```vue
<template>
  <router-view v-slot="{ Component }">
    <suspense>
      <template #default>
        <component :is="Component" />
      </template>
      <template #fallback>
        <div class="loading-spinner">
          页面加载中...
        </div>
      </template>
    </suspense>
  </router-view>
</template>
```

## 性能优化

### 1. 路由懒加载

```javascript
// 静态导入（不推荐用于大型组件）
import Home from '@/views/Home.vue'

// 动态导入（推荐）
const UserProfile = () => import('@/views/UserProfile.vue')

// 带加载状态的懒加载
const UserProfile = defineAsyncComponent({
  loader: () => import('@/views/UserProfile.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorComponent,
  delay: 200, // 延迟显示 loading
  timeout: 3000 // 超时时间
})

// 分组懒加载（webpack chunk）
const AdminDashboard = () => import(/* webpackChunkName: "admin" */ '@/views/admin/Dashboard.vue')
const AdminUsers = () => import(/* webpackChunkName: "admin" */ '@/views/admin/Users.vue')
```

### 2. 路由预加载

```javascript
// 手动预加载
const preloadRoutes = () => {
  // 预加载用户相关路由
  import('@/views/UserProfile.vue')
  import('@/views/UserSettings.vue')
}

// 路由配置中预加载
const routes = [
  {
    path: '/user/:id',
    component: () => import('@/views/UserProfile.vue'),
    meta: {
      preload: true // 自定义预加载标记
    }
  }
]

// 全局后置钩子中预加载
router.afterEach((to, from) => {
  const preloadRoutes = to.matched
    .flatMap(record => record.meta.preloadRoutes || [])
  
  preloadRoutes.forEach(routeName => {
    const route = router.resolve({ name: routeName })
    if (route.meta.component) {
      route.meta.component()
    }
  })
})
```

## TypeScript 支持

### 1. 类型安全的路由

```typescript
// router/types.ts
import type { RouteRecordRaw } from 'vue-router'

export interface RouteMeta {
  requiresAuth?: boolean
  requiredPermissions?: string[]
  title?: string
  transition?: string
  keepAlive?: boolean
}

export type AppRouteRecordRaw = RouteRecordRaw & {
  meta?: RouteMeta
  children?: AppRouteRecordRaw[]
}

// 扩展 RouteMeta 类型
declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    requiredPermissions?: string[]
    title?: string
    transition?: string
    keepAlive?: boolean
  }
}
```

### 2. 类型安全的导航

```typescript
import { useRouter } from 'vue-router'

const router = useRouter()

// 类型安全的导航
const goToUserProfile = (id: number) => {
  router.push({
    name: 'UserProfile',
    params: { id } // 类型检查
  })
}

// 类型安全的参数获取
const route = useRoute()
const userId = computed(() => {
  const id = route.params.id
  return typeof id === 'string' ? parseInt(id) : 0
})
```

## 测试指南

### 1. 路由单元测试

```typescript
// tests/unit/router.spec.ts
import { describe, it, expect } from 'vitest'
import { routes } from '@/router'

describe('路由配置', () => {
  it('应该包含正确的路由', () => {
    const routeNames = routes.map(route => route.name)
    expect(routeNames).toContain('Home')
    expect(routeNames).toContain('About')
  })

  it('路由元信息应该正确', () => {
    const homeRoute = routes.find(route => route.name === 'Home')
    expect(homeRoute?.meta?.requiresAuth).toBe(true)
  })
})

// 组件路由测试
import { mount } from '@vue/test-utils'
import { createRouter, createWebHistory } from 'vue-router'
import Component from '@/components/MyComponent.vue'

const router = createRouter({
  history: createWebHistory(),
  routes: [{ path: '/', component: Component }]
})

describe('组件路由行为', () => {
  it('应该正确导航', async () => {
    const wrapper = mount(Component, {
      global: {
        plugins: [router]
      }
    })
    
    await router.push('/')
    expect(wrapper.text()).toContain('首页内容')
  })
})
```

## 最佳实践

### 1. 路由组织规范

```
src/
├── router/
│   ├── index.ts          # 路由入口
│   ├── types.ts          # 类型定义
│   ├── guards/           # 路由守卫
│   │   ├── auth.ts       # 认证守卫
│   │   ├── permission.ts # 权限守卫
│   │   └── index.ts      # 守卫聚合
│   ├── routes/           # 路由定义
│   │   ├── public.ts     # 公开路由
│   │   ├── private.ts    # 私有路由
│   │   └── index.ts      # 路由聚合
│   └── utils/            # 工具函数
│       └── scroll.ts     # 滚动行为
```

### 2. 安全实践

```javascript
// 路由权限控制
router.beforeEach((to, from, next) => {
  // 敏感路由记录
  if (to.meta.sensitive) {
    console.warn('访问敏感路由:', to.fullPath)
  }
  
  // CSRF 保护
  if (to.meta.requiresCsrf) {
    const hasValidToken = validateCsrfToken()
    if (!hasValidToken) {
      next({ name: 'CsrfError' })
      return
    }
  }
  
  next()
})

// 路由变化监控
let lastRouteChangeTime = 0
const ROUTE_CHANGE_THROTTLE = 1000

router.beforeEach((to, from, next) => {
  const now = Date.now()
  if (now - lastRouteChangeTime < ROUTE_CHANGE_THROTTLE) {
    console.warn('路由变化过于频繁')
    // 可以添加节流逻辑
  }
  lastRouteChangeTime = now
  next()
})
```
