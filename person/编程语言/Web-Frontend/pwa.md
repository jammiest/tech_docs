# PWA 与离线应用

渐进式 Web 应用（PWA）结合了 Web 和原生应用的优点，提供可靠的性能、离线功能和原生般的用户体验。本指南将深入探讨 PWA 的实现策略和最佳实践。

## 核心特性与标准

### 1. PWA 核心要求

```javascript
// PWA 检查清单
const pwaChecklist = {
  required: {
    https: true,                    // 必须使用 HTTPS
    serviceWorker: true,            // 必须注册 Service Worker
    webAppManifest: true,           // 必须提供 Manifest 文件
    responsiveDesign: true,          // 必须响应式设计
  },
  recommended: {
    offlineCapability: true,        // 推荐离线功能
    pushNotifications: true,        // 推荐推送通知
    appLikeExperience: true,        // 推荐应用化体验
    fastLoading: true,              // 推荐快速加载
  },
  optional: {
    backgroundSync: true,           // 可选后台同步
    periodicSync: true,             // 可选定期同步
    paymentRequest: true,           // 可选支付请求
  }
};

// PWA 评分函数
function ratePWA(app) {
  let score = 0;
  const maxScore = 100;
  
  // 检查必需项
  if (app.https) score += 20;
  if (app.serviceWorker) score += 20;
  if (app.webAppManifest) score += 20;
  if (app.responsiveDesign) score += 20;
  
  // 检查推荐项
  if (app.offlineCapability) score += 5;
  if (app.pushNotifications) score += 5;
  if (app.appLikeExperience) score += 5;
  if (app.fastLoading) score += 5;
  
  return {
    score,
    grade: score >= 80 ? 'A' : 
           score >= 60 ? 'B' : 
           score >= 40 ? 'C' : 'D'
  };
}
```

### 2. Web App Manifest

```json
// public/manifest.json
{
  "name": "我的渐进式应用",
  "short_name": "我的应用",
  "description": "一个功能丰富的渐进式Web应用",
  "start_url": "/?source=pwa",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#007bff",
  "orientation": "portrait-primary",
  "scope": "/",
  "lang": "zh-CN",
  "categories": ["productivity", "utilities"],
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/homepage.png",
      "sizes": "1280x720",
      "type": "image/png",
      "label": "主页截图"
    }
  ],
  "shortcuts": [
    {
      "name": "新建项目",
      "short_name": "新建",
      "description": "创建一个新项目",
      "url": "/create?source=shortcut",
      "icons": [{ "src": "/icons/add.png", "sizes": "96x96" }]
    }
  ]
}
```

## Service Worker 实现

### 1. Service Worker 注册

```javascript
// src/serviceWorkerRegistration.js
class ServiceWorkerRegistration {
  constructor() {
    this.isSupported = 'serviceWorker' in navigator;
    this.registration = null;
  }

  // 注册 Service Worker
  async register(swUrl, config = {}) {
    if (!this.isSupported) {
      console.warn('Service Worker 不受支持');
      return;
    }

    if (process.env.NODE_ENV === 'production') {
      try {
        this.registration = await navigator.serviceWorker.register(swUrl, config);
        
        this.registration.onupdatefound = () => {
          const installingWorker = this.registration.installing;
          if (installingWorker == null) return;

          installingWorker.onstatechange = () => {
            if (installingWorker.state === 'installed') {
              if (navigator.serviceWorker.controller) {
                console.log('新内容已可用，请刷新页面');
                this.dispatchEvent('updateAvailable');
              } else {
                console.log('内容已缓存，可供离线使用');
              }
            }
          };
        };

        // 监听控制器变化
        navigator.serviceWorker.addEventListener('controllerchange', () => {
          console.log('Service Worker 控制器已变更');
        });

        return this.registration;
      } catch (error) {
        console.error('Service Worker 注册失败:', error);
        throw error;
      }
    }
  }

  // 卸载 Service Worker
  async unregister() {
    if (!this.isSupported) return false;

    if (this.registration) {
      const result = await this.registration.unregister();
      if (result) {
        console.log('Service Worker 已卸载');
      }
      return result;
    }
    return false;
  }

  // 检查更新
  async checkForUpdates() {
    if (!this.registration) return;
    
    try {
      await this.registration.update();
      console.log('已检查 Service Worker 更新');
    } catch (error) {
      console.error('检查更新失败:', error);
    }
  }

  // 自定义事件系统
  dispatchEvent(eventName, detail = {}) {
    const event = new CustomEvent(`sw:${eventName}`, { detail });
    window.dispatchEvent(event);
  }

  // 监听事件
  on(eventName, callback) {
    window.addEventListener(`sw:${eventName}`, callback);
    return () => window.removeEventListener(`sw:${eventName}`, callback);
  }
}

// 创建全局实例
const swRegistration = new ServiceWorkerRegistration();

// 在应用入口注册
export const registerSW = () => {
  if (process.env.NODE_ENV === 'production') {
    swRegistration.register('/sw.js', {
      scope: '/',
      updateViaCache: 'none'
    }).then(registration => {
      console.log('SW registered: ', registration);
    });
  }
};

export default swRegistration;
```

### 2. Service Worker 核心逻辑

```javascript
// public/sw.js
const CACHE_NAME = 'my-pwa-v1';
const OFFLINE_URL = '/offline.html';
const PRECACHE_URLS = [
  '/',
  '/static/js/bundle.js',
  '/static/css/main.css',
  '/manifest.json',
  OFFLINE_URL
];

// 安装阶段 - 预缓存关键资源
self.addEventListener('install', (event) => {
  console.log('Service Worker 安装中...');
  
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('缓存关键资源');
        return cache.addAll(PRECACHE_URLS);
      })
      .then(() => {
        console.log('跳过等待阶段');
        return self.skipWaiting();
      })
  );
});

// 激活阶段 - 清理旧缓存
self.addEventListener('activate', (event) => {
  console.log('Service Worker 激活中...');
  
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== CACHE_NAME) {
            console.log('删除旧缓存:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    }).then(() => {
      console.log('声明控制权');
      return self.clients.claim();
    })
  );
});

//  fetch 事件 - 缓存优先策略
self.addEventListener('fetch', (event) => {
  // 跳过非 GET 请求和特殊请求
  if (event.request.method !== 'GET') return;
  if (event.request.url.includes('/api/') || 
      event.request.url.includes('/socket.io/')) {
    return;
  }

  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // 返回缓存或网络请求
        return response || fetch(event.request)
          .then(networkResponse => {
            // 缓存新资源
            return caches.open(CACHE_NAME)
              .then(cache => {
                cache.put(event.request, networkResponse.clone());
                return networkResponse;
              });
          })
          .catch(() => {
            // 网络失败时处理
            if (event.request.mode === 'navigate') {
              return caches.match(OFFLINE_URL);
            }
            return new Response('网络连接失败', {
              status: 503,
              statusText: 'Service Unavailable'
            });
          });
      })
  );
});

// 后台同步
self.addEventListener('sync', (event) => {
  if (event.tag === 'background-sync') {
    console.log('后台同步触发');
    event.waitUntil(doBackgroundSync());
  }
});

// 推送通知
self.addEventListener('push', (event) => {
  const data = event.data ? event.data.json() : {};
  
  const options = {
    body: data.body || '您有新消息',
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    vibrate: [200, 100, 200],
    tag: 'push-notification',
    data: {
      url: data.url || '/'
    },
    actions: [
      {
        action: 'open',
        title: '打开应用'
      },
      {
        action: 'dismiss',
        title: '忽略'
      }
    ]
  };

  event.waitUntil(
    self.registration.showNotification(data.title || '通知', options)
  );
});

// 通知点击处理
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'open') {
    event.waitUntil(
      clients.openWindow(event.notification.data.url)
    );
  }
});
```

## 离线功能实现

### 1. 离线数据管理

```javascript
// src/utils/offlineManager.js
class OfflineManager {
  constructor() {
    this.dbName = 'OfflineData';
    this.dbVersion = 1;
    this.db = null;
    this.isSupported = 'indexedDB' in window;
  }

  // 初始化数据库
  async init() {
    if (!this.isSupported) return;

    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.dbVersion);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve(this.db);
      };

      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        
        // 创建对象存储
        if (!db.objectStoreNames.contains('requests')) {
          const store = db.createObjectStore('requests', { 
            keyPath: 'id',
            autoIncrement: true 
          });
          store.createIndex('url', 'url', { unique: false });
          store.createIndex('method', 'method', { unique: false });
          store.createIndex('timestamp', 'timestamp', { unique: false });
        }

        if (!db.objectStoreNames.contains('data')) {
          const store = db.createObjectStore('data', {
            keyPath: 'key'
          });
        }
      };
    });
  }

  // 存储离线请求
  async storeRequest(request) {
    if (!this.db) await this.init();

    const transaction = this.db.transaction(['requests'], 'readwrite');
    const store = transaction.objectStore('requests');

    const offlineRequest = {
      url: request.url,
      method: request.method,
      headers: Object.fromEntries(request.headers.entries()),
      body: await request.clone().text(),
      timestamp: Date.now()
    };

    return store.add(offlineRequest);
  }

  // 获取待处理请求
  async getPendingRequests() {
    if (!this.db) await this.init();

    const transaction = this.db.transaction(['requests'], 'readonly');
    const store = transaction.objectStore('requests');
    
    return new Promise((resolve) => {
      const requests = [];
      store.openCursor().onsuccess = (event) => {
        const cursor = event.target.result;
        if (cursor) {
          requests.push(cursor.value);
          cursor.continue();
        } else {
          resolve(requests);
        }
      };
    });
  }

  // 删除已处理请求
  async deleteRequest(id) {
    if (!this.db) await this.init();

    const transaction = this.db.transaction(['requests'], 'readwrite');
    const store = transaction.objectStore('requests');
    
    return store.delete(id);
  }

  // 存储应用数据
  async storeData(key, data) {
    if (!this.db) await this.init();

    const transaction = this.db.transaction(['data'], 'readwrite');
    const store = transaction.objectStore('data');
    
    return store.put({ key, data, timestamp: Date.now() });
  }

  // 获取存储的数据
  async getData(key) {
    if (!this.db) await this.init();

    const transaction = this.db.transaction(['data'], 'readonly');
    const store = transaction.objectStore('data');
    
    return new Promise((resolve) => {
      const request = store.get(key);
      request.onsuccess = () => resolve(request.result?.data);
      request.onerror = () => resolve(null);
    });
  }

  // 同步离线数据到服务器
  async syncOfflineData() {
    if (!navigator.onLine) return;

    const requests = await this.getPendingRequests();
    
    for (const request of requests) {
      try {
        const response = await fetch(request.url, {
          method: request.method,
          headers: request.headers,
          body: request.method !== 'GET' ? request.body : undefined
        });

        if (response.ok) {
          await this.deleteRequest(request.id);
          console.log(`同步成功: ${request.url}`);
        }
      } catch (error) {
        console.warn(`同步失败: ${request.url}`, error);
      }
    }
  }
}

// 全局离线管理器
export const offlineManager = new OfflineManager();

// React Hook 封装
export const useOfflineData = (key, defaultValue = null) => {
  const [data, setData] = useState(defaultValue);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const loadData = async () => {
      const storedData = await offlineManager.getData(key);
      setData(storedData || defaultValue);
      setIsLoading(false);
    };

    loadData();
  }, [key, defaultValue]);

  const updateData = async (newData) => {
    setData(newData);
    await offlineManager.storeData(key, newData);
  };

  return [data, updateData, isLoading];
};
```

### 2. 网络状态检测

```javascript
// src/hooks/useNetworkStatus.js
import { useState, useEffect } from 'react';

export const useNetworkStatus = () => {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  const [connection, setConnection] = useState(null);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    const handleConnectionChange = () => {
      setConnection(navigator.connection);
    };

    // 基础网络状态监听
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    // 网络信息 API（如果可用）
    if (navigator.connection) {
      navigator.connection.addEventListener('change', handleConnectionChange);
      setConnection(navigator.connection);
    }

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
      
      if (navigator.connection) {
        navigator.connection.removeEventListener('change', handleConnectionChange);
      }
    };
  }, []);

  return {
    isOnline,
    connection,
    effectiveType: connection?.effectiveType,
    downlink: connection?.downlink,
    rtt: connection?.rtt,
    saveData: connection?.saveData
  };
};

// 网络状态组件
export const NetworkStatusIndicator = () => {
  const { isOnline, effectiveType } = useNetworkStatus();

  return (
    <div className={`network-status ${isOnline ? 'online' : 'offline'}`}>
      {isOnline ? (
        <>
          <span className="status-dot online"></span>
          {effectiveType && <span>网络类型: {effectiveType}</span>}
        </>
      ) : (
        <>
          <span className="status-dot offline"></span>
          <span>离线模式</span>
        </>
      )}
    </div>
  );
};
```

## 推送通知系统

### 1. 推送通知管理

```javascript
// src/services/pushNotification.js
class PushNotificationService {
  constructor() {
    this.isSupported = 'PushManager' in window && 'Notification' in window;
    this.subscription = null;
    this.publicKey = process.env.VAPID_PUBLIC_KEY;
  }

  // 请求通知权限
  async requestPermission() {
    if (!this.isSupported) {
      throw new Error('推送通知不受支持');
    }

    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }

  // 订阅推送服务
  async subscribe() {
    if (!this.isSupported) return null;

    try {
      const registration = await navigator.serviceWorker.ready;
      this.subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: this.urlBase64ToUint8Array(this.publicKey)
      });

      // 将订阅信息发送到服务器
      await this.sendSubscriptionToServer(this.subscription);
      
      return this.subscription;
    } catch (error) {
      console.error('推送订阅失败:', error);
      throw error;
    }
  }

  // 取消订阅
  async unsubscribe() {
    if (!this.subscription) return;

    try {
      await this.subscription.unsubscribe();
      await this.removeSubscriptionFromServer(this.subscription);
      this.subscription = null;
    } catch (error) {
      console.error('取消订阅失败:', error);
    }
  }

  // 检查订阅状态
  async checkSubscription() {
    if (!this.isSupported) return false;

    const registration = await navigator.serviceWorker.ready;
    this.subscription = await registration.pushManager.getSubscription();
    return this.subscription !== null;
  }

  // 发送本地通知
  async showLocalNotification(title, options = {}) {
    if (!this.isSupported) return;

    const registration = await navigator.serviceWorker.ready;
    return registration.showNotification(title, {
      icon: '/icons/icon-192x192.png',
      badge: '/icons/badge-72x72.png',
      ...options
    });
  }

  // 工具函数：Base64 转换
  urlBase64ToUint8Array(base64String) {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
      .replace(/-/g, '+')
      .replace(/_/g, '/');

    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);

    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }

    return outputArray;
  }

  // 发送订阅到服务器
  async sendSubscriptionToServer(subscription) {
    const response = await fetch('/api/push/subscribe', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        subscription,
        userAgent: navigator.userAgent
      })
    });

    if (!response.ok) {
      throw new Error('订阅信息发送失败');
    }
  }

  // 从服务器移除订阅
  async removeSubscriptionFromServer(subscription) {
    await fetch('/api/push/unsubscribe', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ subscription })
    });
  }
}

// 全局推送服务实例
export const pushService = new PushNotificationService();

// React Hook 封装
export const usePushNotifications = () => {
  const [permission, setPermission] = useState('default');
  const [isSubscribed, setIsSubscribed] = useState(false);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const init = async () => {
      if (!pushService.isSupported) {
        setLoading(false);
        return;
      }

      // 检查权限状态
      setPermission(Notification.permission);
      
      // 检查订阅状态
      const subscribed = await pushService.checkSubscription();
      setIsSubscribed(subscribed);
      
      setLoading(false);
    };

    init();
  }, []);

  const requestPermission = async () => {
    const granted = await pushService.requestPermission();
    setPermission(granted ? 'granted' : 'denied');
    return granted;
  };

  const subscribe = async () => {
    await pushService.subscribe();
    setIsSubscribed(true);
  };

  const unsubscribe = async () => {
    await pushService.unsubscribe();
    setIsSubscribed(false);
  };

  return {
    isSupported: pushService.isSupported,
    permission,
    isSubscribed,
    loading,
    requestPermission,
    subscribe,
    unsubscribe,
    showLocalNotification: pushService.showLocalNotification
  };
};
```

## 性能优化策略

### 1. 资源预加载

```javascript
// src/utils/preloadManager.js
class PreloadManager {
  constructor() {
    this.preloadedResources = new Set();
    this.isSupported = 'link' in document && 'relList' in document.createElement('link');
  }

  // 预加载资源
  preloadResource(url, as = 'script') {
    if (!this.isSupported || this.preloadedResources.has(url)) return;

    return new Promise((resolve, reject) => {
      const link = document.createElement('link');
      link.rel = 'preload';
      link.as = as;
      link.href = url;
      
      link.onload = () => {
        this.preloadedResources.add(url);
        resolve();
      };
      
      link.onerror = reject;
      
      document.head.appendChild(link);
    });
  }

  // 预连接到源
  preconnect(url) {
    if (!this.isSupported) return;

    const link = document.createElement('link');
    link.rel = 'preconnect';
    link.href = url;
    document.head.appendChild(link);
  }

  // 预加载关键资源
  async preloadCriticalResources() {
    const criticalResources = [
      { url: '/static/css/main.css', as: 'style' },
      { url: '/static/js/runtime.js', as: 'script' },
      { url: '/static/js/main.js', as: 'script' },
      { url: '/api/user', as: 'fetch' }
    ];

    await Promise.all(
      criticalResources.map(resource => 
        this.preloadResource(resource.url, resource.as)
      )
    );
  }

  // 基于路由的预加载
  preloadForRoute(route) {
    const routePreloads = {
      '/dashboard': [
        '/static/js/dashboard.js',
        '/api/dashboard/data'
      ],
      '/profile': [
        '/static/js/profile.js',
        '/api/user/profile'
      ]
    };

    const resources = routePreloads[route] || [];
    resources.forEach(url => this.preloadResource(url));
  }
}

export const preloadManager = new PreloadManager();

// 在路由组件中使用
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

export const RoutePreloader = () => {
  const location = useLocation();

  useEffect(() => {
    preloadManager.preloadForRoute(location.pathname);
  }, [location.pathname]);

  return null;
};
```

### 2. 缓存策略优化

```javascript
// 高级 Service Worker 缓存策略
const CACHE_STRATEGIES = {
  // 缓存优先，网络回退
  CACHE_FIRST: async (request) => {
    const cachedResponse = await caches.match(request);
    if (cachedResponse) return cachedResponse;
    
    try {
      const networkResponse = await fetch(request);
      const cache = await caches.open(CACHE_NAME);
      cache.put(request, networkResponse.clone());
      return networkResponse;
    } catch (error) {
      return new Response('网络不可用', { status: 503 });
    }
  },

  // 网络优先，缓存回退
  NETWORK_FIRST: async (request) => {
    try {
      const networkResponse = await fetch(request);
      const cache = await caches.open(CACHE_NAME);
      cache.put(request, networkResponse.clone());
      return networkResponse;
    } catch (error) {
      const cachedResponse = await caches.match(request);
      if (cachedResponse) return cachedResponse;
      throw error;
    }
  },

  // 仅缓存
  CACHE_ONLY: async (request) => {
    const cachedResponse = await caches.match(request);
    if (!cachedResponse) {
      throw new Error('缓存中未找到资源');
    }
    return cachedResponse;
  },

  // 仅网络
  NETWORK_ONLY: async (request) => {
    const networkResponse = await fetch(request);
    if (!networkResponse.ok) {
      throw new Error('网络请求失败');
    }
    return networkResponse;
  },

  // 重新验证后台更新
  STALE_WHILE_REVALIDATE: async (request) => {
    const cachedResponse = await caches.match(request);
    const fetchPromise = fetch(request)
      .then(networkResponse => {
        const cache = await caches.open(CACHE_NAME);
        cache.put(request, networkResponse.clone());
        return networkResponse;
      })
      .catch(() => {}); // 静默失败

    return cachedResponse || fetchPromise;
  }
};

// 根据请求类型选择策略
function getStrategyForRequest(request) {
  const url = new URL(request.url);
  
  if (url.pathname.startsWith('/static/')) {
    return CACHE_STRATEGIES.CACHE_FIRST;
  }
  
  if (url.pathname.startsWith('/api/')) {
    return CACHE_STRATEGIES.NETWORK_FIRST;
  }
  
  if (url.pathname.endsWith('.html')) {
    return CACHE_STRATEGIES.NETWORK_FIRST;
  }
  
  return CACHE_STRATEGIES.STALE_WHILE_REVALIDATE;
}
```

## 测试与调试

### 1. PWA 测试工具

```javascript
// src/utils/pwaTesting.js
import { audit } from 'lighthouse';
import { generate } from 'pwa-analyzer';

export class PWATesting {
  static async runLighthouseAudit(url) {
    const results = await audit(url, {
      extends: 'lighthouse:default',
      settings: {
        onlyCategories: ['pwa']
      }
    });
    
    return this.parseLighthouseResults(results);
  }

  static parseLighthouseResults(results) {
    const categories = results.lhr.categories;
    const audits = results.lhr.audits;
    
    return {
      score: categories.pwa.score * 100,
      metrics: {
        installable: audits.installable.score,
        serviceWorker: audits.serviceWorker.score,
        splashScreen: audits.splashScreen.score,
        themeColor: audits.themeColor.score,
        viewport: audits.viewport.score
      },
      recommendations: Object.values(audits)
        .filter(audit => audit.score !== 1)
        .map(audit => ({
          id: audit.id,
          title: audit.title,
          description: audit.description,
          score: audit.score * 100
        }))
    };
  }

  static async checkManifest() {
    try {
      const response = await fetch('/manifest.json');
      const manifest = await response.json();
      
      return this.validateManifest(manifest);
    } catch (error) {
      return { valid: false, error: 'Manifest 文件不存在' };
    }
  }

  static validateManifest(manifest) {
    const requiredFields = ['name', 'short_name', 'start_url', 'display', 'icons'];
    const errors = [];
    
    requiredFields.forEach(field => {
      if (!manifest[field]) {
        errors.push(`缺少必需字段: ${field}`);
      }
    });
    
    // 验证图标
    if (manifest.icons) {
      const requiredSizes = [192, 512];
      const existingSizes = manifest.icons.map(icon => 
        parseInt(icon.sizes?.split('x')[0] || 0)
      );
      
      requiredSizes.forEach(size => {
        if (!existingSizes.includes(size)) {
          errors.push(`缺少 ${size}x${size} 尺寸的图标`);
        }
      });
    }
    
    return {
      valid: errors.length === 0,
      errors,
      warnings: this.generateWarnings(manifest)
    };
  }

  static generateWarnings(manifest) {
    const warnings = [];
    
    if (!manifest.theme_color) {
      warnings.push('建议设置 theme_color');
    }
    
    if (!manifest.background_color) {
      warnings.push('建议设置 background_color');
    }
    
    if (!manifest.categories || manifest.categories.length === 0) {
      warnings.push('建议设置 categories');
    }
    
    return warnings;
  }
}

// PWA 健康检查
export const runPWAHealthCheck = async () => {
  const [lighthouseResults, manifestResults] = await Promise.all([
    PWATesting.runLighthouseAudit(window.location.origin),
    PWATesting.checkManifest()
  ]);

  return {
    lighthouse: lighthouseResults,
    manifest: manifestResults,
    overallScore: (lighthouseResults.score + (manifestResults.valid ? 100 : 0)) / 2
  };
};
```

## 部署与监控

### 1. 部署配置

```javascript
// build-pwa.js - 构建脚本
const { generateSW } = require('workbox-build');
const fs = require('fs-extra');
const path = require('path');

async function buildPWA() {
  console.log('开始构建 PWA...');
  
  // 生成 Service Worker
  const { count, size } = await generateSW({
    swDest: 'build/sw.js',
    globDirectory: 'build',
    globPatterns: ['**/*.{js,css,html,json,png,svg,ico}'],
    navigateFallback: '/index.html',
    navigateFallbackAllowlist: [/^(?!\/__).*/],
    runtimeCaching: [
      {
        urlPattern: /\.(?:png|jpg|jpeg|svg|ico)$/,
        handler: 'CacheFirst',
        options: {
          cacheName: 'images',
          expiration: {
            maxEntries: 50,
            maxAgeSeconds: 30 * 24 * 60 * 60 // 30天
          }
        }
      },
      {
        urlPattern: /\.(?:js|css)$/,
        handler: 'StaleWhileRevalidate',
        options: {
          cacheName: 'static-resources'
        }
      },
      {
        urlPattern: /\/api\/.*/,
        handler: 'NetworkFirst',
        options: {
          cacheName: 'api-cache',
          networkTimeoutSeconds: 3,
          expiration: {
            maxEntries: 50,
            maxAgeSeconds: 5 * 60 // 5分钟
          }
        }
      }
    ]
  });

  console.log(`预缓存 ${count} 个文件，总大小 ${size} 字节`);
  
  // 生成 manifest.json
  const manifest = {
    name: process.env.APP_NAME,
    short_name: process.env.APP_SHORT_NAME,
    description: process.env.APP_DESCRIPTION,
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: process.env.PRIMARY_COLOR,
    icons: [
      {
        src: '/icons/icon-192.png',
        sizes: '192x192',
        type: 'image/png'
      },
      {
        src: '/icons/icon-512.png',
        sizes: '512x512',
        type: 'image/png'
      }
    ]
  };
  
  await fs.writeJson(
    path.join('build', 'manifest.json'),
    manifest,
    { spaces: 2 }
  );
  
  console.log('PWA 构建完成');
}

buildPWA().catch(console.error);
```

### 2. 监控与分析

```javascript
// src/utils/pwaAnalytics.js
export class PWAAnalytics {
  static trackEvent(category, action, label, value) {
    if (window.gtag) {
      window.gtag('event', action, {
        event_category: category,
        event_label: label,
        value: value
      });
    }
    
    // 备用日志
    console.log(`PWA Event: ${category}.${action}`, { label, value });
  }

  static trackAppInstall() {
    window.addEventListener('appinstalled', () => {
      this.trackEvent('pwa', 'install', 'app_installed');
    });
  }

  static trackOfflineUsage() {
    const trackOfflineEvent = () => {
      this.trackEvent('pwa', 'offline_usage', 'offline_mode_activated');
    };

    window.addEventListener('offline', trackOfflineEvent);
  }

  static trackServiceWorker() {
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.ready.then(registration => {
        this.trackEvent('pwa', 'sw_ready', 'service_worker_activated');
      });
    }
  }

  static trackPushNotification() {
    if ('PushManager' in window) {
      this.trackEvent('pwa', 'push_support', 'push_notifications_supported');
    }
  }

  static initialize() {
    this.trackAppInstall();
    this.trackOfflineUsage();
    this.trackServiceWorker();
    this.trackPushNotification();
    
    // 性能监控
    this.trackPerformance();
  }

  static trackPerformance() {
    const trackPerfEntry = (entry) => {
      this.trackEvent('performance', entry.name, entry.entryType, entry.duration);
    };

    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        list.getEntries().forEach(trackPerfEntry);
      });

      observer.observe({ entryTypes: ['navigation', 'resource', 'paint'] });
    }
  }
}

// 初始化 PWA 分析
PWAAnalytics.initialize();
```
