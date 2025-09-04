# Hooks 深入解析

React Hooks 是 React 16.8 引入的革命性特性，它让函数组件能够使用状态和其他 React 特性。本文将深入探讨各种 Hooks 的工作原理、最佳实践和高级用法。

## 基础 Hooks

### 1. useState 深度解析

```jsx
import { useState } from 'react'

// 基础用法
const Counter = () => {
  const [count, setCount] = useState(0)
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>增加</button>
    </div>
  )
}

// 函数式更新
const AdvancedCounter = () => {
  const [count, setCount] = useState(0)
  
  // 批量更新示例
  const incrementTwice = () => {
    setCount(prev => prev + 1)
    setCount(prev => prev + 1) // 基于最新值
  }
  
  // 复杂状态对象
  const [user, setUser] = useState({
    name: '',
    age: 0,
    address: {
      city: '',
      street: ''
    }
  })
  
  const updateAddress = (field, value) => {
    setUser(prev => ({
      ...prev,
      address: {
        ...prev.address,
        [field]: value
      }
    }))
  }
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={incrementTwice}>增加两次</button>
      
      <input
        value={user.address.city}
        onChange={(e) => updateAddress('city', e.target.value)}
        placeholder="城市"
      />
    </div>
  )
}

// 惰性初始化
const LazyInitialization = () => {
  const [data, setData] = useState(() => {
    // 昂贵的计算只在初始渲染时执行
    const initialData = JSON.parse(localStorage.getItem('bigData') || '{}')
    return initialData
  })
  
  return <div>数据: {JSON.stringify(data)}</div>
}
```

### 2. useEffect 完整指南

```jsx
import { useEffect, useState } from 'react'

const EffectComponent = () => {
  const [count, setCount] = useState(0)
  const [data, setData] = useState(null)
  
  // 1. 无依赖数组 - 每次渲染都执行
  useEffect(() => {
    console.log('组件渲染或更新')
  })
  
  // 2. 空依赖数组 - 只在挂载时执行
  useEffect(() => {
    console.log('组件挂载')
    return () => console.log('组件卸载')
  }, [])
  
  // 3. 有依赖数组 - 依赖变化时执行
  useEffect(() => {
    document.title = `计数: ${count}`
  }, [count]) // 只在 count 变化时执行
  
  // 4. 清理函数
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(prev => prev + 1)
    }, 1000)
    
    return () => {
      clearInterval(timer)
      console.log('定时器清理')
    }
  }, [])
  
  // 5. 异步操作
  useEffect(() => {
    let isMounted = true
    
    const fetchData = async () => {
      try {
        const response = await fetch('/api/data')
        const result = await response.json()
        if (isMounted) {
          setData(result)
        }
      } catch (error) {
        if (isMounted) {
          console.error('获取数据失败:', error)
        }
      }
    }
    
    fetchData()
    
    return () => {
      isMounted = false
    }
  }, [])
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>增加</button>
    </div>
  )
}
```

### 3. useContext 使用模式

```jsx
import { createContext, useContext, useState } from 'react'

// 创建 Context
const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {}
})

const UserContext = createContext(null)

// Provider 组件
const AppProvider = ({ children }) => {
  const [theme, setTheme] = useState('light')
  const [user, setUser] = useState({ name: 'John', role: 'admin' })
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light')
  }
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <UserContext.Provider value={user}>
        {children}
      </UserContext.Provider>
    </ThemeContext.Provider>
  )
}

// 消费 Context
const ThemedButton = () => {
  const { theme, toggleTheme } = useContext(ThemeContext)
  
  return (
    <button
      onClick={toggleTheme}
      style={{
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#000' : '#fff'
      }}
    >
      切换主题
    </button>
  )
}

const UserProfile = () => {
  const user = useContext(UserContext)
  
  return (
    <div>
      <h3>用户信息</h3>
      <p>姓名: {user?.name}</p>
      <p>角色: {user?.role}</p>
    </div>
  )
}

// 组合使用
const App = () => (
  <AppProvider>
    <ThemedButton />
    <UserProfile />
  </AppProvider>
)
```

## 额外 Hooks

### 1. useReducer 复杂状态管理

```jsx
import { useReducer } from 'react'

// Reducer 函数
const todoReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [...state.todos, {
          id: Date.now(),
          text: action.payload,
          completed: false
        }]
      }
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      }
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      }
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload
      }
    default:
      return state
  }
}

// 初始状态
const initialState = {
  todos: [],
  filter: 'all'
}

// Action 创建函数
const addTodo = (text) => ({
  type: 'ADD_TODO',
  payload: text
})

const toggleTodo = (id) => ({
  type: 'TOGGLE_TODO',
  payload: id
})

const deleteTodo = (id) => ({
  type: 'DELETE_TODO',
  payload: id
})

const setFilter = (filter) => ({
  type: 'SET_FILTER',
  payload: filter
})

// 使用 useReducer
const TodoApp = () => {
  const [state, dispatch] = useReducer(todoReducer, initialState)
  
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'completed') return todo.completed
    if (state.filter === 'active') return !todo.completed
    return true
  })
  
  return (
    <div>
      <input
        type="text"
        onKeyPress={(e) => {
          if (e.key === 'Enter' && e.target.value.trim()) {
            dispatch(addTodo(e.target.value))
            e.target.value = ''
          }
        }}
        placeholder="添加待办事项"
      />
      
      <div>
        <button onClick={() => dispatch(setFilter('all'))}>全部</button>
        <button onClick={() => dispatch(setFilter('active'))}>未完成</button>
        <button onClick={() => dispatch(setFilter('completed'))}>已完成</button>
      </div>
      
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch(toggleTodo(todo.id))}
            />
            <span style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch(deleteTodo(todo.id))}>
              删除
            </button>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

### 2. useCallback 记忆化函数

```jsx
import { useCallback, useState, memo } from 'react'

// 使用 memo 优化子组件
const ExpensiveComponent = memo(({ data, onClick }) => {
  console.log('ExpensiveComponent 渲染')
  return (
    <div>
      <p>数据: {data}</p>
      <button onClick={onClick}>点击我</button>
    </div>
  )
})

const CallbackExample = () => {
  const [count, setCount] = useState(0)
  const [value, setValue] = useState('')
  
  // 没有 useCallback - 每次渲染都会创建新函数
  const handleClickBad = () => {
    console.log('点击处理', count)
  }
  
  // 使用 useCallback - 函数被记忆化
  const handleClickGood = useCallback(() => {
    console.log('点击处理', count)
  }, [count]) // 依赖 count，count 变化时重新创建
  
  return (
    <div>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="输入内容"
      />
      <p>计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>增加计数</button>
      
      {/* 不好的例子 - 每次渲染都会导致子组件重新渲染 */}
      <ExpensiveComponent data={value} onClick={handleClickBad} />
      
      {/* 好的例子 - 使用记忆化函数 */}
      <ExpensiveComponent data={value} onClick={handleClickGood} />
    </div>
  )
}
```

### 3. useMemo 记忆化值

```jsx
import { useMemo, useState } from 'react'

const MemoExample = () => {
  const [count, setCount] = useState(0)
  const [items, setItems] = useState([1, 2, 3, 4, 5])
  
  // 昂贵的计算
  const expensiveCalculation = (numbers) => {
    console.log('执行昂贵计算...')
    return numbers.reduce((acc, num) => acc + num, 0)
  }
  
  // 没有 useMemo - 每次渲染都重新计算
  const totalBad = expensiveCalculation(items)
  
  // 使用 useMemo - 只有 items 变化时重新计算
  const totalGood = useMemo(() => {
    return expensiveCalculation(items)
  }, [items])
  
  // 记忆化组件
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a - b)
  }, [items])
  
  // 记忆化对象（避免不必要的重新渲染）
  const config = useMemo(() => ({
    max: Math.max(...items),
    min: Math.min(...items),
    average: totalGood / items.length
  }), [items, totalGood])
  
  return (
    <div>
      <p>计数: {count}</p>
      <p>总和: {totalGood}</p>
      <p>配置: {JSON.stringify(config)}</p>
      
      <button onClick={() => setCount(count + 1)}>增加计数</button>
      <button onClick={() => setItems([...items, Math.random() * 10])}>
        添加随机数
      </button>
      
      <div>
        <h4>排序后的项目:</h4>
        {sortedItems.map(item => (
          <span key={item}>{item} </span>
        ))}
      </div>
    </div>
  )
}
```

### 4. useRef 引用管理

```jsx
import { useRef, useState, useEffect } from 'react'

const RefExample = () => {
  const [count, setCount] = useState(0)
  const inputRef = useRef(null)
  const previousCountRef = useRef(0)
  const renderCountRef = useRef(0)
  
  // 访问 DOM 元素
  const focusInput = () => {
    inputRef.current?.focus()
  }
  
  // 保存之前的值
  useEffect(() => {
    previousCountRef.current = count
  }, [count])
  
  // 记录渲染次数
  renderCountRef.current++
  
  // 存储可变值（不会触发重新渲染）
  const timeoutRef = useRef(null)
  
  const startTimer = () => {
    timeoutRef.current = setTimeout(() => {
      console.log('定时器执行')
    }, 1000)
  }
  
  const clearTimer = () => {
    clearTimeout(timeoutRef.current)
  }
  
  return (
    <div>
      <p>当前计数: {count}</p>
      <p>之前计数: {previousCountRef.current}</p>
      <p>渲染次数: {renderCountRef.current}</p>
      
      <input ref={inputRef} placeholder="焦点在这里" />
      <button onClick={focusInput}>聚焦输入框</button>
      
      <button onClick={() => setCount(count + 1)}>增加计数</button>
      <button onClick={startTimer}>开始定时器</button>
      <button onClick={clearTimer}>清除定时器</button>
    </div>
  )
}

// 自定义 Hook 使用 useRef
const usePrevious = (value) => {
  const ref = useRef()
  useEffect(() => {
    ref.current = value
  })
  return ref.current
}

const PreviousValueExample = () => {
  const [value, setValue] = useState('')
  const previousValue = usePrevious(value)
  
  return (
    <div>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="输入内容"
      />
      <p>当前值: {value}</p>
      <p>之前值: {previousValue}</p>
    </div>
  )
}
```

## 自定义 Hooks

### 1. 状态管理自定义 Hook

```jsx
import { useState, useEffect, useCallback } from 'react'

// 自定义 Hook: useLocalStorage
const useLocalStorage = (key, initialValue) => {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      console.error('读取 localStorage 失败:', error)
      return initialValue
    }
  })
  
  const setValue = useCallback((value) => {
    try {
      setStoredValue(value)
      window.localStorage.setItem(key, JSON.stringify(value))
    } catch (error) {
      console.error('设置 localStorage 失败:', error)
    }
  }, [key])
  
  return [storedValue, setValue]
}

// 自定义 Hook: useToggle
const useToggle = (initialValue = false) => {
  const [value, setValue] = useState(initialValue)
  
  const toggle = useCallback(() => {
    setValue(prev => !prev)
  }, [])
  
  const setTrue = useCallback(() => {
    setValue(true)
  }, [])
  
  const setFalse = useCallback(() => {
    setValue(false)
  }, [])
  
  return [value, { toggle, setTrue, setFalse }]
}

// 自定义 Hook: useFetch
const useFetch = (url, options = {}) => {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)
  
  const fetchData = useCallback(async () => {
    try {
      setLoading(true)
      setError(null)
      
      const response = await fetch(url, options)
      if (!response.ok) {
        throw new Error(`HTTP错误: ${response.status}`)
      }
      
      const result = await response.json()
      setData(result)
    } catch (err) {
      setError(err.message)
    } finally {
      setLoading(false)
    }
  }, [url, options])
  
  useEffect(() => {
    fetchData()
  }, [fetchData])
  
  const refetch = () => {
    fetchData()
  }
  
  return { data, loading, error, refetch }
}

// 使用自定义 Hooks
const ExampleComponent = () => {
  const [name, setName] = useLocalStorage('username', '')
  const [isVisible, { toggle, setTrue, setFalse }] = useToggle(false)
  const { data, loading, error } = useFetch('/api/user')
  
  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="用户名"
      />
      
      <button onClick={toggle}>
        {isVisible ? '隐藏' : '显示'}
      </button>
      
      {loading && <p>加载中...</p>}
      {error && <p>错误: {error}</p>}
      {data && <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  )
}
```

### 2. UI 交互自定义 Hook

```jsx
import { useState, useCallback, useEffect } from 'react'

// 自定义 Hook: useHover
const useHover = () => {
  const [isHovered, setIsHovered] = useState(false)
  
  const handleMouseEnter = useCallback(() => {
    setIsHovered(true)
  }, [])
  
  const handleMouseLeave = useCallback(() => {
    setIsHovered(false)
  }, [])
  
  return {
    isHovered,
    bind: {
      onMouseEnter: handleMouseEnter,
      onMouseLeave: handleMouseLeave
    }
  }
}

// 自定义 Hook: useClickOutside
const useClickOutside = (callback) => {
  const ref = useRef()
  
  useEffect(() => {
    const handleClick = (event) => {
      if (ref.current && !ref.current.contains(event.target)) {
        callback()
      }
    }
    
    document.addEventListener('mousedown', handleClick)
    return () => {
      document.removeEventListener('mousedown', handleClick)
    }
  }, [callback])
  
  return ref
}

// 自定义 Hook: useKeyPress
const useKeyPress = (targetKey) => {
  const [keyPressed, setKeyPressed] = useState(false)
  
  const downHandler = useCallback(({ key }) => {
    if (key === targetKey) {
      setKeyPressed(true)
    }
  }, [targetKey])
  
  const upHandler = useCallback(({ key }) => {
    if (key === targetKey) {
      setKeyPressed(false)
    }
  }, [targetKey])
  
  useEffect(() => {
    window.addEventListener('keydown', downHandler)
    window.addEventListener('keyup', upHandler)
    
    return () => {
      window.removeEventListener('keydown', downHandler)
      window.removeEventListener('keyup', upHandler)
    }
  }, [downHandler, upHandler])
  
  return keyPressed
}

// 使用自定义 Hooks
const InteractiveComponent = () => {
  const { isHovered, bind } = useHover()
  const [isOpen, setIsOpen] = useState(false)
  const ref = useClickOutside(() => setIsOpen(false))
  const enterPressed = useKeyPress('Enter')
  
  return (
    <div>
      <div
        {...bind}
        style={{
          padding: '20px',
          background: isHovered ? '#f0f0f0' : '#fff',
          border: '1px solid #ccc'
        }}
      >
        悬停我 {isHovered ? '😊' : '😐'}
      </div>
      
      <button onClick={() => setIsOpen(true)}>
        打开菜单
      </button>
      
      {isOpen && (
        <div ref={ref} style={{ border: '1px solid blue', padding: '10px' }}>
          点击外部关闭我
        </div>
      )}
      
      {enterPressed && <p>Enter 键被按下!</p>}
    </div>
  )
}
```

## 高级 Hooks 模式

### 1. 依赖数组高级用法

```jsx
import { useEffect, useState, useRef } from 'react'

const AdvancedDependencies = () => {
  const [count, setCount] = useState(0)
  const [user, setUser] = useState({ id: 1, name: 'John' })
  const previousUserRef = useRef(user)
  
  // 1. 对象依赖 - 使用引用比较
  useEffect(() => {
    console.log('user 对象变化')
  }, [user]) // 注意：每次渲染 user 都是新对象
  
  // 2. 函数依赖
  const fetchData = useCallback(async () => {
    // 数据获取逻辑
  }, []) // 空依赖，函数不会变化
  
  useEffect(() => {
    fetchData()
  }, [fetchData])
  
  // 3. 数组依赖
  const items = [1, 2, 3]
  useEffect(() => {
    console.log('items 变化')
  }, [items]) // 每次渲染 items 都是新数组
  
  // 4. 深度比较依赖（使用 JSON.stringify）
  const userString = JSON.stringify(user)
  useEffect(() => {
    console.log('user 深度变化')
  }, [userString])
  
  // 5. 自定义比较逻辑
  useEffect(() => {
    if (user.id !== previousUserRef.current.id) {
      console.log('用户 ID 变化')
      previousUserRef.current = user
    }
  }, [user])
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>计数: {count}</button>
      <button onClick={() => setUser({ ...user, name: 'Jane' })}>
        更新用户
      </button>
    </div>
  )
}
```

### 2. Hooks 执行顺序规则

```jsx
const HookOrderRules = () => {
  // 1. Hooks 必须在顶层调用
  // ❌ 错误：在条件语句中调用
  // if (condition) {
  //   const [value, setValue] = useState('')
  // }
  
  // ✅ 正确：始终在顶层调用
  const [value, setValue] = useState('')
  const [count, setCount] = useState(0)
  
  // 2. 只能在 React 函数中调用 Hooks
  // ❌ 错误：在普通函数中调用
  // function regularFunction() {
  //   const [state, setState] = useState('')
  // }
  
  // 3. 自定义 Hook 必须使用 use 前缀
  const useCustomHook = () => {
    const [state, setState] = useState('')
    return [state, setState]
  }
  
  return (
    <div>
      <p>值: {value}</p>
      <p>计数: {count}</p>
    </div>
  )
}

// 自定义 Hook 示例
const useCustomHook = () => {
  const [state, setState] = useState('')
  useEffect(() => {
    // 副作用逻辑
  }, [])
  
  return [state, setState]
}
```

## 性能优化模式

### 1. Hooks 性能优化

```jsx
import { useState, useCallback, useMemo, memo } from 'react'

// 使用 React.memo 优化子组件
const ExpensiveChild = memo(({ data, onClick }) => {
  console.log('子组件渲染')
  return (
    <div>
      <p>数据: {data}</p>
      <button onClick={onClick}>点击</button>
    </div>
  )
})

const OptimizedParent = () => {
  const [count, setCount] = useState(0)
  const [text, setText] = useState('')
  
  // 优化函数：使用 useCallback
  const handleClick = useCallback(() => {
    console.log('点击处理:', count)
  }, [count])
  
  // 优化数据：使用 useMemo
  const processedData = useMemo(() => {
    return expensiveProcessing(text)
  }, [text])
  
  return (
    <div>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="输入文本"
      />
      <button onClick={() => setCount(count + 1)}>
        计数: {count}
      </button>
      
      <ExpensiveChild
        data={processedData}
        onClick={handleClick}
      />
    </div>
  )
}

// 昂贵的处理函数
const expensiveProcessing = (text) => {
  // 模拟昂贵计算
  return text.split('').reverse().join('')
}
```

### 2. 避免常见陷阱

```jsx
const CommonPitfalls = () => {
  const [count, setCount] = useState(0)
  
  // 陷阱 1: 无限循环
  // useEffect(() => {
  //   setCount(count + 1) // 会导致无限重新渲染
  // }, [count])
  
  // 正确方式：使用函数式更新或调整依赖
  useEffect(() => {
    // 安全的更新
  }, []) // 空依赖或正确的依赖
  
  // 陷阱 2: 陈旧的闭包
  const handleClick = () => {
    setTimeout(() => {
      console.log(count) // 显示的是创建时的 count 值
    }, 1000)
  }
  
  // 解决方案：使用 ref 或函数式更新
  const countRef = useRef(count)
  countRef.current = count
  
  const handleClickFixed = () => {
    setTimeout(() => {
      console.log(countRef.current) // 总是最新值
    }, 1000)
  }
  
  // 陷阱 3: 不必要的重新渲染
  const [user, setUser] = useState({ name: 'John' })
  
  useEffect(() => {
    // 每次渲染都会执行，因为 user 总是新对象
  }, [user])
  
  // 解决方案：使用 useMemo 或 useCallback
  const stableUser = useMemo(() => user, [user.name])
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={handleClickFixed}>测试闭包</button>
    </div>
  )
}
```

## 测试策略

### 1. Hooks 测试方法

```jsx
// hooks/useCounter.test.js
import { renderHook, act } from '@testing-library/react-hooks'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('应该初始化计数为0', () => {
    const { result } = renderHook(() => useCounter())
    expect(result.current.count).toBe(0)
  })
  
  it('应该增加计数', () => {
    const { result } = renderHook(() => useCounter())
    
    act(() => {
      result.current.increment()
    })
    
    expect(result.current.count).toBe(1)
  })
  
  it('应该重置计数', () => {
    const { result } = renderHook(() => useCounter())
    
    act(() => {
      result.current.increment()
      result.current.reset()
    })
    
    expect(result.current.count).toBe(0)
  })
})

// hooks/useFetch.test.js
import { renderHook } from '@testing-library/react-hooks'
import { useFetch } from './useFetch'

describe('useFetch', () => {
  it('应该处理加载状态', async () => {
    global.fetch = jest.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ data: 'test' })
    })
    
    const { result, waitForNextUpdate } = renderHook(() => 
      useFetch('/api/test')
    )
    
    expect(result.current.loading).toBe(true)
    
    await waitForNextUpdate()
    
    expect(result.current.loading).toBe(false)
    expect(result.current.data).toEqual({ data: 'test' })
  })
})
```
