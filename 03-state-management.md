# State Management in React

## Overview

State management is one of the most critical aspects of building React applications. This guide covers various approaches to managing state, from basic React hooks to advanced patterns and third-party solutions.

---

## Table of Contents

1. [Local Component State](#local-component-state)
2. [Lifting State Up](#lifting-state-up)
3. [Context API](#context-api)
4. [useReducer Hook](#usereducer-hook)
5. [Custom Hooks](#custom-hooks)
6. [Third-Party Libraries](#third-party-libraries)
7. [Best Practices](#best-practices)
8. [Common Patterns](#common-patterns)

---

## Local Component State

### Using useState Hook

The simplest way to manage state in React is with the `useState` hook.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
    </div>
  );
}

export default Counter;
```

**Key Points:**
- `useState` returns an array: `[state, setState]`
- Use state for data that changes and affects the component's output
- Each component instance has its own separate state

### Multiple State Variables

```jsx
function UserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);

  return (
    <form>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="number" value={age} onChange={(e) => setAge(e.target.value)} />
    </form>
  );
}
```

---

## Lifting State Up

When multiple components need to share state, move the state to their common parent (ancestor) component.

### Problem: Sibling Communication

```jsx
// ❌ Problem: Siblings can't share state directly
function Parent() {
  return (
    <div>
      <Child1 />
      <Child2 />
    </div>
  );
}

function Child1() {
  const [data, setData] = useState('');
  return <div>{data}</div>;
}

function Child2() {
  return <button onClick={() => { /* can't update Child1 */ }}>Share</button>;
}
```

### Solution: Lift State to Parent

```jsx
// ✅ Solution: Manage state in parent
function Parent() {
  const [data, setData] = useState('');

  return (
    <div>
      <Child1 data={data} />
      <Child2 onUpdate={setData} />
    </div>
  );
}

function Child1({ data }) {
  return <div>{data}</div>;
}

function Child2({ onUpdate }) {
  return (
    <button onClick={() => onUpdate('New Data')}>
      Share
    </button>
  );
}
```

---

## Context API

Use Context API for avoiding prop drilling when passing data through many intermediate components.

### Creating a Context

```jsx
import { createContext, useState } from 'react';

// Create context
const ThemeContext = createContext();

// Create provider component
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(t => t === 'light' ? 'dark' : 'light');
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

```jsx
import { useContext } from 'react';
import ThemeContext from './ThemeContext';

function Button() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <button
      style={{
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff',
      }}
      onClick={toggleTheme}
    >
      Toggle Theme
    </button>
  );
}
```

### Setting up in App Component

```jsx
import { ThemeProvider } from './contexts/ThemeContext';
import Button from './Button';

function App() {
  return (
    <ThemeProvider>
      <div>
        <h1>My App</h1>
        <Button />
      </div>
    </ThemeProvider>
  );
}

export default App;
```

**When to Use Context:**
- Theme management
- Authentication state
- Language/localization
- User preferences
- Avoiding prop drilling in small to medium apps

---

## useReducer Hook

For complex state logic with multiple actions, `useReducer` provides a more structured approach.

### Basic Pattern

```jsx
import { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
}

export default Counter;
```

### With Payload

```jsx
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        todos: [...state.todos, { id: Date.now(), text: action.payload }]
      };
    case 'REMOVE_TODO':
      return {
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    case 'TOGGLE_TODO':
      return {
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, { todos: [] });

  return (
    <div>
      {state.todos.map(todo => (
        <div key={todo.id}>
          <span>{todo.text}</span>
          <button onClick={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}>
            Toggle
          </button>
          <button onClick={() => dispatch({ type: 'REMOVE_TODO', payload: todo.id })}>
            Remove
          </button>
        </div>
      ))}
    </div>
  );
}
```

---

## Custom Hooks

Combine `useState` and `useReducer` with other hooks to create reusable state logic.

### Custom Hook Example: useCounter

```jsx
import { useState } from 'react';

function useCounter(initialValue = 0, step = 1) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(c => c + step);
  const decrement = () => setCount(c => c - step);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}

// Usage
function App() {
  const counter = useCounter(0, 2);

  return (
    <div>
      <p>Count: {counter.count}</p>
      <button onClick={counter.increment}>+</button>
      <button onClick={counter.decrement}>-</button>
      <button onClick={counter.reset}>Reset</button>
    </div>
  );
}
```

### Custom Hook: useAsync

```jsx
import { useState, useEffect } from 'react';

function useAsync(asyncFunction, immediate = true) {
  const [state, setState] = useState({
    status: 'idle',
    data: null,
    error: null,
  });

  const execute = async () => {
    setState({ status: 'pending', data: null, error: null });
    try {
      const response = await asyncFunction();
      setState({ status: 'success', data: response, error: null });
    } catch (error) {
      setState({ status: 'error', data: null, error });
    }
  };

  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, []);

  return { ...state, execute };
}

// Usage
function UserList() {
  const { status, data, error, execute } = useAsync(
    () => fetch('/api/users').then(r => r.json()),
    true
  );

  if (status === 'pending') return <p>Loading...</p>;
  if (status === 'error') return <p>Error: {error.message}</p>;

  return (
    <div>
      {data?.map(user => <div key={user.id}>{user.name}</div>)}
      <button onClick={execute}>Refresh</button>
    </div>
  );
}
```

---

## Third-Party Libraries

### Redux

For large-scale applications with complex state management needs:

```jsx
// Redux pattern (simplified)
import { createSlice, configureStore } from '@reduxjs/toolkit';
import { useSelector, useDispatch } from 'react-redux';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
    decrement: (state) => { state.value -= 1; },
  },
});

const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
  },
});

function Counter() {
  const count = useSelector(state => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => dispatch(counterSlice.actions.increment())}>+</button>
    </div>
  );
}
```

### Zustand

Lightweight alternative to Redux:

```jsx
import create from 'zustand';

const useCounterStore = create(set => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));

function Counter() {
  const { count, increment, decrement } = useCounterStore();

  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

### Jotai

Primitive and flexible state management:

```jsx
import { atom, useAtom } from 'jotai';

const countAtom = atom(0);

function Counter() {
  const [count, setCount] = useAtom(countAtom);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

---

## Best Practices

### 1. **Keep State as Local as Possible**
```jsx
// ✅ Good: State only where needed
function UserProfile() {
  const [isOpen, setIsOpen] = useState(false);
  return <div>{isOpen && <div>Profile details</div>}</div>;
}

// ❌ Bad: Unnecessary state in parent
function App() {
  const [profileOpen, setProfileOpen] = useState(false);
  return <UserProfile isOpen={profileOpen} />;
}
```

### 2. **Use Derived State Sparingly**
```jsx
// ❌ Bad: Duplicate state
function User() {
  const [firstName, setFirstName] = useState('John');
  const [lastName, setLastName] = useState('Doe');
  const [fullName, setFullName] = useState('John Doe');
}

// ✅ Good: Derive from existing state
function User() {
  const [firstName, setFirstName] = useState('John');
  const [lastName, setLastName] = useState('Doe');
  const fullName = `${firstName} ${lastName}`;
}
```

### 3. **Batch State Updates**
```jsx
// ✅ Good: Single render
function Form() {
  const [form, setForm] = useState({ name: '', email: '' });

  const handleSubmit = () => {
    setForm({ name: 'John', email: 'john@example.com' });
  };
}

// Multiple updates (React 18+ automatically batches)
```

### 4. **Memoize Context Values**
```jsx
import { useMemo } from 'react';

function Provider({ children }) {
  const [theme, setTheme] = useState('light');

  // ✅ Prevent unnecessary re-renders
  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

---

## Common Patterns

### Undo/Redo Pattern

```jsx
function useHistory(initialValue) {
  const [state, setState] = useState(initialValue);
  const [history, setHistory] = useState([initialValue]);
  const [pointer, setPointer] = useState(0);

  const set = (newState) => {
    setHistory([...history.slice(0, pointer + 1), newState]);
    setPointer(pointer + 1);
    setState(newState);
  };

  const undo = () => {
    if (pointer > 0) {
      const newPointer = pointer - 1;
      setPointer(newPointer);
      setState(history[newPointer]);
    }
  };

  const redo = () => {
    if (pointer < history.length - 1) {
      const newPointer = pointer + 1;
      setPointer(newPointer);
      setState(history[newPointer]);
    }
  };

  return { state, set, undo, redo };
}
```

### Form State Management

```jsx
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    // Validate and submit
    onSubmit(values);
  };

  return { values, errors, handleChange, handleSubmit };
}
```

### Persisted State (localStorage)

```jsx
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.log(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.log(error);
    }
  };

  return [storedValue, setValue];
}

// Usage
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return <div>Current theme: {theme}</div>;
}
```

---

## Decision Tree

```
Choose your state management approach:

                        Need to share state?
                              /        \
                            No          Yes
                            |            |
                       useState       Prop drilling?
                                        /      \
                                      No        Yes
                                      |          |
                                 useState    Many levels?
                                            /        \
                                          No          Yes
                                          |            |
                                      Lift up      Context API
                                                 /          \
                                          Simple       Complex
                                            |              |
                                         Context      useReducer
                                                           or
                                                       Redux/Zustand
```

---

## Summary

| Approach | When to Use | Pros | Cons |
|----------|-----------|------|------|
| **useState** | Simple local state | Easy, built-in, performant | Limited to single component |
| **Lifting State** | Few levels deep | Simple, no external deps | Prop drilling |
| **Context API** | Moderate app complexity | No external deps, good for theming | Re-renders whole tree |
| **useReducer** | Complex state logic | Predictable, testable | More boilerplate |
| **Custom Hooks** | Reusable logic | Composable, DRY | Requires planning |
| **Redux** | Large enterprise apps | Powerful, great DevTools | Lots of boilerplate |
| **Zustand** | Modern alternatives | Minimal, flexible | Less ecosystem support |

Choose based on your app's complexity, team experience, and specific requirements!
