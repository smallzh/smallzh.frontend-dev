# 默认参数（Default Parameters）

## 概述

ES6 允许在函数参数中设置默认值，当调用函数时没有提供该参数或参数值为 `undefined` 时，使用默认值。

## 基本语法

### 传统方式

```javascript
// 传统方式：在函数体内设置默认值
function greet(name) {
  name = name || 'Guest';
  return `Hello, ${name}!`;
}

// 问题：当传入假值（0, '', false, null）时也会使用默认值
greet(0); // "Hello, Guest!"（错误）
greet(''); // "Hello, Guest!"（错误）
```

### ES6 默认参数

```javascript
function greet(name = 'Guest') {
  return `Hello, ${name}!`;
}

greet(); // "Hello, Guest!"
greet('John'); // "Hello, John!"
greet(undefined); // "Hello, Guest!"
greet(null); // "Hello, null!"（null 不会触发默认值）
greet(0); // "Hello, 0!"
greet(''); // "Hello, !"
```

## 参数默认值的触发条件

默认参数只在参数为 `undefined` 时触发：

```javascript
function test(value = 'default') {
  console.log(value);
}

test();          // "default"（未传参）
test(undefined); // "default"（显式传入 undefined）
test(null);      // null
test(0);         // 0
test(false);     // false
test('');        // ""（空字符串）
```

## 多个默认参数

```javascript
function createUser(
  name = 'Anonymous',
  age = 0,
  email = 'no-email@example.com'
) {
  return { name, age, email };
}

createUser(); 
// { name: 'Anonymous', age: 0, email: 'no-email@example.com' }

createUser('John', 25); 
// { name: 'John', age: 25, email: 'no-email@example.com' }

createUser('Jane', 30, 'jane@example.com'); 
// { name: 'Jane', age: 30, email: 'jane@example.com' }
```

## 默认参数的计算

默认参数可以是表达式或函数调用：

```javascript
// 表达式
function multiply(a, b = a * 2) {
  return a * b;
}

multiply(5);    // 50 (5 * 10)
multiply(5, 3); // 15 (5 * 3)

// 函数调用
function getDefaultName() {
  return 'Guest';
}

function greet(name = getDefaultName()) {
  return `Hello, ${name}!`;
}

greet(); // "Hello, Guest!"
```

## 默认参数与解构

### 对象解构中的默认值

```javascript
function createUser({ name = 'Anonymous', age = 0 } = {}) {
  return { name, age };
}

createUser(); // { name: 'Anonymous', age: 0 }
createUser({ name: 'John' }); // { name: 'John', age: 0 }
createUser({ age: 25 }); // { name: 'Anonymous', age: 25 }
```

### 数组解构中的默认值

```javascript
function processCoords([x = 0, y = 0] = []) {
  return { x, y };
}

processCoords(); // { x: 0, y: 0 }
processCoords([10]); // { x: 10, y: 0 }
processCoords([10, 20]); // { x: 10, y: 20 }
```

## 默认参数的作用域

默认参数有自己的作用域，位于函数体作用域之前：

```javascript
let x = 1;

function foo(x = x) {
  console.log(x);
}

foo(); // ReferenceError: x is not defined
// 默认参数 x = x 中，x 指向的是参数作用域中的 x，而不是外部的 x
```

```javascript
let x = 1;

function foo(a = x) {
  console.log(a);
}

foo(); // 1（正常工作）
```

## 默认参数的暂时性死区

```javascript
function foo(a = b, b = 1) {
  console.log(a, b);
}

foo(); // ReferenceError: b is not defined
// 参数 a 的默认值 b 在此时还未定义
```

## 实际应用示例

### 1. 配置对象处理

```javascript
function createServer({
  port = 3000,
  host = 'localhost',
  ssl = false,
  timeout = 5000,
  maxConnections = 100
} = {}) {
  console.log(`Server: ${host}:${port}`);
  console.log(`SSL: ${ssl}, Timeout: ${timeout}ms`);
  console.log(`Max connections: ${maxConnections}`);
}

createServer(); // 使用所有默认值
createServer({ port: 8080, ssl: true }); // 只覆盖部分配置
```

### 2. API 请求函数

```javascript
async function fetchUsers({
  page = 1,
  limit = 10,
  sortBy = 'createdAt',
  order = 'desc'
} = {}) {
  const params = new URLSearchParams({ page, limit, sortBy, order });
  const response = await fetch(`/api/users?${params}`);
  return response.json();
}

// 使用示例
fetchUsers(); // 默认参数
fetchUsers({ page: 2, limit: 20 }); // 自定义参数
```

### 3. 事件处理函数

```javascript
function handleEvent(
  event,
  callback = () => {},
  options = {}
) {
  const { 
    preventDefault = true, 
    stopPropagation = false 
  } = options;
  
  if (preventDefault) {
    event.preventDefault();
  }
  
  if (stopPropagation) {
    event.stopPropagation();
  }
  
  callback(event);
}

// 使用示例
button.addEventListener('click', (e) => {
  handleEvent(e, () => console.log('Clicked'));
});

link.addEventListener('click', (e) => {
  handleEvent(e, () => console.log('Link clicked'), {
    preventDefault: false
  });
});
```

### 4. 数据验证函数

```javascript
function validateUser({
  name = '',
  email = '',
  age = 0,
  required = ['name', 'email']
} = {}) {
  const errors = [];
  
  required.forEach(field => {
    if (!eval(field)) {
      errors.push(`${field} is required`);
    }
  });
  
  if (email && !email.includes('@')) {
    errors.push('Invalid email format');
  }
  
  if (age < 0 || age > 150) {
    errors.push('Invalid age');
  }
  
  return { valid: errors.length === 0, errors };
}
```

## 注意事项

### 1. 参数顺序

默认参数应该放在非默认参数之后：

```javascript
// 不推荐
function bad(a = 1, b) {
  return a + b;
}

// 推荐
function good(a, b = 1) {
  return a + b;
}
```

### 2. 性能考虑

默认参数表达式在每次调用时都会求值：

```javascript
function expensive() {
  console.log('Computing...');
  return 42;
}

function test(x = expensive()) {
  console.log(x);
}

test(); // "Computing..." "42"
test(); // "Computing..." "42"（每次都执行）
```

### 3. 与 arguments 对象的关系

```javascript
function test(a = 1) {
  console.log(arguments.length);
  console.log(a);
}

test();      // 0, 1
test(10);    // 1, 10
test(10, 20); // 2, 10
```

## 最佳实践

1. **使用默认参数替代传统方式**：更清晰、更安全
2. **为可选参数设置合理默认值**：提高函数的易用性
3. **避免副作用**：默认参数表达式应该是纯函数
4. **文档化参数**：清晰说明每个参数的作用和默认值

```javascript
// 好的做法
/**
 * 创建用户对象
 * @param {Object} options - 配置选项
 * @param {string} options.name - 用户名，默认 'Anonymous'
 * @param {number} options.age - 年龄，默认 0
 * @returns {Object} 用户对象
 */
function createUser({
  name = 'Anonymous',
  age = 0
} = {}) {
  return { name, age };
}
```

## 参考

- [MDN 默认参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Default_parameters)
