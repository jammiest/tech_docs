# Next.js 指南

Next.js 是一个功能强大的 React 框架，提供了服务端渲染、静态站点生成、API 路由等开箱即用的功能。本指南将全面介绍 Next.js 的核心概念和最佳实践。

## 核心概念

### 1. 项目结构

```
my-next-app/
├── pages/                 # 页面目录（App Router 中为 app/）
│   ├── api/              # API 路由
│   │   └── hello.ts
│   ├── _app.tsx          # 自定义 App 组件
│   ├── _document.tsx     # 自定义 Document 组件
│   ├── index.tsx         # 首页 (/)
│   └── about.tsx         # 关于页面 (/about)
├── public/               # 静态资源
│   └── favicon.ico
├── styles/               # 样式文件
│   └── globals.css
├── components/           # 可复用组件
├── lib/                  # 工具库
├── types/                # TypeScript 类型定义
└── next.config.js        # Next.js 配置
```

### 2. 页面和路由

```tsx
// pages/index.tsx (Pages Router)
const HomePage = () => {
  return (
    <div>
      <h1>首页</h1>
      <Link href="/about">
        <a>关于我们</a>
      </Link>
    </div>
  )
}

export default HomePage

// pages/about.tsx
const AboutPage = () => {
  return (
    <div>
      <h1>关于我们</h1>
      <Link href="/">
        <a>返回首页</a>
      </Link>
    </div>
  )
}

export default AboutPage

// 动态路由: pages/users/[id].tsx
const UserPage = () => {
  const router = useRouter()
  const { id } = router.query
  
  return (
    <div>
      <h1>用户详情: {id}</h1>
    </div>
  )
}

export default UserPage
```

## 渲染策略

### 1. 服务端渲染 (SSR)

```tsx
// pages/ssr-page.tsx
import { GetServerSideProps } from 'next'

interface Props {
  data: {
    title: string
    content: string
  }
}

const SSRPage: React.FC<Props> = ({ data }) => {
  return (
    <div>
      <h1>{data.title}</h1>
      <p>{data.content}</p>
    </div>
  )
}

export const getServerSideProps: GetServerSideProps = async (context) => {
  // 每次请求时都会执行
  const res = await fetch('https://api.example.com/data')
  const data = await res.json()

  return {
    props: {
      data
    }
  }
}

export default SSRPage
```

### 2. 静态站点生成 (SSG)

```tsx
// pages/ssg-page.tsx
import { GetStaticProps } from 'next'

interface Props {
  posts: Array<{
    id: number
    title: string
  }>
}

const SSGPage: React.FC<Props> = ({ posts }) => {
  return (
    <div>
      <h1>博客文章</h1>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  )
}

export const getStaticProps: GetStaticProps = async () => {
  // 构建时执行
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

  return {
    props: {
      posts
    },
    revalidate: 60 // ISR: 60秒后重新生成
  }
}

export default SSGPage
```

### 3. 增量静态再生 (ISR)

```tsx
// pages/isr-page.tsx
import { GetStaticProps } from 'next'

const ISRPage = ({ data }) => {
  return (
    <div>
      <h1>ISR 页面</h1>
      <p>生成时间: {new Date().toLocaleString()}</p>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  )
}

export const getStaticProps: GetStaticProps = async () => {
  const res = await fetch('https://api.example.com/data')
  const data = await res.json()

  return {
    props: {
      data
    },
    revalidate: 30 // 30秒后可以重新生成
  }
}

export default ISRPage
```

## API 路由

### 1. 基础 API 路由

```tsx
// pages/api/users/index.ts
import { NextApiRequest, NextApiResponse } from 'next'

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { method } = req

  switch (method) {
    case 'GET':
      // 获取用户列表
      const users = await fetchUsers()
      res.status(200).json(users)
      break
    
    case 'POST':
      // 创建新用户
      const newUser = await createUser(req.body)
      res.status(201).json(newUser)
      break
    
    default:
      res.setHeader('Allow', ['GET', 'POST'])
      res.status(405).end(`Method ${method} Not Allowed`)
  }
}

// pages/api/users/[id].ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { id } = req.query
  const { method } = req

  switch (method) {
    case 'GET':
      const user = await getUserById(Number(id))
      if (!user) {
        return res.status(404).json({ error: 'User not found' })
      }
      res.status(200).json(user)
      break
    
    case 'PUT':
      const updatedUser = await updateUser(Number(id), req.body)
      res.status(200).json(updatedUser)
      break
    
    case 'DELETE':
      await deleteUser(Number(id))
      res.status(204).end()
      break
    
    default:
      res.setHeader('Allow', ['GET', 'PUT', 'DELETE'])
      res.status(405).end(`Method ${method} Not Allowed`)
  }
}
```

### 2. API 路由最佳实践

```tsx
// lib/api-response.ts
export class ApiResponse<T = any> {
  constructor(
    public readonly success: boolean,
    public readonly data?: T,
    public readonly error?: string,
    public readonly status: number = 200
  ) {}

  static success<T>(data: T, status: number = 200) {
    return new ApiResponse(true, data, undefined, status)
  }

  static error(message: string, status: number = 500) {
    return new ApiResponse(false, undefined, message, status)
  }
}

// 使用响应工具
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const data = await fetchData()
    const response = ApiResponse.success(data)
    res.status(response.status).json(response)
  } catch (error) {
    const response = ApiResponse.error('Failed to fetch data')
    res.status(response.status).json(response)
  }
}

// 中间件模式
// lib/api-middleware.ts
import { NextApiRequest, NextApiResponse } from 'next'

export type ApiHandler = (
  req: NextApiRequest,
  res: NextApiResponse
) => Promise<void>

export const withErrorHandler = (handler: ApiHandler): ApiHandler => {
  return async (req: NextApiRequest, res: NextApiResponse) => {
    try {
      await handler(req, res)
    } catch (error) {
      console.error('API Error:', error)
      res.status(500).json({ error: 'Internal server error' })
    }
  }
}

export const withAuth = (handler: ApiHandler): ApiHandler => {
  return async (req: NextApiRequest, res: NextApiResponse) => {
    const token = req.headers.authorization
    if (!token) {
      return res.status(401).json({ error: 'Unauthorized' })
    }
    await handler(req, res)
  }
}
```

## 样式和资源

### 1. CSS 和样式方案

```tsx
// 全局样式: styles/globals.css
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

// CSS Modules: components/Button.module.css
.button {
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  background-color: #0070f3;
  color: white;
  cursor: pointer;
}

.button:hover {
  background-color: #0051a3;
}

// 使用 CSS Modules
import styles from './Button.module.css'

const Button = ({ children }) => {
  return (
    <button className={styles.button}>
      {children}
    </button>
  )
}

// Styled JSX
const StyledComponent = () => {
  return (
    <div>
      <h1>Styled JSX 示例</h1>
      <style jsx>{`
        h1 {
          color: #0070f3;
          font-size: 2rem;
        }
        div {
          padding: 20px;
          background: #f0f0f0;
        }
      `}</style>
    </div>
  )
}
```

### 2. 静态资源优化

```tsx
// 图片优化
import Image from 'next/image'

const OptimizedImage = () => {
  return (
    <div>
      {/* 本地图片 */}
      <Image
        src="/images/hero.jpg"
        alt="Hero Image"
        width={800}
        height={400}
        priority // 预加载重要图片
      />
      
      {/* 远程图片 */}
      <Image
        src="https://example.com/image.jpg"
        alt="Remote Image"
        width={600}
        height={300}
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..."
      />
    </div>
  )
}

// 字体优化
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ['latin'] })

const FontExample = () => {
  return (
    <div className={inter.className}>
      <h1>优化后的字体</h1>
      <p>这段文字使用优化的字体加载</p>
    </div>
  )
}
```

## 数据获取

### 1. 客户端数据获取

```tsx
import { useState, useEffect } from 'react'
import useSWR from 'swr'

const ClientSideData = () => {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)

  // 使用 useEffect
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('/api/data')
        const result = await response.json()
        setData(result)
      } catch (error) {
        console.error('Failed to fetch data:', error)
      } finally {
        setLoading(false)
      }
    }

    fetchData()
  }, [])

  // 使用 SWR
  const { data: swrData, error, isLoading } = useSWR('/api/data', fetcher)

  if (loading || isLoading) return <div>加载中...</div>
  if (error) return <div>加载失败</div>

  return (
    <div>
      <h1>客户端数据</h1>
      <pre>{JSON.stringify(data || swrData, null, 2)}</pre>
    </div>
  )
}
```

### 2. 服务端数据获取

```tsx
import { GetServerSideProps, GetStaticProps } from 'next'

export const getServerSideProps: GetServerSideProps = async (context) => {
  // 在服务器端获取数据
  const res = await fetch('https://api.example.com/data')
  const data = await res.json()

  return {
    props: {
      data
    }
  }
}

export const getStaticProps: GetStaticProps = async () => {
  // 构建时获取数据
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

  return {
    props: {
      posts
    },
    revalidate: 60
  }
}
```

## 性能优化

### 1. 代码分割和懒加载

```tsx
import dynamic from 'next/dynamic'
import { Suspense } from 'react'

// 动态导入组件
const HeavyComponent = dynamic(() => import('../components/HeavyComponent'), {
  suspense: true,
  loading: () => <div>加载中...</div>
})

const LazyLoadedComponent = dynamic(
  () => import('../components/LazyLoadedComponent'),
  {
    ssr: false // 仅在客户端加载
  }
)

const OptimizedPage = () => {
  return (
    <div>
      <Suspense fallback={<div>加载组件中...</div>}>
        <HeavyComponent />
      </Suspense>
      
      <LazyLoadedComponent />
    </div>
  )
}

// 动态导入库
const importLibrary = async () => {
  const library = await import('some-heavy-library')
  library.doSomething()
}
```

### 2. 图片和资源优化

```tsx
// next.config.js 优化配置
module.exports = {
  images: {
    domains: ['example.com'], // 允许的图片域名
    formats: ['image/webp', 'image/avif'], // 现代格式支持
    deviceSizes: [640, 750, 828, 1080, 1200, 1920], // 设备尺寸
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384], // 图片尺寸
  },
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production', // 生产环境移除 console
  },
  compress: true, // 启用 Gzip 压缩
}
```

## 部署和配置

### 1. 环境配置

```js
// next.config.js
module.exports = {
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
    API_URL: process.env.API_URL,
  },
  async redirects() {
    return [
      {
        source: '/old-path',
        destination: '/new-path',
        permanent: true,
      },
    ]
  },
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-XSS-Protection',
            value: '1; mode=block',
          },
        ],
      },
    ]
  },
}
```

### 2. 部署配置

```yaml
# vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "functions": {
    "pages/api/*.js": {
      "maxDuration": 30
    }
  },
  "env": {
    "DATABASE_URL": "@database_url"
  }
}

# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

## 最佳实践

### 1. 项目结构组织

```
src/
├── components/           # 可复用组件
│   ├── ui/              # 基础UI组件
│   ├── layout/          # 布局组件
│   └── features/        # 功能组件
├── pages/               # 页面组件
│   ├── api/            # API路由
│   └── [dynamic]/      # 动态路由
├── lib/                 # 工具函数
│   ├── api/            # API客户端
│   ├── utils/          # 工具函数
│   └── constants.ts     # 常量定义
├── styles/              # 样式文件
│   ├── globals.css     # 全局样式
│   └── components/     # 组件样式
├── types/               # TypeScript类型
│   └── index.ts        # 类型导出
└── hooks/               # 自定义Hooks
```

### 2. SEO 和元数据

```tsx
import Head from 'next/head'

const SEOPage = () => {
  return (
    <>
      <Head>
        <title>页面标题</title>
        <meta name="description" content="页面描述" />
        <meta name="keywords" content="关键词1,关键词2" />
        <meta property="og:title" content="Open Graph 标题" />
        <meta property="og:description" content="Open Graph 描述" />
        <meta property="og:image" content="/og-image.jpg" />
        <meta name="twitter:card" content="summary_large_image" />
      </Head>
      
      <main>
        <h1>页面内容</h1>
      </main>
    </>
  )
}

// 使用 next-seo
import { NextSeo } from 'next-seo'

const SeoOptimizedPage = () => {
  return (
    <>
      <NextSeo
        title="优化标题"
        description="优化描述"
        openGraph={{
          title: 'OG标题',
          description: 'OG描述',
          images: [{ url: '/og-image.jpg' }],
        }}
      />
      
      <main>
        <h1>SEO优化页面</h1>
      </main>
    </>
  )
}
```

## 故障排除

### 1. 常见问题解决

```js
// 1. 环境变量问题
// 确保在 .env.local 中定义变量
// NEXT_PUBLIC_ 前缀的变量会在客户端暴露

// 2. 图片优化问题
// 在 next.config.js 中配置 images.domains

// 3. API路由问题
// 检查 API 路由文件是否在 pages/api 目录下

// 4. 构建错误
// 检查 getStaticProps/getServerSideProps 的返回格式

// 5. 性能问题
// 使用 next/image 优化图片
// 实现代码分割和懒加载
```
