# 模板字符串（Template Literals）

## 概述

模板字符串是 ES6 引入的字符串字面量语法，使用反引号（`）包裹，支持多行字符串和字符串插值。

## 基本语法

### 基础用法

```javascript
const name = 'John';
const age = 25;

// 传统字符串拼接
const message1 = 'Hello, ' + name + '! You are ' + age + ' years old.';

// 模板字符串
const message2 = `Hello, ${name}! You are ${age} years old.`;
```

### 表达式插值

```javascript
const a = 5;
const b = 10;

console.log(`Fifteen is ${a + b}.`); // Fifteen is 15.
console.log(`Result: ${a * 2 + b}`); // Result: 20

// 函数调用
const greet = name => `Hello, ${name.toUpperCase()}!`;
console.log(greet('John')); // Hello, JOHN!

// 三元运算符
const status = 'active';
console.log(`Status: ${status === 'active' ? 'Active' : 'Inactive'}`);
```

## 多行字符串

### 传统方式的问题

```javascript
// 传统方式（需要拼接或转义）
const text1 = 'Line 1\n' +
              'Line 2\n' +
              'Line 3';

const text2 = 'Line 1\n\
Line 2\n\
Line 3';
```

### 模板字符串的方式

```javascript
// 模板字符串（保留换行）
const text = `Line 1
Line 2
Line 3`;

console.log(text);
// Line 1
// Line 2
// Line 3
```

### 生成 HTML

```javascript
const user = { name: 'John', age: 25 };

const html = `
  <div class="user-card">
    <h2>${user.name}</h2>
    <p>Age: ${user.age}</p>
  </div>
`;

document.body.innerHTML = html;
```

## 标签模板（Tagged Templates）

### 基本用法

标签模板允许你用函数处理模板字符串：

```javascript
function tag(strings, ...values) {
  console.log(strings); // ['Hello, ', '! You are ', ' years old.']
  console.log(values);  // ['John', 25]
  return 'Processed string';
}

const result = tag`Hello, ${name}! You are ${age} years old.`;
```

### 实际应用示例

#### 1. 高亮显示

```javascript
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i] ? `<mark>${values[i]}</mark>` : '';
    return result + str + value;
  }, '');
}

const name = 'John';
const age = 25;
const html = highlight`Hello, ${name}! You are ${age} years old.`;
// "Hello, <mark>John</mark>! You are <mark>25</mark> years old."
```

#### 2. 国际化

```javascript
function i18n(strings, ...values) {
  const translations = {
    'Hello': 'Hola',
    'years old': 'años'
  };
  
  return strings.reduce((result, str, i) => {
    const translated = translations[str] || str;
    const value = values[i] || '';
    return result + translated + value;
  }, '');
}

const name = 'John';
const message = i18n`Hello, ${name}! You are 25 years old.`;
// "Hola, John! You are 25 años."
```

#### 3. SQL 查询构建

```javascript
function sql(strings, ...values) {
  const escaped = values.map(v => 
    typeof v === 'string' ? `'${v}'` : v
  );
  
  return strings.reduce((result, str, i) => {
    return result + str + (escaped[i] || '');
  }, '');
}

const table = 'users';
const name = "John's";
const query = sql`SELECT * FROM ${table} WHERE name = ${name}`;
// "SELECT * FROM users WHERE name = 'John''s'"
```

## 嵌套模板

```javascript
const users = [
  { name: 'John', age: 25 },
  { name: 'Jane', age: 30 }
];

const html = `
  <ul>
    ${users.map(user => `
      <li>
        <strong>${user.name}</strong> - ${user.age} years old
      </li>
    `).join('')}
  </ul>
`;
```

## 原始字符串

通过 `String.raw` 可以获取模板字符串的原始内容（不处理转义字符）：

```javascript
const path = `C:\new\test`;
console.log(path); // C:
// ew	est（\n 和 \t 被转义了）

const rawPath = String.raw`C:\new\test`;
console.log(rawPath); // C:\new\test

// 在标签模板中访问原始字符串
function raw(strings, ...values) {
  console.log(strings.raw[0]); // "First line\nSecond line"
  return strings.raw.join('');
}

const result = raw`First line\nSecond line`;
```

## 最佳实践

1. **优先使用模板字符串**：替代传统的字符串拼接
2. **保持简洁**：避免在插值中写过于复杂的表达式
3. **注意性能**：大量字符串拼接时，考虑使用数组 join
4. **安全处理**：对用户输入进行转义，防止 XSS 攻击

```javascript
// 好的做法
const message = `Welcome, ${userName}!`;

// 避免
const message = 'Welcome, ' + userName + '!';

// 复杂表达式考虑提取为变量
const statusMessage = isActive ? 'Active' : 'Inactive';
const display = `Status: ${statusMessage}`;
```

## 参考

- [MDN 模板字符串](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Template_literals)
