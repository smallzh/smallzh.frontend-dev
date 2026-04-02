# 条件类型（Conditional Types）

## 概述

条件类型允许我们根据条件选择类型，类似于 JavaScript 中的三元运算符。

## 基本语法

### 条件类型基础

```typescript
// 条件类型语法
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false

// 更复杂的条件
type Flatten<T> = T extends Array<infer Item> ? Item : T;

type C = Flatten<string[]>; // string
type D = Flatten<number[]>; // number
type E = Flatten<string>; // string
```

### infer 关键字

```typescript
// infer 关键字
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function createUser() {
  return { id: 1, name: 'John' };
}

type User = ReturnType<typeof createUser>;
// { id: number; name: string; }

// 提取 Promise 类型
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type A = UnwrapPromise<Promise<string>>; // string
type B = UnwrapPromise<number>; // number
```

## 条件类型分发

### 分发特性

```typescript
// 条件类型分发
type ToArray<T> = T extends any ? T[] : never;

type A = ToArray<string | number>; // string[] | number[]

// 禁用分发
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type B = ToArrayNonDist<string | number>; // (string | number)[]
```

## 内置条件类型

### Exclude

```typescript
// Exclude<T, U> - 从 T 中排除可分配给 U 的类型
type T1 = Exclude<'a' | 'b' | 'c', 'a'>; // 'b' | 'c'
type T2 = Exclude<string | number | (() => void), Function>; // string | number

// 等价于
type MyExclude<T, U> = T extends U ? never : T;
```

### Extract

```typescript
// Extract<T, U> - 从 T 中提取可分配给 U 的类型
type T1 = Extract<'a' | 'b' | 'c', 'a' | 'f'>; // 'a'
type T2 = Extract<string | number | (() => void), Function>; // () => void

// 等价于
type MyExtract<T, U> = T extends U ? T : never;
```

### NonNullable

```typescript
// NonNullable<T> - 排除 null 和 undefined
type T1 = NonNullable<string | number | undefined>; // string | number
type T2 = NonNullable<string | null>; // string

// 等价于
type MyNonNullable<T> = T extends null | undefined ? never : T;
```

## 条件类型与映射类型

### 结合使用

```typescript
// 条件类型与映射类型
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

interface User {
  id: number;
  name: string;
  greet(): void;
  update(): void;
}

type UserFunctions = FunctionPropertyNames<User>; // 'greet' | 'update'

// 提取函数属性
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type UserFunctionProps = FunctionProperties<User>;
// { greet: () => void; update: () => void; }
```

## 实际应用示例

### 1. API 响应类型

```typescript
// 提取 API 响应数据类型
type ApiData<T> = T extends ApiResponse<infer D> ? D : never;

interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}

interface User {
  id: number;
  name: string;
}

type UserResponse = ApiResponse<User>;
type UserData = ApiData<UserResponse>; // User
```

### 2. 表单验证

```typescript
// 表单字段验证类型
type ValidateField<T> = T extends string
  ? string
  : T extends number
  ? number
  : T extends boolean
  ? boolean
  : never;

interface FormValues {
  username: string;
  age: number;
  active: boolean;
}

type FormValidation = {
  [K in keyof FormValues]: ValidateField<FormValues[K]>;
};
```

### 3. 状态管理

```typescript
// 状态类型推断
type StateValue<T> = T extends { value: infer V } ? V : never;

interface LoadingState {
  status: 'loading';
}

interface SuccessState<T> {
  status: 'success';
  value: T;
}

interface ErrorState {
  status: 'error';
  error: string;
}

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

type UserState = AsyncState<User>;
type UserValue = StateValue<UserState>; // User
```

### 4. 深度只读

```typescript
// 深度只读
type DeepReadonly<T> = T extends (infer U)[]
  ? ReadonlyArray<DeepReadonly<U>>
  : T extends object
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
  : T;

interface User {
  id: number;
  name: string;
  address: {
    city: string;
    country: string;
  };
}

type ReadonlyUser = DeepReadonly<User>;
// {
//   readonly id: number;
//   readonly name: string;
//   readonly address: {
//     readonly city: string;
//     readonly country: string;
//   };
// }
```

## 最佳实践

1. **使用 infer 提取类型**
2. **注意条件类型分发特性**
3. **结合映射类型使用**
4. **避免过于复杂的条件类型**

```typescript
// 好的做法
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

// 避免
type ComplexType<T> = T extends A
  ? T extends B
    ? T extends C
      ? D
      : E
    : F
  : G;
```

## 参考

- [TypeScript Handbook - Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
