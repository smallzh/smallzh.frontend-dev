# 映射类型（Mapped Types）

## 概述

映射类型允许我们基于现有类型创建新类型，通过遍历现有类型的属性来创建新类型。

## 基本语法

### 基本映射类型

```typescript
// 基本映射类型
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

interface User {
  id: number;
  name: string;
  email: string;
}

type ReadonlyUser = Readonly<User>;
// { readonly id: number; readonly name: string; readonly email: string; }

type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; }
```

### 映射修饰符

```typescript
// + 和 - 修饰符
type Required<T> = {
  [P in keyof T]-?: T[P]; // 移除可选修饰符
};

type Mutable<T> = {
  -readonly [P in keyof T]: T[P]; // 移除只读修饰符
};

interface User {
  readonly id: number;
  name?: string;
  email?: string;
}

type RequiredUser = Required<User>;
// { readonly id: number; name: string; email: string; }

type MutableUser = Mutable<User>;
// { id: number; name?: string; email?: string; }
```

## 内置映射类型

### Partial

```typescript
// Partial<T> - 所有属性变为可选
interface User {
  id: number;
  name: string;
  email: string;
}

function updateUser(id: number, updates: Partial<User>): void {
  // 更新用户
}

updateUser(1, { name: 'Jane' });
updateUser(1, { name: 'Jane', email: 'jane@example.com' });
```

### Required

```typescript
// Required<T> - 所有属性变为必需
interface User {
  id?: number;
  name?: string;
  email?: string;
}

type RequiredUser = Required<User>;
// { id: number; name: string; email: string; }
```

### Readonly

```typescript
// Readonly<T> - 所有属性变为只读
interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = { id: 1, name: 'John' };
// user.name = 'Jane'; // 错误：只读属性
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
// { id: number; name: string; }
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
// { id: number; name: string; email: string; }
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
```

## 高级映射类型

### 重映射键名

```typescript
// 重映射键名（TypeScript 4.1+）
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User {
  id: number;
  name: string;
  email: string;
}

type UserGetters = Getters<User>;
// { getId: () => number; getName: () => string; getEmail: () => string; }
```

### 过滤键名

```typescript
// 过滤键名
type FilterByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface User {
  id: number;
  name: string;
  age: number;
  active: boolean;
}

type StringProps = FilterByType<User, string>; // { name: string; }
type NumberProps = FilterByType<User, number>; // { id: number; age: number; }
```

### 模板字面量类型

```typescript
// 模板字面量类型
type EventNames<T extends string> = `on${Capitalize<T>}`;

type ClickEvent = EventNames<'click'>; // 'onClick'
type FocusEvent = EventNames<'focus'>; // 'onFocus'

// 组合使用
type EventHandlers<T extends string> = {
  [K in T as `on${Capitalize<K>}`]: () => void;
};

type ButtonEvents = EventHandlers<'click' | 'focus' | 'blur'>;
// { onClick: () => void; onFocus: () => void; onBlur: () => void; }
```

## 实际应用示例

### 1. 表单验证

```typescript
interface FormValues {
  username: string;
  email: string;
  password: string;
}

type FormErrors = {
  [K in keyof FormValues]?: string;
};

type FormTouched = {
  [K in keyof FormValues]?: boolean;
};

type FormState = {
  values: FormValues;
  errors: FormErrors;
  touched: FormTouched;
};
```

### 2. API 响应类型

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// 可选字段版本
type PartialUser = Partial<User>;

// 必填字段版本
type RequiredUser = Required<User>;

// 只读版本
type ReadonlyUser = Readonly<User>;

// 选择特定字段
type UserCredentials = Pick<User, 'email' | 'password'>;

// 排除特定字段
type UserPublicInfo = Omit<User, 'password'>;
```

### 3. 配置对象

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  retries: number;
  debug: boolean;
}

// 可选配置
type ConfigOptions = Partial<Config>;

// 必填配置
type RequiredConfig = Required<Config>;

// 只读配置
type ReadonlyConfig = Readonly<Config>;

// 选择特定配置
type NetworkConfig = Pick<Config, 'apiUrl' | 'timeout' | 'retries'>;
```

### 4. 事件系统

```typescript
interface EventMap {
  click: MouseEvent;
  focus: FocusEvent;
  blur: FocusEvent;
  submit: SubmitEvent;
}

// 事件处理器类型
type EventHandlers = {
  [K in keyof EventMap as `on${Capitalize<K>}`]: (event: EventMap[K]) => void;
};

// 使用
const handlers: EventHandlers = {
  onClick: (event: MouseEvent) => console.log(event.clientX),
  onFocus: (event: FocusEvent) => console.log(event.target),
  onBlur: (event: FocusEvent) => console.log(event.target),
  onSubmit: (event: SubmitEvent) => console.log(event.target)
};
```

## 最佳实践

1. **使用内置映射类型**：Partial、Required、Readonly 等
2. **使用 Pick 和 Omit**：选择或排除属性
3. **使用 Record**：创建键值对类型
4. **结合条件类型**：创建更灵活的类型

```typescript
// 好的做法
type FormState<T> = {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
};

// 避免
type FormState = {
  values: any;
  errors: any;
  touched: any;
};
```

## 参考

- [TypeScript Handbook - Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
