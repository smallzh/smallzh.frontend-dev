# 基本类型（Basic Types）

## 概述

TypeScript 提供了丰富的基本类型，用于定义变量、参数和返回值的类型。

## 原始类型

### 字符串（string）

```typescript
// 字符串类型
let name: string = 'John';
let greeting: string = `Hello, ${name}!`;

// 模板字符串
const multiLine: string = `
  This is
  a multi-line
  string.
`;
```

### 数字（number）

```typescript
// 数字类型（整数和浮点数）
let integer: number = 42;
let float: number = 3.14;
let hex: number = 0xff;
let binary: number = 0b1010;
let octal: number = 0o744;

// 特殊数字值
let infinity: number = Infinity;
let negativeInfinity: number = -Infinity;
let notANumber: number = NaN;
```

### 布尔值（boolean）

```typescript
// 布尔类型
let isActive: boolean = true;
let isDeleted: boolean = false;

// 布尔表达式
let hasPermission: boolean = user.role === 'admin';
let isValid: boolean = age >= 18 && age <= 65;
```

### null 和 undefined

```typescript
// null 和 undefined 类型
let nullValue: null = null;
let undefinedValue: undefined = undefined;

// 在严格模式下，null 和 undefined 不能赋值给其他类型
let str: string = 'hello';
// str = null; // 错误
// str = undefined; // 错误

// 联合类型允许 null 和 undefined
let nullableString: string | null = 'hello';
nullableString = null; // 正确

let optionalString: string | undefined = 'hello';
optionalString = undefined; // 正确
```

### Symbol

```typescript
// Symbol 类型
let sym1: symbol = Symbol();
let sym2: symbol = Symbol('description');

// 作为对象属性键
const obj = {
  [sym1]: 'value1',
  [sym2]: 'value2'
};

// 全局 Symbol
let globalSym: symbol = Symbol.for('app.key');
```

### BigInt

```typescript
// BigInt 类型（ES2020）
let bigInt1: bigint = 100n;
let bigInt2: bigint = BigInt(100);

// 不能与 number 混合运算
let num: number = 42;
// let result = num + bigInt1; // 错误

// BigInt 运算
let sum: bigint = bigInt1 + bigInt2;
let product: bigint = bigInt1 * bigInt2;
```

## 数组类型

### 基本语法

```typescript
// 数组类型
let numbers: number[] = [1, 2, 3, 4, 5];
let strings: string[] = ['hello', 'world'];
let booleans: boolean[] = [true, false, true];

// 泛型语法
let numbers2: Array<number> = [1, 2, 3, 4, 5];
let strings2: Array<string> = ['hello', 'world'];

// 只读数组
let readonlyNumbers: readonly number[] = [1, 2, 3];
// readonlyNumbers.push(4); // 错误
// readonlyNumbers[0] = 10; // 错误
```

### 多维数组

```typescript
// 二维数组
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// 三维数组
let cube: number[][][] = [[[1, 2], [3, 4]], [[5, 6], [7, 8]]];
```

## 元组类型

### 基本语法

```typescript
// 元组类型（固定长度和类型的数组）
let tuple: [string, number] = ['John', 25];
// tuple = [25, 'John']; // 错误：类型顺序不匹配
// tuple = ['John', 25, true]; // 错误：长度不匹配

// 访问元组元素
let name: string = tuple[0]; // 'John'
let age: number = tuple[1]; // 25
```

### 可选元素

```typescript
// 可选元素（放在最后）
let optionalTuple: [string, number, boolean?] = ['John', 25];
let optionalTuple2: [string, number, boolean?] = ['John', 25, true];
```

### 剩余元素

```typescript
// 剩余元素
let restTuple: [string, ...number[]] = ['John', 1, 2, 3, 4, 5];
let restTuple2: [string, number, ...boolean[]] = ['John', 25, true, false, true];
```

### 命名元组

```typescript
// 命名元组（TypeScript 4.0+）
type User = [name: string, age: number, email: string];

const user: User = ['John', 25, 'john@example.com'];

// 解构时可以使用名称
const [userName, userAge, userEmail] = user;
```

## 枚举类型

### 数字枚举

```typescript
// 数字枚举
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}

let direction: Direction = Direction.Up;
console.log(direction); // 0
console.log(Direction[0]); // 'Up'

// 自定义起始值
enum StatusCode {
  OK = 200,
  NotFound = 404,
  ServerError = 500
}
```

### 字符串枚举

```typescript
// 字符串枚举
enum Color {
  Red = 'RED',
  Green = 'GREEN',
  Blue = 'BLUE'
}

let color: Color = Color.Red;
console.log(color); // 'RED'
```

### 常量枚举

```typescript
// 常量枚举（编译时会被内联）
const enum Direction {
  Up,
  Down,
  Left,
  Right
}

let direction: Direction = Direction.Up;
// 编译后：let direction = 0;
```

## 对象类型

### 基本语法

```typescript
// 对象类型
let user: { name: string; age: number } = {
  name: 'John',
  age: 25
};

// 可选属性
let user2: { name: string; age?: number } = {
  name: 'Jane'
};

// 只读属性
let user3: { readonly name: string; age: number } = {
  name: 'Bob',
  age: 30
};
// user3.name = 'Alice'; // 错误
```

### 索引签名

```typescript
// 索引签名
let dictionary: { [key: string]: number } = {
  'one': 1,
  'two': 2,
  'three': 3
};

// 混合类型
let mixed: { 
  name: string; 
  [key: string]: string | number 
} = {
  name: 'John',
  age: 25,
  email: 'john@example.com'
};
```

## 函数类型

### 基本语法

```typescript
// 函数类型
let add: (a: number, b: number) => number;
add = (a, b) => a + b;

// 使用类型别名
type AddFunction = (a: number, b: number) => number;
let add2: AddFunction = (a, b) => a + b;

// 使用接口
interface MathOperation {
  (a: number, b: number): number;
}

let multiply: MathOperation = (a, b) => a * b;
```

### 可选参数和默认参数

```typescript
// 可选参数
function greet(name: string, greeting?: string): string {
  return `${greeting || 'Hello'}, ${name}!`;
}

// 默认参数
function greet2(name: string, greeting: string = 'Hello'): string {
  return `${greeting}, ${name}!`;
}
```

### 剩余参数

```typescript
// 剩余参数
function sum(...numbers: number[]): number {
  return numbers.reduce((total, num) => total + num, 0);
}

// 混合参数
function log(message: string, ...args: any[]): void {
  console.log(message, ...args);
}
```

## 类型守卫

### typeof 类型守卫

```typescript
function processValue(value: string | number): string {
  if (typeof value === 'string') {
    return value.toUpperCase(); // value 是 string 类型
  }
  
  return value.toFixed(2); // value 是 number 类型
}
```

### instanceof 类型守卫

```typescript
class Dog {
  bark() { console.log('Woof!'); }
}

class Cat {
  meow() { console.log('Meow!'); }
}

function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    animal.bark(); // animal 是 Dog 类型
  } else {
    animal.meow(); // animal 是 Cat 类型
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
    animal.swim(); // animal 是 Fish 类型
  } else {
    animal.fly(); // animal 是 Bird 类型
  }
}
```

## 实际应用示例

### 1. 表单数据类型

```typescript
type FormData = {
  username: string;
  email: string;
  password: string;
  confirmPassword: string;
  age?: number;
  terms: boolean;
};

function validateForm(data: FormData): string[] {
  const errors: string[] = [];
  
  if (!data.username || data.username.length < 3) {
    errors.push('Username must be at least 3 characters');
  }
  
  if (!data.email || !data.email.includes('@')) {
    errors.push('Invalid email address');
  }
  
  if (data.password !== data.confirmPassword) {
    errors.push('Passwords do not match');
  }
  
  if (!data.terms) {
    errors.push('You must accept the terms');
  }
  
  return errors;
}
```

### 2. API 响应类型

```typescript
type ApiResponse<T> = {
  success: boolean;
  data: T;
  error?: string;
  timestamp: number;
};

type User = {
  id: number;
  name: string;
  email: string;
};

type UserResponse = ApiResponse<User>;
type UsersResponse = ApiResponse<User[]>;

async function fetchUser(id: number): Promise<UserResponse> {
  try {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();
    
    return {
      success: true,
      data,
      timestamp: Date.now()
    };
  } catch (error) {
    return {
      success: false,
      data: null as any,
      error: error instanceof Error ? error.message : 'Unknown error',
      timestamp: Date.now()
    };
  }
}
```

### 3. 状态管理类型

```typescript
type LoadingState = {
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

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

function handleState<T>(state: AsyncState<T>): void {
  switch (state.status) {
    case 'loading':
      console.log('Loading...');
      break;
    case 'success':
      console.log('Data:', state.data);
      break;
    case 'error':
      console.log('Error:', state.error);
      break;
  }
}
```

## 最佳实践

1. **使用类型推断**：让 TypeScript 自动推断类型
2. **避免使用 any**：尽量使用具体类型或 unknown
3. **使用联合类型**：表示多种可能的值
4. **使用类型守卫**：安全地收窄类型
5. **使用 readonly**：保护不可变数据

```typescript
// 好的做法
function processArray(arr: readonly number[]): number[] {
  return arr.filter(n => n > 0);
}

// 避免
function processArrayBad(arr: any[]): any[] {
  return arr.filter(n => n > 0);
}
```

## 参考

- [TypeScript Handbook - Basic Types](https://www.typescriptlang.org/docs/handbook/basic-types.html)
