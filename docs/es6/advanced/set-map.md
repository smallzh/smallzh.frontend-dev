# Set 和 Map

## 概述

Set 和 Map 是 ES6 引入的新的数据结构，提供了更强大的数据存储和操作能力。

## Set

### 基本概念

Set 是一种新的数据结构，类似于数组，但成员的值都是唯一的，没有重复的值。

### 创建 Set

```javascript
// 通过构造函数创建
const set1 = new Set();
const set2 = new Set([1, 2, 3, 4, 5]);
const set3 = new Set('hello');
console.log(set3); // Set { 'h', 'e', 'l', 'o' }
```

### Set 属性和方法

```javascript
const set = new Set([1, 2, 3, 4, 5]);

// size 属性
console.log(set.size); // 5

// 添加元素
set.add(6);
console.log(set); // Set { 1, 2, 3, 4, 5, 6 }

// 添加重复元素（无效）
set.add(1);
console.log(set); // Set { 1, 2, 3, 4, 5, 6 }

// 删除元素
set.delete(3);
console.log(set); // Set { 1, 2, 4, 5, 6 }

// 检查元素是否存在
console.log(set.has(4)); // true
console.log(set.has(3)); // false

// 清空集合
set.clear();
console.log(set.size); // 0
```

### 遍历 Set

```javascript
const set = new Set(['red', 'green', 'blue']);

// forEach
set.forEach(value => {
  console.log(value);
});

// for...of
for (const value of set) {
  console.log(value);
}

// keys() 和 values()（等价）
console.log([...set.keys()]); // ['red', 'green', 'blue']
console.log([...set.values()]); // ['red', 'green', 'blue']

// entries()
console.log([...set.entries()]);
// [['red', 'red'], ['green', 'green'], ['blue', 'blue']]
```

### Set 操作

#### 并集

```javascript
const setA = new Set([1, 2, 3]);
const setB = new Set([3, 4, 5]);

const union = new Set([...setA, ...setB]);
console.log(union); // Set { 1, 2, 3, 4, 5 }
```

#### 交集

```javascript
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

const intersection = new Set([...setA].filter(x => setB.has(x)));
console.log(intersection); // Set { 2, 3 }
```

#### 差集

```javascript
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

const difference = new Set([...setA].filter(x => !setB.has(x)));
console.log(difference); // Set { 1 }
```

### Set 应用示例

#### 数组去重

```javascript
const arr = [1, 2, 2, 3, 3, 4, 5, 5];
const unique = [...new Set(arr)];
console.log(unique); // [1, 2, 3, 4, 5]
```

#### 字符串去重

```javascript
const str = 'abracadabra';
const unique = [...new Set(str)].join('');
console.log(unique); // 'abrcd'
```

#### 判断是否有重复元素

```javascript
function hasDuplicates(arr) {
  return new Set(arr).size !== arr.length;
}

console.log(hasDuplicates([1, 2, 3, 4])); // false
console.log(hasDuplicates([1, 2, 2, 3])); // true
```

## WeakSet

### 基本概念

WeakSet 结构与 Set 类似，也是不重复的值的集合。但 WeakSet 的成员只能是对象，而不能是其他类型的值。

### 创建 WeakSet

```javascript
const ws = new WeakSet();

const obj1 = { name: 'obj1' };
const obj2 = { name: 'obj2' };

ws.add(obj1);
ws.add(obj2);

console.log(ws.has(obj1)); // true
console.log(ws.has(obj2)); // true
```

### WeakSet 特性

```javascript
// 不能遍历
const ws = new WeakSet();
ws.add({ name: 'test' });

// 没有 size 属性
console.log(ws.size); // undefined

// 不能使用 forEach
// ws.forEach(() => {}); // TypeError

// 不能使用 for...of
// for (const item of ws) {} // TypeError

// 对象被垃圾回收后，WeakSet 中的引用也会被清除
let obj = { name: 'test' };
ws.add(obj);
obj = null; // 对象被垃圾回收，WeakSet 中的引用也会被清除
```

### WeakSet 应用示例

#### 存储 DOM 节点

```javascript
const domNodes = new WeakSet();

function markAsProcessed(node) {
  domNodes.add(node);
}

function isProcessed(node) {
  return domNodes.has(node);
}

const div = document.createElement('div');
markAsProcessed(div);
console.log(isProcessed(div)); // true
```

#### 检查循环引用

```javascript
function hasCircularReference(obj) {
  const seen = new WeakSet();
  
  function detect(obj) {
    if (obj && typeof obj === 'object') {
      if (seen.has(obj)) {
        return true;
      }
      seen.add(obj);
      
      for (const key in obj) {
        if (detect(obj[key])) {
          return true;
        }
      }
    }
    return false;
  }
  
  return detect(obj);
}

const obj = { name: 'test' };
obj.self = obj;
console.log(hasCircularReference(obj)); // true
```

## Map

### 基本概念

Map 是一种新的数据结构，类似于对象，但键可以是任意类型的值（包括对象）。

### 创建 Map

```javascript
// 通过构造函数创建
const map1 = new Map();
const map2 = new Map([
  ['name', 'John'],
  ['age', 25],
  [true, 'boolean']
]);

// 通过数组创建
const map3 = new Map([
  [{ name: 'John' }, 'person1'],
  [{ name: 'Jane' }, 'person2']
]);
```

### Map 属性和方法

```javascript
const map = new Map();

// 设置键值对
map.set('name', 'John');
map.set('age', 25);
map.set(true, 'boolean');

// 获取值
console.log(map.get('name')); // 'John'
console.log(map.get('age')); // 25
console.log(map.get(true)); // 'boolean'

// 检查键是否存在
console.log(map.has('name')); // true
console.log(map.has('email')); // false

// 删除键值对
map.delete('age');
console.log(map.has('age')); // false

// size 属性
console.log(map.size); // 2

// 清空 Map
map.clear();
console.log(map.size); // 0
```

### 遍历 Map

```javascript
const map = new Map([
  ['name', 'John'],
  ['age', 25],
  ['city', 'New York']
]);

// forEach
map.forEach((value, key) => {
  console.log(`${key}: ${value}`);
});

// for...of
for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}

// keys()
console.log([...map.keys()]); // ['name', 'age', 'city']

// values()
console.log([...map.values()]); // ['John', 25, 'New York']

// entries()
console.log([...map.entries()]);
// [['name', 'John'], ['age', 25], ['city', 'New York']]
```

### Map 与 Object 的区别

```javascript
// 键可以是任意类型
const map = new Map();
map.set(1, 'number');
map.set('1', 'string');
map.set(true, 'boolean');
map.set({ name: 'obj' }, 'object');

console.log(map.get(1)); // 'number'
console.log(map.get('1')); // 'string'

// 保持插入顺序
const map2 = new Map();
map2.set('c', 3);
map2.set('a', 1);
map2.set('b', 2);

console.log([...map2.keys()]); // ['c', 'a', 'b']

// 可以知道大小
console.log(map2.size); // 3
```

### Map 应用示例

#### 缓存实现

```javascript
class Cache {
  constructor(ttl = 60000) {
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  set(key, value) {
    this.cache.set(key, {
      value,
      timestamp: Date.now()
    });
  }
  
  get(key) {
    const item = this.cache.get(key);
    
    if (!item) {
      return undefined;
    }
    
    if (Date.now() - item.timestamp > this.ttl) {
      this.cache.delete(key);
      return undefined;
    }
    
    return item.value;
  }
  
  has(key) {
    return this.get(key) !== undefined;
  }
  
  delete(key) {
    return this.cache.delete(key);
  }
  
  clear() {
    this.cache.clear();
  }
  
  get size() {
    return this.cache.size;
  }
}

const cache = new Cache(5000);
cache.set('user', { name: 'John' });
console.log(cache.get('user')); // { name: 'John' }
```

#### 对象属性映射

```javascript
const objMap = new Map();

const user1 = { name: 'John' };
const user2 = { name: 'Jane' };

objMap.set(user1, 'admin');
objMap.set(user2, 'user');

console.log(objMap.get(user1)); // 'admin'
console.log(objMap.get(user2)); // 'user'

// 遍历
for (const [user, role] of objMap) {
  console.log(`${user.name}: ${role}`);
}
```

#### 统计词频

```javascript
function wordFrequency(text) {
  const words = text.toLowerCase().split(/\s+/);
  const frequency = new Map();
  
  for (const word of words) {
    frequency.set(word, (frequency.get(word) || 0) + 1);
  }
  
  return frequency;
}

const text = 'apple banana apple orange banana apple';
const frequency = wordFrequency(text);

console.log(frequency.get('apple')); // 3
console.log(frequency.get('banana')); // 2
console.log(frequency.get('orange')); // 1

// 获取出现最多的单词
const mostFrequent = [...frequency.entries()]
  .reduce((max, [word, count]) => 
    count > max[1] ? [word, count] : max
  );

console.log(mostFrequent); // ['apple', 3]
```

## WeakMap

### 基本概念

WeakMap 结构与 Map 类似，但键只能是对象（null 除外），且键是弱引用。

### 创建 WeakMap

```javascript
const wm = new WeakMap();

const key1 = { id: 1 };
const key2 = { id: 2 };

wm.set(key1, 'value1');
wm.set(key2, 'value2');

console.log(wm.get(key1)); // 'value1'
console.log(wm.get(key2)); // 'value2'
```

### WeakMap 特性

```javascript
// 不能遍历
const wm = new WeakMap();
wm.set({ id: 1 }, 'value');

// 没有 size 属性
console.log(wm.size); // undefined

// 不能使用 forEach
// wm.forEach(() => {}); // TypeError

// 不能使用 for...of
// for (const item of wm) {} // TypeError

// 键被垃圾回收后，对应的键值对也会被清除
let obj = { id: 1 };
wm.set(obj, 'value');
obj = null; // 对象被垃圾回收，WeakMap 中的键值对也会被清除
```

### WeakMap 应用示例

#### 私有属性

```javascript
const privateData = new WeakMap();

class Person {
  constructor(name, age) {
    privateData.set(this, { name, age });
  }
  
  getName() {
    return privateData.get(this).name;
  }
  
  getAge() {
    return privateData.get(this).age;
  }
}

const person = new Person('John', 25);
console.log(person.getName()); // 'John'
console.log(person.getAge()); // 25
console.log(person.name); // undefined（私有）
```

#### DOM 节点数据

```javascript
const domData = new WeakMap();

function setData(element, data) {
  domData.set(element, data);
}

function getData(element) {
  return domData.get(element);
}

const div = document.createElement('div');
setData(div, { clickCount: 0 });

div.addEventListener('click', () => {
  const data = getData(div);
  data.clickCount++;
  console.log(`点击次数: ${data.clickCount}`);
});
```

#### 缓存计算结果

```javascript
const cache = new WeakMap();

function heavyComputation(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }
  
  // 复杂计算
  const result = Object.keys(obj).length * 2;
  cache.set(obj, result);
  
  return result;
}

const obj1 = { a: 1, b: 2, c: 3 };
const obj2 = { x: 1, y: 2 };

console.log(heavyComputation(obj1)); // 6
console.log(heavyComputation(obj1)); // 6（从缓存获取）
console.log(heavyComputation(obj2)); // 4
```

## 最佳实践

1. **使用 Set 进行去重**：替代数组手动去重
2. **使用 Map 存储键值对**：键可以是任意类型
3. **使用 WeakSet/WeakMap 避免内存泄漏**：弱引用不会阻止垃圾回收
4. **使用 Map 的 size 属性**：比 Object.keys(obj).length 更高效

```javascript
// 好的做法
const unique = [...new Set(array)];
const map = new Map([[obj, value]]);

// 避免
const unique2 = array.filter((item, index) => array.indexOf(item) === index);
const obj = { [objKey]: value }; // 键会被转换为字符串
```

## 参考

- [MDN Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)
- [MDN Map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)
