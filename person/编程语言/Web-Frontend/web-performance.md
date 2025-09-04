# Web 性能优化

Web 性能优化是提升用户体验和业务指标的关键技术。本指南将全面介绍从加载性能到运行时性能的完整优化策略。

## 性能指标体系

### 1. Core Web Vitals 监控

```javascript
// src/utils/webVitals.js
import { getCLS, getFID, getLCP, getFCP, getTTFB } from 'web-vitals';

class WebVitalsMonitor {
  constructor() {
    this.metrics = new Map();
    this.reportQueue = [];
    this.isReporting = false;
  }

  // 初始化监控
  init() {
    this.setupCoreWebVitals();
    this.setupCustomMetrics();
    this.setupPerformanceObserver();
  }

  // Core Web Vitals 监控
  setupCoreWebVitals() {
    getCLS(this.handleMetric.bind(this, 'CLS'));
    getFID(this.handleMetric.bind(this, 'FID'));
    getLCP(this.handleMetric.bind(this, 'LCP'));
    getFCP(this.handleMetric.bind(this, 'FCP'));
    getTTFB(this.handleMetric.bind(this, 'TTFB'));
  }

  // 自定义性能指标
  setupCustomMetrics() {
    // 最大内容绘制时间
    this.measureLCP();
    
    // 首次输入延迟
    this.measureFID();
    
    // 布局偏移
    this.measureCLS();
  }

  // Performance Observer 监控
  setupPerformanceObserver() {
    if ('PerformanceObserver' in window) {
      // 资源加载监控
      const resourceObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach(entry => {
          this.handleMetric('resource', entry);
        });
      });
      resourceObserver.observe({ entryTypes: ['resource'] });

      // 长任务监控
      const longTaskObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach(entry => {
          if (entry.duration > 50) {
            this.handleMetric('longtask', entry);
          }
        });
      });
      longTaskObserver.observe({ entryTypes: ['longtask'] });

      // 布局偏移监控
      const layoutShiftObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach(entry => {
          if (!entry.hadRecentInput) {
            this.handleMetric('layout-shift', entry);
          }
        });
      });
      layoutShiftObserver.observe({ entryTypes: ['layout-shift'] });
    }
  }

  // 处理性能指标
  handleMetric(name, metric) {
    const data = {
      name,
      value: metric.value || metric.duration,
      timestamp: Date.now(),
      ...this.getContext(metric)
    };

    this.metrics.set(name, data);
    this.reportToAnalytics(data);
  }

  // 获取上下文信息
  getContext(metric) {
    return {
      url: window.location.href,
      userAgent: navigator.userAgent,
      connection: navigator.connection ? {
        effectiveType: navigator.connection.effectiveType,
        downlink: navigator.connection.downlink,
        rtt: navigator.connection.rtt
      } : null,
      ...(metric.entryType && { entryType: metric.entryType })
    };
  }

  // 上报到分析平台
  async reportToAnalytics(data) {
    this.reportQueue.push(data);
    
    if (!this.isReporting) {
      this.isReporting = true;
      
      while (this.reportQueue.length > 0) {
        const metric = this.reportQueue.shift();
        
        try {
          // 使用 sendBeacon 或 fetch 上报
          await this.sendMetric(metric);
        } catch (error) {
          console.warn('Failed to report metric:', error);
          // 重试逻辑
          this.reportQueue.unshift(metric);
          break;
        }
      }
      
      this.isReporting = false;
    }
  }

  // 发送指标数据
  async sendMetric(metric) {
    const url = '/api/analytics/web-vitals';
    const blob = new Blob([JSON.stringify(metric)], {
      type: 'application/json; charset=UTF-8'
    });

    if (navigator.sendBeacon) {
      return navigator.sendBeacon(url, blob);
    } else {
      return fetch(url, {
        method: 'POST',
        body: blob,
        keepalive: true
      });
    }
  }

  // 获取性能报告
  getReport() {
    return Array.from(this.metrics.entries()).reduce((report, [name, data]) => {
      report[name] = data;
      return report;
    }, {});
  }
}

// 全局性能监控实例
export const webVitals = new WebVitalsMonitor();

// React Hook 封装
export const useWebVitals = () => {
  const [report, setReport] = useState({});

  useEffect(() => {
    webVitals.init();
    
    // 定期更新报告
    const interval = setInterval(() => {
      setReport(webVitals.getReport());
    }, 5000);

    return () => clearInterval(interval);
  }, []);

  return report;
};
```

### 2. 性能评分系统

```javascript
// src/utils/performanceScoring.js
export class PerformanceScorer {
  static scoreWebVitals(metrics) {
    const scores = {
      LCP: this.scoreLCP(metrics.LCP),
      FID: this.scoreFID(metrics.FID),
      CLS: this.scoreCLS(metrics.CLS),
      FCP: this.scoreFCP(metrics.FCP),
      TTFB: this.scoreTTFB(metrics.TTFB)
    };

    const totalScore = Object.values(scores).reduce((sum, score) => sum + score, 0) / Object.keys(scores).length;

    return {
      scores,
      totalScore: Math.round(totalScore),
      grade: this.getGrade(totalScore)
    };
  }

  static scoreLCP(lcp) {
    if (lcp <= 2500) return 100;
    if (lcp <= 4000) return 80 - ((lcp - 2500) / 1500) * 20;
    return 0;
  }

  static scoreFID(fid) {
    if (fid <= 100) return 100;
    if (fid <= 300) return 80 - ((fid - 100) / 200) * 20;
    return 0;
  }

  static scoreCLS(cls) {
    if (cls <= 0.1) return 100;
    if (cls <= 0.25) return 80 - ((cls - 0.1) / 0.15) * 20;
    return 0;
  }

  static scoreFCP(fcp) {
    if (fcp <= 1800) return 100;
    if (fcp <= 3000) return 80 - ((fcp - 1800) / 1200) * 20;
    return 0;
  }

  static scoreTTFB(ttfb) {
    if (ttfb <= 800) return 100;
    if (ttfb <= 1800) return 80 - ((ttfb - 800) / 1000) * 20;
    return 0;
  }

  static getGrade(score) {
    if (score >= 90) return 'A';
    if (score >= 75) return 'B';
    if (score >= 50) return 'C';
    return 'D';
  }

  static generateRecommendations(metrics) {
    const recommendations = [];

    if (metrics.LCP > 2500) {
      recommendations.push({
        metric: 'LCP',
        severity: 'high',
        message: '最大内容绘制时间过长，优化图片加载和渲染性能',
        suggestions: [
          '使用下一代图片格式（WebP/AVIF）',
          '实现图片懒加载',
          '优化关键CSS加载',
          '使用CDN加速资源加载'
        ]
      });
    }

    if (metrics.FID > 100) {
      recommendations.push({
        metric: 'FID',
        severity: 'medium',
        message: '首次输入延迟较高，优化JavaScript执行',
        suggestions: [
          '减少主线程工作',
          '代码分割和懒加载',
          '优化事件处理程序',
          '使用Web Workers'
        ]
      });
    }

    if (metrics.CLS > 0.1) {
      recommendations.push({
        metric: 'CLS',
        severity: 'high',
        message: '累积布局偏移严重，影响用户体验',
        suggestions: [
          '为图片和媒体指定尺寸',
          '预留广告位空间',
          '避免动态注入内容',
          '使用transform动画替代布局变化'
        ]
      });
    }

    return recommendations;
  }
}
```

## 加载性能优化

### 1. 资源加载策略

```javascript
// src/utils/loadingOptimizer.js
class LoadingOptimizer {
  constructor() {
    this.preloadedResources = new Set();
    this.lazyLoadObserver = null;
  }

  // 关键资源预加载
  preloadCriticalResources() {
    const criticalResources = this.identifyCriticalResources();
    
    criticalResources.forEach(resource => {
      if (!this.preloadedResources.has(resource.url)) {
        this.createPreloadLink(resource);
        this.preloadedResources.add(resource.url);
      }
    });
  }

  // 识别关键资源
  identifyCriticalResources() {
    const resources = [];
    
    // 关键CSS
    const criticalCSS = document.querySelector('link[rel="stylesheet"][data-critical]');
    if (criticalCSS) {
      resources.push({
        url: criticalCSS.href,
        as: 'style',
        type: 'text/css'
      });
    }

    // 关键JavaScript
    const criticalJS = document.querySelector('script[data-critical]');
    if (criticalJS && criticalJS.src) {
      resources.push({
        url: criticalJS.src,
        as: 'script',
        type: 'application/javascript'
      });
    }

    // 关键字体
    const criticalFonts = document.querySelectorAll('link[rel="stylesheet"][data-font]');
    criticalFonts.forEach(link => {
      resources.push({
        url: link.href,
        as: 'font',
        type: 'font/woff2',
        crossorigin: true
      });
    });

    return resources;
  }

  // 创建预加载链接
  createPreloadLink(resource) {
    const link = document.createElement('link');
    link.rel = 'preload';
    link.as = resource.as;
    link.href = resource.url;
    
    if (resource.type) link.type = resource.type;
    if (resource.crossorigin) link.crossOrigin = 'anonymous';
    
    document.head.appendChild(link);
  }

  // 图片懒加载
  setupLazyLoading() {
    if ('IntersectionObserver' in window) {
      this.lazyLoadObserver = new IntersectionObserver((entries, observer) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            const img = entry.target;
            this.loadLazyImage(img);
            observer.unobserve(img);
          }
        });
      }, {
        rootMargin: '200px 0px',
        threshold: 0.1
      });

      // 观察所有懒加载图片
      document.querySelectorAll('img[data-src], img[data-srcset]').forEach(img => {
        this.lazyLoadObserver.observe(img);
      });
    } else {
      // 回退方案：直接加载所有图片
      this.loadAllLazyImages();
    }
  }

  // 加载懒加载图片
  loadLazyImage(img) {
    if (img.dataset.src) {
      img.src = img.dataset.src;
      delete img.dataset.src;
    }
    
    if (img.dataset.srcset) {
      img.srcset = img.dataset.srcset;
      delete img.dataset.srcset;
    }
    
    img.classList.add('lazy-loaded');
  }

  // 资源优先级管理
  setResourcePriority() {
    // 关键CSS最高优先级
    const criticalCSS = document.querySelector('style[data-critical], link[rel="stylesheet"][data-critical]');
    if (criticalCSS) {
      criticalCSS.setAttribute('fetchpriority', 'high');
    }

    // 首屏图片高优先级
    const aboveFoldImages = this.getAboveFoldImages();
    aboveFoldImages.forEach(img => {
      img.setAttribute('fetchpriority', 'high');
    });

    // 非关键资源低优先级
    const nonCriticalResources = document.querySelectorAll(
      'script[data-non-critical], link[rel="stylesheet"][data-non-critical]'
    );
    nonCriticalResources.forEach(resource => {
      resource.setAttribute('fetchpriority', 'low');
    });
  }

  // 获取首屏图片
  getAboveFoldImages() {
    const viewportHeight = window.innerHeight;
    const images = Array.from(document.images);
    
    return images.filter(img => {
      const rect = img.getBoundingClientRect();
      return rect.top < viewportHeight;
    });
  }

  // 连接预连接
  preconnectToOrigins() {
    const origins = new Set();
    
    // 收集需要预连接的源
    document.querySelectorAll('link[rel="dns-prefetch"], link[rel="preconnect"]').forEach(link => {
      const url = new URL(link.href);
      origins.add(url.origin);
    });

    // 创建预连接
    origins.forEach(origin => {
      const link = document.createElement('link');
      link.rel = 'preconnect';
      link.href = origin;
      link.crossOrigin = 'anonymous';
      document.head.appendChild(link);
    });
  }
}

export const loadingOptimizer = new LoadingOptimizer();

// 在应用启动时调用
export const optimizeLoading = () => {
  loadingOptimizer.preloadCriticalResources();
  loadingOptimizer.setupLazyLoading();
  loadingOptimizer.setResourcePriority();
  loadingOptimizer.preconnectToOrigins();
};
```

### 2. 代码分割与懒加载

```javascript
// src/utils/codeSplitting.js
import React from 'react';

export class CodeSplittingManager {
  // 动态导入组件
  static lazyImport(importFn, fallback = null) {
    const LazyComponent = React.lazy(importFn);
    
    const SuspenseWrapper = (props) => (
      <React.Suspense fallback={fallback || <div>加载中...</div>}>
        <LazyComponent {...props} />
      </React.Suspense>
    );
    
    return SuspenseWrapper;
  }

  // 路由级代码分割
  static createLazyRoute(importFn, options = {}) {
    const { preload = false, timeout = 5000 } = options;
    let preloadedComponent = null;

    const importComponent = () => {
      if (preloadedComponent) {
        return preloadedComponent;
      }
      return importFn().then(module => module.default);
    };

    // 预加载支持
    if (preload) {
      importFn().then(module => {
        preloadedComponent = Promise.resolve(module.default);
      });
    }

    return this.lazyImport(importComponent);
  }

  // 基于条件的懒加载
  static conditionalLazy(importFn, condition, fallback = null) {
    return function ConditionalComponent(props) {
      const [shouldLoad, setShouldLoad] = useState(false);
      const LazyComponent = this.lazyImport(importFn, fallback);

      useEffect(() => {
        if (condition(props) && !shouldLoad) {
          setShouldLoad(true);
        }
      }, [props, condition, shouldLoad]);

      return shouldLoad ? <LazyComponent {...props} /> : fallback;
    };
  }

  // 预加载策略
  static preloadOnInteraction(element, importFn) {
    const preload = () => {
      importFn();
      element.removeEventListener('mouseenter', preload);
      element.removeEventListener('touchstart', preload);
    };

    element.addEventListener('mouseenter', preload, { once: true });
    element.addEventListener('touchstart', preload, { once: true });
  }
}

// Webpack 魔法注释优化
export const MAGIC_COMMENTS = {
  WEBPACK_PREFETCH: (priority = 'low') => 
    `/* webpackPrefetch: true, webpackChunkName: "[request]", webpackFetchPriority: "${priority}" */`,
  
  WEBPACK_PRELOAD: (priority = 'auto') =>
    `/* webpackPreload: true, webpackChunkName: "[request]", webpackFetchPriority: "${priority}" */`,
  
  WEBPACK_IGNORE: () =>
    `/* webpackIgnore: true */`
};

// 使用示例
export const LazyHomePage = CodeSplittingManager.createLazyRoute(
  () => import(/* webpackChunkName: "home" */ './pages/HomePage'),
  { preload: true }
);

export const LazyDashboard = CodeSplittingManager.createLazyRoute(
  () => import(/* webpackChunkName: "dashboard" */ './pages/Dashboard'),
  { preload: false }
);
```

## 渲染性能优化

### 1. 渲染优化策略

```javascript
// src/utils/renderingOptimizer.js
class RenderingOptimizer {
  constructor() {
    this.animationFrameQueue = [];
    this.isProcessingQueue = false;
  }

  // 使用 requestAnimationFrame 批量更新
  scheduleUpdate(callback, priority = 'normal') {
    const task = { callback, priority };
    this.animationFrameQueue.push(task);
    
    // 按优先级排序
    this.animationFrameQueue.sort((a, b) => {
      const priorityOrder = { high: 0, normal: 1, low: 2 };
      return priorityOrder[a.priority] - priorityOrder[b.priority];
    });

    if (!this.isProcessingQueue) {
      this.isProcessingQueue = true;
      requestAnimationFrame(this.processQueue.bind(this));
    }
  }

  // 处理任务队列
  processQueue() {
    const startTime = performance.now();
    
    while (this.animationFrameQueue.length > 0) {
      const task = this.animationFrameQueue.shift();
      
      try {
        task.callback();
      } catch (error) {
        console.error('Error in scheduled task:', error);
      }
      
      // 避免阻塞主线程
      if (performance.now() - startTime > 4) {
        break;
      }
    }

    if (this.animationFrameQueue.length > 0) {
      requestAnimationFrame(this.processQueue.bind(this));
    } else {
      this.isProcessingQueue = false;
    }
  }

  // 防抖函数
  debounce(func, wait, immediate = false) {
    let timeout;
    
    return function executedFunction(...args) {
      const later = () => {
        timeout = null;
        if (!immediate) func.apply(this, args);
      };
      
      const callNow = immediate && !timeout;
      clearTimeout(timeout);
      timeout = setTimeout(later, wait);
      
      if (callNow) func.apply(this, args);
    };
  }

  // 节流函数
  throttle(func, limit) {
    let inThrottle;
    
    return function(...args) {
      if (!inThrottle) {
        func.apply(this, args);
        inThrottle = true;
        setTimeout(() => inThrottle = false, limit);
      }
    };
  }

  // 批量 DOM 操作
  batchDOMUpdates(updates) {
    const fragment = document.createDocumentFragment();
    
    updates.forEach(update => {
      const element = document.createElement(update.tag);
      Object.assign(element, update.properties);
      fragment.appendChild(element);
    });
    
    this.scheduleUpdate(() => {
      document.getElementById(update.container).appendChild(fragment);
    }, 'high');
  }

  // 优化样式计算
  optimizeStyleRecalculations() {
    // 使用 will-change 提示浏览器
    const elements = document.querySelectorAll('[data-animate]');
    elements.forEach(el => {
      el.style.willChange = 'transform, opacity';
    });
  }

  // 避免布局抖动
  preventLayoutThrashing() {
    let scheduled = false;
    const reads = [];
    const writes = [];

    function schedule() {
      if (!scheduled) {
        scheduled = true;
        requestAnimationFrame(() => {
          scheduled = false;
          
          // 先执行所有读取操作
          reads.forEach(fn => fn());
          
          // 再执行所有写入操作
          writes.forEach(fn => fn());
          
          reads.length = 0;
          writes.length = 0;
        });
      }
    }

    return {
      read(fn) {
        reads.push(fn);
        schedule();
      },
      write(fn) {
        writes.push(fn);
        schedule();
      }
    };
  }
}

export const renderingOptimizer = new RenderingOptimizer();

// React 组件优化
export const withRenderingOptimization = (Component) => {
  return class OptimizedComponent extends React.Component {
    shouldComponentUpdate(nextProps, nextState) {
      // 浅比较优化
      return !shallowEqual(this.props, nextProps) || !shallowEqual(this.state, nextState);
    }

    componentDidUpdate() {
      // 在更新后调度非关键任务
      renderingOptimizer.scheduleUpdate(() => {
        this.performNonCriticalWork();
      }, 'low');
    }

    render() {
      return <Component {...this.props} />;
    }
  };
};
```

### 2. 内存管理

```javascript
// src/utils/memoryManager.js
class MemoryManager {
  constructor() {
    this.observers = new Map();
    this.cleanupCallbacks = new Set();
  }

  // 监控内存使用
  monitorMemory() {
    if ('memory' in performance) {
      setInterval(() => {
        const memory = performance.memory;
        this.checkMemoryUsage(memory);
      }, 30000);
    }
  }

  // 检查内存使用
  checkMemoryUsage(memory) {
    const usedJSHeapSize = memory.usedJSHeapSize;
    const totalJSHeapSize = memory.totalJSHeapSize;
    const usagePercentage = (usedJSHeapSize / totalJSHeapSize) * 100;

    if (usagePercentage > 80) {
      this.triggerCleanup('high_memory_usage', { usagePercentage });
    }

    // 报告内存指标
    this.reportMemoryMetrics({
      usedJSHeapSize,
      totalJSHeapSize,
      usagePercentage
    });
  }

  // 触发清理
  triggerCleanup(reason, data) {
    this.cleanupCallbacks.forEach(callback => {
      try {
        callback(reason, data);
      } catch (error) {
        console.error('Cleanup callback error:', error);
      }
    });
  }

  // 注册清理回调
  registerCleanup(callback) {
    this.cleanupCallbacks.add(callback);
    
    return () => {
      this.cleanupCallbacks.delete(callback);
    };
  }

  // 对象池管理
  createObjectPool(createFn, resetFn, maxSize = 100) {
    const pool = [];
    const used = new WeakSet();

    return {
      acquire() {
        if (pool.length > 0) {
          const obj = pool.pop();
          resetFn(obj);
          used.add(obj);
          return obj;
        }
        
        const obj = createFn();
        used.add(obj);
        return obj;
      },

      release(obj) {
        if (used.has(obj)) {
          used.delete(obj);
          if (pool.length < maxSize) {
            pool.push(obj);
          }
        }
      },

      get size() {
        return pool.length;
      }
    };
  }

  // 缓存管理
  createCache(maxSize = 50, ttl = 300000) {
    const cache = new Map();
    const timers = new Map();

    return {
      set(key, value) {
        if (cache.size >= maxSize) {
          const oldestKey = cache.keys().next().value;
          this.delete(oldestKey);
        }

        cache.set(key, value);
        
        // 设置过期时间
        if (ttl > 0) {
          timers.set(key, setTimeout(() => {
            this.delete(key);
          }, ttl));
        }
      },

      get(key) {
        return cache.get(key);
      },

      delete(key) {
        cache.delete(key);
        const timer = timers.get(key);
        if (timer) {
          clearTimeout(timer);
          timers.delete(key);
        }
      },

      clear() {
        cache.clear();
        timers.forEach(timer => clearTimeout(timer));
        timers.clear();
      }
    };
  }
}

export const memoryManager = new MemoryManager();

// React Hook 内存管理
export const useMemoryManagement = () => {
  const cleanupRef = useRef(null);

  useEffect(() => {
    const cleanup = memoryManager.registerCleanup((reason, data) => {
      // 执行组件特定的清理
      console.log('Memory cleanup triggered:', reason, data);
    });

    cleanupRef.current = cleanup;

    return () => {
      if (cleanupRef.current) {
        cleanupRef.current();
      }
    };
  }, []);

  return {
    triggerCleanup: memoryManager.triggerCleanup.bind(memoryManager)
  };
};
```

## 网络优化策略

### 1. 缓存策略

```javascript
// src/utils/cacheStrategies.js
class CacheStrategyManager {
  constructor() {
    this.strategies = new Map();
    this.setupDefaultStrategies();
  }

  setupDefaultStrategies() {
    // 静态资源缓存策略
    this.addStrategy('static', {
      match: request => request.url.includes('/static/'),
      handler: async request => {
        const cache = await caches.open('static-v1');
        const cached = await cache.match(request);
        
        if (cached) {
          return cached;
        }
        
        const response = await fetch(request);
        if (response.ok) {
          cache.put(request, response.clone());
        }
        
        return response;
      }
    });

    // API 缓存策略
    this.addStrategy('api', {
      match: request => request.url.includes('/api/'),
      handler: async request => {
        const cache = await caches.open('api-v1');
        
        try {
          const response = await fetch(request);
          if (response.ok) {
            cache.put(request, response.clone());
          }
          return response;
        } catch (error) {
          const cached = await cache.match(request);
          if (cached) {
            return cached;
          }
          throw error;
        }
      }
    });

    // 图片缓存策略
    this.addStrategy('images', {
      match: request => request.url.match(/\.(png|jpg|jpeg|webp|avif)$/),
      handler: async request => {
        const cache = await caches.open('images-v1');
        const cached = await cache.match(request);
        
        if (cached) {
          return cached;
        }
        
        const response = await fetch(request);
        if (response.ok) {
          cache.put(request, response.clone());
        }
        
        return response;
      }
    });
  }

  addStrategy(name, strategy) {
    this.strategies.set(name, strategy);
  }

  getStrategyForRequest(request) {
    for (const [name, strategy] of this.strategies) {
      if (strategy.match(request)) {
        return strategy.handler;
      }
    }
    
    // 默认策略
    return async (request) => {
      return fetch(request);
    };
  }

  async handleRequest(request) {
    const handler = this.getStrategyForRequest(request);
    return handler(request);
  }
}

export const cacheStrategyManager = new CacheStrategyManager();

// Service Worker 集成
self.addEventListener('fetch', (event) => {
  if (event.request.method === 'GET') {
    event.respondWith(
      cacheStrategyManager.handleRequest(event.request)
    );
  }
});
```

### 2. 连接优化

```javascript
// src/utils/connectionOptimizer.js
class ConnectionOptimizer {
  constructor() {
    this.connection = navigator.connection;
    this.adapters = new Map();
  }

  // 根据网络状况调整策略
  adaptToNetworkConditions() {
    if (!this.connection) return;

    const conditions = {
      effectiveType: this.connection.effectiveType,
      downlink: this.connection.downlink,
      rtt: this.connection.rtt,
      saveData: this.connection.saveData
    };

    this.applyAdaptations(conditions);
  }

  applyAdaptations(conditions) {
    // 低速网络适配
    if (conditions.effectiveType === 'slow-2g' || conditions.effectiveType === '2g') {
      this.enableLowBandwidthMode();
    }

    // 节省数据模式
    if (conditions.saveData) {
      this.enableDataSaverMode();
    }

    // 高延迟网络
    if (conditions.rtt > 300) {
      this.enableHighLatencyMode();
    }

    // 触发适配器
    this.adapters.forEach(adapter => {
      adapter(conditions);
    });
  }

  enableLowBandwidthMode() {
    // 降低图片质量
    document.querySelectorAll('img').forEach(img => {
      if (img.dataset.lowSrc) {
        img.src = img.dataset.lowSrc;
      }
    });

    // 禁用视频自动播放
    document.querySelectorAll('video').forEach(video => {
      video.autoplay = false;
    });

    // 减少数据请求
    this.throttleRequests();
  }

  enableDataSaverMode() {
    // 使用更轻量的资源
    console.log('Data saver mode enabled');
  }

  enableHighLatencyMode() {
    // 增加超时时间
    console.log('High latency mode enabled');
  }

  throttleRequests() {
    // 实现请求节流
    console.log('Requests throttled for low bandwidth');
  }

  registerAdapter(adapter) {
    const id = Math.random().toString(36);
    this.adapters.set(id, adapter);
    
    return () => {
      this.adapters.delete(id);
    };
  }

  // 网络变化监听
  setupNetworkListeners() {
    if (this.connection) {
      this.connection.addEventListener('change', () => {
        this.adaptToNetworkConditions();
      });
    }

    window.addEventListener('online', () => {
      this.onConnectionRestored();
    });

    window.addEventListener('offline', () => {
      this.onConnectionLost();
    });
  }

  onConnectionRestored() {
    console.log('Connection restored');
    // 重新尝试失败的请求
  }

  onConnectionLost() {
    console.log('Connection lost');
    // 进入离线模式
  }
}

export const connectionOptimizer = new ConnectionOptimizer();

// React Hook 网络适配
export const useNetworkAdaptation = () => {
  const [networkConditions, setNetworkConditions] = useState({});

  useEffect(() => {
    const updateConditions = () => {
      if (navigator.connection) {
        setNetworkConditions({
          effectiveType: navigator.connection.effectiveType,
          downlink: navigator.connection.downlink,
          rtt: navigator.connection.rtt,
          saveData: navigator.connection.saveData
        });
      }
    };

    const unsubscribe = connectionOptimizer.registerAdapter(updateConditions);
    connectionOptimizer.setupNetworkListeners();
    connectionOptimizer.adaptToNetworkConditions();

    return unsubscribe;
  }, []);

  return networkConditions;
};
```

## 构建与部署优化

### 1. 构建优化配置

```javascript
// webpack.config.optimization.js
const path = require('path');
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const { BundleAnalyzerPlugin } = require('bundle-analyzer-plugin');
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  mode: 'production',
  
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        parallel: true,
        terserOptions: {
          compress: {
            drop_console: true,
            drop_debugger: true,
            pure_funcs: ['console.log']
          },
          mangle: {
            properties: {
              regex: /^_/ // 混淆以下划线开头的属性
            }
          }
        }
      }),
      new CssMinimizerPlugin({
        minimizerOptions: {
          preset: ['default', { discardComments: { removeAll: true } }]
        }
      })
    ],
    
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
          chunks: 'all'
        },
        react: {
          test: /react|react-dom[\\/]/,
          name: 'react',
          priority: 20,
          chunks: 'all'
        },
        common: {
          name: 'common',
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    },
    
    runtimeChunk: {
      name: 'runtime'
    }
  },
  
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html'
    }),
    
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 8192,
      minRatio: 0.8
    }),
    
    new CompressionPlugin({
      filename: '[path][base].br',
      algorithm: 'brotliCompress',
      test: /\.(js|css|html|svg)$/,
      compressionOptions: {
        level: 11
      },
      threshold: 8192,
      minRatio: 0.8
    })
  ],
  
  performance: {
    hints: 'warning',
    maxEntrypointSize: 512000,
    maxAssetSize: 512000
  }
};
```

### 2. 部署优化策略

```javascript
// deploy-optimization.js
const fs = require('fs-extra');
const path = require('path');
const { execSync } = require('child_process');

class DeploymentOptimizer {
  constructor(buildDir) {
    this.buildDir = buildDir;
  }

  // 优化静态资源
  async optimizeAssets() {
    await this.optimizeImages();
    await this.generateWebp();
    await this.generateSizes();
    await this.addCacheHeaders();
  }

  // 图片优化
  async optimizeImages() {
    const imagesDir = path.join(this.buildDir, 'static', 'media');
    
    if (await fs.pathExists(imagesDir)) {
      // 使用 imagemin 优化图片
      const files = await fs.readdir(imagesDir);
      
      for (const file of files) {
        if (/\.(png|jpg|jpeg)$/.test(file)) {
          const filePath = path.join(imagesDir, file);
          await this.compressImage(filePath);
        }
      }
    }
  }

  // 生成 WebP 格式
  async generateWebp() {
    const imagesDir = path.join(this.buildDir, 'static', 'media');
    
    if (await fs.pathExists(imagesDir)) {
      const files = await fs.readdir(imagesDir);
      
      for (const file of files) {
        if (/\.(png|jpg|jpeg)$/.test(file)) {
          const filePath = path.join(imagesDir, file);
          const webpPath = filePath.replace(/\.(png|jpg|jpeg)$/, '.webp');
          
          await this.convertToWebp(filePath, webpPath);
        }
      }
    }
  }

  // 生成尺寸声明
  async generateSizes() {
    const htmlFiles = await this.findHtmlFiles();
    
    for (const file of htmlFiles) {
      let content = await fs.readFile(file, 'utf8');
      
      // 为图片添加 width 和 height
      content = content.replace(
        /<img([^>]*)src="([^"]*)"([^>]*)>/g,
        (match, before, src, after) => {
          if (!before.includes('width=') && !after.includes('width=')) {
            const dimensions = this.getImageDimensions(src);
            if (dimensions) {
              return `<img${before}src="${src}" width="${dimensions.width}" height="${dimensions.height}"${after}>`;
            }
          }
          return match;
        }
      );
      
      await fs.writeFile(file, content);
    }
  }

  // 添加缓存头
  async addCacheHeaders() {
    const config = {
      '*.js': 'public, max-age=31536000, immutable',
      '*.css': 'public, max-age=31536000, immutable',
      '*.woff2': 'public, max-age=31536000, immutable',
      '*.webp': 'public, max-age=31536000, immutable',
      '*.html': 'public, max-age=0, must-revalidate'
    };
    
    // 生成 .htaccess 或 netlify.toml 配置
    await this.generateCacheConfig(config);
  }

  // 工具方法
  async compressImage(filePath) {
    // 实现图片压缩逻辑
  }

  async convertToWebp(sourcePath, destPath) {
    // 实现 WebP 转换逻辑
  }

  getImageDimensions(src) {
    // 获取图片尺寸逻辑
    return null;
  }

  async findHtmlFiles() {
    return await fs.glob(path.join(this.buildDir, '**/*.html'));
  }

  async generateCacheConfig(config) {
    // 生成缓存配置逻辑
  }
}

// 使用示例
const optimizer = new DeploymentOptimizer('./build');
optimizer.optimizeAssets().then(() => {
  console.log('Deployment optimization completed');
});
```

## 监控与分析

### 1. 实时性能监控

```javascript
// src/utils/realTimeMonitoring.js
class RealTimeMonitor {
  constructor() {
    this.metrics = new Map();
    this.subscribers = new Set();
    this.reportInterval = 5000;
  }

  // 开始监控
  start() {
    this.setupPerformanceObservers();
    this.setupResourceMonitoring();
    this.setupErrorMonitoring();
    this.startReporting();
  }

  // 性能观察器设置
  setupPerformanceObservers() {
    // 长任务监控
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        list.getEntries().forEach(entry => {
          this.recordMetric('longtask', {
            duration: entry.duration,
            name: entry.name
          });
        });
      });
      observer.observe({ entryTypes: ['longtask'] });
    }
  }

  // 资源监控
  setupResourceMonitoring() {
    // 资源加载时间监控
    const resources = performance.getEntriesByType('resource');
    resources.forEach(resource => {
      this.recordMetric('resource', {
        name: resource.name,
        duration: resource.duration,
        type: resource.initiatorType
      });
    });
  }

  // 错误监控
  setupErrorMonitoring() {
    window.addEventListener('error', (event) => {
      this.recordMetric('error', {
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno
      });
    });

    window.addEventListener('unhandledrejection', (event) => {
      this.recordMetric('unhandledrejection', {
        reason: event.reason
      });
    });
  }

  // 记录指标
  recordMetric(type, data) {
    const timestamp = Date.now();
    const metric = { type, data, timestamp };
    
    if (!this.metrics.has(type)) {
      this.metrics.set(type, []);
    }
    
    this.metrics.get(type).push(metric);
    
    // 通知订阅者
    this.notifySubscribers(metric);
  }

  // 通知订阅者
  notifySubscribers(metric) {
    this.subscribers.forEach(subscriber => {
      try {
        subscriber(metric);
      } catch (error) {
        console.error('Subscriber error:', error);
      }
    });
  }

  // 开始报告
  startReporting() {
    setInterval(() => {
      this.reportMetrics();
    }, this.reportInterval);
  }

  // 报告指标
  async reportMetrics() {
    const report = this.generateReport();
    
    if (Object.keys(report).length > 0) {
      try {
        await this.sendReport(report);
        this.clearReportedMetrics();
      } catch (error) {
        console.error('Failed to send metrics report:', error);
      }
    }
  }

  // 生成报告
  generateReport() {
    const report = {};
    
    this.metrics.forEach((metrics, type) => {
      report[type] = metrics.slice(-100); // 保留最近100条记录
    });
    
    return report;
  }

  // 发送报告
  async sendReport(report) {
    const response = await fetch('/api/performance/metrics', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(report)
    });

    if (!response.ok) {
      throw new Error('Failed to send metrics');
    }
  }

  // 清除已报告的指标
  clearReportedMetrics() {
    this.metrics.clear();
  }

  // 订阅指标更新
  subscribe(callback) {
    this.subscribers.add(callback);
    
    return () => {
      this.subscribers.delete(callback);
    };
  }

  // 获取当前指标
  getCurrentMetrics() {
    return this.generateReport();
  }
}

export const realTimeMonitor = new RealTimeMonitor();

// React Hook 集成
export const useRealTimeMetrics = () => {
  const [metrics, setMetrics] = useState({});

  useEffect(() => {
    const unsubscribe = realTimeMonitor.subscribe((metric) => {
      setMetrics(prev => ({
        ...prev,
        [metric.type]: [...(prev[metric.type] || []), metric]
      }));
    });

    return unsubscribe;
  }, []);

  return metrics;
};
```
