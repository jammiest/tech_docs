# 异步编程模式

现代前端开发中，异步编程是处理I/O操作、网络请求和用户交互的核心技术。本文将全面介绍JavaScript中的异步编程模式及其演进过程。

## 异步编程演进史

### 1. 回调函数 (Callback)

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback('Data received');
  }, 1000);
}

fetchData((data) => {
  console.log(data); // "Data received"
});
```

**问题：回调地狱(Callback Hell)**

```javascript
getUser(userId, (user) => {
  getOrders(user.id, (orders) => {
    getOrderDetails(orders[0].id, (details) => {
      renderUI(details, () => {
        // 更多嵌套...
      });
    });
  });
});
```

### 2. Promise

```javascript
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('Data received');
      // 或 reject(new Error('Failed'));
    }, 1000);
  });
}

fetchData()
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

**链式调用解决回调地狱**

```javascript
getUser(userId)
  .then(user => getOrders(user.id))
  .then(orders => getOrderDetails(orders[0].id))
  .then(details => renderUI(details))
  .catch(error => console.error(error));
```

### 3. Async/Await

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// 使用
(async () => {
  const result = await fetchData();
  console.log(result);
})();
```

## 高级异步模式

### 1. 并行执行

```javascript
// Promise.all - 全部成功或失败
const [user, orders] = await Promise.all([
  getUser(userId),
  getOrders(userId)
]);

// Promise.allSettled - 等待所有完成
const results = await Promise.allSettled([
  fetch('/api/user'),
  fetch('/api/orders')
]);

// Promise.race - 第一个完成（无论成功失败）
const firstResponse = await Promise.race([
  fetch('/fast-api'),
  fetch('/slow-api')
]);
```

### 2. 错误处理策略

```javascript
// 1. 集中式错误处理
async function fetchWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url);
    } catch (err) {
      if (i === retries - 1) throw err;
      await new Promise(res => setTimeout(res, 1000 * (i + 1)));
    }
  }
}

// 2. 错误边界模式
class ErrorBoundary {
  static async wrap(promise) {
    try {
      return await promise;
    } catch (error) {
      return { error };
    }
  }
}
```

### 3. 取消异步操作

```javascript
// 使用AbortController
const controller = new AbortController();
const signal = controller.signal;

fetch('/api/data', { signal })
  .then(response => response.json())
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('Fetch aborted');
    }
  });

// 取消请求
controller.abort();

// 自定义可取消Promise
function makeCancelable(promise) {
  let isCanceled = false;
  
  const wrappedPromise = new Promise((resolve, reject) => {
    promise.then(
      val => isCanceled ? reject({isCanceled: true}) : resolve(val),
      error => isCanceled ? reject({isCanceled: true}) : reject(error)
    );
  });

  return {
    promise: wrappedPromise,
    cancel() {
      isCanceled = true;
    }
  };
}
```

## 现代异步实践

### 1. 前端状态管理集成

```javascript
// 在Redux中处理异步
const fetchUser = (userId) => async (dispatch) => {
  dispatch({ type: 'USER_FETCH_START' });
  try {
    const user = await api.getUser(userId);
    dispatch({ type: 'USER_FETCH_SUCCESS', payload: user });
  } catch (error) {
    dispatch({ type: 'USER_FETCH_FAILURE', error });
  }
};

// 在Vue Composition API中
const useUser = (userId) => {
  const user = ref(null);
  const error = ref(null);
  const loading = ref(false);

  const fetchUser = async () => {
    loading.value = true;
    try {
      user.value = await getUser(userId);
    } catch (err) {
      error.value = err;
    } finally {
      loading.value = false;
    }
  };

  return { user, error, loading, fetchUser };
};
```

### 2. Web Worker多线程

```javascript
// 主线程
const worker = new Worker('worker.js');
worker.postMessage({ type: 'CALCULATE', data: largeDataSet });

worker.onmessage = (event) => {
  console.log('Result:', event.data.result);
};

// worker.js
self.onmessage = async (event) => {
  if (event.data.type === 'CALCULATE') {
    const result = await heavyCalculation(event.data.data);
    self.postMessage({ result });
  }
};
```

### 3. 流式数据处理

```javascript
// 使用异步迭代器处理流数据
async function processStream(stream) {
  for await (const chunk of stream) {
    console.log('Processing chunk:', chunk);
    // 处理数据块
  }
}

// Fetch API的流式响应
const response = await fetch('/large-data');
const reader = response.body.getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  console.log('Received chunk:', value);
}
```

## 性能优化策略

1. **批量请求**：合并多个小请求
   ```javascript
   async function batchRequests(requests) {
     const batch = [];
     while (requests.length > 0) {
       batch.push(requests.splice(0, 5)); // 每批5个
     }
     
     for (const group of batch) {
       await Promise.all(group.map(req => req()));
     }
   }
   ```

2. **请求缓存**：
   ```javascript
   const cache = new Map();
   
   async function cachedFetch(url) {
     if (cache.has(url)) {
       return cache.get(url);
     }
     
     const response = await fetch(url);
     const data = await response.json();
     cache.set(url, data);
     return data;
   }
   ```

3. **懒加载与预加载**：
   ```javascript
   // 懒加载组件
   const LazyComponent = React.lazy(() => import('./HeavyComponent'));
   
   // 预加载资源
   function prefetchResource(url) {
     const link = document.createElement('link');
     link.rel = 'prefetch';
     link.href = url;
     document.head.appendChild(link);
   }
   ```

## 错误监控与调试

1. **全局错误捕获**：
   ```javascript
   // 未捕获的Promise错误
   window.addEventListener('unhandledrejection', (event) => {
     console.error('Unhandled rejection:', event.reason);
     // 上报错误
   });
   
   // Async/Await错误跟踪
   async function trackErrors(fn) {
     try {
       return await fn();
     } catch (error) {
       console.error('Async error:', error);
       // 上报错误
       throw error;
     }
   }
   ```

2. **性能监控**：
   ```javascript
   async function measurePerformance(fn) {
     const start = performance.now();
     try {
       return await fn();
     } finally {
       const duration = performance.now() - start;
       console.log(`Function took ${duration.toFixed(2)}ms`);
     }
   }
   ```
