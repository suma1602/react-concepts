# React Hooks - Comprehensive Guide

Hooks are functions that let you "hook into" React state and lifecycle features from function components. They were introduced in React 16.8 and have become the modern way to write React applications.

## Table of Contents
1. [useState](#usestate)
2. [useEffect](#useeffect)
3. [useContext](#usecontext)
4. [useReducer](#usereducer)
5. [useMemo](#usememo)
6. [useCallback](#usecallback)
7. [useRef](#useref)
8. [Custom Hooks](#custom-hooks)

---

## useState

`useState` is the most fundamental hook that lets you add state to functional components.

### Syntax
```javascript
const [state, setState] = useState(initialValue);
```

### Key Concepts
- Returns an array with two elements: current state and a function to update it
- Initial state is only used on the first render
- Updates trigger a re-render of the component

### Basic Example
```javascript
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

### Advanced Example - Multiple State Variables
```javascript
function UserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);

  const handleReset = () => {
    setName('');
    setEmail('');
    setAge(0);
  };

  return (
    <form>
      <input 
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input 
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input 
        type="number"
        value={age}
        onChange={(e) => setAge(Number(e.target.value))}
        placeholder="Age"
      />
      <button type="button" onClick={handleReset}>Reset</button>
    </form>
  );
}
```

### Best Practices
- Use multiple `useState` calls instead of one state object
- Functional updates for dependent state: `setState(prev => prev + 1)`
- Keep state as granular as possible

---

## useEffect

`useEffect` lets you perform side effects in functional components. It's the replacement for lifecycle methods like `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.

### Syntax
```javascript
useEffect(() => {
  // Side effect code here
  
  return () => {
    // Cleanup code (optional)
  };
}, [dependencies]);
```

### Dependency Array Behaviors
- **No dependency array**: Runs after every render
- **Empty array `[]`**: Runs only once after initial render (like `componentDidMount`)
- **With dependencies**: Runs when any dependency changes

### Basic Example
```javascript
import React, { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // Cleanup function
    return () => clearInterval(interval);
  }, []); // Runs only once

  return <div>Seconds: {seconds}</div>;
}
```

### Advanced Example - Data Fetching
```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let isMounted = true;

    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch');
        const data = await response.json();
        
        if (isMounted) {
          setUser(data);
          setError(null);
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message);
          setUser(null);
        }
      } finally {
        if (isMounted) setLoading(false);
      }
    };

    fetchUser();

    return () => {
      isMounted = false; // Prevent state updates on unmounted component
    };
  }, [userId]); // Re-run when userId changes

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{user?.name}</div>;
}
```

### Best Practices
- Always include a dependency array
- Use cleanup functions for subscriptions, timers, etc.
- Avoid setting state in useEffect if it depends on props/state from the same effect
- Consider using `useLayoutEffect` for DOM measurements

---

## useContext

`useContext` lets you subscribe to context without wrapping your component in a Consumer component. It simplifies consuming context values.

### Syntax
```javascript
const value = useContext(MyContext);
```

### Basic Setup
```javascript
// Create a context
import React, { createContext, useState } from 'react';

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

export default ThemeContext;
```

### Using Context
```javascript
import React, { useContext } from 'react';
import ThemeContext from './ThemeContext';

function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <button 
      style={{ 
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#000' : '#fff'
      }}
      onClick={toggleTheme}
    >
      Toggle Theme (Current: {theme})
    </button>
  );
}
```

### Advanced Example - Multiple Contexts
```javascript
const UserContext = createContext();
const NotificationContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  const [notifications, setNotifications] = useState([]);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <NotificationContext.Provider value={{ notifications, setNotifications }}>
        <MainContent />
      </NotificationContext.Provider>
    </UserContext.Provider>
  );
}

function MyComponent() {
  const { user } = useContext(UserContext);
  const { notifications } = useContext(NotificationContext);

  return (
    <div>
      <p>User: {user?.name}</p>
      <p>Notifications: {notifications.length}</p>
    </div>
  );
}
```

### Best Practices
- Create separate contexts for different concerns
- Use context for global state (auth, theme, notifications)
- Avoid performance issues by splitting contexts
- Memoize context value to prevent unnecessary re-renders

---

## useReducer

`useReducer` is a more advanced alternative to `useState` for managing complex state logic. It's useful when you have multiple related state variables or complex state transitions.

### Syntax
```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

### Basic Example - Counter
```javascript
import React, { useReducer } from 'react';

// Reducer function
function countReducer(state, action) {
  switch(action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    case 'ADD':
      return { count: state.count + action.payload };
    default:
      return state;
  }
}

function AdvancedCounter() {
  const [state, dispatch] = useReducer(countReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+1</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-1</button>
      <button onClick={() => dispatch({ type: 'ADD', payload: 5 })}>+5</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
}
```

### Complex Example - Todo App
```javascript
const initialState = {
  todos: [],
  filter: 'all'
};

function todoReducer(state, action) {
  switch(action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload,
            completed: false
          }
        ]
      };
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload
      };
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [input, setInput] = useState('');

  const handleAddTodo = () => {
    if (input.trim()) {
      dispatch({ type: 'ADD_TODO', payload: input });
      setInput('');
    }
  };

  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && handleAddTodo()}
        placeholder="Add a todo"
      />
      <button onClick={handleAddTodo}>Add</button>
      
      <div>
        {state.todos.map(todo => (
          <div key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}>
              Delete
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Best Practices
- Use when state logic is complex or involves multiple sub-values
- Keep reducer functions pure (no side effects)
- Initialize state lazily for expensive computations
- Use for coordinated state updates

---

## useMemo

`useMemo` memoizes expensive computations and returns the cached result, preventing unnecessary recalculations on every render.

### Syntax
```javascript
const memoizedValue = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a, b]);
```

### Basic Example
```javascript
import React, { useMemo, useState } from 'react';

function ExpensiveComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Expensive calculation - only runs when count changes
  const fibonacci = useMemo(() => {
    console.log('Computing fibonacci...');
    const fib = (n) => n <= 1 ? n : fib(n - 1) + fib(n - 2);
    return fib(35); // Heavy computation
  }, [count]);

  return (
    <div>
      <p>Fibonacci(35): {fibonacci}</p>
      <button onClick={() => setCount(count + 1)}>Recompute</button>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Type name (won't trigger calculation)"
      />
    </div>
  );
}
```

### Advanced Example - Filtering Large Lists
```javascript
function UserList({ users, searchTerm }) {
  const [sortBy, setSortBy] = useState('name');

  // Memoized filtered and sorted users
  const filteredUsers = useMemo(() => {
    console.log('Filtering users...');
    let result = users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase())
    );

    result.sort((a, b) => {
      if (sortBy === 'name') {
        return a.name.localeCompare(b.name);
      } else if (sortBy === 'age') {
        return a.age - b.age;
      }
      return 0;
    });

    return result;
  }, [users, searchTerm, sortBy]);

  return (
    <div>
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="name">Sort by Name</option>
        <option value="age">Sort by Age</option>
      </select>
      {filteredUsers.map(user => (
        <div key={user.id}>{user.name} - {user.age}</div>
      ))}
    </div>
  );
}
```

### Best Practices
- Only use when computation is expensive
- Don't overuse - can hurt performance if dependencies change frequently
- Include all dependencies in the dependency array
- Measure performance before and after using `useMemo`

---

## useCallback

`useCallback` memoizes a function definition. It's useful when passing callbacks to optimized child components that rely on reference equality.

### Syntax
```javascript
const memoizedCallback = useCallback(() => {
  // Callback code
}, [dependencies]);
```

### Basic Example
```javascript
import React, { useCallback, useState, memo } from 'react';

// Child component that only re-renders if props change (reference equality)
const Button = memo(({ onClick, label }) => {
  console.log(`Rendering button: ${label}`);
  return <button onClick={onClick}>{label}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState('');

  // Without useCallback, a new function is created every render
  // With useCallback, the same function reference is maintained
  const handleIncrement = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // Dependencies array

  const handleOtherChange = useCallback(() => {
    setOther('changed');
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <input
        value={other}
        onChange={(e) => setOther(e.target.value)}
        placeholder="Type something"
      />
      <Button onClick={handleIncrement} label="Increment" />
      <Button onClick={handleOtherChange} label="Change Other" />
    </div>
  );
}
```

### Advanced Example - Form with Dependencies
```javascript
function SearchUsers({ onSearch }) {
  const [query, setQuery] = useState('');

  // Callback updates when onSearch dependency changes
  const handleSearch = useCallback(() => {
    onSearch(query);
  }, [query, onSearch]); // Both dependencies needed

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search"
      />
      <button onClick={handleSearch}>Search</button>
    </div>
  );
}
```

### Best Practices
- Use with `memo()` for child components
- Don't overuse - most callbacks don't need memoization
- Include all dependencies that the callback uses
- Combine with `useMemo` for object/array dependencies

---

## useRef

`useRef` creates a mutable reference that persists across renders. It's commonly used to access DOM elements directly or store mutable values that don't cause re-renders.

### Syntax
```javascript
const ref = useRef(initialValue);
```

### DOM Access Example
```javascript
import React, { useRef } from 'react';

function TextInput() {
  const inputRef = useRef(null);

  const handleFocus = () => {
    inputRef.current.focus();
  };

  const handleClear = () => {
    inputRef.current.value = '';
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Click button to focus" />
      <button onClick={handleFocus}>Focus Input</button>
      <button onClick={handleClear}>Clear Input</button>
    </div>
  );
}
```

### Storing Mutable Values
```javascript
import React, { useRef, useEffect, useState } from 'react';

function Timer() {
  const [isRunning, setIsRunning] = useState(false);
  const intervalRef = useRef(null);
  const [time, setTime] = useState(0);

  const handleStart = () => {
    if (!isRunning) {
      setIsRunning(true);
      intervalRef.current = setInterval(() => {
        setTime(prev => prev + 1);
      }, 1000);
    }
  };

  const handleStop = () => {
    setIsRunning(false);
    clearInterval(intervalRef.current);
  };

  const handleReset = () => {
    clearInterval(intervalRef.current);
    setTime(0);
    setIsRunning(false);
  };

  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);

  return (
    <div>
      <p>Time: {time}s</p>
      <button onClick={handleStart}>Start</button>
      <button onClick={handleStop}>Stop</button>
      <button onClick={handleReset}>Reset</button>
    </div>
  );
}
```

### Advanced Example - Previous Value Tracking
```javascript
function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();

  useEffect(() => {
    prevCountRef.current = count;
  }, [count]);

  return (
    <div>
      <p>Now: {count}</p>
      <p>Before: {prevCountRef.current}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### Best Practices
- Use for DOM manipulation, not state management
- Access current value with `ref.current`
- Changing ref doesn't trigger re-renders
- Don't return refs from render method for reading
- Use `forwardRef` to pass refs to child components

---

## Custom Hooks

Custom hooks are JavaScript functions that use React hooks. They let you extract component logic into reusable functions.

### Rules of Custom Hooks
- Must start with "use" prefix
- Must call other hooks unconditionally
- Can call other hooks and custom hooks

### Basic Example - useAsync
```javascript
import { useState, useEffect } from 'react';

function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState('idle');
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(async () => {
    setStatus('pending');
    setData(null);
    setError(null);
    try {
      const response = await asyncFunction();
      setData(response);
      setStatus('success');
      return response;
    } catch (err) {
      setError(err);
      setStatus('error');
    }
  }, [asyncFunction]);

  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { execute, status, data, error };
}

// Usage
function UserProfile({ userId }) {
  const { status, data: user, error } = useAsync(
    () => fetch(`/api/users/${userId}`).then(r => r.json()),
    true
  );

  if (status === 'pending') return <div>Loading...</div>;
  if (status === 'error') return <div>Error: {error?.message}</div>;
  return <div>{user?.name}</div>;
}
```

### useLocalStorage Custom Hook
```javascript
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}

// Usage
function UserPreferences() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'en');

  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      <select value={language} onChange={(e) => setLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Spanish</option>
      </select>
    </div>
  );
}
```

### useFetch Custom Hook
```javascript
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let isMounted = true;

    const fetchData = async () => {
      try {
        const response = await fetch(url, options);
        if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
        const result = await response.json();
        if (isMounted) {
          setData(result);
          setError(null);
        }
      } catch (err) {
        if (isMounted) {
          setError(err);
          setData(null);
        }
      } finally {
        if (isMounted) setLoading(false);
      }
    };

    fetchData();

    return () => {
      isMounted = false;
    };
  }, [url, options]);

  return { data, loading, error };
}

// Usage
function Posts() {
  const { data: posts, loading, error } = useFetch('/api/posts');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return (
    <ul>
      {posts?.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

### useDebounce Custom Hook
```javascript
function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchUsers() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 300);

  useEffect(() => {
    if (debouncedSearchTerm) {
      console.log('Searching for:', debouncedSearchTerm);
      // Perform search
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search users..."
    />
  );
}
```

### Best Practices for Custom Hooks
- Keep hooks focused and single-purpose
- Document hook usage and parameters
- Return values in a consistent format
- Test custom hooks thoroughly
- Publish reusable hooks as npm packages
- Use TypeScript for better type safety

---

## Summary

| Hook | Purpose | Use Case |
|------|---------|----------|
| `useState` | Manage state in functional components | Form inputs, toggles, counters |
| `useEffect` | Handle side effects | Data fetching, subscriptions, DOM updates |
| `useContext` | Access context values | Global state, theming, authentication |
| `useReducer` | Manage complex state logic | Complex state transitions, todo apps |
| `useMemo` | Memoize expensive calculations | Performance optimization for computed values |
| `useCallback` | Memoize function definitions | Passing callbacks to optimized children |
| `useRef` | Persist mutable values | DOM access, storing intervals/timeouts |
| **Custom Hooks** | Extract reusable logic | Sharing logic between components |

## Additional Resources

- [React Hooks Documentation](https://react.dev/reference/react/hooks)
- [Rules of Hooks](https://react.dev/warnings/invalid-hook-call-warning)
- [Hooks API Reference](https://react.dev/reference/react)
- [Custom Hooks Patterns](https://usehooks.com/)

---

*Last Updated: December 17, 2025*