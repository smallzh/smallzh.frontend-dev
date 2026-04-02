# 类型断言（Type Assertion）

## 概述

类型断言是一种告诉 TypeScript 编译器"我知道这个值的类型比 TypeScript 推断的更具体"的方式。

## 基本语法

### 尖括号语法

```typescript
// 尖括号语法
let someValue: any = 'hello';
let strLength: number = (<string>someValue).length;

// 在 JSX 中不能使用尖括号语法
// let strLength = (<string>someValue).length; // JSX 中会报错
```

### as 语法

```typescript
// as 语法（推荐）
let someValue: any = 'hello';
let strLength: number = (someValue as string).length;

// 在 JSX 中使用 as 语法
let strLength2 = (someValue as string).length;
```

## 常见使用场景

### 1. 将 any 转换为具体类型

```typescript
// 从 API 获取数据
const response: any = await fetch('/api/user');
const user = response as User;

// DOM 操作
const element = document.getElementById('myElement') as HTMLInputElement;
console.log(element.value);
```

### 2. 将联合类型转换为具体类型

```typescript
// 联合类型断言
function formatValue(value: string | number): string {
  if (typeof value === 'string') {
    return (value as string).toUpperCase();
  }
  return (value as number).toFixed(2);
}
```

### 3. 将父类转换为子类

```typescript
class Animal {
  name: string;
}

class Dog extends Animal {
  bark(): void {
    console.log('Woof!');
  }
}

const animal: Animal = new Dog();
const dog = animal as Dog;
dog.bark();
```

## 非空断言

### 基本用法

```typescript
// 非空断言操作符 !
function getLength(str: string | null | undefined): number {
  // 告诉 TypeScript str 不是 null 或 undefined
  return str!.length;
}

// DOM 操作
const element = document.getElementById('myElement')!;
console.log(element.innerHTML);
```

### 谨慎使用

```typescript
// 可能导致运行时错误
function badExample(str: string | null): number {
  return str!.length; // 如果 str 是 null，会抛出错误
}

// 更安全的方式
function goodExample(str: string | null): number {
  return str?.length ?? 0; // 使用可选链和空值合并
}
```

## const 断言

### 基本用法

```typescript
// const 断言
let str = 'hello' as const; // 类型为 'hello'，而不是 string
let num = 42 as const; // 类型为 42，而不是 number
let bool = true as const; // 类型为 true，而不是 boolean

// 数组 const 断言
let arr = [1, 2, 3] as const; // 类型为 readonly [1, 2, 3]
// arr.push(4); // 错误：只读数组

// 对象 const 断言
let obj = { name: 'John', age: 25 } as const;
// obj.name = 'Jane'; // 错误：只读属性
```

### 实际应用

```typescript
// 配置对象
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
} as const;

// 状态类型
const Status = {
  Active: 'ACTIVE',
  Inactive: 'INACTIVE',
  Pending: 'PENDING'
} as const;

type Status = typeof Status[keyof typeof Status];
// 'ACTIVE' | 'INACTIVE' | 'PENDING'
```

## 类型守卫

### typeof 类型守卫

```typescript
function processValue(value: string | number): string {
  if (typeof value === 'string') {
    // 此处 TypeScript 知道 value 是 string 类型
    return value.toUpperCase();
  }
  // 此处 TypeScript 知道 value 是 number 类型
  return value.toFixed(2);
}
```

### instanceof 类型守卫

```typescript
class Dog {
  bark(): void {
    console.log('Woof!');
  }
}

class Cat {
  meow(): void {
    console.log('Meow!');
  }
}

function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    animal.bark(); // TypeScript 知道 animal 是 Dog
  } else {
    animal.meow(); // TypeScript 知道 animal 是 Cat
  }
}
```

### in 类型守卫

```typescript
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function move(animal: Fish | Bird): void {
  if ('swim' in animal) {
    animal.swim(); // TypeScript 知道 animal 是 Fish
  } else {
    animal.fly(); // TypeScript 知道 animal 是 Bird
  }
}
```

### 自定义类型守卫

```typescript
// 自定义类型守卫函数
interface Cat {
  meow(): void;
}

interface Dog {
  bark(): void;
}

function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}

function makeSound(animal: Cat | Dog): void {
  if (isCat(animal)) {
    animal.meow(); // TypeScript 知道 animal 是 Cat
  } else {
    animal.bark(); // TypeScript 知道 animal 是 Dog
  }
}
```

## 实际应用示例

### 1. DOM 操作

```typescript
// 获取元素
const input = document.getElementById('username') as HTMLInputElement;
const button = document.querySelector('.submit-btn') as HTMLButtonElement;

// 类型守卫
function handleElement(element: HTMLElement): void {
  if (element instanceof HTMLInputElement) {
    console.log('Input value:', element.value);
  } else if (element instanceof HTMLButtonElement) {
    console.log('Button text:', element.textContent);
  }
}
```

### 2. API 响应处理

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

interface ApiResponse<T> {
  data: T;
  status: number;
}

async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const data: ApiResponse<any> = await response.json();
  
  // 类型断言
  return data.data as User;
}

// 类型守卫
function isUser(obj: any): obj is User {
  return (
    typeof obj === 'object' &&
    typeof obj.id === 'number' &&
    typeof obj.name === 'string' &&
    typeof obj.email === 'string'
  );
}
```

### 3. 配置处理

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  debug: boolean;
}

function loadConfig(): Config {
  const rawConfig = JSON.parse(localStorage.getItem('config') || '{}');
  
  // 类型守卫验证
  if (
    typeof rawConfig.apiUrl === 'string' &&
    typeof rawConfig.timeout === 'number' &&
    typeof rawConfig.debug === 'boolean'
  ) {
    return rawConfig as Config;
  }
  
  throw new Error('Invalid config format');
}
```

## 注意事项

### 1. 避免过度使用类型断言

```typescript
// 不推荐
const value: any = 'hello';
const length: number = (value as string).length; // 不安全

// 推荐
const value: string | number = 'hello';
if (typeof value === 'string') {
  const length: number = value.length; // 安全
}
```

### 2. 使用类型守卫代替类型断言

```typescript
// 不推荐
function process(value: string | number) {
  const str = value as string; // 不安全
  console.log(str.toUpperCase());
}

// 推荐
function process(value: string | number) {
  if (typeof value === 'string') {
    console.log(value.toUpperCase());
  }
}
```

### 3. 使用 const 断言保护常量

```typescript
// 不推荐
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

config.apiUrl = 'https://other.com'; // 可以修改

// 推荐
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
} as const;

// config.apiUrl = 'https://other.com'; // 错误：只读
```

## 最佳实践

1. **优先使用类型守卫**：更安全
2. **使用 as 语法**：避免 JSX 冲突
3. **使用 const 断言**：保护常量
4. **避免不必要的类型断言**：让 TypeScript 推断

```typescript
// 好的做法
function process(value: string | number): string {
  if (typeof value === 'string') {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}

// 避免
function process(value: string | number): string {
  return (value as string).toUpperCase(); // 可能运行时错误
}
```

## 参考

- [TypeScript Handbook - Type Assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)
