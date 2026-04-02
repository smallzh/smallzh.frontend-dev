# 模块（import/export）

## 概述

ES6 模块系统提供了代码组织和复用的机制，使用 `import` 和 `export` 关键字。

## 导出（export）

### 命名导出

#### 逐个导出

```javascript
// math.js
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export class Calculator {
  // ...
}
```

#### 统一导出

```javascript
// math.js
const PI = 3.14159;

function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

class Calculator {
  // ...
}

export { PI, add, subtract, Calculator };
```

#### 重命名导出

```javascript
// utils.js
function internalHelper() {
  // ...
}

function anotherHelper() {
  // ...
}

export { 
  internalHelper as helper,
  anotherHelper as utils 
};
```

### 默认导出

```javascript
// user.js
export default class User {
  constructor(name) {
    this.name = name;
  }
}

// 或者
class User {
  constructor(name) {
    this.name = name;
  }
}

export default User;
```

### 混合导出

```javascript
// api.js
export const API_URL = 'https://api.example.com';

export function fetchData(endpoint) {
  return fetch(`${API_URL}${endpoint}`);
}

export default class ApiClient {
  constructor(baseURL = API_URL) {
    this.baseURL = baseURL;
  }
  
  async get(endpoint) {
    return fetchData(endpoint);
  }
}
```

## 导入（import）

### 导入命名导出

```javascript
// 导入特定导出
import { add, subtract } from './math.js';

console.log(add(5, 3)); // 8
console.log(subtract(5, 3)); // 2

// 导入所有命名导出
import * as MathUtils from './math.js';

console.log(MathUtils.add(5, 3)); // 8
console.log(MathUtils.PI); // 3.14159
```

### 导入默认导出

```javascript
// 导入默认导出（可以任意命名）
import User from './user.js';

const user = new User('John');

// 混合导入
import ApiClient, { API_URL, fetchData } from './api.js';

const client = new ApiClient();
console.log(API_URL); // 'https://api.example.com'
```

### 重命名导入

```javascript
import { add as addition, subtract as subtraction } from './math.js';

console.log(addition(5, 3)); // 8
console.log(subtraction(5, 3)); // 2
```

### 动态导入

```javascript
// 条件导入
async function loadModule(condition) {
  if (condition) {
    const module = await import('./feature-a.js');
    return module.default;
  } else {
    const module = await import('./feature-b.js');
    return module.default;
  }
}

// 按需加载
button.addEventListener('click', async () => {
  const { default: HeavyComponent } = await import('./HeavyComponent.js');
  // 渲染组件
});

// 代码分割
const routes = {
  home: () => import('./pages/Home.js'),
  about: () => import('./pages/About.js'),
  contact: () => import('./pages/Contact.js')
};

async function loadPage(pageName) {
  const { default: Page } = await routes[pageName]();
  return Page;
}
```

## 重新导出

### 基本重新导出

```javascript
// utils/index.js
export { add, subtract } from './math.js';
export { formatDate, parseDate } from './date.js';
export { default as User } from './user.js';
```

### 重命名重新导出

```javascript
// utils/index.js
export { add as addition } from './math.js';
export { formatDate as format } from './date.js';
```

### 重新导出所有

```javascript
// utils/index.js
export * from './math.js';
export * from './date.js';
export * from './user.js';
```

## 实际应用示例

### 1. 工具函数库

```javascript
// utils/string.js
export function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

export function truncate(str, length, suffix = '...') {
  if (str.length <= length) return str;
  return str.slice(0, length - suffix.length) + suffix;
}

export function slugify(str) {
  return str
    .toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/[\s_-]+/g, '-')
    .replace(/^-+|-+$/g, '');
}

// utils/array.js
export function unique(arr) {
  return [...new Set(arr)];
}

export function chunk(arr, size) {
  const chunks = [];
  for (let i = 0; i < arr.length; i += size) {
    chunks.push(arr.slice(i, i + size));
  }
  return chunks;
}

export function shuffle(arr) {
  return arr.sort(() => Math.random() - 0.5);
}

// utils/index.js
export * from './string.js';
export * from './array.js';

// 使用
import { capitalize, unique, chunk } from './utils/index.js';

console.log(capitalize('hello')); // 'Hello'
console.log(unique([1, 2, 2, 3, 3])); // [1, 2, 3]
console.log(chunk([1, 2, 3, 4, 5], 2)); // [[1, 2], [3, 4], [5]]
```

### 2. 组件库

```javascript
// components/Button.js
export default function Button({ 
  children, 
  onClick, 
  variant = 'primary',
  disabled = false 
}) {
  return `
    <button 
      class="btn btn-${variant}" 
      onclick="${onClick}"
      ${disabled ? 'disabled' : ''}
    >
      ${children}
    </button>
  `;
}

export const ButtonVariants = ['primary', 'secondary', 'danger'];

// components/Input.js
export default function Input({
  type = 'text',
  placeholder = '',
  value = '',
  onChange
}) {
  return `
    <input 
      type="${type}"
      placeholder="${placeholder}"
      value="${value}"
      oninput="${onChange}"
    />
  `;
}

// components/index.js
export { default as Button, ButtonVariants } from './Button.js';
export { default as Input } from './Input.js';

// 使用
import { Button, Input, ButtonVariants } from './components/index.js';

const app = `
  ${Button({ 
    children: 'Click me', 
    onClick: 'handleClick()',
    variant: 'primary'
  })}
  ${Input({ 
    placeholder: 'Enter text',
    onChange: 'handleChange(event)'
  })}
`;
```

### 3. API 客户端

```javascript
// api/config.js
export const API_BASE_URL = 'https://api.example.com';
export const API_TIMEOUT = 5000;

export const DEFAULT_HEADERS = {
  'Content-Type': 'application/json'
};

// api/endpoints.js
export const ENDPOINTS = {
  USERS: '/users',
  POSTS: '/posts',
  COMMENTS: '/comments'
};

// api/client.js
import { API_BASE_URL, API_TIMEOUT, DEFAULT_HEADERS } from './config.js';

export class ApiClient {
  constructor(baseURL = API_BASE_URL) {
    this.baseURL = baseURL;
  }
  
  async request(endpoint, options = {}) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), API_TIMEOUT);
    
    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, {
        ...options,
        headers: { ...DEFAULT_HEADERS, ...options.headers },
        signal: controller.signal
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      return await response.json();
    } finally {
      clearTimeout(timeoutId);
    }
  }
  
  get(endpoint) {
    return this.request(endpoint);
  }
  
  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
}

export default new ApiClient();

// api/index.js
export { ApiClient, default as apiClient } from './client.js';
export { API_BASE_URL, API_TIMEOUT } from './config.js';
export { ENDPOINTS } from './endpoints.js';

// 使用
import { apiClient, ENDPOINTS } from './api/index.js';

async function getUsers() {
  return apiClient.get(ENDPOINTS.USERS);
}
```

### 4. 状态管理

```javascript
// store/actions.js
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';
export const SET_VALUE = 'SET_VALUE';

export const increment = () => ({ type: INCREMENT });
export const decrement = () => ({ type: DECREMENT });
export const setValue = (value) => ({ type: SET_VALUE, payload: value });

// store/reducer.js
import { INCREMENT, DECREMENT, SET_VALUE } from './actions.js';

const initialState = { count: 0 };

export function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return { ...state, count: state.count + 1 };
    case DECREMENT:
      return { ...state, count: state.count - 1 };
    case SET_VALUE:
      return { ...state, count: action.payload };
    default:
      return state;
  }
}

// store/index.js
export { counterReducer } from './reducer.js';
export * from './actions.js';

export function createStore(reducer) {
  let state = reducer(undefined, {});
  const listeners = [];
  
  return {
    getState() {
      return state;
    },
    dispatch(action) {
      state = reducer(state, action);
      listeners.forEach(listener => listener(state));
    },
    subscribe(listener) {
      listeners.push(listener);
      return () => {
        const index = listeners.indexOf(listener);
        listeners.splice(index, 1);
      };
    }
  };
}

// 使用
import { createStore, increment, decrement, counterReducer } from './store/index.js';

const store = createStore(counterReducer);

store.subscribe(state => {
  console.log('State:', state);
});

store.dispatch(increment()); // State: { count: 1 }
store.dispatch(increment()); // State: { count: 2 }
store.dispatch(decrement()); // State: { count: 1 }
```

## 注意事项

### 1. 模块是单例的

```javascript
// counter.js
export let count = 0;

export function increment() {
  count++;
}

// a.js
import { count, increment } from './counter.js';
increment();
console.log(count); // 1

// b.js
import { count, increment } from './counter.js';
console.log(count); // 1（同一个模块实例）
increment();
console.log(count); // 2

// a.js
import { count } from './counter.js';
console.log(count); // 2（值已更新）
```

### 2. 导入是只读的

```javascript
// config.js
export const API_URL = 'https://api.example.com';

// main.js
import { API_URL } from './config.js';
// API_URL = 'https://other.com'; // TypeError: Assignment to constant variable
```

### 3. 循环依赖

```javascript
// a.js
import { b } from './b.js';
export const a = 'a';
export const getB = () => b;

// b.js
import { a } from './a.js';
export const b = 'b';
export const getA = () => a;

// main.js
import { getB } from './a.js';
import { getA } from './b.js';

console.log(getB()); // 'b'
console.log(getA()); // 'a'
```

## 最佳实践

1. **使用命名导出**：便于 tree-shaking
2. **默认导出用于主要功能**：一个模块一个主要导出
3. **使用 index.js 聚合导出**：便于导入
4. **按需导入**：只导入需要的成员
5. **使用动态导入**：代码分割和按需加载

```javascript
// 好的做法
import { useState, useEffect } from 'react';
import { formatDate, parseDate } from './utils/date.js';

// 避免
import * as React from 'react';
import * as utils from './utils/date.js';
```

## 参考

- [MDN import](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import)
- [MDN export](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/export)
