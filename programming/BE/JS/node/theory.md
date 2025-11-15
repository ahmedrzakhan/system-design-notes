# Node.js Interview Preparation Guide

## 1. Event Loop & Asynchronous Model

### Understanding the Event Loop

**Single-Threaded Model:**

- Node.js runs on a single JavaScript thread
- Uses an event-driven, non-blocking I/O model
- Handles thousands of concurrent connections efficiently
- Doesn't block on I/O operations

**Event Loop Phases (libuv implementation):**

1. **Timers** - Execute callbacks scheduled by setTimeout/setInterval
2. **Pending Callbacks** - Execute system callbacks deferred to next loop
3. **Idle, Prepare** - Internal operations
4. **Poll** - Retrieve new I/O events, execute I/O callbacks (except timers, close callbacks, setImmediate)
5. **Check** - Execute setImmediate callbacks
6. **Close Callbacks** - Execute close event callbacks

```
┌───────────────────────────────┐
│           timers              │ - setTimeout, setInterval
├───────────────────────────────┤
│       pending callbacks       │ - Deferred I/O callbacks
├───────────────────────────────┤
│       idle, prepare           │ - Internal
├───────────────────────────────┤
│          poll                 │ - I/O operations (fs, net)
├───────────────────────────────┤
│          check                │ - setImmediate
├───────────────────────────────┤
│     close callbacks           │ - socket.destroy(), stream.close()
└───────────────────────────────┘
```

**Key Points:**

- JavaScript code execution happens synchronously
- Once the call stack is empty, the event loop checks for callbacks
- Can process multiple callbacks per phase
- May pause at poll phase if no callbacks

### Microtasks vs Macrotasks

**Microtasks (Higher Priority):**

- Promise callbacks (.then, .catch, .finally)
- queueMicrotask()
- Executed after synchronous code, before next phase
- All microtasks drain before moving to next phase

**Macrotasks (Lower Priority):**

- setTimeout/setInterval
- setImmediate
- I/O operations
- One macrotask per event loop phase iteration

```javascript
console.log("Start");

setTimeout(() => console.log("setTimeout"), 0);
Promise.resolve().then(() => console.log("Promise"));
setImmediate(() => console.log("setImmediate"));

console.log("End");

// Output:
// Start
// End
// Promise (microtask - drains before next phase)
// setTimeout (macrotask - timers phase)
// setImmediate (macrotask - check phase)
```

**Key Interview Point:** Understand microtask queue drains completely before moving to next macrotask phase.

### process.nextTick()

```javascript
console.log("Start");

process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("Promise"));
setImmediate(() => console.log("setImmediate"));

console.log("End");

// Output:
// Start
// End
// nextTick (highest priority after current phase)
// Promise (microtask)
// setImmediate (check phase)
```

**Note:** process.nextTick() is NOT part of event loop phases; it executes after current phase completes but before next phase starts. It's technically a microtask but processed before all other microtasks.

---

## 2. Callbacks, Promises & Async/Await

### Callbacks (Callback Hell / Pyramid of Doom)

```javascript
// ❌ Callback Hell
fs.readFile("file1.txt", "utf8", (err, data1) => {
  if (err) throw err;
  fs.readFile("file2.txt", "utf8", (err, data2) => {
    if (err) throw err;
    fs.readFile("file3.txt", "utf8", (err, data3) => {
      if (err) throw err;
      console.log(data1, data2, data3);
    });
  });
});
```

**Problems:**

- Hard to read and maintain
- Error handling scattered throughout
- Easy to make mistakes
- Difficult to debug

### Promises

```javascript
// Creating promises
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("Success!"), 1000);
});

// Consuming promises
promise
  .then((result) => console.log(result))
  .catch((error) => console.error(error))
  .finally(() => console.log("Done"));

// ✅ Promise chaining (better than callbacks)
fs.promises
  .readFile("file1.txt", "utf8")
  .then((data1) => fs.promises.readFile("file2.txt", "utf8"))
  .then((data2) => fs.promises.readFile("file3.txt", "utf8"))
  .then((data3) => console.log(data1, data2, data3))
  .catch((err) => console.error(err));
```

**Promise States:**

- **Pending** - Initial state
- **Fulfilled** - resolve() called
- **Rejected** - reject() called
- Once settled (fulfilled or rejected), cannot change state

```javascript
// Promise.all - Parallel, all must succeed
Promise.all([promise1, promise2, promise3])
  .then(([result1, result2, result3]) => {
    /* all succeeded */
  })
  .catch((error) => {
    /* if any failed */
  });

// Promise.race - Returns first settled
Promise.race([promise1, promise2]).then((result) => {
  /* first to resolve */
});

// Promise.allSettled - All complete, don't care about failures
Promise.allSettled([promise1, promise2]).then((results) => {
  /* all settled */
});

// Promise.any - First to fulfill (ignores rejections)
Promise.any([promise1, promise2]).then((result) => {
  /* first fulfilled */
});
```

### Async/Await

```javascript
// ✅ Async/await (cleanest syntax)
async function readFiles() {
  try {
    const data1 = await fs.promises.readFile("file1.txt", "utf8");
    const data2 = await fs.promises.readFile("file2.txt", "utf8");
    const data3 = await fs.promises.readFile("file3.txt", "utf8");
    console.log(data1, data2, data3);
  } catch (err) {
    console.error(err);
  }
}

// Async function always returns a Promise
readFiles().then(() => console.log("Done"));

// Parallel execution with await
async function parallel() {
  const [data1, data2, data3] = await Promise.all([
    fs.promises.readFile("file1.txt", "utf8"),
    fs.promises.readFile("file2.txt", "utf8"),
    fs.promises.readFile("file3.txt", "utf8"),
  ]);
}

// Error handling with async/await
async function withErrorHandling() {
  try {
    const result = await riskyOperation();
  } catch (error) {
    console.error("Caught error:", error);
  } finally {
    console.log("Cleanup");
  }
}
```

**Key Interview Points:**

- Async functions always return Promises
- Await can only be used in async functions
- Use Promise.all() for parallel operations, not sequential awaits
- Async/await is syntactic sugar over Promises

---

## 3. Callbacks & Node.js Conventions

### Node.js Callback Style (Error-First)

```javascript
// Standard Node.js callback convention
fs.readFile("file.txt", "utf8", (err, data) => {
  if (err) {
    console.error("Error reading file:", err);
    return;
  }
  console.log("File contents:", data);
});

// Custom function following Node convention
function fetchUser(userId, callback) {
  if (!userId) {
    callback(new Error("User ID required"));
    return;
  }

  // Simulate async operation
  setTimeout(() => {
    callback(null, { id: userId, name: "Alice" });
  }, 100);
}

// Usage
fetchUser(1, (err, user) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(user);
});
```

### Promisify Callbacks

```javascript
const { promisify } = require("util");

// Convert callback-based function to promise
const fetchUserAsync = promisify(fetchUser);

// Now it returns a Promise
fetchUserAsync(1)
  .then((user) => console.log(user))
  .catch((err) => console.error(err));

// Manual promisification
function promisifyReadFile(filename) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, "utf8", (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}
```

---

## 4. Streams & Buffers

### What are Streams?

Streams are collections of data that may not be available all at once and don't have to fit in memory.

**Types of Streams:**

- **Readable** - Read data (e.g., fs.createReadStream)
- **Writable** - Write data (e.g., fs.createWriteStream)
- **Duplex** - Both readable and writable (e.g., net.Socket)
- **Transform** - Modify data as it passes (e.g., zlib.createGzip)

### Readable Streams

```javascript
// Reading file in chunks instead of loading entire file
const readStream = fs.createReadStream("large-file.txt", {
  highWaterMark: 16 * 1024, // 16KB chunks
});

readStream.on("data", (chunk) => {
  console.log("Received chunk of size:", chunk.length);
});

readStream.on("end", () => {
  console.log("Stream ended");
});

readStream.on("error", (err) => {
  console.error("Stream error:", err);
});

// Pausing and resuming
readStream.on("data", (chunk) => {
  console.log("Processing chunk");
  readStream.pause();

  setTimeout(() => {
    readStream.resume();
  }, 1000);
});
```

### Writable Streams

```javascript
const writeStream = fs.createWriteStream("output.txt");

writeStream.write("Hello ");
writeStream.write("World");
writeStream.end(); // Signals no more data

writeStream.on("finish", () => {
  console.log("Writing finished");
});

writeStream.on("error", (err) => {
  console.error("Write error:", err);
});

// Handling backpressure
const readStream = fs.createReadStream("input.txt");
const writeStream = fs.createWriteStream("output.txt");

readStream.on("data", (chunk) => {
  const canContinue = writeStream.write(chunk);

  if (!canContinue) {
    console.log("Backpressure! Pausing read");
    readStream.pause();
  }
});

writeStream.on("drain", () => {
  console.log("Drain event! Resuming read");
  readStream.resume();
});
```

### Pipe & Pipeline

```javascript
// Simple pipe (handles backpressure automatically)
fs.createReadStream("input.txt").pipe(fs.createWriteStream("output.txt"));

// Multiple pipes (for transformation)
fs.createReadStream("input.txt")
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream("input.txt.gz"));

// Pipeline (better error handling - Node 10+)
const { pipeline } = require("stream");

pipeline(
  fs.createReadStream("input.txt"),
  zlib.createGzip(),
  fs.createWriteStream("input.txt.gz"),
  (err) => {
    if (err) console.error("Pipeline failed", err);
    else console.log("Pipeline succeeded");
  }
);

// Or with promises (Node 15+)
const { pipeline } = require("stream/promises");
await pipeline(
  fs.createReadStream("input.txt"),
  zlib.createGzip(),
  fs.createWriteStream("input.txt.gz")
);
```

### Transform Streams

```javascript
const { Transform } = require("stream");

// Create custom transform stream
const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback(); // Signal completion
  },
});

// Use it
fs.createReadStream("input.txt")
  .pipe(upperCaseTransform)
  .pipe(fs.createWriteStream("output.txt"));

// Transform with error
const validateTransform = new Transform({
  transform(chunk, encoding, callback) {
    try {
      const data = JSON.parse(chunk);
      this.push(JSON.stringify(data));
      callback();
    } catch (err) {
      callback(err); // Pass error
    }
  },
});
```

### Buffers

```javascript
// Creating buffers
const buf1 = Buffer.alloc(10); // Allocate 10 bytes (zeroed)
const buf2 = Buffer.from("hello"); // From string
const buf3 = Buffer.from([1, 2, 3]); // From array

// Buffer operations
buf1.write("hello");
console.log(buf1.toString());

// Concatenating buffers
const combined = Buffer.concat([buf2, buf3]);

// Accessing buffer data
for (let i = 0; i < buf2.length; i++) {
  console.log(buf2[i]); // Get byte at index
}

// Buffer is memory-efficient
const largeBuffer = Buffer.alloc(1000000); // 1MB allocated
console.log(largeBuffer.length); // 1000000 bytes
```

**Key Interview Point:** Streams are memory-efficient for large files; use pipes to handle backpressure automatically.

---

## 5. Module System & Require/Import

### CommonJS (require/module.exports)

```javascript
// math.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = {
  add,
  subtract,
};

// app.js
const math = require("./math");
console.log(math.add(5, 3));

// Or destructure
const { add, subtract } = require("./math");

// Built-in modules
const fs = require("fs");
const path = require("path");
```

### ES6 Modules (import/export)

```javascript
// math.js (with .mjs extension or "type": "module" in package.json)
export function add(a, b) {
  return a + b;
}

export const PI = 3.14159;

export default class Calculator {
  // ...
}

// app.js
import { add, PI } from "./math.js";
import Calculator from "./math.js";

console.log(add(5, 3));
```

### Module Caching

```javascript
// Node.js caches modules
const math1 = require("./math");
const math2 = require("./math");

console.log(math1 === math2); // true - same object

// For class instances (not good practice)
const connection1 = require("./db");
const connection2 = require("./db");

// Same connection instance - can share state (good for singletons)
```

---

## 6. Callbacks vs Promises vs Async/Await Comparison

### Sequential Operations

```javascript
// ❌ Callbacks
setTimeout(() => {
  console.log("Step 1");
  setTimeout(() => {
    console.log("Step 2");
    setTimeout(() => {
      console.log("Step 3");
    }, 100);
  }, 100);
}, 100);

// ⚠️ Promises
Promise.resolve()
  .then(() => {
    console.log("Step 1");
    return new Promise((resolve) => setTimeout(resolve, 100));
  })
  .then(() => {
    console.log("Step 2");
    return new Promise((resolve) => setTimeout(resolve, 100));
  })
  .then(() => {
    console.log("Step 3");
  });

// ✅ Async/Await
async function steps() {
  await new Promise((resolve) => setTimeout(resolve, 100));
  console.log("Step 1");

  await new Promise((resolve) => setTimeout(resolve, 100));
  console.log("Step 2");

  await new Promise((resolve) => setTimeout(resolve, 100));
  console.log("Step 3");
}

steps();
```

### Error Handling Comparison

```javascript
// ❌ Callbacks - error handling scattered
function operation(callback) {
  doSomething((err, result) => {
    if (err) {
      callback(err);
      return;
    }
    doAnother(result, (err, result2) => {
      if (err) {
        callback(err);
        return;
      }
      callback(null, result2);
    });
  });
}

// ⚠️ Promises - better, but verbose
Promise.resolve()
  .then(() => doSomething())
  .then((result) => doAnother(result))
  .catch((err) => console.error(err));

// ✅ Async/Await - cleanest
async function operation() {
  try {
    const result = await doSomething();
    const result2 = await doAnother(result);
    return result2;
  } catch (err) {
    console.error(err);
  }
}
```

---

## 7. Memory Management & Garbage Collection

### Memory in Node.js

```javascript
// Heap memory
const arr = [];
for (let i = 0; i < 1000000; i++) {
  arr.push({ id: i, name: `User ${i}` }); // Objects allocated on heap
}

// Stack memory (local variables)
function calculate() {
  const x = 10; // Stack
  const y = 20; // Stack
  return x + y;
}

// Check memory usage
const memUsage = process.memoryUsage();
console.log("Heap Used:", memUsage.heapUsed / 1024 / 1024, "MB");
console.log("Heap Total:", memUsage.heapTotal / 1024 / 1024, "MB");
```

### Memory Leaks

```javascript
// ❌ Memory leak - closure retains reference
function createLeakyCache() {
  const cache = {};
  return function (key, value) {
    cache[key] = value; // Grows unbounded, never cleaned
    return cache;
  };
}

// ❌ Memory leak - event listener not removed
const emitter = new EventEmitter();
emitter.on("data", (data) => {
  console.log(data);
});
// Never calls emitter.off() - listener stays in memory

// ✅ Fix - Limit cache size
function createLimitedCache(maxSize) {
  const cache = {};
  let size = 0;

  return function (key, value) {
    if (size >= maxSize) {
      const firstKey = Object.keys(cache)[0];
      delete cache[firstKey];
      size--;
    }
    cache[key] = value;
    size++;
    return cache;
  };
}

// ✅ Fix - Remove listeners
const handler = (data) => console.log(data);
emitter.on("data", handler);
// Later:
emitter.off("data", handler);
```

### Garbage Collection

```javascript
// Forcing garbage collection (only with --expose-gc flag)
if (global.gc) {
  global.gc();
  console.log("GC triggered");
}

// Monitor GC (Node v15+)
const obs = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log("GC event:", entry);
  }
});

obs.observe({ entryTypes: ["gc"] });
```

---

## 8. Cluster & Child Processes

### Child Processes

```javascript
const { spawn, exec, fork } = require("child_process");

// spawn - Launch process (for long-running, streaming data)
const ls = spawn("ls", ["-la"]);

ls.stdout.on("data", (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on("data", (data) => {
  console.error(`stderr: ${data}`);
});

// exec - Execute command (shell command, collects output)
exec("ls -la", (error, stdout, stderr) => {
  if (error) {
    console.error(`Error: ${error.message}`);
    return;
  }
  console.log(`Output: ${stdout}`);
});

// fork - Spawn new Node.js process (IPC communication)
const child = fork("./child.js");
child.send({ hello: "world" });
child.on("message", (msg) => {
  console.log("Message from child:", msg);
});

// child.js
process.on("message", (msg) => {
  console.log("Message from parent:", msg);
  process.send({ reply: "hi from child" });
});
```

### Cluster Module

```javascript
const cluster = require("cluster");
const http = require("http");
const os = require("os");

if (cluster.isMaster) {
  const numWorkers = os.cpus().length;
  console.log(`Master process starting ${numWorkers} workers`);

  // Fork workers
  for (let i = 0; i < numWorkers; i++) {
    cluster.fork();
  }

  // Handle worker exits
  cluster.on("exit", (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Respawn
  });
} else {
  // Worker process
  http
    .createServer((req, res) => {
      res.writeHead(200);
      res.end(`Hello from worker ${process.pid}`);
    })
    .listen(3000);

  console.log(`Worker ${process.pid} started`);
}
```

---

## 9. Error Handling Best Practices

### Unhandled Promise Rejections

```javascript
// ❌ WRONG - Unhandled rejection
Promise.reject(new Error("Oops!"));

// ✅ CORRECT - Handle rejection
Promise.reject(new Error("Oops!")).catch((err) =>
  console.error("Handled:", err)
);

// Global handler for unhandled rejections
process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection at:", promise, "reason:", reason);
  // Log to file, alert, etc.
});

// Global handler for uncaught exceptions
process.on("uncaughtException", (error) => {
  console.error("Uncaught Exception:", error);
  process.exit(1); // Must exit after uncaughtException
});
```

### Custom Error Classes

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

class DatabaseError extends Error {
  constructor(message, code) {
    super(message);
    this.name = "DatabaseError";
    this.code = code;
  }
}

// Usage
try {
  throw new ValidationError("Invalid email", "email");
} catch (err) {
  if (err instanceof ValidationError) {
    console.log(`Validation failed on ${err.field}: ${err.message}`);
  }
}
```

---

## 10. Performance & Optimization

### Benchmarking

```javascript
// Simple timing
console.time("operation");
// Do something
console.timeEnd("operation");

// More precise
const start = process.hrtime.bigint();
// Operation
const end = process.hrtime.bigint();
const duration = Number(end - start) / 1000000; // Convert to ms
console.log(`Duration: ${duration}ms`);

// Using performance API
const perf = require("perf_hooks").performance;

perf.mark("start");
// Operation
perf.mark("end");
perf.measure("operation", "start", "end");

const measure = perf.getEntriesByName("operation")[0];
console.log(`Duration: ${measure.duration}ms`);
```

### Memory Profiling

```javascript
// Check memory before and after
const before = process.memoryUsage();
// Do something
const after = process.memoryUsage();

console.log(
  "Heap used increased by:",
  (after.heapUsed - before.heapUsed) / 1024 / 1024,
  "MB"
);

// V8 heap snapshot
const heapdump = require("heapdump");
heapdump.writeSnapshot(`./heapdump-${Date.now()}.heapsnapshot`);
```

### CPU Profiling

```javascript
// Using built-in inspector
const inspector = require("inspector");
const fs = require("fs");

const session = new inspector.Session();
session.connect();

session.post("Profiler.enable", () => {
  session.post("Profiler.start", () => {
    // Code to profile
    setTimeout(() => {
      session.post("Profiler.stop", (err, { profile }) => {
        // Write profile to file
        fs.writeFileSync("profile.cpuprofile", JSON.stringify(profile));
        session.disconnect();
      });
    }, 2000);
  });
});
```

---

## 11. Common Pitfalls

### Losing Context (this)

```javascript
// ❌ WRONG - 'this' is undefined
class Counter {
  count = 0;

  increment() {
    this.count++;
  }
}

const counter = new Counter();
const fn = counter.increment;
fn(); // TypeError: Cannot read property 'count' of undefined

// ✅ FIX 1 - Bind method
const boundFn = counter.increment.bind(counter);

// ✅ FIX 2 - Arrow function (lexical this)
class Counter {
  count = 0;

  increment = () => {
    this.count++;
  };
}

// ✅ FIX 3 - Call/Apply
counter.increment.call(counter);
```

### Race Conditions

```javascript
// ❌ WRONG - Race condition
let result;
setTimeout(() => {
  result = "done";
}, 100);
console.log(result); // undefined - timing issue

// ✅ CORRECT - Use promises/async
new Promise((resolve) => {
  setTimeout(() => resolve("done"), 100);
}).then((result) => console.log(result));
```

### Blocking Event Loop

```javascript
// ❌ WRONG - Blocking operation on event loop
function expensiveCalculation() {
  let sum = 0;
  for (let i = 0; i < 10000000000; i++) {
    sum += i;
  }
  return sum;
}

// Blocks the entire event loop
expensiveCalculation();

// ✅ FIX 1 - Use worker threads
const { Worker } = require("worker_threads");

const worker = new Worker("./worker.js");
worker.on("message", (result) => {
  console.log("Result:", result);
});

// ✅ FIX 2 - Use setImmediate to yield
function yieldingCalculation() {
  let sum = 0;
  let i = 0;

  return new Promise((resolve) => {
    function chunk() {
      const end = Math.min(i + 1000000, 10000000000);
      while (i < end) {
        sum += i++;
      }

      if (i < 10000000000) {
        setImmediate(chunk);
      } else {
        resolve(sum);
      }
    }
    chunk();
  });
}

yieldingCalculation().then((result) => console.log(result));
```

### Variable Scope in Loops

```javascript
// ❌ WRONG - Closure captures reference, not value
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i); // Prints 3, 3, 3
  }, 100);
}

// ✅ FIX 1 - Use let (block scope)
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i); // Prints 0, 1, 2
  }, 100);
}

// ✅ FIX 2 - Create closure with IIFE
for (var i = 0; i < 3; i++) {
  (function (j) {
    setTimeout(() => {
      console.log(j);
    }, 100);
  })(i);
}
```

### Unhandled Promise Rejections

```javascript
// ❌ WRONG - Rejected promise with no catch
async function fetchData() {
  throw new Error("Network error");
}

// Fire and forget - error is lost
fetchData();

// ✅ CORRECT - Always handle
fetchData().catch((err) => console.error(err));

// Or in concurrent code
const results = await Promise.allSettled([
  fetchData(),
  fetchData(),
  fetchData(),
]);
```

---

## 12. Testing

### Unit Testing with Jest

```javascript
// math.test.js
const { add, subtract } = require("./math");

describe("Math operations", () => {
  describe("add", () => {
    test("should add two positive numbers", () => {
      expect(add(2, 3)).toBe(5);
    });

    test("should handle negative numbers", () => {
      expect(add(-2, -3)).toBe(-5);
    });
  });

  describe("subtract", () => {
    test("should subtract two numbers", () => {
      expect(subtract(5, 3)).toBe(2);
    });
  });
});
```

### Async Testing

```javascript
test("should fetch user", async () => {
  const user = await fetchUser(1);
  expect(user.id).toBe(1);
});

test("should handle errors", async () => {
  await expect(fetchUser(-1)).rejects.toThrow();
});

// Test timeouts
test("should resolve within 1 second", async () => {
  const promise = new Promise((resolve) =>
    setTimeout(() => resolve("done"), 500)
  );
  await expect(promise).resolves.toBe("done");
}, 2000); // 2 second timeout
```

### Mocking

```javascript
const fs = require("fs");
jest.mock("fs");

test("should read file", () => {
  fs.readFile.mockImplementation((path, callback) => {
    callback(null, "file contents");
  });

  fs.readFile("test.txt", (err, data) => {
    expect(data).toBe("file contents");
  });
});
```

---

## 13. Common Patterns

### Observer Pattern (EventEmitter)

```javascript
const EventEmitter = require("events");

class DataEmitter extends EventEmitter {
  fetchData() {
    this.emit("start");

    setTimeout(() => {
      this.emit("data", { id: 1, name: "Alice" });
    }, 1000);

    setTimeout(() => {
      this.emit("end");
    }, 2000);
  }
}

const emitter = new DataEmitter();

emitter.on("start", () => console.log("Fetching..."));
emitter.on("data", (data) => console.log("Received:", data));
emitter.on("end", () => console.log("Complete"));

emitter.fetchData();
```

### Singleton Pattern

```javascript
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }

    this.connection = null;
    Database.instance = this;
  }

  connect() {
    this.connection = "connected";
  }
}

const db1 = new Database();
const db2 = new Database();

console.log(db1 === db2); // true
```

### Builder Pattern

```javascript
class QueryBuilder {
  constructor(table) {
    this.table = table;
    this.filters = [];
    this.limitValue = null;
  }

  where(field, value) {
    this.filters.push({ field, value });
    return this;
  }

  limit(n) {
    this.limitValue = n;
    return this;
  }

  build() {
    let query = `SELECT * FROM ${this.table}`;

    if (this.filters.length > 0) {
      query +=
        " WHERE " +
        this.filters.map((f) => `${f.field} = ${f.value}`).join(" AND ");
    }

    if (this.limitValue) {
      query += ` LIMIT ${this.limitValue}`;
    }

    return query;
  }
}

const query = new QueryBuilder("users")
  .where("age", 25)
  .where("city", "NYC")
  .limit(10)
  .build();

console.log(query);
```

---

## 14. Practice Questions

1. Explain the Node.js event loop and its phases
2. What's the difference between process.nextTick() and setImmediate()?
3. Explain microtasks vs macrotasks
4. What's the difference between callbacks, promises, and async/await?
5. How do streams help with memory efficiency?
6. Explain backpressure in streams
7. What's a memory leak in Node.js? How do you prevent them?
8. How does the cluster module work?
9. What's the difference between child_process.spawn() and fork()?
10. How do you handle unhandled promise rejections?
11. What are common Node.js anti-patterns?
12. How do you profile CPU and memory in Node.js?
13. Explain the difference between require() and import
14. What's the purpose of module caching?
15. How do you handle blocking operations in Node.js?

# Node.js Interview: Practical Code Examples

## 1. Event Loop & Execution Order

### Event Loop Phases Demo

```javascript
// Demonstrates the order of execution in event loop

console.log("1. Synchronous start");

// Macrotask: timers phase
setTimeout(() => {
  console.log("3. setTimeout (timers phase)");
}, 0);

// Macrotask: check phase
setImmediate(() => {
  console.log("4. setImmediate (check phase)");
});

// Microtask: promise callback
Promise.resolve().then(() => {
  console.log("2. Promise (microtask)");
});

// Microtask: process.nextTick (highest priority)
process.nextTick(() => {
  console.log("2a. process.nextTick (before other microtasks)");
});

console.log("1. Synchronous end");

// Expected output:
// 1. Synchronous start
// 1. Synchronous end
// 2a. process.nextTick (before other microtasks)
// 2. Promise (microtask)
// 3. setTimeout (timers phase)
// 4. setImmediate (check phase)
```

### Complex Event Loop Example

```javascript
console.log("Start");

// setTimeout creates macrotask in timers phase
const timer1 = setTimeout(() => {
  console.log("setTimeout 1");
  Promise.resolve().then(() => console.log("Promise in setTimeout 1"));
}, 0);

const timer2 = setTimeout(() => {
  console.log("setTimeout 2");
}, 0);

// Promise (microtask)
Promise.resolve()
  .then(() => {
    console.log("Promise 1");
    return "result";
  })
  .then((result) => {
    console.log("Promise 2:", result);
  });

// process.nextTick (executes after current phase, before other microtasks)
process.nextTick(() => {
  console.log("nextTick 1");
});

// setImmediate (check phase)
setImmediate(() => {
  console.log("setImmediate 1");
});

console.log("End");

// Expected output:
// Start
// End
// nextTick 1
// Promise 1
// Promise 2: result
// setTimeout 1
// Promise in setTimeout 1
// setTimeout 2
// setImmediate 1
```

---

## 2. Callbacks vs Promises vs Async/Await

### Comparing Three Approaches

```javascript
const fs = require("fs").promises;

// ❌ Callback Hell
function readFilesCallback() {
  fs.readFile("file1.txt", "utf8", (err, data1) => {
    if (err) throw err;
    console.log("File 1:", data1);

    fs.readFile("file2.txt", "utf8", (err, data2) => {
      if (err) throw err;
      console.log("File 2:", data2);

      fs.readFile("file3.txt", "utf8", (err, data3) => {
        if (err) throw err;
        console.log("File 3:", data3);
      });
    });
  });
}

// ⚠️ Promise Chain
function readFilesPromise() {
  return fs
    .readFile("file1.txt", "utf8")
    .then((data1) => {
      console.log("File 1:", data1);
      return fs.readFile("file2.txt", "utf8");
    })
    .then((data2) => {
      console.log("File 2:", data2);
      return fs.readFile("file3.txt", "utf8");
    })
    .then((data3) => {
      console.log("File 3:", data3);
    })
    .catch((err) => {
      console.error("Error:", err);
    });
}

// ✅ Async/Await
async function readFilesAsync() {
  try {
    const data1 = await fs.readFile("file1.txt", "utf8");
    console.log("File 1:", data1);

    const data2 = await fs.readFile("file2.txt", "utf8");
    console.log("File 2:", data2);

    const data3 = await fs.readFile("file3.txt", "utf8");
    console.log("File 3:", data3);
  } catch (err) {
    console.error("Error:", err);
  }
}
```

### Parallel vs Sequential

```javascript
// ❌ Sequential (slow - waits for each file)
async function slowSequential() {
  const start = Date.now();

  const data1 = await fs.readFile("file1.txt", "utf8");
  const data2 = await fs.readFile("file2.txt", "utf8");
  const data3 = await fs.readFile("file3.txt", "utf8");

  console.log("Sequential took:", Date.now() - start, "ms");
  return [data1, data2, data3];
}

// ✅ Parallel (fast - all at once with Promise.all)
async function fastParallel() {
  const start = Date.now();

  const [data1, data2, data3] = await Promise.all([
    fs.readFile("file1.txt", "utf8"),
    fs.readFile("file2.txt", "utf8"),
    fs.readFile("file3.txt", "utf8"),
  ]);

  console.log("Parallel took:", Date.now() - start, "ms");
  return [data1, data2, data3];
}

// Test
fastParallel().then(() => {
  console.log("Done");
});
```

---

## 3. Promise Methods

### Promise.all vs Promise.race vs Promise.allSettled

```javascript
const delay = (ms, value) =>
  new Promise((resolve) => setTimeout(() => resolve(value), ms));

const delay_err = (ms) =>
  new Promise((_, reject) => setTimeout(() => reject(new Error("Failed")), ms));

// Promise.all - All must succeed (rejects if any fails)
async function testAll() {
  try {
    const results = await Promise.all([
      delay(100, "A"),
      delay(200, "B"),
      delay(150, "C"),
    ]);
    console.log("All:", results); // ['A', 'B', 'C']
  } catch (err) {
    console.error("Failed:", err);
  }
}

// Promise.race - First to settle wins
async function testRace() {
  const result = await Promise.race([
    delay(100, "A"),
    delay(200, "B"),
    delay(50, "C"),
  ]);
  console.log("Race:", result); // 'C' (fastest)
}

// Promise.allSettled - All complete, returns status
async function testSettled() {
  const results = await Promise.allSettled([
    delay(100, "A"),
    delay_err(200),
    delay(150, "C"),
  ]);

  console.log("Settled:", results);
  // [
  //   { status: 'fulfilled', value: 'A' },
  //   { status: 'rejected', reason: Error('Failed') },
  //   { status: 'fulfilled', value: 'C' }
  // ]
}

// Promise.any - First to fulfill (ignores rejections)
async function testAny() {
  try {
    const result = await Promise.any([
      delay_err(100),
      delay(200, "B"),
      delay(150, "C"),
    ]);
    console.log("Any:", result); // 'B' (first fulfilled)
  } catch (err) {
    console.error("All rejected:", err);
  }
}
```

---

## 4. Streams & Buffers

### Reading Streams

```javascript
const fs = require("fs");

// Create readable stream
const stream = fs.createReadStream("large-file.txt", {
  encoding: "utf8",
  highWaterMark: 16 * 1024, // 16KB chunks
});

let chunks = 0;
let totalSize = 0;

stream.on("data", (chunk) => {
  chunks++;
  totalSize += chunk.length;
  console.log(`Chunk ${chunks}: ${chunk.length} bytes`);

  // Can pause stream
  // stream.pause();
});

stream.on("end", () => {
  console.log(`Received ${chunks} chunks, total: ${totalSize} bytes`);
});

stream.on("error", (err) => {
  console.error("Stream error:", err);
});
```

### Writing Streams & Backpressure

```javascript
// Writing to file using stream
const writeStream = fs.createWriteStream("output.txt");

// Simulate writing lots of data
for (let i = 0; i < 100000; i++) {
  const canContinue = writeStream.write(`Line ${i}\n`);

  if (!canContinue) {
    console.log("Backpressure! Pausing writes");
    break;
  }
}

// Listen for 'drain' event (buffer emptied)
writeStream.on("drain", () => {
  console.log("Drain! Can write more");
  // Resume writing...
});

writeStream.on("finish", () => {
  console.log("Write complete");
});

writeStream.end();
```

### Pipe (Automatic Backpressure Handling)

```javascript
// Reading from one file and writing to another
fs.createReadStream("input.txt").pipe(fs.createWriteStream("output.txt"));

// Piping through multiple streams (with compression)
const zlib = require("zlib");

fs.createReadStream("input.txt")
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream("input.txt.gz"))
  .on("finish", () => console.log("Compressed!"));

// Pipeline with better error handling (Node 10+)
const { pipeline } = require("stream");

pipeline(
  fs.createReadStream("input.txt"),
  zlib.createGzip(),
  fs.createWriteStream("input.txt.gz"),
  (err) => {
    if (err) {
      console.error("Pipeline failed:", err);
    } else {
      console.log("Pipeline succeeded");
    }
  }
);
```

### Transform Streams

```javascript
const { Transform } = require("stream");
const fs = require("fs");

// Custom transform: convert to uppercase
const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    const transformed = chunk.toString().toUpperCase();
    this.push(transformed);
    callback();
  },
});

fs.createReadStream("input.txt", "utf8")
  .pipe(upperCaseTransform)
  .pipe(fs.createWriteStream("output-uppercase.txt"));

// Transform with JSON parsing
const jsonTransform = new Transform({
  transform(chunk, encoding, callback) {
    try {
      const obj = JSON.parse(chunk.toString());
      obj.processed = true;
      this.push(JSON.stringify(obj) + "\n");
      callback();
    } catch (err) {
      callback(err);
    }
  },
});

// Usage with pipeline
const { pipeline } = require("stream/promises");

(async () => {
  try {
    await pipeline(
      fs.createReadStream("data.jsonl", "utf8"),
      jsonTransform,
      fs.createWriteStream("output.jsonl")
    );
    console.log("Transform pipeline succeeded");
  } catch (err) {
    console.error("Pipeline failed:", err);
  }
})();
```

### Buffer Operations

```javascript
// Creating buffers
const buf1 = Buffer.alloc(10); // 10 bytes, zeroed
const buf2 = Buffer.from("hello"); // From string
const buf3 = Buffer.from([72, 105]); // From bytes [Hi]

console.log(buf2.toString()); // 'hello'
console.log(buf3.toString()); // 'Hi'

// Buffer manipulation
buf1.write("hello");
console.log(buf1); // <Buffer 68 65 6c 6c 6f 00 00 00 00 00>

// Concatenating buffers
const combined = Buffer.concat([buf2, buf3]);
console.log(combined.toString()); // 'helloHi'

// Slicing buffers
const slice = buf2.slice(1, 4);
console.log(slice.toString()); // 'ell'

// Copying buffers
const dest = Buffer.alloc(5);
buf2.copy(dest);
console.log(dest.toString()); // 'hello'
```

---

## 5. Error Handling

### Promise Error Handling

```javascript
// Unhandled rejection (bad)
Promise.reject(new Error("Oops!"));

// Handled rejection
Promise.reject(new Error("Oops!")).catch((err) =>
  console.error("Caught:", err.message)
);

// Global handler for unhandled rejections
process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection:", reason);
  // Log to file, send alert, etc.
});

process.on("uncaughtException", (error) => {
  console.error("Uncaught Exception:", error);
  // Must exit after uncaughtException
  process.exit(1);
});
```

### Async/Await Error Handling

```javascript
async function operation() {
  try {
    const result = await riskyOperation();
    console.log("Success:", result);
  } catch (error) {
    console.error("Error caught:", error.message);

    // Can check error type
    if (error.code === "ENOENT") {
      console.error("File not found");
    }
  } finally {
    console.log("Cleanup regardless of outcome");
  }
}

// Parallel error handling
async function parallelOps() {
  const results = await Promise.allSettled([
    operation1(),
    operation2(),
    operation3(),
  ]);

  for (const result of results) {
    if (result.status === "rejected") {
      console.error("Operation failed:", result.reason);
    } else {
      console.log("Success:", result.value);
    }
  }
}
```

### Custom Errors

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
    this.statusCode = 400;
  }
}

class NotFoundError extends Error {
  constructor(resource) {
    super(`${resource} not found`);
    this.name = "NotFoundError";
    this.statusCode = 404;
  }
}

// Usage
async function createUser(data) {
  if (!data.email) {
    throw new ValidationError("Email is required", "email");
  }

  const existing = await User.findByEmail(data.email);
  if (existing) {
    throw new ValidationError("Email already in use", "email");
  }

  return User.create(data);
}

// Handle with specific logic
createUser({ name: "Alice" }).catch((err) => {
  if (err instanceof ValidationError) {
    console.error(`Validation failed on ${err.field}: ${err.message}`);
  } else if (err instanceof NotFoundError) {
    console.error(`Not found: ${err.message}`);
  } else {
    console.error("Unknown error:", err);
  }
});
```

---

## 6. Concurrency Patterns

### Worker Threads (CPU-Intensive Tasks)

```javascript
const { Worker } = require("worker_threads");

// CPU-intensive calculation
function expensiveCalculation() {
  let sum = 0;
  for (let i = 0; i < 1000000000; i++) {
    sum += i;
  }
  return sum;
}

// Without workers (blocks event loop)
function blockingApproach() {
  console.log("Calculating...");
  const result = expensiveCalculation();
  console.log("Result:", result);
}

// With workers (non-blocking)
function workerApproach() {
  return new Promise((resolve, reject) => {
    const worker = new Worker("./worker.js");

    worker.on("message", resolve);
    worker.on("error", reject);
    worker.on("exit", (code) => {
      if (code !== 0) {
        reject(new Error(`Worker exited with code ${code}`));
      }
    });
  });
}

// worker.js
// const { parentPort } = require('worker_threads');
//
// function calculate() {
//     let sum = 0;
//     for (let i = 0; i < 1000000000; i++) {
//         sum += i;
//     }
//     return sum;
// }
//
// parentPort.postMessage(calculate());

// Usage
workerApproach().then((result) => console.log("Result:", result));
```

### Rate Limiting / Concurrency Control

```javascript
// Limit concurrent promises
async function pLimit(limit, promises) {
  const results = [];
  let active = 0;
  let queue = [];

  function processQueue() {
    while (queue.length > 0 && active < limit) {
      active++;
      const { promise, index } = queue.shift();

      promise
        .then((result) => {
          results[index] = result;
        })
        .finally(() => {
          active--;
          processQueue();
        });
    }
  }

  promises.forEach((promise, index) => {
    queue.push({ promise, index });
  });

  processQueue();

  return new Promise((resolve) => {
    const check = setInterval(() => {
      if (active === 0 && queue.length === 0) {
        clearInterval(check);
        resolve(results);
      }
    }, 10);
  });
}

// Usage: fetch 100 URLs but only 5 at a time
const urls = Array(100).fill("https://api.example.com/data");
const promises = urls.map((url) => fetch(url));
pLimit(5, promises).then((results) => console.log("Done"));

// Or use a library
const pLimit = require("p-limit");
const limit = pLimit(5);
const promises = urls.map((url) => limit(() => fetch(url)));
Promise.all(promises).then(() => console.log("Done"));
```

### Queue Pattern

```javascript
class Queue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }

  add(task) {
    return new Promise((resolve, reject) => {
      this.queue.push({ task, resolve, reject });
      this.process();
    });
  }

  process() {
    while (this.running < this.concurrency && this.queue.length > 0) {
      this.running++;
      const { task, resolve, reject } = this.queue.shift();

      Promise.resolve()
        .then(() => task())
        .then(resolve)
        .catch(reject)
        .finally(() => {
          this.running--;
          this.process();
        });
    }
  }
}

// Usage
const queue = new Queue(3); // Max 3 concurrent

const tasks = Array(10)
  .fill(null)
  .map(
    (_, i) => () =>
      new Promise((resolve) => {
        console.log(`Task ${i} started`);
        setTimeout(() => {
          console.log(`Task ${i} completed`);
          resolve(i);
        }, Math.random() * 1000);
      })
  );

Promise.all(tasks.map((task) => queue.add(task))).then(() =>
  console.log("All tasks done")
);
```

---

## 7. Cluster Module

### Clustering for Load Distribution

```javascript
const cluster = require("cluster");
const http = require("http");
const os = require("os");

if (cluster.isMaster) {
  const numWorkers = os.cpus().length;
  console.log(`Master ${process.pid} starting ${numWorkers} workers`);

  // Fork workers
  for (let i = 0; i < numWorkers; i++) {
    cluster.fork();
  }

  // Handle worker exit
  cluster.on("exit", (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died (${signal || code})`);
    // Restart worker
    cluster.fork();
  });
} else {
  // Worker process
  const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Hello from worker ${process.pid}\n`);
  });

  server.listen(3000);
  console.log(`Worker ${process.pid} started, listening on port 3000`);
}

// With pm2, set instances: 'max' for auto-clustering
// pm2.json:
// {
//   "apps": [{
//     "name": "app",
//     "script": "./index.js",
//     "instances": "max",
//     "exec_mode": "cluster"
//   }]
// }
```

---

## 8. Callbacks vs Event Emitter

### Event Emitter Pattern

```javascript
const EventEmitter = require("events");

class DataProcessor extends EventEmitter {
  async process(data) {
    this.emit("start", { id: data.id });

    try {
      await new Promise((resolve) => setTimeout(resolve, 1000));

      const result = data.value * 2;
      this.emit("data", result);
    } catch (error) {
      this.emit("error", error);
      return;
    }

    this.emit("complete", { id: data.id, success: true });
  }
}

// Usage
const processor = new DataProcessor();

processor.on("start", (info) => console.log("Starting:", info));
processor.on("data", (result) => console.log("Result:", result));
processor.on("complete", (info) => console.log("Complete:", info));
processor.on("error", (err) => console.error("Error:", err));

processor.process({ id: 1, value: 21 });
```

---

## 9. Memory Management

### Detecting Memory Leaks

```javascript
// Memory usage tracking
function printMemory(label) {
  const usage = process.memoryUsage();
  console.log(`${label}:`);
  console.log(`  RSS: ${Math.round(usage.rss / 1024 / 1024)}MB`);
  console.log(`  Heap Used: ${Math.round(usage.heapUsed / 1024 / 1024)}MB`);
  console.log(`  Heap Total: ${Math.round(usage.heapTotal / 1024 / 1024)}MB`);
}

// Test: possible memory leak
printMemory("Start");

const leak = [];
for (let i = 0; i < 1000000; i++) {
  leak.push({
    data: new Array(1000).fill("x"),
    timestamp: Date.now(),
  });
}

printMemory("After allocation");

// Clear to free memory
leak.length = 0;
if (global.gc) {
  global.gc();
}

printMemory("After cleanup");

// Run with: node --expose-gc script.js
```

### Circular References

```javascript
// Circular reference (GC should handle it)
class Node {
  constructor(name) {
    this.name = name;
    this.next = null;
  }
}

// Create cycle
const a = new Node("A");
const b = new Node("B");
a.next = b;
b.next = a;

// Even though it's circular, GC will clean up when no external references
// (Don't do this in real code, just for demo)
```

---

## 10. Common Patterns

### Middleware Pattern (Express-like)

```javascript
function compose(middlewares) {
  return (req) => {
    let index = -1;

    function dispatch(i) {
      if (i <= index)
        return Promise.reject(new Error("next() called multiple times"));
      index = i;

      const middleware = middlewares[i];
      if (!middleware) return Promise.resolve();

      try {
        return Promise.resolve(middleware(req, () => dispatch(i + 1)));
      } catch (err) {
        return Promise.reject(err);
      }
    }

    return dispatch(0);
  };
}

// Usage
const middleware1 = (req, next) => {
  console.log("Middleware 1 - before");
  return next();
};

const middleware2 = (req, next) => {
  console.log("Middleware 2 - before");
  return next();
};

const middleware3 = (req, next) => {
  console.log("Middleware 3");
};

const app = compose([middleware1, middleware2, middleware3]);
app({}).then(() => console.log("Done"));
```

### Singleton Pattern

```javascript
class Database {
  constructor(url) {
    if (Database.instance) {
      return Database.instance;
    }

    this.url = url;
    this.connected = false;
    Database.instance = this;
  }

  connect() {
    this.connected = true;
    console.log(`Connected to ${this.url}`);
  }
}

const db1 = new Database("mongodb://localhost");
const db2 = new Database("mongodb://other");

console.log(db1 === db2); // true
db1.connect();
console.log(db2.connected); // true
```

### Retry Pattern

```javascript
async function retry(fn, maxRetries = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (err) {
      if (attempt === maxRetries) {
        throw err;
      }
      console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
}

// Usage
async function unreliableOperation() {
  if (Math.random() < 0.7) {
    throw new Error("Random failure");
  }
  return "Success!";
}

retry(unreliableOperation)
  .then((result) => console.log(result))
  .catch((err) => console.error("Failed after retries:", err.message));
```

---

## 11. Testing

### Async Testing with Jest

```javascript
// async.test.js

// Test resolved promise
test("should resolve with value", async () => {
  const result = await Promise.resolve("success");
  expect(result).toBe("success");
});

// Test rejected promise
test("should reject", async () => {
  await expect(Promise.reject(new Error("failed"))).rejects.toThrow("failed");
});

// Test async function
async function fetchUser(id) {
  if (id < 0) throw new Error("Invalid ID");
  return { id, name: "Alice" };
}

test("should fetch user", async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe("Alice");
});

// Test with timeout
test("should complete within timeout", async () => {
  const result = await new Promise((resolve) => {
    setTimeout(() => resolve("done"), 100);
  });
  expect(result).toBe("done");
}, 1000); // 1 second timeout

// Describe async operations
describe("User API", () => {
  test("creates user", async () => {
    const user = await fetchUser(1);
    expect(user.id).toBe(1);
  });

  test("rejects invalid id", async () => {
    await expect(fetchUser(-1)).rejects.toThrow();
  });
});
```

### Mocking

```javascript
const fs = require("fs").promises;

jest.mock("fs");

test("should read file", async () => {
  fs.readFile.mockResolvedValue("file contents");

  const content = await fs.readFile("test.txt", "utf8");
  expect(content).toBe("file contents");
  expect(fs.readFile).toHaveBeenCalledWith("test.txt", "utf8");
});

test("should handle error", async () => {
  fs.readFile.mockRejectedValue(new Error("Not found"));

  await expect(fs.readFile("missing.txt", "utf8")).rejects.toThrow("Not found");
});
```
