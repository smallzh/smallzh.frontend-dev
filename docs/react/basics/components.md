# React 组件与 Props

## 0x01 组件基础

组件是 React 应用的可复用构建块，将 UI 拆分为独立、可组合的部分。每个组件维护自己的状态和逻辑。

### 函数组件

函数组件是现代 React 开发的主流方式，配合 Hooks 使用：

```tsx
// 基本函数组件
function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}

// 使用组件
function App() {
  return <Greeting name="React" />;
}
```

### 类组件

类组件是早期的组件写法，现在较少使用：

```tsx
import React, { Component } from 'react';

interface GreetingProps {
  name: string;
}

class Greeting extends Component<GreetingProps> {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

### 组件类型声明

```tsx
// 函数组件类型
type ComponentProps<T> = {
  data: T;
  onChange: (value: T) => void;
};

function MyComponent<T>({ data, onChange }: ComponentProps<T>) {
  return <div onClick={() => onChange(data)}>{data}</div>;
}

// 泛型组件
function List<T>({ items }: { items: T[] }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{String(item)}</li>
      ))}
    </ul>
  );
}
```

## 0x02 Props 基础

Props 是传递给组件的数据，作为组件的只读参数。

### 基本 Props

```tsx
// 定义 Props 接口
interface ButtonProps {
  text: string;
  onClick: () => void;
  disabled?: boolean;  // 可选属性
}

// 使用 Props
function Button({ text, onClick, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {text}
    </button>
  );
}

// 调用组件
function App() {
  return (
    <Button 
      text="Click Me" 
      onClick={() => console.log('Clicked!')} 
    />
  );
}
```

### 默认 Props

```tsx
// 方式一：默认参数
function Avatar({ 
  src, 
  size = 48, 
  alt = 'avatar' 
}: { 
  src: string; 
  size?: number; 
  alt?: string;
}) {
  return (
    <img 
      src={src} 
      width={size} 
      height={size} 
      alt={alt} 
      style={{ borderRadius: '50%' }}
    />
  );
}

// 方式二：defaultProps（类组件）
interface AlertProps {
  message: string;
  type?: 'info' | 'warning' | 'error';
}

function Alert({ message, type = 'info' }: AlertProps) {
  return <div className={`alert alert-${type}`}>{message}</div>;
}

Alert.defaultProps = {
  type: 'info'
};
```

### children Props

```tsx
// 接收 children
function Card({ 
  title, 
  children 
}: { 
  title: string; 
  children: React.ReactNode;
}) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// 使用
function App() {
  return (
    <Card title="Welcome">
      <p>This is the card content.</p>
      <button>Learn More</button>
    </Card>
  );
}
```

### 函数作为 Props

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item, index)}</li>
      ))}
    </ul>
  );
}

// 使用
function App() {
  const users = ['Alice', 'Bob', 'Charlie'];
  
  return (
    <List 
      items={users}
      renderItem={(name, index) => (
        <span>{index + 1}. {name}</span>
      )}
    />
  );
}
```

### Props 展开

```tsx
interface ButtonProps {
  variant: 'primary' | 'secondary';
  size: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
}

function Button({ variant, size, children, ...rest }: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`}
      {...rest}
    >
      {children}
    </button>
  );
}
```

## 0x03 组件组合

### 组合模式

```tsx
// Layout 组件
function Layout({ 
  header, 
  main, 
  footer 
}: { 
  header: React.ReactNode; 
  main: React.ReactNode; 
  footer: React.ReactNode;
}) {
  return (
    <div className="layout">
      <header>{header}</header>
      <main>{main}</main>
      <footer>{footer}</footer>
    </div>
  );
}

// 使用
function App() {
  return (
    <Layout
      header={<h1>My App</h1>}
      main={<p>Main content goes here</p>}
      footer={<p>&copy; 2024</p>}
    />
  );
}
```

### 渲染属性模式

```tsx
// 渲染属性组件
function MouseTracker({ 
  render 
}: { 
  render: (position: { x: number; y: number }) => React.ReactNode;
}) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return render(position);
}

// 使用
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <p>Mouse position: {x}, {y}</p>
      )}
    />
  );
}
```

### 高阶组件（HOC）

高阶组件是一个函数，接受组件并返回新组件：

```tsx
// 高阶组件：添加日志功能
function withLogger<T extends { name: string }>(
  WrappedComponent: React.ComponentType<T>
) {
  return function (props: T) {
    useEffect(() => {
      console.log(`Component ${props.name} mounted`);
      return () => console.log(`Component ${props.name} unmounted`);
    }, [props.name]);
    
    return <WrappedComponent {...props} />;
  };
}

// 使用
interface UserCardProps {
  name: string;
  email: string;
}

function UserCard({ name, email }: UserCardProps) {
  return (
    <div className="user-card">
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}

const UserCardWithLogger = withLogger(UserCard);

function App() {
  return (
    <UserCardWithLogger 
      name="User Card" 
      email="user@example.com" 
    />
  );
}
```

### 受控组件与非受控组件

```tsx
// 受控组件：状态由 React 控制
function ControlledInput() {
  const [value, setValue] = useState('');
  
  return (
    <input 
      value={value}
      onChange={e => setValue(e.target.value)}
    />
  );
}

// 非受控组件：状态由 DOM 控制
function UncontrolledInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const handleSubmit = () => {
    console.log('Value:', inputRef.current?.value);
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}

// 受控select
function ControlledSelect() {
  const [value, setValue] = useState('apple');
  
  return (
    <select value={value} onChange={e => setValue(e.target.value)}>
      <option value="apple">Apple</option>
      <option value="banana">Banana</option>
      <option value="orange">Orange</option>
    </select>
  );
}
```

## 0x04 组件通信

### 父子组件通信

```tsx
// 父组件
function Parent() {
  const [message, setMessage] = useState('');
  
  return (
    <div>
      <Child onSend={setMessage} />
      <p>Received: {message}</p>
    </div>
  );
}

// 子组件
function Child({ onSend }: { onSend: (msg: string) => void }) {
  const [input, setInput] = useState('');
  
  return (
    <div>
      <input 
        value={input}
        onChange={e => setInput(e.target.value)}
      />
      <button onClick={() => onSend(input)}>
        Send to Parent
      </button>
    </div>
  );
}
```

### 兄弟组件通信

```tsx
// 使用父组件作为中介
function Siblings() {
  const [sharedState, setSharedState] = useState(0);
  
  return (
    <div>
      <SiblingA count={sharedState} />
      <SiblingB onIncrement={() => setSharedState(s => s + 1)} />
    </div>
  );
}

function SiblingA({ count }: { count: number }) {
  return <p>Count: {count}</p>;
}

function SiblingB({ onIncrement }: { onIncrement: () => void }) {
  return <button onClick={onIncrement}>Increment</button>;
}
```

### Context 跨层级通信

```tsx
// 创建 Context
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | null>(null);

// Provider
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(t => t === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 使用 Context 的组件
function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button 
      onClick={toggleTheme}
      style={{
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#333'
      }}
    >
      Current: {theme}
    </button>
  );
}
```

### 状态提升

```tsx
// 将状态提升到公共父组件
function Calculator() {
  const [num1, setNum1] = useState(0);
  const [num2, setNum2] = useState(0);
  
  const result = num1 + num2;
  
  return (
    <div>
      <InputValue value={num1} onChange={setNum1} label="Number 1" />
      <InputValue value={num2} onChange={setNum2} label="Number 2" />
      <p>Result: {result}</p>
    </div>
  );
}

function InputValue({ 
  value, 
  onChange, 
  label 
}: { 
  value: number; 
  onChange: (v: number) => void;
  label: string;
}) {
  return (
    <div>
      <label>{label}: </label>
      <input
        type="number"
        value={value}
        onChange={e => onChange(Number(e.target.value))}
      />
    </div>
  );
}
```

## 0x05 组件性能优化

### React.memo

使用 `React.memo` 缓存组件，避免不必要的渲染：

```tsx
import { memo } from 'react';

// memoized 组件
const ListItem = memo(function ListItem({ 
  item, 
  onClick 
}: { 
  item: { id: number; name: string };
  onClick: (id: number) => void;
}) {
  console.log('ListItem rendered:', item.id);
  
  return (
    <li onClick={() => onClick(item.id)}>
      {item.name}
    </li>
  );
});

// 使用
function List({ items, onItemClick }: { 
  items: Array<{ id: number; name: string }>;
  onItemClick: (id: number) => void;
}) {
  return (
    <ul>
      {items.map(item => (
        <ListItem 
          key={item.id} 
          item={item} 
          onClick={onItemClick} 
        />
      ))}
    </ul>
  );
}
```

### useMemo 优化 props

```tsx
interface User {
  id: number;
  name: string;
  email: string;
}

function UserList({ 
  users, 
  filter 
}: { 
  users: User[]; 
  filter: string;
}) {
  // 缓存过滤后的用户列表
  const filteredUsers = useMemo(() => {
    return users.filter(user => 
      user.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [users, filter]);
  
  return (
    <ul>
      {filteredUsers.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### useCallback 优化回调

```tsx
function Parent() {
  const [items] = useState(['a', 'b', 'c']);
  const [count, setCount] = useState(0);
  
  // 缓存回调函数，避免子组件重渲染
  const handleClick = useCallback((item: string) => {
    console.log('Clicked:', item);
  }, []);
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Re-render: {count}
      </button>
      <ChildList items={items} onItemClick={handleClick} />
    </div>
  );
}
```

## 0x06 组件最佳实践

### 单一职责

```tsx
// ❌ 错误：一个组件做太多事情
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  
  // 获取用户、帖子...
  // 渲染用户信息、帖子列表...
  
  return <div>...</div>;
}

// ✅ 正确：拆分组件
function UserProfile({ userId }: { userId: string }) {
  return (
    <div>
      <UserInfo userId={userId} />
      <UserPosts userId={userId} />
    </div>
  );
}

function UserInfo({ userId }: { userId: string }) {
  // 只负责用户信息
  const [user, setUser] = useState(null);
  // ...
  return <div>...</div>;
}

function UserPosts({ userId }: { userId: string }) {
  // 只负责帖子列表
  const [posts, setPosts] = useState([]);
  // ...
  return <div>...</div>;
}
```

### Props 校验

```tsx
import PropTypes from 'prop-types';

function Button({ text, variant, disabled }: ButtonProps) {
  return (
    <button className={`btn-${variant}`} disabled={disabled}>
      {text}
    </button>
  );
}

Button.propTypes = {
  text: PropTypes.string.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary', 'danger']),
  disabled: PropTypes.bool
};

Button.defaultProps = {
  variant: 'primary',
  disabled: false
};
```

### 类型化 Hooks

```tsx
// 自定义 Hook 类型
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  return [storedValue, setStoredValue] as const;
}

// 使用
function App() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  
  return <div>{theme}</div>;
}
```

## 参考

- [React 组件文档](https://react.dev/learn/your-component)
- [Props 文档](https://react.dev/learn/passing-props-to-a-component)
- [组件组合](https://react.dev/learn/composition)
