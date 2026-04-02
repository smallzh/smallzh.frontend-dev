# 泛型（Generics）

## 概述

泛型是 TypeScript 的核心特性之一，允许我们编写可重用的组件，同时保持类型安全。

## 基本语法

### 泛型函数

```typescript
// 泛型函数
function identity<T>(arg: T): T {
  return arg;
}

// 类型推断
const num = identity(42); // T 推断为 number
const str = identity('hello'); // T 推断为 string

// 显式指定类型
const result = identity<number>(42);
const result2 = identity<string>('hello');
```

### 泛型接口

```typescript
// 泛型接口
interface Container<T> {
  value: T;
  getValue(): T;
  setValue(value: T): void;
}

// 实现泛型接口
class Box<T> implements Container<T> {
  constructor(public value: T) {}

  getValue(): T {
    return this.value;
  }

  setValue(value: T): void {
    this.value = value;
  }
}

const numberBox = new Box<number>(42);
const stringBox = new Box<string>('hello');
```

### 泛型类

```typescript
// 泛型类
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
numberStack.push(3);

const stringStack = new Stack<string>();
stringStack.push('a');
stringStack.push('b');
```

## 泛型约束

### 基本约束

```typescript
// 泛型约束
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength('hello'); // 正确
logLength([1, 2, 3]); // 正确
// logLength(42); // 错误：number 没有 length 属性
```

### keyof 约束

```typescript
// keyof 约束
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'John', age: 25 };
const name = getProperty(user, 'name'); // string
const age = getProperty(user, 'age'); // number
// getProperty(user, 'email'); // 错误：email 不是 User 的键
```

### 多重约束

```typescript
// 多重约束
interface Printable {
  print(): void;
}

interface Loggable {
  log(): void;
}

function process<T extends Printable & Loggable>(item: T): void {
  item.print();
  item.log();
}
```

## 泛型工具类型

### Partial

```typescript
// Partial<T> - 将所有属性变为可选
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// 等价于：{ id?: number; name?: string; email?: string; }

function updateUser(id: number, updates: Partial<User>): void {
  // 更新用户
}

updateUser(1, { name: 'Jane' }); // 正确
updateUser(1, { name: 'Jane', email: 'jane@example.com' }); // 正确
```

### Required

```typescript
// Required<T> - 将所有属性变为必需
interface User {
  id?: number;
  name?: string;
  email?: string;
}

type RequiredUser = Required<User>;
// 等价于：{ id: number; name: string; email: string; }
```

### Readonly

```typescript
// Readonly<T> - 将所有属性变为只读
interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = { id: 1, name: 'John' };
// user.name = 'Jane'; // 错误：只读属性
```

### Record

```typescript
// Record<K, V> - 创建键值对类型
type UserRoles = Record<number, string>;

const roles: UserRoles = {
  1: 'admin',
  2: 'user',
  3: 'guest'
};

// 更复杂的例子
interface User {
  id: number;
  name: string;
}

type UserMap = Record<string, User>;

const users: UserMap = {
  'user-1': { id: 1, name: 'John' },
  'user-2': { id: 2, name: 'Jane' }
};
```

### Pick

```typescript
// Pick<T, K> - 选择部分属性
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type UserBasicInfo = Pick<User, 'id' | 'name'>;
// 等价于：{ id: number; name: string; }
```

### Omit

```typescript
// Omit<T, K> - 排除部分属性
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type UserWithoutAge = Omit<User, 'age'>;
// 等价于：{ id: number; name: string; email: string; }
```

### Exclude 和 Extract

```typescript
// Exclude<T, U> - 从联合类型中排除类型
type T1 = Exclude<'a' | 'b' | 'c', 'a'>; // 'b' | 'c'
type T2 = Exclude<string | number | (() => void), Function>; // string | number

// Extract<T, U> - 从联合类型中提取类型
type T3 = Extract<'a' | 'b' | 'c', 'a' | 'f'>; // 'a'
type T4 = Extract<string | number | (() => void), Function>; // () => void
```

### NonNullable

```typescript
// NonNullable<T> - 排除 null 和 undefined
type T1 = NonNullable<string | number | undefined>; // string | number
type T2 = NonNullable<string | null>; // string
```

### ReturnType

```typescript
// ReturnType<T> - 获取函数返回类型
function createUser() {
  return { id: 1, name: 'John' };
}

type User = ReturnType<typeof createUser>;
// { id: number; name: string; }
```

### Parameters

```typescript
// Parameters<T> - 获取函数参数类型
function createUser(name: string, age: number) {
  return { id: 1, name, age };
}

type CreateUserParams = Parameters<typeof createUser>;
// [string, number]
```

## 条件类型

### 基本条件类型

```typescript
// 条件类型
type IsString<T> = T extends string ? true : false;

type T1 = IsString<string>; // true
type T2 = IsString<number>; // false

// 更复杂的例子
type Flatten<T> = T extends Array<infer Item> ? Item : T;

type T3 = Flatten<string[]>; // string
type T4 = Flatten<number[]>; // number
type T5 = Flatten<string>; // string
```

### infer 关键字

```typescript
// infer 关键字
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type T1 = ReturnType<() => string>; // string
type T2 = ReturnType<(x: number) => number>; // number

// 提取 Promise 类型
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type T3 = UnwrapPromise<Promise<string>>; // string
type T4 = UnwrapPromise<number>; // number
```

## 映射类型

### 基本映射类型

```typescript
// 映射类型
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
```

### 自定义映射类型

```typescript
// 自定义映射类型
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

interface User {
  readonly id: number;
  name: string;
}

type NullableUser = Nullable<User>;
type MutableUser = Mutable<User>;
```

## 实际应用示例

### 1. API 响应类型

```typescript
type ApiResponse<T> = {
  success: boolean;
  data: T;
  error?: string;
};

type PaginatedResponse<T> = ApiResponse<{
  items: T[];
  total: number;
  page: number;
  pageSize: number;
}>;

interface User {
  id: number;
  name: string;
}

type UserResponse = ApiResponse<User>;
type UsersResponse = PaginatedResponse<User>;
```

### 2. 状态管理

```typescript
type LoadingState<T> = {
  status: 'loading';
};

type SuccessState<T> = {
  status: 'success';
  data: T;
};

type ErrorState = {
  status: 'error';
  error: string;
};

type AsyncState<T> = LoadingState<T> | SuccessState<T> | ErrorState;
```

### 3. 表单类型

```typescript
type FormField<T> = {
  value: T;
  error?: string;
  touched: boolean;
};

type Form<T> = {
  [K in keyof T]: FormField<T[K]>;
};

interface LoginForm {
  username: string;
  password: string;
  remember: boolean;
}

type LoginFormFields = Form<LoginForm>;
```

## 最佳实践

1. **使用泛型提高代码复用性**
2. **使用约束限制泛型类型**
3. **使用工具类型简化类型定义**
4. **避免过度使用泛型**

```typescript
// 好的做法
function identity<T>(arg: T): T {
  return arg;
}

interface Repository<T> {
  getById(id: number): T | null;
  save(item: T): void;
}

// 避免
function identity(arg: any): any {
  return arg;
}
```

## 参考

- [TypeScript Handbook - Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
