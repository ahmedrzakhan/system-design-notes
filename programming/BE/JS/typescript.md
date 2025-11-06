# TypeScript Interview Questions & Answers

## 1. What is TypeScript and why use it?

**Q: Explain TypeScript**

A: TypeScript is a superset of JavaScript that adds static typing and other features. Compiles to JavaScript.

**Key features**:

- Static typing (catch errors at compile time)
- Type inference
- Interfaces and type aliases
- Classes with modifiers
- Generics
- Decorators
- Enums
- Namespaces
- Better IDE support and autocomplete
- Optional chaining, nullish coalescing
- Strict null checks

**Why use TypeScript**:

- Catch errors before runtime
- Self-documenting code (types as documentation)
- Better IDE autocomplete
- Refactoring safety
- Scales well for large projects
- Great for team development

**vs JavaScript**:

- TypeScript: Compiled, statically typed, verbose
- JavaScript: Interpreted, dynamically typed, flexible

**Setup**:

```bash
npm install -g typescript
tsc --init
tsc              # Compile
tsc --watch      # Watch mode
```

---

## 2. What is type annotation?

**Q: Explain type annotations**

A: Explicitly declare variable types.

```typescript
// Basic types
const name: string = "John";
const age: number = 30;
const active: boolean = true;
const nothing: null = null;
const unknown: undefined = undefined;

// Arrays
const numbers: number[] = [1, 2, 3];
const strings: Array<string> = ["a", "b", "c"];
const mixed: (string | number)[] = [1, "two", 3];

// Tuples
const tuple: [string, number] = ["hello", 42];
const optional: [string, number?] = ["hello"];

// Any (avoid when possible)
const anything: any = "anything goes";

// Unknown (safer than any)
const unknownValue: unknown = "value";
if (typeof unknownValue === "string") {
  console.log(unknownValue.toUpperCase());
}
```

**Function types**:

```typescript
function add(a: number, b: number): number {
  return a + b;
}

const multiply = (a: number, b: number): number => a * b;

// Optional parameter
function greet(name: string, greeting?: string): string {
  return greeting ? `${greeting}, ${name}` : `Hello, ${name}`;
}

// Default parameter
function createUser(name: string, role: string = "user"): void {
  console.log(`${name} is a ${role}`);
}

// Rest parameter
function sum(...numbers: number[]): number {
  return numbers.reduce((a, b) => a + b, 0);
}
```

---

## 3. What are interfaces?

**Q: Explain interfaces**

A: Contract defining structure of objects.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Optional property
}

const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com",
};

// Extending interfaces
interface Admin extends User {
  role: string;
  permissions: string[];
}

const admin: Admin = {
  id: 1,
  name: "John",
  email: "john@example.com",
  role: "admin",
  permissions: ["read", "write", "delete"],
};

// Function interface
interface Greet {
  (name: string): string;
}

const greet: Greet = (name) => `Hello, ${name}`;

// Indexable interface
interface StringArray {
  [index: number]: string;
}

const arr: StringArray = ["a", "b", "c"];

// Interface with methods
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

const calc: Calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
};
```

---

## 4. What are type aliases?

**Q: Explain type aliases and difference from interfaces**

A: Type aliases define custom types. Similar to interfaces but more flexible.

```typescript
// Basic type alias
type ID = string | number;
type Status = "pending" | "completed" | "failed";

const id: ID = 123;
const status: Status = "completed";

// Object type alias
type User = {
  id: number;
  name: string;
  email?: string;
};

const user: User = {
  id: 1,
  name: "John",
};

// Union types
type Result = { success: true; data: any } | { success: false; error: string };

const result: Result = {
  success: true,
  data: { name: "John" },
};

// Intersection types
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;

const person: Person = {
  name: "John",
  age: 30,
};

// Function type alias
type Callback = (data: string) => void;

const callback: Callback = (data) => {
  console.log(data);
};
```

**Interface vs Type Alias**:

| Interface                 | Type Alias       |
| ------------------------- | ---------------- |
| Extendable with `extends` | Merged with `&`  |
| Can be merged             | Cannot be merged |
| Better for OOP            | Better for FP    |
| Can implement             | More flexible    |

---

## 5. What are generics?

**Q: Explain generics**

A: Write reusable components that work with multiple types.

```typescript
// Generic function
function getFirstElement<T>(arr: T[]): T {
  return arr[0];
}

const firstNum = getFirstElement<number>([1, 2, 3]);
const firstStr = getFirstElement<string>(["a", "b", "c"]);

// Generic interface
interface Container<T> {
  value: T;
}

const numContainer: Container<number> = { value: 42 };
const strContainer: Container<string> = { value: "hello" };

// Generic class
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);

// Generic constraints
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}

getLength("hello"); // 5
getLength([1, 2, 3]); // 3
getLength({ length: 5 }); // 5

// Multiple generic types
function combine<T, U>(first: T, second: U): (T | U)[] {
  return [first, second];
}

combine<string, number>("hello", 42);

// Generic with default
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

createArray(3, "a"); // ["a", "a", "a"]
createArray<number>(3, 0); // [0, 0, 0]
```

---

## 6. What are classes?

**Q: Explain TypeScript classes**

A: Object-oriented programming with strong typing.

```typescript
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet(): string {
    return `Hello, I'm ${this.name}`;
  }
}

const person = new Person("John", 30);
person.greet(); // "Hello, I'm John"

// Access modifiers
class User {
  public id: number; // Accessible everywhere
  private password: string; // Accessible only in class
  protected email: string; // Accessible in class and subclasses
  readonly name: string; // Cannot be changed after initialization

  constructor(id: number, password: string, email: string, name: string) {
    this.id = id;
    this.password = password;
    this.email = email;
    this.name = name;
  }
}

// Shorthand
class User {
  constructor(
    public id: number,
    private password: string,
    protected email: string,
    readonly name: string
  ) {}
}

// Inheritance
class Admin extends User {
  role: string;

  constructor(
    id: number,
    password: string,
    email: string,
    name: string,
    role: string
  ) {
    super(id, password, email, name);
    this.role = role;
  }

  // Override method
  greet(): string {
    return `Hello, I'm an admin: ${this.name}`;
  }
}

// Abstract classes
abstract class Animal {
  abstract makeSound(): void;

  move(): void {
    console.log("Moving");
  }
}

class Dog extends Animal {
  makeSound(): void {
    console.log("Woof!");
  }
}

// Static members
class MathUtil {
  static PI = 3.14159;

  static add(a: number, b: number): number {
    return a + b;
  }
}

MathUtil.add(2, 3);
```

---

## 7. What are enums?

**Q: Explain enums**

A: Set of named constants.

```typescript
// Numeric enum
enum Direction {
  Up = 0,
  Down = 1,
  Left = 2,
  Right = 3,
}

let dir: Direction = Direction.Up;

// String enum
enum Status {
  Active = "active",
  Inactive = "inactive",
  Pending = "pending",
}

let status: Status = Status.Active;

// Heterogeneous enum (mixed)
enum Mixed {
  No = 0,
  Yes = "yes",
}

// Reverse mapping (numeric enums only)
enum Color {
  Red = 0,
  Green = 1,
  Blue = 2,
}

const colorName = Color[0]; // "Red"

// Const enum (better performance)
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}

// Using enums in functions
function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

move(Direction.Up);
```

---

## 8. What are unions and intersections?

**Q: Explain union and intersection types**

A:

**Union** — value can be one of several types:

```typescript
type ID = string | number;

function getId(id: ID) {
  if (typeof id === "string") {
    return id.toUpperCase();
  } else {
    return id * 2;
  }
}

getId("abc");
getId(123);

// With interfaces
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; size: number };
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
  }
}
```

**Intersection** — value has all properties of multiple types:

```typescript
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;

const person: Person = {
  name: "John",
  age: 30,
};

// With interfaces
interface Animal {
  eat(): void;
}

interface Mammal {
  warmBlooded: boolean;
}

type Creature = Animal & Mammal;

const creature: Creature = {
  warmBlooded: true,
  eat: () => console.log("eating"),
};
```

---

## 9. What is type narrowing?

**Q: Explain type narrowing (type guards)**

A: Refine types in conditional branches.

```typescript
// typeof narrowing
function process(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else {
    console.log(value * 2);
  }
}

// instanceof narrowing
class Dog {
  bark() {
    console.log("Woof!");
  }
}

class Cat {
  meow() {
    console.log("Meow!");
  }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// Truthiness narrowing
function printLength(str: string | null) {
  if (str) {
    console.log(str.length);
  } else {
    console.log("String is null");
  }
}

// Discriminated union (exhaustiveness)
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; size: number };
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
  }
}

// Custom type guard
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function process(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase());
  }
}
```

---

## 10. What are utility types?

**Q: Explain built-in utility types**

A: TypeScript provides utility types for common transformations.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Partial — all properties optional
type PartialUser = Partial<User>;
const user: PartialUser = { name: "John" };

// Required — all properties required
type RequiredUser = Required<PartialUser>;
// Must have all properties

// Readonly — all properties readonly
type ReadonlyUser = Readonly<User>;
// Cannot modify properties

// Pick — select specific properties
type UserPreview = Pick<User, "id" | "name">;
const preview: UserPreview = { id: 1, name: "John" };

// Omit — exclude specific properties
type UserWithoutEmail = Omit<User, "email">;

// Record — object with specific keys
type UserRole = Record<"admin" | "user" | "guest", User>;
const roles: UserRole = {
  admin: { id: 1, name: "Admin", email: "admin@example.com", age: 30 },
  user: { id: 2, name: "User", email: "user@example.com", age: 25 },
  guest: { id: 3, name: "Guest", email: "guest@example.com", age: 20 },
};

// Exclude — exclude types from union
type Exclude1 = Exclude<"a" | "b" | "c", "b">; // 'a' | 'c'

// Extract — extract types from union
type Extract1 = Extract<"a" | "b" | "c", "a" | "b">; // 'a' | 'b'

// NonNullable — remove null/undefined
type NonNull = NonNullable<string | null | undefined>; // string

// ReturnType — get return type of function
type GetUserReturn = ReturnType<typeof getUser>;

// Parameters — get parameter types
type GetUserParams = Parameters<typeof getUser>;
```

---

## 11. What are conditional types?

**Q: Explain conditional types**

A: Types that depend on other types.

```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type A = IsString<"hello">; // true
type B = IsString<number>; // false

// With generics
type Flatten<T> = T extends Array<infer U> ? U : T;

type Str = Flatten<string[]>; // string
type Num = Flatten<number>; // number

// Extracting from union
type ToArray<T> = T extends any ? T[] : never;

type StrArray = ToArray<string | number>; // string[] | number[]

// Type inference
type GetReturnType<T> = T extends (...args: any) => infer R ? R : any;

function getUser() {
  return { id: 1, name: "John" };
}

type UserType = GetReturnType<typeof getUser>; // { id: number; name: string }
```

---

## 12. What is null and undefined?

**Q: Strict null checks**

A: TypeScript can enforce strict null checking.

```typescript
// Without strictNullChecks
let value: string = null; // OK (not recommended)

// With strictNullChecks (recommended)
let value: string = null; // Error!

// Explicitly allow null/undefined
let value: string | null = null;
let value2: string | undefined = undefined;
let value3: string | null | undefined = null;

// Optional chaining
const user = { name: "John", address: { city: "NYC" } };
const city = user?.address?.city; // "NYC"
const zip = user?.address?.zip; // undefined

// Nullish coalescing
const city = user?.address?.city ?? "Unknown";
const id = null ?? 5; // 5
const count = 0 ?? 10; // 0 (not 10!)

// Non-null assertion (use sparingly)
const value: string | null = "hello";
const length = value!.length; // Remove null from type
```

---

## 13. What are decorators?

**Q: Explain decorators**

A: Functions that modify classes or properties.

```typescript
// Enable experimentalDecorators in tsconfig.json

// Class decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Greeter {
  greeting: string = "Hello";
}

// Property decorator
function validate(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    if (args[0] < 0) {
      throw new Error("Must be positive");
    }
    return originalMethod.apply(this, args);
  };
}

class User {
  @validate
  setAge(age: number) {
    this.age = age;
  }
}

// Parameter decorator
function logParameter(target: any, methodName: string, paramIndex: number) {
  console.log(`Logging parameter ${paramIndex} of ${methodName}`);
}

class Calculator {
  add(@logParameter a: number, b: number) {
    return a + b;
  }
}

// Accessor decorator
function readonly(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  descriptor.writable = false;
}

class User {
  @readonly
  name: string = "John";
}
```

---

## 14. What is async/await in TypeScript?

**Q: Async operations with types**

A:

```typescript
// Promise typing
function fetchUser(id: number): Promise<User> {
  return fetch(`/api/users/${id}`).then((res) => res.json());
}

// Async function
async function getUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// Error handling
async function getUser(id: number): Promise<User | null> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      return null;
    }
    return response.json();
  } catch (error) {
    console.error(error);
    return null;
  }
}

// Multiple awaits
async function getUserWithPosts(id: number) {
  const user = await getUser(id);
  if (!user) return null;

  const posts = await getPosts(user.id);
  return { user, posts };
}

// Parallel execution
async function getUserAndPosts(id: number) {
  const [user, posts] = await Promise.all([getUser(id), getPosts(id)]);
  return { user, posts };
}
```

---

## 15. What are modules?

**Q: TypeScript modules**

A:

```typescript
// Export
export function add(a: number, b: number): number {
  return a + b;
}

export interface User {
  id: number;
  name: string;
}

export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

export default function greet(name: string) {
  return `Hello, ${name}`;
}

// Import
import add, { User, Calculator } from "./math";
import * as math from "./math";

// Named imports
import { add, subtract } from "./math";

// Default import
import greet from "./greet";

// Re-export
export { add, subtract } from "./math";
export * from "./utils";
```

---

## 16. What is tsconfig.json?

**Q: Explain TypeScript configuration**

A:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noImplicitAny": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "alwaysStrict": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

**Key options**:

- `target` — ECMAScript version to compile to
- `module` — Module system (commonjs, es2020, etc.)
- `lib` — Library files included
- `strict` — Enable all strict type checks
- `outDir` — Output directory
- `rootDir` — Input directory
- `declaration` — Generate .d.ts files

---

## 17. What is type inference?

**Q: TypeScript infers types**

A: TypeScript automatically detects types when not specified.

```typescript
// Type inference
let name = "John"; // string
let age = 30; // number
let active = true; // boolean
let anything = null; // any (null is tricky)

// Function return type inference
function add(a: number, b: number) {
  return a + b; // number (inferred)
}

// Array inference
const numbers = [1, 2, 3]; // number[] (inferred)
const mixed = [1, "a"]; // (string | number)[] (inferred)

// Object inference
const user = {
  id: 1,
  name: "John",
  active: true,
};
// Inferred as: { id: number; name: string; active: boolean; }

// Context-based inference
const handler = (event: MouseEvent) => {
  // event type is inferred from context
};

// Type widening
let x = 42; // number
let y = "hello" as const; // "hello" (literal)
const z = "hello"; // "hello" (literal, const)
```

---

## 18. What is declaration files?

**Q: Explain .d.ts files**

A: Type definitions for JavaScript libraries.

```typescript
// user.d.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export function getUser(id: number): Promise<User>;
export function createUser(user: Omit<User, "id">): Promise<User>;

// OR in your package
declare module "some-library" {
  export function someFunction(): string;
}

// Using in code
import { User, getUser } from "./user";

const user = await getUser(1);
// TypeScript knows user is of type User
```

**Writing type definitions**:

```typescript
// math.d.ts
export function add(a: number, b: number): number;
export function subtract(a: number, b: number): number;

interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

export const calculator: Calculator;
```

---

## 19. What is type compatibility?

**Q: Structural typing in TypeScript**

A: TypeScript uses structural typing (duck typing).

```typescript
interface User {
  id: number;
  name: string;
}

// These objects are compatible with User
const user1: User = {
  id: 1,
  name: "John",
};

const user2 = {
  id: 2,
  name: "Jane",
  email: "jane@example.com", // Extra property OK
};

const user3: User = user2; // OK - has all required properties

// Excess properties in literals are checked
const user4: User = {
  id: 3,
  name: "Bob",
  email: "bob@example.com", // Error - excess property in literal
};

// Function compatibility
type Callback = (x: number) => void;
const callback: Callback = (x: number) => console.log(x);

// Can assign function with same/subtype parameters
const callback2: Callback = (x: number | string) => console.log(x); // OK

// Contravariance - parameter types
function printNumber(x: number) {}
function printNumberOrString(x: number | string) {}

let fn: (x: number) => void = printNumberOrString; // Error!
let fn2: (x: number | string) => void = printNumber; // OK
```

---

## 20. What is type assertion?

**Q: Type casting**

A: Tell TypeScript what type something is.

```typescript
// Using 'as'
const value: unknown = "hello";
const length = (value as string).length;

// Using <> (not in JSX)
const length = (<string>value).length;

// Non-null assertion (!)
const user: User | null = getUser();
const name = user!.name; // Tells TypeScript user is not null

// Const assertion
const colors = ["red", "green", "blue"] as const;
// colors is now readonly ["red", "green", "blue"]

// Assertions must be compatible
const age = "30" as number; // Error! String can't become number
const age = "30" as unknown as number; // OK (double assertion, avoid)

// Better: proper typing
const age = parseInt("30"); // Better than assertion
```

---

## 21. What is readonly?

**Q: Explain readonly properties**

A:

```typescript
interface User {
  readonly id: number;
  name: string;
}

const user: User = {
  id: 1,
  name: "John",
};

user.name = "Jane"; // OK
user.id = 2; // Error - readonly

// Readonly arrays
const numbers: readonly number[] = [1, 2, 3];
numbers.push(4); // Error

// Readonly class property
class User {
  readonly id: number;
  name: string;

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }
}

// Readonly utility type
type ReadonlyUser = Readonly<User>;
// All properties become readonly
```

---

## 22. What are advanced patterns?

**Q: Advanced TypeScript patterns**

A:

**Builder pattern**:

```typescript
class UserBuilder {
  private user: Partial<User> = {};

  setId(id: number): this {
    this.user.id = id;
    return this;
  }

  setName(name: string): this {
    this.user.name = name;
    return this;
  }

  build(): User {
    return this.user as User;
  }
}

const user = new UserBuilder().setId(1).setName("John").build();
```

**Mixin pattern**:

```typescript
function mixin<T extends { new (...args: any[]): {} }>(base: T) {
  return class extends base {
    timestamp = Date.now();
  };
}

class User {
  name: string = "John";
}

const UserWithTimestamp = mixin(User);
const user = new UserWithTimestamp(); // Has name and timestamp
```

**Discriminated union pattern**:

```typescript
type Result<T> =
  | { status: "success"; data: T }
  | { status: "error"; error: string };

function handle<T>(result: Result<T>) {
  switch (result.status) {
    case "success":
      return result.data;
    case "error":
      throw new Error(result.error);
  }
}
```

---

## 23. What is React with TypeScript?

**Q: TypeScript in React**

A:

```typescript
import React, { FC, useState, useEffect } from "react";

interface Props {
  title: string;
  count?: number;
  onClick: (count: number) => void;
}

const Counter: FC<Props> = ({ title, count = 0, onClick }) => {
  const [internal, setInternal] = useState<number>(0);

  useEffect(() => {
    console.log(`Component mounted with initial count: ${internal}`);
  }, []);

  const handleClick = () => {
    const newCount = internal + 1;
    setInternal(newCount);
    onClick(newCount);
  };

  return (
    <div>
      <h1>{title}</h1>
      <p>Count: {count + internal}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
};

export default Counter;

// With Redux
import { useDispatch, useSelector } from "react-redux";

interface RootState {
  count: number;
}

const selector = (state: RootState) => state.count;

const MyComponent: FC = () => {
  const dispatch = useDispatch();
  const count = useSelector(selector);

  return <div>{count}</div>;
};
```

---

## 24. What is testing with TypeScript?

**Q: Test TypeScript**

A:

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

// math.test.ts
import { add } from "./math";

describe("Math", () => {
  it("should add two numbers", () => {
    expect(add(2, 3)).toBe(5);
  });

  it("should add negative numbers", () => {
    expect(add(-2, 3)).toBe(1);
  });
});

// Testing async functions
export async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// user.test.ts
describe("User API", () => {
  it("should fetch user", async () => {
    const user = await fetchUser(1);
    expect(user.id).toBe(1);
  });
});
```

---

## 25. What are common issues?

**Q: Common TypeScript mistakes**

A:

**1. Using `any` too much**:

```typescript
// Bad
let value: any = 42;
value.unknownMethod(); // No error but runtime crash

// Good
let value: unknown = 42;
if (typeof value === "number") {
  // Now TypeScript knows it's a number
}
```

**2. Incorrect function overloading**:

```typescript
// Bad
function greet(name: string): string;
function greet(name: string, greeting: string): string;
function greet(name: string, greeting?: string): string {
  return greeting ? `${greeting}, ${name}` : `Hello, ${name}`;
}

// Good - more specific first
function greet(name: string, greeting: string): string;
function greet(name: string): string;
function greet(name: string, greeting?: string): string {
  return greeting ? `${greeting}, ${name}` : `Hello, ${name}`;
}
```

**3. Non-null assertions everywhere**:

```typescript
// Bad
const value = getValue()!;
const length = value!.length;

// Good
const value = getValue();
if (value !== null) {
  console.log(value.length);
}
```

**4. Loose type inference**:

```typescript
// Bad
const config = { timeout: 5000 };
const timeout: string = config.timeout; // Error later

// Good
const config = { timeout: 5000 } as const;
// or
interface Config {
  timeout: number;
}
```

---

## TypeScript Interview Tips

1. **Know type annotations** — Core feature
2. **Understand interfaces vs types** — Use appropriately
3. **Generics** — Reusable type-safe code
4. **Type narrowing** — Discriminated unions, type guards
5. **Utility types** — Pick, Omit, Partial, Required, etc.
6. **Decorators** — Understand basic usage
7. **Modules** — Import/export patterns
8. **Strict mode** — Enable strict checking
9. **Avoid `any`** — Use `unknown` or better types
10. **Type inference** — Let TypeScript figure out types
11. **React with TypeScript** — Props typing, hooks
12. **Testing** — Jest with TypeScript
13. **Performance** — Avoid excessive type instantiation
14. **Production code** — Show real TypeScript projects
15. **tsconfig.json** — Know common settings
