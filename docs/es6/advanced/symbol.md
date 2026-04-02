# Symbol

## 概述

Symbol 是 ES6 引入的新的原始数据类型，表示独一无二的值，主要用于对象属性的唯一标识符。

## 基本用法

### 创建 Symbol

```javascript
// 创建 Symbol
const sym1 = Symbol();
const sym2 = Symbol();

console.log(sym1 === sym2); // false
console.log(typeof sym1); // 'symbol'

// 带描述的 Symbol
const sym3 = Symbol('description');
const sym4 = Symbol('description');

console.log(sym3 === sym4); // false（描述只是标识）
console.log(sym3.toString()); // 'Symbol(description)'
```

### Symbol 作为属性键

```javascript
const mySymbol = Symbol('mySymbol');

const obj = {
  [mySymbol]: 'Hello',
  regularProperty: 'World'
};

console.log(obj[mySymbol]); // 'Hello'
console.log(obj.regularProperty); // 'World'

// Symbol 属性不会被常规方法遍历
console.log(Object.keys(obj)); // ['regularProperty']
console.log(Object.getOwnPropertyNames(obj)); // ['regularProperty']

// 获取 Symbol 属性
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(mySymbol)]
console.log(Reflect.ownKeys(obj)); // ['regularProperty', Symbol(mySymbol)]
```

## 内置 Symbol

### Symbol.iterator

定义对象的默认迭代器：

```javascript
const myIterable = {
  [Symbol.iterator]() {
    let step = 0;
    return {
      next() {
        step++;
        if (step <= 3) {
          return { value: step, done: false };
        }
        return { done: true };
      }
    };
  }
};

for (const value of myIterable) {
  console.log(value); // 1, 2, 3
}

// 展开运算符
console.log([...myIterable]); // [1, 2, 3]
```

### Symbol.toPrimitive

定义对象转换为原始值的行为：

```javascript
const obj = {
  value: 42,
  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case 'number':
        return this.value;
      case 'string':
        return `Value: ${this.value}`;
      default:
        return this.value;
    }
  }
};

console.log(+obj); // 42
console.log(`${obj}`); // 'Value: 42'
console.log(obj + 1); // 43
```

### Symbol.toStringTag

定义对象的默认字符串描述：

```javascript
class CustomCollection {
  get [Symbol.toStringTag]() {
    return 'CustomCollection';
  }
}

const collection = new CustomCollection();
console.log(Object.prototype.toString.call(collection)); // '[object CustomCollection]'
```

### Symbol.hasInstance

定义 instanceof 运算符的行为：

```javascript
class EvenNumber {
  static [Symbol.hasInstance](num) {
    return typeof num === 'number' && num % 2 === 0;
  }
}

console.log(2 instanceof EvenNumber); // true
console.log(3 instanceof EvenNumber); // false
console.log('2' instanceof EvenNumber); // false
```

### Symbol.species

定义派生对象的构造函数：

```javascript
class MyArray extends Array {
  static get [Symbol.species]() {
    return Array; // 返回普通数组而不是 MyArray
  }
}

const myArray = new MyArray(1, 2, 3);
const mapped = myArray.map(x => x * 2);

console.log(mapped instanceof MyArray); // false
console.log(mapped instanceof Array); // true
```

### Symbol.isConcatSpreadable

定义 Array.prototype.concat 的行为：

```javascript
const obj = {
  0: 'a',
  1: 'b',
  length: 2,
  [Symbol.isConcatSpreadable]: true
};

const arr = ['x', 'y'];
console.log(arr.concat(obj)); // ['x', 'y', 'a', 'b']

const obj2 = {
  0: 'a',
  1: 'b',
  length: 2,
  [Symbol.isConcatSpreadable]: false
};

console.log(arr.concat(obj2)); // ['x', 'y', { 0: 'a', 1: 'b', length: 2 }]
```

### Symbol.match、Symbol.replace、Symbol.search、Symbol.split

定义正则表达式方法的行为：

```javascript
const customMatcher = {
  [Symbol.match](string) {
    return string.includes('custom');
  },
  [Symbol.replace](string, replacement) {
    return string.replace('custom', replacement);
  }
};

console.log('custom string'.match(customMatcher)); // true
console.log('custom string'.replace(customMatcher, 'replaced')); // 'replaced string'
```

## Symbol 注册表

### 全局 Symbol

```javascript
// 全局注册表
const globalSym1 = Symbol.for('app.key');
const globalSym2 = Symbol.for('app.key');

console.log(globalSym1 === globalSym2); // true

// 从 Symbol 获取键
console.log(Symbol.keyFor(globalSym1)); // 'app.key'

// 本地 Symbol 没有键
const localSym = Symbol('local');
console.log(Symbol.keyFor(localSym)); // undefined
```

## 实际应用示例

### 1. 私有属性

```javascript
const _name = Symbol('name');
const _age = Symbol('age');

class Person {
  constructor(name, age) {
    this[_name] = name;
    this[_age] = age;
  }
  
  getInfo() {
    return `${this[_name]} is ${this[_age]} years old`;
  }
}

const person = new Person('John', 25);
console.log(person.getInfo()); // 'John is 25 years old'

// 无法直接访问
console.log(person.name); // undefined
console.log(person[_name]); // 'John'（但需要引用 Symbol）
```

### 2. 常量枚举

```javascript
const Colors = {
  RED: Symbol('red'),
  GREEN: Symbol('green'),
  BLUE: Symbol('blue')
};

function getColorName(color) {
  switch (color) {
    case Colors.RED:
      return 'Red';
    case Colors.GREEN:
      return 'Green';
    case Colors.BLUE:
      return 'Blue';
    default:
      return 'Unknown';
  }
}

console.log(getColorName(Colors.RED)); // 'Red'
console.log(getColorName(Symbol('red'))); // 'Unknown'（不同的 Symbol）
```

### 3. 迭代器实现

```javascript
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    
    return {
      next() {
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
}

const range = new Range(1, 5);
for (const num of range) {
  console.log(num); // 1, 2, 3, 4, 5
}

console.log([...range]); // [1, 2, 3, 4, 5]
```

### 4. 自定义类型转换

```javascript
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }
  
  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case 'number':
        return this.amount;
      case 'string':
        return `${this.currency} ${this.amount.toFixed(2)}`;
      default:
        return this.amount;
    }
  }
}

const price = new Money(99.9, 'USD');
console.log(+price); // 99.9
console.log(`${price}`); // 'USD 99.90'
console.log(price + 0.1); // 100
```

### 5. 元编程

```javascript
const validator = {
  [Symbol.hasInstance](instance) {
    return instance && 
           typeof instance.validate === 'function' &&
           typeof instance.errors !== 'undefined';
  }
};

class Form {
  constructor() {
    this.errors = [];
  }
  
  validate() {
    return this.errors.length === 0;
  }
}

const form = new Form();
console.log(form instanceof validator); // true

const obj = { name: 'test' };
console.log(obj instanceof validator); // false
```

## 注意事项

### 1. Symbol 不会被自动转换为字符串

```javascript
const sym = Symbol('test');

// 错误方式
// console.log('Symbol: ' + sym); // TypeError

// 正确方式
console.log('Symbol: ' + sym.toString()); // 'Symbol: Symbol(test)'
console.log('Symbol: ' + String(sym)); // 'Symbol: Symbol(test)'
```

### 2. Symbol 属性不会被 JSON.stringify 序列化

```javascript
const sym = Symbol('test');
const obj = { [sym]: 'value', regular: 'property' };

console.log(JSON.stringify(obj)); // '{"regular":"property"}'
```

### 3. Symbol 属性不会被 for...in 遍历

```javascript
const sym = Symbol('test');
const obj = { [sym]: 'value', regular: 'property' };

for (const key in obj) {
  console.log(key); // 只输出 'regular'
}
```

## 最佳实践

1. **使用 Symbol 作为唯一的属性键**：避免属性名冲突
2. **使用全局 Symbol**：需要跨模块共享 Symbol 时
3. **使用内置 Symbol**：自定义对象行为
4. **避免将 Symbol 转换为字符串**：会失去唯一性

```javascript
// 好的做法
const EVENT_TYPES = {
  CLICK: Symbol('click'),
  HOVER: Symbol('hover'),
  FOCUS: Symbol('focus')
};

class EventEmitter {
  constructor() {
    this.listeners = {
      [EVENT_TYPES.CLICK]: [],
      [EVENT_TYPES.HOVER]: [],
      [EVENT_TYPES.FOCUS]: []
    };
  }
}

// 避免
const events = {
  click: 'click',
  hover: 'hover',
  focus: 'focus'
}; // 容易冲突
```

## 参考

- [MDN Symbol](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
