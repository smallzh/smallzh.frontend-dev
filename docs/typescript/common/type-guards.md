# 类型守卫（Type Guards）

## 概述

类型守卫是一种运行时检查，用于缩小变量的类型范围，使 TypeScript 能够更精确地推断类型。

## typeof 类型守卫

### 基本用法

```typescript
// typeof 类型守卫
function processValue(value: string | number): string {
  if (typeof value === 'string') {
    // 此处 TypeScript 知道 value 是 string 类型
    return value.toUpperCase();
  }
  // 此处 TypeScript 知道 value 是 number 类型
  return value.toFixed(2);
}

// 检查多种类型
function handle(value: string | number | boolean): void {
  if (typeof value === 'string') {
    console.log(value.length);
  } else if (typeof value === 'number') {
    console.log(value.toFixed(2));
  } else {
    console.log(value ? 'true' : 'false');
  }
}
```

## instanceof 类型守卫

### 基本用法

```typescript
// instanceof 类型守卫
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

### 检查内置类型

```typescript
// 检查内置类型
function formatDate(value: string | Date): string {
  if (value instanceof Date) {
    return value.toISOString();
  }
  return new Date(value).toISOString();
}

// 检查错误类型
function handleError(error: Error | string): void {
  if (error instanceof Error) {
    console.error(error.message);
    console.error(error.stack);
  } else {
    console.error(error);
  }
}
```

## in 类型守卫

### 基本用法

```typescript
// in 类型守卫
interface Fish {
  swim: () => void;
  name: string;
}

interface Bird {
  fly: () => void;
  name: string;
}

function move(animal: Fish | Bird): void {
  if ('swim' in animal) {
    animal.swim(); // TypeScript 知道 animal 是 Fish
  } else {
    animal.fly(); // TypeScript 知道 animal 是 Bird
  }
}
```

### 检查可选属性

```typescript
// 检查可选属性
interface User {
  id: number;
  name: string;
  email?: string;
  phone?: string;
}

function getContact(user: User): string {
  if ('email' in user && user.email) {
    return user.email;
  }
  if ('phone' in user && user.phone) {
    return user.phone;
  }
  return 'No contact info';
}
```

## 自定义类型守卫

### 基本语法

```typescript
// 自定义类型守卫函数
function isString(value: any): value is string {
  return typeof value === 'string';
}

function isNumber(value: any): value is number {
  return typeof value === 'number';
}

// 使用
function process(value: string | number): void {
  if (isString(value)) {
    console.log(value.toUpperCase());
  } else {
    console.log(value.toFixed(2));
  }
}
```

### 检查对象类型

```typescript
// 检查对象类型
interface User {
  id: number;
  name: string;
  email: string;
}

interface Admin {
  id: number;
  name: string;
  permissions: string[];
}

function isUser(obj: any): obj is User {
  return (
    typeof obj === 'object' &&
    typeof obj.id === 'number' &&
    typeof obj.name === 'string' &&
    typeof obj.email === 'string'
  );
}

function isAdmin(obj: any): obj is Admin {
  return (
    typeof obj === 'object' &&
    typeof obj.id === 'number' &&
    typeof obj.name === 'string' &&
    Array.isArray(obj.permissions)
  );
}
```

### 检查联合类型

```typescript
// 检查联合类型
type StringOrNumber = string | number;

function isString(value: StringOrNumber): value is string {
  return typeof value === 'string';
}

function process(value: StringOrNumber): string {
  if (isString(value)) {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}
```

## 实际应用示例

### 1. API 响应处理

```typescript
interface SuccessResponse<T> {
  success: true;
  data: T;
}

interface ErrorResponse {
  success: false;
  error: string;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccess<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
  return response.success;
}

function handleResponse<T>(response: ApiResponse<T>): T | null {
  if (isSuccess(response)) {
    return response.data;
  }
  console.error(response.error);
  return null;
}
```

### 2. 表单验证

```typescript
interface ValidField {
  isValid: true;
  value: string;
}

interface InvalidField {
  isValid: false;
  error: string;
}

type FieldValidation = ValidField | InvalidField;

function isValidField(field: FieldValidation): field is ValidField {
  return field.isValid;
}

function processForm(fields: FieldValidation[]): string[] {
  return fields
    .filter(isValidField)
    .map(field => field.value);
}
```

### 3. 状态管理

```typescript
interface LoadingState {
  status: 'loading';
}

interface SuccessState<T> {
  status: 'success';
  data: T;
}

interface ErrorState {
  status: 'error';
  error: string;
}

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

function isSuccess<T>(state: AsyncState<T>): state is SuccessState<T> {
  return state.status === 'success';
}

function isError<T>(state: AsyncState<T>): state is ErrorState {
  return state.status === 'error';
}

function handleState<T>(state: AsyncState<T>): T | null {
  if (isSuccess(state)) {
    return state.data;
  }
  if (isError(state)) {
    console.error(state.error);
  }
  return null;
}
```

### 4. DOM 元素检查

```typescript
function isInputElement(element: HTMLElement): element is HTMLInputElement {
  return element instanceof HTMLInputElement;
}

function isButtonElement(element: HTMLElement): element is HTMLButtonElement {
  return element instanceof HTMLButtonElement;
}

function handleElement(element: HTMLElement): void {
  if (isInputElement(element)) {
    console.log('Input value:', element.value);
  } else if (isButtonElement(element)) {
    console.log('Button text:', element.textContent);
  }
}
```

## 类型守卫的组合

### 组合多个守卫

```typescript
// 组合多个类型守卫
interface Dog {
  bark: () => void;
}

interface Cat {
  meow: () => void;
}

interface Bird {
  fly: () => void;
}

type Animal = Dog | Cat | Bird;

function isDog(animal: Animal): animal is Dog {
  return 'bark' in animal;
}

function isCat(animal: Animal): animal is Cat {
  return 'meow' in animal;
}

function isBird(animal: Animal): animal is Bird {
  return 'fly' in animal;
}

function handleAnimal(animal: Animal): void {
  if (isDog(animal)) {
    animal.bark();
  } else if (isCat(animal)) {
    animal.meow();
  } else if (isBird(animal)) {
    animal.fly();
  }
}
```

## 最佳实践

1. **优先使用 typeof 和 instanceof**
2. **自定义类型守卫用于复杂检查**
3. **守卫函数返回类型使用 `value is Type`**
4. **避免在守卫函数中修改值**

```typescript
// 好的做法
function isUser(obj: any): obj is User {
  return (
    typeof obj === 'object' &&
    typeof obj.id === 'number' &&
    typeof obj.name === 'string'
  );
}

// 避免
function isUser(obj: any): boolean {
  return (
    typeof obj === 'object' &&
    typeof obj.id === 'number' &&
    typeof obj.name === 'string'
  );
}
```

## 参考

- [TypeScript Handbook - Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
