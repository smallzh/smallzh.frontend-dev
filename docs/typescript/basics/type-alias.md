# 类型别名（Type Alias）

## 概述

类型别名使用 `type` 关键字为类型创建新名称，可以用于任何类型，包括原始类型、联合类型、元组等。

## 基本语法

### 简单类型别名

```typescript
// 原始类型别名
type ID = string | number;
type Name = string;
type Age = number;

// 使用类型别名
let userId: ID = 123;
userId = 'abc';

let userName: Name = 'John';
let userAge: Age = 25;
```

### 对象类型别名

```typescript
// 对象类型别名
type User = {
  id: number;
  name: string;
  email: string;
};

type Product = {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
};

const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};
```

### 函数类型别名

```typescript
// 函数类型别名
type Add = (a: number, b: number) => number;
type Callback = (data: string) => void;
type Predicate<T> = (item: T) => boolean;

const add: Add = (a, b) => a + b;
const callback: Callback = (data) => console.log(data);
const isEven: Predicate<number> = (n) => n % 2 === 0;
```

## 联合类型别名

```typescript
// 联合类型
type StringOrNumber = string | number;
type Status = 'active' | 'inactive' | 'pending';
type Result<T> = { success: true; data: T } | { success: false; error: string };

function processValue(value: StringOrNumber): string {
  if (typeof value === 'string') {
    return value.toUpperCase();
  }
  return value.toString();
}

function handleResult<T>(result: Result<T>): T | null {
  if (result.success) {
    return result.data;
  }
  console.error(result.error);
  return null;
}
```

## 交叉类型别名

```typescript
// 交叉类型
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

## 泛型类型别名

```typescript
// 泛型类型别名
type ApiResponse<T> = {
  data: T;
  status: number;
  message: string;
};

type User = { id: number; name: string };
type UserResponse = ApiResponse<User>;
type UsersResponse = ApiResponse<User[]>;

type Container<T> = {
  value: T;
  getValue: () => T;
  setValue: (value: T) => void;
};

// 使用泛型类型别名
const userResponse: UserResponse = {
  data: { id: 1, name: 'John' },
  status: 200,
  message: 'Success'
};
```

## 条件类型别名

```typescript
// 条件类型
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false

// 更复杂的条件类型
type NonNullable<T> = T extends null | undefined ? never : T;

type C = NonNullable<string | null>; // string
type D = NonNullable<number | undefined>; // number
```

## 映射类型别名

```typescript
// 映射类型
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Optional<T> = {
  [P in keyof T]?: T[P];
};

type User = {
  id: number;
  name: string;
  email: string;
};

type ReadonlyUser = Readonly<User>;
type OptionalUser = Optional<User>;
```

## 实际应用示例

### 1. 状态管理

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

### 2. 表单类型

```typescript
type FormField<T> = {
  value: T;
  error?: string;
  touched: boolean;
};

type LoginForm = {
  username: FormField<string>;
  password: FormField<string>;
  remember: FormField<boolean>;
};

type FormValidation<T> = {
  [K in keyof T]: T[K] extends FormField<infer U> ? (value: U) => string | undefined : never;
};
```

### 3. 事件处理

```typescript
type EventType = 'click' | 'focus' | 'blur' | 'submit';

type EventHandler<T extends EventType = 'click'> = 
  T extends 'click' ? (event: MouseEvent) => void :
  T extends 'focus' | 'blur' ? (event: FocusEvent) => void :
  T extends 'submit' ? (event: SubmitEvent) => void :
  never;

type EventHandlers = {
  [K in EventType]?: EventHandler<K>;
};
```

## 类型别名 vs 接口

### 相似之处

```typescript
// 两者都可以定义对象类型
interface UserInterface {
  id: number;
  name: string;
}

type UserType = {
  id: number;
  name: string;
};
```

### 类型别名的优势

```typescript
// 1. 联合类型
type StringOrNumber = string | number;

// 2. 元组类型
type Tuple = [string, number, boolean];

// 3. 原始类型别名
type ID = string | number;

// 4. 条件类型
type IsString<T> = T extends string ? true : false;

// 5. 映射类型
type Readonly<T> = { readonly [P in keyof T]: T[P] };
```

### 接口的优势

```typescript
// 1. 扩展
interface User {
  id: number;
  name: string;
}

interface Employee extends User {
  employeeId: number;
}

// 2. 声明合并
interface User {
  id: number;
}

interface User {
  name: string;
}

// 合并后：User 有 id 和 name 属性
```

## 最佳实践

1. **使用类型别名定义联合类型和元组**
2. **使用接口定义对象类型**
3. **使用泛型类型别名提高复用性**
4. **避免过度使用类型别名**

```typescript
// 好的做法
type ID = string | number;
type Status = 'active' | 'inactive' | 'pending';
type ApiResponse<T> = { data: T; status: number };

interface User {
  id: ID;
  name: string;
  status: Status;
}

// 避免
type User = {
  id: ID;
  name: string;
  status: Status;
}; // 接口更适合对象类型
```

## 参考

- [TypeScript Handbook - Type Aliases](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
