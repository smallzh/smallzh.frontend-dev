# React Hooks 核心指南

## 0x01 什么是 Hooks？

Hooks 是 React 16.8 引入的特性，允许在函数组件中使用状态和其他 React 特性。Hooks 使得组件逻辑可复用，让函数组件具备类组件的能力。

## 0x02 useState —— 状态管理

`useState` 是最常用的 Hook，用于在函数组件中添加状态。

### 基本用法

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### 函数式更新

当新状态依赖旧状态时，使用函数式更新：

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  
  // 使用函数式更新，基于旧值计算新值
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### 多个状态变量

```tsx
function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  
  return (
    <form>
      <input 
        value={name} 
        onChange={e => setName(e.target.value)} 
        placeholder="Name"
      />
      <input 
        value={email} 
        onChange={e => setEmail(e.target.value)} 
        placeholder="Email"
      />
      <input 
        type="number"
        value={age} 
        onChange={e => setAge(Number(e.target.value))} 
        placeholder="Age"
      />
    </form>
  );
}
```

### 对象状态

```tsx
function UserProfile() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  const updateName = (name) => {
    setUser(prev => ({ ...prev, name }));
  };
  
  const updateEmail = (email) => {
    setUser(prev => ({ ...prev, email }));
  };
  
  return (
    <div>
      <p>Name: {user.name}</p>
      <p>Email: {user.email}</p>
      <p>Age: {user.age}</p>
    </div>
  );
}
```

### 惰性初始化

当状态初始值计算成本较高时，使用函数进行惰性初始化：

```tsx
function ExpensiveComponent() {
  // 只在首次渲染时计算初始值
  const [data, setData] = useState(() => {
    const initialData = computeExpensiveValue();
    return initialData;
  });
  
  // 或者直接传入函数
  const [config, setConfig] = useState(() => loadConfig());
}
```

## 0x03 useEffect —— 副作用处理

`useEffect` 用于处理副作用，如数据获取、订阅、手动 DOM 操作等。

### 基本用法

```tsx
import { useState, useEffect } from 'react';

function DataFetcher() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // 组件挂载后执行
    fetchData()
      .then(result => {
        setData(result);
        setLoading(false);
      });
  }, []); // 空依赖数组表示只在挂载时执行
  
  if (loading) return <p>Loading...</p>;
  return <div>{data}</div>;
}
```

### 依赖项

```tsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  // 当 userId 变化时重新执行
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]); // 依赖 userId
  
  return <div>{user?.name}</div>;
}
```

### 清理副作用

```tsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    // 返回清理函数
    return () => {
      clearInterval(interval);
    };
  }, []); // 空依赖，只在挂载时设置定时器
  
  return <p>Seconds: {seconds}</p>;
}
```

### 订阅模式

```tsx
function WindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    
    window.addEventListener('resize', handleResize);
    
    // 清理函数 - 移除订阅
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return <p>Window width: {width}</p>;
}
```

### 条件执行

```tsx
function Search({ query }) {
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    if (!query) {
      setResults([]);
      return;
    }
    
    const searchResults = performSearch(query);
    setResults(searchResults);
  }, [query]); // 当 query 变化时执行
  
  return (
    <ul>
      {results.map(r => <li key={r.id}>{r.text}</li>)}
    </ul>
  );
}
```

## 0x04 useContext —— 跨组件通信

`useContext` 用于在组件树中共享数据，避免层层传递 props。

### 创建 Context

```tsx
// ThemeContext.js
import { createContext } from 'react';

export const ThemeContext = createContext('light');
export const LanguageContext = createContext('zh');
```

### 使用 Context

```tsx
import { useContext } from 'react';
import { ThemeContext, LanguageContext } from './ThemeContext';

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <LanguageContext.Provider value="en">
        <Header />
      </LanguageContext.Provider>
    </ThemeContext.Provider>
  );
}

function Header() {
  const theme = useContext(ThemeContext);
  const language = useContext(LanguageContext);
  
  return (
    <header className={theme}>
      <p>Language: {language}</p>
    </header>
  );
}
```

### 优化 Context 性能

```tsx
import { useCallback, useMemo } from 'react';

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const login = useCallback((credentials) => {
    // 登录逻辑
    authenticate(credentials).then(setUser);
  }, []);
  
  const logout = useCallback(() => {
    setUser(null);
  }, []);
  
  // 使用 useMemo 缓存 context value，避免不必要的重渲染
  const contextValue = useMemo(() => ({
    user,
    login,
    logout
  }), [user, login, logout]);
  
  return (
    <AuthContext.Provider value={contextValue}>
      {children}
    </AuthContext.Provider>
  );
}
```

## 0x05 useRef —— 引用与可变值

`useRef` 用于访问 DOM 元素或存储可变值，且不会触发重新渲染。

### 访问 DOM

```tsx
import { useRef } from 'react';

function TextInput() {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  const clearInput = () => {
    if (inputRef.current) {
      inputRef.current.value = '';
    }
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus</button>
      <button onClick={clearInput}>Clear</button>
    </div>
  );
}
```

### 存储可变值

```tsx
function Timer() {
  const [count, setCount] = useState(0);
  const countRef = useRef(0); // 可变值，不触发渲染
  
  const increment = () => {
    countRef.current += 1;
    console.log('countRef:', countRef.current);
    setCount(c => c + 1);
  };
  
  return (
    <div>
      <p>State: {count}</p>
      <p>Ref: {countRef.current}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

### 跟踪前一个状态

```tsx
function PreviousValue() {
  const [value, setValue] = useState(0);
  const prevValueRef = useRef();
  
  useEffect(() => {
    prevValueRef.current = value;
  }, [value]);
  
  return (
    <div>
      <p>Current: {value}</p>
      <p>Previous: {prevValueRef.current}</p>
      <button onClick={() => setValue(v => v + 1)}>Increment</button>
    </div>
  );
}
```

## 0x06 useMemo —— 值缓存

`useMemo` 用于缓存计算结果，避免不必要的重复计算。

### 基本用法

```tsx
import { useMemo } from 'react';

function ExpensiveList({ items, filter }) {
  // 只有 items 或 filter 变化时才重新计算
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.includes(filter)
    );
  }, [items, filter]);
  
  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### 对象优化

```tsx
function Component({ user, theme }) {
  // 避免每次渲染都创建新对象
  const userInfo = useMemo(() => ({
    name: user.name,
    email: user.email,
    avatar: user.avatar
  }), [user.name, user.email, user.avatar]);
  
  const style = useMemo(() => ({
    color: theme === 'dark' ? '#fff' : '#000',
    backgroundColor: theme === 'dark' ? '#333' : '#fff'
  }), [theme]);
  
  return <div style={style}>{userInfo.name}</div>;
}
```

## 0x07 useCallback —— 函数缓存

`useCallback` 用于缓存函数，避免函数重新创建。

### 基本用法

```tsx
import { useCallback, memo } from 'react';

const List = memo(function List({ items, onItemClick }) {
  console.log('List rendered');
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
});

function App() {
  const [items] = useState([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' }
  ]);
  
  // 缓存函数，避免子组件不必要地重渲染
  const handleClick = useCallback((id) => {
    console.log('Clicked:', id);
  }, []);
  
  return <List items={items} onItemClick={handleClick} />;
}
```

### 带依赖的回调

```tsx
function App() {
  const [count, setCount] = useState(0);
  const [multiplier, setMultiplier] = useState(1);
  
  // 当 multiplier 变化时重新创建
  const multiply = useCallback((value) => {
    return value * multiplier;
  }, [multiplier]);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <button onClick={() => setMultiplier(m => m + 1)}>
        Increase Multiplier
      </button>
      <p>Result: {multiply(10)}</p>
    </div>
  );
}
```

## 0x08 useReducer —— 复杂状态逻辑

`useReducer` 用于管理复杂的状态逻辑，类似于 Redux 的模式。

### 基本用法

```tsx
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>
        +1
      </button>
      <button onClick={() => dispatch({ type: 'decrement' })}>
        -1
      </button>
      <button onClick={() => dispatch({ type: 'reset' })}>
        Reset
      </button>
    </div>
  );
}
```

### 复杂状态示例：待办事项

```tsx
import { useReducer, createContext, useContext } from 'react';

// State 类型
interface Task {
  id: number;
  text: string;
  completed: boolean;
}

interface State {
  tasks: Task[];
  filter: 'all' | 'active' | 'completed';
}

// Action 定义
type Action =
  | { type: 'add'; text: string }
  | { type: 'toggle'; id: number }
  | { type: 'delete'; id: number }
  | { type: 'setFilter'; filter: State['filter'] };

// Reducer
function tasksReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'add':
      return {
        ...state,
        tasks: [
          ...state.tasks,
          {
            id: Date.now(),
            text: action.text,
            completed: false
          }
        ]
      };
    case 'toggle':
      return {
        ...state,
        tasks: state.tasks.map(task =>
          task.id === action.id
            ? { ...task, completed: !task.completed }
            : task
        )
      };
    case 'delete':
      return {
        ...state,
        tasks: state.tasks.filter(task => task.id !== action.id)
      };
    case 'setFilter':
      return { ...state, filter: action.filter };
    default:
      return state;
  }
}

// Context
const TasksContext = createContext<State | null>(null);
const TasksDispatchContext = createContext<React.Dispatch<Action> | null>(null);

// Provider
export function TasksProvider({ children }) {
  const [state, dispatch] = useReducer(tasksReducer, {
    tasks: [],
    filter: 'all'
  });
  
  return (
    <TasksContext.Provider value={state}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// 自定义 Hooks
export function useTasks() {
  const context = useContext(TasksContext);
  if (!context) {
    throw new Error('useTasks must be used within TasksProvider');
  }
  return context;
}

export function useTasksDispatch() {
  const context = useContext(TasksDispatchContext);
  if (!context) {
    throw new Error('useTasksDispatch must be used within TasksProvider');
  }
  return context;
}

// 组件中使用
function TaskList() {
  const { tasks, filter } = useTasks();
  const dispatch = useTasksDispatch();
  
  const filteredTasks = tasks.filter(task => {
    if (filter === 'active') return !task.completed;
    if (filter === 'completed') return task.completed;
    return true;
  });
  
  return (
    <ul>
      {filteredTasks.map(task => (
        <li key={task.id}>
          <input
            type="checkbox"
            checked={task.completed}
            onChange={() => dispatch({ type: 'toggle', id: task.id })}
          />
          <span style={{
              textDecoration: task.completed ? 'line-through' : 'none'
            }}>
            {task.text}
          </span>
          <button onClick={() => dispatch({ type: 'delete', id: task.id })}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

### 异步操作

```tsx
import { useReducer } from 'react';

interface State<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

type AsyncAction<T> =
  | { type: 'pending' }
  | { type: 'fulfilled'; payload: T }
  | { type: 'rejected'; error: string };

function asyncReducer<T>(state: State<T>, action: AsyncAction<T>): State<T> {
  switch (action.type) {
    case 'pending':
      return { data: null, loading: true, error: null };
    case 'fulfilled':
      return { data: action.payload, loading: false, error: null };
    case 'rejected':
      return { data: null, loading: false, error: action.error };
    default:
      return state;
  }
}

function useAsync<T>(asyncFn: () => Promise<T>) {
  const [state, dispatch] = useReducer(asyncReducer<T>, {
    data: null,
    loading: false,
    error: null
  });
  
  const execute = async () => {
    dispatch({ type: 'pending' });
    try {
      const data = await asyncFn();
      dispatch({ type: 'fulfilled', payload: data });
    } catch (error) {
      dispatch({ type: 'rejected', error: error.message });
    }
  };
  
  return { ...state, execute };
}
```

## 0x09 自定义 Hook

自定义 Hook 是复用状态逻辑的方式，本质是一个使用其他 Hook 的函数。

### 基本结构

```tsx
import { useState, useEffect } from 'react';

// 自定义 Hook：以 use 开头的函数
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// 使用自定义 Hook
function App() {
  const { width, height } = useWindowSize();
  
  return (
    <div>
      <p>Window size: {width} x {height}</p>
    </div>
  );
}
```

### 复用状态逻辑

```tsx
// useLocalStorage.ts
import { useState, useEffect } from 'react';

function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setStoredValue] as const;
}

// 使用
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [name, setName] = useLocalStorage('name', '');
  
  return (
    <div className={theme}>
      <input value={name} onChange={e => setName(e.target.value)} />
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}
```

### 条件判断 Hook

```tsx
// useOnClickOutside.ts
import { useEffect, RefObject } from 'react';

function useOnClickOutside<T extends HTMLElement>(
  ref: RefObject<T>,
  handler: (event: MouseEvent | TouchEvent) => void
) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) {
        return;
      }
      handler(event);
    };
    
    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}

// 使用
function Modal({ onClose }: { onClose: () => void }) {
  const modalRef = useRef<HTMLDivElement>(null);
  
  useOnClickOutside(modalRef, onClose);
  
  return (
    <div ref={modalRef} className="modal">
      <p>Modal Content</p>
    </div>
  );
}
```

## 0x10 Hooks 规则

### 规则一：只在顶层调用 Hook

```tsx
// ❌ 错误：在条件语句中调用 Hook
function MyComponent() {
  const [value, setValue] = useState(0);
  
  if (condition) {
    const [data, setData] = useState(null); // 错误！
  }
  
  // ...
}

// ✅ 正确：将条件放在 Hook 内部
function MyComponent() {
  const [value, setValue] = useState(0);
  const [data, setData] = useState(null);
  
  if (condition) {
    // 在这里使用 data
  }
  
  // ...
}
```

### 规则二：只在函数组件或自定义 Hook 中调用

```tsx
// ❌ 错误：普通函数中调用 Hook
function regularFunction() {
  const [value, setValue] = useState(0); // 错误！
}

// ✅ 正确：在组件或自定义 Hook 中调用
function MyComponent() {
  const [value, setValue] = useState(0);
  return <div>{value}</div>;
}

function useCustomHook() {
  const [value, setValue] = useState(0);
  return value;
}
```

### 使用 ESLint 插件

```bash
# 安装
npm install eslint-plugin-react-hooks --save-dev

# .eslintrc.js
module.exports = {
  plugins: ['react-hooks'],
  rules: {
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn'
  }
};
```

## 0x11 常见问题

### 为什么 useEffect 会执行两次？

在开发模式下，React 会故意双重调用组件函数以检测副作用中的错误。这仅在开发环境出现，生产环境中不会发生。

### 如何正确设置依赖数组？

```tsx
function Component({ id }) {
  const [data, setData] = useState(null);
  
  // ❌ 错误：忘记添加依赖
  useEffect(() => {
    fetchData(id).then(setData);
  }, []); // 缺少 id
  
  // ✅ 正确：添加所有依赖
  useEffect(() => {
    fetchData(id).then(setData);
  }, [id]);
  
  // ✅ 正确：使用函数避免依赖
  useEffect(() => {
    fetchData(id).then(setData);
  }, [fetchData]); // 或者确保 fetchData 使用 useCallback
}
```

### useState 和 useReducer 怎么选择？

- **useState**：适合简单的局部状态
- **useReducer**：适合复杂的状态逻辑，或多个子值相关的状态

```tsx
// 使用 useState
const [state, setState] = useState({ a: 1, b: 2 });
setState(prev => ({ ...prev, b: prev.b + 1 }));

// 使用 useReducer
const [state, dispatch] = useReducer(reducer, { a: 1, b: 2 });
dispatch({ type: 'increment_b' });
```

## 参考

- [React Hooks 官方文档](https://react.dev/reference/react)
- [useState API](https://react.dev/reference/react/useState)
- [useEffect API](https://react.dev/reference/react/useEffect)
- [useContext API](https://react.dev/reference/react/useContext)
- [useReducer API](https://react.dev/reference/react/useReducer)
