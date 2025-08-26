# Vue 3 速查手册

## 🎯 组合式 API (Composition API)

### 响应式基础
```javascript
import { ref, reactive, computed, watch, watchEffect } from 'vue'

// 响应式变量
const count = ref(0)          // 基本类型
const user = reactive({       // 对象类型
  name: 'John',
  age: 30
})

// 计算属性
const doubleCount = computed(() => count.value * 2)
const fullName = computed({
  get: () => `${user.name} Doe`,
  set: (val) => { user.name = val.split(' ')[0] }
})

// 侦听器
watch(count, (newVal, oldVal) => {
  console.log(`count changed from ${oldVal} to ${newVal}`)
})

watchEffect(() => {
  console.log(`count is: ${count.value}`)
})
```

### 生命周期钩子
```javascript
import { onMounted, onUpdated, onUnmounted } from 'vue'

onMounted(() => {
  console.log('组件挂载完成')
})

onUpdated(() => {
  console.log('组件更新完成')
})

onUnmounted(() => {
  console.log('组件卸载完成')
})

// 所有生命周期钩子：
// onBeforeMount, onMounted, onBeforeUpdate, onUpdated,
// onBeforeUnmount, onUnmounted, onErrorCaptured,
// onRenderTracked, onRenderTriggered
```

## 📝 模板语法

### 文本插值
```vue
<template>
  <span>{{ message }}</span>
  <span v-once>{{ neverChange }}</span> <!-- 一次性插值 -->
</template>
```

### 指令
```vue
<template>
  <!-- 条件渲染 -->
  <div v-if="isVisible">显示内容</div>
  <div v-else-if="otherCondition">其他内容</div>
  <div v-else>默认内容</div>
  <div v-show="isVisible">显示/隐藏（CSS控制）</div>

  <!-- 列表渲染 -->
  <li v-for="(item, index) in items" :key="item.id">
    {{ index }}: {{ item.name }}
  </li>

  <!-- 属性绑定 -->
  <div :class="{ active: isActive, 'text-danger': hasError }"></div>
  <div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

  <!-- 事件处理 -->
  <button @click="handleClick">点击</button>
  <button @click.stop="handleClick">阻止冒泡</button>
  <button @click.prevent="handleClick">阻止默认</button>
  <input @keyup.enter="submit">

  <!-- 表单绑定 -->
  <input v-model="message">
  <input v-model.trim="message">   <!-- 自动去除首尾空格 -->
  <input v-model.number="age">     <!-- 自动转为数字 -->
  <input v-model.lazy="message">   <!-- change事件后更新 -->
</template>
```

## 🔧 组件相关

### 组件定义
```vue
<script setup>
// 组合式 API + <script setup>
import { defineProps, defineEmits, defineExpose } from 'vue'

// 定义 Props
const props = defineProps({
  title: String,
  count: {
    type: Number,
    default: 0,
    validator: (value) => value >= 0
  }
})

// 定义 Emits
const emit = defineEmits(['update:count', 'change'])

// 暴露给父组件的方法
defineExpose({
  someMethod() {
    console.log('暴露的方法')
  }
})
</script>
```

### 组件通信
```vue
<!-- 父组件 -->
<template>
  <ChildComponent 
    :title="pageTitle" 
    @update-title="updateTitle"
    v-model:count="countValue"
  />
</template>

<!-- 子组件 -->
<script setup>
defineProps(['title'])
defineEmits(['update-title'])

// 使用 v-model
const props = defineProps(['count'])
const emit = defineEmits(['update:count'])
</script>
```

## 🎨 响应式进阶

### 响应式工具
```javascript
import { 
  readonly, 
  shallowRef, 
  shallowReactive, 
  toRef, 
  toRefs,
  markRaw 
} from 'vue'

// 只读代理
const readOnlyUser = readonly(user)

// 浅层响应式
const shallowUser = shallowReactive({ name: 'John', details: { age: 30 } })
const shallowCount = shallowRef(0)

// 转换响应式
const nameRef = toRef(user, 'name')
const { name, age } = toRefs(user)

// 标记非响应式
const rawObject = markRaw({ shouldNotBeReactive: true })
```

### 自定义 Ref
```javascript
import { customRef } from 'vue'

function useDebouncedRef(value, delay = 200) {
  return customRef((track, trigger) => {
    let timeout
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
          value = newValue
          trigger()
        }, delay)
      }
    }
  })
}
```

## 📚 依赖注入

### Provide / Inject
```vue
<!-- 祖先组件 -->
<script setup>
import { provide, ref } from 'vue'

const theme = ref('dark')
provide('theme', theme)

// 提供方法
provide('updateTheme', (newTheme) => {
  theme.value = newTheme
})
</script>

<!-- 后代组件 -->
<script setup>
import { inject } from 'vue'

const theme = inject('theme', 'light') // 默认值
const updateTheme = inject('updateTheme')
</script>
```

## 🎪 内置组件

### Teleport
```vue
<template>
  <teleport to="body">
    <div class="modal">
      模态框内容（会渲染到body末尾）
    </div>
  </teleport>
</template>
```

### Suspense
```vue
<template>
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <div>加载中...</div>
    </template>
  </Suspense>
</template>
```

### KeepAlive
```vue
<template>
  <KeepAlive :include="['ComponentA']" :max="10">
    <component :is="currentComponent" />
  </KeepAlive>
</template>
```

## 🔄 路由 (Vue Router 4)

### 路由配置
```javascript
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('./views/Home.vue')
  },
  {
    path: '/user/:id',
    name: 'User',
    props: true,
    component: () => import('./views/User.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})
```

### 路由使用
```vue
<script setup>
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// 编程式导航
router.push('/home')
router.push({ name: 'User', params: { id: 1 } })

// 获取路由参数
const userId = route.params.id
</script>

<template>
  <router-link to="/">首页</router-link>
  <router-link :to="{ name: 'User', params: { id: 1 } }">用户</router-link>
  <router-view />
</template>
```

## 🏪 状态管理 (Pinia)

### Store 定义
```javascript
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0
  }),
  getters: {
    doubleCount: (state) => state.count * 2
  },
  actions: {
    increment() {
      this.count++
    }
  }
})
```

### Store 使用
```vue
<script setup>
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()

// 访问状态
console.log(counter.count)

// 使用getter
console.log(counter.doubleCount)

// 调用action
counter.increment()

// 重置状态
counter.$reset()

// 监听变化
counter.$subscribe((mutation, state) => {
  console.log('状态变化', mutation, state)
})
</script>
```

## 🎭 自定义指令

### 全局指令
```javascript
// main.js
app.directive('focus', {
  mounted(el) {
    el.focus()
  }
})
```

### 局部指令
```vue
<script setup>
const vColor = {
  mounted(el, binding) {
    el.style.color = binding.value
  },
  updated(el, binding) {
    el.style.color = binding.value
  }
}
</script>

<template>
  <div v-color="'red'">红色文字</div>
</template>
```

## 🔧 实用工具

### 下一帧更新
```javascript
import { nextTick } from 'vue'

await nextTick()
// DOM 已更新
```

### 版本信息
```javascript
import { version } from 'vue'

console.log(`Vue版本: ${version}`)
```

## 📱 组合式函数 (Composables)

### 自定义组合式函数
```javascript
// composables/useMouse.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useMouse() {
  const x = ref(0)
  const y = ref(0)

  function update(event) {
    x.value = event.pageX
    y.value = event.pageY
  }

  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))

  return { x, y }
}
```

### 使用组合式函数
```vue
<script setup>
import { useMouse } from './composables/useMouse'

const { x, y } = useMouse()
</script>

<template>
  鼠标位置: {{ x }}, {{ y }}
</template>
```

## 🚀 性能优化

### 懒加载组件
```vue
<script setup>
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() =>
  import('./components/HeavyComponent.vue')
)
</script>

<template>
  <AsyncComponent />
</template>
```

### 虚拟滚动
```vue
<template>
  <RecycleScroller
    :items="largeList"
    :item-size="50"
    key-field="id"
  >
    <template #default="{ item }">
      <div>{{ item.name }}</div>
    </template>
  </RecycleScroller>
</template>
```

这个速查表涵盖了 Vue 3 的核心特性和常用模式，适合日常开发中快速查阅。建议结合官方文档深入学习各个特性的详细用法。