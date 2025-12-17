# React Performance Optimization Guide

## Table of Contents
1. [Understanding React Rendering](#understanding-react-rendering)
2. [Memoization Techniques](#memoization-techniques)
3. [Code Splitting](#code-splitting)
4. [Lazy Loading](#lazy-loading)
5. [Virtual Lists](#virtual-lists)
6. [Profiling and Monitoring](#profiling-and-monitoring)
7. [Bundle Optimization](#bundle-optimization)
8. [Best Practices](#best-practices)

---

## Understanding React Rendering

### Why Performance Matters
- Affects user experience and engagement
- Impacts SEO rankings
- Reduces bounce rates
- Improves mobile performance

### React Rendering Cycle
```
State/Props Change → Re-render → Reconciliation → DOM Update
```

### Key Concepts

**Reconciliation (Diffing Algorithm)**
```javascript
// React compares virtual DOM trees to find minimal changes
const oldVDOM = { type: 'div', props: { id: '1' } };
const newVDOM = { type: 'div', props: { id: '2' } };
// Only id attribute is updated
```

**Unnecessary Re-renders**
- Parent component updates cause child re-renders
- Props/state changes trigger re-renders even if content is unchanged
- Inline function definitions cause re-renders

---

## Memoization Techniques

### 1. React.memo - Prevent Unnecessary Component Re-renders

```javascript
// Without memoization - re-renders on every parent update
const ExpensiveComponent = ({ data, id }) => {
  console.log('Rendered');
  return <div>{data.name} - {id}</div>;
};

// With memoization - only re-renders if props change
const MemoizedComponent = React.memo(ExpensiveComponent);

// Custom comparison function
const CustomMemo = React.memo(
  ExpensiveComponent,
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.id === nextProps.id;
  }
);
```

**When to Use:**
- Pure functional components
- Expensive rendering logic
- Components that receive same props frequently

### 2. useMemo Hook - Memoize Expensive Computations

```javascript
import { useMemo } from 'react';

function DataProcessor({ data, multiplier }) {
  // Without useMemo - expensive calculation runs every render
  const expensiveValue = data.map(x => x * multiplier)
    .filter(x => x > 10)
    .reduce((a, b) => a + b, 0);

  // With useMemo - only recalculates when dependencies change
  const optimizedValue = useMemo(() => {
    console.log('Computing expensive value...');
    return data
      .map(x => x * multiplier)
      .filter(x => x > 10)
      .reduce((a, b) => a + b, 0);
  }, [data, multiplier]);

  return <div>{optimizedValue}</div>;
}
```

**Use Cases:**
- Heavy computations (sorting, filtering, transformations)
- Creating objects/arrays to prevent re-renders of child components
- Results of expensive API calls

### 3. useCallback Hook - Memoize Functions

```javascript
import { useCallback, useState } from 'react';

function Parent() {
  const [count, setCount] = useState(0);

  // Without useCallback - new function created every render
  const handleClick = () => {
    setCount(count + 1);
  };

  // With useCallback - same function reference if dependencies unchanged
  const memoizedCallback = useCallback(() => {
    setCount(count + 1);
  }, [count]);

  return (
    <div>
      <p>Count: {count}</p>
      <MemoChild onClick={memoizedCallback} />
    </div>
  );
}

// Memoized child component
const MemoChild = React.memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Increment</button>;
});
```

**When Needed:**
- Passing callbacks to memoized child components
- Dependencies in useEffect
- Event handlers in optimization-critical scenarios

---

## Code Splitting

### Dynamic Imports

```javascript
// Traditional import - loaded at bundle time
import HeavyComponent from './HeavyComponent';

// Dynamic import - loaded on demand
const HeavyComponent = lazy(() => import('./HeavyComponent'));
```

### Route-based Code Splitting

```javascript
import { Suspense, lazy } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Split code by routes
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Analytics = lazy(() => import('./pages/Analytics'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/analytics" element={<Analytics />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### Component-based Code Splitting

```javascript
import { Suspense, lazy } from 'react';

// Split modal components
const EditModal = lazy(() => import('./modals/EditModal'));
const DeleteConfirmModal = lazy(() => import('./modals/DeleteConfirmModal'));

function DataTable() {
  const [showEditModal, setShowEditModal] = useState(false);

  return (
    <div>
      <table>{/* ... */}</table>
      
      <Suspense fallback={<div>Loading modal...</div>}>
        {showEditModal && <EditModal onClose={() => setShowEditModal(false)} />}
      </Suspense>
    </div>
  );
}
```

---

## Lazy Loading

### Image Lazy Loading

```javascript
// Native browser lazy loading
function ImageGallery({ images }) {
  return (
    <div>
      {images.map(image => (
        <img
          key={image.id}
          src={image.src}
          alt={image.alt}
          loading="lazy"
        />
      ))}
    </div>
  );
}

// Intersection Observer API (more control)
import { useEffect, useRef } from 'react';

function LazyImage({ src, alt }) {
  const imgRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = src;
        img.srcSet = srcSet;
        observer.unobserve(img);
      }
    });

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, [src]);

  return <img ref={imgRef} alt={alt} />;
}
```

### Data Lazy Loading

```javascript
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const containerRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(async ([entry]) => {
      if (entry.isIntersecting && !isLoading) {
        setIsLoading(true);
        const newItems = await fetchMoreItems();
        setItems(prev => [...prev, ...newItems]);
        setIsLoading(false);
      }
    });

    if (containerRef.current) {
      observer.observe(containerRef.current);
    }

    return () => observer.disconnect();
  }, [isLoading]);

  return (
    <div>
      {items.map(item => <Item key={item.id} {...item} />)}
      <div ref={containerRef}>{isLoading && <LoadingSpinner />}</div>
    </div>
  );
}
```

---

## Virtual Lists

### Why Virtual Lists Matter
- Rendering 10,000 items = poor performance
- Virtual lists render only visible items
- Significant memory and CPU savings

### Using react-window

```javascript
import { FixedSizeList as List } from 'react-window';

function VirtualList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );

  return (
    <List
      height={600}
      itemCount={items.length}
      itemSize={35}
      width="100%"
    >
      {Row}
    </List>
  );
}

// For variable-size items
import { VariableSizeList } from 'react-window';

function VariableList({ items }) {
  const itemSizes = [50, 75, 50, 100]; // Dynamic sizes

  return (
    <VariableSizeList
      height={600}
      itemCount={items.length}
      itemSize={index => itemSizes[index]}
      width="100%"
    >
      {Row}
    </VariableSizeList>
  );
}
```

### Custom Virtual Scroll Implementation

```javascript
function SimpleVirtualScroll({ items, itemHeight = 50, containerHeight = 500 }) {
  const [scrollTop, setScrollTop] = useState(0);

  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.ceil((scrollTop + containerHeight) / itemHeight);
  const visibleItems = items.slice(startIndex, endIndex);
  const totalHeight = items.length * itemHeight;
  const offsetY = startIndex * itemHeight;

  return (
    <div
      style={{
        height: containerHeight,
        overflow: 'auto',
        onScroll: (e) => setScrollTop(e.currentTarget.scrollTop)
      }}
    >
      <div style={{ height: totalHeight }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, i) => (
            <div key={startIndex + i} style={{ height: itemHeight }}>
              {item.name}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
```

---

## Profiling and Monitoring

### React DevTools Profiler

```javascript
// Mark components for profiling
import { Profiler } from 'react';

function onRenderCallback(
  id, // unique id of the profiler
  phase, // "mount" or "update"
  actualDuration, // time spent rendering
  baseDuration, // estimated time without memoization
  startTime, // React assigned time
  commitTime, // time React committed the update
  interactions // Set of interactions
) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Header />
      <Main />
      <Footer />
    </Profiler>
  );
}
```

### Web Vitals Monitoring

```javascript
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

// Cumulative Layout Shift
getCLS(console.log);

// First Input Delay
getFID(console.log);

// First Contentful Paint
getFCP(console.log);

// Largest Contentful Paint
getLCP(console.log);

// Time to First Byte
getTTFB(console.log);

// Send metrics to analytics
function sendMetricsToAnalytics(metric) {
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/analytics', JSON.stringify(metric));
  }
}

getLCP(sendMetricsToAnalytics);
getFID(sendMetricsToAnalytics);
getCLS(sendMetricsToAnalytics);
```

### Performance Monitoring Hook

```javascript
function useRenderCount(componentName) {
  const renderCount = useRef(0);

  useEffect(() => {
    renderCount.current++;
    console.log(`${componentName} rendered ${renderCount.current} times`);
  });

  return renderCount.current;
}

function MyComponent() {
  const renders = useRenderCount('MyComponent');
  return <div>Renders: {renders}</div>;
}
```

---

## Bundle Optimization

### Webpack Configuration

```javascript
// webpack.config.js
module.exports = {
  mode: 'production',
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: true,
          mangle: true
        }
      })
    ],
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        },
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

### Tree Shaking

```javascript
// Avoid default exports for better tree shaking
// ❌ Bad
export default {
  util1: () => {},
  util2: () => {}
};

// ✅ Good
export const util1 = () => {};
export const util2 = () => {};

// Only unused exports are removed in production build
import { util1 } from './utils'; // util2 is tree-shaken
```

### Analyzing Bundle Size

```bash
# Install bundle analyzer
npm install --save-dev webpack-bundle-analyzer

# webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin()
  ]
};

# Run build to see interactive visualization
npm run build
```

---

## Best Practices

### 1. **Avoid Creating Functions Inside Render**

```javascript
// ❌ Bad - new function created every render
function Component() {
  return <button onClick={() => doSomething()}>Click</button>;
}

// ✅ Good - function reference stays same
function Component() {
  const handleClick = useCallback(() => doSomething(), []);
  return <button onClick={handleClick}>Click</button>;
}
```

### 2. **Avoid Creating Objects Inside Render**

```javascript
// ❌ Bad - new object every render
function Component() {
  const style = { color: 'red', fontSize: '16px' };
  return <div style={style}>Text</div>;
}

// ✅ Good - object created once
const style = { color: 'red', fontSize: '16px' };
function Component() {
  return <div style={style}>Text</div>;
}

// Or for dynamic styles
function Component({ isDark }) {
  const style = useMemo(
    () => ({ color: isDark ? 'white' : 'black' }),
    [isDark]
  );
  return <div style={style}>Text</div>;
}
```

### 3. **Use Keys Correctly in Lists**

```javascript
// ❌ Bad - using index as key (causes performance issues with reordering)
{items.map((item, index) => (
  <li key={index}>{item.name}</li>
))}

// ✅ Good - use unique, stable identifier
{items.map(item => (
  <li key={item.id}>{item.name}</li>
))}
```

### 4. **Debounce and Throttle Event Handlers**

```javascript
import { useCallback, useRef } from 'react';

// Debounce - wait for user to stop typing
function useDebounce(callback, delay) {
  const timeoutRef = useRef(null);

  return useCallback((...args) => {
    clearTimeout(timeoutRef.current);
    timeoutRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  }, [callback, delay]);
}

// Throttle - limit function calls
function useThrottle(callback, delay) {
  const lastRanRef = useRef(Date.now());

  return useCallback((...args) => {
    const now = Date.now();
    if (now - lastRanRef.current >= delay) {
      callback(...args);
      lastRanRef.current = now;
    }
  }, [callback, delay]);
}

function SearchComponent() {
  const handleSearch = useDebounce(async (query) => {
    const results = await api.search(query);
    setResults(results);
  }, 300);

  return (
    <input
      type="text"
      onChange={(e) => handleSearch(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### 5. **Optimize Re-renders with useReducer**

```javascript
// Instead of multiple useState calls
function useReducer(reducer, initialState) {
  const [state, setState] = React.useState(initialState);

  return [
    state,
    (action) => setState(prev => reducer(prev, action))
  ];
}

function Counter() {
  const [state, dispatch] = useReducer(
    (state, action) => {
      switch (action.type) {
        case 'INCREMENT':
          return { ...state, count: state.count + 1 };
        case 'DECREMENT':
          return { ...state, count: state.count - 1 };
        default:
          return state;
      }
    },
    { count: 0 }
  );

  return (
    <div>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
    </div>
  );
}
```

### 6. **Fragment for Conditional Rendering**

```javascript
// ❌ Bad - unnecessary wrapper div
function Component() {
  return (
    <div>
      <Header />
      <Content />
      <Footer />
    </div>
  );
}

// ✅ Good - no extra DOM nodes
function Component() {
  return (
    <>
      <Header />
      <Content />
      <Footer />
    </>
  );
}
```

### 7. **Conditional Component Loading**

```javascript
// Load components only when needed
function App() {
  const [showSettings, setShowSettings] = useState(false);

  return (
    <div>
      <button onClick={() => setShowSettings(true)}>Settings</button>
      {showSettings && <Settings onClose={() => setShowSettings(false)} />}
    </div>
  );
}
```

### 8. **Context Performance**

```javascript
// ❌ Bad - all consumers re-render on any value change
const MyContext = React.createContext();

function Provider({ children }) {
  const [user, setUser] = useState(null);
  const [notifications, setNotifications] = useState([]);

  return (
    <MyContext.Provider value={{ user, notifications }}>
      {children}
    </MyContext.Provider>
  );
}

// ✅ Good - split context to prevent unnecessary re-renders
const UserContext = React.createContext();
const NotificationsContext = React.createContext();

function Provider({ children }) {
  const [user, setUser] = useState(null);
  const [notifications, setNotifications] = useState([]);

  return (
    <UserContext.Provider value={user}>
      <NotificationsContext.Provider value={notifications}>
        {children}
      </NotificationsContext.Provider>
    </UserContext.Provider>
  );
}
```

### 9. **Server-Side Rendering (SSR) for Initial Load**

```javascript
// Express server setup
import React from 'react';
import ReactDOMServer from 'react-dom/server';
import App from './App';

app.get('/', (req, res) => {
  const markup = ReactDOMServer.renderToString(<App />);
  res.send(`
    <!DOCTYPE html>
    <html>
      <head><title>My App</title></head>
      <body>
        <div id="root">${markup}</div>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
});
```

### 10. **Progressive Enhancement**

```javascript
// Load critical CSS first, defer non-critical
function App() {
  return (
    <>
      <link rel="stylesheet" href="/critical.css" />
      <link rel="preload" href="/non-critical.css" as="style" onLoad={(e) => e.target.rel = 'stylesheet'} />
      <noscript><link rel="stylesheet" href="/non-critical.css" /></noscript>
      
      <main>
        <Header />
        <Content />
      </main>
    </>
  );
}
```

---

## Performance Checklist

- [ ] Use React DevTools Profiler to identify slow components
- [ ] Memoize expensive components with React.memo
- [ ] Use useMemo for heavy computations
- [ ] Use useCallback for callback dependencies
- [ ] Implement code splitting at routes and components
- [ ] Use lazy() and Suspense for dynamic imports
- [ ] Implement virtual lists for large datasets
- [ ] Optimize images with lazy loading and formats
- [ ] Monitor Web Vitals metrics
- [ ] Analyze bundle size regularly
- [ ] Avoid inline function definitions
- [ ] Split large contexts into smaller ones
- [ ] Use proper keys in lists
- [ ] Debounce/throttle event handlers
- [ ] Consider SSR for better initial load

---

## Resources

- [React Official Performance Documentation](https://react.dev/reference/react/memo)
- [Web Vitals](https://web.dev/vitals/)
- [React Profiler API](https://react.dev/reference/react/Profiler)
- [webpack Documentation](https://webpack.js.org/configuration/)
- [react-window for Virtual Lists](https://github.com/bvaughn/react-window)
