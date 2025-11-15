# Python Advanced Concepts Guide

## 1. The Global Interpreter Lock (GIL) and Why It Exists

The GIL is a mutex that protects access to Python objects in CPython. It ensures that only one thread can execute Python bytecode at a time, even on multi-core systems.

**Why it exists:** Python's memory management relies on reference counting. Without the GIL, every reference count modification would need locking, creating massive overhead and making single-threaded programs slower. The GIL is a pragmatic tradeoff: one global lock is cheaper than fine-grained locking on every object.

**The tradeoff:** The GIL prevents true parallelism in CPU-bound multi-threaded code but works fine for I/O-bound operations (since threads release the GIL during I/O waits).

## 2. Threading vs Multiprocessing

**Use threading when:**

- Your workload is I/O-bound (network requests, file I/O, database queries). Threads excel here because they release the GIL during I/O operations.
- You need shared memory and lightweight communication between workers
- Your application is already event-driven

**Use multiprocessing when:**

- Your workload is CPU-bound (data processing, calculations, image manipulation). Each process has its own Python interpreter and GIL, enabling true parallelism.
- You need true isolation between workers (one crashing won't affect others)
- You can tolerate higher overhead (processes are heavier than threads)

**Example comparison:**

```python
import threading
import multiprocessing
import time

def cpu_bound(n):
    total = 0
    for i in range(n):
        total += i
    return total

def io_bound():
    time.sleep(1)  # Simulates I/O
    return "done"

# Threading wins for I/O-bound
# Multiprocessing wins for CPU-bound
```

## 3. Python's Garbage Collection

Python uses a two-level garbage collection system:

**Level 1: Reference counting** is the primary mechanism. Each object has a reference count; when it reaches zero, the object is immediately deallocated. This is automatic and happens continuously.

**Level 2: Cycle detection** handles circular references that reference counting can't clean up. Python has a garbage collector module that runs periodically (or can be triggered manually) to detect and collect objects in reference cycles.

```python
import gc

# Check how many objects are in each generation
print(gc.get_count())  # (objects in gen0, gen1, gen2)

# Manually trigger collection
gc.collect()

# Disable automatic collection if needed
gc.disable()
```

**Generations:** Python's garbage collector uses generational collection—objects are grouped into three generations (0, 1, 2). Newer objects are checked more frequently; the assumption is that most objects die young.

## 4. Closures

A closure is a function that captures variables from its enclosing scope. The inner function "closes over" variables, maintaining access to them even after the outer function returns.

```python
def outer(x):
    def inner(y):
        return x + y  # 'x' is captured from outer scope
    return inner

add_five = outer(5)
print(add_five(3))  # Output: 8

# The closure captured 'x=5'
print(add_five.__closure__[0].cell_contents)  # 8 (shows captured value)
```

**Practical example - decorator factory:**

```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}")

greet("Alice")  # Prints greeting 3 times
```

## 5. Decorators

Decorators are functions that wrap another function or class, modifying its behavior without permanently changing it. They're implemented using closures.

**Simple decorator:**

```python
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    import time
    time.sleep(1)

slow_function()  # Prints: slow_function took 1.0001 seconds
```

**Decorator with arguments:**

```python
def retry(max_attempts):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed, retrying...")
        return wrapper
    return decorator

@retry(max_attempts=3)
def unreliable_api_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("API unavailable")
    return "Success!"
```

**Practical example - validation decorator:**

```python
def validate_types(**type_checks):
    def decorator(func):
        def wrapper(*args, **kwargs):
            # Combine args with their parameter names
            sig = func.__code__.co_varnames[:func.__code__.co_argcount]
            for name, value in zip(sig, args):
                if name in type_checks and not isinstance(value, type_checks[name]):
                    raise TypeError(f"{name} must be {type_checks[name]}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_types(name=str, age=int)
def create_user(name, age):
    return f"User: {name}, Age: {age}"

create_user("Alice", 30)  # Works
create_user("Bob", "25")  # Raises TypeError
```

## 6. Method Resolution Order (MRO)

MRO defines the order in which Python looks up attributes and methods in a hierarchy of classes, particularly important with multiple inheritance. Python uses C3 linearization algorithm.

```python
class A:
    def method(self):
        print("A")

class B(A):
    def method(self):
        print("B")

class C(A):
    def method(self):
        print("C")

class D(B, C):
    pass

# Check the MRO
print(D.mro())
# Output: [<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>]

# Or use __mro__
print(D.__mro__)

d = D()
d.method()  # Output: B (follows MRO)
```

**The MRO rule:** Start with the class itself, then follow left-to-right order in the inheritance list, but don't visit a class until all its parents have been visited. This prevents the "diamond problem."

**Using super() with MRO:**

```python
class A:
    def greet(self):
        print("Hello from A")

class B(A):
    def greet(self):
        super().greet()
        print("Hello from B")

class C(A):
    def greet(self):
        super().greet()
        print("Hello from C")

class D(B, C):
    pass

d = D()
d.greet()
# Output:
# Hello from A
# Hello from C
# Hello from B
```

## 7. `__init__` vs `__new__`

**`__new__`** is responsible for creating a new instance of the class. It's a static method that receives the class as its first argument and returns a new instance.

**`__init__`** initializes the instance after it's been created. It receives the instance as `self` and modifies it.

```python
class MyClass:
    def __new__(cls, value):
        print(f"__new__ called with {value}")
        instance = super().__new__(cls)  # Create the instance
        return instance

    def __init__(self, value):
        print(f"__init__ called with {value}")
        self.value = value

obj = MyClass(42)
# Output:
# __new__ called with 42
# __init__ called with 42
```

**Practical example - singleton pattern:**

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True - same instance
```

**Immutable object example:**

```python
class ImmutablePoint:
    def __new__(cls, x, y):
        instance = super().__new__(cls)
        object.__setattr__(instance, 'x', x)
        object.__setattr__(instance, 'y', y)
        return instance

    def __setattr__(self, name, value):
        raise AttributeError("Cannot modify immutable object")

p = ImmutablePoint(1, 2)
p.x = 5  # Raises AttributeError
```

## 8. Context Managers

Context managers implement resource management using the `with` statement. They guarantee cleanup code runs, even if exceptions occur.

**How they work:** A context manager implements `__enter__()` (called when entering the `with` block) and `__exit__()` (called when exiting, even on exception).

```python
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None

    def __enter__(self):
        print(f"Opening {self.filename}")
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"Closing {self.filename}")
        if self.file:
            self.file.close()
        # Return False to propagate exceptions, True to suppress

with FileManager('test.txt', 'w') as f:
    f.write("Hello")
# Output:
# Opening test.txt
# Closing test.txt
```

**Using contextlib decorator:**

```python
from contextlib import contextmanager

@contextmanager
def database_connection(url):
    print(f"Connecting to {url}")
    connection = "fake_connection"  # Simulate connection
    try:
        yield connection
    finally:
        print(f"Closing connection")

with database_connection("db://localhost") as conn:
    print(f"Using {conn}")
# Output:
# Connecting to db://localhost
# Using fake_connection
# Closing connection
```

**Practical example - timer context manager:**

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(name):
    start = time.time()
    try:
        yield
    finally:
        elapsed = time.time() - start
        print(f"{name} took {elapsed:.4f} seconds")

with timer("Data processing"):
    time.sleep(1.5)
# Output: Data processing took 1.5001 seconds
```

## 9. The LEGB Rule for Variable Scope

LEGB defines the order Python searches for variables: **Local → Enclosing → Global → Built-in**.

**Local (L):** Inside a function.

**Enclosing (E):** In the outer function if nested.

**Global (G):** At module level.

**Built-in (B):** Python's built-in namespace.

```python
x = "global"  # Global scope

def outer():
    x = "enclosing"  # Enclosing scope

    def inner():
        x = "local"  # Local scope
        print(x)  # Prints "local"

    inner()
    print(x)  # Prints "enclosing"

print(x)  # Prints "global"
outer()
```

**Modifying variables at different scopes:**

```python
x = "global"

def modify_scopes():
    x = "local"

    def inner():
        nonlocal x  # Modify enclosing scope
        x = "modified enclosing"

    inner()
    print(x)  # Prints "modified enclosing"

modify_scopes()
print(x)  # Prints "global" (unchanged)

# Modifying global
y = "original global"

def modify_global():
    global y  # Modify global scope
    y = "modified global"

modify_global()
print(y)  # Prints "modified global"
```

**The `global` and `nonlocal` keywords:**

Use `global` when you want to modify a variable in the global scope from inside a function. Use `nonlocal` to modify a variable in the enclosing scope from inside a nested function. Without these keywords, assignment creates a new local variable.
