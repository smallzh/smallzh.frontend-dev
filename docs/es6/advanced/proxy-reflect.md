# Proxy 和 Reflect

## 概述

Proxy 和 Reflect 是 ES6 引入的元编程特性，允许拦截和自定义对象的基本操作。

## Proxy

### 基本概念

Proxy 用于创建一个对象的代理，从而实现基本操作的拦截和自定义。

### 创建 Proxy

```javascript
const target = {
  name: 'John',
  age: 25
};

const handler = {
  get(target, property, receiver) {
    console.log(`访问属性: ${property}`);
    return target[property];
  },
  
  set(target, property, value, receiver) {
    console.log(`设置属性: ${property} = ${value}`);
    target[property] = value;
    return true;
  }
};

const proxy = new Proxy(target, handler);

console.log(proxy.name); // 访问属性: name, John
proxy.age = 30; // 设置属性: age = 30
```

### 处理器方法

#### get

拦截属性读取操作：

```javascript
const handler = {
  get(target, property, receiver) {
    if (property in target) {
      return target[property];
    }
    return `属性 ${property} 不存在`;
  }
};

const proxy = new Proxy({}, handler);
proxy.name = 'John';

console.log(proxy.name); // 'John'
console.log(proxy.age); // '属性 age 不存在'
```

#### set

拦截属性设置操作：

```javascript
const handler = {
  set(target, property, value, receiver) {
    if (typeof value !== 'string') {
      throw new TypeError('值必须是字符串');
    }
    target[property] = value;
    return true;
  }
};

const proxy = new Proxy({}, handler);
proxy.name = 'John'; // 正常
// proxy.age = 25; // TypeError: 值必须是字符串
```

#### has

拦截 `in` 操作符：

```javascript
const handler = {
  has(target, property) {
    console.log(`检查属性: ${property}`);
    return property in target;
  }
};

const proxy = new Proxy({ name: 'John' }, handler);

console.log('name' in proxy); // 检查属性: name, true
console.log('age' in proxy); // 检查属性: age, false
```

#### deleteProperty

拦截 `delete` 操作符：

```javascript
const handler = {
  deleteProperty(target, property) {
    if (property === 'name') {
      throw new Error('不能删除 name 属性');
    }
    delete target[property];
    return true;
  }
};

const proxy = new Proxy({ name: 'John', age: 25 }, handler);

delete proxy.age; // 正常
// delete proxy.name; // Error: 不能删除 name 属性
```

#### apply

拦截函数调用：

```javascript
function sum(a, b) {
  return a + b;
}

const handler = {
  apply(target, thisArg, argumentsList) {
    console.log(`调用函数: 参数 = ${argumentsList}`);
    const result = target.apply(thisArg, argumentsList);
    console.log(`返回结果: ${result}`);
    return result;
  }
};

const proxy = new Proxy(sum, handler);

console.log(proxy(1, 2));
// 调用函数: 参数 = 1,2
// 返回结果: 3
// 3
```

#### construct

拦截 `new` 操作符：

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

const handler = {
  construct(target, args, newTarget) {
    console.log(`创建实例: 参数 = ${args}`);
    return new target(...args);
  }
};

const proxy = new Proxy(Person, handler);

const person = new proxy('John', 25);
// 创建实例: 参数 = John,25
console.log(person.name); // 'John'
```

#### 其他处理器方法

```javascript
const handler = {
  // 拦截 Object.defineProperty
  defineProperty(target, property, descriptor) {
    console.log(`定义属性: ${property}`);
    return Reflect.defineProperty(target, property, descriptor);
  },
  
  // 拦截 Object.getOwnPropertyDescriptor
  getOwnPropertyDescriptor(target, property) {
    console.log(`获取属性描述: ${property}`);
    return Reflect.getOwnPropertyDescriptor(target, property);
  },
  
  // 拦截 Object.getPrototypeOf
  getPrototypeOf(target) {
    console.log('获取原型');
    return Reflect.getPrototypeOf(target);
  },
  
  // 拦截 Object.setPrototypeOf
  setPrototypeOf(target, prototype) {
    console.log('设置原型');
    return Reflect.setPrototypeOf(target, prototype);
  },
  
  // 拦截 Object.preventExtensions
  preventExtensions(target) {
    console.log('阻止扩展');
    return Reflect.preventExtensions(target);
  },
  
  // 拦截 Object.isExtensible
  isExtensible(target) {
    console.log('检查是否可扩展');
    return Reflect.isExtensible(target);
  },
  
  // 拦截 Object.keys、Object.values 等
  ownKeys(target) {
    console.log('获取自身属性键');
    return Reflect.ownKeys(target);
  }
};
```

## Reflect

### 基本概念

Reflect 是一个内置对象，提供了拦截 JavaScript 操作的方法，与 Proxy 的处理器方法一一对应。

### 静态方法

#### Reflect.get()

```javascript
const obj = { name: 'John', age: 25 };

console.log(Reflect.get(obj, 'name')); // 'John'
console.log(Reflect.get(obj, 'age')); // 25
console.log(Reflect.get(obj, 'email')); // undefined

// 带有 receiver
const obj2 = {
  name: 'John',
  get greeting() {
    return `Hello, ${this.name}`;
  }
};

const receiver = { name: 'Jane' };
console.log(Reflect.get(obj2, 'greeting', receiver)); // 'Hello, Jane'
```

#### Reflect.set()

```javascript
const obj = { name: 'John' };

Reflect.set(obj, 'age', 25);
console.log(obj); // { name: 'John', age: 25 }

// 带有 receiver
const obj2 = {
  set age(value) {
    this._age = value * 2;
  }
};

const receiver = {};
Reflect.set(obj2, 'age', 25, receiver);
console.log(receiver); // { _age: 50 }
```

#### Reflect.has()

```javascript
const obj = { name: 'John', age: 25 };

console.log(Reflect.has(obj, 'name')); // true
console.log(Reflect.has(obj, 'email')); // false
console.log(Reflect.has(obj, 'toString')); // true（继承的属性）
```

#### Reflect.deleteProperty()

```javascript
const obj = { name: 'John', age: 25 };

console.log(Reflect.deleteProperty(obj, 'age')); // true
console.log(obj); // { name: 'John' }

console.log(Reflect.deleteProperty(obj, 'email')); // true（不存在的属性也返回 true）
```

#### Reflect.construct()

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

const person = Reflect.construct(Person, ['John', 25]);
console.log(person.name); // 'John'
console.log(person instanceof Person); // true
```

#### Reflect.apply()

```javascript
function sum(a, b) {
  return a + b;
}

console.log(Reflect.apply(sum, null, [1, 2])); // 3

// 替代 Function.prototype.apply
const numbers = [5, 2, 8, 1, 9];
const max = Reflect.apply(Math.max, null, numbers);
console.log(max); // 9
```

#### Reflect.defineProperty()

```javascript
const obj = {};

const success = Reflect.defineProperty(obj, 'name', {
  value: 'John',
  writable: false,
  enumerable: true,
  configurable: false
});

console.log(success); // true
console.log(obj.name); // 'John'

obj.name = 'Jane'; // 无效，因为 writable: false
console.log(obj.name); // 'John'
```

#### Reflect.getOwnPropertyDescriptor()

```javascript
const obj = { name: 'John' };
Object.defineProperty(obj, 'age', { value: 25, writable: false });

const descriptor = Reflect.getOwnPropertyDescriptor(obj, 'age');
console.log(descriptor);
// { value: 25, writable: false, enumerable: true, configurable: true }

const descriptor2 = Reflect.getOwnPropertyDescriptor(obj, 'email');
console.log(descriptor2); // undefined
```

#### Reflect.getPrototypeOf()

```javascript
const obj = { name: 'John' };
const proto = Reflect.getPrototypeOf(obj);

console.log(proto === Object.prototype); // true
console.log(Reflect.getPrototypeOf(proto) === null); // true
```

#### Reflect.setPrototypeOf()

```javascript
const obj = {};
const proto = { greet() { return 'Hello'; } };

const success = Reflect.setPrototypeOf(obj, proto);
console.log(success); // true
console.log(obj.greet()); // 'Hello'
```

#### Reflect.preventExtensions() 和 Reflect.isExtensible()

```javascript
const obj = { name: 'John' };

console.log(Reflect.isExtensible(obj)); // true

Reflect.preventExtensions(obj);
console.log(Reflect.isExtensible(obj)); // false

obj.age = 25; // 无效
console.log(obj); // { name: 'John' }
```

#### Reflect.ownKeys()

```javascript
const obj = { name: 'John', age: 25 };
Object.defineProperty(obj, 'hidden', { value: 'secret', enumerable: false });

const sym = Symbol('test');
obj[sym] = 'symbol value';

console.log(Reflect.ownKeys(obj));
// ['name', 'age', 'hidden', Symbol(test)]

console.log(Object.keys(obj)); // ['name', 'age']（只返回可枚举的字符串属性）
```

## Proxy 和 Reflect 结合使用

### 基本模式

```javascript
const target = { name: 'John', age: 25 };

const handler = {
  get(target, property, receiver) {
    console.log(`获取 ${property}`);
    return Reflect.get(target, property, receiver);
  },
  set(target, property, value, receiver) {
    console.log(`设置 ${property} = ${value}`);
    return Reflect.set(target, property, value, receiver);
  }
};

const proxy = new Proxy(target, handler);

console.log(proxy.name); // 获取 name, John
proxy.age = 30; // 设置 age = 30
```

### 验证对象

```javascript
function createValidatedObject(target, validators) {
  return new Proxy(target, {
    set(target, property, value, receiver) {
      const validator = validators[property];
      
      if (validator) {
        if (!validator.validate(value)) {
          throw new Error(validator.message);
        }
      }
      
      return Reflect.set(target, property, value, receiver);
    }
  });
}

const validators = {
  name: {
    validate: value => typeof value === 'string' && value.length > 0,
    message: 'name 必须是非空字符串'
  },
  age: {
    validate: value => Number.isInteger(value) && value >= 0 && value <= 150,
    message: 'age 必须是 0-150 之间的整数'
  }
};

const user = createValidatedObject({}, validators);

user.name = 'John'; // 正常
user.age = 25; // 正常
// user.age = -1; // Error: age 必须是 0-150 之间的整数
// user.age = '25'; // Error: age 必须是 0-150 之间的整数
```

### 响应式对象

```javascript
function createReactiveObject(target, onChange) {
  return new Proxy(target, {
    get(target, property, receiver) {
      const value = Reflect.get(target, property, receiver);
      
      // 递归代理嵌套对象
      if (value && typeof value === 'object') {
        return createReactiveObject(value, (nestedProperty, oldValue, newValue) => {
          onChange(`${property}.${nestedProperty}`, oldValue, newValue);
        });
      }
      
      return value;
    },
    
    set(target, property, value, receiver) {
      const oldValue = target[property];
      const success = Reflect.set(target, property, value, receiver);
      
      if (oldValue !== value) {
        onChange(property, oldValue, value);
      }
      
      return success;
    }
  });
}

const state = createReactiveObject({ 
  user: { name: 'John', age: 25 },
  count: 0
}, (property, oldValue, newValue) => {
  console.log(`${property}: ${oldValue} -> ${newValue}`);
});

state.count = 1; // count: 0 -> 1
state.user.name = 'Jane'; // user.name: John -> Jane
state.user.age = 30; // user.age: 25 -> 30
```

### 只读对象

```javascript
function createReadonlyObject(target) {
  return new Proxy(target, {
    set(target, property, value, receiver) {
      throw new Error(`对象是只读的，不能设置属性 ${property}`);
    },
    
    deleteProperty(target, property) {
      throw new Error(`对象是只读的，不能删除属性 ${property}`);
    },
    
    defineProperty(target, property, descriptor) {
      throw new Error(`对象是只读的，不能定义属性 ${property}`);
    }
  });
}

const config = createReadonlyObject({
  apiUrl: 'https://api.example.com',
  timeout: 5000
});

console.log(config.apiUrl); // 'https://api.example.com'
// config.apiUrl = 'https://other.com'; // Error: 对象是只读的
// delete config.apiUrl; // Error: 对象是只读的
```

## 实际应用示例

### 1. 属性访问日志

```javascript
function createLoggedObject(target, name = 'Object') {
  return new Proxy(target, {
    get(target, property, receiver) {
      const value = Reflect.get(target, property, receiver);
      console.log(`[${name}] 访问属性: ${property} = ${value}`);
      return value;
    },
    
    set(target, property, value, receiver) {
      console.log(`[${name}] 设置属性: ${property} = ${value}`);
      return Reflect.set(target, property, value, receiver);
    }
  });
}

const user = createLoggedObject({ name: 'John', age: 25 }, 'User');
console.log(user.name); // [User] 访问属性: name = John
user.age = 30; // [User] 设置属性: age = 30
```

### 2. 自动类型转换

```javascript
function createAutoConvertObject(target) {
  return new Proxy(target, {
    set(target, property, value, receiver) {
      const descriptor = Reflect.getOwnPropertyDescriptor(target, property);
      
      if (descriptor && descriptor.type) {
        switch (descriptor.type) {
          case 'number':
            value = Number(value);
            break;
          case 'string':
            value = String(value);
            break;
          case 'boolean':
            value = Boolean(value);
            break;
        }
      }
      
      return Reflect.set(target, property, value, receiver);
    }
  });
}

const config = createAutoConvertObject({
  port: { value: 3000, type: 'number' },
  host: { value: 'localhost', type: 'string' },
  debug: { value: false, type: 'boolean' }
});

config.port = '8080'; // 自动转换为数字
config.debug = 1; // 自动转换为 true

console.log(config.port); // 8080
console.log(config.debug); // true
```

### 3. 数组边界检查

```javascript
function createBoundedArray(length) {
  const array = new Array(length).fill(0);
  
  return new Proxy(array, {
    get(target, property, receiver) {
      const index = Number(property);
      
      if (!isNaN(index) && (index < 0 || index >= length)) {
        throw new RangeError(`索引 ${index} 超出边界 [0, ${length - 1}]`);
      }
      
      return Reflect.get(target, property, receiver);
    },
    
    set(target, property, value, receiver) {
      const index = Number(property);
      
      if (!isNaN(index) && (index < 0 || index >= length)) {
        throw new RangeError(`索引 ${index} 超出边界 [0, ${length - 1}]`);
      }
      
      return Reflect.set(target, property, value, receiver);
    }
  });
}

const boundedArray = createBoundedArray(5);
boundedArray[0] = 1; // 正常
boundedArray[4] = 5; // 正常
// boundedArray[5] = 6; // RangeError: 索引 5 超出边界 [0, 4]
// boundedArray[-1] = -1; // RangeError: 索引 -1 超出边界 [0, 4]
```

### 4. 函数参数验证

```javascript
function validateArgs(...validators) {
  return function(target, thisArg, argumentsList) {
    for (let i = 0; i < validators.length; i++) {
      const validator = validators[i];
      const arg = argumentsList[i];
      
      if (!validator.validate(arg)) {
        throw new Error(`参数 ${i}: ${validator.message}`);
      }
    }
    
    return Reflect.apply(target, thisArg, argumentsList);
  };
}

function add(a, b) {
  return a + b;
}

const validators = [
  { validate: Number.isFinite, message: '必须是数字' },
  { validate: Number.isFinite, message: '必须是数字' }
];

const validatedAdd = new Proxy(add, {
  apply: validateArgs(...validators)
});

console.log(validatedAdd(1, 2)); // 3
// validatedAdd('1', 2); // Error: 参数 0: 必须是数字
```

## 最佳实践

1. **使用 Reflect 方法**：确保代理行为正确
2. **处理返回值**：set 处理器必须返回 boolean
3. **避免无限递归**：在 get/set 中谨慎访问代理对象
4. **性能考虑**：代理会增加性能开销

```javascript
// 好的做法
const handler = {
  get(target, property, receiver) {
    // 使用 Reflect.get 确保正确行为
    return Reflect.get(target, property, receiver);
  },
  
  set(target, property, value, receiver) {
    // 验证逻辑
    if (typeof value !== 'string') {
      return false; // 返回 false 表示设置失败
    }
    
    // 使用 Reflect.set
    return Reflect.set(target, property, value, receiver);
  }
};

// 避免
const badHandler = {
  get(target, property) {
    // 直接访问可能导致无限递归
    return target[property];
  },
  
  set(target, property, value) {
    // 没有返回值会导致错误
    target[property] = value;
  }
};
```

## 参考

- [MDN Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
- [MDN Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
