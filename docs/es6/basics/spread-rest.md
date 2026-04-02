# 展开运算符和剩余参数（Spread and Rest Operators）

## 概述

ES6 引入了展开运算符（`...`），用于展开数组或对象；以及剩余参数语法，用于收集函数参数。

## 展开运算符（Spread Operator）

### 数组展开

#### 基本用法

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// 合并数组
const merged = [...arr1, ...arr2];
console.log(merged); // [1, 2, 3, 4, 5, 6]

// 在指定位置插入元素
const withInserted = [0, ...arr1, 4, ...arr2, 7];
console.log(withInserted); // [0, 1, 2, 3, 4, 4, 5, 6, 7]
```

#### 复制数组

```javascript
const original = [1, 2, 3];
const copy = [...original];

// 浅拷贝
copy.push(4);
console.log(original); // [1, 2, 3]（不受影响）
console.log(copy); // [1, 2, 3, 4]
```

#### 与 apply 的对比

```javascript
// 传统方式
function sum(a, b, c) {
  return a + b + c;
}
const numbers = [1, 2, 3];
const result1 = sum.apply(null, numbers);

// 使用展开运算符
const result2 = sum(...numbers);
console.log(result1, result2); // 6, 6
```

#### Math 函数应用

```javascript
const numbers = [5, 2, 8, 1, 9];

// 传统方式
const max1 = Math.max.apply(null, numbers);
const min1 = Math.min.apply(null, numbers);

// 使用展开运算符
const max2 = Math.max(...numbers);
const min2 = Math.min(...numbers);

console.log(max2, min2); // 9, 1
```

### 对象展开

#### 基本用法

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

// 合并对象
const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }
```

#### 复制对象

```javascript
const original = { a: 1, b: 2 };
const copy = { ...original };

// 浅拷贝
copy.c = 3;
console.log(original); // { a: 1, b: 2 }（不受影响）
console.log(copy); // { a: 1, b: 2, c: 3 }
```

#### 覆盖属性

```javascript
const defaults = { 
  color: 'blue', 
  size: 'medium',
  price: 10 
};

const custom = { color: 'red', price: 20 };

// 后面的属性覆盖前面的
const final = { ...defaults, ...custom };
console.log(final); 
// { color: 'red', size: 'medium', price: 20 }
```

#### 添加新属性

```javascript
const user = { name: 'John', age: 25 };
const userWithAddress = { 
  ...user, 
  address: 'New York' 
};

console.log(userWithAddress); 
// { name: 'John', age: 25, address: 'New York' }
```

### 字符串展开

```javascript
const str = 'hello';
const chars = [...str];
console.log(chars); // ['h', 'e', 'l', 'l', 'o']

// 处理 Unicode 字符
const emoji = '👨‍👩‍👧‍👦';
console.log(emoji.length); // 11
console.log([...emoji].length); // 1（正确处理 Unicode）
```

## 剩余参数（Rest Parameters）

### 基本用法

```javascript
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3)); // 6
console.log(sum(1, 2, 3, 4, 5)); // 15
```

### 与其他参数结合

```javascript
function process(first, second, ...others) {
  console.log('First:', first);
  console.log('Second:', second);
  console.log('Others:', others);
}

process(1, 2, 3, 4, 5);
// First: 1
// Second: 2
// Others: [3, 4, 5]
```

### 箭头函数中的剩余参数

```javascript
const multiply = (...args) => args.reduce((a, b) => a * b, 1);

console.log(multiply(2, 3, 4)); // 24

const log = (label, ...data) => {
  console.log(`[${label}]`, ...data);
};

log('INFO', 'User logged in', { id: 123 });
// [INFO] User logged in { id: 123 }
```

## 实际应用示例

### 1. 函数柯里化

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return (...moreArgs) => curried(...args, ...moreArgs);
  };
}

const add = curry((a, b, c) => a + b + c);
console.log(add(1)(2)(3)); // 6
console.log(add(1, 2)(3)); // 6
console.log(add(1)(2, 3)); // 6
```

### 2. 不定参数处理

```javascript
function createMessage(template, ...args) {
  return args.reduce((msg, arg, i) => {
    return msg.replace(`{${i}}`, arg);
  }, template);
}

const message = createMessage(
  'Hello {0}, you have {1} new messages',
  'John',
  5
);
console.log(message); // "Hello John, you have 5 new messages"
```

### 3. 组合函数

```javascript
function compose(...fns) {
  return (x) => fns.reduceRight((acc, fn) => fn(acc), x);
}

const double = x => x * 2;
const addOne = x => x + 1;
const square = x => x * x;

const transform = compose(square, addOne, double);
console.log(transform(3)); // ((3 * 2) + 1)² = 49
```

### 4. React 组件 props 传递

```javascript
function Button({ children, ...props }) {
  return (
    <button {...props}>
      {children}
    </button>
  );
}

// 使用
<Button 
  className="primary" 
  onClick={handleClick}
  disabled={isLoading}
>
  Click me
</Button>
```

### 5. 配置合并

```javascript
const defaultConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
};

function createConfig(userConfig = {}) {
  return {
    ...defaultConfig,
    ...userConfig,
    headers: {
      ...defaultConfig.headers,
      ...userConfig.headers
    }
  };
}

const config = createConfig({
  timeout: 10000,
  headers: { 'Authorization': 'Bearer token' }
});
```

### 6. 数组操作

```javascript
// 去重
const arr = [1, 2, 2, 3, 3, 4];
const unique = [...new Set(arr)];
console.log(unique); // [1, 2, 3, 4]

// 浅拷贝并修改
const original = [{ id: 1 }, { id: 2 }];
const modified = [...original, { id: 3 }];

// 合并多个数组
const arrays = [[1, 2], [3, 4], [5, 6]];
const flattened = [].concat(...arrays);
// 或者
const flattened2 = arrays.reduce((acc, arr) => [...acc, ...arr], []);
```

## 注意事项

### 1. 浅拷贝问题

```javascript
// 嵌套对象仍然是引用
const original = { 
  user: { name: 'John' },
  items: [1, 2, 3]
};

const copy = { ...original };
copy.user.name = 'Jane'; // 会影响 original
copy.items.push(4); // 会影响 original

console.log(original.user.name); // 'Jane'
console.log(original.items); // [1, 2, 3, 4]
```

### 2. 性能考虑

```javascript
// 大数组展开可能影响性能
const largeArray = Array(100000).fill(0);

// 不推荐：性能差
const copy1 = [...largeArray];

// 推荐：性能好
const copy2 = largeArray.slice();
```

### 3. 属性顺序

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };

// 后面的属性覆盖前面的
const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 3, c: 4 }
```

## 最佳实践

1. **用于数组和对象的浅拷贝**
2. **处理函数不定参数**
3. **合并配置对象**
4. **React props 传递**
5. **避免深层嵌套对象的展开**

```javascript
// 好的做法
const newUser = { ...user, isActive: true };

// 避免深层嵌套
const bad = { 
  ...data,
  user: { ...data.user, profile: { ...data.user.profile, settings: { ... } } }
};

// 更好的方式
const good = {
  ...data,
  user: updateUserProfile(data.user)
};
```

## 参考

- [MDN 展开语法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
- [MDN 剩余参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/rest_parameters)
