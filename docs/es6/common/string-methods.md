# 字符串新方法（String Methods）

## 概述

ES6 为字符串引入了多个新方法，使字符串操作更加方便和直观。

## 查找方法

### String.prototype.includes()

判断字符串是否包含指定的子字符串：

```javascript
const str = 'Hello, World!';

console.log(str.includes('World')); // true
console.log(str.includes('world')); // false（区分大小写）
console.log(str.includes('Hello', 6)); // false（从索引 6 开始查找）

// 实际应用
const email = 'user@example.com';
if (email.includes('@')) {
  console.log('有效的邮箱格式');
}

// 检查文件扩展名
const filename = 'document.pdf';
if (filename.includes('.pdf')) {
  console.log('这是一个 PDF 文件');
}
```

### String.prototype.startsWith()

判断字符串是否以指定的子字符串开头：

```javascript
const str = 'Hello, World!';

console.log(str.startsWith('Hello')); // true
console.log(str.startsWith('World')); // false
console.log(str.startsWith('World', 7)); // true（从索引 7 开始）

// 实际应用
const url = 'https://api.example.com/users';
if (url.startsWith('https://')) {
  console.log('安全连接');
}

// 检查协议
const path = '/api/users';
if (path.startsWith('/api')) {
  console.log('API 路径');
}
```

### String.prototype.endsWith()

判断字符串是否以指定的子字符串结尾：

```javascript
const str = 'Hello, World!';

console.log(str.endsWith('World!')); // true
console.log(str.endsWith('Hello')); // false
console.log(str.endsWith('Hello', 5)); // true（检查前 5 个字符）

// 实际应用
const filename = 'document.pdf';
if (filename.endsWith('.pdf')) {
  console.log('PDF 文件');
}

// 检查文件类型
function isImageFile(filename) {
  const imageExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.webp'];
  return imageExtensions.some(ext => filename.endsWith(ext));
}

console.log(isImageFile('photo.jpg')); // true
console.log(isImageFile('document.pdf')); // false
```

## 重复方法

### String.prototype.repeat()

返回指定次数重复的字符串：

```javascript
const str = 'abc';

console.log(str.repeat(0)); // ''
console.log(str.repeat(1)); // 'abc'
console.log(str.repeat(2)); // 'abcabc'
console.log(str.repeat(3)); // 'abcabcabc'

// 实际应用
// 生成分隔线
const separator = '-'.repeat(50);
console.log(separator);
// --------------------------------------------------

// 生成缩进
function indent(level, char = ' ') {
  return char.repeat(level * 2);
}

console.log(indent(1)); // '  '
console.log(indent(2)); // '    '

// 生成占位符
function maskEmail(email) {
  const [local, domain] = email.split('@');
  const maskedLocal = local[0] + '*'.repeat(local.length - 1);
  return `${maskedLocal}@${domain}`;
}

console.log(maskEmail('john@example.com')); // 'j***@example.com'
```

## 填充方法

### String.prototype.padStart()

在字符串开头填充指定字符，直到达到指定长度：

```javascript
const str = '5';

console.log(str.padStart(3, '0')); // '005'
console.log(str.padStart(5, '0')); // '00005'
console.log(str.padStart(3)); // '  5'（默认填充空格）

// 实际应用
// 格式化时间
function formatTime(hours, minutes, seconds) {
  return [
    hours.toString().padStart(2, '0'),
    minutes.toString().padStart(2, '0'),
    seconds.toString().padStart(2, '0')
  ].join(':');
}

console.log(formatTime(9, 5, 3)); // '09:05:03'

// 格式化订单号
function formatOrderNumber(num) {
  return `ORD-${num.toString().padStart(8, '0')}`;
}

console.log(formatOrderNumber(1234)); // 'ORD-00001234'

// 格式化金额
function formatCurrency(amount) {
  return amount.toFixed(2).padStart(10, ' ');
}

console.log(formatCurrency(123.4)); // '    123.40'
```

### String.prototype.padEnd()

在字符串结尾填充指定字符，直到达到指定长度：

```javascript
const str = 'Hello';

console.log(str.padEnd(10, '.')); // 'Hello.....'
console.log(str.padEnd(10)); // 'Hello     '
console.log(str.padEnd(3)); // 'Hello'（已经够长）

// 实际应用
// 格式化表格
function formatTable(rows, columnWidth = 20) {
  return rows.map(row => 
    row.map(cell => 
      cell.toString().padEnd(columnWidth, ' ')
    ).join('')
  ).join('\n');
}

const data = [
  ['Name', 'Age', 'City'],
  ['John', 25, 'New York'],
  ['Jane', 30, 'Los Angeles']
];

console.log(formatTable(data));
// Name                Age                 City                
// John                25                  New York            
// Jane                30                  Los Angeles         

// 对齐输出
function alignText(text, width, align = 'left') {
  switch (align) {
    case 'left':
      return text.padEnd(width);
    case 'right':
      return text.padStart(width);
    case 'center':
      const padding = width - text.length;
      const leftPad = Math.floor(padding / 2);
      const rightPad = padding - leftPad;
      return ' '.repeat(leftPad) + text + ' '.repeat(rightPad);
  }
}

console.log(alignText('Hello', 20, 'center'));
// '       Hello        '
```

## 去除空白方法

### String.prototype.trim()

去除字符串两端的空白字符：

```javascript
const str = '   Hello, World!   ';

console.log(str.trim()); // 'Hello, World!'
console.log(str.trimStart()); // 'Hello, World!   '
console.log(str.trimEnd()); // '   Hello, World!'

// 实际应用
// 清理用户输入
function cleanInput(input) {
  return input.trim().toLowerCase();
}

console.log(cleanInput('  John Doe  ')); // 'john doe'

// 处理表单数据
function processFormData(data) {
  return Object.fromEntries(
    Object.entries(data).map(([key, value]) => [
      key,
      typeof value === 'string' ? value.trim() : value
    ])
  );
}

const formData = { name: '  John  ', email: ' john@example.com ' };
console.log(processFormData(formData));
// { name: 'John', email: 'john@example.com' }
```

## 模板字符串相关

### String.raw()

返回原始字符串（不处理转义字符）：

```javascript
const path = `C:\new\test`;
console.log(path); // C:
// ew	est（\n 和 \t 被转义了）

const rawPath = String.raw`C:\new\test`;
console.log(rawPath); // C:\new\test

// 生成正则表达式
const regex = String.raw`\d+\.\d+`;
console.log(new RegExp(regex)); // /\d+\.\d+/

// 在标签模板中使用
function tag(strings, ...values) {
  console.log(strings.raw[0]); // "Line 1\nLine 2"
}

tag`Line 1\nLine 2`;
```

## 实际应用示例

### 1. 文本处理工具

```javascript
const TextUtils = {
  // 首字母大写
  capitalize(str) {
    return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
  },
  
  // 驼峰命名转下划线
  camelToSnake(str) {
    return str.replace(/[A-Z]/g, letter => `_${letter.toLowerCase()}`);
  },
  
  // 下划线转驼峰命名
  snakeToCamel(str) {
    return str.replace(/_([a-z])/g, (_, letter) => letter.toUpperCase());
  },
  
  // 截断字符串
  truncate(str, length, suffix = '...') {
    if (str.length <= length) return str;
    return str.slice(0, length - suffix.length) + suffix;
  },
  
  // 统计单词数
  wordCount(str) {
    return str.trim().split(/\s+/).filter(word => word.length > 0).length;
  }
};

console.log(TextUtils.capitalize('hello')); // 'Hello'
console.log(TextUtils.camelToSnake('userName')); // 'user_name'
console.log(TextUtils.snakeToCamel('user_name')); // 'userName'
console.log(TextUtils.truncate('Hello, World!', 8)); // 'Hello...'
console.log(TextUtils.wordCount('Hello World foo bar')); // 4
```

### 2. 数据格式化

```javascript
// 格式化数字
function formatNumber(num, options = {}) {
  const { 
    decimals = 0, 
    thousandsSeparator = ',', 
    decimalSeparator = '.' 
  } = options;
  
  const parts = num.toFixed(decimals).split('.');
  parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, thousandsSeparator);
  
  return parts.join(decimalSeparator);
}

console.log(formatNumber(1234567.891)); // '1,234,568'
console.log(formatNumber(1234567.891, { decimals: 2 })); // '1,234,567.89'

// 格式化文件大小
function formatFileSize(bytes) {
  const units = ['B', 'KB', 'MB', 'GB', 'TB'];
  let unitIndex = 0;
  let size = bytes;
  
  while (size >= 1024 && unitIndex < units.length - 1) {
    size /= 1024;
    unitIndex++;
  }
  
  return `${size.toFixed(2)} ${units[unitIndex]}`;
}

console.log(formatFileSize(1024)); // '1.00 KB'
console.log(formatFileSize(1234567)); // '1.18 MB'
```

### 3. 模板引擎

```javascript
function renderTemplate(template, data) {
  return template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
    return data.hasOwnProperty(key) ? data[key] : match;
  });
}

const template = 'Hello, {{name}}! You have {{count}} new messages.';
const data = { name: 'John', count: 5 };

console.log(renderTemplate(template, data));
// "Hello, John! You have 5 new messages."

// 支持嵌套属性
function renderAdvancedTemplate(template, data) {
  return template.replace(/\{\{([^}]+)\}\}/g, (match, path) => {
    const keys = path.split('.');
    let value = data;
    
    for (const key of keys) {
      if (value && typeof value === 'object' && key in value) {
        value = value[key];
      } else {
        return match;
      }
    }
    
    return value;
  });
}

const advancedTemplate = 'User: {{user.name}}, Email: {{user.email}}';
const advancedData = { user: { name: 'John', email: 'john@example.com' } };

console.log(renderAdvancedTemplate(advancedTemplate, advancedData));
// "User: John, Email: john@example.com"
```

### 4. 字符串验证

```javascript
const Validators = {
  // 检查是否为空或只包含空白
  isEmpty(str) {
    return !str || str.trim().length === 0;
  },
  
  // 检查邮箱格式
  isEmail(str) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(str);
  },
  
  // 检查是否只包含字母
  isAlpha(str) {
    return /^[a-zA-Z]+$/.test(str);
  },
  
  // 检查是否只包含数字
  isNumeric(str) {
    return /^\d+$/.test(str);
  },
  
  // 检查是否是有效 URL
  isURL(str) {
    try {
      new URL(str);
      return true;
    } catch {
      return false;
    }
  }
};

console.log(Validators.isEmpty('   ')); // true
console.log(Validators.isEmail('john@example.com')); // true
console.log(Validators.isAlpha('Hello')); // true
console.log(Validators.isNumeric('12345')); // true
console.log(Validators.isURL('https://example.com')); // true
```

## 最佳实践

1. **使用 `includes()` 代替 `indexOf()`**：更语义化
2. **使用 `startsWith()` 和 `endsWith()`**：检查字符串开头和结尾
3. **使用 `padStart()` 和 `padEnd()`**：格式化输出
4. **使用 `trim()`**：清理用户输入
5. **使用 `repeat()`**：生成重复字符串

```javascript
// 好的做法
if (email.includes('@')) {
  // 处理邮箱
}

if (filename.endsWith('.pdf')) {
  // 处理 PDF 文件
}

const formatted = number.toString().padStart(8, '0');

// 避免
if (email.indexOf('@') !== -1) {
  // 处理邮箱
}

if (filename.slice(-4) === '.pdf') {
  // 处理 PDF 文件
}
```

## 参考

- [MDN String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)
