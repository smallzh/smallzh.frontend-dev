# 变量声明（let/const）

## 概述

ES6 引入了 `let` 和 `const` 关键字，用于声明变量，解决了 `var` 关键字的一些问题。

## let

### 基本用法

```javascript
let name = 'John';
let age = 25;
let isStudent = true;
```

### 块级作用域

`let` 声明的变量只在块级作用域内有效：

```javascript
if (true) {
  let x = 10;
  console.log(x); // 10
}
console.log(x); // ReferenceError: x is not defined
```

### 不存在变量提升

```javascript
console.log(y); // ReferenceError: y is not defined
let y = 5;
```

### 暂时性死区（TDZ）

在块级作用域内，使用 `let` 声明变量之前，该变量不可用：

```javascript
function example() {
  console.log(z); // ReferenceError: z is not defined
  let z = 10;
}
```

### 不能重复声明

```javascript
let a = 1;
let a = 2; // SyntaxError: Identifier 'a' has already been declared
```

### 经典问题：循环中的闭包

```javascript
// 使用 var 的问题
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 输出：3, 3, 3

// 使用 let 解决
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 输出：0, 1, 2
```

## const

### 基本用法

`const` 用于声明常量，一旦声明，值不能改变：

```javascript
const PI = 3.14159;
const MAX_SIZE = 100;
```

### 必须初始化

```javascript
const x; // SyntaxError: Missing initializer in const declaration
```

### 不能重新赋值

```javascript
const PI = 3.14159;
PI = 3.14; // TypeError: Assignment to constant variable
```

### 对象和数组的 const

`const` 保证的是变量指向的内存地址不变，而不是值不变：

```javascript
const person = { name: 'John', age: 25 };
person.age = 26; // 允许，修改对象属性
person = {}; // TypeError: Assignment to constant variable

const numbers = [1, 2, 3];
numbers.push(4); // 允许，修改数组内容
numbers = []; // TypeError: Assignment to constant variable
```

## 最佳实践

1. **默认使用 `const`**：如果变量不会重新赋值，使用 `const`
2. **需要重新赋值时使用 `let`**：例如循环计数器
3. **避免使用 `var`**：`var` 存在变量提升和作用域问题

```javascript
// 好的做法
const API_URL = 'https://api.example.com';
const users = ['Alice', 'Bob'];

let currentPage = 1;
let isLoading = false;

// 避免的做法
var oldStyle = 'should avoid';
```

## 与 var 的区别

| 特性 | var | let | const |
|------|-----|-----|-------|
| 作用域 | 函数作用域 | 块级作用域 | 块级作用域 |
| 变量提升 | 是 | 否 | 否 |
| 重复声明 | 允许 | 不允许 | 不允许 |
| 重新赋值 | 允许 | 允许 | 不允许 |

## 参考

- [MDN let](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)
- [MDN const](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/const)
