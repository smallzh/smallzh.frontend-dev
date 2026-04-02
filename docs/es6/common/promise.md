# Promise

## 概述

Promise 是 ES6 引入的异步编程解决方案，用于处理异步操作，避免回调地狱。

## 基本概念

### 什么是 Promise

Promise 是一个表示异步操作最终完成或失败的对象。它有三种状态：
- **pending**（进行中）：初始状态
- **fulfilled**（已完成）：操作成功完成
- **rejected**（已拒绝）：操作失败

### 创建 Promise

```javascript
const promise = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('操作成功');
    } else {
      reject(new Error('操作失败'));
    }
  }, 1000);
});
```

## Promise 方法

### then()

处理 Promise 成功的情况：

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve('成功'), 1000);
});

promise.then(result => {
  console.log(result); // '成功'
});

// 链式调用
promise
  .then(result => {
    console.log(result);
    return '下一步';
  })
  .then(result2 => {
    console.log(result2); // '下一步'
  });
```

### catch()

处理 Promise 失败的情况：

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error('失败')), 1000);
});

promise.catch(error => {
  console.log(error.message); // '失败'
});

// 链式调用中的错误处理
promise
  .then(result => {
    console.log(result);
  })
  .catch(error => {
    console.log('捕获错误:', error.message);
  });
```

### finally()

无论成功或失败都会执行：

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve('成功'), 1000);
});

promise
  .then(result => {
    console.log(result);
  })
  .catch(error => {
    console.log(error);
  })
  .finally(() => {
    console.log('清理工作');
  });
```

## Promise 静态方法

### Promise.resolve()

创建一个已成功的 Promise：

```javascript
Promise.resolve('成功')
  .then(result => console.log(result)); // '成功'

// 将 thenable 对象转换为 Promise
const thenable = {
  then(resolve) {
    resolve('thenable');
  }
};

Promise.resolve(thenable)
  .then(result => console.log(result)); // 'thenable'
```

### Promise.reject()

创建一个已失败的 Promise：

```javascript
Promise.reject(new Error('失败'))
  .catch(error => console.log(error.message)); // '失败'
```

### Promise.all()

等待所有 Promise 都成功：

```javascript
const promise1 = Promise.resolve(1);
const promise2 = Promise.resolve(2);
const promise3 = Promise.resolve(3);

Promise.all([promise1, promise2, promise3])
  .then(results => {
    console.log(results); // [1, 2, 3]
  });

// 如果有一个失败，整个 Promise.all 就会失败
const promise4 = Promise.reject(new Error('失败'));

Promise.all([promise1, promise4, promise3])
  .catch(error => {
    console.log(error.message); // '失败'
  });
```

### Promise.race()

返回第一个完成的 Promise：

```javascript
const promise1 = new Promise(resolve => setTimeout(() => resolve('1'), 1000));
const promise2 = new Promise(resolve => setTimeout(() => resolve('2'), 500));
const promise3 = new Promise(resolve => setTimeout(() => resolve('3'), 200));

Promise.race([promise1, promise2, promise3])
  .then(result => {
    console.log(result); // '3'（最快的）
  });

// 超时处理
function timeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('超时')), ms)
    )
  ]);
}

timeout(fetch('/api/data'), 5000)
  .then(response => console.log(response))
  .catch(error => console.log(error.message)); // '超时'
```

### Promise.allSettled()

等待所有 Promise 都完成（无论成功或失败）：

```javascript
const promise1 = Promise.resolve('成功1');
const promise2 = Promise.reject(new Error('失败'));
const promise3 = Promise.resolve('成功3');

Promise.allSettled([promise1, promise2, promise3])
  .then(results => {
    results.forEach(result => {
      if (result.status === 'fulfilled') {
        console.log('成功:', result.value);
      } else {
        console.log('失败:', result.reason.message);
      }
    });
  });
```

### Promise.any()

返回第一个成功的 Promise：

```javascript
const promise1 = Promise.reject(new Error('失败1'));
const promise2 = Promise.resolve('成功2');
const promise3 = Promise.resolve('成功3');

Promise.any([promise1, promise2, promise3])
  .then(result => {
    console.log(result); // '成功2'（第一个成功的）
  });

// 如果全部失败，抛出 AggregateError
Promise.all([
  Promise.reject(new Error('失败1')),
  Promise.reject(new Error('失败2'))
]).catch(error => {
  console.log(error.constructor.name); // 'AggregateError'
  console.log(error.errors); // [Error: 失败1, Error: 失败2]
});
```

## async/await

### 基本用法

```javascript
async function fetchData() {
  const result = await fetch('/api/data');
  const data = await result.json();
  return data;
}

// 等同于
function fetchData2() {
  return fetch('/api/data')
    .then(result => result.json());
}
```

### 错误处理

```javascript
async function fetchData() {
  try {
    const result = await fetch('/api/data');
    const data = await result.json();
    return data;
  } catch (error) {
    console.error('获取数据失败:', error);
    throw error;
  }
}

// 使用 catch
async function fetchData2() {
  const result = await fetch('/api/data').catch(error => {
    console.error('获取数据失败:', error);
    return null;
  });
  
  if (!result) return null;
  
  return result.json();
}
```

### 并行执行

```javascript
// 顺序执行（慢）
async function sequential() {
  const result1 = await fetch('/api/users');
  const result2 = await fetch('/api/posts');
  return [await result1.json(), await result2.json()];
}

// 并行执行（快）
async function parallel() {
  const [users, posts] = await Promise.all([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/posts').then(r => r.json())
  ]);
  return [users, posts];
}

// 错误处理的并行执行
async function parallelWithErrorHandling() {
  const results = await Promise.allSettled([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/posts').then(r => r.json())
  ]);
  
  return results.map(result => 
    result.status === 'fulfilled' ? result.value : null
  );
}
```

## 实际应用示例

### 1. API 请求封装

```javascript
class ApiClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    
    try {
      const response = await fetch(url, {
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        },
        ...options
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('API 请求失败:', error);
      throw error;
    }
  }
  
  async get(endpoint) {
    return this.request(endpoint);
  }
  
  async post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
}

// 使用
const api = new ApiClient('https://api.example.com');

async function getUsers() {
  try {
    const users = await api.get('/users');
    return users;
  } catch (error) {
    console.error('获取用户失败:', error);
    return [];
  }
}
```

### 2. 重试机制

```javascript
async function retry(fn, maxAttempts = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) {
        throw error;
      }
      
      console.log(`第 ${attempt} 次尝试失败，${delay}ms 后重试...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// 使用
async function fetchData() {
  return retry(async () => {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  }, 3, 1000);
}
```

### 3. 并发控制

```javascript
async function parallelLimit(tasks, limit) {
  const results = [];
  const executing = [];
  
  for (const task of tasks) {
    const promise = task().then(result => {
      executing.splice(executing.indexOf(promise), 1);
      return result;
    });
    
    results.push(promise);
    executing.push(promise);
    
    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }
  
  return Promise.all(results);
}

// 使用
const urls = ['url1', 'url2', 'url3', 'url4', 'url5'];

const tasks = urls.map(url => () => 
  fetch(url).then(r => r.json())
);

parallelLimit(tasks, 2).then(results => {
  console.log('所有请求完成:', results);
});
```

### 4. 缓存机制

```javascript
function createCache(ttl = 60000) {
  const cache = new Map();
  
  return {
    async get(key, fetcher) {
      const item = cache.get(key);
      
      if (item && Date.now() - item.timestamp < ttl) {
        return item.value;
      }
      
      const value = await fetcher();
      cache.set(key, { value, timestamp: Date.now() });
      return value;
    },
    
    clear() {
      cache.clear();
    }
  };
}

// 使用
const cache = createCache(5000); // 5秒缓存

async function getUser(id) {
  return cache.get(`user:${id}`, () => 
    fetch(`/api/users/${id}`).then(r => r.json())
  );
}
```

## 注意事项

### 1. 避免回调地狱

```javascript
// 不推荐
fetch('/api/user')
  .then(user => {
    fetch(`/api/posts/${user.id}`)
      .then(posts => {
        fetch(`/api/comments/${posts[0].id}`)
          .then(comments => {
            console.log(comments);
          });
      });
  });

// 推荐
async function getData() {
  const user = await fetch('/api/user').then(r => r.json());
  const posts = await fetch(`/api/posts/${user.id}`).then(r => r.json());
  const comments = await fetch(`/api/comments/${posts[0].id}`).then(r => r.json());
  return comments;
}
```

### 2. 错误处理

```javascript
// 不推荐
async function badExample() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    // 静默失败
  }
}

// 推荐
async function goodExample() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    console.error('获取数据失败:', error);
    throw error; // 重新抛出，让调用者处理
  }
}
```

### 3. 性能优化

```javascript
// 不推荐：顺序执行
async function sequential() {
  const users = await fetchUsers();
  const posts = await fetchPosts();
  return { users, posts };
}

// 推荐：并行执行
async function parallel() {
  const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts()
  ]);
  return { users, posts };
}
```

## 最佳实践

1. **使用 async/await**：比 then 链更清晰
2. **并行执行独立操作**：使用 Promise.all
3. **正确处理错误**：使用 try/catch 或 catch()
4. **避免回调地狱**：扁平化异步代码
5. **使用 Promise.allSettled()**：处理部分失败的情况

```javascript
// 好的做法
async function fetchUserData(userId) {
  try {
    const [user, posts, followers] = await Promise.all([
      fetchUser(userId),
      fetchPosts(userId),
      fetchFollowers(userId)
    ]);
    
    return { user, posts, followers };
  } catch (error) {
    console.error('获取用户数据失败:', error);
    throw error;
  }
}
```

## 参考

- [MDN Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
