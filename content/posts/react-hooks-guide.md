---
title: "Hướng Dẫn React Hooks Cho Người Mới Bắt Đầu"
date: 2024-01-05T09:15:00+07:00
draft: false
tags: ['react', 'hooks', 'javascript', 'frontend']
description: "Hướng dẫn chi tiết về React Hooks cho những người mới học React"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# Hướng Dẫn React Hooks Cho Người Mới Bắt Đầu

React Hooks là một tính năng mạnh mẽ được giới thiệu từ React 16.8, cho phép bạn sử dụng state và các tính năng khác của React mà không cần viết class component.

## Tại sao cần Hooks?

Trước khi có Hooks, chúng ta phải sử dụng class component để quản lý state:

```javascript
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>+</button>
      </div>
    );
  }
}
```

Với Hooks, code trở nên đơn giản hơn:

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
    </div>
  );
}
```

## useState Hook

`useState` là Hook cơ bản nhất để quản lý state:

```javascript
import { useState } from 'react';

function Example() {
  // Khai báo state
  const [name, setName] = useState('');
  const [age, setAge] = useState(0);
  const [items, setItems] = useState([]);

  return (
    <div>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
        placeholder="Tên của bạn"
      />
      <p>Xin chào {name}!</p>
    </div>
  );
}
```

## useEffect Hook

`useEffect` thay thế cho `componentDidMount`, `componentDidUpdate`, và `componentWillUnmount`:

```javascript
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Chạy khi component mount hoặc userId thay đổi
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        console.error('Error fetching user:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]); // Dependency array

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## useContext Hook

`useContext` giúp chia sẻ data giữa các component mà không cần prop drilling:

```javascript
import { createContext, useContext } from 'react';

// Tạo context
const ThemeContext = createContext();

// Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Consumer component
function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button 
      style={{ 
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff'
      }}
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
    >
      Toggle Theme
    </button>
  );
}
```

## useReducer Hook

`useReducer` phù hợp cho state phức tạp:

```javascript
import { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

## Custom Hooks

Bạn có thể tạo custom hooks để tái sử dụng logic:

```javascript
// Custom hook
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}

// Sử dụng custom hook
function Counter() {
  const { count, increment, decrement, reset } = useCounter(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

## Rules of Hooks

1. **Chỉ gọi Hooks ở top level** - Không gọi trong loops, conditions, hoặc nested functions
2. **Chỉ gọi Hooks từ React functions** - Chỉ trong function components hoặc custom hooks

```javascript
// ✅ Đúng
function MyComponent() {
  const [count, setCount] = useState(0);
  
  if (count > 0) {
    // ❌ Sai - gọi Hook trong condition
    // const [name, setName] = useState('');
  }
  
  return <div>{count}</div>;
}

// ✅ Đúng
function MyComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState(''); // Gọi ở top level
  
  if (count > 0) {
    // Logic khác
  }
  
  return <div>{count}</div>;
}
```

## Kết luận

React Hooks giúp code React trở nên đơn giản và dễ hiểu hơn. Bắt đầu với `useState` và `useEffect`, sau đó khám phá các Hooks khác khi cần thiết.

Hãy thực hành và tạo ra những ứng dụng React tuyệt vời! 🚀
