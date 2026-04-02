# 对象字面量增强（Object Literal Enhancements）

## 概述

ES6 对对象字面量进行了多项增强，包括属性简写、方法简写、计算属性名等。

## 属性简写

### 基本用法

```javascript
// 传统方式
const name = 'John';
const age = 25;
const person = {
  name: name,
  age: age
};

// 属性简写（属性名和变量名相同时）
const person2 = { name, age };
console.log(person2); // { name: 'John', age: 25 }
```

### 实际应用

```javascript
function createUser(name, email, age) {
  return { name, email, age };
}

const user = createUser('John', 'john@example.com', 25);
console.log(user); // { name: 'John', email: 'john@example.com', age: 25 }
```

## 方法简写

### 基本用法

```javascript
// 传统方式
const person = {
  name: 'John',
  greet: function() {
    return `Hello, ${this.name}!`;
  }
};

// 方法简写
const person2 = {
  name: 'John',
  greet() {
    return `Hello, ${this.name}!`;
  }
};

// 箭头函数（注意 this 绑定）
const person3 = {
  name: 'John',
  greet: () => {
    return `Hello, ${this.name}!`; // this 不指向 person3
  }
};
```

### 简写方法的特性

```javascript
const obj = {
  // 简写方法可以使用 super
  parentMethod() {
    return 'parent';
  },
  
  // 简写方法不能用作构造函数
  myMethod() {
    // new this.myMethod() 会报错
  }
};
```

## 计算属性名

### 基本用法

```javascript
// 传统方式
const propName = 'name';
const person = {};
person[propName] = 'John';

// 计算属性名
const person2 = {
  [propName]: 'John'
};
console.log(person2.name); // 'John'
```

### 表达式作为属性名

```javascript
const prefix = 'user';
const suffix = 'Id';

const obj = {
  [`${prefix}_${suffix}`]: 123,
  [prefix + '_' + suffix]: 456,
  ['hello' + 'World']: 'test'
};

console.log(obj.user_Id); // 123
console.log(obj.helloWorld); // 'test'
```

### 方法名计算

```javascript
const methodName = 'sayHello';

const person = {
  name: 'John',
  [methodName]() {
    return `Hello, ${this.name}!`;
  }
};

console.log(person.sayHello()); // Hello, John!
```

## 属性名表达式

### 动态属性名

```javascript
function createObject(key, value) {
  return {
    [key]: value
  };
}

const obj = createObject('status', 'active');
console.log(obj); // { status: 'active' }
```

### Symbol 作为属性名

```javascript
const mySymbol = Symbol('mySymbol');

const obj = {
  [mySymbol]: 'symbol value',
  regularProperty: 'regular value'
};

console.log(obj[mySymbol]); // 'symbol value'
```

## 新增方法

### Object.is()

比较两个值是否严格相等，与 `===` 类似，但处理 `NaN` 和 `+0/-0` 不同：

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

### Object.assign()

将源对象的可枚举属性复制到目标对象：

```javascript
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3, a: 4 };

const result = Object.assign(target, source1, source2);
console.log(result); // { a: 4, b: 2, c: 3 }
console.log(target === result); // true（target 被修改）

// 浅拷贝
const copy = Object.assign({}, source1);
```

#### 注意事项

```javascript
// 只拷贝可枚举属性
const obj = {};
Object.defineProperties(obj, {
  visible: { value: 1, enumerable: true },
  hidden: { value: 2, enumerable: false }
});

const copy = Object.assign({}, obj);
console.log(copy); // { visible: 1 }（hidden 不会被拷贝）

// Symbol 属性也会被拷贝
const sym = Symbol('test');
const source = { [sym]: 'value' };
const copy2 = Object.assign({}, source);
console.log(copy2[sym]); // 'value'
```

### Object.keys()、Object.values()、Object.entries()

```javascript
const person = {
  name: 'John',
  age: 25,
  city: 'New York'
};

// 获取所有键
console.log(Object.keys(person)); // ['name', 'age', 'city']

// 获取所有值
console.log(Object.values(person)); // ['John', 25, 'New York']

// 获取键值对
console.log(Object.entries(person)); 
// [['name', 'John'], ['age', 25], ['city', 'New York']]

// 遍历对象
Object.entries(person).forEach(([key, value]) => {
  console.log(`${key}: ${value}`);
});

// 转换为 Map
const map = new Map(Object.entries(person));
console.log(map); // Map { 'name' => 'John', 'age' => 25, 'city' => 'New York' }
```

## 实际应用示例

### 1. API 响应处理

```javascript
function processUserResponse(data) {
  const { id, name, email, avatar } = data;
  
  return {
    id,
    name,
    email,
    avatar: avatar || '/default-avatar.png',
    displayName: name.toUpperCase(),
    getInitials() {
      return name.split(' ').map(n => n[0]).join('');
    }
  };
}
```

### 2. 配置对象

```javascript
function createConfig(env) {
  const base = {
    apiUrl: 'https://api.example.com',
    timeout: 5000
  };
  
  const configs = {
    development: { debug: true, logLevel: 'verbose' },
    production: { debug: false, logLevel: 'error' },
    test: { debug: true, logLevel: 'silent' }
  };
  
  return {
    ...base,
    ...configs[env],
    [`${env}Mode`]: true
  };
}

const config = createConfig('development');
// { apiUrl: '...', timeout: 5000, debug: true, logLevel: 'verbose', developmentMode: true }
```

### 3. 动态对象创建

```javascript
function createEntity(type, data) {
  const timestamp = Date.now();
  
  return {
    type,
    [`${type}Id`]: data.id,
    ...data,
    createdAt: timestamp,
    updatedAt: timestamp,
    [`get${type.charAt(0).toUpperCase() + type.slice(1)}Info`]() {
      return `${type}: ${data.name}`;
    }
  };
}

const user = createEntity('user', { id: 123, name: 'John' });
console.log(user.userId); // 123
console.log(user.getUserInfo()); // "user: John"
```

### 4. 状态管理

```javascript
function createReducer(initialState, handlers) {
  return function reducer(state = initialState, action) {
    const handler = handlers[action.type];
    return handler ? handler(state, action) : state;
  };
}

const userReducer = createReducer(
  { users: [], loading: false },
  {
    FETCH_USERS: (state) => ({ ...state, loading: true }),
    FETCH_USERS_SUCCESS: (state, action) => ({
      ...state,
      loading: false,
      users: action.payload
    }),
    FETCH_USERS_ERROR: (state, action) => ({
      ...state,
      loading: false,
      error: action.error
    })
  }
);
```

## 最佳实践

1. **使用属性简写**：当属性名和变量名相同时
2. **使用方法简写**：定义对象方法时
3. **合理使用计算属性名**：处理动态属性
4. **注意 Object.assign() 的浅拷贝特性**
5. **使用 Object.entries() 遍历对象**

```javascript
// 好的做法
const createUser = (name, age) => ({ name, age });

const actions = {
  increment: (state) => ({ ...state, count: state.count + 1 }),
  decrement: (state) => ({ ...state, count: state.count - 1 }),
  reset: () => ({ count: 0 })
};

// 避免
const bad = {
  name: name,
  age: age,
  greet: function() { return 'Hello'; }
};
```

## 参考

- [MDN 对象初始化](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Object_initializer)
- [MDN Object.assign()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
