# React Routing Guide

## Table of Contents
1. [Introduction to Routing](#introduction-to-routing)
2. [React Router v6 Setup](#react-router-v6-setup)
3. [Basic Routing](#basic-routing)
4. [Route Parameters](#route-parameters)
5. [Nested Routes](#nested-routes)
6. [Programmatic Navigation](#programmatic-navigation)
7. [Protected Routes](#protected-routes)
8. [Query Parameters](#query-parameters)
9. [Advanced Routing Patterns](#advanced-routing-patterns)
10. [Best Practices](#best-practices)

---

## Introduction to Routing

Routing is the process of determining what content to display based on the URL. In single-page applications (SPAs), routing allows you to navigate between different views without full page reloads.

### Why Use React Router?
- **Declarative Routing**: Define routes using components
- **Dynamic Route Matching**: Support for parameterized routes
- **Lazy Code Loading**: Load components on demand
- **Dynamic Route Matching**: Render components based on URL patterns
- **Location Transition Handling**: Manage navigation state

---

## React Router v6 Setup

### Installation

```bash
npm install react-router-dom
```

### Basic Setup

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

**Note**: `BrowserRouter` wraps your entire application and manages navigation state.

---

## Basic Routing

### Creating Routes

```jsx
import { Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';
import Contact from './pages/Contact';

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/contact" element={<Contact />} />
    </Routes>
  );
}

export default App;
```

### Navigation with Links

```jsx
import { Link, NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/contact">Contact</Link>
      
      {/* NavLink highlights active route */}
      <NavLink 
        to="/about"
        className={({ isActive }) => isActive ? 'active' : ''}
      >
        About
      </NavLink>
    </nav>
  );
}

export default Navigation;
```

---

## Route Parameters

### Dynamic Route Segments

```jsx
import { Routes, Route, useParams } from 'react-router-dom';

function App() {
  return (
    <Routes>
      <Route path="/user/:id" element={<UserProfile />} />
      <Route path="/posts/:postId/comments/:commentId" element={<Comment />} />
    </Routes>
  );
}

function UserProfile() {
  const { id } = useParams();
  
  return <h1>User Profile - ID: {id}</h1>;
}

function Comment() {
  const { postId, commentId } = useParams();
  
  return (
    <div>
      <h1>Post: {postId}</h1>
      <p>Comment: {commentId}</p>
    </div>
  );
}
```

### Optional Parameters

```jsx
function App() {
  return (
    <Routes>
      {/* /user or /user/123 */}
      <Route path="/user/:id?" element={<User />} />
    </Routes>
  );
}

function User() {
  const { id } = useParams();
  
  return <h1>User: {id || 'Not specified'}</h1>;
}
```

---

## Nested Routes

### Creating Nested Routes

```jsx
import { Routes, Route, Outlet } from 'react-router-dom';

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />}>
        <Route path="stats" element={<Stats />} />
        <Route path="users" element={<Users />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}

function Dashboard() {
  return (
    <div className="dashboard">
      <nav>
        <Link to="stats">Stats</Link>
        <Link to="users">Users</Link>
        <Link to="settings">Settings</Link>
      </nav>
      
      {/* Renders child routes here */}
      <Outlet />
    </div>
  );
}

function Stats() {
  return <h1>Dashboard Stats</h1>;
}

function Users() {
  return <h1>Users Management</h1>;
}

function Settings() {
  return <h1>Dashboard Settings</h1>;
}
```

### Default Child Route

```jsx
function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />}>
        <Route index element={<Overview />} />
        <Route path="stats" element={<Stats />} />
      </Route>
    </Routes>
  );
}

// /dashboard renders Overview
// /dashboard/stats renders Stats
```

---

## Programmatic Navigation

### useNavigate Hook

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    // Perform login logic
    navigate('/dashboard');
  };
  
  const handleCancel = () => {
    // Navigate back
    navigate(-1);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="email" placeholder="Email" />
      <button type="submit">Login</button>
      <button type="button" onClick={handleCancel}>Cancel</button>
    </form>
  );
}

export default LoginForm;
```

### Navigation Options

```jsx
const navigate = useNavigate();

// Replace in history
navigate('/dashboard', { replace: true });

// With state
navigate('/user/123', { state: { from: 'profile' } });

// Relative navigation
navigate('../parent', { relative: 'route' });
```

### useLocation Hook

```jsx
import { useLocation } from 'react-router-dom';

function Page() {
  const location = useLocation();
  
  console.log(location.pathname); // Current path
  console.log(location.search);   // Query string
  console.log(location.state);    // Passed state
  
  return <h1>Current path: {location.pathname}</h1>;
}
```

---

## Protected Routes

### Route Guard Pattern

```jsx
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ element, isAuthenticated }) {
  return isAuthenticated ? element : <Navigate to="/login" />;
}

function App() {
  const [isAuthenticated, setIsAuthenticated] = React.useState(false);
  
  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      <Route 
        path="/dashboard" 
        element={<ProtectedRoute 
          element={<Dashboard />} 
          isAuthenticated={isAuthenticated} 
        />} 
      />
    </Routes>
  );
}
```

### Enhanced Protected Route Component

```jsx
import { Navigate } from 'react-router-dom';

const ProtectedRoute = ({ element, isAuthenticated, isLoading }) => {
  if (isLoading) {
    return <div>Loading...</div>;
  }
  
  return isAuthenticated ? element : <Navigate to="/login" replace />;
};

export default ProtectedRoute;
```

---

## Query Parameters

### Accessing Query Parameters

```jsx
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  // Get query parameter
  const query = searchParams.get('q');
  const page = searchParams.get('page') || 1;
  
  const handleSearch = (term) => {
    // Update URL with new query params
    setSearchParams({ q: term, page: 1 });
  };
  
  return (
    <div>
      <input 
        type="text" 
        defaultValue={query}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      <p>Results for: {query}</p>
      <p>Page: {page}</p>
    </div>
  );
}
```

### Passing Query Parameters

```jsx
import { Link } from 'react-router-dom';

function App() {
  return (
    <>
      <Link to="/search?q=react&page=1">Search React</Link>
      <Link to="/search?q=routing&page=2">Search Routing</Link>
    </>
  );
}
```

---

## Advanced Routing Patterns

### Wildcard Routes (404 Handling)

```jsx
import { Routes, Route } from 'react-router-dom';

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      {/* Catch-all route */}
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}

function NotFound() {
  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <Link to="/">Go Home</Link>
    </div>
  );
}
```

### Lazy Loading Routes

```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';
import Home from './pages/Home';

// Code splitting - component loaded on demand
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Suspense>
  );
}
```

### Route Transitions

```jsx
import { useState, useEffect } from 'react';
import { useLocation } from 'react-router-dom';

function RouteTransition({ children }) {
  const [isTransitioning, setIsTransitioning] = useState(false);
  const location = useLocation();
  
  useEffect(() => {
    setIsTransitioning(true);
    const timer = setTimeout(() => setIsTransitioning(false), 300);
    
    return () => clearTimeout(timer);
  }, [location]);
  
  return (
    <div className={isTransitioning ? 'fade-out' : 'fade-in'}>
      {children}
    </div>
  );
}

export default RouteTransition;
```

### Scroll to Top on Route Change

```jsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

function ScrollToTop() {
  const { pathname } = useLocation();
  
  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);
  
  return null;
}

export default ScrollToTop;

// In App.jsx
function App() {
  return (
    <>
      <ScrollToTop />
      <Routes>
        {/* routes */}
      </Routes>
    </>
  );
}
```

---

## Best Practices

### 1. **Structure and Organization**
```
src/
├── pages/
│   ├── Home.jsx
│   ├── About.jsx
│   └── NotFound.jsx
├── components/
├── hooks/
├── App.jsx
└── index.jsx
```

### 2. **Centralized Route Configuration**
```jsx
// routes/index.js
export const routes = [
  { path: '/', element: 'Home', component: () => import('../pages/Home') },
  { path: '/about', element: 'About', component: () => import('../pages/About') },
  { path: '/contact', element: 'Contact', component: () => import('../pages/Contact') },
];

// App.jsx
function App() {
  return (
    <Routes>
      {routes.map((route) => (
        <Route key={route.path} path={route.path} element={<route.component />} />
      ))}
    </Routes>
  );
}
```

### 3. **Use Custom Hooks for Navigation**
```jsx
// hooks/useNavigation.js
import { useNavigate, useLocation } from 'react-router-dom';

export function useNavigation() {
  const navigate = useNavigate();
  const location = useLocation();
  
  return {
    goHome: () => navigate('/'),
    goBack: () => navigate(-1),
    goToUser: (id) => navigate(`/user/${id}`),
    currentPath: location.pathname,
  };
}

// Usage
function Component() {
  const { goHome, currentPath } = useNavigation();
  
  return <button onClick={goHome}>Home</button>;
}
```

### 4. **Handle Loading and Error States**
```jsx
function RouteWithErrorBoundary() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorPage error={error} />;
  
  return <PageContent />;
}
```

### 5. **SEO Optimization**
```jsx
// Use Helmet for meta tags
import { Helmet } from 'react-helmet-async';

function Home() {
  return (
    <>
      <Helmet>
        <title>Home - My App</title>
        <meta name="description" content="Welcome to My App" />
      </Helmet>
      <h1>Home Page</h1>
    </>
  );
}
```

### 6. **Preserve Scroll Position**
```jsx
// Use a custom hook to manage scroll restoration
export function useScrollRestoration() {
  const location = useLocation();
  const scrollPositions = useRef({});
  
  // Save scroll position before navigation
  useEffect(() => {
    const handleBeforeUnload = () => {
      scrollPositions.current[location.pathname] = window.scrollY;
    };
    
    window.addEventListener('beforeunload', handleBeforeUnload);
    return () => window.removeEventListener('beforeunload', handleBeforeUnload);
  }, [location]);
  
  // Restore scroll position after navigation
  useEffect(() => {
    const scrollPos = scrollPositions.current[location.pathname] || 0;
    window.scrollTo(0, scrollPos);
  }, [location]);
}
```

### 7. **Avoid Common Pitfalls**
- **Don't use inline functions for routes**: Creates new component on each render
- **Don't forget to wrap with BrowserRouter**: Required at app root
- **Don't hardcode routes**: Use constants or configuration
- **Test navigation flows**: Ensure all routes work correctly
- **Handle 404s**: Provide user-friendly error pages

---

## Summary

React Router v6 provides powerful tools for managing navigation in SPAs:
- Use `<Routes>` and `<Route>` for declarative routing
- Use `useParams()` for dynamic segments
- Use `useNavigate()` for programmatic navigation
- Use `useSearchParams()` for query parameters
- Implement protected routes for authentication
- Use lazy loading for code splitting
- Follow best practices for scalability and maintainability

For more information, visit [React Router Documentation](https://reactrouter.com/)