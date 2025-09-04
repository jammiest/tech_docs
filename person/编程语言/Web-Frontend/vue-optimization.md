# Vue 性能优化指南

Vue.js 应用性能优化涉及多个层面，从代码编写到构建配置，再到运行时优化。本指南提供全面的 Vue 3 性能优化策略和实践。

## 编译时优化

### 1. 模板编译优化

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [
    vue({
      // 模板编译选项
      template: {
        // 编译时优化选项
        compilerOptions: {
          // 移除空白字符
          whitespace: 'condense',
          // 自定义指令优化
          directiveTransforms: {
            // 自定义指令编译时优化
          }
        }
      },
      // 响应式语法糖（实验性）
      reactivityTransform: true
    })
  ]
})
```

### 2. 构建配置优化

```javascript
// vite.config.js
export default defineConfig({
  build: {
    // 代码分割配置
    rollupOptions: {
      output: {
        manualChunks: {
          // 第三方库分组
          vendor: ['vue', 'vue-router', 'pinia'],
          // UI 库分组
          ui: ['element-plus', 'vant'],
          // 工具库分组
          utils: ['lodash', 'dayjs'],
          // 按路由分组
          'home-page': ['src/views/Home.vue'],
          'user-page': ['src/views/User/**/*.vue']
        },
        // 文件命名优化
        chunkFileNames: 'js/[name]-[hash].js',
        entryFileNames: 'js/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]'
      }
    },
    // 构建优化选项
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,    // 生产环境移除 console
        drop_debugger: true    // 移除 debugger
      }
    },
    // 源映射配置
    sourcemap: process.env.NODE_ENV !== 'production',
    // 分块大小警告限制
    chunkSizeWarningLimit: 1000
  }
})
```

## 组件级优化

### 1. 组件懒加载

```vue
<script setup>
import { defineAsyncComponent } from 'vue'

// 基础懒加载组件
const LazyComponent = defineAsyncComponent(() =>
  import('./LazyComponent.vue')
)

// 带加载状态的懒加载组件
const LazyWithLoading = defineAsyncComponent({
  loader: () => import('./HeavyComponent.vue'),
  loadingComponent: LoadingSpinner,
  delay: 200, // 延迟显示 loading
  timeout: 3000 // 超时时间
})

// 错误处理组件
const LazyWithError = defineAsyncComponent({
  loader: () => import('./CriticalComponent.vue'),
  errorComponent: ErrorComponent,
  onError(error, retry, fail) {
    // 错误处理逻辑
    if (error.message.includes('加载失败')) {
      retry()
    } else {
      fail()
    }
  }
})
</script>

<template>
  <Suspense>
    <template #default>
      <LazyComponent />
    </template>
    <template #fallback>
      <div class="loading">加载中...</div>
    </template>
  </Suspense>
</template>
```

### 2. 组件性能优化

```vue
<script setup>
import { shallowRef, computed, watch } from 'vue'

// 使用 shallowRef 避免不必要的深度响应式
const largeObject = shallowRef({ 
  /* 大数据对象 */
})

// 计算属性缓存优化
const expensiveValue = computed(() => {
  // 昂贵的计算操作
  return heavyComputation(largeObject.value)
})

// 监听器优化
watch(
  () => largeObject.value.id, // 只监听特定属性
  (newId) => {
    console.log('ID 变化:', newId)
  },
  { immediate: true }
)

// 防抖监听
import { debounce } from 'lodash-es'
const debouncedHandler = debounce((value) => {
  // 处理逻辑
}, 300)

watch(someValue, debouncedHandler)
</script>
```

### 3. 列表渲染优化

```vue
<script setup>
import { computed } from 'vue'

const props = defineProps({
  items: {
    type: Array,
    required: true
  },
  filter: String
})

// 计算属性优化列表数据
const filteredItems = computed(() => {
  return props.items.filter(item => 
    item.name.includes(props.filter)
  )
})

// 虚拟滚动集成
import { useVirtualList } from '@vueuse/core'

const { list, containerProps, wrapperProps } = useVirtualList(
  filteredItems,
  {
    itemHeight: 50,
    overscan: 5 // 预渲染项目数
  }
)
</script>

<template>
  <!-- 虚拟滚动容器 -->
  <div v-bind="containerProps" class="virtual-container">
    <div v-bind="wrapperProps">
      <div 
        v-for="item in list" 
        :key="item.id"
        class="virtual-item"
      >
        {{ item.name }}
      </div>
    </div>
  </div>

  <!-- 常规列表优化 -->
  <div class="optimized-list">
    <div 
      v-for="item in filteredItems" 
      :key="item.id"
      class="list-item"
    >
      <!-- 使用简单的子组件 -->
      <ListItem :item="item" />
    </div>
  </div>
</template>

<style>
.virtual-container {
  height: 400px;
  overflow-y: auto;
}

.virtual-item {
  height: 50px;
  line-height: 50px;
}

.optimized-list {
  max-height: 400px;
  overflow-y: auto;
}
</style>
```

## 状态管理优化

### 1. Pinia 状态优化

```typescript
// stores/optimized.ts
import { defineStore } from 'pinia'

export const useOptimizedStore = defineStore('optimized', {
  state: () => ({
    // 使用浅层响应式数据
    largeData: shallowRef({}),
    // 分页数据
    paginatedData: [] as any[],
    // 避免不必要的响应式
    staticConfig: markRaw({
      version: '1.0.0',
      settings: Object.freeze({})
    })
  }),

  actions: {
    // 批量更新优化
    updateDataInBatches(newData: any[]) {
      // 使用 patch 批量更新
      this.$patch((state) => {
        state.paginatedData = [...state.paginatedData, ...newData]
      })
    },

    // 防抖操作
    debouncedUpdate: debounce(function (this: any, data: any) {
      this.$patch({ largeData: { ...this.largeData, ...data } })
    }, 300),

    // 内存敏感操作
    clearMemory() {
      this.largeData = {}
      this.paginatedData = []
    }
  }
})
```

### 2. 状态订阅优化

```typescript
// 组件中使用
import { storeToRefs } from 'pinia'

const store = useOptimizedStore()

// 只解构需要的状态
const { paginatedData } = storeToRefs(store)

// 精细化的状态订阅
const unsubscribe = store.$subscribe(
  (mutation, state) => {
    // 只处理特定状态变化
    if (mutation.type === 'direct' && mutation.events?.key === 'paginatedData') {
      console.log('分页数据更新')
    }
  },
  { detached: true } // 组件卸载后保持订阅
)

// 组件卸载时清理
onUnmounted(() => {
  unsubscribe()
})
```

## 运行时优化

### 1. 事件处理优化

```vue
<script setup>
import { throttle, debounce } from 'lodash-es'

// 节流事件处理
const throttledScroll = throttle((event) => {
  // 处理滚动事件
}, 100)

// 防抖事件处理
const debouncedInput = debounce((value) => {
  // 处理输入事件
}, 300)

// 手动事件监听优化
onMounted(() => {
  const handler = (event) => {
    // 事件处理逻辑
  }
  
  window.addEventListener('scroll', handler, { passive: true })
  
  onUnmounted(() => {
    window.removeEventListener('scroll', handler)
  })
})
</script>

<template>
  <!-- 原生事件优化 -->
  <input @input="debouncedInput($event.target.value)">
  
  <!-- 被动事件监听 -->
  <div @scroll.passive="throttledScroll">
    <!-- 长列表内容 -->
  </div>
</template>
```

### 2. DOM 操作优化

```vue
<script setup>
import { nextTick } from 'vue'

// 批量 DOM 更新
const updateMultipleElements = async () => {
  // 在下一个 tick 批量更新
  await nextTick()
  
  // 使用 documentFragment 进行批量操作
  const fragment = document.createDocumentFragment()
  // ... 批量操作
}

// 动画优化
const animateElement = (element: HTMLElement) => {
  // 使用 transform 和 opacity 进行动画
  element.style.transform = 'translateX(100px)'
  element.style.opacity = '0.5'
  
  // 使用 will-change 提示浏览器
  element.style.willChange = 'transform, opacity'
}

// 使用 Web Animations API
const useWebAnimations = (element: HTMLElement) => {
  element.animate([
    { transform: 'translateX(0)', opacity: 1 },
    { transform: 'translateX(100px)', opacity: 0.5 }
  ], {
    duration: 300,
    easing: 'ease-in-out',
    fill: 'forwards'
  })
}
</script>
```

## 内存管理

### 1. 内存泄漏预防

```vue
<script setup>
import { onUnmounted, ref } from 'vue'

// 定时器清理
const timer = ref<NodeJS.Timeout | null>(null)

onMounted(() => {
  timer.value = setInterval(() => {
    // 定时任务
  }, 1000)
})

onUnmounted(() => {
  if (timer.value) {
    clearInterval(timer.value)
    timer.value = null
  }
})

// 事件监听器清理
const eventListeners = ref<(() => void)[]>([])

const setupEventListeners = () => {
  const handler = () => { /* 事件处理 */ }
  window.addEventListener('resize', handler)
  eventListeners.value.push(() => {
    window.removeEventListener('resize', handler)
  })
}

onUnmounted(() => {
  eventListeners.value.forEach(cleanup => cleanup())
  eventListeners.value = []
})

// 大型对象清理
const largeData = ref<any>(null)

const loadLargeData = async () => {
  largeData.value = await fetchLargeData()
}

const clearLargeData = () => {
  largeData.value = null
}

// 组件卸载时清理
onUnmounted(() => {
  clearLargeData()
})
</script>
```

### 2. 对象池模式

```typescript
// utils/object-pool.ts
export class ObjectPool<T> {
  private pool: T[] = []
  private create: () => T
  private reset: (obj: T) => void

  constructor(create: () => T, reset: (obj: T) => void) {
    this.create = create
    this.reset = reset
  }

  acquire(): T {
    return this.pool.length > 0 ? this.pool.pop()! : this.create()
  }

  release(obj: T): void {
    this.reset(obj)
    this.pool.push(obj)
  }

  clear(): void {
    this.pool = []
  }
}

// 使用对象池
const pool = new ObjectPool(
  () => ({ data: new Array(1000).fill(0) }),
  (obj) => { obj.data.fill(0) }
)

const heavyObject = pool.acquire()
// 使用完成后释放
pool.release(heavyObject)
```

## 网络优化

### 1. 资源加载优化

```javascript
// 资源预加载
const preloadResources = () => {
  // 图片预加载
  const images = [
    '/images/hero-large.jpg',
    '/images/background.png'
  ]
  
  images.forEach(src => {
    const link = document.createElement('link')
    link.rel = 'preload'
    link.as = 'image'
    link.href = src
    document.head.appendChild(link)
  })
  
  // 组件预加载
  import('./HeavyComponent.vue')
}

// 关键资源优先加载
const loadCriticalResources = async () => {
  // 优先加载关键 CSS
  const criticalCSS = await fetch('/styles/critical.css')
  const style = document.createElement('style')
  style.textContent = await criticalCSS.text()
  document.head.appendChild(style)
  
  // 延迟加载非关键资源
  setTimeout(() => {
    import('./NonCriticalComponent.vue')
  }, 3000)
}
```

### 2. API 调用优化

```typescript
// services/optimized-api.ts
import { throttle, debounce } from 'lodash-es'

class OptimizedAPI {
  private cache = new Map<string, any>()
  private pendingRequests = new Map<string, Promise<any>>()

  // 请求缓存
  async cachedRequest<T>(key: string, request: () => Promise<T>): Promise<T> {
    if (this.cache.has(key)) {
      return this.cache.get(key)
    }

    if (this.pendingRequests.has(key)) {
      return this.pendingRequests.get(key)
    }

    const promise = request().then(response => {
      this.cache.set(key, response)
      this.pendingRequests.delete(key)
      return response
    })

    this.pendingRequests.set(key, promise)
    return promise
  }

  // 批量请求
  async batchRequests<T>(requests: (() => Promise<T>)[]): Promise<T[]> {
    return Promise.all(requests.map(req => req()))
  }

  // 防抖搜索
  debouncedSearch = debounce(async (query: string) => {
    return this.searchAPI(query)
  }, 300)

  // 节流更新
  throttledUpdate = throttle(async (data: any) => {
    return this.updateAPI(data)
  }, 1000)
}
```

## 监控与分析

### 1. 性能监控

```typescript
// utils/performance-monitor.ts
export class PerformanceMonitor {
  static measure(name: string, callback: () => void) {
    const start = performance.now()
    callback()
    const duration = performance.now() - start
    console.log(`${name} 耗时: ${duration.toFixed(2)}ms`)
    
    // 发送到监控系统
    if (duration > 100) {
      this.reportSlowOperation(name, duration)
    }
  }

  static startMeasure(name: string) {
    performance.mark(`${name}-start`)
  }

  static endMeasure(name: string) {
    performance.mark(`${name}-end`)
    performance.measure(name, `${name}-start`, `${name}-end`)
    
    const measures = performance.getEntriesByName(name)
    const duration = measures[0]?.duration || 0
    
    if (duration > 50) {
      console.warn(`操作 ${name} 耗时较长: ${duration.toFixed(2)}ms`)
    }
  }

  private static reportSlowOperation(name: string, duration: number) {
    // 上报到监控系统
    if (navigator.sendBeacon) {
      const data = new Blob([JSON.stringify({ name, duration })], {
        type: 'application/json'
      })
      navigator.sendBeacon('/api/performance', data)
    }
  }
}

// 使用性能监控
PerformanceMonitor.measure('heavyOperation', () => {
  // 执行昂贵操作
})

PerformanceMonitor.startMeasure('componentRender')
// 组件渲染逻辑
PerformanceMonitor.endMeasure('componentRender')
```

### 2. 内存监控

```typescript
// utils/memory-monitor.ts
export class MemoryMonitor {
  private static samples: number[] = []
  private static maxSamples = 100

  static startMonitoring(interval: number = 5000) {
    setInterval(() => {
      this.recordMemoryUsage()
    }, interval)
  }

  static recordMemoryUsage() {
    if (performance.memory) {
      const used = performance.memory.usedJSHeapSize
      this.samples.push(used)
      
      if (this.samples.length > this.maxSamples) {
        this.samples.shift()
      }

      // 检测内存泄漏
      if (this.isMemoryLeaking()) {
        console.warn('检测到可能的内存泄漏')
        this.reportMemoryLeak()
      }
    }
  }

  private static isMemoryLeaking(): boolean {
    if (this.samples.length < 10) return false
    
    const first = this.samples[0]
    const last = this.samples[this.samples.length - 1]
    const growthRate = (last - first) / first
    
    return growthRate > 0.1 // 内存增长超过10%
  }

  private static reportMemoryLeak() {
    // 上报内存泄漏信息
    const data = {
      type: 'memory_leak',
      samples: this.samples,
      timestamp: Date.now()
    }
    
    // 使用 sendBeacon 上报
    navigator.sendBeacon('/api/monitoring', JSON.stringify(data))
  }
}

// 启动内存监控
MemoryMonitor.startMonitoring()
```

## 最佳实践总结

### 1. 开发阶段优化

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    // 禁止使用可能影响性能的模式
    'vue/no-unused-components': 'error',
    'vue/no-unused-vars': 'error',
    'vue/no-mutating-props': 'error',
    // 强制使用 key 属性
    'vue/require-key-for-v-for': 'error'
  }
}

// 开发环境性能提示
if (process.env.NODE_ENV === 'development') {
  // 启用 Vue 性能提示
  app.config.performance = true
  
  // 开发环境监控
  const monitor = new DevelopmentMonitor()
  monitor.start()
}
```

### 2. 生产环境优化清单

| 优化项 | 检查点 | 状态 |
|-------|--------|------|
| 代码分割 | 路由懒加载配置 | ✅ |
| 资源压缩 | Gzip/Brotli 启用 | ✅ |
| 缓存策略 | 静态资源缓存头 | ✅ |
| 图片优化 | WebP格式，懒加载 | ✅ |
| 第三方库 | 按需引入，CDN | ✅ |
| 监控系统 | 性能指标收集 | ✅ |

### 3. 持续优化流程

```javascript
// 自动化性能测试
import { performance } from 'perf_hooks'

export function runPerformanceTests() {
  const tests = [
    {
      name: '组件渲染性能',
      test: async () => {
        const start = performance.now()
        // 渲染测试组件
        const duration = performance.now() - start
        return duration < 100 // 阈值检查
      }
    },
    {
      name: 'API响应时间',
      test: async () => {
        const response = await fetch('/api/health')
        return response.status === 200
      }
    }
  ]

  return Promise.all(tests.map(async test => {
    const result = await test.test()
    return { name: test.name, passed: result }
  }))
}

// 定期运行性能测试
setInterval(async () => {
  const results = await runPerformanceTests()
  console.log('性能测试结果:', results)
}, 3600000) // 每小时一次
```
