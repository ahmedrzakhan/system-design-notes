# Node.js Interview Questions & Answers

## 1. What is Node.js and why use it?

**Q: Explain Node.js**

A: Node.js is a JavaScript runtime built on Chrome's V8 engine that allows running JavaScript outside the browser, typically on servers.

**Key features**:

- Event-driven, non-blocking I/O
- Single-threaded (with worker threads)
- Asynchronous programming model
- NPM ecosystem (largest package repository)
- Fast execution (V8 engine)
- JavaScript on both frontend and backend
- Great for real-time applications

**Why use Node.js**:

- Fast development
- Scalable with async I/O
- Large ecosystem
- Good for APIs, microservices, real-time apps
- Same language frontend and backend
- Non-blocking I/O handles many connections efficiently

**Limitations**:

- CPU-intensive tasks are slower
- Single-threaded (can use Worker Threads)
- Less mature than some alternatives

---

## 2. What is the event loop?

**Q: Explain the event loop**

A: Core mechanism that allows Node.js to handle async operations with a single thread.

**How it works**:

```
┌───────────────────────────────┐
│ Check timers (setTimeout, setInterval)
├───────────────────────────────┤
│ Execute pending callbacks
├───────────────────────────────┤
│ Idle/prepare
├───────────────────────────────┤
│ Poll (I/O, filesystem, network)
├───────────────────────────────┤
│ Check (setImmediate)
├───────────────────────────────┤
│ Close callbacks
└───────────────────────────────┘
```

**Phases**:

1. **timers** — setTimeout/setInterval
2. **pending callbacks** — deferred I/O callbacks
3. **idle/prepare** — internal
4. **poll** — retrieve new I/O events
5. **check** — setImmediate
6. **close callbacks** — cleanup

**Example**:

```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout");
}, 0);

setImmediate(() => {
  console.log("Immediate");
});

console.log("End");

// Output:
// Start
// End
// Immediate
// Timeout
```

---

## 3. What is asynchronous programming?

**Q: Async/await vs callbacks vs promises**

A: Handle non-blocking operations.

**Callbacks** (old way):

```javascript
function readFile(filename, callback) {
  fs.readFile(filename, "utf8", (err, data) => {
    if (err) callback(err);
    else callback(null, data);
  });
}

readFile("file.txt", (err, data) => {
  if (err) console.error(err);
  else console.log(data);
});
```

**Promises**:

```javascript
function readFile(filename) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, "utf8", (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

readFile("file.txt")
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

**Async/await** (modern):

```javascript
async function main() {
  try {
    const data = await readFile("file.txt");
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}

main();
```

**Parallel operations**:

```javascript
// Sequence (slow)
const file1 = await readFile("file1.txt");
const file2 = await readFile("file2.txt");

// Parallel (fast)
const [file1, file2] = await Promise.all([
  readFile("file1.txt"),
  readFile("file2.txt"),
]);
```

---

## 4. What are callbacks?

**Q: Explain callbacks**

A: Function passed as argument to be called later.

```javascript
function greet(name, callback) {
  console.log("Hello " + name);
  callback();
}

greet("John", () => {
  console.log("Callback executed");
});
```

**Common in Node.js** (before promises):

```javascript
const fs = require("fs");

fs.readFile("file.txt", "utf8", (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

**Callback hell** (avoid):

```javascript
// Deep nesting — hard to read
fs.readFile("file.txt", (err, data) => {
  if (err) throw err;
  fs.writeFile("output.txt", data, (err) => {
    if (err) throw err;
    fs.readFile("output.txt", (err, data) => {
      if (err) throw err;
      console.log(data);
    });
  });
});

// Use async/await instead
```

---

## 5. What are Promises?

**Q: Explain Promises**

A: Object representing eventual completion/failure of async operation.

```javascript
const promise = new Promise((resolve, reject) => {
  if (condition) {
    resolve("Success");
  } else {
    reject("Error");
  }
});

promise
  .then((result) => console.log(result))
  .catch((error) => console.error(error))
  .finally(() => console.log("Done"));
```

**States**:

- Pending — initial state
- Fulfilled — operation completed successfully
- Rejected — operation failed

**Chain promises**:

```javascript
readFile("file1.txt")
  .then((data) => writeFile("file2.txt", data))
  .then(() => readFile("file2.txt"))
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

**Promise.all** — wait for all:

```javascript
Promise.all([promise1, promise2, promise3])
  .then((results) => console.log(results))
  .catch((err) => console.error(err));
```

**Promise.race** — first to complete:

```javascript
Promise.race([promise1, promise2]).then((result) => console.log(result));
```

---

## 6. What is async/await?

**Q: Explain async/await**

A: Syntactic sugar for Promises. Makes async code look synchronous.

```javascript
async function main() {
  const result = await someAsyncFunction();
  console.log(result);
}

main();
```

**Error handling**:

```javascript
async function main() {
  try {
    const data = await readFile("file.txt");
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

**Parallel operations**:

```javascript
// Sequence (slow)
async function sequence() {
  const file1 = await readFile("file1.txt");
  const file2 = await readFile("file2.txt");
  return [file1, file2];
}

// Parallel (fast)
async function parallel() {
  const [file1, file2] = await Promise.all([
    readFile("file1.txt"),
    readFile("file2.txt"),
  ]);
  return [file1, file2];
}
```

**Rules**:

- `await` only works inside `async` function
- `async` function returns Promise
- `await` pauses execution until Promise resolves

---

## 7. What is the module system?

**Q: Explain CommonJS and ES Modules**

A: Two ways to organize code into reusable modules.

**CommonJS** (traditional Node.js):

```javascript
// math.js
function add(a, b) {
  return a + b;
}

module.exports = { add };

// main.js
const { add } = require("./math");
console.log(add(2, 3));
```

**ES Modules** (modern):

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

export const PI = 3.14;

// main.js
import { add, PI } from "./math.js";
console.log(add(2, 3));
```

**Default export**:

```javascript
// logger.js
export default class Logger {
  log(msg) {
    console.log(msg);
  }
}

// main.js
import Logger from "./logger.js";
const logger = new Logger();
logger.log("Hello");
```

**Using ES modules in Node.js**:

```json
{
  "type": "module"
}
```

Or use `.mjs` extension.

---

## 8. What is npm?

**Q: Explain npm (Node Package Manager)**

A: Package manager for Node.js. Manage dependencies and publish packages.

**Common commands**:

```bash
npm init                    # Initialize project
npm install package         # Install package
npm install -g package      # Install globally
npm install -D package      # Install as dev dependency
npm list                    # List installed packages
npm update package          # Update package
npm uninstall package       # Remove package
npm search keyword          # Search packages
npm publish                 # Publish package
npm run script              # Run npm script
```

**package.json**:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0",
    "jest": "^27.0.0"
  }
}
```

**Version syntax**:

- `1.2.3` — exact version
- `^1.2.3` — compatible with version (minor and patch)
- `~1.2.3` — approximately version (patch only)
- `*` or `latest` — latest version

---

## 9. What is Express.js?

**Q: Explain Express.js**

A: Lightweight web framework for Node.js. Build APIs and web servers.

```javascript
const express = require("express");
const app = express();

app.use(express.json());

// Route
app.get("/", (req, res) => {
  res.json({ message: "Hello" });
});

// Route with parameter
app.get("/users/:id", (req, res) => {
  const { id } = req.params;
  res.json({ user_id: id });
});

// Query parameters
app.get("/search", (req, res) => {
  const { q } = req.query;
  res.json({ query: q });
});

// POST with body
app.post("/items", (req, res) => {
  const { name, price } = req.body;
  res.json({ name, price });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

**Middleware**:

```javascript
// Built-in middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Custom middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// Error middleware (4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});
```

---

## 10. What are middleware functions?

**Q: Explain middleware**

A: Function that has access to request, response, and next middleware.

```javascript
const express = require("express");
const app = express();

// Middleware 1
app.use((req, res, next) => {
  console.log("Middleware 1");
  next(); // Pass to next middleware
});

// Middleware 2
app.use((req, res, next) => {
  console.log("Middleware 2");
  next();
});

// Route
app.get("/", (req, res) => {
  res.send("Hello");
});

// Error middleware
app.use((err, req, res, next) => {
  res.status(500).send("Error");
});
```

**Common middleware**:

```javascript
// CORS
const cors = require("cors");
app.use(cors());

// Logging
app.use((req, res, next) => {
  console.log(`${new Date()} - ${req.method} ${req.url}`);
  next();
});

// Authentication
app.use((req, res, next) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).send("Unauthorized");
  }
  next();
});

// Error handling
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({ error: err.message });
});
```

**Middleware execution order**:

```javascript
// Runs in order
app.use(middleware1);
app.use(middleware2);
app.use(middleware3);

// middleware1 → middleware2 → middleware3 → route
```

---

## 11. What is routing?

**Q: Explain routing**

A: Mapping HTTP requests to handlers.

```javascript
// Basic routes
app.get("/", (req, res) => res.send("GET"));
app.post("/", (req, res) => res.send("POST"));
app.put("/", (req, res) => res.send("PUT"));
app.delete("/", (req, res) => res.send("DELETE"));

// Dynamic routes
app.get("/users/:id", (req, res) => {
  res.json({ id: req.params.id });
});

app.get("/posts/:postId/comments/:commentId", (req, res) => {
  res.json({
    postId: req.params.postId,
    commentId: req.params.commentId,
  });
});

// Regex routes
app.get(/^\/users\/\d+$/, (req, res) => {
  res.send("User ID must be numeric");
});

// Query parameters
app.get("/search", (req, res) => {
  const { q, page = 1 } = req.query;
  res.json({ query: q, page });
});
```

**Router (modular)**:

```javascript
// routes/users.js
const express = require("express");
const router = express.Router();

router.get("/", (req, res) => {
  res.json({ users: [] });
});

router.get("/:id", (req, res) => {
  res.json({ user_id: req.params.id });
});

module.exports = router;

// main.js
const userRoutes = require("./routes/users");
app.use("/api/users", userRoutes);

// GET /api/users
// GET /api/users/5
```

---

## 12. What is database integration?

**Q: Connect to databases**

A: Multiple database options with Node.js.

**MongoDB with Mongoose**:

```javascript
const mongoose = require("mongoose");

mongoose.connect("mongodb://localhost:27017/mydb");

const userSchema = new mongoose.Schema({
  name: String,
  email: String,
});

const User = mongoose.model("User", userSchema);

// Create
const user = await User.create({ name: "John", email: "john@example.com" });

// Read
const users = await User.find();
const user = await User.findById(id);

// Update
await User.findByIdAndUpdate(id, { name: "Jane" });

// Delete
await User.findByIdAndDelete(id);
```

**PostgreSQL with Sequelize**:

```javascript
const { Sequelize, DataTypes } = require("sequelize");

const sequelize = new Sequelize("database", "user", "password", {
  host: "localhost",
  dialect: "postgres",
});

const User = sequelize.define("User", {
  name: DataTypes.STRING,
  email: DataTypes.STRING,
});

await sequelize.sync();
const user = await User.create({ name: "John", email: "john@example.com" });
```

**PostgreSQL with node-postgres**:

```javascript
const { Pool } = require("pg");

const pool = new Pool({
  user: "postgres",
  host: "localhost",
  database: "mydb",
  password: "password",
  port: 5432,
});

const result = await pool.query("SELECT * FROM users WHERE id = $1", [1]);
console.log(result.rows);
```

---

## 13. What is authentication?

**Q: How do you implement authentication?**

A: Multiple strategies.

**JWT (JSON Web Token)**:

```javascript
const jwt = require("jsonwebtoken");

const SECRET_KEY = "your-secret-key";

// Generate token
app.post("/login", (req, res) => {
  const { username, password } = req.body;

  // Verify credentials (simplified)
  if (username === "john" && password === "password") {
    const token = jwt.sign({ username }, SECRET_KEY, { expiresIn: "1h" });
    res.json({ token });
  } else {
    res.status(401).send("Invalid credentials");
  }
});

// Middleware to verify token
const verifyToken = (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).send("No token");

  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).send("Invalid token");
  }
};

// Protected route
app.get("/protected", verifyToken, (req, res) => {
  res.json({ user: req.user });
});
```

**bcrypt for passwords**:

```javascript
const bcrypt = require("bcrypt");

// Hash password
const hashedPassword = await bcrypt.hash(password, 10);

// Compare password
const isValid = await bcrypt.compare(password, hashedPassword);
```

---

## 14. What is error handling?

**Q: Handle errors properly**

A:

```javascript
// Sync errors
app.get("/divide/:a/:b", (req, res) => {
  try {
    const result = req.params.a / req.params.b;
    res.json({ result });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// Async errors
app.get("/users/:id", async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: "User not found" });
    }
    res.json(user);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: err.message,
    status: err.status || 500,
  });
});
```

**Async wrapper**:

```javascript
// Avoid try-catch in every route
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get(
  "/users/:id",
  asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);
    res.json(user);
  })
);
```

---

## 15. What is CORS?

**Q: Cross-Origin Resource Sharing**

A:

```javascript
const cors = require("cors");

// Allow all origins (development only)
app.use(cors());

// Allow specific origins
app.use(
  cors({
    origin: "http://localhost:3000",
    methods: ["GET", "POST"],
    credentials: true,
  })
);

// Or use routes
app.get("/public", cors(), (req, res) => {
  res.json({ data: "public" });
});

app.get("/private", (req, res) => {
  res.json({ data: "private" });
});
```

---

## 16. What are environment variables?

**Q: Configure app with .env**

A:

```javascript
require("dotenv").config();

const PORT = process.env.PORT || 3000;
const DATABASE_URL = process.env.DATABASE_URL;
const SECRET_KEY = process.env.SECRET_KEY;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**.env file**:

```
PORT=3000
DATABASE_URL=mongodb://localhost:27017/mydb
SECRET_KEY=my-secret-key
NODE_ENV=development
```

---

## 17. What is testing?

**Q: Test Node.js apps**

A: Use Jest or Mocha.

**Jest**:

```javascript
// math.js
function add(a, b) {
  return a + b;
}

module.exports = { add };

// math.test.js
const { add } = require("./math");

describe("Math", () => {
  test("adds two numbers", () => {
    expect(add(2, 3)).toBe(5);
  });

  test("adds negative numbers", () => {
    expect(add(-2, 3)).toBe(1);
  });
});
```

**API testing**:

```javascript
const request = require("supertest");
const app = require("./app");

describe("GET /users", () => {
  test("returns status 200", async () => {
    const res = await request(app).get("/users");

    expect(res.statusCode).toBe(200);
    expect(res.body).toHaveProperty("users");
  });
});
```

**Run tests**:

```bash
npm test
jest --coverage
```

---

## 18. What are streams?

**Q: Explain streams**

A: Handle large data efficiently without loading everything in memory.

```javascript
const fs = require("fs");

// Reading stream
const readable = fs.createReadStream("large-file.txt");
readable.on("data", (chunk) => {
  console.log("Received chunk:", chunk.length);
});

readable.on("end", () => {
  console.log("Finished reading");
});

// Writing stream
const writable = fs.createWriteStream("output.txt");
writable.write("Hello\n");
writable.write("World\n");
writable.end();

// Pipe (connect streams)
fs.createReadStream("input.txt").pipe(fs.createWriteStream("output.txt"));

// Multiple pipes
fs.createReadStream("input.txt")
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream("input.txt.gz"));
```

**Stream types**:

- Readable
- Writable
- Duplex (both readable and writable)
- Transform (modify data while reading/writing)

---

## 19. What is file system operations?

**Q: File operations with fs module**

A:

```javascript
const fs = require("fs");

// Read file
fs.readFile("file.txt", "utf8", (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Write file
fs.writeFile("output.txt", "Hello", (err) => {
  if (err) throw err;
  console.log("File written");
});

// Append
fs.appendFile("output.txt", "\nWorld", (err) => {
  if (err) throw err;
});

// Delete file
fs.unlink("output.txt", (err) => {
  if (err) throw err;
});

// List files
fs.readdir("./", (err, files) => {
  console.log(files);
});

// Check if file exists
if (fs.existsSync("file.txt")) {
  console.log("File exists");
}

// Promises (modern)
const data = await fs.promises.readFile("file.txt", "utf8");
await fs.promises.writeFile("output.txt", data);
```

---

## 20. What are Worker Threads?

**Q: CPU-intensive tasks**

A: Use Worker Threads for multi-threading.

```javascript
const { Worker } = require("worker_threads");

// main.js
const worker = new Worker("./worker.js");

worker.on("message", (result) => {
  console.log("Result:", result);
});

worker.postMessage({ number: 10 });

// worker.js
const { parentPort } = require("worker_threads");

parentPort.on("message", (data) => {
  const result = expensiveCalculation(data.number);
  parentPort.postMessage(result);
});

function expensiveCalculation(n) {
  // CPU-intensive work
  return n * n;
}
```

**When to use**:

- CPU-intensive calculations
- Heavy data processing
- Avoid blocking event loop

---

## 21. What is clustering?

**Q: Distribute work across CPU cores**

A:

```javascript
const cluster = require("cluster");
const os = require("os");
const express = require("express");

const numCPUs = os.cpus().length;

if (cluster.isMaster) {
  console.log(`Master process ${process.pid} is running`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on("exit", (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} exited`);
  });
} else {
  const app = express();

  app.get("/", (req, res) => {
    res.json({ pid: process.pid });
  });

  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started`);
  });
}
```

---

## 22. What is caching?

**Q: Implement caching**

A:

```javascript
const redis = require("redis");
const client = redis.createClient();

// Connect
await client.connect();

// Set cache
await client.set("user:1", JSON.stringify({ id: 1, name: "John" }), {
  EX: 3600, // Expires in 1 hour
});

// Get cache
const user = await client.get("user:1");

// Delete cache
await client.del("user:1");

// Using with Express
app.get("/users/:id", async (req, res) => {
  const cacheKey = `user:${req.params.id}`;

  // Check cache
  const cached = await client.get(cacheKey);
  if (cached) {
    return res.json(JSON.parse(cached));
  }

  // Get from DB
  const user = await User.findById(req.params.id);

  // Set cache
  await client.set(cacheKey, JSON.stringify(user), { EX: 3600 });

  res.json(user);
});
```

---

## 23. What is security best practices?

**Q: Secure Node.js apps**

A:

```javascript
const helmet = require("helmet");
const mongoSanitize = require("express-mongo-sanitize");
const rateLimit = require("express-rate-limit");

// Use Helmet for security headers
app.use(helmet());

// Sanitize input
app.use(mongoSanitize());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
});

app.use("/api/", limiter);

// Validate input
app.post("/users", (req, res) => {
  const { email, password } = req.body;

  if (!email || !password) {
    return res.status(400).json({ error: "Missing fields" });
  }

  // Process...
});

// Environment variables
require("dotenv").config();
const SECRET = process.env.SECRET_KEY;

// HTTPS only (in production)
if (process.env.NODE_ENV === "production") {
  app.use((req, res, next) => {
    if (req.header("x-forwarded-proto") !== "https") {
      return res.redirect(`https://${req.header("host")}${req.url}`);
    }
    next();
  });
}

// CORS restriction
app.use(
  cors({
    origin: process.env.ALLOWED_ORIGINS?.split(","),
  })
);

// Input validation library
const { body, validationResult } = require("express-validator");

app.post(
  "/users",
  [body("email").isEmail(), body("password").isLength({ min: 8 })],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Process...
  }
);
```

---

## 24. What is deployment?

**Q: Deploy Node.js apps**

A: Multiple hosting options.

**Heroku**:

```bash
git init
heroku create my-app
git push heroku main
```

**Docker**:

```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "index.js"]
```

**Docker Compose**:

```yaml
version: "3"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: mongodb://db:27017/mydb
    depends_on:
      - db
  db:
    image: mongo:5
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

**Using PM2 (process manager)**:

```bash
npm install -g pm2

pm2 start app.js
pm2 logs
pm2 save
pm2 startup
```

**Cloud platforms**:

- Vercel (serverless)
- AWS (EC2, Lambda, ECS)
- Google Cloud (App Engine, Cloud Run)
- Azure (App Service)
- DigitalOcean (Droplets, App Platform)

---

## 25. What is real-world example?

**Q: Complete API project**

A:

```javascript
// app.js
const express = require("express");
const mongoose = require("mongoose");
const dotenv = require("dotenv");
const cors = require("cors");

dotenv.config();

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGODB_URI);

// Models
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
  createdAt: { type: Date, default: Date.now },
});

const User = mongoose.model("User", userSchema);

// Routes
app.post("/api/users", async (req, res) => {
  try {
    const { name, email, password } = req.body;

    if (!name || !email || !password) {
      return res.status(400).json({ error: "Missing fields" });
    }

    const user = new User({ name, email, password });
    await user.save();

    res.status(201).json(user);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.get("/api/users", async (req, res) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.get("/api/users/:id", async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: "User not found" });
    }
    res.json(user);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.put("/api/users/:id", async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(req.params.id, req.body, {
      new: true,
    });
    res.json(user);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.delete("/api/users/:id", async (req, res) => {
  try {
    await User.findByIdAndDelete(req.params.id);
    res.json({ message: "User deleted" });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Error handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## Node.js Interview Tips

1. **Event loop** — Understand it deeply
2. **Async/await** — Use instead of callbacks
3. **Promises** — Know Promise methods
4. **Express.js** — Common framework
5. **Middleware** — How they work and chain
6. **Error handling** — Try-catch for async
7. **Databases** — MongoDB, PostgreSQL
8. **Authentication** — JWT, bcrypt
9. **Testing** — Jest, supertest
10. **Streams** — Efficient for large data
11. **Security** — Helmet, validation, sanitization
12. **Performance** — Clustering, caching, streams
13. **Deployment** — Docker, PM2, cloud platforms
14. **Production code** — Talk about real projects
