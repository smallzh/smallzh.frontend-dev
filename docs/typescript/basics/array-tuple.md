# 数组和元组（Array and Tuple）

## 概述

TypeScript 提供了丰富的数组和元组类型定义方式，用于精确描述数据结构。

## 数组类型

### 基本语法

```typescript
// 类型 + 方括号
let numbers: number[] = [1, 2, 3, 4, 5];
let strings: string[] = ['hello', 'world'];
let booleans: boolean[] = [true, false];

// 泛型语法
let numbers2: Array<number> = [1, 2, 3];
let strings2: Array<string> = ['hello', 'world'];

// 类型推断
let inferred = [1, 2, 3]; // 推断为 number[]
let mixed = [1, 'two', true]; // 推断为 (string | number | boolean)[]
```

### 只读数组

```typescript
// readonly 关键字
let readonlyNumbers: readonly number[] = [1, 2, 3];
// readonlyNumbers.push(4); // 错误
// readonlyNumbers[0] = 10; // 错误

// ReadonlyArray 泛型
let readonlyStrings: ReadonlyArray<string> = ['a', 'b', 'c'];

// as const
const constArray = [1, 2, 3] as const;
// constArray.push(4); // 错误
```

### 多维数组

```typescript
// 二维数组
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// 三维数组
let cube: number[][][] = [
  [[1, 2], [3, 4]],
  [[5, 6], [7, 8]]
];

// 泛型写法
let matrix2: Array<Array<number>> = [[1, 2], [3, 4]];
```

### 数组方法类型

```typescript
// 数组方法返回类型
const numbers: number[] = [1, 2, 3, 4, 5];

// map 返回新类型
const strings: string[] = numbers.map(n => n.toString());

// filter 类型守卫
const items: (string | number)[] = [1, 'two', 3, 'four'];
const numbersOnly: number[] = items.filter((item): item is number => 
  typeof item === 'number'
);

// reduce 指定类型
const sum: number = numbers.reduce((acc, n) => acc + n, 0);
const sum2: number = numbers.reduce<number>((acc, n) => acc + n, 0);
```

## 元组类型

### 基本语法

```typescript
// 元组：固定长度、固定类型的数组
let tuple: [string, number] = ['John', 25];

// 访问元素
let name: string = tuple[0]; // 'John'
let age: number = tuple[1]; // 25

// 类型顺序必须匹配
// let wrong: [string, number] = [25, 'John']; // 错误

// 长度必须匹配
// let wrong2: [string, number] = ['John']; // 错误
// let wrong3: [string, number] = ['John', 25, true]; // 错误
```

### 可选元素

```typescript
// 可选元素（必须放在最后）
let tuple1: [string, number, boolean?] = ['John', 25];
let tuple2: [string, number, boolean?] = ['John', 25, true];

// 多个可选元素
let tuple3: [string, number?, boolean?] = ['John'];
let tuple4: [string, number?, boolean?] = ['John', 25];
let tuple5: [string, number?, boolean?] = ['John', 25, true];
```

### 剩余元素

```typescript
// 剩余元素
let restTuple: [string, ...number[]] = ['John', 1, 2, 3, 4, 5];

// 剩余元素在中间
let mixed: [string, ...number[], boolean] = ['John', 1, 2, 3, true];

// 复杂剩余元素
let complex: [string, ...Array<[number, boolean]>] = ['John', [1, true], [2, false]];
```

### 只读元组

```typescript
// readonly 元组
let readonlyTuple: readonly [string, number] = ['John', 25];
// readonlyTuple[0] = 'Jane'; // 错误

// as const
const constTuple = ['John', 25] as const;
// constTuple[0] = 'Jane'; // 错误
```

### 命名元组

```typescript
// 命名元组（TypeScript 4.0+）
type User = [name: string, age: number, email: string];

const user: User = ['John', 25, 'john@example.com'];

// 解构时可以使用名称
const [userName, userAge, userEmail] = user;
```

## 实际应用示例

### 1. 函数返回多个值

```typescript
// 使用元组返回多个值
function divide(a: number, b: number): [number, number] {
  const quotient = Math.floor(a / b);
  const remainder = a % b;
  return [quotient, remainder];
}

const [q, r] = divide(10, 3);
console.log(q, r); // 3, 1

// 命名元组更清晰
type DivisionResult = [quotient: number, remainder: number];

function divide2(a: number, b: number): DivisionResult {
  return [Math.floor(a / b), a % b];
}
```

### 2. React Hooks

```typescript
// useState 返回元组
function useState<T>(initial: T): [T, (value: T) => void] {
  let state = initial;
  const setState = (newValue: T) => {
    state = newValue;
  };
  return [state, setState];
}

const [count, setCount] = useState(0);
const [name, setName] = useState('John');

// 自定义 Hook 返回元组
function useToggle(initial: boolean): [boolean, () => void] {
  const [value, setValue] = useState(initial);
  const toggle = () => setValue(!value);
  return [value, toggle];
}

const [isOpen, toggleOpen] = useToggle(false);
```

### 3. 配置选项

```typescript
// 配置元组
type DatabaseConfig = [host: string, port: number, database: string];

const config: DatabaseConfig = ['localhost', 5432, 'myapp'];

function connect([host, port, database]: DatabaseConfig): void {
  console.log(`Connecting to ${host}:${port}/${database}`);
}

// 可选配置
type ApiConfig = [
  baseUrl: string,
  timeout?: number,
  retries?: number
];

const apiConfig1: ApiConfig = ['https://api.example.com'];
const apiConfig2: ApiConfig = ['https://api.example.com', 5000];
const apiConfig3: ApiConfig = ['https://api.example.com', 5000, 3];
```

### 4. CSV 数据处理

```typescript
// CSV 行类型
type CsvRow = [id: number, name: string, email: string, age: number];

const data: CsvRow[] = [
  [1, 'John', 'john@example.com', 25],
  [2, 'Jane', 'jane@example.com', 30],
  [3, 'Bob', 'bob@example.com', 35]
];

// 处理 CSV 数据
function processCsv(rows: CsvRow[]): { name: string; email: string }[] {
  return rows.map(([id, name, email, age]) => ({ name, email }));
}

// 过滤数据
function filterByAge(rows: CsvRow[], minAge: number): CsvRow[] {
  return rows.filter(([id, name, email, age]) => age >= minAge);
}
```

### 5. 坐标和范围

```typescript
// 2D 坐标
type Point2D = [x: number, y: number];

// 3D 坐标
type Point3D = [x: number, y: number, z: number];

// 范围
type Range = [start: number, end: number];

// 矩形
type Rect = [x: number, y: number, width: number, height: number];

function isInRect(point: Point2D, rect: Rect): boolean {
  const [px, py] = point;
  const [rx, ry, rw, rh] = rect;
  return px >= rx && px <= rx + rw && py >= ry && py <= ry + rh;
}

// 颜色
type RGB = [r: number, g: number, b: number];
type RGBA = [r: number, g: number, b: number, a: number];
```

### 6. 缓存和映射

```typescript
// 键值对元组
type KeyValuePair<K, V> = [key: K, value: V];

const pairs: KeyValuePair<string, number>[] = [
  ['age', 25],
  ['score', 100],
  ['level', 5]
];

// Map 构造
const map = new Map(pairs);

// 缓存条目
type CacheEntry<T> = [key: string, value: T, timestamp: number];

const cache: CacheEntry<User>[] = [
  ['user:1', { id: 1, name: 'John' }, Date.now()],
  ['user:2', { id: 2, name: 'Jane' }, Date.now()]
];
```

## 最佳实践

1. **使用元组返回多个值**：比对象更轻量
2. **使用命名元组**：提高可读性
3. **使用只读元组**：保护数据不被修改
4. **使用剩余元素**：处理可变长度数据

```typescript
// 好的做法
type FetchResult<T> = [data: T | null, error: Error | null];

async function fetchUser(id: number): Promise<FetchResult<User>> {
  try {
    const user = await getUserById(id);
    return [user, null];
  } catch (error) {
    return [null, error as Error];
  }
}

const [user, error] = await fetchUser(1);

// 避免
async function fetchUserBad(id: number): Promise<any> {
  // 返回类型不明确
}
```

## 参考

- [TypeScript Handbook - Arrays](https://www.typescriptlang.org/docs/handbook/2/objects.html#array-types)
- [TypeScript Handbook - Tuples](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)
