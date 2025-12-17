# React Components: A Comprehensive Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Functional Components](#functional-components)
3. [Class Components](#class-components)
4. [Pure Components](#pure-components)
5. [Component Composition](#component-composition)
6. [Props Drilling](#props-drilling)
7. [Best Practices](#best-practices)

---

## Introduction

React components are the fundamental building blocks of React applications. A component is a JavaScript function or class that returns JSX (JavaScript XML) describing what should appear on the screen. Components allow you to split the UI into independent, reusable pieces.

### Types of Components:
- **Functional Components**: JavaScript functions that return JSX
- **Class Components**: ES6 classes that extend React.Component
- **Pure Components**: Class components that perform shallow prop/state comparison
- **Stateless Components**: Components that don't manage state

---

## Functional Components

Functional components are JavaScript functions that return JSX. They are simpler, more concise, and have become the preferred way to write React components, especially with the introduction of Hooks.

### Basic Functional Component

```jsx
// Simple greeting component
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Arrow function variant
const Welcome = (props) => {
  return <h1>Hello, {props.name}!</h1>;
};

// Usage
<Welcome name="Alice" />
```

### Functional Component with State (Using Hooks)

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return (
    <div>
      <h2>Counter: {count}</h2>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

### Functional Component with Effects

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]); // Effect runs when userId changes

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!user) return <p>No user found</p>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Role: {user.role}</p>
    </div>
  );
}
```

### Functional Component with Multiple States

```jsx
import { useState } from 'react';

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');
  const [filter, setFilter] = useState('all'); // 'all', 'completed', 'pending'

  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos([...todos, { id: Date.now(), text: inputValue, completed: false }]);
      setInputValue('');
    }
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  const filteredTodos = todos.filter(todo => {
    if (filter === 'completed') return todo.completed;
    if (filter === 'pending') return !todo.completed;
    return true;
  });

  return (
    <div className="todo-app">
      <h1>My Todos</h1>
      <input
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
        placeholder="Add a new todo..."
      />
      <button onClick={addTodo}>Add Todo</button>

      <div>
        <button onClick={() => setFilter('all')}>All</button>
        <button onClick={() => setFilter('completed')}>Completed</button>
        <button onClick={() => setFilter('pending')}>Pending</button>
      </div>

      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Class Components

Class components are ES6 classes that extend `React.Component`. They have lifecycle methods and manage state through `this.state`.

### Basic Class Component

```jsx
import React from 'react';

class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

// Usage
<Welcome name="Bob" />
```

### Class Component with State

```jsx
import React from 'react';

class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  decrement = () => {
    this.setState({ count: this.state.count - 1 });
  };

  render() {
    return (
      <div>
        <h2>Counter: {this.state.count}</h2>
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.decrement}>Decrement</button>
      </div>
    );
  }
}
```

### Class Component with Lifecycle Methods

```jsx
import React from 'react';

class UserProfile extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      user: null,
      loading: true,
      error: null
    };
  }

  componentDidMount() {
    // Called after component is rendered
    this.fetchUser();
  }

  componentDidUpdate(prevProps) {
    // Called after component is updated
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }

  componentWillUnmount() {
    // Called before component is removed from DOM
    console.log('Component is being removed');
  }

  fetchUser = async () => {
    try {
      this.setState({ loading: true });
      const response = await fetch(`/api/users/${this.props.userId}`);
      const data = await response.json();
      this.setState({ user: data, loading: false });
    } catch (error) {
      this.setState({ error: error.message, loading: false });
    }
  };

  render() {
    const { user, loading, error } = this.state;

    if (loading) return <p>Loading...</p>;
    if (error) return <p>Error: {error}</p>;
    if (!user) return <p>No user found</p>;

    return (
      <div>
        <h2>{user.name}</h2>
        <p>Email: {user.email}</p>
        <p>Role: {user.role}</p>
      </div>
    );
  }
}
```

### Class Component with Complex State Management

```jsx
import React from 'react';

class TodoApp extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      todos: [],
      inputValue: '',
      filter: 'all'
    };
  }

  componentDidMount() {
    // Load todos from localStorage
    const savedTodos = localStorage.getItem('todos');
    if (savedTodos) {
      this.setState({ todos: JSON.parse(savedTodos) });
    }
  }

  componentDidUpdate() {
    // Save todos to localStorage whenever they change
    localStorage.setItem('todos', JSON.stringify(this.state.todos));
  }

  addTodo = () => {
    const { todos, inputValue } = this.state;
    if (inputValue.trim()) {
      const newTodo = {
        id: Date.now(),
        text: inputValue,
        completed: false
      };
      this.setState({
        todos: [...todos, newTodo],
        inputValue: ''
      });
    }
  };

  toggleTodo = (id) => {
    this.setState({
      todos: this.state.todos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    });
  };

  deleteTodo = (id) => {
    this.setState({
      todos: this.state.todos.filter(todo => todo.id !== id)
    });
  };

  handleInputChange = (e) => {
    this.setState({ inputValue: e.target.value });
  };

  handleFilterChange = (filter) => {
    this.setState({ filter });
  };

  getFilteredTodos = () => {
    const { todos, filter } = this.state;
    switch (filter) {
      case 'completed':
        return todos.filter(todo => todo.completed);
      case 'pending':
        return todos.filter(todo => !todo.completed);
      default:
        return todos;
    }
  };

  render() {
    const { inputValue } = this.state;
    const filteredTodos = this.getFilteredTodos();

    return (
      <div className="todo-app">
        <h1>My Todos</h1>
        <input
          value={inputValue}
          onChange={this.handleInputChange}
          onKeyPress={(e) => e.key === 'Enter' && this.addTodo()}
          placeholder="Add a new todo..."
        />
        <button onClick={this.addTodo}>Add Todo</button>

        <div>
          <button onClick={() => this.handleFilterChange('all')}>All</button>
          <button onClick={() => this.handleFilterChange('completed')}>Completed</button>
          <button onClick={() => this.handleFilterChange('pending')}>Pending</button>
        </div>

        <ul>
          {filteredTodos.map(todo => (
            <li key={todo.id}>
              <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => this.toggleTodo(todo.id)}
              />
              <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
                {todo.text}
              </span>
              <button onClick={() => this.deleteTodo(todo.id)}>Delete</button>
            </li>
          ))}
        </ul>
      </div>
    );
  }
}
```

---

## Pure Components

Pure Components perform a shallow comparison of props and state. If they haven't changed, the component won't re-render. This is useful for performance optimization.

### Class-based Pure Component

```jsx
import React from 'react';

class UserCard extends React.PureComponent {
  render() {
    const { user } = this.props;
    console.log('UserCard rendered');
    
    return (
      <div className="user-card">
        <h3>{user.name}</h3>
        <p>Email: {user.email}</p>
        <p>Role: {user.role}</p>
      </div>
    );
  }
}

// Usage
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [
        { id: 1, name: 'Alice', email: 'alice@example.com', role: 'Admin' },
        { id: 2, name: 'Bob', email: 'bob@example.com', role: 'User' }
      ],
      count: 0
    };
  }

  render() {
    const { users, count } = this.state;

    return (
      <div>
        <h1>Users</h1>
        {users.map(user => (
          <UserCard key={user.id} user={user} />
        ))}
        <p>Count: {count}</p>
        <button onClick={() => this.setState({ count: count + 1 })}>
          Increment Count
        </button>
      </div>
    );
  }
}
```

### Functional Component with React.memo (Equivalent to PureComponent)

```jsx
import React from 'react';

// Simple memoized component
const UserCard = React.memo(({ user }) => {
  console.log('UserCard rendered');
  
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>Email: {user.email}</p>
      <p>Role: {user.role}</p>
    </div>
  );
});

// Memoized component with custom comparison
const UserCard = React.memo(
  ({ user, onSelect }) => {
    console.log('UserCard rendered');
    
    return (
      <div className="user-card" onClick={() => onSelect(user.id)}>
        <h3>{user.name}</h3>
        <p>Email: {user.email}</p>
        <p>Role: {user.role}</p>
      </div>
    );
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (don't re-render)
    return prevProps.user.id === nextProps.user.id;
  }
);
```

### Shallow Comparison Behavior

```jsx
import React from 'react';

class PureComponentExample extends React.PureComponent {
  render() {
    const { person } = this.props;
    console.log('PureComponentExample rendered');
    return <p>{person.name}</p>;
  }
}

// Parent component
class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    // ❌ ISSUE: New object created each render, PureComponent will re-render
    const person = { name: 'Alice', age: 30 };

    return (
      <div>
        <PureComponentExample person={person} />
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Count: {this.state.count}
        </button>
      </div>
    );
  }
}

// SOLUTION: Keep object reference stable
class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      person: { name: 'Alice', age: 30 }
    };
  }

  render() {
    // ✅ CORRECT: Same object reference, PureComponent won't re-render
    return (
      <div>
        <PureComponentExample person={this.state.person} />
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Count: {this.state.count}
        </button>
      </div>
    );
  }
}
```

---

## Component Composition

Component composition is the practice of building complex components from simpler, reusable components. There are several composition patterns:

### Container and Presentational Components

```jsx
// Presentational Component (Dumb Component)
function UserListPresentation({ users, onUserSelect, loading }) {
  if (loading) return <p>Loading users...</p>;

  return (
    <div className="user-list">
      <h2>Users</h2>
      <ul>
        {users.map(user => (
          <li key={user.id} onClick={() => onUserSelect(user.id)}>
            {user.name} ({user.email})
          </li>
        ))}
      </ul>
    </div>
  );
}

// Container Component (Smart Component)
function UserListContainer() {
  const [users, setUsers] = React.useState([]);
  const [loading, setLoading] = React.useState(true);
  const [selectedUserId, setSelectedUserId] = React.useState(null);

  React.useEffect(() => {
    const fetchUsers = async () => {
      try {
        setLoading(true);
        const response = await fetch('/api/users');
        const data = await response.json();
        setUsers(data);
      } catch (error) {
        console.error('Failed to fetch users:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  const handleUserSelect = (userId) => {
    setSelectedUserId(userId);
  };

  return (
    <UserListPresentation
      users={users}
      onUserSelect={handleUserSelect}
      loading={loading}
    />
  );
}
```

### Compound Components

```jsx
// Tab Components
function Tabs({ children, defaultTab = 0 }) {
  const [activeTab, setActiveTab] = React.useState(defaultTab);

  const tabs = React.Children.toArray(children).filter(
    child => child.type === Tabs.Tab
  );
  const panels = React.Children.toArray(children).filter(
    child => child.type === Tabs.Panel
  );

  return (
    <div>
      <div className="tabs">
        {tabs.map((tab, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            className={activeTab === index ? 'active' : ''}
          >
            {tab.props.label}
          </button>
        ))}
      </div>
      <div className="tab-content">
        {panels[activeTab]}
      </div>
    </div>
  );
}

Tabs.Tab = ({ label }) => null;
Tabs.Panel = ({ children }) => <div>{children}</div>;

// Usage
function App() {
  return (
    <Tabs defaultTab={0}>
      <Tabs.Tab label="Tab 1" />
      <Tabs.Panel>Content for Tab 1</Tabs.Panel>

      <Tabs.Tab label="Tab 2" />
      <Tabs.Panel>Content for Tab 2</Tabs.Panel>

      <Tabs.Tab label="Tab 3" />
      <Tabs.Panel>Content for Tab 3</Tabs.Panel>
    </Tabs>
  );
}
```

### Higher-Order Component (HOC)

```jsx
// HOC that adds data fetching capability
function withDataFetching(url) {
  return function WithDataFetchingComponent(WrappedComponent) {
    return function(props) {
      const [data, setData] = React.useState(null);
      const [loading, setLoading] = React.useState(true);
      const [error, setError] = React.useState(null);

      React.useEffect(() => {
        const fetchData = async () => {
          try {
            setLoading(true);
            const response = await fetch(url);
            const result = await response.json();
            setData(result);
          } catch (err) {
            setError(err.message);
          } finally {
            setLoading(false);
          }
        };

        fetchData();
      }, [url]);

      return (
        <WrappedComponent
          data={data}
          loading={loading}
          error={error}
          {...props}
        />
      );
    };
  };
}

// Component to wrap
function UserList({ data, loading, error }) {
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// Create enhanced component
const UserListWithData = withDataFetching('/api/users')(UserList);
```

### Render Props Pattern

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = React.useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return (
    <div onMouseMove={handleMouseMove} style={{ width: '100%', height: '500px' }}>
      {render(position)}
    </div>
  );
}

// Usage
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <h1>Mouse position: ({x}, {y})</h1>
      )}
    />
  );
}

// Alternative with children as render prop
function MouseTracker({ children }) {
  const [position, setPosition] = React.useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return (
    <div onMouseMove={handleMouseMove} style={{ width: '100%', height: '500px' }}>
      {typeof children === 'function' ? children(position) : children}
    </div>
  );
}

// Usage
<MouseTracker>
  {({ x, y }) => <h1>Mouse position: ({x}, {y})</h1>}
</MouseTracker>
```

---

## Props Drilling

Props drilling (also called "prop threading") occurs when you pass props through multiple levels of components that don't directly use those props. It's a common issue in deeply nested component hierarchies.

### The Props Drilling Problem

```jsx
// Level 1: App
function App() {
  const [theme, setTheme] = React.useState('light');
  const [language, setLanguage] = React.useState('en');

  return (
    <div>
      <Header theme={theme} language={language} />
      <MainContent theme={theme} language={language} />
      <Footer theme={theme} language={language} />
    </div>
  );
}

// Level 2: MainContent doesn't use these props but passes them down
function MainContent({ theme, language }) {
  return (
    <div>
      <Sidebar theme={theme} language={language} />
      <Content theme={theme} language={language} />
    </div>
  );
}

// Level 3: Sidebar doesn't use these props but passes them down
function Sidebar({ theme, language }) {
  return (
    <aside>
      <Menu theme={theme} language={language} />
    </aside>
  );
}

// Level 4: Menu finally uses the props
function Menu({ theme, language }) {
  return (
    <nav style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
      {language === 'en' ? 'Menu' : 'Menú'}
    </nav>
  );
}

// ❌ PROBLEM: theme and language are passed through MainContent and Sidebar
// even though they don't use them. Adds unnecessary complexity.
```

### Solution 1: Context API

```jsx
import React from 'react';

// Create Context
const AppContext = React.createContext();

// Provider Component
function App() {
  const [theme, setTheme] = React.useState('light');
  const [language, setLanguage] = React.useState('en');

  const value = {
    theme,
    setTheme,
    language,
    setLanguage
  };

  return (
    <AppContext.Provider value={value}>
      <div>
        <Header />
        <MainContent />
        <Footer />
      </div>
    </AppContext.Provider>
  );
}

// Create a custom hook for easier consumption
function useAppContext() {
  const context = React.useContext(AppContext);
  if (!context) {
    throw new Error('useAppContext must be used within AppContext.Provider');
  }
  return context;
}

// Level 2: MainContent no longer needs to pass props
function MainContent() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// Level 3: Sidebar no longer needs to pass props
function Sidebar() {
  return (
    <aside>
      <Menu />
    </aside>
  );
}

// Level 4: Menu gets context directly
function Menu() {
  const { theme, language } = useAppContext();

  return (
    <nav style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
      {language === 'en' ? 'Menu' : 'Menú'}
    </nav>
  );
}

// ✅ SOLUTION: theme and language are available directly to Menu
// without being passed through intermediate components.
```

### Solution 2: Component Composition

```jsx
// Restructure components to reduce nesting depth

function Header() {
  return <header>Header</header>;
}

function Footer() {
  return <footer>Footer</footer>;
}

// Pass Menu directly to MainContent instead of nesting deeper
function MainContent({ menu }) {
  return (
    <div>
      <Sidebar />
      {menu}
      <Content />
    </div>
  );
}

function App() {
  const [theme, setTheme] = React.useState('light');
  const [language, setLanguage] = React.useState('en');

  const menu = (
    <Menu theme={theme} language={language} />
  );

  return (
    <div>
      <Header />
      <MainContent menu={menu} />
      <Footer />
    </div>
  );
}

// ✅ Menu receives props directly from App, reducing drilling depth.
```

### Solution 3: Compound Component Pattern

```jsx
function App() {
  const [theme, setTheme] = React.useState('light');
  const [language, setLanguage] = React.useState('en');

  return (
    <AppLayout theme={theme} language={language}>
      <AppLayout.Header />
      <AppLayout.MainContent>
        <AppLayout.Sidebar />
        <AppLayout.Menu />
      </AppLayout.MainContent>
      <AppLayout.Footer />
    </AppLayout>
  );
}

function AppLayout({ children, theme, language }) {
  const value = { theme, language };

  return (
    <AppContext.Provider value={value}>
      <div className="app-layout">
        {children}
      </div>
    </AppContext.Provider>
  );
}

AppLayout.Header = function Header() {
  const { language } = React.useContext(AppContext);
  return <header>{language === 'en' ? 'Header' : 'Encabezado'}</header>;
};

AppLayout.MainContent = function MainContent({ children }) {
  return <main>{children}</main>;
};

AppLayout.Sidebar = function Sidebar() {
  return <aside>Sidebar</aside>;
};

AppLayout.Menu = function Menu() {
  const { theme, language } = React.useContext(AppContext);
  return (
    <nav style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
      {language === 'en' ? 'Menu' : 'Menú'}
    </nav>
  );
};

AppLayout.Footer = function Footer() {
  return <footer>Footer</footer>;
};
```

### Solution 4: State Management Library (Redux/Zustand)

```jsx
// Using Zustand (simpler alternative to Redux)
import create from 'zustand';

const useAppStore = create(set => ({
  theme: 'light',
  language: 'en',
  setTheme: (theme) => set({ theme }),
  setLanguage: (language) => set({ language })
}));

function Menu() {
  const { theme, language } = useAppStore();

  return (
    <nav style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
      {language === 'en' ? 'Menu' : 'Menú'}
    </nav>
  );
}

function LanguageSwitcher() {
  const { language, setLanguage } = useAppStore();

  return (
    <button onClick={() => setLanguage(language === 'en' ? 'es' : 'en')}>
      {language === 'en' ? 'English' : 'Español'}
    </button>
  );
}

// ✅ Any component can access and modify state without props drilling.
```

---

## Best Practices

### 1. Choose the Right Component Type

```jsx
// ✅ Use functional components with hooks (modern approach)
function Counter() {
  const [count, setCount] = React.useState(0);
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}

// ⚠️ Use class components only if necessary (legacy code)
// ⚠️ Avoid mixing old and new patterns
```

### 2. Keep Components Small and Focused

```jsx
// ❌ DON'T: Component does too many things
function Dashboard() {
  const [users, setUsers] = React.useState([]);
  const [reports, setReports] = React.useState([]);
  const [settings, setSettings] = React.useState({});
  // ... 200 lines of code
}

// ✅ DO: Break into smaller components
function Dashboard() {
  return (
    <div>
      <UserSection />
      <ReportSection />
      <SettingsSection />
    </div>
  );
}
```

### 3. Prop Validation

```jsx
import PropTypes from 'prop-types';

function UserCard({ user, onSelect }) {
  return (
    <div onClick={() => onSelect(user.id)}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}

// Validate props
UserCard.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired
  }).isRequired,
  onSelect: PropTypes.func.isRequired
};

// Default props
UserCard.defaultProps = {
  onSelect: () => {}
};
```

### 4. Memoization for Performance

```jsx
import React from 'react';

// ✅ Memoize expensive components
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
  console.log('Expensive render');
  return <div>{JSON.stringify(data)}</div>;
});

// ✅ Memoize callbacks to prevent unnecessary re-renders
function Parent() {
  const [count, setCount] = React.useState(0);
  
  const memoizedCallback = React.useCallback(() => {
    console.log('Callback called');
  }, []);

  return (
    <div>
      <ExpensiveComponent onCallback={memoizedCallback} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}
```

### 5. Avoid Common Pitfalls

```jsx
// ❌ DON'T: Create new objects in render
function BadExample() {
  return <Component style={{ color: 'red' }} />; // New object each render
}

// ✅ DO: Use constants or memoization
const STYLE = { color: 'red' };
function GoodExample() {
  return <Component style={STYLE} />;
}

// ❌ DON'T: Bind functions in render
<button onClick={this.handleClick.bind(this)}>Click</button>

// ✅ DO: Use arrow functions or bind in constructor
<button onClick={() => this.handleClick()}>Click</button>
// or
<button onClick={this.handleClick}>Click</button> // with arrow function property

// ❌ DON'T: Use array index as key
{items.map((item, index) => <Item key={index} {...item} />)}

// ✅ DO: Use unique, stable identifier
{items.map(item => <Item key={item.id} {...item} />)}
```

---

## Summary

| Component Type | Best For | Pros | Cons |
|---|---|---|---|
| Functional | Modern code, most use cases | Simple, hooks support | Newer pattern |
| Class | Complex logic, lifecycle | Mature, lifecycle methods | More verbose |
| Pure | Performance optimization | Prevents unnecessary re-renders | Shallow comparison only |
| Memoized | Expensive renders | Equivalent to PureComponent | Shallow comparison only |

**Key Takeaways:**
- Use functional components with hooks as the default
- Use composition to build complex UIs
- Use Context API or state management to avoid props drilling
- Keep components small and focused
- Validate props and use default values
- Optimize performance with memoization when needed
- Choose the right abstraction level for your use case
