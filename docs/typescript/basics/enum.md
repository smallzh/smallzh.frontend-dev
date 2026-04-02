# 枚举（Enum）

## 概述

枚举是一种特殊的类型，用于定义一组命名的常量值。

## 数字枚举

### 基本语法

```typescript
// 数字枚举（默认从 0 开始）
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}

// 使用枚举
let direction: Direction = Direction.Up;
console.log(direction); // 0
console.log(Direction.Up); // 0
console.log(Direction[0]); // 'Up'
```

### 自定义起始值

```typescript
// 自定义起始值
enum StatusCode {
  OK = 200,
  Created = 201,
  BadRequest = 400,
  NotFound = 404,
  ServerError = 500
}

console.log(StatusCode.OK); // 200
console.log(StatusCode.NotFound); // 404
```

### 自动递增

```typescript
// 后续值会自动递增
enum Status {
  Active = 1,
  Inactive,    // 2
  Pending,     // 3
  Suspended    // 4
}

// 混合定义
enum Mixed {
  A,           // 0
  B = 5,
  C,           // 6
  D = 10,
  E            // 11
}
```

## 字符串枚举

### 基本语法

```typescript
// 字符串枚举
enum Color {
  Red = 'RED',
  Green = 'GREEN',
  Blue = 'BLUE'
}

let color: Color = Color.Red;
console.log(color); // 'RED'
console.log(Color.Green); // 'GREEN'

// 字符串枚举不能反向映射
// console.log(Color['RED']); // undefined
```

### 混合枚举

```typescript
// 数字和字符串混合枚举
enum Status {
  Active = 'ACTIVE',
  Inactive = 'INACTIVE',
  Pending = 1,
  Processing = 2
}

console.log(Status.Active); // 'ACTIVE'
console.log(Status.Pending); // 1
```

## 常量枚举

### 基本语法

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

### 常量枚举的优势

```typescript
// 普通枚举会生成对象
enum RegularEnum {
  A,
  B,
  C
}
// 编译后会生成 IIFE 代码

// 常量枚举直接内联
const enum ConstEnum {
  A,
  B,
  C
}
// 编译后直接使用数字值

// 性能更好
const value = ConstEnum.A; // 编译为：const value = 0;
```

## 枚举类型

### 枚举成员类型

```typescript
// 每个枚举成员都是独立的类型
enum Status {
  Active,
  Inactive
}

let status: Status.Active = Status.Active;
// status = Status.Inactive; // 错误
```

### 联合枚举类型

```typescript
// 枚举类型是所有成员的联合
enum Direction {
  Up,
  Down,
  Left,
  Right
}

function move(direction: Direction): void {
  switch (direction) {
    case Direction.Up:
      console.log('Moving up');
      break;
    case Direction.Down:
      console.log('Moving down');
      break;
    // 必须处理所有情况
  }
}
```

## 实际应用示例

### 1. HTTP 状态码

```typescript
enum HttpStatus {
  OK = 200,
  Created = 201,
  NoContent = 204,
  BadRequest = 400,
  Unauthorized = 401,
  Forbidden = 403,
  NotFound = 404,
  InternalServerError = 500
}

function handleResponse(status: HttpStatus): void {
  if (status >= 200 && status < 300) {
    console.log('Success');
  } else if (status >= 400 && status < 500) {
    console.log('Client error');
  } else {
    console.log('Server error');
  }
}
```

### 2. 用户角色

```typescript
enum UserRole {
  Admin = 'ADMIN',
  Editor = 'EDITOR',
  Viewer = 'VIEWER',
  Guest = 'GUEST'
}

interface User {
  id: number;
  name: string;
  role: UserRole;
}

function hasPermission(user: User, action: string): boolean {
  switch (user.role) {
    case UserRole.Admin:
      return true;
    case UserRole.Editor:
      return ['read', 'write', 'edit'].includes(action);
    case UserRole.Viewer:
      return ['read'].includes(action);
    case UserRole.Guest:
      return false;
    default:
      return false;
  }
}
```

### 3. 订单状态

```typescript
enum OrderStatus {
  Pending = 'PENDING',
  Confirmed = 'CONFIRMED',
  Shipped = 'SHIPPED',
  Delivered = 'DELIVERED',
  Cancelled = 'CANCELLED'
}

interface Order {
  id: number;
  status: OrderStatus;
  createdAt: Date;
}

function canCancel(order: Order): boolean {
  return [OrderStatus.Pending, OrderStatus.Confirmed].includes(order.status);
}

function canTrack(order: Order): boolean {
  return [OrderStatus.Shipped, OrderStatus.Delivered].includes(order.status);
}
```

### 4. 环境配置

```typescript
enum Environment {
  Development = 'development',
  Staging = 'staging',
  Production = 'production'
}

const config = {
  [Environment.Development]: {
    apiUrl: 'http://localhost:3000',
    debug: true
  },
  [Environment.Staging]: {
    apiUrl: 'https://staging-api.example.com',
    debug: true
  },
  [Environment.Production]: {
    apiUrl: 'https://api.example.com',
    debug: false
  }
};

function getConfig(env: Environment) {
  return config[env];
}
```

### 5. 事件类型

```typescript
enum EventType {
  Click = 'CLICK',
  Focus = 'FOCUS',
  Blur = 'BLUR',
  Submit = 'SUBMIT',
  Change = 'CHANGE'
}

type EventHandler = (event: Event) => void;

const handlers: Record<EventType, EventHandler[]> = {
  [EventType.Click]: [],
  [EventType.Focus]: [],
  [EventType.Blur]: [],
  [EventType.Submit]: [],
  [EventType.Change]: []
};

function addEventListener(type: EventType, handler: EventHandler): void {
  handlers[type].push(handler);
}
```

## 枚举的替代方案

### const 对象

```typescript
// 使用 const 对象代替枚举
const Direction = {
  Up: 0,
  Down: 1,
  Left: 2,
  Right: 3
} as const;

type Direction = typeof Direction[keyof typeof Direction];

let direction: Direction = Direction.Up;
```

### 联合类型

```typescript
// 使用联合类型代替字符串枚举
type Status = 'active' | 'inactive' | 'pending';

function setStatus(status: Status): void {
  console.log(status);
}

setStatus('active'); // 正确
// setStatus('unknown'); // 错误
```

### const 断言

```typescript
// 使用 const 断言创建不可变对象
const ROLES = {
  Admin: 'ADMIN',
  Editor: 'EDITOR',
  Viewer: 'VIEWER'
} as const;

type Role = typeof ROLES[keyof typeof ROLES];

function checkRole(role: Role): boolean {
  return role === ROLES.Admin;
}
```

## 最佳实践

1. **使用字符串枚举**：可读性更好
2. **使用常量枚举**：性能更好
3. **避免数字枚举**：除非有明确的数字含义
4. **考虑 const 对象**：更轻量的替代方案

```typescript
// 好的做法
enum Status {
  Active = 'ACTIVE',
  Inactive = 'INACTIVE'
}

// 避免
enum Status {
  Active,    // 0 是什么意思？
  Inactive   // 1 是什么意思？
}
```

## 参考

- [TypeScript Handbook - Enums](https://www.typescriptlang.org/docs/handbook/enums.html)
