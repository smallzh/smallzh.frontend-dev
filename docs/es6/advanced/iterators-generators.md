# 迭代器和生成器（Iterators and Generators）

## 概述

迭代器（Iterator）是一种接口，为各种不同的数据结构提供统一的访问机制。生成器（Generator）是 ES6 提供的一种异步编程解决方案。

## 迭代器（Iterator）

### 迭代器协议

迭代器是一个对象，它满足迭代器协议：

```javascript
const iterator = {
  next() {
    return {
      value: any, // 当前迭代的值
      done: boolean // 是否迭代完成
    };
  }
};
```

### 基本示例

```javascript
// 创建一个简单的迭代器
function createRangeIterator(start, end, step = 1) {
  let current = start;
  
  return {
    next() {
      if (current < end) {
        const value = current;
        current += step;
        return { value, done: false };
      }
      return { done: true };
    }
  };
}

const iterator = createRangeIterator(1, 5);
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: 4, done: false }
console.log(iterator.next()); // { done: true }
```

### 可迭代对象（Iterable）

可迭代对象实现了 `Symbol.iterator` 方法：

```javascript
const myIterable = {
  [Symbol.iterator]() {
    let step = 0;
    return {
      next() {
        step++;
        if (step <= 3) {
          return { value: step * 10, done: false };
        }
        return { done: true };
      }
    };
  }
};

// 使用 for...of
for (const value of myIterable) {
  console.log(value); // 10, 20, 30
}

// 使用展开运算符
console.log([...myIterable]); // [10, 20, 30]

// 使用解构赋值
const [a, b, c] = myIterable;
console.log(a, b, c); // 10 20 30
```

### 内置可迭代对象

```javascript
// 数组
const arr = [1, 2, 3];
for (const item of arr) {
  console.log(item);
}

// 字符串
const str = 'hello';
for (const char of str) {
  console.log(char);
}

// Map
const map = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of map) {
  console.log(key, value);
}

// Set
const set = new Set([1, 2, 3]);
for (const value of set) {
  console.log(value);
}

// arguments 对象
function example() {
  for (const arg of arguments) {
    console.log(arg);
  }
}
```

## 生成器（Generator）

### 基本语法

```javascript
// 生成器函数
function* generatorFunction() {
  yield 1;
  yield 2;
  yield 3;
}

const generator = generatorFunction();
console.log(generator.next()); // { value: 1, done: false }
console.log(generator.next()); // { value: 2, done: false }
console.log(generator.next()); // { value: 3, done: false }
console.log(generator.next()); // { value: undefined, done: true }
```

### 生成器特性

#### 惰性求值

```javascript
function* infiniteSequence() {
  let i = 0;
  while (true) {
    yield i++;
  }
}

const infinite = infiniteSequence();
console.log(infinite.next().value); // 0
console.log(infinite.next().value); // 1
console.log(infinite.next().value); // 2
// 可以无限继续
```

#### 可暂停和恢复

```javascript
function* pausedGenerator() {
  console.log('开始');
  yield 1;
  console.log('继续');
  yield 2;
  console.log('结束');
}

const gen = pausedGenerator();
console.log('调用 next()');
gen.next(); // 开始
console.log('再次调用 next()');
gen.next(); // 继续
gen.next(); // 结束
```

### yield 表达式

```javascript
function* calculator() {
  const x = yield '输入第一个数';
  const y = yield '输入第二个数';
  yield x + y;
}

const calc = calculator();
console.log(calc.next().value); // '输入第一个数'
console.log(calc.next(10).value); // '输入第二个数'
console.log(calc.next(20).value); // 30
```

### 生成器委托

```javascript
function* generator1() {
  yield 1;
  yield 2;
}

function* generator2() {
  yield 3;
  yield 4;
}

function* combined() {
  yield* generator1();
  yield* generator2();
}

for (const value of combined()) {
  console.log(value); // 1, 2, 3, 4
}
```

### 生成器与迭代器

```javascript
// 生成器自动实现迭代器协议
function* range(start, end) {
  for (let i = start; i < end; i++) {
    yield i;
  }
}

// 可以使用 for...of
for (const num of range(1, 5)) {
  console.log(num); // 1, 2, 3, 4
}

// 可以使用展开运算符
console.log([...range(1, 5)]); // [1, 2, 3, 4]

// 可以使用解构赋值
const [first, second, ...rest] = range(1, 6);
console.log(first, second, rest); // 1 2 [3, 4, 5]
```

## 实际应用示例

### 1. 自定义可迭代对象

```javascript
class Collection {
  constructor(items) {
    this.items = items;
  }
  
  *[Symbol.iterator]() {
    for (const item of this.items) {
      yield item;
    }
  }
  
  *filter(predicate) {
    for (const item of this) {
      if (predicate(item)) {
        yield item;
      }
    }
  }
  
  *map(transform) {
    for (const item of this) {
      yield transform(item);
    }
  }
}

const collection = new Collection([1, 2, 3, 4, 5]);

// 使用迭代器
for (const item of collection) {
  console.log(item);
}

// 使用 filter 和 map
const result = collection
  .filter(x => x % 2 === 0)
  .map(x => x * 2);

console.log([...result]); // [4, 8]
```

### 2. 异步迭代器

```javascript
class AsyncRange {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  
  [Symbol.asyncIterator]() {
    let current = this.start;
    const end = this.end;
    
    return {
      async next() {
        await new Promise(resolve => setTimeout(resolve, 100));
        
        if (current < end) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
}

// 使用异步迭代器
async function main() {
  const range = new AsyncRange(1, 5);
  
  for await (const value of range) {
    console.log(value); // 每 100ms 输出 1, 2, 3, 4
  }
}

main();
```

### 3. 生成器实现协程

```javascript
function* taskRunner() {
  console.log('任务 1');
  yield;
  console.log('任务 2');
  yield;
  console.log('任务 3');
}

const task = taskRunner();
task.next(); // 任务 1
task.next(); // 任务 2
task.next(); // 任务 3
```

### 4. 状态机

```javascript
function* stateMachine() {
  while (true) {
    const action = yield;
    
    switch (action) {
      case 'INCREMENT':
        console.log('递增');
        break;
      case 'DECREMENT':
        console.log('递减');
        break;
      case 'RESET':
        console.log('重置');
        break;
    }
  }
}

const machine = stateMachine();
machine.next(); // 初始化
machine.next('INCREMENT'); // 递增
machine.next('DECREMENT'); // 递减
machine.next('RESET'); // 重置
```

### 5. 数据流处理

```javascript
function* readLines(text) {
  const lines = text.split('\n');
  for (const line of lines) {
    yield line.trim();
  }
}

function* filterLines(lines, pattern) {
  for (const line of lines) {
    if (line.includes(pattern)) {
      yield line;
    }
  }
}

function* take(lines, count) {
  let taken = 0;
  for (const line of lines) {
    if (taken >= count) break;
    yield line;
    taken++;
  }
}

const text = `
  Apple is red
  Banana is yellow
  Apple is delicious
  Orange is orange
  Apple is sweet
`;

const lines = readLines(text);
const appleLines = filterLines(lines, 'Apple');
const firstTwo = take(appleLines, 2);

console.log([...firstTwo]);
// ['Apple is red', 'Apple is delicious']
```

### 6. 斐波那契数列

```function* fibonacci() {
  let a = 0;
  let b = 1;
  
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const fib = fibonacci();
console.log(fib.next().value); // 0
console.log(fib.next().value); // 1
console.log(fib.next().value); // 1
console.log(fib.next().value); // 2
console.log(fib.next().value); // 3
console.log(fib.next().value); // 5

// 获取前 10 个斐波那契数
const first10 = [];
const fib2 = fibonacci();
for (let i = 0; i < 10; i++) {
  first10.push(fib2.next().value);
}
console.log(first10); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

## 高级用法

### 1. 生成器组合

```javascript
function* map(iterable, transform) {
  for (const item of iterable) {
    yield transform(item);
  }
}

function* filter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) {
      yield item;
    }
  }
}

function* take(iterable, count) {
  let taken = 0;
  for (const item of iterable) {
    if (taken >= count) break;
    yield item;
    taken++;
  }
}

// 组合使用
const numbers = function* () {
  for (let i = 1; i <= 100; i++) {
    yield i;
  }
};

const result = take(
  filter(
    map(numbers(), x => x * 2),
    x => x % 3 === 0
  ),
  5
);

console.log([...result]); // [6, 12, 18, 24, 30]
```

### 2. 错误处理

```function* errorHandling() {
  try {
    const result = yield riskyOperation();
    console.log('成功:', result);
  } catch (error) {
    console.log('捕获错误:', error.message);
  }
}

function riskyOperation() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.5) {
        resolve('操作成功');
      } else {
        reject(new Error('操作失败'));
      }
    }, 1000);
  });
}

const gen = errorHandling();
gen.next().value
  .then(result => gen.next(result))
  .catch(error => gen.throw(error));
```

### 3. 异步流程控制

```javascript
function run(generatorFunction) {
  const generator = generatorFunction();
  
  function handle(result) {
    if (result.done) return Promise.resolve(result.value);
    
    return Promise.resolve(result.value)
      .then(res => handle(generator.next(res)))
      .catch(err => handle(generator.throw(err)));
  }
  
  return handle(generator.next());
}

// 使用
function* fetchUserData() {
  try {
    const user = yield fetch('/api/user').then(r => r.json());
    const posts = yield fetch(`/api/posts/${user.id}`).then(r => r.json());
    const comments = yield fetch(`/api/comments/${posts[0].id}`).then(r => r.json());
    
    return { user, posts, comments };
  } catch (error) {
    console.error('获取数据失败:', error);
    throw error;
  }
}

run(fetchUserData)
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

## 注意事项

### 1. 生成器只能暂停，不能中断

```javascript
function* infinite() {
  let i = 0;
  while (true) {
    yield i++;
  }
}

const gen = infinite();
console.log(gen.next().value); // 0
console.log(gen.next().value); // 1
// 生成器会一直运行，需要外部控制
```

### 2. 生成器的 return 方法

```javascript
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.return('结束')); // { value: '结束', done: true }
console.log(gen.next()); // { value: undefined, done: true }
```

### 3. 生成器的 throw 方法

```javascript
function* generator() {
  try {
    yield 1;
    yield 2;
  } catch (error) {
    console.log('捕获错误:', error.message);
  }
  yield 3;
}

const gen = generator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.throw(new Error('测试错误')));
// 捕获错误: 测试错误
// { value: 3, done: false }
```

## 最佳实践

1. **使用生成器处理大量数据**：惰性求值节省内存
2. **使用迭代器协议**：使自定义对象可迭代
3. **组合生成器**：使用 `yield*` 委托
4. **处理异步操作**：使用生成器实现协程

```javascript
// 好的做法
function* processLargeDataset(data) {
  for (const item of data) {
    if (item.isValid) {
      yield transform(item);
    }
  }
}

// 处理大量数据时不会占用太多内存
const processed = processLargeDataset(largeArray);
for (const item of processed) {
  // 逐个处理
}

// 避免
const processed = largeArray
  .filter(item => item.isValid)
  .map(transform); // 一次性创建新数组，占用内存
```

## 参考

- [MDN Iterators and Generators](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators)
