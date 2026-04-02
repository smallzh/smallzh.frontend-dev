# 函数类型（Function Types）

## 概述

TypeScript 提供了丰富的函数类型定义方式，用于描述函数的参数类型、返回值类型和调用签名。

## 基本函数类型

### 函数声明

```typescript
// 函数声明
function add(a: number, b: number): number {
  return a + b;
}

// 箭头函数
const multiply = (a: number, b: number): number => a * b;

// 函数表达式
const divide = function(a: number, b: number): number {
  return a / b;
};
```

### 函数类型表达式

```typescript
// 函数类型表达式
type AddFunction = (a: number, b: number) => number;

const add: AddFunction = (a, b) => a + b;

// 直接定义
let operation: (a: number, b: number) => number;
operation = (a, b) => a + b;
```

### 调用签名

```typescript
// 调用签名（接口形式）
interface MathOperation {
  (a: number, b: number): number;
}

const add: MathOperation = (a, b) => a + b;
const multiply: MathOperation = (a, b) => a * b;

// 带属性的调用签名
interface Counter {
  (): number;
  count: number;
}

const counter: Counter = Object.assign(
  () => ++counter.count,
  { count: 0 }
);
```

## 参数类型

### 可选参数

```typescript
// 可选参数
function greet(name: string, greeting?: string): string {
  return `${greeting || 'Hello'}, ${name}!`;
}

greet('John'); // 'Hello, John!'
greet('John', 'Hi'); // 'Hi, John!'
```

### 默认参数

```typescript
// 默认参数
function greet(name: string, greeting: string = 'Hello'): string {
  return `${greeting}, ${name}!`;
}

greet('John'); // 'Hello, John!'
greet('John', 'Hi'); // 'Hi, John!'
```

### 剩余参数

```typescript
// 剩余参数
function sum(...numbers: number[]): number {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3, 4, 5); // 15

// 混合参数
function log(message: string, ...args: any[]): void {
  console.log(message, ...args);
}
```

## 返回值类型

### 显式返回类型

```typescript
// 显式返回类型
function add(a: number, b: number): number {
  return a + b;
}

function greet(name: string): string {
  return `Hello, ${name}!`;
}

function log(message: string): void {
  console.log(message);
}
```

### 类型推断

```typescript
// TypeScript 可以推断返回类型
function add(a: number, b: number) {
  return a + b; // 推断返回类型为 number
}

function greet(name: string) {
  return `Hello, ${name}!`; // 推断返回类型为 string
}
```

## 泛型函数

### 基本泛型函数

```typescript
// 泛型函数
function identity<T>(arg: T): T {
  return arg;
}

const num = identity(42); // 推断 T 为 number
const str = identity('hello'); // 推断 T 为 string

// 显式指定类型
const result = identity<number>(42);
```

### 泛型约束

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

## 函数重载

### 基本函数重载

```typescript
// 函数重载
function add(a: number, b: number): number;
function add(a: string, b: string): string;
function add(a: any, b: any): any {
  return a + b;
}

const num = add(1, 2); // number
const str = add('hello', 'world'); // string
```

### 复杂重载

```typescript
// 复杂重载
function createElement(tag: 'a'): HTMLAnchorElement;
function createElement(tag: 'div'): HTMLDivElement;
function createElement(tag: 'span'): HTMLSpanElement;
function createElement(tag: string): HTMLElement {
  return document.createElement(tag);
}

const link = createElement('a'); // HTMLAnchorElement
const div = createElement('div'); // HTMLDivElement
```

## 实际应用示例

### 1. 事件处理

```typescript
type EventHandler<T = Event> = (event: T) => void;
type ClickHandler = EventHandler<MouseEvent>;
type FocusHandler = EventHandler<FocusEvent>;

const handleClick: ClickHandler = (event) => {
  console.log('Clicked', event.clientX, event.clientY);
};

const handleFocus: FocusHandler = (event) => {
  console.log('Focused', event.target);
};
```

### 2. 回调函数

```typescript
type Callback<T, R = void> = (error: Error | null, result?: T) => R;

function fetchData<T>(url: string, callback: Callback<T>): void {
  fetch(url)
    .then(response => response.json())
    .then(data => callback(null, data))
    .catch(error => callback(error));
}

fetchData<User>('/api/user', (error, user) => {
  if (error) {
    console.error(error);
  } else {
    console.log(user);
  }
});
```

### 3. 高阶函数

```typescript
type Mapper<T, U> = (item: T, index: number) => U;
type Predicate<T> = (item: T, index: number) => boolean;
type Reducer<T, U> = (accumulator: U, current: T, index: number) => U;

function map<T, U>(arr: T[], mapper: Mapper<T, U>): U[] {
  return arr.map(mapper);
}

function filter<T>(arr: T[], predicate: Predicate<T>): T[] {
  return arr.filter(predicate);
}

function reduce<T, U>(arr: T[], reducer: Reducer<T, U>, initial: U): U {
  return arr.reduce(reducer, initial);
}

const numbers = [1, 2, 3, 4, 5];
const doubled = map(numbers, n => n * 2);
const evens = filter(numbers, n => n % 2 === 0);
const sum = reduce(numbers, (acc, n) => acc + n, 0);
```

### 4. 异步函数

```typescript
type AsyncFunction<T, R> = (arg: T) => Promise<R>;

const fetchUser: AsyncFunction<number, User> = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

// 并行执行
async function fetchAllUsers(ids: number[]): Promise<User[]> {
  const promises = ids.map(id => fetchUser(id));
  return Promise.all(promises);
}
```

## 最佳实践

1. **使用显式返回类型**：对于公共 API
2. **使用泛型**：提高函数复用性
3. **使用可选参数**：而不是函数重载
4. **使用类型推断**：对于简单函数

```typescript
// 好的做法
function identity<T>(arg: T): T {
  return arg;
}

// 避免
function identity(arg: any): any {
  return arg;
}
```

## 参考

- [TypeScript Handbook - Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)
