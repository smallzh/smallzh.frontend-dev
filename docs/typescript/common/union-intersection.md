# 联合类型和交叉类型（Union and Intersection Types）

## 概述

联合类型和交叉类型是 TypeScript 中组合类型的两种方式。联合类型使用 `|` 表示"或"关系，交叉类型使用 `&` 表示"与"关系。

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

### 字面量联合类型

```typescript
// 字符串字面量联合类型
type Direction = 'up' | 'down' | 'left' | 'right';
type Status = 'active' | 'inactive' | 'pending';

function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

move('up'); // 正确
move('down'); // 正确
// move('forward'); // 错误

// 数字字面量联合类型
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

// 布尔字面量联合类型
type TrueOrFalse = true | false;
```

### 可辨识联合类型

```typescript
// 可辨识联合类型
interface Circle {
  kind: 'circle';
  radius: number;
}

interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}

interface Triangle {
  kind: 'triangle';
  base: number;
  height: number;
}

type Shape = Circle | Rectangle | Triangle;

function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'rectangle':
      return shape.width * shape.height;
    case 'triangle':
      return (shape.base * shape.height) / 2;
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

### 合并接口

```typescript
// 合并接口
interface User {
  id: number;
  name: string;
}

interface Admin {
  permissions: string[];
}

type AdminUser = User & Admin;

const adminUser: AdminUser = {
  id: 1,
  name: 'John',
  permissions: ['read', 'write', 'delete']
};
```

## 联合类型和交叉类型的区别

### 联合类型：或关系

```typescript
// 联合类型：值可以是其中任意一种类型
let value: string | number;
value = 'hello';
value = 42;

// 函数返回联合类型
function getValue(): string | number {
  return Math.random() > 0.5 ? 'hello' : 42;
}
```

### 交叉类型：与关系

```typescript
// 交叉类型：值必须同时满足所有类型
type A = { a: string };
type B = { b: number };
type C = A & B; // { a: string; b: number }

const c: C = {
  a: 'hello',
  b: 42
};
```

## 实际应用示例

### 1. API 响应类型

```typescript
// 成功响应
interface SuccessResponse<T> {
  success: true;
  data: T;
}

// 错误响应
interface ErrorResponse {
  success: false;
  error: string;
}

// 联合类型
type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function handleResponse<T>(response: ApiResponse<T>): T | null {
  if (response.success) {
    return response.data;
  }
  console.error(response.error);
  return null;
}
```

### 2. 状态管理

```typescript
// 加载状态
interface LoadingState {
  status: 'loading';
}

// 成功状态
interface SuccessState<T> {
  status: 'success';
  data: T;
}

// 错误状态
interface ErrorState {
  status: 'error';
  error: string;
}

// 联合类型
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

### 3. 组件 Props

```typescript
// 基础 Props
interface BaseProps {
  className?: string;
  style?: React.CSSProperties;
}

// 按钮 Props
interface ButtonProps {
  onClick: () => void;
  disabled?: boolean;
}

// 输入框 Props
interface InputProps {
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
}

// 组合 Props
type ButtonComponentProps = BaseProps & ButtonProps;
type InputComponentProps = BaseProps & InputProps;
```

### 4. 配置对象

```typescript
// 基础配置
interface BaseConfig {
  debug: boolean;
  timeout: number;
}

// 数据库配置
interface DatabaseConfig {
  host: string;
  port: number;
  database: string;
}

// Redis 配置
interface RedisConfig {
  host: string;
  port: number;
}

// 应用配置
type AppConfig = BaseConfig & {
  database: DatabaseConfig;
  redis?: RedisConfig;
};

const config: AppConfig = {
  debug: true,
  timeout: 5000,
  database: {
    host: 'localhost',
    port: 5432,
    database: 'myapp'
  }
};
```

## 类型守卫与联合类型

### 类型守卫

```typescript
// 使用类型守卫处理联合类型
interface Dog {
  bark: () => void;
}

interface Cat {
  meow: () => void;
}

type Animal = Dog | Cat;

function isDog(animal: Animal): animal is Dog {
  return 'bark' in animal;
}

function makeSound(animal: Animal): void {
  if (isDog(animal)) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

### 可辨识联合类型

```typescript
// 可辨识联合类型
interface Square {
  kind: 'square';
  size: number;
}

interface Circle {
  kind: 'circle';
  radius: number;
}

type Shape = Square | Circle;

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'square':
      return shape.size ** 2;
    case 'circle':
      return Math.PI * shape.radius ** 2;
  }
}
```

## 泛型与联合类型/交叉类型

### 泛型联合类型

```typescript
// 泛型联合类型
type Result<T, E> = { success: true; value: T } | { success: false; error: E };

function parseJSON<T>(json: string): Result<T, Error> {
  try {
    const value = JSON.parse(json);
    return { success: true, value };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}

const result = parseJSON<User>('{"name":"John"}');
if (result.success) {
  console.log(result.value);
} else {
  console.error(result.error);
}
```

### 泛型交叉类型

```typescript
// 泛型交叉类型
type WithId<T> = T & { id: number };

interface User {
  name: string;
  email: string;
}

type UserWithId = WithId<User>;

const user: UserWithId = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};
```

## 最佳实践

1. **使用联合类型表示"或"关系**
2. **使用交叉类型表示"与"关系**
3. **使用可辨识联合类型处理复杂状态**
4. **使用类型守卫处理联合类型**

```typescript
// 好的做法
type Status = 'active' | 'inactive' | 'pending';

interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

// 避免
type Status = string; // 过于宽泛
```

## 参考

- [TypeScript Handbook - Union Types](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
- [TypeScript Handbook - Intersection Types](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
