# 装饰器（Decorators）

## 概述

装饰器是一种特殊类型的声明，可以附加到类、方法、属性或参数上，用于修改或扩展其行为。

## 启用装饰器

```json
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

## 类装饰器

### 基本语法

```typescript
// 类装饰器
function Sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@Sealed
class User {
  constructor(public name: string) {}
}
```

### 带参数的类装饰器

```typescript
// 带参数的装饰器工厂
function Component(options: { selector: string; template: string }) {
  return function(constructor: Function) {
    constructor.prototype.selector = options.selector;
    constructor.prototype.template = options.template;
  };
}

@Component({
  selector: 'app-user',
  template: '<div>{{name}}</div>'
})
class UserComponent {
  name: string = 'John';
}
```

### 类装饰器继承

```typescript
// 装饰器返回新类
function Timestamped<T extends { new (...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    timestamp = Date.now();
  };
}

@Timestamped
class User {
  name = 'John';
}

const user = new User();
console.log(user.timestamp); // 时间戳
```

## 方法装饰器

### 基本语法

```typescript
// 方法装饰器
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @Log
  add(a: number, b: number): number {
    return a + b;
  }
}
```

### 方法装饰器示例

```typescript
// 性能监控装饰器
function Measure(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const start = performance.now();
    const result = originalMethod.apply(this, args);
    const end = performance.now();
    console.log(`${propertyKey} took ${end - start}ms`);
    return result;
  };
  
  return descriptor;
}

class DataService {
  @Measure
  processData(data: any[]): any[] {
    return data.map(item => item * 2);
  }
}
```

## 属性装饰器

### 基本语法

```typescript
// 属性装饰器
function Required(target: any, propertyKey: string) {
  // 元数据
  const requiredFields = Reflect.getMetadata('required', target) || [];
  requiredFields.push(propertyKey);
  Reflect.defineMetadata('required', requiredFields, target);
}

class User {
  @Required
  name: string;
  
  @Required
  email: string;
  
  age?: number;
}
```

## 参数装饰器

### 基本语法

```typescript
// 参数装饰器
function Validate(target: any, propertyKey: string, parameterIndex: number) {
  const existingValidators = Reflect.getMetadata('validators', target, propertyKey) || [];
  existingValidators.push(parameterIndex);
  Reflect.defineMetadata('validators', existingValidators, target, propertyKey);
}

class UserService {
  createUser(@Validate name: string, @Validate email: string): void {
    // ...
  }
}
```

## 实际应用示例

### 1. 日志装饰器

```typescript
// 日志装饰器
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`[${propertyKey}] 调用参数:`, args);
    try {
      const result = originalMethod.apply(this, args);
      console.log(`[${propertyKey}] 返回结果:`, result);
      return result;
    } catch (error) {
      console.error(`[${propertyKey}] 错误:`, error);
      throw error;
    }
  };
  
  return descriptor;
}

class UserService {
  @Log
  getUser(id: number): User {
    return { id, name: 'John' };
  }
}
```

### 2. 缓存装饰器

```typescript
// 缓存装饰器
function Cache(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const cache = new Map<string, any>();
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('从缓存返回');
      return cache.get(key);
    }
    
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };
  
  return descriptor;
}

class DataService {
  @Cache
  fetchData(id: number): any {
    console.log('从服务器获取数据');
    return { id, data: 'some data' };
  }
}
```

### 3. 验证装饰器

```typescript
// 验证装饰器
function Validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    // 验证逻辑
    for (const arg of args) {
      if (arg === null || arg === undefined) {
        throw new Error(`参数不能为 null 或 undefined`);
      }
    }
    
    return originalMethod.apply(this, args);
  };
  
  return descriptor;
}

class UserService {
  @Validate
  createUser(name: string, email: string): void {
    console.log('创建用户:', name, email);
  }
}
```

### 4. 重试装饰器

```typescript
// 重试装饰器
function Retry(maxAttempts: number = 3, delay: number = 1000) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function(...args: any[]) {
      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          if (attempt === maxAttempts) {
            throw error;
          }
          console.log(`第 ${attempt} 次尝试失败，${delay}ms 后重试...`);
          await new Promise(resolve => setTimeout(resolve, delay));
        }
      }
    };
    
    return descriptor;
  };
}

class ApiService {
  @Retry(3, 1000)
  async fetchData(url: string): Promise<any> {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  }
}
```

## 最佳实践

1. **使用装饰器工厂**：允许传参
2. **保持装饰器简单**：单一职责
3. **注意装饰器顺序**：从下往上执行
4. **使用元数据**：配合 reflect-metadata

```typescript
// 好的做法
function Log(prefix: string) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    descriptor.value = function(...args: any[]) {
      console.log(`${prefix} ${propertyKey}`, args);
      return originalMethod.apply(this, args);
    };
  };
}

// 避免
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  // 过于复杂
}
```

## 参考

- [TypeScript Handbook - Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
