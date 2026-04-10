# React 状态管理

## 0x01 状态管理概述

React 提供了多种状态管理方案，从简单的本地状态到全局状态管理库。理解不同场景下如何选择合适的状态管理方式，是构建可维护 React 应用的关键。

## 0x02 本地状态管理

### useState

适合简单的 UI 状态：

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

### useReducer

适合复杂的状态逻辑：

```tsx
import { useReducer } from 'react';

interface State {
  isLoading: boolean;
  data: any;
  error: string | null;
}

type Action =
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: any }
  | { type: 'FETCH_ERROR'; error: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'FETCH_START':
      return { isLoading: true, data: null, error: null };
    case 'FETCH_SUCCESS':
      return { isLoading: false, data: action.payload, error: null };
    case 'FETCH_ERROR':
      return { isLoading: false, data: null, error: action.error };
    default:
      return state;
  }
}

function DataFetcher() {
  const [state, dispatch] = useReducer(reducer, {
    isLoading: false,
    data: null,
    error: null
  });
  
  const fetchData = async () => {
    dispatch({ type: 'FETCH_START' });
    try {
      const data = await api.getData();
      dispatch({ type: 'FETCH_SUCCESS', payload: data });
    } catch (err) {
      dispatch({ type: 'FETCH_ERROR', error: err.message });
    }
  };
  
  return (
    <div>
      <button onClick={fetchData} disabled={state.isLoading}>
        {state.isLoading ? 'Loading...' : 'Fetch Data'}
      </button>
      {state.error && <p>Error: {state.error}</p>}
      {state.data && <p>Data: {JSON.stringify(state.data)}</p>}
    </div>
  );
}
```

## 0x03 Context API

### 基础 Context

```tsx
import { createContext, useContext, useState } from 'react';

// 创建 Context
interface AuthContextType {
  user: { name: string; email: string } | null;
  login: (credentials: any) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | null>(null);

// Provider 组件
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<{ name: string; email: string } | null>(null);
  
  const login = async (credentials: any) => {
    const user = await api.login(credentials);
    setUser(user);
  };
  
  const logout = () => {
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// 自定义 Hook
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// 使用
function LoginButton() {
  const { user, login, logout } = useAuth();
  
  if (user) {
    return <button onClick={logout}>Logout {user.name}</button>;
  }
  
  return (
    <button onClick={() => login({ username: 'test', password: '123' })}>
      Login
    </button>
  );
}
```

### 分离 Context（优化性能）

```tsx
// 分离 State 和 Dispatch Context
const TasksContext = createContext<Task[] | null>(null);
const TasksDispatchContext = createContext<React.Dispatch<Action> | null>(null);

export function TasksProvider({ children }: { children: React.ReactNode }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// 使用分离的 Context
function TaskList() {
  const tasks = useContext(TasksContext);
  const dispatch = useContext(TasksDispatchContext);
  
  // 只有 tasks 变化时重渲染
  return (
    <ul>
      {tasks?.map(task => (
        <TaskItem 
          key={task.id} 
          task={task}
          onToggle={() => dispatch({ type: 'toggle', id: task.id })}
        />
      ))}
    </ul>
  );
}
```

## 0x04 状态管理模式

### 复合 Context Provider

```tsx
// 完整的状态管理示例
interface AppState {
  user: User | null;
  theme: 'light' | 'dark';
  notifications: Notification[];
}

interface User {
  id: string;
  name: string;
}

interface Notification {
  id: string;
  message: string;
  type: 'info' | 'success' | 'error';
}

type AppAction =
  | { type: 'SET_USER'; user: User | null }
  | { type: 'SET_THEME'; theme: 'light' | 'dark' }
  | { type: 'ADD_NOTIFICATION'; notification: Notification }
  | { type: 'REMOVE_NOTIFICATION'; id: string };

function appReducer(state: AppState, action: AppAction): AppState {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.user };
    case 'SET_THEME':
      return { ...state, theme: action.theme };
    case 'ADD_NOTIFICATION':
      return {
        ...state,
        notifications: [...state.notifications, action.notification]
      };
    case 'REMOVE_NOTIFICATION':
      return {
        ...state,
        notifications: state.notifications.filter(n => n.id !== action.id)
      };
    default:
      return state;
  }
}

// 创建 Provider
const AppStateContext = createContext<AppState | null>(null);
const AppDispatchContext = createContext<React.Dispatch<AppAction> | null>(null);

export function AppProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(appReducer, {
    user: null,
    theme: 'light',
    notifications: []
  });
  
  return (
    <AppStateContext.Provider value={state}>
      <AppDispatchContext.Provider value={dispatch}>
        {children}
      </AppDispatchContext.Provider>
    </AppStateContext.Provider>
  );
}

// 自定义 Hooks
export function useAppState() {
  const context = useContext(AppStateContext);
  if (!context) {
    throw new Error('useAppState must be used within AppProvider');
  }
  return context;
}

export function useAppDispatch() {
  const context = useContext(AppDispatchContext);
  if (!context) {
    throw new Error('useAppDispatch must be used within AppProvider');
  }
  return context;
}

// 使用
function ThemeToggle() {
  const { theme } = useAppState();
  const dispatch = useAppDispatch();
  
  return (
    <button onClick={() => 
      dispatch({ 
        type: 'SET_THEME', 
        theme: theme === 'light' ? 'dark' : 'light' 
      })
    }>
      Toggle Theme
    </button>
  );
}
```

### 原子化状态（Atoms）

类似 Zustand 的原子化状态模式：

```tsx
// atom.ts - 原子化状态
import { createContext, useContext, useState, useCallback } from 'react';

type Listener = () => void;

class Atom<T> {
  private value: T;
  private listeners: Set<Listener> = new Set();
  
  constructor(initialValue: T) {
    this.value = initialValue;
  }
  
  get(): T {
    return this.value;
  }
  
  set(value: T): void {
    this.value = value;
    this.listeners.forEach(listener => listener());
  }
  
  subscribe(listener: Listener): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }
}

function atom<T>(initialValue: T): Atom<T> {
  return new Atom(initialValue);
}

// 创建 atoms
const userAtom = atom<User | null>(null);
const themeAtom = atom<'light' | 'dark'>('light');
const cartAtom = atom<CartItem[]>([]);

// Store 上下文
interface StoreValue {
  userAtom: Atom<User | null>;
  themeAtom: Atom<'light' | 'dark'>;
  cartAtom: Atom<CartItem[]>;
}

const StoreContext = createContext<StoreValue | null>(null);

export function StoreProvider({ children }: { children: React.ReactNode }) {
  return (
    <StoreContext.Provider value={{
      userAtom,
      themeAtom,
      cartAtom
    }}>
      {children}
    </StoreContext.Provider>
  );
}

// useAtom hook
function useAtom<T>(atom: Atom<T>): [T, (value: T) => void] {
  const store = useContext(StoreContext);
  if (!store) {
    throw new Error('useAtom must be used within StoreProvider');
  }
  
  const [value, setValue] = useState(() => atom.get());
  
  useEffect(() => {
    return atom.subscribe(() => {
      setValue(atom.get());
    });
  }, [atom]);
  
  return [value, atom.set.bind(atom)];
}

// 使用
function UserProfile() {
  const [user, setUser] = useAtom(userAtom);
  
  return user ? (
    <p>Welcome, {user.name}</p>
  ) : (
    <button onClick={() => setUser({ id: '1', name: 'John' })}>
      Login
    </button>
  );
}

function Cart() {
  const [items, setItems] = useAtom(cartAtom);
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

## 0x05 外部状态管理库

### Zustand

```tsx
import { create } from 'zustand';

interface User {
  id: string;
  name: string;
}

interface CartItem {
  id: string;
  name: string;
  price: number;
}

interface Store {
  user: User | null;
  theme: 'light' | 'dark';
  cart: CartItem[];
  
  // Actions
  setUser: (user: User | null) => void;
  toggleTheme: () => void;
  addToCart: (item: CartItem) => void;
  removeFromCart: (id: string) => void;
  clearCart: () => void;
}

const useStore = create<Store>((set) => ({
  user: null,
  theme: 'light',
  cart: [],
  
  setUser: (user) => set({ user }),
  toggleTheme: () => set((state) => ({
    theme: state.theme === 'light' ? 'dark' : 'light'
  })),
  addToCart: (item) => set((state) => ({
    cart: [...state.cart, item]
  })),
  removeFromCart: (id) => set((state) => ({
    cart: state.cart.filter(item => item.id !== id)
  })),
  clearCart: () => set({ cart: [] })
}));

// 选择性订阅
function CartBadge() {
  const count = useStore((state) => state.cart.length);
  return <span>Cart: {count}</span>;
}

// 使用
function App() {
  const { user, theme, cart, toggleTheme, addToCart } = useStore();
  
  return (
    <div className={theme}>
      <CartBadge />
      <button onClick={toggleTheme}>Toggle Theme</button>
      <button onClick={() => addToCart({ 
        id: '1', 
        name: 'Product', 
        price: 100 
      })}>
        Add to Cart
      </button>
    </div>
  );
}
```

### Jotai

```tsx
import { atom, useAtom } from 'jotai';

// 基础原子
const countAtom = atom(0);
const userAtom = atom<{ name: string } | null>(null);

// 派生原子（计算值）
const doubledCountAtom = atom((get) => get(countAtom) * 2);

// 异步原子
const dataAtom = atom(async () => {
  const response = await fetch('/api/data');
  return response.json();
});

// 带写入的原子
const persistentCountAtom = atom(
  (get) => get(countAtom),
  (get, set, newValue: number) => {
    set(countAtom, newValue);
    localStorage.setItem('count', String(newValue));
  }
);

// 使用
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const [doubled] = useAtom(doubledCountAtom);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled: {doubled}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// 异步数据
function DataDisplay() {
  const [data] = useAtom(dataAtom);
  
  if (!data) return <p>Loading...</p>;
  
  return <p>Data: {JSON.stringify(data)}</p>;
}
```

### Redux Toolkit

```tsx
import { configureStore, createSlice, PayloadAction } from '@reduxjs/toolkit';

// Slice 定义
interface User {
  id: string;
  name: string;
}

interface UserState {
  currentUser: User | null;
  loading: boolean;
  error: string | null;
}

const userSlice = createSlice({
  name: 'user',
  initialState: {
    currentUser: null,
    loading: false,
    error: null
  } as UserState,
  reducers: {
    setUser: (state, action: PayloadAction<User | null>) => {
      state.currentUser = action.payload;
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    setError: (state, action: PayloadAction<string | null>) => {
      state.error = action.payload;
    }
  }
});

const { setUser, setLoading, setError } = userSlice.actions;

// Store 配置
const store = configureStore({
  reducer: {
    user: userSlice.reducer
  }
});

type RootState = ReturnType<typeof store.getState>;
type AppDispatch = typeof store.dispatch;

// React Hooks
const useAppSelector = <T>(selector: (state: RootState) => T): T => {
  return selector(store.getState());
};

const useAppDispatch = () => store.dispatch;

// 使用
function UserDisplay() {
  const { currentUser, loading, error } = useAppSelector(
    (state) => state.user
  );
  const dispatch = useAppDispatch();
  
  const login = () => {
    dispatch(setLoading(true));
    api.login().then(user => {
      dispatch(setUser(user));
      dispatch(setLoading(false));
    }).catch(err => {
      dispatch(setError(err.message));
      dispatch(setLoading(false));
    });
  };
  
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  
  return currentUser ? (
    <p>Welcome, {currentUser.name}</p>
  ) : (
    <button onClick={login}>Login</button>
  );
}
```

## 0x06 状态管理选择指南

| 场景 | 推荐方案 |
|------|----------|
| 简单组件内状态 | useState |
| 复杂状态逻辑 | useReducer |
| 跨组件共享状态 | Context API |
| 中小型应用 | Zustand / Jotai |
| 大型企业应用 | Redux Toolkit |
| 服务器状态 | React Query / SWR |

## 0x07 最佳实践

### 状态就近原则

```tsx
// ❌ 错误：状态提升过高
function Parent() {
  const [item, setItem] = useState('');
  
  return (
    <div>
      <Input value={item} onChange={setItem} />
      <Display value={item} />
    </div>
  );
}

// ✅ 正确：状态放在需要的地方
function Parent() {
  return (
    <div>
      <Input />
      <Display />
    </div>
  );
}

function Input() {
  const [item, setItem] = useState('');
  
  return <input value={item} onChange={e => setItem(e.target.value)} />;
}

function Display() {
  return <DisplayContent />; // 从 Context 获取需要的共享状态
}
```

### 状态不可变性

```tsx
// ❌ 错误：直接修改状态
function BadComponent() {
  const [items, setItems] = useState<string[]>([]);
  
  const addItem = (item: string) => {
    items.push(item); // 直接修改数组
    setItems(items);
  };
}

// ✅ 正确：创建新的引用
function GoodComponent() {
  const [items, setItems] = useState<string[]>([]);
  
  const addItem = (item: string) => {
    setItems([...items, item]); // 创建新数组
  };
}

// 对象同样适用
function GoodObjectComponent() {
  const [user, setUser] = useState({ name: 'John', age: 30 });
  
  const updateAge = (newAge: number) => {
    setUser({ ...user, age: newAge }); // 创建新对象
  };
}
```

### 派生状态

```tsx
function DerivedState() {
  const [items, setItems] = useState([
    { id: 1, name: 'Apple', price: 1 },
    { id: 2, name: 'Banana', price: 2 }
  ]);
  
  // ✅ 正确：从现有状态派生，不需要独立状态
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const count = items.length;
  
  return (
    <div>
      <p>Total items: {count}</p>
      <p>Total price: ${total}</p>
    </div>
  );
}
```

## 参考

- [React 状态管理](https://react.dev/learn/managing-state)
- [Context API](https://react.dev/reference/react/createContext)
- [Zustand](https://github.com/pmndrs/zustand)
- [Jotai](https://github.com/pmndrs/jotai)
- [Redux Toolkit](https://redux-toolkit.js.org)
