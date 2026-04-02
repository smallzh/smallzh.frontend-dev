# 类（Class）

## 概述

ES6 引入了类（class）语法，提供了更清晰、更面向对象的方式来创建对象和处理继承。

## 基本语法

### 类定义

```javascript
class Person {
  // 构造函数
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // 实例方法
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  // 获取器
  get info() {
    return `${this.name}, ${this.age} years old`;
  }
  
  // 设置器
  set nickname(value) {
    this._nickname = value;
  }
  
  get nickname() {
    return this._nickname || this.name;
  }
}

const person = new Person('John', 25);
console.log(person.greet()); // "Hello, I'm John"
console.log(person.info); // "John, 25 years old"

person.nickname = 'Johnny';
console.log(person.nickname); // "Johnny"
```

### 类表达式

```javascript
// 匿名类表达式
const Person = class {
  constructor(name) {
    this.name = name;
  }
};

// 命名类表达式
const Person = class MyClass {
  constructor(name) {
    this.name = name;
  }
  
  getClassName() {
    return MyClass.name;
  }
};

const person = new Person('John');
console.log(person.getClassName()); // "MyClass"
```

## 静态成员

### 静态方法

```javascript
class MathUtils {
  static add(a, b) {
    return a + b;
  }
  
  static subtract(a, b) {
    return a - b;
  }
  
  static multiply(a, b) {
    return a * b;
  }
}

console.log(MathUtils.add(5, 3)); // 8
console.log(MathUtils.subtract(5, 3)); // 2
console.log(MathUtils.multiply(5, 3)); // 15
```

### 静态属性

```javascript
class Person {
  static count = 0;
  static species = 'Homo sapiens';
  
  constructor(name) {
    this.name = name;
    Person.count++;
  }
  
  static getCount() {
    return Person.count;
  }
}

const p1 = new Person('John');
const p2 = new Person('Jane');

console.log(Person.count); // 2
console.log(Person.getCount()); // 2
console.log(Person.species); // "Homo sapiens"
```

## 继承

### 基本继承

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name} makes a noise.`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // 调用父类构造函数
    this.breed = breed;
  }
  
  speak() {
    return `${this.name} barks.`;
  }
  
  fetch() {
    return `${this.name} fetches the ball.`;
  }
}

const dog = new Dog('Rex', 'German Shepherd');
console.log(dog.speak()); // "Rex barks."
console.log(dog.fetch()); // "Rex fetches the ball."
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
```

### super 关键字

```javascript
class Vehicle {
  constructor(type, speed) {
    this.type = type;
    this.speed = speed;
  }
  
  describe() {
    return `This is a ${this.type} moving at ${this.speed} mph.`;
  }
}

class Car extends Vehicle {
  constructor(speed, brand) {
    super('car', speed);
    this.brand = brand;
  }
  
  describe() {
    const baseDescription = super.describe();
    return `${baseDescription} It's a ${this.brand}.`;
  }
}

const car = new Car(60, 'Toyota');
console.log(car.describe());
// "This is a car moving at 60 mph. It's a Toyota."
```

### 继承静态成员

```javascript
class Shape {
  static create(type, ...args) {
    switch (type) {
      case 'circle':
        return new Circle(...args);
      case 'rectangle':
        return new Rectangle(...args);
    }
  }
  
  static getShapeCount() {
    return this.count || 0;
  }
}

class Circle extends Shape {
  static count = 0;
  
  constructor(radius) {
    super();
    this.radius = radius;
    Circle.count++;
  }
  
  get area() {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  static count = 0;
  
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
    Rectangle.count++;
  }
  
  get area() {
    return this.width * this.height;
  }
}

const circle = Shape.create('circle', 5);
const rectangle = Shape.create('rectangle', 4, 6);

console.log(circle.area); // 78.53981633974483
console.log(rectangle.area); // 24
console.log(Circle.getShapeCount()); // 1
console.log(Rectangle.getShapeCount()); // 1
```

## 私有成员

### 私有字段

```javascript
class BankAccount {
  #balance = 0; // 私有字段
  
  constructor(initialBalance) {
    this.#balance = initialBalance;
  }
  
  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
      return true;
    }
    return false;
  }
  
  withdraw(amount) {
    if (amount > 0 && amount <= this.#balance) {
      this.#balance -= amount;
      return true;
    }
    return false;
  }
  
  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount(100);
account.deposit(50);
account.withdraw(30);
console.log(account.getBalance()); // 120
// console.log(account.#balance); // SyntaxError: Private field
```

### 私有方法

```javascript
class User {
  #password;
  
  constructor(username, password) {
    this.username = username;
    this.#password = password;
  }
  
  #hashPassword(password) {
    // 简单的哈希示例
    return password.split('').reverse().join('');
  }
  
  #validatePassword(password) {
    return this.#password === this.#hashPassword(password);
  }
  
  authenticate(password) {
    return this.#validatePassword(password);
  }
  
  changePassword(oldPassword, newPassword) {
    if (this.#validatePassword(oldPassword)) {
      this.#password = this.#hashPassword(newPassword);
      return true;
    }
    return false;
  }
}

const user = new User('john', 'secret');
console.log(user.authenticate('secret')); // true
// user.#hashPassword('test'); // SyntaxError: Private method
```

## 抽象类模拟

### 使用 new.target

```javascript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('Shape is an abstract class');
    }
  }
  
  area() {
    throw new Error('area() must be implemented');
  }
  
  perimeter() {
    throw new Error('perimeter() must be implemented');
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }
  
  area() {
    return Math.PI * this.radius ** 2;
  }
  
  perimeter() {
    return 2 * Math.PI * this.radius;
  }
}

// const shape = new Shape(); // Error: Shape is an abstract class
const circle = new Circle(5);
console.log(circle.area()); // 78.53981633974483
```

## Mixin 模式

### 基本 Mixin

```javascript
const Serializable = (superclass) => class extends superclass {
  serialize() {
    return JSON.stringify(this);
  }
  
  static deserialize(json) {
    return Object.assign(new this(), JSON.parse(json));
  }
};

const Validatable = (superclass) => class extends superclass {
  validate() {
    for (const key of Object.keys(this)) {
      if (this[key] === null || this[key] === undefined) {
        return false;
      }
    }
    return true;
  }
};

class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

// 组合多个 mixin
class EnhancedUser extends Serializable(Validatable(User)) {
  constructor(name, email) {
    super(name, email);
  }
}

const user = new EnhancedUser('John', 'john@example.com');
console.log(user.validate()); // true
console.log(user.serialize()); // {"name":"John","email":"john@example.com"}

const deserialized = EnhancedUser.deserialize('{"name":"Jane","email":"jane@example.com"}');
console.log(deserialized.name); // "Jane"
```

## 实际应用示例

### 1. 事件发射器

```javascript
class EventEmitter {
  #events = new Map();
  
  on(event, callback) {
    if (!this.#events.has(event)) {
      this.#events.set(event, new Set());
    }
    this.#events.get(event).add(callback);
    return this;
  }
  
  off(event, callback) {
    if (this.#events.has(event)) {
      this.#events.get(event).delete(callback);
    }
    return this;
  }
  
  emit(event, ...args) {
    if (this.#events.has(event)) {
      for (const callback of this.#events.get(event)) {
        callback.apply(this, args);
      }
    }
    return this;
  }
  
  once(event, callback) {
    const wrapper = (...args) => {
      this.off(event, wrapper);
      callback.apply(this, args);
    };
    return this.on(event, wrapper);
  }
}

const emitter = new EventEmitter();
emitter.on('data', (value) => console.log('Received:', value));
emitter.emit('data', 'Hello'); // "Received: Hello"
```

### 2. 状态管理

```javascript
class Store {
  #state;
  #listeners = new Set();
  
  constructor(initialState = {}) {
    this.#state = { ...initialState };
  }
  
  get state() {
    return { ...this.#state };
  }
  
  setState(updater) {
    const newState = typeof updater === 'function' 
      ? updater(this.#state) 
      : updater;
    
    this.#state = { ...this.#state, ...newState };
    this.#notify();
  }
  
  subscribe(listener) {
    this.#listeners.add(listener);
    return () => this.#listeners.delete(listener);
  }
  
  #notify() {
    for (const listener of this.#listeners) {
      listener(this.state);
    }
  }
}

const store = new Store({ count: 0 });

const unsubscribe = store.subscribe((state) => {
  console.log('State changed:', state);
});

store.setState({ count: 1 }); // "State changed: { count: 1 }"
store.setState((state) => ({ count: state.count + 1 })); // "State changed: { count: 2 }"
```

### 3. 数据验证

```javascript
class Validator {
  #rules = new Map();
  
  addRule(field, validator, message) {
    if (!this.#rules.has(field)) {
      this.#rules.set(field, []);
    }
    this.#rules.get(field).push({ validator, message });
    return this;
  }
  
  validate(data) {
    const errors = [];
    
    for (const [field, rules] of this.#rules) {
      for (const { validator, message } of rules) {
        if (!validator(data[field])) {
          errors.push({ field, message });
        }
      }
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
}

const userValidator = new Validator()
  .addRule('name', (v) => v && v.length >= 2, 'Name must be at least 2 characters')
  .addRule('email', (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v), 'Invalid email')
  .addRule('age', (v) => v >= 18 && v <= 120, 'Age must be between 18 and 120');

const result = userValidator.validate({
  name: 'John',
  email: 'john@example.com',
  age: 25
});

console.log(result.valid); // true
```

## 最佳实践

1. **使用类而非构造函数**：更清晰的语法
2. **私有字段保护数据**：使用 `#` 语法
3. **静态方法用于工具函数**：不需要实例化
4. **继承时调用 super()**：正确初始化父类
5. **使用 getter/setter**：控制属性访问

```javascript
// 好的做法
class User {
  #email;
  
  constructor(name, email) {
    this.name = name;
    this.#email = email;
  }
  
  get email() {
    return this.#email;
  }
  
  set email(value) {
    if (!this.#isValidEmail(value)) {
      throw new Error('Invalid email');
    }
    this.#email = value;
  }
  
  #isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
  
  static createFromJSON(json) {
    const data = JSON.parse(json);
    return new User(data.name, data.email);
  }
}
```

## 参考

- [MDN Classes](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
