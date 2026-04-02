# 类型基础（Type Basics）

## 概述

TypeScript 的核心是类型系统，它允许我们在编写代码时定义变量、参数和返回值的类型。

## 类型注解

### 基本语法

```typescript
// 变量类型注解
let name: string = 'John';
let age: number = 25;
let isStudent: boolean = true;

// 函数参数类型注解
function greet(name: string): string {
  return `Hello, ${name}!`;
}

// 函数返回值类型注解
function add(a: number, b: number): number {
  return a + b;
}
```

### 类型推断

TypeScript 可以自动推断类型：

```typescript
// TypeScript 自动推断类型
let message = 'Hello'; // 推断为 string
let count = 42; // 推断为 number
let isActive = true; // 推断为 boolean

// 数组推断
let numbers = [1, 2, 3]; // 推断为 number[]
let mixed = [1, 'two', 3]; // 推断为 (string | number)[]

// 函数推断
function multiply(a: number, b: number) {
  return a * b; // 推断返回类型为 number
}
```

## 基本类型

### 原始类型

```typescript
// 字符串
let str1: string = 'hello';
let str2: string = "world";
let str3: string = `Hello, ${str1}!`;

// 数字
let num1: number = 42;
let num2: number = 3.14;
let num3: number = 0xff; // 十六进制
let num4: number = 0b1010; // 二进制
let num5: number = 0o744; // 八进制

// 布尔值
let bool1: boolean = true;
let bool2: boolean = false;

// null 和 undefined
let nullValue: null = null;
let undefinedValue: undefined = undefined;

// Symbol
let sym1: symbol = Symbol();
let sym2: symbol = Symbol('description');

// BigInt
let bigInt1: bigint = 100n;
let bigInt2: bigint = BigInt(100);
```

### any 类型

```typescript
// any 类型可以赋值给任何类型
let anyValue: any = 'hello';
anyValue = 42;
anyValue = true;
anyValue = { name: 'John' };

// 可以调用任何方法
anyValue.foo();
anyValue.bar();
```

### unknown 类型

```typescript
// unknown 类型比 any 更安全
let unknownValue: unknown = 'hello';

// 不能直接调用方法
// unknownValue.foo(); // 错误

// 需要类型检查
if (typeof unknownValue === 'string') {
  console.log(unknownValue.toUpperCase()); // 正确
}

// 可以赋值给 any 或 unknown
let a: any = unknownValue;
let b: unknown = unknownValue;
// let c: string = unknownValue; // 错误
```

### void 类型

```typescript
// void 表示没有返回值
function logMessage(message: string): void {
  console.log(message);
}

// void 类型的变量只能赋值为 undefined 或 null
let voidValue: void = undefined;
// voidValue = 'hello'; // 错误
```

### never 类型

```typescript
// never 表示永远不会返回的函数
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {
    // 无限循环
  }
}

// never 类型的变量不能赋值
let neverValue: never;
// neverValue = 'hello'; // 错误
// neverValue = 42; // 错误
```

## 类型断言

### 基本语法

```typescript
// 尖括号语法
let someValue: any = 'hello';
let strLength: number = (<string>someValue).length;

// as 语法
let someValue2: any = 'hello';
let strLength2: number = (someValue2 as string).length;
```

### 非空断言

```typescript
// 非空断言操作符 !
function getLength(str: string | null | undefined): number {
  // 告诉 TypeScript str 不是 null 或 undefined
  return str!.length;
}

// 使用可选链更安全
function getLengthSafe(str: string | null | undefined): number {
  return str?.length ?? 0;
}
```

## 联合类型

### 基本语法

```typescript
// 联合类型使用 | 分隔
let value: string | number;
value = 'hello';
value = 42;
// value = true; // 错误

// 函数参数联合类型
function formatValue(value: string | number): string {
  if (typeof value === 'string') {
    return value.toUpperCase();
  }
  return value.toString();
}
```

### 类型守卫

```typescript
function processValue(value: string | number): string {
  // typeof 类型守卫
  if (typeof value === 'string') {
    return value.toUpperCase();
  }
  
  // 此时 value 被收窄为 number 类型
  return value.toFixed(2);
}

// instanceof 类型守卫
class Dog {
  bark() { console.log('Woof!'); }
}

class Cat {
  meow() { console.log('Meow!'); }
}

function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

## 交叉类型

### 基本语法

```typescript
// 交叉类型使用 & 合并
type Person = {
  name: string;
  age: number;
};

type Employee = {
  employeeId: number;
  department: string;
};

type EmployeePerson = Person & Employee;

const employee: EmployeePerson = {
  name: 'John',
  age: 25,
  employeeId: 123,
  department: 'Engineering'
};
```

## 字面量类型

### 字符串字面量

```typescript
// 字符串字面量类型
type Direction = 'up' | 'down' | 'left' | 'right';

function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

move('up'); // 正确
move('down'); // 正确
// move('forward'); // 错误
```

### 数字字面量

```typescript
// 数字字面量类型
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

function rollDice(): DiceRoll {
  return Math.floor(Math.random() * 6) + 1 as DiceRoll;
}
```

### 布尔字面量

```typescript
// 布尔字面量类型
type True = true;
type False = false;

const alwaysTrue: true = true;
// const alwaysTrue2: true = false; // 错误
```

## 类型别名

### 基本语法

```typescript
// 类型别名
type StringOrNumber = string | number;
type Callback = (data: string) => void;
type User = {
  id: number;
  name: string;
  email: string;
};

// 使用类型别名
function processValue(value: StringOrNumber): void {
  console.log(value);
}

const callback: Callback = (data) => {
  console.log(data);
};

const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};
```

## 实际应用示例

### 1. API 响应类型

```typescript
type ApiResponse<T> = {
  data: T;
  status: number;
  message: string;
};

type User = {
  id: number;
  name: string;
  email: string;
};

type UserListResponse = ApiResponse<User[]>;
type UserResponse = ApiResponse<User>;

const usersResponse: UserListResponse = {
  data: [
    { id: 1, name: 'John', email: 'john@example.com' },
    { id: 2, name: 'Jane', email: 'jane@example.com' }
  ],
  status: 200,
  message: 'Success'
};
```

### 2. 事件处理

```typescript
type EventType = 'click' | 'focus' | 'blur' | 'submit';

type EventHandler = (event: Event) => void;

type EventHandlers = {
  [K in EventType]?: EventHandler;
};

const handlers: EventHandlers = {
  click: (event) => console.log('Clicked'),
  focus: (event) => console.log('Focused'),
  submit: (event) => console.log('Submitted')
};
```

### 3. 配置对象

```typescript
type DatabaseConfig = {
  host: string;
  port: number;
  database: string;
  username: string;
  password: string;
  ssl?: boolean;
  timeout?: number;
};

type AppConfig = {
  database: DatabaseConfig;
  redis?: {
    host: string;
    port: number;
  };
  logging: {
    level: 'debug' | 'info' | 'warn' | 'error';
    file?: string;
  };
};

const config: AppConfig = {
  database: {
    host: 'localhost',
    port: 5432,
    database: 'myapp',
    username: 'admin',
    password: 'secret',
    ssl: true
  },
  logging: {
    level: 'info',
    file: '/var/log/app.log'
  }
};
```

## 最佳实践

1. **使用类型推断**：让 TypeScript 自动推断类型
2. **避免使用 any**：尽量使用具体类型或 unknown
3. **使用联合类型**：表示多种可能的值
4. **使用类型别名**：提高代码可读性
5. **使用类型守卫**：安全地收窄类型

```typescript
// 好的做法
function processUser(user: User | null): void {
  if (user) {
    console.log(user.name);
  }
}

// 避免
function processUserBad(user: any): void {
  console.log(user.name); // 可能运行时错误
}
```

## 参考

- [TypeScript Handbook - Basic Types](https://www.typescriptlang.org/docs/handbook/basic-types.html)
