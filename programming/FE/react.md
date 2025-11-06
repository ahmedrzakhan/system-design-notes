# React Interview Questions & Answers

## 1. What is React and what problem does it solve?

**Q: Explain React**

A: React is a JavaScript library for building user interfaces using components. Created by Facebook (now Meta).

**Problem it solves**:

- DOM manipulation is slow and error-prone
- Keeping UI in sync with data is complex
- Building reusable UI components is difficult

**Key idea**: Declarative, component-based UI. You describe what the UI should look like, React handles updates.

```jsx
function Button() {
  return <button>Click me</button>;
}
```

Instead of manually updating DOM, you declare the UI and React manages changes efficiently.

---

## 2. What is JSX and how does it work?

**Q: What is JSX?**

A: JSX is syntax that looks like HTML but is actually JavaScript. Compiles to JavaScript function calls.

```jsx
// JSX
const element = <h1>Hello, World!</h1>;

// Compiles to
const element = React.createElement("h1", null, "Hello, World!");
```

**JSX rules**:

- Can embed JavaScript with `{}`
- Single root element
- className instead of class
- Attributes are camelCase (onClick, onChange, onSubmit)
- Must return element

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

**Why JSX?**

- More readable than `React.createElement()`
- Feels like HTML but it's JavaScript
- Compiler errors caught early

---

## 3. What is a component and what are the types?

**Q: Explain React components**

A: Component is a reusable piece of UI. Two types:

**Function Component** (modern, preferred):

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

**Class Component** (older):

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

**Difference**:

- Function components are simpler
- Class components have more features (but hooks provide same features now)
- Hooks only work in function components
- Most new code uses function components

**Best practice**: Use function components with hooks.

---

## 4. What are props?

**Q: Explain props in React**

A: Props are arguments passed to components. Read-only way to pass data down.

```jsx
function Greeting({ name, age }) {
  return (
    <p>
      {name} is {age} years old
    </p>
  );
}

// Usage
<Greeting name="John" age={30} />;
```

**Key points**:

- Props are immutable (read-only)
- Passed from parent to child
- Used to customize components
- Can be any type (string, number, object, function, etc.)

```jsx
function Button({ onClick, disabled, children }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

<Button onClick={handleClick} disabled={false}>
  Click me
</Button>;
```

**Prop drilling**: Passing props through many levels is tedious. Use Context API for deeply nested props.

---

## 5. What is state and how is it different from props?

**Q: State vs Props**

A:

| Props                 | State                    |
| --------------------- | ------------------------ |
| Passed from parent    | Local to component       |
| Immutable             | Mutable (via setState)   |
| Used to configure     | Used for dynamic data    |
| Received as parameter | Managed inside component |

```jsx
function Counter() {
  const [count, setCount] = useState(0); // State

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// count is STATE (internal)
// Passed to child as PROPS
<Counter />;
```

**Rule**: Update state → React re-renders → UI updates.

---

## 6. What are React Hooks?

**Q: Explain hooks and common hooks**

A: Hooks are functions that let you use state and other features in function components. They "hook into" React features.

**useState** — manage state:

```jsx
const [count, setCount] = useState(0);
```

**useEffect** — side effects (fetching data, subscriptions):

```jsx
useEffect(() => {
  // Runs after every render
  document.title = `Count: ${count}`;
}, [count]); // Dependency array
```

**useContext** — consume context:

```jsx
const theme = useContext(ThemeContext);
```

**useReducer** — complex state logic:

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
dispatch({ type: "INCREMENT" });
```

**useCallback** — memoize function:

```jsx
const memoizedCallback = useCallback(() => doSomething(a, b), [a, b]);
```

**useMemo** — memoize value:

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

**useRef** — access DOM directly:

```jsx
const inputRef = useRef(null);
inputRef.current.focus();
```

**Custom hooks** — reuse logic:

```jsx
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);
  return size;
}
```

---

## 7. What is the Virtual DOM?

**Q: Explain Virtual DOM**

A: Virtual DOM is a lightweight JavaScript representation of the real DOM. React uses it for efficient updates.

**How it works**:

1. React renders component to Virtual DOM
2. Compares new Virtual DOM with old (diffing)
3. Calculates minimal changes needed
4. Updates only changed parts of real DOM (reconciliation)

```jsx
// React creates Virtual DOM
function App() {
  return <div><p>Hello</p></div>;
}

// Virtual DOM representation
{
  type: 'div',
  props: {},
  children: [
    { type: 'p', props: {}, children: ['Hello'] }
  ]
}
```

**Why it matters**:

- Real DOM updates are slow
- Virtual DOM updates are fast (just JavaScript objects)
- Batch updates for better performance
- You don't touch DOM directly

---

## 8. What is reconciliation and the key prop?

**Q: Explain reconciliation and key**

A: Reconciliation is React's algorithm for updating DOM efficiently. The `key` prop helps it identify which items changed.

**Without key** — can cause issues:

```jsx
function List({ items }) {
  return items.map((item) => <div>{item}</div>);
}
```

If you reorder items, React doesn't know they moved. Can cause state bugs.

**With key** — React knows which item is which:

```jsx
function List({ items }) {
  return items.map((item) => <div key={item.id}>{item}</div>);
}
```

**Key rules**:

- Use unique identifier (id, not index)
- Don't use array index as key (breaks if list reorders)
- Key must be unique among siblings

```jsx
// BAD — index as key
items.map((item, index) => <div key={index}>{item}</div>);

// GOOD — unique id
items.map((item) => <div key={item.id}>{item}</div>);
```

---

## 9. What is the dependency array in useEffect?

**Q: Explain useEffect dependency array**

A: Dependency array controls when effect runs.

**No dependency array** — runs after every render:

```jsx
useEffect(() => {
  console.log("Runs after every render");
});
```

**Empty dependency array** — runs once on mount:

```jsx
useEffect(() => {
  console.log("Runs once on mount");

  return () => console.log("Cleanup on unmount");
}, []);
```

**With dependencies** — runs when dependencies change:

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  console.log("Runs when count changes");
}, [count]);
```

**Cleanup function** — runs before effect re-runs or component unmounts:

```jsx
useEffect(() => {
  const subscription = subscribe();

  return () => subscription.unsubscribe(); // Cleanup
}, []);
```

**Common mistake** — forgetting dependencies:

```jsx
// BAD — effect runs infinitely if fetchData changes
useEffect(() => {
  fetchData();
}, []); // Should include fetchData

// BETTER — use useCallback to memoize fetchData
const fetchData = useCallback(() => {
  // fetch
}, []);

useEffect(() => {
  fetchData();
}, [fetchData]);
```

---

## 10. What is the Context API?

**Q: Explain Context API and when to use it**

A: Context API provides way to pass data through component tree without prop drilling.

```jsx
const ThemeContext = React.createContext();

function App() {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <Main />
      <Footer />
    </ThemeContext.Provider>
  );
}

function Header() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      Toggle Theme
    </button>
  );
}
```

**When to use**:

- Global state (theme, language, user auth)
- Avoid prop drilling
- Not for frequent changes (performance issue)

**When not to use**:

- Frequently changing data (use Redux/Zustand)
- Performance-critical apps (Context causes re-renders)

---

## 11. What is React.memo and when to use it?

**Q: Explain React.memo**

A: `React.memo` memoizes component to prevent re-renders if props haven't changed.

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>;
}

export default React.memo(Greeting);
```

Without memo, if parent re-renders, Greeting re-renders even if `name` prop unchanged.

With memo, Greeting only re-renders if `name` changes.

**Custom comparison**:

```jsx
const Greeting = React.memo(
  ({ name, age }) => (
    <h1>
      {name}, {age}
    </h1>
  ),
  (prevProps, nextProps) => {
    // Return true if props are equal (don't re-render)
    // Return false if props differ (re-render)
    return prevProps.name === nextProps.name;
  }
);
```

**When to use**:

- Component receives same props frequently
- Component is expensive to render
- Parent re-renders often

**Don't overuse** — adds overhead. Premature optimization is bad.

---

## 12. What is useMemo and useCallback?

**Q: Explain useMemo and useCallback**

A: Both memoize values/functions to avoid recalculating on every render.

**useMemo** — memoizes computed value:

```jsx
function ExpensiveComponent({ a, b }) {
  const expensiveValue = useMemo(() => {
    return expensiveCalculation(a, b);
  }, [a, b]); // Only recalculate when a or b change

  return <div>{expensiveValue}</div>;
}
```

**useCallback** — memoizes function:

```jsx
function Parent() {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []); // Function doesn't change

  return <Child onClick={handleClick} />;
}
```

Without useCallback, new function created every render, causing Child to re-render even with React.memo.

**When to use**:

- Passing function as prop to memoized component
- Expensive computations
- Function in useEffect dependency

**Don't overuse** — memoization has cost too.

---

## 13. What is controlled vs uncontrolled components?

**Q: Controlled vs uncontrolled form elements**

A:

**Controlled** — React manages form value:

```jsx
function Form() {
  const [name, setName] = useState("");

  const handleChange = (e) => setName(e.target.value);

  return <input value={name} onChange={handleChange} />;
}
```

React state is source of truth.

**Uncontrolled** — DOM manages form value:

```jsx
function Form() {
  const inputRef = useRef(null);

  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };

  return <input ref={inputRef} />;
}
```

DOM is source of truth.

**Use controlled** — most cases, easier to validate and sync.

**Use uncontrolled** — file inputs, integrating with non-React code.

---

## 14. What is lifting state up?

**Q: Explain lifting state up**

A: Moving state to parent component so multiple children can share it.

**Problem** — two components need to share state:

```jsx
function Parent() {
  return (
    <div>
      <InputComponent />
      <DisplayComponent />
    </div>
  );
}
```

**Solution** — lift state to parent:

```jsx
function Parent() {
  const [value, setValue] = useState("");

  return (
    <div>
      <InputComponent value={value} onChange={setValue} />
      <DisplayComponent value={value} />
    </div>
  );
}
```

Now both children can access and update the same state.

---

## 15. What is conditional rendering?

**Q: How do you conditionally render components?**

A: Show/hide components based on condition.

**if/else**:

```jsx
function Component({ isLoggedIn }) {
  if (isLoggedIn) {
    return <Dashboard />;
  }
  return <Login />;
}
```

**Ternary**:

```jsx
return isLoggedIn ? <Dashboard /> : <Login />;
```

**Logical &&** (only render if true):

```jsx
return isLoggedIn && <Dashboard />;
```

**switch**:

```jsx
switch (status) {
  case "loading":
    return <Loading />;
  case "error":
    return <Error />;
  default:
    return <Content />;
}
```

**Avoid rendering null or false** — they're falsy but still render:

```jsx
// BAD — renders nothing visible but element exists
{
  false && <Component />;
}

// Correct — truly doesn't render
{
  condition && <Component />;
}
```

---

## 16. What is event handling in React?

**Q: How do you handle events?**

A: Events are camelCase and receive synthetic event object.

```jsx
function Button() {
  const handleClick = (e) => {
    e.preventDefault();
    console.log("clicked");
  };

  return <button onClick={handleClick}>Click</button>;
}
```

**Event types**:

```jsx
// Mouse events
<button onClick={handleClick} />
<div onMouseEnter={handleHover} />
<div onMouseLeave={handleLeave} />

// Form events
<input onChange={handleChange} />
<form onSubmit={handleSubmit} />

// Keyboard events
<input onKeyDown={handleKeyDown} />
<input onKeyUp={handleKeyUp} />

// Focus events
<input onFocus={handleFocus} />
<input onBlur={handleBlur} />
```

**Synthetic events** — React wraps native events for cross-browser compatibility.

**Common mistake** — calling function instead of passing it:

```jsx
// BAD — calls function immediately
<button onClick={handleClick()}>Click</button>

// GOOD — passes function
<button onClick={handleClick}>Click</button>

// With arguments
<button onClick={() => handleClick(arg)}>Click</button>
```

---

## 17. What is the difference between a class component and function component lifecycle?

**Q: Component lifecycle differences**

A:

**Function component** (with hooks):

```jsx
function Component() {
  // componentDidMount
  useEffect(() => {
    console.log("Mounted");
    return () => console.log("Unmounted");
  }, []);

  // componentDidUpdate
  useEffect(() => {
    console.log("Updated");
  });

  return <div>Component</div>;
}
```

**Class component**:

```jsx
class Component extends React.Component {
  componentDidMount() {
    console.log("Mounted");
  }

  componentDidUpdate() {
    console.log("Updated");
  }

  componentWillUnmount() {
    console.log("Unmounted");
  }

  render() {
    return <div>Component</div>;
  }
}
```

**Lifecycle phases**:

1. **Mount** — component created and inserted into DOM
2. **Update** — component re-rendered due to state/prop change
3. **Unmount** — component removed from DOM

**Common pattern**:

```jsx
// Fetch data on mount
useEffect(() => {
  fetchData();
}, []);

// Cleanup subscription on unmount
useEffect(() => {
  const subscription = subscribe();
  return () => subscription.unsubscribe();
}, []);
```

---

## 18. What is lazy loading and code splitting?

**Q: Explain code splitting and lazy loading**

A: Load code only when needed instead of everything upfront.

**React.lazy** — dynamic import:

```jsx
const HeavyComponent = React.lazy(() => import("./HeavyComponent"));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

When HeavyComponent is needed, its code is loaded. Until loaded, fallback shows.

**Route-based code splitting**:

```jsx
const Dashboard = React.lazy(() => import("./Dashboard"));
const Settings = React.lazy(() => import("./Settings"));

function App() {
  return (
    <Router>
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </Router>
  );
}
```

**Benefits**:

- Smaller initial bundle
- Faster page load
- Load features on demand

---

## 19. What is error boundary?

**Q: Explain error boundary**

A: Component that catches errors in children and displays fallback UI.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.log("Error caught:", error);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong</h1>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>;
```

**Important**: Only works for errors in render/lifecycle. Not for:

- Event handlers (use try/catch)
- Async code
- Server-side rendering

---

## 20. What is strict mode?

**Q: What is React.StrictMode?**

A: Highlights potential problems in app during development.

```jsx
<React.StrictMode>
  <App />
</React.StrictMode>
```

**What it does**:

- Runs effects twice (to find impure functions)
- Warns about deprecated APIs
- Identifies components without keys
- Logs warnings for legacy APIs

**Only in development** — no performance impact in production.

---

## 21. What is fragment?

**Q: What is fragment?**

A: Wrapper that doesn't create DOM node.

```jsx
// Without fragment — creates extra div
function Component() {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
}

// With fragment — no extra node
function Component() {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
}

// Or explicit
return <React.Fragment>...</React.Fragment>;
```

**When to use**:

- Return multiple elements without wrapper
- When wrapper div causes CSS issues (flexbox, grid)
- Semantic HTML (no extra divs)

---

## 22. What is refs and when to use them?

**Q: Explain refs**

A: Refs provide way to access DOM nodes directly. Usually avoid.

```jsx
function TextInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

**When to use**:

- Focus, text selection, media playback
- Trigger animations
- Integrate with third-party DOM libraries

**Don't use for**:

- Anything that can be declared (use props)
- Most data updates (use state)

**Class component refs**:

```jsx
class Component extends React.Component {
  constructor(props) {
    super(props);
    this.ref = React.createRef();
  }

  render() {
    return <input ref={this.ref} />;
  }
}
```

---

## 23. What is prop drilling and how to avoid it?

**Q: What is prop drilling?**

A: Passing props through many levels of components that don't need them.

```jsx
// Prop drilling — data passed through intermediary components
function App() {
  const [user, setUser] = useState({});
  return <Level1 user={user} />;
}

function Level1({ user }) {
  return <Level2 user={user} />;
}

function Level2({ user }) {
  return <Level3 user={user} />;
}

function Level3({ user }) {
  return <p>{user.name}</p>;
}
```

**Problem** — tedious, hard to maintain, hard to know where data comes from.

**Solution 1 — Context API**:

```jsx
const UserContext = React.createContext();

function App() {
  const [user, setUser] = useState({});
  return (
    <UserContext.Provider value={user}>
      <Level1 />
    </UserContext.Provider>
  );
}

function Level3() {
  const user = useContext(UserContext);
  return <p>{user.name}</p>;
}
```

**Solution 2 — State management** (Redux, Zustand, etc.)

---

## 24. What is HOC (Higher Order Component)?

**Q: Explain Higher Order Components**

A: Function that takes component and returns enhanced component.

```jsx
function withLogger(Component) {
  return function LoggerComponent(props) {
    useEffect(() => {
      console.log("Component mounted");
      return () => console.log("Component unmounted");
    }, []);

    return <Component {...props} />;
  };
}

function MyComponent() {
  return <h1>Hello</h1>;
}

export default withLogger(MyComponent);
```

**Common use cases**:

- Add props to component
- Abstract logic
- Render props alternative
- Access context without hooks

**Render props alternative**:

```jsx
function Component() {
  const [count, setCount] = useState(0);
  return (
    <div>
      {/* Render prop */}
      {/* Now simpler with custom hooks */}
    </div>
  );
}
```

---

## 25. What is the difference between key and index?

**Q: Why not use index as key?**

A: Using index breaks if list reorders.

```jsx
// BAD — index as key
function List({ items }) {
  return items.map((item, index) => <div key={index}>{item.name}</div>);
}
```

If list reorders:

- Item A (index 0) → index 1
- Item B (index 1) → index 0
- React thinks items swapped, not reordered

Can cause:

- State loss
- Input value issues
- Animation problems

**GOOD** — unique identifier:

```jsx
items.map((item) => <div key={item.id}>{item.name}</div>);
```

---

## 26. What is functional composition?

**Q: Explain functional composition**

A: Building complex UIs from simple components.

```jsx
function Button({ children, onClick }) {
  return <button onClick={onClick}>{children}</button>;
}

function Input({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}

function Form() {
  const [name, setName] = useState("");

  return (
    <div>
      <Input value={name} onChange={(e) => setName(e.target.value)} />
      <Button onClick={() => console.log(name)}>Submit</Button>
    </div>
  );
}
```

Components are functions. Composition is combining them.

---

## 27. What is pure component?

**Q: Pure component and PureComponent**

A: Component that always renders same output for same props and state.

```jsx
// Pure — same input = same output
function Pure({ name }) {
  return <h1>Hello, {name}</h1>;
}

// Not pure — depends on external state
let counter = 0;
function Impure({ name }) {
  return (
    <h1>
      Hello, {name} #{counter++}
    </h1>
  );
}
```

**PureComponent** (class):

```jsx
class Pure extends React.PureComponent {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

PureComponent does shallow comparison of props/state.

Function component equivalent:

```jsx
const Pure = React.memo(({ name }) => {
  return <h1>Hello, {name}</h1>;
});
```

---

## 28. What is uncontrolled component?

**Q: Uncontrolled component example**

A: Component where form data handled by DOM, not React.

```jsx
function FileInput() {
  const fileRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    const file = fileRef.current.files[0];
    console.log(file);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="file" ref={fileRef} />
      <button>Upload</button>
    </form>
  );
}
```

**When to use**:

- File inputs (can't set value)
- Integrating with non-React code
- One-off forms

**Controlled is usually better** — React state as source of truth.

---

## 29. What is rendering in React?

**Q: How does React rendering work?**

A: Rendering is process of producing output (virtual DOM).

**Phases**:

1. **Render phase** — React calculates what changed (can pause/abort)
2. **Commit phase** — React applies changes to DOM (can't stop)

```jsx
// Triggers re-render
function App() {
  const [count, setCount] = useState(0);

  // This function body is the "render phase"
  console.log("Rendering"); // Runs often

  useEffect(() => {
    // This is "commit phase" (side effects)
    document.title = `Count: ${count}`;
  }, [count]);

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**What triggers re-render**:

- State change (via setState)
- Props change
- Parent re-render
- Context value change

**Optimization**:

- React.memo to skip re-render
- useCallback to memoize functions
- useMemo to memoize values
- Keys to prevent unnecessary re-renders

---

## 30. What is Fiber?

**Q: What is React Fiber?**

A: Internal architecture of React. Fiber is unit of work.

**Before Fiber** — React rendered synchronously. If large component tree, would block main thread.

**After Fiber** — React can:

- Break work into small units (fibers)
- Pause and resume work
- Assign priority to work
- Reuse previously completed work

**Benefits**:

- Non-blocking rendering
- Better performance on slower devices
- Concurrent features (Suspense, transitions)

You don't need to understand internals, but good to know:

- React doesn't render everything at once
- Can pause for high-priority updates (user input)
- Long renders don't freeze UI

---

## React Interview Tips

1. **Hooks are key** — Know useState, useEffect, useContext deeply
2. **Think in components** — Break UI into reusable pieces
3. **Know when to optimize** — Don't over-memoize
4. **Understand virtual DOM** — Know reconciliation basics
5. **Data flow** — Props down, state up, events up
6. **Performance** — Keys, memoization, code splitting
7. **Testing** — Be ready for testing questions
8. **Production experience** — Talk about real projects
