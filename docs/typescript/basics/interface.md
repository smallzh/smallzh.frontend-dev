# 接口（Interface）

## 概述

接口是 TypeScript 的核心特性之一，用于定义对象的结构和类型契约。

## 基本语法

### 定义接口

```typescript
// 定义接口
interface User {
  id: number;
  name: string;
  email: string;
}

// 使用接口
const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};
```

### 可选属性

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string; // 可选属性
  age?: number;    // 可选属性
}

const user1: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};

const user2: User = {
  id: 2,
  name: 'Jane',
  email: 'jane@example.com',
  avatar: 'https://example.com/avatar.jpg',
  age: 25
};
```

### 只读属性

```typescript
interface User {
  readonly id: number;
  name: string;
  email: string;
}

const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};

user.name = 'Jane'; // 正确
// user.id = 2; // 错误：只读属性
```

## 方法定义

### 对象方法

```typescript
interface User {
  id: number;
  name: string;
  greet(): string;
  setName(name: string): void;
}

const user: User = {
  id: 1,
  name: 'John',
  greet() {
    return `Hello, ${this.name}!`;
  },
  setName(name: string) {
    this.name = name;
  }
};
```

### 函数类型属性

```typescript
interface MathOperation {
  (a: number, b: number): number;
}

const add: MathOperation = (a, b) => a + b;
const multiply: MathOperation = (a, b) => a * b;

interface Config {
  onSuccess: (data: any) => void;
  onError: (error: Error) => void;
}
```

## 索引签名

### 数字索引

```typescript
interface StringArray {
  [index: number]: string;
}

const array: StringArray = ['hello', 'world'];
console.log(array[0]); // 'hello'
```

### 字符串索引

```typescript
interface Dictionary {
  [key: string]: number;
}

const dict: Dictionary = {
  apple: 1,
  banana: 2,
  orange: 3
};

interface User {
  id: number;
  name: string;
  [key: string]: string | number;
}

const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  age: 25
};
```

## 接口继承

### 单继承

```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee extends Person {
  employeeId: number;
  department: string;
}

const employee: Employee = {
  name: 'John',
  age: 25,
  employeeId: 123,
  department: 'Engineering'
};
```

### 多继承

```typescript
interface Printable {
  print(): void;
}

interface Loggable {
  log(): void;
}

interface Document extends Printable, Loggable {
  content: string;
}

const doc: Document = {
  content: 'Hello',
  print() {
    console.log(this.content);
  },
  log() {
    console.log('Document logged');
  }
};
```

## 泛型接口

### 基本泛型接口

```typescript
interface Repository<T> {
  getById(id: number): T;
  getAll(): T[];
  save(item: T): void;
  delete(id: number): void;
}

interface User {
  id: number;
  name: string;
}

const userRepo: Repository<User> = {
  getById(id: number): User {
    return { id, name: 'John' };
  },
  getAll(): User[] {
    return [{ id: 1, name: 'John' }];
  },
  save(user: User): void {
    console.log('Saving user', user);
  },
  delete(id: number): void {
    console.log('Deleting user', id);
  }
};
```

### 约束泛型接口

```typescript
interface Entity {
  id: number;
}

interface Repository<T extends Entity> {
  getById(id: number): T | null;
  save(item: T): void;
}
```

## 实际应用示例

### 1. API 响应接口

```typescript
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
  timestamp: number;
}

interface User {
  id: number;
  name: string;
  email: string;
}

type UserResponse = ApiResponse<User>;
type UsersResponse = ApiResponse<User[]>;

const response: UserResponse = {
  success: true,
  data: {
    id: 1,
    name: 'John',
    email: 'john@example.com'
  },
  timestamp: Date.now()
};
```

### 2. 事件处理接口

```typescript
interface EventHandler<T = any> {
  (event: T): void;
}

interface EventMap {
  click: MouseEvent;
  focus: FocusEvent;
  blur: FocusEvent;
  submit: SubmitEvent;
}

class EventEmitter {
  private handlers: Partial<Record<keyof EventMap, EventHandler[]>> = {};

  on<K extends keyof EventMap>(event: K, handler: EventHandler<EventMap[K]>): void {
    if (!this.handlers[event]) {
      this.handlers[event] = [];
    }
    this.handlers[event]!.push(handler);
  }

  emit<K extends keyof EventMap>(event: K, data: EventMap[K]): void {
    this.handlers[event]?.forEach(handler => handler(data));
  }
}
```

### 3. 配置接口

```typescript
interface DatabaseConfig {
  host: string;
  port: number;
  database: string;
  username: string;
  password: string;
  ssl?: boolean;
  pool?: {
    min: number;
    max: number;
  };
}

interface AppConfig {
  database: DatabaseConfig;
  redis?: {
    host: string;
    port: number;
  };
  logging: {
    level: 'debug' | 'info' | 'warn' | 'error';
    file?: string;
  };
}

const config: AppConfig = {
  database: {
    host: 'localhost',
    port: 5432,
    database: 'myapp',
    username: 'admin',
    password: 'secret'
  },
  logging: {
    level: 'info'
  }
};
```

## 最佳实践

1. **使用接口定义对象类型**：比类型别名更清晰
2. **使用可选属性**：提高灵活性
3. **使用只读属性**：保护不可变数据
4. **使用泛型接口**：提高复用性

```typescript
// 好的做法
interface User {
  readonly id: number;
  name: string;
  email: string;
  avatar?: string;
}

// 避免
type User = {
  id: number;
  name: string;
  email: string;
  avatar: string | undefined;
};
```

## 参考

- [TypeScript Handbook - Interfaces](https://www.typescriptlang.org/docs/handbook/2/objects.html)
