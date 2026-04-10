# React 19 新特性

## 0x01 React 19 概述

React 19 是 React 近年来最大的版本更新，引入了多项革命性特性，主要包括：

- **Server Components**：服务端组件，直接在服务器上渲染
- **Actions**：内置的表单处理和数据提交功能
- **`use` 钩子**：消费 Promise 的新方式
- **`useOptimistic`**：乐观更新模式
- **`useFormStatus` / `useActionState`**：表单状态管理
- **Ref 作为 Props**：ref 可以直接作为 props 传递

## 0x02 Server Components（服务端组件）

服务端组件（Server Components）是 React 19 最重要的特性，允许组件在服务器上渲染，减少客户端 JavaScript 体积。

### 基本概念

```tsx
// Server Component - 默认（无 'use client' 指令）
// 这个组件只在服务器上渲染
async function Article({ id }: { id: string }) {
  // 可以直接使用 async/await
  const article = await db.articles.get(id);
  
  return (
    <article>
      <h1>{article.title}</h1>
      <p>{article.content}</p>
    </article>
  );
}

// Client Component - 需要交互的组件
'use client';

function LikeButton({ articleId }: { articleId: string }) {
  const [likes, setLikes] = useState(0);
  
  return (
    <button onClick={() => setLikes(l => l + 1)}>
      Like ({likes})
    </button>
  );
}
```

### 服务端数据获取

```tsx
// 异步 Server Component
async function UserProfile({ userId }: { userId: string }) {
  // 并行获取多个数据
  const [user, posts, settings] = await Promise.all([
    db.users.get(userId),
    db.posts.listByUser(userId),
    db.settings.get(userId)
  ]);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
      <PostList posts={posts} />
      <UserSettings settings={settings} />
    </div>
  );
}

// 流式渲染与 Suspense
import { Suspense } from 'react';

async function Page({ userId }: { userId: string }) {
  const user = await db.users.get(userId);
  
  return (
    <div>
      <h1>{user.name}</h1>
      {/* 非关键数据使用 Suspense */}
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts userId={userId} />
      </Suspense>
      <Suspense fallback={<Skeleton />}>
        <UserActivity userId={userId} />
      </Suspense>
    </div>
  );
}
```

## 0x03 `use` 钩子

`use` 是 React 19 引入的新钩子，用于在组件中消费 Promise 和 Context。

### 消费 Promise

```tsx
'use client';

import { use } from 'react';

// 消费 Promise
function Comments({ commentsPromise }: { commentsPromise: Promise<Comment[]> }) {
  const comments = use(commentsPromise);
  
  return (
    <ul>
      {comments.map(comment => (
        <li key={comment.id}>{comment.text}</li>
      ))}
    </ul>
  );
}

// Server Component 传递 Promise
async function ArticlePage({ articleId }: { articleId: string }) {
  const articlePromise = db.articles.get(articleId);
  const commentsPromise = db.comments.listByArticle(articleId);
  
  const article = await articlePromise;
  
  return (
    <article>
      <h1>{article.title}</h1>
      <p>{article.content}</p>
      <Comments commentsPromise={commentsPromise} />
    </article>
  );
}
```

### 错误处理

```tsx
'use client';

import { use } from 'react';

// 使用 error boundary 处理错误
function UserData({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);
  return <div>{user.name}</div>;
}

// 包装组件处理错误
function UserDataWithError({ userPromise }: { userPromise: Promise<User> }) {
  try {
    const user = use(userPromise);
    return <div>{user.name}</div>;
  } catch (error) {
    if (error instanceof Error && error.message === 'Not Found') {
      return <div>User not found</div>;
    }
    throw error;
  }
}
```

### 消费 Context

```tsx
// React 19 中可以用 use 消费 Context
'use client';

import { use, createContext } from 'react';

const ThemeContext = createContext<string>('light');

function ThemedButton() {
  const theme = use(ThemeContext);
  
  return (
    <button className={theme}>
      Themed Button
    </button>
  );
}

// Provider
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ThemedButton />
    </ThemeContext.Provider>
  );
}
```

## 0x04 Actions（表单操作）

React 19 引入了 Actions，简化表单提交和数据 mutation。

### 基本用法

```tsx
'use client';

import { useActionState, useState } from 'react';

// Server Action（在服务器执行）
async function createItem(formData: FormData) {
  const name = formData.get('name');
  const description = formData.get('description');
  
  await db.items.create({ name, description });
  
  return { success: true };
}

// 使用 useActionState
function CreateItem() {
  const [state, action, isPending] = useActionState(createItem, null);
  
  return (
    <form action={action}>
      <input name="name" placeholder="Item name" />
      <input name="description" placeholder="Description" />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create'}
      </button>
      {state?.success && <p>Item created!</p>}
    </form>
  );
}
```

### 渐进式增强

```tsx
'use client';

import { useActionState, useState } from 'react';

// Server Action
async function submitForm(prevState: any, formData: FormData) {
  const email = formData.get('email');
  const password = formData.get('password');
  
  const errors: Record<string, string> = {};
  
  if (!email || !email.includes('@')) {
    errors.email = 'Invalid email';
  }
  if (!password || password.length < 8) {
    errors.password = 'Password must be at least 8 characters';
  }
  
  if (Object.keys(errors).length > 0) {
    return { errors, values: { email, password } };
  }
  
  await auth.login({ email, password });
  
  return { success: true };
}

// 表单组件
function LoginForm() {
  const [state, action, isPending] = useActionState(submitForm, null);
  
  return (
    <form action={action}>
      <div>
        <label>Email</label>
        <input 
          name="email" 
          type="email"
          defaultValue={state?.values?.email}
        />
        {state?.errors?.email && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>
      <div>
        <label>Password</label>
        <input 
          name="password" 
          type="password"
          defaultValue={state?.values?.password}
        />
        {state?.errors?.password && (
          <span className="error">{state.errors.password}</span>
        )}
      </div>
      <button type="submit" disabled={isPending}>
        {isPending ? 'Logging in...' : 'Login'}
      </button>
      {state?.success && <p>Welcome back!</p>}
    </form>
  );
}
```

## 0x05 useOptimistic（乐观更新）

`useOptimistic` 允许在异步操作完成前立即更新 UI，提供即时反馈。

### 基本用法

```tsx
'use client';

import { useOptimistic } from 'react';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

function TodoItem({ todo, onToggle }: { 
  todo: Todo;
  onToggle: (id: number) => void;
}) {
  // 乐观更新状态
  const [optimisticTodo, addOptimistic] = useOptimistic(
    todo,
    (state, newCompleted: boolean) => ({
      ...state,
      completed: newCompleted
    })
  );
  
  const handleToggle = async () => {
    // 立即更新 UI
    addOptimistic(!todo.completed);
    // 异步操作在后台进行
    await onToggle(todo.id);
  };
  
  return (
    <li>
      <input
        type="checkbox"
        checked={optimisticTodo.completed}
        onChange={handleToggle}
      />
      <span style={{
        textDecoration: optimisticTodo.completed ? 'line-through' : 'none'
      }}>
        {todo.text}
      </span>
    </li>
  );
}
```

### 更复杂的乐观更新

```tsx
'use client';

import { useOptimistic } from 'react';

interface Message {
  id: string;
  text: string;
  status: 'sending' | 'sent' | 'failed';
}

function Chat() {
  const [messages, setMessages] = useState<Message[]>([]);
  
  // 乐观状态
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage: Message) => [...state, newMessage]
  );
  
  const sendMessage = async (text: string) => {
    const tempId = crypto.randomUUID();
    
    // 添加乐观消息
    addOptimisticMessage({
      id: tempId,
      text,
      status: 'sending'
    });
    
    // 发送到服务器
    try {
      await api.sendMessage(text);
      // 成功后更新状态
      setMessages(prev => prev.map(msg => 
        msg.id === tempId ? { ...msg, status: 'sent' } : msg
      ));
    } catch {
      // 失败后更新状态
      setMessages(prev => prev.map(msg => 
        msg.id === tempId ? { ...msg, status: 'failed' } : msg
      ));
    }
  };
  
  return (
    <div>
      {optimisticMessages.map(msg => (
        <div 
          key={msg.id}
          className={msg.status}
        >
          {msg.text}
          {msg.status === 'sending' && '...'}
        </div>
      ))}
    </div>
  );
}
```

## 0x06 useFormStatus

`useFormStatus` 提供表单提交期间的状态信息。

### 基本用法

```tsx
'use client';

import { useFormStatus } from 'react';

function SubmitButton() {
  const { pending, data, method } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

function LoginForm() {
  return (
    <form>
      <input name="username" />
      <input name="password" />
      <SubmitButton />
    </form>
  );
}
```

### 与 Server Action 结合

```tsx
'use client';

import { useFormStatus } from 'react';

function AsyncButton({ action }: { action: (formData: FormData) => void }) {
  const { pending } = useFormStatus();
  
  return (
    <button formAction={action} disabled={pending}>
      {pending ? 'Loading...' : 'Submit'}
    </button>
  );
}

// Server Action
async function createItem(formData: FormData) {
  'use server';
  await db.items.create({
    name: formData.get('name')
  });
}

function Form() {
  return (
    <form>
      <input name="name" />
      <AsyncButton action={createItem} />
    </form>
  );
}
```

## 0x07 Ref 作为 Props

React 19 允许将 ref 直接作为 props 传递给组件。

### 基本用法

```tsx
'use client';

import { useRef } from 'react';

// 直接传递 ref
function MyInput({ ref }: { ref: React.Ref<HTMLInputElement> }) {
  return <input ref={ref} />;
}

function Parent() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const handleClick = () => {
    inputRef.current?.focus();
  };
  
  return (
    <div>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus Input</button>
    </div>
  );
}
```

### 转发 Ref

```tsx
'use client';

import { forwardRef, useRef } from 'react';

// 使用 forwardRef 转发 ref
const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  function Button({ children, ...props }, ref) {
    return (
      <button ref={ref} {...props}>
        {children}
      </button>
    );
  }
);

function App() {
  const buttonRef = useRef<HTMLButtonElement>(null);
  
  return (
    <Button ref={buttonRef}>Click Me</Button>
  );
}
```

## 0x08 新的 Hooks API

### useDeferredValue

```tsx
'use client';

import { useDeferredValue, useState } from 'react';

function SearchResults({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query);
  const results = useMemo(() => 
    search(deferredQuery), 
    [deferredQuery]
  );
  
  return (
    <ul>
      {results.map(result => (
        <li key={result.id}>{result.title}</li>
      ))}
    </ul>
  );
}

function Search() {
  const [query, setQuery] = useState('');
  
  return (
    <div>
      <input 
        value={query}
        onChange={e => setQuery(e.target.value)}
      />
      <SearchResults query={query} />
    </div>
  );
}
```

### useTransition

```tsx
'use client';

import { useTransition, useState } from 'react';

function TabContent() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');
  
  const selectTab = (newTab: string) => {
    startTransition(() => {
      setTab(newTab);
    });
  };
  
  return (
    <div>
      {isPending ? <p>Loading...</p> : null}
      <button onClick={() => selectTab('about')}>About</button>
      <button onClick={() => selectTab('posts')}>Posts</button>
      <button onClick={() => selectTab('contact')}>Contact</button>
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </div>
  );
}
```

### useId

```tsx
'use client';

import { useId } from 'react';

function FormField({ label }: { label: string }) {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input id={id} type="text" />
    </div>
  );
}
```

### useSyncExternalStore

```tsx
'use client';

import { useSyncExternalStore } from 'react';

// 订阅外部 store
function useOnlineStatus() {
  return useSyncExternalStore(
    (onStoreChange) => {
      window.addEventListener('online', onStoreChange);
      window.addEventListener('offline', onStoreChange);
      
      return () => {
        window.removeEventListener('online', onStoreChange);
        window.removeEventListener('offline', onStoreChange);
      };
    },
    () => navigator.onLine,
    () => false
  );
}

function StatusBanner() {
  const isOnline = useOnlineStatus();
  
  return (
    <div className={isOnline ? 'online' : 'offline'}>
      {isOnline ? 'Online' : 'Offline'}
    </div>
  );
}
```

## 0x09 迁移指南

### 从 React 18 升级

```tsx
// React 18 写法
import { useState, useEffect } from 'react';

function Component() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  
  return <div>{data}</div>;
}

// React 19 写法（更简洁）
async function Component() {
  const data = await fetchData();
  return <div>{data}</div>;
}

// 渐进式迁移
// 1. 更新依赖
npm install react@19 react-dom@19

// 2. 逐个迁移组件
// 3. 使用新的 use 钩子
// 4. 启用 Actions 功能
```

### 检查兼容性

```bash
# 安装 React 19
npm install react@19 react-dom@19

# 检查不兼容的包
npm audit
```

## 0x10 最佳实践

### 选择正确的组件类型

```tsx
// 需要交互 → Client Component
'use client';
function LikeButton() {
  const [likes, setLikes] = useState(0);
  return <button onClick={() => setLikes(l => l + 1)}>Like</button>;
}

// 需要获取数据 → Server Component
async function ArticleList() {
  const articles = await db.articles.list();
  return articles.map(a => <Article key={a.id} {...a} />);
}

// 混合使用
async function Page() {
  return (
    <div>
      <h1>Articles</h1>
      <ArticleList />
      <LikeButton />  {/* 客户端交互 */}
    </div>
  );
}
```

### 正确使用 Actions

```tsx
// ✅ 正确：使用 Actions 处理表单
async function createItem(prevState: State, formData: FormData) {
  const result = await db.create(formData);
  return { success: true };
}

// ❌ 错误：不要在 Actions 中做太多客户端逻辑
async function badAction(formData: FormData) {
  // 这应该在客户端处理
  const processed = complexClientLogic(formData);
  await db.create(processed);
}
```

## 参考

- [React 19 官方文档](https://react.dev/blog/2024/04/25/react-19)
- [React 19 升级指南](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)
- [Server Components](https://react.dev/reference/rsc)
- [Actions](https://react.dev/reference/rsc/actions)
