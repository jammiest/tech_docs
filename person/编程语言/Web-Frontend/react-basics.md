# React 基础概念

React 是一个用于构建用户界面的 JavaScript 库，以其组件化、声明式和高效的虚拟 DOM 机制著称。本文将全面介绍 React 的核心概念和基础用法。

## 核心哲学

### 1. 声明式编程

```jsx
// 命令式编程（不推荐）
const updateUI = () => {
  const container = document.getElementById('app')
  container.innerHTML = ''
  const button = document.createElement('button')
  button.textContent = '点击我'
  button.onclick = handleClick
  container.appendChild(button)
}

// 声明式编程（React方式）
const App = () => {
  const [count, setCount] = useState(0)
  
  const handleClick = () => {
    setCount(count + 1)
  }
  
  return (
    <div>
      <button onClick={handleClick}>
        点击我: {count}
      </button>
    </div>
  )
}
```

### 2. 组件化思维

```jsx
// 组件就像乐高积木，可以组合和复用
const UserProfile = ({ user }) => (
  <div className="user-profile">
    <Avatar src={user.avatar} />
    <UserInfo name={user.name} email={user.email} />
    <ActionButtons onEdit={handleEdit} onDelete={handleDelete} />
  </div>
)

// 组合使用
const App = () => (
  <div className="app">
    <Header />
    <Sidebar />
    <MainContent>
      <UserProfile user={currentUser} />
      <PostList posts={userPosts} />
    </MainContent>
    <Footer />
  </div>
)
```

## JSX 语法

### 1. JSX 基础

```jsx
// 基本 JSX
const element = <h1>Hello, React!</h1>

// 嵌入表达式
const name = 'John'
const element = <h1>Hello, {name}!</h1>

// JSX 也是表达式
const getGreeting = (user) => {
  if (user) {
    return <h1>Hello, {user.name}!</h1>
  }
  return <h1>Hello, Stranger!</h1>
}

// 属性使用
const element = <img src={user.avatarUrl} alt={user.name} className="avatar" />

// 防止注入攻击
const title = response.potentiallyMaliciousInput
const element = <h1>{title}</h1> // 自动转义
```

### 2. JSX 高级用法

```jsx
// 条件渲染
const WelcomeMessage = ({ isLoggedIn, userName }) => (
  <div>
    {isLoggedIn ? (
      <h1>Welcome back, {userName}!</h1>
    ) : (
      <h1>Please sign in.</h1>
    )}
  </div>
)

// 列表渲染
const NumberList = ({ numbers }) => (
  <ul>
    {numbers.map(number => (
      <li key={number.id}>{number.value}</li>
    ))}
  </ul>
)

// 片段包裹
const FragmentExample = () => (
  <>
    <td>First</td>
    <td>Second</td>
  </>
)

// 字符串字面量
const messages = [
  'Hello React',
  'Learn once, write anywhere'
]

const MessageList = () => (
  <div>
    {messages.map((message, index) => (
      <p key={index}>{message}</p>
    ))}
  </div>
)
```

## 组件系统

### 1. 函数组件

```jsx
// 基础函数组件
const Welcome = (props) => {
  return <h1>Hello, {props.name}</h1>
}

// 使用箭头函数
const Button = ({ onClick, children, variant = 'primary' }) => (
  <button 
    className={`btn btn-${variant}`}
    onClick={onClick}
  >
    {children}
  </button>
)

// 带默认值的组件
const Card = ({ 
  title = '默认标题', 
  children, 
  className = '' 
}) => (
  <div className={`card ${className}`}>
    {title && <h3 className="card-title">{title}</h3>}
    <div className="card-content">
      {children}
    </div>
  </div>
)
```

### 2. Props 系统

```jsx
// Props 类型检查
import PropTypes from 'prop-types'

const UserProfile = ({ user, onEdit, isEditable }) => (
  <div className="user-profile">
    <h2>{user.name}</h2>
    <p>{user.email}</p>
    {isEditable && (
      <button onClick={onEdit}>编辑</button>
    )}
  </div>
)

UserProfile.propTypes = {
  user: PropTypes.shape({
    name: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired
  }).isRequired,
  onEdit: PropTypes.func,
  isEditable: PropTypes.bool
}

UserProfile.defaultProps = {
  isEditable: false,
  onEdit: () => {}
}

// Children Props
const Container = ({ children, title }) => (
  <div className="container">
    {title && <h2>{title}</h2>}
    <div className="content">
      {children}
    </div>
  </div>
)

// 使用 children
const App = () => (
  <Container title="用户列表">
    <UserList users={users} />
    <Pagination />
  </Container>
)
```

## 状态管理

### 1. useState Hook

```jsx
import { useState } from 'react'

// 基础用法
const Counter = () => {
  const [count, setCount] = useState(0)
  
  const increment = () => {
    setCount(count + 1)
  }
  
  const decrement = () => {
    setCount(count - 1)
  }
  
  const reset = () => {
    setCount(0)
  }
  
  return (
    <div>
      <p>当前计数: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>重置</button>
    </div>
  )
}

// 对象状态
const UserForm = () => {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: ''
  })
  
  const handleChange = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }))
  }
  
  return (
    <form>
      <input
        value={user.name}
        onChange={(e) => handleChange('name', e.target.value)}
        placeholder="姓名"
      />
      <input
        value={user.email}
        onChange={(e) => handleChange('email', e.target.value)}
        placeholder="邮箱"
      />
      <input
        value={user.age}
        onChange={(e) => handleChange('age', e.target.value)}
        placeholder="年龄"
      />
    </form>
  )
}

// 数组状态
const TodoList = () => {
  const [todos, setTodos] = useState([])
  const [inputValue, setInputValue] = useState('')
  
  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos(prevTodos => [
        ...prevTodos,
        {
          id: Date.now(),
          text: inputValue,
          completed: false
        }
      ])
      setInputValue('')
    }
  }
  
  const toggleTodo = (id) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    )
  }
  
  return (
    <div>
      <input
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="添加待办事项"
      />
      <button onClick={addTodo}>添加</button>
      <ul>
        {todos.map(todo => (
          <li
            key={todo.id}
            onClick={() => toggleTodo(todo.id)}
            style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  )
}
```

### 2. 状态更新模式

```jsx
// 函数式更新（推荐）
const Counter = () => {
  const [count, setCount] = useState(0)
  
  const increment = () => {
    setCount(prevCount => prevCount + 1)
  }
  
  const incrementMultiple = () => {
    setCount(prevCount => prevCount + 1)
    setCount(prevCount => prevCount + 1) // 基于最新值
  }
  
  return <button onClick={incrementMultiple}>增加两次: {count}</button>
}

// 状态合并（对象）
const Form = () => {
  const [formData, setFormData] = useState({
    username: '',
    password: '',
    rememberMe: false
  })
  
  const updateField = (field, value) => {
    setFormData(prev => ({
      ...prev,
      [field]: value
    }))
  }
  
  return (
    <form>
      <input
        value={formData.username}
        onChange={(e) => updateField('username', e.target.value)}
      />
      <input
        type="password"
        value={formData.password}
        onChange={(e) => updateField('password', e.target.value)}
      />
      <input
        type="checkbox"
        checked={formData.rememberMe}
        onChange={(e) => updateField('rememberMe', e.target.checked)}
      />
    </form>
  )
}
```

## 事件处理

### 1. 事件基础

```jsx
const EventHandling = () => {
  // 基本事件处理
  const handleClick = (event) => {
    event.preventDefault()
    console.log('按钮被点击', event)
  }
  
  // 带参数的事件处理
  const handleItemClick = (itemId, event) => {
    console.log('项目被点击:', itemId, event)
  }
  
  // 合成事件
  const handleInputChange = (event) => {
    console.log('输入值:', event.target.value)
  }
  
  return (
    <div>
      <button onClick={handleClick}>点击我</button>
      
      <ul>
        {[1, 2, 3].map(item => (
          <li
            key={item}
            onClick={(e) => handleItemClick(item, e)}
          >
            项目 {item}
          </li>
        ))}
      </ul>
      
      <input
        type="text"
        onChange={handleInputChange}
        placeholder="输入内容"
      />
    </div>
  )
}
```

### 2. 事件优化

```jsx
import { useCallback } from 'react'

const OptimizedEvents = () => {
  const [count, setCount] = useState(0)
  
  // 使用 useCallback 避免不必要的重新创建
  const handleClick = useCallback(() => {
    setCount(prevCount => prevCount + 1)
  }, [])
  
  // 带依赖的处理函数
  const handleSubmit = useCallback((data) => {
    console.log('提交数据:', data, count)
  }, [count])
  
  // 事件代理
  const handleListClick = useCallback((event) => {
    if (event.target.tagName === 'LI') {
      console.log('列表项被点击:', event.target.dataset.id)
    }
  }, [])
  
  return (
    <div>
      <button onClick={handleClick}>计数: {count}</button>
      
      <ul onClick={handleListClick}>
        <li data-id="1">项目 1</li>
        <li data-id="2">项目 2</li>
        <li data-id="3">项目 3</li>
      </ul>
    </div>
  )
}
```

## 条件渲染

### 1. 条件渲染模式

```jsx
const ConditionalRendering = () => {
  const [isLoggedIn, setIsLoggedIn] = useState(false)
  const [userRole, setUserRole] = useState('user')
  const [items, setItems] = useState([])
  
  return (
    <div>
      {/* && 操作符 */}
      {isLoggedIn && <WelcomeMessage />}
      
      {/* 三元操作符 */}
      {isLoggedIn ? (
        <LogoutButton onClick={() => setIsLoggedIn(false)} />
      ) : (
        <LoginButton onClick={() => setIsLoggedIn(true)} />
      )}
      
      {/* 多条件渲染 */}
      {userRole === 'admin' && isLoggedIn && (
        <AdminPanel />
      )}
      
      {/* 空状态处理 */}
      {items.length === 0 ? (
        <EmptyState message="暂无数据" />
      ) : (
        <ItemList items={items} />
      )}
      
      {/* 提前返回 */}
      {!isLoggedIn && (
        <div>请先登录</div>
      )}
      
      {/* 使用组件封装条件逻辑 */}
      <AccessControl
        requiredRole="admin"
        currentRole={userRole}
        fallback={<div>权限不足</div>}
      >
        <AdminContent />
      </AccessControl>
    </div>
  )
}

// 封装条件组件
const AccessControl = ({ requiredRole, currentRole, children, fallback = null }) => {
  return currentRole === requiredRole ? children : fallback
}
```

### 2. 渲染优化技巧

```jsx
const RenderingOptimization = () => {
  const [visible, setVisible] = useState(false)
  const [data, setData] = useState(null)
  
  // 避免不必要的渲染
  if (!visible) {
    return null
  }
  
  // 复杂条件提取
  const shouldShowDetails = data && data.status === 'completed'
  
  return (
    <div>
      {/* 使用变量存储JSX */}
      {shouldShowDetails && (
        <div className="details">
          {/* 复杂内容 */}
        </div>
      )}
      
      {/* 使用函数返回JSX */}
      {renderContent()}
    </div>
  )
  
  function renderContent() {
    if (!data) {
      return <LoadingSpinner />
    }
    
    if (data.error) {
      return <ErrorMessage error={data.error} />
    }
    
    return <DataDisplay data={data} />
  }
}
```

## 列表渲染

### 1. 列表渲染基础

```jsx
const ListRendering = () => {
  const [users, setUsers] = useState([
    { id: 1, name: 'Alice', age: 25 },
    { id: 2, name: 'Bob', age: 30 },
    { id: 3, name: 'Charlie', age: 35 }
  ])
  
  const [selectedUserId, setSelectedUserId] = useState(null)
  
  return (
    <div>
      <h2>用户列表</h2>
      
      {/* 基础列表渲染 */}
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.age}岁
          </li>
        ))}
      </ul>
      
      {/* 带事件的列表 */}
      <div className="user-grid">
        {users.map(user => (
          <UserCard
            key={user.id}
            user={user}
            isSelected={selectedUserId === user.id}
            onSelect={() => setSelectedUserId(user.id)}
          />
        ))}
      </div>
      
      {/* 条件列表渲染 */}
      {users.length > 0 ? (
        <UserTable users={users} />
      ) : (
        <div>暂无用户数据</div>
      )}
    </div>
  )
}

// 列表项组件
const UserCard = ({ user, isSelected, onSelect }) => (
  <div
    className={`user-card ${isSelected ? 'selected' : ''}`}
    onClick={onSelect}
  >
    <h3>{user.name}</h3>
    <p>{user.age}岁</p>
  </div>
)
```

### 2. 列表性能优化

```jsx
import { memo, useCallback } from 'react'

// 使用 memo 优化列表项
const OptimizedUserCard = memo(({ user, onSelect, isSelected }) => (
  <div
    className={`user-card ${isSelected ? 'selected' : ''}`}
    onClick={() => onSelect(user.id)}
  >
    <h3>{user.name}</h3>
    <p>{user.age}岁</p>
  </div>
))

// 虚拟滚动列表
const VirtualizedList = ({ items, itemHeight, containerHeight }) => {
  const [scrollTop, setScrollTop] = useState(0)
  
  const startIndex = Math.floor(scrollTop / itemHeight)
  const endIndex = Math.min(
    items.length - 1,
    startIndex + Math.ceil(containerHeight / itemHeight)
  )
  
  const visibleItems = items.slice(startIndex, endIndex + 1)
  
  return (
    <div
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={(e) => setScrollTop(e.target.scrollTop)}
    >
      <div style={{ height: items.length * itemHeight }}>
        {visibleItems.map(item => (
          <div
            key={item.id}
            style={{
              position: 'absolute',
              top: item.index * itemHeight,
              height: itemHeight
            }}
          >
            {item.content}
          </div>
        ))}
      </div>
    </div>
  )
}

// 使用
const App = () => {
  const largeList = Array.from({ length: 1000 }, (_, i) => ({
    id: i,
    content: `项目 ${i + 1}`
  }))
  
  return (
    <VirtualizedList
      items={largeList}
      itemHeight={50}
      containerHeight={400}
    />
  )
}
```

## 表单处理

### 1. 受控组件

```jsx
const ControlledForm = () => {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    rememberMe: false,
    gender: 'male',
    interests: []
  })
  
  const handleChange = (field, value) => {
    setFormData(prev => ({
      ...prev,
      [field]: value
    }))
  }
  
  const handleSubmit = (e) => {
    e.preventDefault()
    console.log('表单提交:', formData)
  }
  
  const handleCheckboxChange = (interest) => {
    setFormData(prev => ({
      ...prev,
      interests: prev.interests.includes(interest)
        ? prev.interests.filter(i => i !== interest)
        : [...prev.interests, interest]
    }))
  }
  
  return (
    <form onSubmit={handleSubmit}>
      {/* 文本输入 */}
      <input
        type="text"
        value={formData.username}
        onChange={(e) => handleChange('username', e.target.value)}
        placeholder="用户名"
      />
      
      {/* 邮箱输入 */}
      <input
        type="email"
        value={formData.email}
        onChange={(e) => handleChange('email', e.target.value)}
        placeholder="邮箱"
      />
      
      {/* 密码输入 */}
      <input
        type="password"
        value={formData.password}
        onChange={(e) => handleChange('password', e.target.value)}
        placeholder="密码"
      />
      
      {/* 复选框 */}
      <label>
        <input
          type="checkbox"
          checked={formData.rememberMe}
          onChange={(e) => handleChange('rememberMe', e.target.checked)}
        />
        记住我
      </label>
      
      {/* 单选按钮 */}
      <div>
        <label>
          <input
            type="radio"
            value="male"
            checked={formData.gender === 'male'}
            onChange={(e) => handleChange('gender', e.target.value)}
          />
          男
        </label>
        <label>
          <input
            type="radio"
            value="female"
            checked={formData.gender === 'female'}
            onChange={(e) => handleChange('gender', e.target.value)}
          />
          女
        </label>
      </div>
      
      {/* 多选框 */}
      <div>
        {['reading', 'sports', 'music'].map(interest => (
          <label key={interest}>
            <input
              type="checkbox"
              checked={formData.interests.includes(interest)}
              onChange={() => handleCheckboxChange(interest)}
            />
            {interest}
          </label>
        ))}
      </div>
      
      {/* 下拉选择 */}
      <select
        value={formData.gender}
        onChange={(e) => handleChange('gender', e.target.value)}
      >
        <option value="male">男</option>
        <option value="female">女</option>
        <option value="other">其他</option>
      </select>
      
      <button type="submit">提交</button>
    </form>
  )
}
```

### 2. 表单优化

```jsx
import { useReducer } from 'react'

// 使用 useReducer 管理复杂表单状态
const formReducer = (state, action) => {
  switch (action.type) {
    case 'UPDATE_FIELD':
      return {
        ...state,
        [action.field]: action.value
      }
    case 'RESET_FORM':
      return initialState
    default:
      return state
  }
}

const AdvancedForm = () => {
  const initialState = {
    username: '',
    email: '',
    password: '',
    rememberMe: false
  }
  
  const [formData, dispatch] = useReducer(formReducer, initialState)
  
  const updateField = (field, value) => {
    dispatch({ type: 'UPDATE_FIELD', field, value })
  }
  
  const resetForm = () => {
    dispatch({ type: 'RESET_FORM' })
  }
  
  const handleSubmit = (e) => {
    e.preventDefault()
    // 表单验证逻辑
    if (!formData.username || !formData.email) {
      alert('请填写必填字段')
      return
    }
    console.log('提交数据:', formData)
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.username}
        onChange={(e) => updateField('username', e.target.value)}
        placeholder="用户名 *"
      />
      <input
        type="email"
        value={formData.email}
        onChange={(e) => updateField('email', e.target.value)}
        placeholder="邮箱 *"
      />
      <input
        type="password"
        value={formData.password}
        onChange={(e) => updateField('password', e.target.value)}
        placeholder="密码"
      />
      <button type="submit">提交</button>
      <button type="button" onClick={resetForm}>重置</button>
    </form>
  )
}
```

## 组件通信

### 1. Props 通信

```jsx
// 父传子
const ParentComponent = () => {
  const [message, setMessage] = useState('Hello from parent')
  const [count, setCount] = useState(0)
  
  return (
    <div>
      <ChildComponent
        message={message}
        count={count}
        onCountChange={setCount}
      />
    </div>
  )
}

const ChildComponent = ({ message, count, onCountChange }) => (
  <div>
    <p>{message}</p>
    <p>计数: {count}</p>
    <button onClick={() => onCountChange(count + 1)}>
      增加
    </button>
  </div>
)

// 子传父（回调函数）
const TodoApp = () => {
  const [todos, setTodos] = useState([])
  
  const addTodo = (text) => {
    setTodos([...todos, { id: Date.now(), text }])
  }
  
  return (
    <div>
      <TodoInput onAddTodo={addTodo} />
      <TodoList todos={todos} />
    </div>
  )
}

const TodoInput = ({ onAddTodo }) => {
  const [input, setInput] = useState('')
  
  const handleSubmit = () => {
    if (input.trim()) {
      onAddTodo(input)
      setInput('')
    }
  }
  
  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleSubmit}>添加</button>
    </div>
  )
}
```

### 2. 组合模式

```jsx
// 使用 children
const Layout = ({ header, sidebar, content, footer }) => (
  <div className="layout">
    <header>{header}</header>
    <aside>{sidebar}</aside>
    <main>{content}</main>
    <footer>{footer}</footer>
  </div>
)

// 使用 props 组合
const Card = ({ title, actions, children }) => (
  <div className="card">
    {title && <div className="card-header">{title}</div>}
    <div className="card-body">{children}</div>
    {actions && <div className="card-actions">{actions}</div>}
  </div>
)

// 使用
const App = () => (
  <Layout
    header={<Header />}
    sidebar={<Sidebar />}
    content={<MainContent />}
    footer={<Footer />}
  />
)

const UserCard = () => (
  <Card
    title="用户信息"
    actions={
      <button onClick={() => console.log('操作')}>
        编辑
      </button>
    }
  >
    <p>用户名: John Doe</p>
    <p>邮箱: john@example.com</p>
  </Card>
)
```
