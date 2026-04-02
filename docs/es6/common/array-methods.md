# 数组新方法（Array Methods）

## 概述

ES6 为数组引入了许多新方法，使数组操作更加方便和高效。

## 查找方法

### Array.prototype.find()

返回数组中满足条件的第一个元素：

```javascript
const numbers = [1, 2, 3, 4, 5];
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 3, name: 'Bob' }
];

// 查找第一个大于 3 的数
const firstGreaterThan3 = numbers.find(n => n > 3);
console.log(firstGreaterThan3); // 4

// 查找用户
const user = users.find(u => u.id === 2);
console.log(user); // { id: 2, name: 'Jane' }

// 没找到返回 undefined
const notFound = numbers.find(n => n > 10);
console.log(notFound); // undefined
```

### Array.prototype.findIndex()

返回数组中满足条件的第一个元素的索引：

```javascript
const numbers = [1, 2, 3, 4, 5];
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 3, name: 'Bob' }
];

// 查找索引
const index = numbers.findIndex(n => n > 3);
console.log(index); // 3

// 查找用户索引
const userIndex = users.findIndex(u => u.id === 2);
console.log(userIndex); // 1

// 没找到返回 -1
const notFoundIndex = numbers.findIndex(n => n > 10);
console.log(notFoundIndex); // -1
```

### Array.prototype.includes()

判断数组是否包含指定元素：

```javascript
const numbers = [1, 2, 3, 4, 5];
const fruits = ['apple', 'banana', 'orange'];

console.log(numbers.includes(3)); // true
console.log(numbers.includes(6)); // false
console.log(fruits.includes('apple')); // true
console.log(fruits.includes('grape')); // false

// 支持 NaN
console.log([1, 2, NaN].includes(NaN)); // true

// 可以指定起始位置
console.log(numbers.includes(2, 2)); // false（从索引 2 开始查找）
```

## 遍历方法

### Array.prototype.forEach()

对数组的每个元素执行一次函数：

```javascript
const numbers = [1, 2, 3, 4, 5];

numbers.forEach((num, index, array) => {
  console.log(`Index ${index}: ${num}`);
});

// 修改原数组
numbers.forEach((num, index, array) => {
  array[index] = num * 2;
});
console.log(numbers); // [2, 4, 6, 8, 10]
```

### Array.prototype.map()

创建一个新数组，包含调用函数处理后的每个元素：

```javascript
const numbers = [1, 2, 3, 4, 5];

// 基本用法
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// 处理对象数组
const users = [
  { firstName: 'John', lastName: 'Doe' },
  { firstName: 'Jane', lastName: 'Smith' }
];

const fullNames = users.map(user => ({
  ...user,
  fullName: `${user.firstName} ${user.lastName}`
}));

console.log(fullNames);
// [
//   { firstName: 'John', lastName: 'Doe', fullName: 'John Doe' },
//   { firstName: 'Jane', lastName: 'Smith', fullName: 'Jane Smith' }
// ]
```

### Array.prototype.filter()

创建一个新数组，包含通过测试的所有元素：

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 过滤偶数
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// 过滤大于 5 的数
const greaterThan5 = numbers.filter(n => n > 5);
console.log(greaterThan5); // [6, 7, 8, 9, 10]

// 过滤对象数组
const users = [
  { name: 'John', age: 25, active: true },
  { name: 'Jane', age: 30, active: false },
  { name: 'Bob', age: 35, active: true }
];

const activeUsers = users.filter(user => user.active);
console.log(activeUsers);
// [{ name: 'John', age: 25, active: true }, { name: 'Bob', age: 35, active: true }]
```

## 归约方法

### Array.prototype.reduce()

将数组归约为单个值：

```javascript
const numbers = [1, 2, 3, 4, 5];

// 求和
const sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum); // 15

// 求最大值
const max = numbers.reduce((acc, num) => Math.max(acc, num), -Infinity);
console.log(max); // 5

// 统计字符出现次数
const words = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const wordCount = words.reduce((acc, word) => {
  acc[word] = (acc[word] || 0) + 1;
  return acc;
}, {});
console.log(wordCount); // { apple: 3, banana: 2, orange: 1 }

// 数组扁平化
const nested = [[1, 2], [3, 4], [5, 6]];
const flat = nested.reduce((acc, arr) => acc.concat(arr), []);
console.log(flat); // [1, 2, 3, 4, 5, 6]
```

### Array.prototype.reduceRight()

从右到左归约数组：

```javascript
const numbers = [1, 2, 3, 4, 5];

// 从右到左求和
const sum = numbers.reduceRight((acc, num) => acc + num, 0);
console.log(sum); // 15

// 字符串拼接
const words = ['Hello', 'World', '!'];
const sentence = words.reduceRight((acc, word) => `${acc} ${word}`);
console.log(sentence); // "! World Hello"
```

## 排序方法

### Array.prototype.sort()

对数组元素进行排序（原地排序）：

```javascript
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];

// 数字排序（默认按字符串排序会有问题）
numbers.sort((a, b) => a - b); // 升序
console.log(numbers); // [1, 1, 2, 3, 4, 5, 6, 9]

numbers.sort((a, b) => b - a); // 降序
console.log(numbers); // [9, 6, 5, 4, 3, 2, 1, 1]

// 对象排序
const users = [
  { name: 'John', age: 25 },
  { name: 'Jane', age: 30 },
  { name: 'Bob', age: 20 }
];

// 按年龄排序
users.sort((a, b) => a.age - b.age);
console.log(users);
// [{ name: 'Bob', age: 20 }, { name: 'John', age: 25 }, { name: 'Jane', age: 30 }]

// 按名字排序
users.sort((a, b) => a.name.localeCompare(b.name));
console.log(users);
// [{ name: 'Bob', age: 20 }, { name: 'Jane', age: 30 }, { name: 'John', age: 25 }]
```

## 其他方法

### Array.from()

从类数组或可迭代对象创建新数组：

```javascript
// 从字符串创建
const chars = Array.from('Hello');
console.log(chars); // ['H', 'e', 'l', 'l', 'o']

// 从 Set 创建
const set = new Set([1, 2, 3, 4, 5]);
const array = Array.from(set);
console.log(array); // [1, 2, 3, 4, 5]

// 从 Map 创建
const map = new Map([['a', 1], ['b', 2]]);
const entries = Array.from(map);
console.log(entries); // [['a', 1], ['b', 2]]

// 使用映射函数
const doubled = Array.from([1, 2, 3], x => x * 2);
console.log(doubled); // [2, 4, 6]

// 生成序列
const range = Array.from({ length: 5 }, (_, i) => i);
console.log(range); // [0, 1, 2, 3, 4]

// 生成指定范围的序列
const range2 = Array.from({ length: 5 }, (_, i) => i + 1);
console.log(range2); // [1, 2, 3, 4, 5]
```

### Array.of()

创建一个具有可变数量参数的新数组：

```javascript
// 与 Array() 的区别
console.log(Array(3)); // [empty × 3]
console.log(Array.of(3)); // [3]

console.log(Array(1, 2, 3)); // [1, 2, 3]
console.log(Array.of(1, 2, 3)); // [1, 2, 3]

// 创建包含单个数字的数组
const singleNumber = Array.of(5);
console.log(singleNumber); // [5]
```

### Array.prototype.fill()

用固定值填充数组：

```javascript
const array = [1, 2, 3, 4, 5];

// 填充所有元素
array.fill(0);
console.log(array); // [0, 0, 0, 0, 0]

// 指定起始和结束位置
const array2 = [1, 2, 3, 4, 5];
array2.fill(0, 2, 4);
console.log(array2); // [1, 2, 0, 0, 5]

// 创建并填充数组
const filledArray = new Array(5).fill(0);
console.log(filledArray); // [0, 0, 0, 0, 0]
```

### Array.prototype.copyWithin()

在数组内部复制元素：

```javascript
const array = [1, 2, 3, 4, 5];

// 将索引 3 到末尾的元素复制到索引 0 开始的位置
array.copyWithin(0, 3);
console.log(array); // [4, 5, 3, 4, 5]

// 指定复制范围
const array2 = [1, 2, 3, 4, 5];
array2.copyWithin(1, 3, 5);
console.log(array2); // [1, 4, 5, 4, 5]
```

### Array.prototype.keys()、values()、entries()

返回数组迭代器：

```javascript
const array = ['a', 'b', 'c'];

// keys() - 索引迭代器
for (const key of array.keys()) {
  console.log(key); // 0, 1, 2
}

// values() - 值迭代器
for (const value of array.values()) {
  console.log(value); // 'a', 'b', 'c'
}

// entries() - 键值对迭代器
for (const [index, value] of array.entries()) {
  console.log(`${index}: ${value}`);
  // 0: a
  // 1: b
  // 2: c
}
```

## 实际应用示例

### 1. 数据处理管道

```javascript
const users = [
  { id: 1, name: 'John', age: 25, active: true },
  { id: 2, name: 'Jane', age: 30, active: false },
  { id: 3, name: 'Bob', age: 35, active: true },
  { id: 4, name: 'Alice', age: 28, active: true }
];

const result = users
  .filter(user => user.active)
  .map(user => ({
    ...user,
    displayName: `${user.name} (${user.age})`
  }))
  .sort((a, b) => a.age - b.age);

console.log(result);
// [
//   { id: 1, name: 'John', age: 25, active: true, displayName: 'John (25)' },
//   { id: 4, name: 'Alice', age: 28, active: true, displayName: 'Alice (28)' },
//   { id: 3, name: 'Bob', age: 35, active: true, displayName: 'Bob (35)' }
// ]
```

### 2. 分组统计

```javascript
const items = [
  { category: 'fruit', name: 'apple' },
  { category: 'vegetable', name: 'carrot' },
  { category: 'fruit', name: 'banana' },
  { category: 'vegetable', name: 'broccoli' },
  { category: 'fruit', name: 'orange' }
];

const grouped = items.reduce((acc, item) => {
  acc[item.category] = acc[item.category] || [];
  acc[item.category].push(item.name);
  return acc;
}, {});

console.log(grouped);
// {
//   fruit: ['apple', 'banana', 'orange'],
//   vegetable: ['carrot', 'broccoli']
// }
```

### 3. 去重和转换

```javascript
const numbers = [1, 2, 2, 3, 3, 4, 5, 5];

// 使用 Set 去重
const unique = [...new Set(numbers)];
console.log(unique); // [1, 2, 3, 4, 5]

// 使用 filter 去重
const unique2 = numbers.filter((num, index) => numbers.indexOf(num) === index);
console.log(unique2); // [1, 2, 3, 4, 5]
```

## 最佳实践

1. **优先使用新方法**：`find`、`includes` 等方法更语义化
2. **链式调用**：使用 `filter`、`map`、`reduce` 构建数据处理管道
3. **避免修改原数组**：使用 `map`、`filter` 创建新数组
4. **使用 `Array.from()`**：处理类数组对象

```javascript
// 好的做法
const activeUsers = users
  .filter(user => user.active)
  .map(user => user.name);

// 避免
const activeUsers2 = [];
for (let i = 0; i < users.length; i++) {
  if (users[i].active) {
    activeUsers2.push(users[i].name);
  }
}
```

## 参考

- [MDN Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)
