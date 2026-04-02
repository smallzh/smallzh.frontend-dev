# 解构赋值（Destructuring）

## 概述

解构赋值是 ES6 引入的语法，允许从数组或对象中提取值，按照对应位置对变量赋值。

## 数组解构

### 基本用法

```javascript
// 传统方式
const arr = [1, 2, 3];
const a = arr[0];
const b = arr[1];
const c = arr[2];

// 解构赋值
const [x, y, z] = [1, 2, 3];
console.log(x, y, z); // 1, 2, 3
```

### 跳过元素

```javascript
const [first, , third] = [1, 2, 3];
console.log(first, third); // 1, 3

// 剩余元素
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]
```

### 默认值

```javascript
const [a = 1, b = 2] = [10];
console.log(a, b); // 10, 2

// 只有解构值为 undefined 时才使用默认值
const [x = 5] = [null];
console.log(x); // null
```

### 交换变量

```javascript
let a = 1;
let b = 2;

// 传统方式
let temp = a;
a = b;
b = temp;

// 解构赋值
[a, b] = [b, a];
console.log(a, b); // 2, 1
```

### 嵌套数组解构

```javascript
const matrix = [[1, 2], [3, 4]];
const [[a, b], [c, d]] = matrix;
console.log(a, b, c, d); // 1, 2, 3, 4

const [first, [second, third]] = [1, [2, 3]];
console.log(first, second, third); // 1, 2, 3
```

## 对象解构

### 基本用法

```javascript
// 传统方式
const person = { name: 'John', age: 25 };
const name = person.name;
const age = person.age;

// 解构赋值
const { name, age } = { name: 'John', age: 25 };
console.log(name, age); // John, 25
```

### 重命名变量

```javascript
const person = { name: 'John', age: 25 };

// 属性名: 变量名
const { name: userName, age: userAge } = person;
console.log(userName, userAge); // John, 25

// 简写形式（属性名和变量名相同）
const { name, age } = person;
```

### 默认值

```javascript
const person = { name: 'John' };
const { name, age = 30 } = person;
console.log(name, age); // John, 30

// 重命名 + 默认值
const { name: userName, age: userAge = 30 } = person;
console.log(userName, userAge); // John, 30
```

### 嵌套对象解构

```javascript
const user = {
  name: 'John',
  address: {
    city: 'New York',
    country: 'USA'
  }
};

const { name, address: { city, country } } = user;
console.log(name, city, country); // John, New York, USA

// 重命名嵌套属性
const { name, address: { city: userCity } } = user;
console.log(userCity); // New York
```

### 剩余属性

```javascript
const person = { name: 'John', age: 25, city: 'New York', country: 'USA' };
const { name, age, ...others } = person;
console.log(name, age); // John, 25
console.log(others); // { city: 'New York', country: 'USA' }
```

## 函数参数解构

### 对象参数解构

```javascript
// 传统方式
function createUser(options) {
  const name = options.name || 'Anonymous';
  const age = options.age || 0;
  return { name, age };
}

// 解构赋值
function createUser({ name = 'Anonymous', age = 0 }) {
  return { name, age };
}

// 箭头函数
const createUser = ({ name = 'Anonymous', age = 0 }) => ({ name, age });

const user = createUser({ name: 'John', age: 25 });
```

### 数组参数解构

```javascript
function processCoords([x, y]) {
  return { x, y };
}

const point = processCoords([10, 20]);
console.log(point); // { x: 10, y: 20 }
```

### 复杂参数解构

```javascript
function displayUser({ name, address: { city } }) {
  console.log(`${name} lives in ${city}`);
}

const user = {
  name: 'John',
  address: {
    city: 'New York',
    country: 'USA'
  }
};

displayUser(user); // John lives in New York
```

## 实际应用示例

### 1. 从函数返回多个值

```javascript
function getUserInfo() {
  return {
    name: 'John',
    age: 25,
    email: 'john@example.com'
  };
}

const { name, age, email } = getUserInfo();
```

### 2. 处理 API 响应

```javascript
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  const { data: { user, permissions } } = await response.json();
  return { user, permissions };
}
```

### 3. 配置对象处理

```javascript
function createServer({
  port = 3000,
  host = 'localhost',
  ssl = false,
  timeout = 5000
} = {}) {
  console.log(`Server running on ${host}:${port}`);
  // ...
}

createServer({ port: 8080, ssl: true });
```

### 4. React 组件 props

```javascript
// React 组件
function UserCard({ name, age, avatar, onClick }) {
  return (
    <div onClick={onClick}>
      <img src={avatar} alt={name} />
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  );
}
```

## 注意事项

### 1. undefined 的处理

```javascript
const { a } = { a: undefined };
console.log(a); // undefined

const { b = 1 } = { b: undefined };
console.log(b); // 1

const { c = 1 } = {};
console.log(c); // 1
```

### 2. 圆括号问题

```javascript
// 错误：解构赋值不能在语句开头使用圆括号
// { a } = { a: 1 }; // SyntaxError

// 正确：使用圆括号包裹
({ a } = { a: 1 });

// 或者使用 let/const
let { a } = { a: 1 };
```

### 3. 已声明变量的解构

```javascript
let x;
// { x } = { x: 1 }; // SyntaxError
({ x } = { x: 1 }); // 正确

let y;
[y] = [1]; // 正确（数组解构不需要圆括号）
```

## 最佳实践

1. **保持简洁**：避免过深的嵌套解构
2. **使用默认值**：提高代码健壮性
3. **合理重命名**：避免变量名冲突
4. **注意性能**：大量解构可能影响性能

```javascript
// 好的做法
const { name, age = 0 } = user;

// 避免过深嵌套
const { 
  user: { 
    profile: { 
      settings: { 
        theme 
      } 
    } 
  } 
} = data; // 过于复杂

// 更好的方式
const theme = data.user.profile.settings.theme;
```

## 参考

- [MDN 解构赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
