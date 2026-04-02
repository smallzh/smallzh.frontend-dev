# TypeScript 概述

## 什么是 TypeScript？

TypeScript 是由微软开发的开源编程语言，是 JavaScript 的超集，添加了可选的静态类型系统和基于类的面向对象编程。

## TypeScript 的特点

### 1. 静态类型

```typescript
// 类型注解
let name: string = 'John';
let age: number = 25;
let isStudent: boolean = true;

// 类型推断
let message = 'Hello'; // 自动推断为 string 类型
let count = 42; // 自动推断为 number 类型
```

### 2. 类型系统

```typescript
// 接口
interface User {
  name: string;
  age: number;
  email?: string; // 可选属性
}

// 类型别名
type ID = string | number;
type Callback = (data: string) => void;

// 泛型
function identity<T>(arg: T): T {
  return arg;
}
```

### 3. 编译时类型检查

```typescript
function greet(name: string): string {
  return `Hello, ${name}!`;
}

greet('John'); // 正确
// greet(123); // 编译错误：类型不匹配
```

### 4. 类和接口

```typescript
interface Printable {
  print(): void;
}

class Document implements Printable {
  constructor(private content: string) {}
  
  print(): void {
    console.log(this.content);
  }
}
```

### 5. 模块系统

```typescript
// 导出
export interface User {
  name: string;
  age: number;
}

export function createUser(name: string, age: number): User {
  return { name, age };
}

// 导入
import { User, createUser } from './user';
```

## TypeScript 的优势

### 1. 更好的开发体验

```typescript
// IDE 支持
const user = {
  name: 'John',
  age: 25,
  email: 'john@example.com'
};

// 自动补全和类型检查
console.log(user.name); // IDE 知道 name 是 string 类型
// console.log(user.phone); // 编译错误：属性不存在
```

### 2. 减少运行时错误

```typescript
// 编译时捕获错误
function divide(a: number, b: number): number {
  return a / b;
}

divide(10, 2); // 正确
// divide(10, '2'); // 编译错误：类型不匹配
// divide(10, 0); // 运行时错误，但至少类型是正确的
```

### 3. 更好的代码文档

```typescript
// 类型即文档
interface ApiConfig {
  baseUrl: string;
  timeout: number;
  headers?: Record<string, string>;
}

function createApi(config: ApiConfig): ApiClient {
  // ...
}
```

### 4. 更强的重构能力

```typescript
// 重命名变量时，TypeScript 会检查所有引用
let userName = 'John';
console.log(userName);

// 重命名后，所有引用都会更新
let currentUser = 'John';
console.log(currentUser);
```

## TypeScript 与 JavaScript 的关系

### TypeScript 是 JavaScript 的超集

```typescript
// 所有有效的 JavaScript 都是有效的 TypeScript
const message = 'Hello';
const numbers = [1, 2, 3];
const user = { name: 'John', age: 25 };

// TypeScript 添加了类型系统
const typedMessage: string = 'Hello';
const typedNumbers: number[] = [1, 2, 3];
const typedUser: { name: string; age: number } = { name: 'John', age: 25 };
```

### 编译到 JavaScript

```typescript
// TypeScript 代码
interface User {
  name: string;
  age: number;
}

function greet(user: User): string {
  return `Hello, ${user.name}!`;
}

// 编译后的 JavaScript 代码
function greet(user) {
  return "Hello, ".concat(user.name, "!");
}
```

## TypeScript 的应用场景

### 1. 大型项目

```typescript
// 复杂的类型定义
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: number;
  name: string;
  email: string;
}

interface Post {
  id: number;
  title: string;
  content: string;
  author: User;
}

// 类型安全的 API 调用
async function fetchUsers(): Promise<ApiResponse<User[]>> {
  const response = await fetch('/api/users');
  return response.json();
}
```

### 2. 团队协作

```typescript
// 清晰的接口定义
interface UserService {
  createUser(data: CreateUserDto): Promise<User>;
  getUserById(id: number): Promise<User | null>;
  updateUser(id: number, data: UpdateUserDto): Promise<User>;
  deleteUser(id: number): Promise<void>;
}

// 实现接口
class UserServiceImpl implements UserService {
  // 必须实现所有方法
}
```

### 3. 前端框架

```typescript
// React 组件
interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({ 
  text, 
  onClick, 
  variant = 'primary', 
  disabled = false 
}) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {text}
    </button>
  );
};
```

## 开发环境搭建

### 1. 安装 TypeScript

```bash
# 全局安装
npm install -g typescript

# 项目内安装
npm install --save-dev typescript
```

### 2. 初始化配置

```bash
# 创建 tsconfig.json
tsc --init
```

### 3. 基本配置

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 4. 编译 TypeScript

```bash
# 编译单个文件
tsc src/index.ts

# 编译整个项目
tsc

# 监听模式
tsc --watch
```

## 学习路径

### 初级阶段

1. **类型基础**：基本类型、数组、元组
2. **接口和类型别名**：定义对象类型
3. **函数类型**：参数类型、返回值类型
4. **类**：属性、方法、继承

### 中级阶段

1. **泛型**：类型参数、约束
2. **联合类型和交叉类型**：组合类型
3. **类型守卫**：类型收窄
4. **枚举**：数字枚举、字符串枚举

### 高级阶段

1. **高级类型**：条件类型、映射类型
2. **装饰器**：类装饰器、方法装饰器
3. **模块和命名空间**：代码组织
4. **声明文件**：类型定义

## 最佳实践

1. **启用严格模式**：`strict: true`
2. **避免使用 `any`**：尽量使用具体类型
3. **使用接口定义对象类型**：更清晰的类型定义
4. **使用泛型提高代码复用**：类型参数化
5. **利用类型推断**：让 TypeScript 自动推断类型

```typescript
// 好的做法
interface User {
  id: number;
  name: string;
  email: string;
}

function getUser(id: number): Promise<User> {
  return fetch(`/api/users/${id}`).then(res => res.json());
}

// 避免
function getUser(id: any): any {
  return fetch(`/api/users/${id}`).then(res => res.json());
}
```

## 参考资料

- [TypeScript 官方文档](https://www.typescriptlang.org/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
