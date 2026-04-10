# React 概述

## 0x01 什么是 React？

React 是 Facebook 开发的用于构建用户界面的 JavaScript 库。其核心特点是：

- **声明式**：以声明式编写 UI，代码更加简洁且可预测
- **基于组件**：构建封装好的自包含组件，每个组件维护自己的状态
- **Learn Once, Write Anywhere**：支持 Web、移动端（React Native）等多平台

## 0x02 环境搭建

### 使用 Vite 创建项目

```bash
# 创建 React 项目（推荐）
npm create vite@latest my-react-app -- --template react-ts

# 进入项目目录
cd my-react-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

### 使用 Next.js 创建项目

```bash
# 创建 Next.js 项目（支持 React 19）
npx create-next-app@latest my-next-app

# 启动开发服务器
npm run dev
```

## 0x03 React 核心概念

### JSX 语法

JSX 是 JavaScript 的语法扩展，允许在 JavaScript 中编写类似 HTML 的标记。

```tsx
// 基本 JSX
function App() {
  return (
    <div className="app">
      <h1>Hello React</h1>
      <p>Welcome to React</p>
    </div>
  );
}

// 在 JSX 中嵌入表达式
function Greeting({ name }) {
  const now = new Date();
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>Current time: {now.toLocaleTimeString()}</p>
    </div>
  );
}
```

### 元素与组件

```tsx
// React 元素（最小的构建块）
const element = <h1>Hello World</h1>;

// 函数组件
function Welcome({ name }) {
  return <h1>Welcome, {name}!</h1>;
}

// 类组件
class Welcome extends React.Component {
  render() {
    return <h1>Welcome, {this.props.name}!</h1>;
  }
}
```

## 0x04 React 19 新特性

React 19 引入了多项重大更新：

- **Server Components**：服务组件直接在服务器上渲染
- **Actions**：内置的表单处理和数据提交功能
- **use` 钩子**：消费 Promise 的新方式
- **useOptimistic`：乐观更新模式
- **useFormStatus` / `useFormAction`：表单状态管理

```tsx
// React 19 新特性示例
'use client';

import { use } from 'react';

// 使用 use 钩子消费 Promise
function Comments({ commentsPromise }) {
  const comments = use(commentsPromise);
  return comments.map(comment => (
    <div key={comment.id}>{comment.text}</div>
  ));
}

// 乐观更新
import { useOptimistic } from 'react';

function TodoItem({ id, text, completed }) {
  const [optimistic, addOptimistic] = useOptimistic(
    { text, completed },
    (state, newText) => ({ ...state, text: newText })
  );
  
  return <li>{optimistic.text}</li>;
}
```

## 0x05 项目结构

典型的 React 项目结构：

```
my-react-app/
├── src/
│   ├── components/      # 组件目录
│   ├── hooks/           # 自定义 Hooks
│   ├── pages/           # 页面组件
│   ├── App.tsx          # 主应用组件
│   └── main.tsx         # 入口文件
├── index.html           # HTML 模板
├── package.json         # 项目配置
└── vite.config.ts       # Vite 配置
```

## 参考

- [React 官方文档](https://react.dev)
- [React GitHub](https://github.com/facebook/react)
