# 索引类型（Index Types）

## 概述

索引类型允许我们使用索引来描述对象的属性类型，包括索引签名和 keyof 操作符。

## 索引签名

### 基本语法

```typescript
// 字符串索引签名
interface StringDictionary {
  [key: string]: string;
}

const dict: StringDictionary = {
  name: 'John',
  city: 'New York',
  country: 'USA'
};

// 数字索引签名
interface NumberDictionary {
  [index: number]: string;
}

const array: NumberDictionary = ['hello', 'world'];
console.log(array[0]); // 'hello'
```

### 混合索引签名

```typescript
// 混合索引签名
interface MixedDictionary {
  [key: string]: string | number;
  length: number; // 必须是索引签名类型的子类型
}

const mixed: MixedDictionary = {
  length: 5,
  name: 'John',
  age: 25
};
```

### 只读索引签名

```typescript
// 只读索引签名
interface ReadonlyDictionary {
  readonly [key: string]: string;
}

const dict: ReadonlyDictionary = {
  name: 'John',
  city: 'New York'
};

// dict.name = 'Jane'; // 错误：只读属性
```

## keyof 操作符

### 基本用法

```typescript
// keyof 获取对象的所有键
interface User {
  id: number;
  name: string;
  email: string;
}

type UserKeys = keyof User; // 'id' | 'name' | 'email'

// 使用 keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};

const name = getProperty(user, 'name'); // string
const id = getProperty(user, 'id'); // number
// getProperty(user, 'email'); // 错误：email 不是 User 的键
```

### 索引访问类型

```typescript
// 索引访问类型
interface User {
  id: number;
  name: string;
  address: {
    city: string;
    country: string;
  };
}

type UserId = User['id']; // number
type UserName = User['name']; // string
type UserAddress = User['address']; // { city: string; country: string; }

// 嵌套访问
type UserCity = User['address']['city']; // string

// 联合类型
type UserBasic = User['id' | 'name']; // number | string
```

## 实际应用示例

### 1. 类型安全的对象访问

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  for (const key of keys) {
    result[key] = obj[key];
  }
  return result;
}

const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  age: 25
};

const basicInfo = pick(user, ['id', 'name']); // { id: number; name: string }
```

### 2. 映射类型

```typescript
// 映射类型
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

interface User {
  id: number;
  name: string;
  email?: string;
}

type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
type RequiredUser = Required<User>;
```

### 3. 条件类型与 keyof

```typescript
// 条件类型与 keyof
type ExtractStringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
}[keyof T];

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type StringKeys = ExtractStringKeys<User>; // 'name' | 'email'
```

### 4. 记录类型

```typescript
// Record 类型
type UserRoles = Record<number, string>;

const roles: UserRoles = {
  1: 'admin',
  2: 'user',
  3: 'guest'
};

// 自定义 Record
type MyRecord<K extends keyof any, V> = {
  [P in K]: V;
};

type Status = 'active' | 'inactive' | 'pending';
type StatusMap = MyRecord<Status, boolean>;

const statusMap: StatusMap = {
  active: true,
  inactive: false,
  pending: true
};
```

## 最佳实践

1. **使用 keyof 获取对象键的联合类型**
2. **使用索引访问类型获取属性类型**
3. **使用映射类型创建新类型**
4. **结合泛型使用索引类型**

```typescript
// 好的做法
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// 避免
function getProperty(obj: any, key: string): any {
  return obj[key];
}
```

## 参考

- [TypeScript Handbook - Index Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html)
