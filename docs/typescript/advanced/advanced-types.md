# 高级类型技巧（Advanced Type Techniques）

## 概述

TypeScript 提供了丰富的高级类型特性，用于创建灵活、类型安全的代码。

## 模板字面量类型

### 基本语法

```typescript
// 模板字面量类型
type EventName = `on${Capitalize<'click' | 'focus' | 'blur'>}`;
// 'onClick' | 'onFocus' | 'onBlur'

// 联合类型分发
type Color = 'red' | 'blue';
type Size = 'small' | 'large';
type Style = `${Size}-${Color}`;
// 'small-red' | 'small-blue' | 'large-red' | 'large-blue'
```

### 实际应用

```typescript
// CSS 类名生成
type Spacing = 'p' | 'm';
type Direction = 't' | 'r' | 'b' | 'l' | 'x' | 'y';
type Size = '1' | '2' | '3' | '4' | '5';

type SpacingClass = `${Spacing}${Direction}-${Size}`;
// 'p-1' | 'p-2' | 'm-1' | 'm-2' | ...

// 事件处理器
type EventHandler<T extends string> = `on${Capitalize<T>}`;

type ButtonEvents = EventHandler<'click' | 'hover' | 'focus'>;
// 'onClick' | 'onHover' | 'onFocus'
```

## 类型推断

### infer 关键字

```typescript
// 提取函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function createUser() {
  return { id: 1, name: 'John' };
}

type User = ReturnType<typeof createUser>;
// { id: number; name: string; }

// 提取 Promise 类型
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type A = UnwrapPromise<Promise<string>>; // string

// 提取数组元素类型
type ElementOf<T> = T extends (infer E)[] ? E : never;

type B = ElementOf<string[]>; // string
```

### 条件类型推断

```typescript
// 复杂类型推断
type FunctionParams<T> = T extends (...args: infer P) => any ? P : never;
type FunctionReturn<T> = T extends (...args: any[]) => infer R ? R : never;

function add(a: number, b: number): number {
  return a + b;
}

type AddParams = FunctionParams<typeof add>; // [number, number]
type AddReturn = FunctionReturn<typeof add>; // number
```

## 类型收窄

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

interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}

type Shape = Square | Circle | Rectangle;

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'square':
      return shape.size ** 2;
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'rectangle':
      return shape.width * shape.height;
  }
}
```

### 守卫类型

```typescript
// 自定义类型守卫
interface Cat {
  meow: () => void;
}

interface Dog {
  bark: () => void;
}

function isCat(animal: Cat | Dog): animal is Cat {
  return 'meow' in animal;
}

function makeSound(animal: Cat | Dog): void {
  if (isCat(animal)) {
    animal.meow();
  } else {
    animal.bark();
  }
}
```

## 类型编程

### 递归类型

```typescript
// 递归类型
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
```

### 类型级计算

```typescript
// 类型级计算
type Length<T extends any[]> = T['length'];

type Zero = Length<[]>; // 0
type One = Length<[1]>; // 1
type Two = Length<[1, 2]>; // 2

// 元组操作
type Push<T extends any[], V> = [...T, V];
type Pop<T extends any[]> = T extends [...infer Rest, any] ? Rest : never;

type A = Push<[1, 2], 3>; // [1, 2, 3]
type B = Pop<[1, 2, 3]>; // [1, 2]
```

## 实际应用示例

### 1. 路由类型

```typescript
// 路由类型
type Route = '/' | '/about' | '/users' | '/users/:id';

type RouteParams<T extends string> = T extends `${string}:${infer Param}/${infer Rest}`
  ? Param | RouteParams<Rest>
  : T extends `${string}:${infer Param}`
  ? Param
  : never;

type UserRouteParams = RouteParams<'/users/:id'>; // 'id'
type ComplexParams = RouteParams<'/users/:id/posts/:postId'>; // 'id' | 'postId'
```

### 2. 状态机类型

```typescript
// 状态机类型
type State = 'idle' | 'loading' | 'success' | 'error';
type Event = 'FETCH' | 'RESOLVE' | 'REJECT' | 'RESET';

type Transition<S extends State, E extends Event> = 
  S extends 'idle' 
    ? E extends 'FETCH' ? 'loading' : never
    : S extends 'loading'
    ? E extends 'RESOLVE' ? 'success' : E extends 'REJECT' ? 'error' : never
    : S extends 'success' | 'error'
    ? E extends 'RESET' ? 'idle' : never
    : never;

type T1 = Transition<'idle', 'FETCH'>; // 'loading'
type T2 = Transition<'loading', 'RESOLVE'>; // 'success'
type T3 = Transition<'loading', 'REJECT'>; // 'error'
```

### 3. 表单类型

```typescript
// 表单类型
interface FormField<T> {
  value: T;
  error?: string;
  touched: boolean;
}

type Form<T> = {
  [K in keyof T]: FormField<T[K]>;
};

interface LoginForm {
  username: string;
  password: string;
  remember: boolean;
}

type LoginFormFields = Form<LoginForm>;
```

### 4. API 类型

```typescript
// API 类型
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';

type Endpoint = {
  [K in HttpMethod]?: {
    path: string;
    response: any;
  };
};

type Api = {
  '/users': {
    GET: {
      path: '/users';
      response: User[];
    };
    POST: {
      path: '/users';
      response: User;
    };
  };
  '/users/:id': {
    GET: {
      path: '/users/:id';
      response: User;
    };
    PUT: {
      path: '/users/:id';
      response: User;
    };
    DELETE: {
      path: '/users/:id';
      response: void;
    };
  };
};
```

## 最佳实践

1. **使用模板字面量类型**：生成字符串类型
2. **使用 infer 提取类型**：从复杂类型中提取部分
3. **使用递归类型**：处理嵌套结构
4. **避免过度复杂**：保持类型可读

```typescript
// 好的做法
type EventName<T extends string> = `on${Capitalize<T>}`;

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

- [TypeScript Handbook - Type Programming](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
