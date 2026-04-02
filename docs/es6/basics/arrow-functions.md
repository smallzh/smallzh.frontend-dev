# 箭头函数（Arrow Functions）

## 概述

箭头函数是 ES6 引入的函数简写语法，使用 `=>` 符号，使函数定义更简洁。

## 基本语法

### 传统函数 vs 箭头函数

```javascript
// 传统函数
function add(a, b) {
  return a + b;
}

// 箭头函数
const add = (a, b) => {
  return a + b;
};

// 简写形式（单行返回）
const add = (a, b) => a + b;

// 单个参数可以省略括号
const square = x => x * x;

// 无参数需要空括号
const greet = () => 'Hello!';
```

## 语法变体

### 1. 单行返回（隐式返回）

```javascript
const double = x => x * 2;
const isEven = n => n % 2 === 0;
const getFullName = (first, last) => `${first} ${last}`;
```

### 2. 返回对象字面量

```javascript
// 需要用括号包裹对象
const createUser = (name, age) => ({ name, age });

const user = createUser('John', 25);
console.log(user); // { name: 'John', age: 25 }
```

### 3. 多行函数体

```javascript
const processData = (data) => {
  const filtered = data.filter(item => item.active);
  const mapped = filtered.map(item => item.name);
  return mapped;
};
```

## 特性

### 1. 没有自己的 this

箭头函数不绑定自己的 `this`，而是继承外层作用域的 `this`：

```javascript
const person = {
  name: 'John',
  // 传统函数：this 指向 person
  greet: function() {
    console.log(`Hello, ${this.name}`);
  },
  // 箭头函数：this 继承外层作用域
  sayHi: () => {
    console.log(`Hi, ${this.name}`); // this.name 是 undefined
  }
};

person.greet(); // Hello, John
person.sayHi(); // Hi, undefined
```

### 2. 不能用作构造函数

```javascript
const Person = (name) => {
  this.name = name;
};

// TypeError: Person is not a constructor
const john = new Person('John');
```

### 3. 没有 arguments 对象

```javascript
const sum = () => {
  console.log(arguments); // ReferenceError: arguments is not defined
};

// 使用剩余参数代替
const sum = (...args) => {
  return args.reduce((a, b) => a + b, 0);
};
```

### 4. 不能用作 Generator 函数

```javascript
// 不能使用 yield 关键字
const gen = *() => { // SyntaxError
  yield 1;
};
```

## 使用场景

### 1. 回调函数

```javascript
// 数组方法
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);
const sum = numbers.reduce((acc, n) => acc + n, 0);

// 事件处理
button.addEventListener('click', () => {
  console.log('Clicked!');
});
```

### 2. Promise 链

```javascript
fetch('/api/users')
  .then(response => response.json())
  .then(users => users.map(u => u.name))
  .then(names => console.log(names))
  .catch(error => console.error(error));
```

### 3. 函数式编程

```javascript
const compose = (f, g) => x => f(g(x));
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

const double = x => x * 2;
const addOne = x => x + 1;
const square = x => x * x;

const transform = pipe(double, addOne, square);
console.log(transform(3)); // ((3 * 2) + 1)² = 49
```

## 注意事项

### 1. 不适合用作对象方法

```javascript
// 不推荐
const person = {
  name: 'John',
  greet: () => {
    console.log(this.name); // undefined
  }
};

// 推荐
const person = {
  name: 'John',
  greet() {
    console.log(this.name); // John
  }
};
```

### 2. 不适合用作原型方法

```javascript
// 不推荐
Animal.prototype.speak = () => {
  console.log(this.sound); // undefined
};

// 推荐
Animal.prototype.speak = function() {
  console.log(this.sound);
};
```

### 3. 不适合需要 this 动态绑定的场景

```javascript
// 不推荐
const button = {
  text: 'Click me',
  handleClick: () => {
    console.log(this.text); // undefined
  }
};

// 推荐
const button = {
  text: 'Click me',
  handleClick() {
    console.log(this.text); // Click me
  }
};
```

## 最佳实践

1. **优先使用箭头函数**：对于回调函数和短函数
2. **需要 this 绑定时使用传统函数**：对象方法、原型方法
3. **保持简洁**：单行函数使用简写形式
4. **避免嵌套**：复杂的箭头函数考虑使用传统函数

## 参考

- [MDN 箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
