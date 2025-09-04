# Vite 快速入门

Vite 是一个现代化的前端构建工具，提供极快的冷启动、热更新和构建性能。本指南将帮助您快速掌握 Vite 的核心概念和使用方法。

## 核心概念

### 1. 项目创建和初始化

```bash
# 使用 npm
npm create vite@latest my-vite-app -- --template react

# 使用 yarn
yarn create vite my-vite-app --template react

# 使用 pnpm
pnpm create vite my-vite-app --template react

# 选择模板
# vanilla, vanilla-ts, vue, vue-ts, react, react-ts, react-swc, react-swc-ts, preact, preact-ts, lit, lit-ts, svelte, svelte-ts
```

### 2. 基础项目结构

```
my-vite-app/
├── src/
│   ├── main.jsx          # 入口文件
│   ├── App.jsx           # 根组件
│   ├── index.css         # 全局样式
│   └── components/       # 组件目录
├── public/               # 静态资源
│   └── vite.svg
├── index.html            # HTML 模板
├── vite.config.js        # Vite 配置
├── package.json
└── tsconfig.json         # TypeScript 配置（如果使用 TS）
```

## 基础配置

### 1. 基本配置文件

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  // 插件配置
  plugins: [react()],
  
  // 开发服务器配置
  server: {
    port: 3000,
    open: true,
    host: true
  },
  
  // 构建配置
  build: {
    outDir: 'dist',
    sourcemap: true
  },
  
  // 解析配置
  resolve: {
    alias: {
      '@': '/src',
      'components': '/src/components'
    }
  }
})
```

### 2. 环境特定配置

```javascript
// vite.config.js
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  // 加载环境变量
  const env = loadEnv(mode, process.cwd())
  
  return {
    // 基础配置
    base: env.VITE_APP_BASE_URL || '/',
    
    // 开发服务器
    server: {
      proxy: {
        '/api': {
          target: env.VITE_API_URL || 'http://localhost:8000',
          changeOrigin: true
        }
      }
    },
    
    // 构建配置
    build: {
      minify: mode === 'production' ? 'esbuild' : false,
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ['react', 'react-dom'],
            utils: ['lodash', 'axios']
          }
        }
      }
    }
  }
})
```

## 开发服务器

### 1. 服务器配置

```javascript
// vite.config.js
export default defineConfig({
  server: {
    // 服务器配置
    port: 3000,
    host: '0.0.0.0',
    open: true,
    cors: true,
    
    // HTTPS 配置
    https: false,
    
    // 代理配置
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      },
      '/socket.io': {
        target: 'ws://localhost:3001',
        ws: true
      }
    },
    
    // HMR 配置
    hmr: {
      overlay: true
    }
  }
})
```

### 2. 热模块替换 (HMR)

```javascript
// 在代码中使用 HMR
if (import.meta.hot) {
  // 监听自定义事件
  import.meta.hot.on('my-event', (data) => {
    console.log('收到自定义事件:', data)
  })
  
  // 发送事件
  import.meta.hot.send('my-event', { message: 'Hello from client' })
  
  // 接受更新
  import.meta.hot.accept((newModule) => {
    console.log('模块已更新:', newModule)
  })
  
  // 处理错误
  import.meta.hot.dispose(() => {
    // 清理资源
  })
}
```

## 插件系统

### 1. 官方插件

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import vue from '@vitejs/plugin-vue'
import legacy from '@vitejs/plugin-legacy'

export default defineConfig({
  plugins: [
    // React 插件
    react({
      jsxImportSource: '@emotion/react',
      babel: {
        plugins: ['@emotion/babel-plugin']
      }
    }),
    
    // Vue 插件
    vue({
      template: {
        compilerOptions: {
          // Vue 模板编译选项
        }
      }
    }),
    
    // 传统浏览器支持
    legacy({
      targets: ['defaults', 'not IE 11']
    })
  ]
})
```

### 2. 社区插件

```javascript
import { defineConfig } from 'vite'
import viteImagemin from 'vite-plugin-imagemin'
import { VitePWA } from 'vite-plugin-pwa'
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    // 图片压缩
    viteImagemin({
      gifsicle: { optimizationLevel: 3 },
      mozjpeg: { quality: 75 },
      pngquant: { quality: [0.8, 0.9] },
      svgo: {
        plugins: [
          { name: 'removeViewBox' },
          { name: 'removeEmptyAttrs', active: false }
        ]
      }
    }),
    
    // PWA 支持
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg}']
      },
      manifest: {
        name: 'My Vite App',
        short_name: 'ViteApp',
        theme_color: '#ffffff'
      }
    }),
    
    // 包分析
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true
    })
  ]
})
```

## 构建优化

### 1. 构建配置

```javascript
export default defineConfig({
  build: {
    // 输出目录
    outDir: 'dist',
    
    // 源码映射
    sourcemap: true,
    
    // 资源目录
    assetsDir: 'assets',
    
    // 资源大小限制
    assetsInlineLimit: 4096,
    
    // CSS 配置
    cssCodeSplit: true,
    cssTarget: 'es6',
    
    // Rollup 配置
    rollupOptions: {
      output: {
        // 代码分割
        manualChunks: (id) => {
          if (id.includes('node_modules')) {
            if (id.includes('react')) return 'vendor-react'
            if (id.includes('lodash')) return 'vendor-utils'
            return 'vendor'
          }
        },
        
        // 文件命名
        chunkFileNames: 'js/[name]-[hash].js',
        entryFileNames: 'js/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]'
      }
    },
    
    // 最小化配置
    minify: 'esbuild',
    
    // 目标环境
    target: 'es2015'
  }
})
```

### 2. 性能优化

```javascript
export default defineConfig({
  build: {
    // 打包分析
    rollupOptions: {
      output: {
        manualChunks: {
          react: ['react', 'react-dom'],
          router: ['react-router-dom'],
          state: ['redux', '@reduxjs/toolkit'],
          utils: ['lodash', 'axios', 'dayjs']
        }
      }
    },
    
    // 压缩选项
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true
      }
    }
  },
  
  // 依赖优化
  optimizeDeps: {
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'lodash',
      'axios'
    ],
    exclude: ['vue'] // 排除不需要优化的依赖
  }
})
```

## 样式处理

### 1. CSS 配置

```javascript
export default defineConfig({
  css: {
    // CSS 模块配置
    modules: {
      scopeBehaviour: 'local',
      localsConvention: 'camelCase',
      generateScopedName: '[name]__[local]--[hash:base64:5]'
    },
    
    // 预处理器配置
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      },
      less: {
        math: 'always',
        globalVars: {
          primaryColor: '#1890ff'
        }
      }
    },
    
    // PostCSS 配置
    postcss: {
      plugins: [
        require('autoprefixer'),
        require('postcss-preset-env')
      ]
    }
  }
})
```

### 2. 样式文件示例

```css
/* src/styles/variables.scss */
$primary-color: #1890ff;
$border-radius: 4px;

:export {
  primaryColor: $primary-color;
  borderRadius: $border-radius;
}

/* src/styles/globals.css */
@import './variables.scss';

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
```

## TypeScript 支持

### 1. TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "components/*": ["src/components/*"]
    }
  },
  "include": ["src", "vite.config.ts"],
  "exclude": ["node_modules", "dist"]
}
```

### 2. Vite 环境类型

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string
  readonly VITE_API_URL: string
  readonly VITE_APP_VERSION: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

## 环境变量

### 1. 环境变量配置

```bash
# .env
VITE_APP_TITLE=My Vite App
VITE_API_URL=http://localhost:3000/api

# .env.development
VITE_API_URL=http://localhost:3000/api

# .env.production
VITE_API_URL=https://api.example.com
VITE_APP_VERSION=1.0.0
```

### 2. 使用环境变量

```javascript
// 在代码中使用环境变量
const apiUrl = import.meta.env.VITE_API_URL
const appTitle = import.meta.env.VITE_APP_TITLE

// 配置智能提示
export const env = {
  API_URL: import.meta.env.VITE_API_URL,
  APP_TITLE: import.meta.env.VITE_APP_TITLE,
  MODE: import.meta.env.MODE,
  DEV: import.meta.env.DEV,
  PROD: import.meta.env.PROD
}
```

## 部署配置

### 1. 多环境部署

```javascript
// vite.config.js
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())
  
  return {
    base: env.VITE_BASE_PATH || '/',
    
    build: {
      outDir: 'dist',
      assetsDir: 'static',
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ['react', 'react-dom'],
            utils: ['lodash', 'axios']
          }
        }
      }
    },
    
    server: {
      proxy: {
        '/api': {
          target: env.VITE_PROXY_TARGET,
          changeOrigin: true
        }
      }
    }
  }
})
```

### 2. Docker 部署

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
    
    location /api {
        proxy_pass http://backend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 高级功能

### 1. 模块联邦

```javascript
// vite.config.js (远程应用)
import { defineConfig } from 'vite'
import federation from '@originjs/vite-plugin-federation'

export default defineConfig({
  plugins: [
    federation({
      name: 'remote-app',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/components/Button',
        './Header': './src/components/Header'
      },
      shared: ['react', 'react-dom']
    })
  ],
  
  build: {
    target: 'esnext'
  }
})

// vite.config.js (主机应用)
import { defineConfig } from 'vite'
import federation from '@originjs/vite-plugin-federation'

export default defineConfig({
  plugins: [
    federation({
      name: 'host-app',
      remotes: {
        remote: 'http://localhost:5001/assets/remoteEntry.js'
      },
      shared: ['react', 'react-dom']
    })
  ]
})
```

### 2. Web Workers

```javascript
// 创建 Worker
// src/workers/example.worker.js
self.onmessage = (event) => {
  const result = event.data * 2
  self.postMessage(result)
}

// 使用 Worker
const worker = new Worker(new URL('./workers/example.worker.js', import.meta.url))

worker.onmessage = (event) => {
  console.log('Worker 结果:', event.data)
}

worker.postMessage(42)
```

## 测试配置

### 1. 测试环境配置

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.js',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html']
    }
  }
})
```

### 2. 测试示例

```javascript
// src/test/setup.js
import { expect, afterEach } from 'vitest'
import { cleanup } from '@testing-library/react'
import * as matchers from '@testing-library/jest-dom/matchers'

expect.extend(matchers)

afterEach(() => {
  cleanup()
})

// src/components/Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react'
import Button from './Button'

describe('Button', () => {
  it('渲染按钮文本', () => {
    render(<Button>点击我</Button>)
    expect(screen.getByText('点击我')).toBeInTheDocument()
  })
  
  it('处理点击事件', () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>点击我</Button>)
    
    fireEvent.click(screen.getByText('点击我'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

## 常见问题解决

### 1. 路径别名问题

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import path from 'path'

export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      'components': path.resolve(__dirname, 'src/components'),
      'utils': path.resolve(__dirname, 'src/utils')
    }
  }
})

// tsconfig.json (TypeScript 项目)
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "components/*": ["src/components/*"]
    }
  }
}
```

### 2. 依赖优化问题

```javascript
export default defineConfig({
  optimizeDeps: {
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'lodash',
      'axios'
    ],
    exclude: [
      'vue' // 排除不需要预构建的依赖
    ]
  }
})
```

### 3. 构建性能问题

```javascript
export default defineConfig({
  build: {
    // 禁用某些优化
    minify: false,
    
    // 调整块大小警告
    chunkSizeWarningLimit: 1000,
    
    // 详细输出
    rollupOptions: {
      output: {
        manualChunks: {
          // 自定义代码分割
        }
      }
    }
  }
})
```

## 最佳实践

### 1. 项目结构优化

```
src/
├── components/           # 可复用组件
│   ├── ui/              # 基础UI组件
│   ├── forms/           # 表单组件
│   └── layout/          # 布局组件
├── pages/               # 页面组件
├── hooks/               # 自定义Hooks
├── utils/               # 工具函数
├── services/            # API服务
├── stores/              # 状态管理
├── styles/              # 样式文件
│   ├── variables.css    # CSS变量
│   ├── globals.css      # 全局样式
│   └── components/      # 组件样式
├── types/               # TypeScript类型
└── assets/              # 静态资源
```

### 2. 配置维护建议

```javascript
// config/vite.config.base.js - 基础配置
export const baseConfig = {
  plugins: [],
  resolve: {
    alias: {
      '@': '/src'
    }
  }
}

// config/vite.config.dev.js - 开发配置
export const devConfig = {
  server: {
    port: 3000,
    open: true
  }
}

// config/vite.config.prod.js - 生产配置
export const prodConfig = {
  build: {
    minify: 'esbuild',
    sourcemap: false
  }
}

// vite.config.js - 主配置文件
import { defineConfig, mergeConfig } from 'vite'
import { baseConfig } from './config/vite.config.base'
import { devConfig } from './config/vite.config.dev'
import { prodConfig } from './config/vite.config.prod'

export default defineConfig(({ mode }) => {
  let envConfig = {}
  
  if (mode === 'development') {
    envConfig = devConfig
  } else {
    envConfig = prodConfig
  }
  
  return mergeConfig(baseConfig, envConfig)
})
```
