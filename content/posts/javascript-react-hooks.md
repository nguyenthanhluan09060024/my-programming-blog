---
title: "React Hooks - Cách Sử Dụng Hiệu Quả"
date: 2024-01-14T11:00:00+07:00
draft: false
tags: ['javascript', 'react', 'hooks', 'frontend']
description: "Hướng dẫn chi tiết về React Hooks và cách sử dụng chúng hiệu quả trong ứng dụng React"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# React Hooks - Cách Sử Dụng Hiệu Quả

React Hooks đã thay đổi cách chúng ta viết React components, giúp code trở nên ngắn gọn và dễ hiểu hơn. Trong bài viết này, tôi sẽ hướng dẫn bạn cách sử dụng các hooks phổ biến một cách hiệu quả.

## useState - Quản Lý State

`useState` là hook cơ bản nhất để quản lý state trong functional components.

### Cú pháp cơ bản:
```javascript
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  );
}
```

### State với object:
```javascript
function UserProfile() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  const updateUser = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };
  
  return (
    <div>
      <input 
        value={user.name}
        onChange={(e) => updateUser('name', e.target.value)}
        placeholder="Name"
      />
      <input 
        value={user.email}
        onChange={(e) => updateUser('email', e.target.value)}
        placeholder="Email"
      />
    </div>
  );
}
```

### Lazy initial state:
```javascript
function ExpensiveComponent() {
  // Chỉ tính toán một lần khi component mount
  const [data, setData] = useState(() => {
    return expensiveCalculation();
  });
  
  return <div>{data}</div>;
}
```

## useEffect - Side Effects

`useEffect` thay thế cho lifecycle methods trong class components.

### Cú pháp cơ bản:
```javascript
import React, { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Chạy sau mỗi lần render
    fetchUsers();
  });
  
  useEffect(() => {
    // Chạy chỉ một lần khi component mount
    fetchUsers();
  }, []);
  
  useEffect(() => {
    // Chạy khi userId thay đổi
    fetchUser(userId);
  }, [userId]);
  
  const fetchUsers = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (error) {
      console.error('Error fetching users:', error);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <div>Loading...</div>;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Cleanup function:
```javascript
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // Cleanup function
    return () => {
      clearInterval(interval);
    };
  }, []);
  
  return <div>Timer: {seconds}s</div>;
}
```

## useContext - Context API

`useContext` giúp chia sẻ data giữa các components mà không cần prop drilling.

### Tạo Context:
```javascript
import React, { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

### Sử dụng Context:
```javascript
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

function Header() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <header style={{ background: theme === 'light' ? '#fff' : '#333' }}>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}
```

## useReducer - State Management Phức Tạp

`useReducer` phù hợp cho state phức tạp với nhiều actions.

### Cú pháp cơ bản:
```javascript
import React, { useReducer } from 'react';

const initialState = {
  count: 0,
  loading: false,
  error: null
};

function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'RESET':
      return { ...state, count: 0 };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);
  
  const increment = () => dispatch({ type: 'INCREMENT' });
  const decrement = () => dispatch({ type: 'DECREMENT' });
  const reset = () => dispatch({ type: 'RESET' });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

## useMemo - Tối Ưu Performance

`useMemo` giúp tối ưu performance bằng cách memoize kết quả tính toán.

```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveList({ items, filter }) {
  const [searchTerm, setSearchTerm] = useState('');
  
  // Chỉ tính toán lại khi items, filter hoặc searchTerm thay đổi
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items
      .filter(item => item.category === filter)
      .filter(item => item.name.toLowerCase().includes(searchTerm.toLowerCase()));
  }, [items, filter, searchTerm]);
  
  return (
    <div>
      <input 
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## useCallback - Tối Ưu Functions

`useCallback` giúp memoize functions để tránh re-render không cần thiết.

```javascript
import React, { useState, useCallback, memo } from 'react';

const ChildComponent = memo(({ onClick, name }) => {
  console.log(`${name} rendered`);
  return <button onClick={onClick}>{name}</button>;
});

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('Child');
  
  // Function được memoize, chỉ tạo lại khi count thay đổi
  const handleClick = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <input 
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <ChildComponent onClick={handleClick} name={name} />
    </div>
  );
}
```

## useRef - Truy Cập DOM và Persist Values

`useRef` giúp truy cập DOM elements và lưu trữ values không gây re-render.

### Truy cập DOM:
```javascript
import React, { useRef, useEffect } from 'react';

function FocusInput() {
  const inputRef = useRef(null);
  
  useEffect(() => {
    // Focus vào input khi component mount
    inputRef.current.focus();
  }, []);
  
  const handleClick = () => {
    inputRef.current.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus Input</button>
    </div>
  );
}
```

### Persist values:
```javascript
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);
  
  const startTimer = () => {
    if (intervalRef.current) return;
    
    intervalRef.current = setInterval(() => {
      setCount(prev => prev + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}
```

## Custom Hooks

Custom hooks giúp tái sử dụng logic giữa các components.

### useCounter Hook:
```javascript
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(prev => prev - 1);
  }, []);
  
  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);
  
  return { count, increment, decrement, reset };
}

// Sử dụng
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

### useFetch Hook:
```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const result = await response.json();
        setData(result);
        setError(null);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return { data, loading, error };
}

// Sử dụng
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## Rules of Hooks

1. **Chỉ gọi hooks ở top level** - Không gọi trong loops, conditions, hoặc nested functions
2. **Chỉ gọi hooks từ React functions** - Chỉ trong function components hoặc custom hooks

```javascript
// ✅ Đúng
function MyComponent() {
  const [count, setCount] = useState(0);
  
  if (count > 0) {
    // ❌ Sai - gọi hook trong condition
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

## Kết Luận

React Hooks đã làm cho việc viết React components trở nên đơn giản và mạnh mẽ hơn. Với những kiến thức này, bạn có thể:

- Quản lý state hiệu quả với `useState` và `useReducer`
- Xử lý side effects với `useEffect`
- Chia sẻ data với `useContext`
- Tối ưu performance với `useMemo` và `useCallback`
- Tạo custom hooks để tái sử dụng logic

Hãy thực hành và tạo ra những ứng dụng React tuyệt vời! 🚀


