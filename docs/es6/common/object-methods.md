# 对象新方法（Object Methods）

## 概述

ES6 为 Object 对象引入了许多新方法，用于对象操作、属性遍历和对象转换。

## 属性获取方法

### Object.keys()

返回对象自身可枚举属性的键名数组：

```javascript
const person = {
  name: 'John',
  age: 25,
  city: 'New York'
};

const keys = Object.keys(person);
console.log(keys); // ['name', 'age', 'city']

// 遍历对象属性
Object.keys(person).forEach(key => {
  console.log(`${key}: ${person[key]}`);
});
```

### Object.values()

返回对象自身可枚举属性的键值数组：

```javascript
const person = {
  name: 'John',
  age: 25,
  city: 'New York'
};

const values = Object.values(person);
console.log(values); // ['John', 25, 'New York']

// 计算所有值的总和
const scores = { math: 90, english: 85, science: 95 };
const total = Object.values(scores).reduce((sum, score) => sum + score, 0);
console.log(total); // 270
```

### Object.entries()

返回对象自身可枚举属性的键值对数组：

```javascript
const person = {
  name: 'John',
  age: 25,
  city: 'New York'
};

const entries = Object.entries(person);
console.log(entries);
// [['name', 'John'], ['age', 25], ['city', 'New York']]

// 遍历对象
for (const [key, value] of Object.entries(person)) {
  console.log(`${key}: ${value}`);
}

// 转换为 Map
const map = new Map(Object.entries(person));
console.log(map); // Map { 'name' => 'John', 'age' => 25, 'city' => 'New York' }
```

## 对象创建和复制

### Object.assign()

将源对象的可枚举属性复制到目标对象：

```javascript
const target = { a: 1, b: 2 };
const source = { b: 3, c: 4 };

const result = Object.assign(target, source);
console.log(result); // { a: 1, b: 3, c: 4 }
console.log(target === result); // true

// 浅拷贝
const copy = Object.assign({}, source);

// 合并多个对象
const merged = Object.assign({}, { a: 1 }, { b: 2 }, { c: 3 });
console.log(merged); // { a: 1, b: 2, c: 3 }

// 处理 Symbol 属性
const sym = Symbol('test');
const obj = { [sym]: 'value' };
const copy2 = Object.assign({}, obj);
console.log(copy2[sym]); // 'value'
```

### Object.fromEntries()

将键值对列表转换为对象：

```javascript
// 从 entries 创建对象
const entries = [['name', 'John'], ['age', 25], ['city', 'New York']];
const obj = Object.fromEntries(entries);
console.log(obj); // { name: 'John', age: 25, city: 'New York' }

// 从 Map 创建对象
const map = new Map([['a', 1], ['b', 2]]);
const obj2 = Object.fromEntries(map);
console.log(obj2); // { a: 1, b: 2 }

// 与 Object.entries() 配合使用
const person = { name: 'John', age: 25 };
const modified = Object.fromEntries(
  Object.entries(person).map(([key, value]) => [key, String(value)])
);
console.log(modified); // { name: 'John', age: '25' }
```

## 对象比较和检查

### Object.is()

比较两个值是否严格相等：

```javascript
// 与 === 的区别
console.log(NaN === NaN); // false
console.log(Object.is(NaN, NaN)); // true

console.log(+0 === -0); // true
console.log(Object.is(+0, -0)); // false

// 其他情况与 === 相同
console.log(Object.is(1, 1)); // true
console.log(Object.is('a', 'a')); // true
console.log(Object.is({}, {})); // false
```

### Object.hasOwnProperty()

检查对象是否拥有指定的自身属性：

```javascript
const person = { name: 'John', age: 25 };
const student = Object.create(person);
student.grade = 'A';

console.log(person.hasOwnProperty('name')); // true
console.log(student.hasOwnProperty('name')); // false（继承的属性）
console.log(student.hasOwnProperty('grade')); // true

// 更安全的检查方式
console.log(Object.prototype.hasOwnProperty.call(student, 'name')); // false
```

## 对象属性描述符

### Object.defineProperty()

定义或修改对象属性：

```javascript
const person = {};

Object.defineProperty(person, 'name', {
  value: 'John',
  writable: false, // 不可写
  enumerable: true, // 可枚举
  configurable: false // 不可配置
});

console.log(person.name); // 'John'
person.name = 'Jane'; // 严格模式下报错
console.log(person.name); // 'John'（没有改变）

// 不可枚举属性
Object.defineProperty(person, 'hidden', {
  value: 'secret',
  enumerable: false
});

console.log(Object.keys(person)); // ['name']（不包含 hidden）
```

### Object.defineProperties()

定义或修改多个对象属性：

```javascript
const person = {};

Object.defineProperties(person, {
  firstName: {
    value: 'John',
    writable: true,
    enumerable: true
  },
  lastName: {
    value: 'Doe',
    writable: true,
    enumerable: true
  },
  fullName: {
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
    enumerable: true
  }
});

console.log(person.fullName); // 'John Doe'
```

### Object.getOwnPropertyDescriptor()

获取对象属性的描述符：

```javascript
const person = { name: 'John' };
const descriptor = Object.getOwnPropertyDescriptor(person, 'name');

console.log(descriptor);
// {
//   value: 'John',
//   writable: true,
//   enumerable: true,
//   configurable: true
// }
```

## 对象冻结和密封

### Object.freeze()

冻结对象，使其完全不可变：

```javascript
const person = { name: 'John', age: 25 };
Object.freeze(person);

person.name = 'Jane'; // 无效
person.email = 'john@example.com'; // 无效
delete person.age; // 无效

console.log(person); // { name: 'John', age: 25 }

// 检查是否冻结
console.log(Object.isFrozen(person)); // true

// 浅冻结（嵌套对象仍可修改）
const user = { 
  name: 'John', 
  address: { city: 'New York' } 
};
Object.freeze(user);
user.address.city = 'Los Angeles'; // 有效！
console.log(user.address.city); // 'Los Angeles'
```

### Object.seal()

密封对象，防止添加或删除属性，但可以修改现有属性：

```javascript
const person = { name: 'John', age: 25 };
Object.seal(person);

person.name = 'Jane'; // 有效
person.email = 'john@example.com'; // 无效
delete person.age; // 无效

console.log(person); // { name: 'Jane', age: 25 }

// 检查是否密封
console.log(Object.isSealed(person)); // true
```

### Object.preventExtensions()

阻止对象扩展，防止添加新属性：

```javascript
const person = { name: 'John', age: 25 };
Object.preventExtensions(person);

person.name = 'Jane'; // 有效
person.email = 'john@example.com'; // 无效
delete person.age; // 有效

console.log(person); // { name: 'Jane' }

// 检查是否可扩展
console.log(Object.isExtensible(person)); // false
```

## 对象原型操作

### Object.getPrototypeOf()

获取对象的原型：

```javascript
const person = { name: 'John' };
const proto = Object.getPrototypeOf(person);
console.log(proto === Object.prototype); // true

// 自定义原型
const animal = { speak() { console.log('...'); } };
const dog = Object.create(animal);
dog.bark = () => console.log('Woof!');

console.log(Object.getPrototypeOf(dog) === animal); // true
```

### Object.setPrototypeOf()

设置对象的原型：

```javascript
const animal = { speak() { console.log('...'); } };
const dog = { bark() { console.log('Woof!'); } };

Object.setPrototypeOf(dog, animal);
dog.speak(); // '...'
dog.bark(); // 'Woof!'

console.log(Object.getPrototypeOf(dog) === animal); // true
```

## 实际应用示例

### 1. 配置对象合并

```javascript
const defaultConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
  headers: {
    'Content-Type': 'application/json'
  }
};

function mergeConfig(userConfig) {
  return Object.assign({}, defaultConfig, userConfig, {
    headers: Object.assign({}, defaultConfig.headers, userConfig.headers)
  });
}

const config = mergeConfig({
  timeout: 10000,
  headers: { 'Authorization': 'Bearer token' }
});

console.log(config);
// {
//   apiUrl: 'https://api.example.com',
//   timeout: 10000,
//   retries: 3,
//   headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer token' }
// }
```

### 2. 对象转换

```javascript
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 3, name: 'Bob' }
];

// 数组转对象（以 id 为键）
const usersById = Object.fromEntries(
  users.map(user => [user.id, user])
);

console.log(usersById);
// {
//   1: { id: 1, name: 'John' },
//   2: { id: 2, name: 'Jane' },
//   3: { id: 3, name: 'Bob' }
// }

// 对象转数组
const usersArray = Object.values(usersById);
console.log(usersArray); // [原始数组]
```

### 3. 数据验证

```javascript
function validateObject(obj, requiredKeys) {
  const missingKeys = requiredKeys.filter(key => !obj.hasOwnProperty(key));
  
  if (missingKeys.length > 0) {
    throw new Error(`Missing required keys: ${missingKeys.join(', ')}`);
  }
  
  return true;
}

const user = { name: 'John', email: 'john@example.com' };
const requiredKeys = ['name', 'email', 'age'];

try {
  validateObject(user, requiredKeys);
} catch (error) {
  console.log(error.message); // "Missing required keys: age"
}
```

### 4. 对象不可变性

```javascript
function createImmutableObject(obj) {
  return Object.freeze({
    ...obj,
    nested: typeof obj.nested === 'object' ? 
      createImmutableObject(obj.nested) : obj.nested
  });
}

const config = createImmutableObject({
  apiUrl: 'https://api.example.com',
  nested: {
    timeout: 5000,
    retries: 3
  }
});

// 无法修改
config.apiUrl = 'https://other.com'; // 无效
config.nested.timeout = 10000; // 无效
```

## 最佳实践

1. **使用 `Object.entries()` 遍历对象**：比 `for...in` 更安全
2. **使用 `Object.assign()` 合并对象**：替代手动属性复制
3. **使用 `Object.freeze()` 保护配置对象**：防止意外修改
4. **使用 `Object.fromEntries()` 转换数据格式**
5. **使用 `Object.is()` 进行严格比较**

```javascript
// 好的做法
const config = Object.freeze({
  apiUrl: 'https://api.example.com',
  timeout: 5000
});

Object.entries(config).forEach(([key, value]) => {
  console.log(`${key}: ${value}`);
});

// 避免
for (const key in config) {
  if (config.hasOwnProperty(key)) {
    console.log(`${key}: ${config[key]}`);
  }
}
```

## 参考

- [MDN Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)
