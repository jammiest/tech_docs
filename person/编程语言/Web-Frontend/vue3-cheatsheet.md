# Vue3 速查手册

Vue 3 是一个渐进式 JavaScript 框架，提供了更优秀的性能、更好的 TypeScript 支持和更灵活的 Composition API。本手册涵盖 Vue 3 的核心概念和常用模式。

## 核心概念

### 1. 应用创建与挂载

```javascript
// 创建应用
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// 全局配置
app.config.globalProperties.$myGlobal = '全局属性'
app.config.errorHandler = (err, instance, info) => {
  console.error('Vue 错误:', err)
}

// 挂载应用
app.mount('#app')
```

### 2. 选项式 API vs 组合式 API

#### 选项式 API (Options API)
```vue
<template>
  <div>{{ message }}</div>
  <button @click="increment">计数: {{ count }}</button>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello Vue 3!',
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  mounted() {
    console.log('组件已挂载')
  }
}
</script>
```

#### 组合式 API (Composition API)
```vue
<template>
  <div>{{ message }}</div>
  <button @click="increment">计数: {{ count }}</button>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const message = ref('Hello Vue 3!')
const count = ref(0)

const increment = () => {
  count.value++
}

onMounted(() => {
  console.log('组件已挂载')
})
</script>
```

## 响应式系统

### 1. 响应式基础

```javascript
import { ref, reactive, computed, watch, watchEffect } from 'vue'

// ref - 基本类型响应式
const count = ref(0)
console.log(count.value) // 访问值

// reactive - 对象响应式
const state = reactive({
  name: 'Vue',
  version: 3,
  features: ['Composition API', 'Better Performance']
})

// computed - 计算属性
const doubledCount = computed(() => count.value * 2)

// watch - 侦听器
watch(count, (newValue, oldValue) => {
  console.log(`计数从 ${oldValue} 变为 ${newValue}`)
})

// watchEffect - 立即执行侦听器
watchEffect(() => {
  console.log('计数变化:', count.value)
})
```

### 2. 响应式工具函数

```javascript
import { 
  isRef, 
  unref, 
  toRef, 
  toRefs, 
  markRaw,
  shallowRef,
  shallowReactive,
  readonly 
} from 'vue'

// 工具函数使用示例
const user = reactive({ name: 'John', age: 30 })

// 转换为 ref
const nameRef = toRef(user, 'name')
const { age } = toRefs(user)

// 检查类型
console.log(isRef(count)) // true
console.log(isRef(user)) // false

// 取消引用
const rawValue = unref(count)

// 只读对象
const readOnlyUser = readonly(user)

// 浅层响应式
const shallowState = shallowReactive({ nested: { data: 'test' } })
const shallowCount = shallowRef(0)
```

## 组件系统

### 1. 组件定义与使用

```vue
<!-- 父组件 ParentComponent.vue -->
<template>
  <ChildComponent 
    :title="pageTitle" 
    @update-title="updateTitle"
  >
    <template #default>默认插槽内容</template>
    <template #footer>页脚内容</template>
  </ChildComponent>
</template>

<script setup>
import ChildComponent from './ChildComponent.vue'
import { ref } from 'vue'

const pageTitle = ref('页面标题')

const updateTitle = (newTitle) => {
  pageTitle.value = newTitle
}
</script>

<!-- 子组件 ChildComponent.vue -->
<template>
  <div class="child">
    <h2>{{ title }}</h2>
    <slot></slot>
    <slot name="footer"></slot>
    <button @click="emitUpdate">更新标题</button>
  </div>
</template>

<script setup>
import { defineProps, defineEmits, defineExpose } from 'vue'

const props = defineProps({
  title: {
    type: String,
    required: true,
    default: '默认标题'
  }
})

const emit = defineEmits(['update-title'])

const emitUpdate = () => {
  emit('update-title', '新标题')
}

// 暴露给父组件的方法
const publicMethod = () => {
  console.log('公共方法被调用')
}

defineExpose({
  publicMethod
})
</script>
```

### 2. 组件通信

```javascript
// Props 传递
defineProps({
  // 基础类型检查
  title: String,
  
  // 多个类型
  content: [String, Number],
  
  // 必填项
  requiredProp: {
    type: String,
    required: true
  },
  
  // 默认值
  optionalProp: {
    type: Number,
    default: 100
  },
  
  // 自定义验证
  customProp: {
    validator(value) {
      return ['success', 'warning', 'danger'].includes(value)
    }
  }
})

// 事件发射
const emit = defineEmits(['update', 'delete', 'custom-event'])

// 触发事件
emit('update', newValue)
emit('delete', itemId)
emit('custom-event', { data: 'custom' })

// 使用 provide/inject
import { provide, inject } from 'vue'

// 祖先组件
provide('theme', 'dark')
provide('user', { name: 'John', id: 1 })

// 后代组件
const theme = inject('theme', 'light') // 默认值
const user = inject('user')
```

## 生命周期

### 1. 生命周期钩子

```vue
<script setup>
import { 
  onBeforeMount, 
  onMounted, 
  onBeforeUpdate, 
  onUpdated,
  onBeforeUnmount, 
  onUnmounted,
  onErrorCaptured,
  onRenderTracked,
  onRenderTriggered
} from 'vue'

onBeforeMount(() => {
  console.log('组件挂载前')
})

onMounted(() => {
  console.log('组件已挂载')
  // DOM 操作可以在这里进行
})

onBeforeUpdate(() => {
  console.log('组件更新前')
})

onUpdated(() => {
  console.log('组件已更新')
})

onBeforeUnmount(() => {
  console.log('组件卸载前')
})

onUnmounted(() => {
  console.log('组件已卸载')
  // 清理工作
})

onErrorCaptured((error, instance, info) => {
  console.error('错误捕获:', error)
  return false // 阻止错误继续向上传播
})

// 开发调试钩子
onRenderTracked((event) => {
  console.log('渲染跟踪:', event)
})

onRenderTriggered((event) => {
  console.log('渲染触发:', event)
})
</script>
```

### 2. 生命周期对比

| 选项式 API | 组合式 API | 描述 |
|------------|------------|------|
| beforeCreate | - | 实例初始化后，数据观测之前 |
| created | - | 实例创建完成，数据观测已建立 |
| beforeMount | onBeforeMount | 挂载开始之前 |
| mounted | onMounted | 实例挂载完成 |
| beforeUpdate | onBeforeUpdate | 数据更新时，DOM 更新前 |
| updated | onUpdated | 数据更新后，DOM 更新完成 |
| beforeUnmount | onBeforeUnmount | 实例销毁前 |
| unmounted | onUnmounted | 实例销毁后 |
| errorCaptured | onErrorCaptured | 捕获后代组件错误 |

## 模板语法

### 1. 指令系统

```vue
<template>
  <!-- 文本插值 -->
  <div>{{ message }}</div>
  
  <!-- 原始 HTML -->
  <div v-html="rawHtml"></div>
  
  <!-- 属性绑定 -->
  <div :id="dynamicId" :class="[activeClass, errorClass]"></div>
  <div :style="{ color: activeColor, fontSize: size + 'px' }"></div>
  
  <!-- 条件渲染 -->
  <div v-if="isVisible">条件渲染</div>
  <div v-else-if="isOtherCondition">其他条件</div>
  <div v-else>否则</div>
  <div v-show="isVisible">显示/隐藏</div>
  
  <!-- 列表渲染 -->
  <li v-for="(item, index) in items" :key="item.id">
    {{ index }} - {{ item.name }}
  </li>
  
  <!-- 事件处理 -->
  <button @click="handleClick">点击</button>
  <button @click="handleClick($event, '参数')">带参数点击</button>
  <form @submit.prevent="onSubmit">提交表单</form>
  
  <!-- 双向绑定 -->
  <input v-model="text" />
  <input v-model.trim="text" />
  <input v-model.number="age" />
  <input v-model.lazy="lazyText" />
  
  <!-- 动态指令 -->
  <div v-bind:[attributeName]="value"></div>
  <button @[eventName]="handler"></button>
  
  <!-- 插槽 -->
  <slot name="header"></slot>
  <slot :item="currentItem"></slot>
</template>
```

### 2. 自定义指令

```javascript
// 全局自定义指令
app.directive('focus', {
  mounted(el) {
    el.focus()
  }
})

// 局部自定义指令
const vFocus = {
  mounted: (el) => el.focus()
}

// 使用
<input v-focus />

// 完整指令钩子
const myDirective = {
  beforeMount(el, binding, vnode) {
    // 指令第一次绑定到元素时调用
  },
  mounted(el, binding, vnode) {
    // 元素插入父DOM时调用
  },
  beforeUpdate(el, binding, vnode, prevVnode) {
    // 元素更新前调用
  },
  updated(el, binding, vnode, prevVnode) {
    // 元素更新后调用
  },
  beforeUnmount(el, binding, vnode) {
    // 元素卸载前调用
  },
  unmounted(el, binding, vnode) {
    // 元素卸载后调用
  }
}
```

## 状态管理

### 1. Pinia (推荐)

```javascript
// store/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0,
    name: 'Counter Store'
  }),
  
  getters: {
    doubleCount: (state) => state.count * 2,
    doubleCountPlusOne() {
      return this.doubleCount + 1
    }
  },
  
  actions: {
    increment() {
      this.count++
    },
    async incrementAsync() {
      setTimeout(() => {
        this.increment()
      }, 1000)
    }
  }
})

// 组件中使用
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()

// 访问状态
console.log(counter.count)
console.log(counter.doubleCount)

// 调用action
counter.increment()

// 重置状态
counter.$reset()

// 订阅变化
counter.$subscribe((mutation, state) => {
  console.log('状态变化:', mutation, state)
})
```

### 2. 组合式函数状态管理

```javascript
// composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter() {
  const count = ref(0)
  
  const double = computed(() => count.value * 2)
  
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  return {
    count,
    double,
    increment,
    decrement
  }
}

// 组件中使用
import { useCounter } from '@/composables/useCounter'

const { count, double, increment } = useCounter()
```

## 路由系统 (Vue Router 4)

### 1. 路由配置

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue'),
    children: [
      {
        path: 'team',
        component: () => import('@/views/Team.vue')
      }
    ]
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: () => import('@/views/NotFound.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else {
      return { top: 0 }
    }
  }
})

// 导航守卫
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth && !isAuthenticated()) {
    next('/login')
  } else {
    next()
  }
})
```

### 2. 路由使用

```vue
<template>
  <!-- 路由链接 -->
  <router-link to="/">首页</router-link>
  <router-link :to="{ name: 'About' }">关于</router-link>
  <router-link :to="{ path: '/user', query: { id: 1 } }">用户</router-link>
  
  <!-- 路由视图 -->
  <router-view></router-view>
  <router-view name="sidebar"></router-view>
</template>

<script setup>
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// 编程式导航
const navigate = () => {
  router.push('/about')
  router.push({ name: 'Home' })
  router.replace('/login')
  router.go(-1)
}

// 访问路由信息
console.log(route.path)
console.log(route.params)
console.log(route.query)
</script>
```

## 性能优化

### 1. 组件优化

```vue
<template>
  <!-- 异步组件 -->
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <div>加载中...</div>
    </template>
  </Suspense>
  
  <!-- 保持组件状态 -->
  <KeepAlive :include="['ComponentA']" :max="10">
    <component :is="currentComponent"></component>
  </KeepAlive>
</template>

<script setup>
import { defineAsyncComponent } from 'vue'

// 异步组件
const AsyncComponent = defineAsyncComponent(() =>
  import('./AsyncComponent.vue')
)

// 懒加载组件
const LazyComponent = defineAsyncComponent({
  loader: () => import('./LazyComponent.vue'),
  loadingComponent: LoadingComponent,
  errorComponent: ErrorComponent,
  delay: 200,
  timeout: 3000
})
</script>
```

### 2. 渲染优化

```javascript
// 虚拟滚动
import { useVirtualList } from '@vueuse/core'

const { list, containerProps, wrapperProps } = useVirtualList(
  largeArray,
  { itemHeight: 50 }
)

// 防抖和节流
import { debounce, throttle } from 'lodash-es'

const debouncedSearch = debounce((query) => {
  searchAPI(query)
}, 300)

const throttledScroll = throttle(() => {
  handleScroll()
}, 100)
```

## TypeScript 支持

### 1. 类型定义

```typescript
// 组件 Props 类型
interface Props {
  title: string
  count?: number
  items: string[]
  onClick?: (event: MouseEvent) => void
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
  items: () => []
})

// 组件 Emits 类型
const emit = defineEmits<{
  (e: 'update', value: string): void
  (e: 'delete', id: number): void
}>()

// 模板 Ref 类型
const inputRef = ref<HTMLInputElement | null>(null)

// 组合式函数类型
interface UseCounterReturn {
  count: Ref<number>
  increment: () => void
  decrement: () => void
}

export function useCounter(): UseCounterReturn {
  const count = ref(0)
  
  const increment = () => {
    count.value++
  }
  
  const decrement = () => {
    count.value--
  }
  
  return {
    count,
    increment,
    decrement
  }
}
```

## 开发工具

### 1. Vue DevTools

```javascript
// 开发环境配置
if (process.env.NODE_ENV === 'development') {
  // 启用性能监测
  app.config.performance = true
  
  // 启用开发工具
  app.config.devtools = true
}

// 自定义审查器
app.config.globalProperties.$inspect = (value: any) => {
  console.log('审查:', value)
  return value
}
```

### 2. 调试技巧

```javascript
// 组件实例调试
const instance = getCurrentInstance()
console.log('组件实例:', instance)

// 响应式调试
import { debug } from 'vue'

debug('counter', count)

// 性能分析
const { pause, resume } = useRafFn(() => {
  // 高频操作
}, { immediate: true })
```
