# 类（Class）

## 概述

TypeScript 的类系统在 JavaScript 类的基础上增加了类型支持，包括访问修饰符、抽象类等特性。

## 基本语法

### 类定义

```typescript
class User {
  // 属性声明
  name: string;
  age: number;

  // 构造函数
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  // 方法
  greet(): string {
    return `Hello, ${this.name}!`;
  }
}

const user = new User('John', 25);
console.log(user.greet()); // 'Hello, John!'
```

### 属性初始化

```typescript
class User {
  // 直接初始化
  name: string = '';
  age: number = 0;
  isActive: boolean = true;

  // 可选属性
  email?: string;

  // 只读属性
  readonly id: number;

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }
}
```

## 访问修饰符

### public

```typescript
class User {
  public name: string;
  public age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  public greet(): string {
    return `Hello, ${this.name}!`;
  }
}

// public 是默认修饰符，可以省略
class User2 {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```

### private

```typescript
class BankAccount {
  private balance: number;

  constructor(initialBalance: number) {
    this.balance = initialBalance;
  }

  // 私有方法
  private validateAmount(amount: number): boolean {
    return amount > 0 && amount <= this.balance;
  }

  public deposit(amount: number): void {
    if (amount > 0) {
      this.balance += amount;
    }
  }

  public withdraw(amount: number): boolean {
    if (this.validateAmount(amount)) {
      this.balance -= amount;
      return true;
    }
    return false;
  }

  public getBalance(): number {
    return this.balance;
  }
}

const account = new BankAccount(1000);
account.deposit(500);
// account.balance; // 错误：balance 是私有属性
// account.validateAmount(100); // 错误：validateAmount 是私有方法
```

### protected

```typescript
class Animal {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }

  protected makeSound(): string {
    return '...';
  }
}

class Dog extends Animal {
  private breed: string;

  constructor(name: string, breed: string) {
    super(name);
    this.breed = breed;
  }

  public bark(): string {
    // 可以访问父类的 protected 成员
    return `${this.name} barks!`;
  }

  protected makeSound(): string {
    return 'Woof!';
  }
}

const dog = new Dog('Rex', 'German Shepherd');
// dog.name; // 错误：name 是 protected
// dog.makeSound(); // 错误：makeSound 是 protected
```

## 构造函数

### 参数属性

```typescript
// 参数属性简写
class User {
  constructor(
    public name: string,
    public age: number,
    private email: string,
    readonly id: number
  ) {
    // 不需要手动赋值
  }
}

const user = new User('John', 25, 'john@example.com', 1);
console.log(user.name); // 'John'
console.log(user.id); // 1
// user.email; // 错误：email 是私有属性
```

### 重载构造函数

```typescript
class User {
  name: string;
  age: number;
  email?: string;

  constructor(name: string, age: number);
  constructor(name: string, age: number, email: string);
  constructor(name: string, age: number, email?: string) {
    this.name = name;
    this.age = age;
    this.email = email;
  }
}

const user1 = new User('John', 25);
const user2 = new User('Jane', 30, 'jane@example.com');
```

## 继承

### 基本继承

```typescript
class Animal {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }

  public speak(): string {
    return `${this.name} makes a sound.`;
  }
}

class Dog extends Animal {
  private breed: string;

  constructor(name: string, breed: string) {
    super(name);
    this.breed = breed;
  }

  public speak(): string {
    return `${this.name} barks.`;
  }

  public fetch(): string {
    return `${this.name} fetches the ball.`;
  }
}

const dog = new Dog('Rex', 'German Shepherd');
console.log(dog.speak()); // 'Rex barks.'
console.log(dog.fetch()); // 'Rex fetches the ball.'
```

### super 关键字

```typescript
class Vehicle {
  protected speed: number;

  constructor(speed: number) {
    this.speed = speed;
  }

  public describe(): string {
    return `Moving at ${this.speed} mph.`;
  }
}

class Car extends Vehicle {
  private brand: string;

  constructor(speed: number, brand: string) {
    super(speed);
    this.brand = brand;
  }

  public describe(): string {
    // 调用父类方法
    return `${super.describe()} It's a ${this.brand}.`;
  }
}

const car = new Car(60, 'Toyota');
console.log(car.describe()); // 'Moving at 60 mph. It's a Toyota.'
```

## 静态成员

### 静态属性和方法

```typescript
class MathUtils {
  // 静态属性
  static PI = 3.14159;

  // 静态方法
  static add(a: number, b: number): number {
    return a + b;
  }

  static circleArea(radius: number): number {
    return MathUtils.PI * radius * radius;
  }
}

console.log(MathUtils.PI); // 3.14159
console.log(MathUtils.add(5, 3)); // 8
console.log(MathUtils.circleArea(5)); // 78.53975
```

## 抽象类

### 基本抽象类

```typescript
abstract class Shape {
  abstract area(): number;
  abstract perimeter(): number;

  // 具体方法
  describe(): string {
    return `Area: ${this.area()}, Perimeter: ${this.perimeter()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }

  perimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

class Rectangle extends Shape {
  constructor(private width: number, private height: number) {
    super();
  }

  area(): number {
    return this.width * this.height;
  }

  perimeter(): number {
    return 2 * (this.width + this.height);
  }
}

// const shape = new Shape(); // 错误：不能实例化抽象类
const circle = new Circle(5);
const rectangle = new Rectangle(4, 6);
```

## 接口实现

### 实现接口

```typescript
interface Printable {
  print(): void;
}

interface Loggable {
  log(): void;
}

class Document implements Printable, Loggable {
  constructor(private content: string) {}

  print(): void {
    console.log('Printing:', this.content);
  }

  log(): void {
    console.log('Logging:', this.content);
  }
}
```

## Getter 和 Setter

```typescript
class User {
  private _name: string;
  private _age: number;

  constructor(name: string, age: number) {
    this._name = name;
    this._age = age;
  }

  get name(): string {
    return this._name;
  }

  set name(value: string) {
    if (value.length < 2) {
      throw new Error('Name must be at least 2 characters');
    }
    this._name = value;
  }

  get age(): number {
    return this._age;
  }

  set age(value: number) {
    if (value < 0 || value > 150) {
      throw new Error('Invalid age');
    }
    this._age = value;
  }

  get isAdult(): boolean {
    return this._age >= 18;
  }
}

const user = new User('John', 25);
console.log(user.name); // 'John'
console.log(user.isAdult); // true
user.name = 'Jane'; // 正确
// user.name = 'A'; // 错误：Name must be at least 2 characters
```

## 实际应用示例

### 1. 事件发射器

```typescript
class EventEmitter {
  private events: Map<string, Set<Function>> = new Map();

  on(event: string, handler: Function): void {
    if (!this.events.has(event)) {
      this.events.set(event, new Set());
    }
    this.events.get(event)!.add(handler);
  }

  off(event: string, handler: Function): void {
    this.events.get(event)?.delete(handler);
  }

  emit(event: string, ...args: any[]): void {
    this.events.get(event)?.forEach(handler => handler(...args));
  }
}
```

### 2. 状态管理

```typescript
class Store<T> {
  private state: T;
  private listeners: Set<(state: T) => void> = new Set();

  constructor(initialState: T) {
    this.state = initialState;
  }

  getState(): T {
    return this.state;
  }

  setState(updater: T | ((state: T) => T)): void {
    this.state = typeof updater === 'function' 
      ? (updater as (state: T) => T)(this.state)
      : updater;
    this.notify();
  }

  subscribe(listener: (state: T) => void): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  private notify(): void {
    this.listeners.forEach(listener => listener(this.state));
  }
}
```

## 最佳实践

1. **使用访问修饰符**：保护内部状态
2. **使用接口实现**：定义契约
3. **使用抽象类**：定义公共行为
4. **使用参数属性**：简化构造函数

```typescript
// 好的做法
class User {
  constructor(
    public readonly id: number,
    public name: string,
    private email: string
  ) {}
}

// 避免
class User {
  id: number;
  name: string;
  email: string;

  constructor(id: number, name: string, email: string) {
    this.id = id;
    this.name = name;
    this.email = email;
  }
}
```

## 参考

- [TypeScript Handbook - Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)
