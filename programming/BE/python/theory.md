# Python Theory Interview Preparation Guide

## 1. Threading & Concurrency

### Single-threaded vs Multi-threaded

**Single-threaded:**

- One thread executes code sequentially
- Simpler to debug and reason about
- Cannot utilize multiple CPU cores effectively
- No race conditions or deadlocks

**Multi-threaded:**

- Multiple threads share the same process memory
- Can improve responsiveness (I/O-bound operations)
- Requires synchronization mechanisms
- Subject to race conditions and deadlocks

### Python's Global Interpreter Lock (GIL)

**What is it?**

- A mutex that protects access to Python objects in CPython
- Only one thread can execute Python bytecode at a time
- Prevents true parallelism on multi-core systems for CPU-bound tasks

**Why does it exist?**

- Simplifies memory management (reference counting)
- Makes single-threaded code faster
- Reduces need for fine-grained locks

**Implications:**

- Multi-threading in Python works well for I/O-bound tasks (network, file operations)
- CPU-bound tasks don't benefit from multi-threading; use multiprocessing instead
- Threads can still improve responsiveness by allowing I/O operations to occur concurrently

**Key Interview Points:**

- Threading is good for I/O-bound work, multiprocessing for CPU-bound work
- GIL doesn't block on I/O operations (releases during system calls)
- Async/await is another alternative for I/O-bound concurrency

### Thread Synchronization

**Lock/Mutex:**

```python
import threading
lock = threading.Lock()
with lock:
    # critical section
    shared_resource += 1
```

**RLock (Reentrant Lock):**

- Can be acquired multiple times by the same thread
- Useful when recursive functions access shared resources

**Semaphore:**

- Maintains a counter; useful for limiting resource access
- `acquire()` decrements counter, `release()` increments it
- Blocks when counter reaches zero

**Event:**

- Simple flag for signaling between threads
- One thread sets it, others wait for it

**Condition Variable:**

- Combines lock with waiting mechanism
- Allows threads to wait for specific conditions

### Common Interview Questions on Threading

**Q: Why is the GIL bad? What alternatives exist?**

- Multiprocessing: separate processes, each with own GIL (true parallelism for CPU-bound)
- Async/await: cooperative multitasking, no true parallelism but efficient I/O handling
- C extensions that release GIL (NumPy, etc.)

**Q: When should I use threading vs multiprocessing?**

- Threading: I/O-bound tasks, need shared memory, lighter weight
- Multiprocessing: CPU-bound tasks, isolation between processes, true parallelism

---

## 2. Memory Management & Pointers

### Python doesn't have explicit pointers, but...

**References (Python's equivalent):**

- Everything in Python is an object
- Variables are references to objects, not the objects themselves
- No pointer arithmetic or pointer types visible to the user

```python
a = [1, 2, 3]
b = a  # b references the same list object as a
b.append(4)
print(a)  # [1, 2, 3, 4] - both refer to same object
```

### Memory Model

**Stack vs Heap:**

- Stack: local variables, function calls, reference storage (fast, limited size)
- Heap: actual objects (large, dynamic, slower access)

**Object ID and Identity:**

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a
print(id(a) == id(c))  # True (same object)
print(id(a) == id(b))  # False (different objects, same content)
print(a is c)          # True (identity check)
print(a == b)          # True (equality check)
```

### Reference Counting & Garbage Collection

**Reference Counting:**

- Every object has a reference count
- Decremented when references go out of scope
- Object deleted when count reaches zero
- Fast but can't handle circular references

**Garbage Collection:**

- Handles circular references (objects referencing each other)
- Runs periodically, not immediately
- Can be controlled with `gc` module

```python
import gc
gc.collect()  # Force garbage collection
gc.disable()  # Disable automatic garbage collection
gc.enable()   # Re-enable garbage collection
```

**Circular Reference Example:**

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

a = Node(1)
b = Node(2)
a.next = b
b.next = a  # Circular reference - GC needed to clean up
```

### Deep Copy vs Shallow Copy

**Shallow Copy:**

- Creates new object but references same nested objects
- Modifying nested objects affects both copies

```python
import copy
original = [[1, 2], [3, 4]]
shallow = copy.copy(original)
shallow[0][0] = 999
print(original)  # [[999, 2], [3, 4]] - nested list modified
```

**Deep Copy:**

- Creates new object and recursively copies all nested objects
- Modifications don't affect original

```python
deep = copy.deepcopy(original)
deep[0][0] = 999
print(original)  # [[1, 2], [3, 4]] - original unchanged
```

### Memory Leaks in Python

- Reference cycles can prevent garbage collection (if GC is disabled)
- Keeping large objects in scope (caching issues)
- Listeners/callbacks not being unregistered

---

## 3. Data Types & Mutability

### Mutable vs Immutable

**Immutable (cannot be changed):**

- int, float, str, tuple, frozenset, bytes
- Creating "new" versions creates new objects
- Can be used as dictionary keys or in sets

```python
x = 5
y = x
x = 10  # Creates new int object, y still references 5

s = "hello"
s = s + " world"  # Creates new string object
```

**Mutable (can be changed):**

- list, dict, set, bytearray, custom objects
- Cannot be used as dictionary keys (hashable requirement)
- Changes affect all references to that object

```python
lst = [1, 2, 3]
lst2 = lst
lst.append(4)
print(lst2)  # [1, 2, 3, 4] - both reference same list
```

**Key Interview Point:**

- Immutable objects are thread-safe by nature
- Mutable objects need synchronization in multi-threaded code

### String Interning (CPython Optimization)

```python
a = "hello"
b = "hello"
print(a is b)  # True - same object (string interning)

a = "hello world"
b = "hello world"
print(a is b)  # May be False - not interned (contains space)
```

---

## 4. Scope & Namespaces

### LEGB Rule

Variables are looked up in this order:

1. **Local** - inside current function
2. **Enclosing** - in outer functions (closures)
3. **Global** - module level
4. **Built-in** - Python's built-in namespace

```python
x = 'global'

def outer():
    x = 'enclosing'
    def inner():
        x = 'local'
        print(x)  # Prints 'local'
    inner()
    print(x)      # Prints 'enclosing'

outer()
print(x)          # Prints 'global'
```

### Global and Nonlocal

```python
x = 'global'

def func():
    global x
    x = 'modified'  # Changes global x

def outer():
    x = 'enclosing'
    def inner():
        nonlocal x
        x = 'modified enclosing'  # Changes enclosing x
    inner()
    return x

print(func())      # Modifies global
print(outer())     # Prints 'modified enclosing'
```

---

## 5. Object-Oriented Programming Concepts

### Class vs Instance

```python
class Dog:
    species = 'Canis familiaris'  # Class variable (shared)

    def __init__(self, name):
        self.name = name  # Instance variable (unique per instance)

    def speak(self):
        return f"{self.name} barks"

d1 = Dog("Rex")
d2 = Dog("Max")
print(d1.name)  # Rex
print(d2.name)  # Max
print(Dog.species)  # Canis familiaris
```

**Class variables:** Shared across all instances
**Instance variables:** Unique to each instance

### Inheritance & Method Resolution Order (MRO)

```python
class Animal:
    def speak(self):
        return "Sound"

class Dog(Animal):
    def speak(self):
        return "Woof"

class Cat(Animal):
    def speak(self):
        return "Meow"

class Hybrid(Dog, Cat):
    pass

h = Hybrid()
print(h.speak())  # Woof (follows MRO: Hybrid -> Dog -> Cat -> Animal)
print(Hybrid.__mro__)  # Shows method resolution order
```

**C3 Linearization:** Python's MRO algorithm for multiple inheritance

### Properties and Descriptors

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius

    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9

t = Temperature()
t.fahrenheit = 32  # Calls setter
print(t.celsius)   # 0
```

### Magic Methods (**dunder** methods)

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __len__(self):
        return int((self.x**2 + self.y**2)**0.5)

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
```

---

## 6. Iterators & Generators

### Iterators

```python
class CountUp:
    def __init__(self, max):
        self.max = max
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current < self.max:
            self.current += 1
            return self.current
        else:
            raise StopIteration

for num in CountUp(3):
    print(num)  # 1, 2, 3
```

### Generators (simpler iterators)

```python
def count_up(max):
    current = 0
    while current < max:
        current += 1
        yield current

for num in count_up(3):
    print(num)  # 1, 2, 3
```

**Advantages of generators:**

- Lazy evaluation (compute on demand)
- Memory efficient (don't store all values)
- Can represent infinite sequences

### Generator Expressions

```python
squares = (x**2 for x in range(10))  # Generator expression
squares_list = [x**2 for x in range(10)]  # List comprehension
```

---

## 7. Exception Handling

### Try-Except-Finally

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
except Exception as e:
    print(f"Unexpected error: {e}")
else:
    print("No exception occurred")
finally:
    print("Always executed")
```

**Key Points:**

- Multiple except blocks catch different exceptions
- else block runs if no exception
- finally always runs (cleanup)
- Order matters: catch specific exceptions before general ones

### Context Managers

```python
class FileHandler:
    def __init__(self, filename):
        self.filename = filename
        self.file = None

    def __enter__(self):
        self.file = open(self.filename, 'r')
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        return False  # Don't suppress exceptions

with FileHandler('data.txt') as f:
    data = f.read()
```

**Benefits:** Automatic resource cleanup, exception handling

---

## 8. Collections & Data Structures

### List vs Tuple vs Set vs Dict

| Feature    | List | Tuple | Set | Dict                  |
| ---------- | ---- | ----- | --- | --------------------- |
| Mutable    | Yes  | No    | Yes | Yes                   |
| Ordered    | Yes  | Yes   | No  | Yes (3.7+)            |
| Hashable   | No   | Yes\* | No  | No                    |
| Duplicates | Yes  | Yes   | No  | Keys: No, Values: Yes |
| Indexable  | Yes  | Yes   | No  | No                    |

\*Tuples are hashable only if all elements are hashable

### Collections Module

```python
from collections import defaultdict, Counter, namedtuple, deque

# defaultdict
dd = defaultdict(list)
dd['key'].append(1)  # No KeyError

# Counter
c = Counter(['a', 'b', 'a', 'c', 'a'])
print(c)  # Counter({'a': 3, 'b': 1, 'c': 1})

# namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
print(p.x)  # 1

# deque
d = deque([1, 2, 3])
d.appendleft(0)
d.pop()
```

---

## 9. Function & Decorator Concepts

### First-Class Functions

```python
def add(a, b):
    return a + b

def apply_operation(func, x, y):
    return func(x, y)

result = apply_operation(add, 2, 3)  # 5
# Functions can be passed as arguments
```

### Closures

```python
def make_multiplier(n):
    def multiplier(x):
        return x * n
    return multiplier

times_three = make_multiplier(3)
print(times_three(10))  # 30
```

### Decorators

```python
def timing_decorator(func):
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"Took {end - start} seconds")
        return result
    return wrapper

@timing_decorator
def slow_function():
    import time
    time.sleep(1)

slow_function()
```

### \*args and \*\*kwargs

```python
def func(*args, **kwargs):
    print(args)     # tuple of positional arguments
    print(kwargs)   # dict of keyword arguments

func(1, 2, 3, name='Alice', age=30)
# args = (1, 2, 3)
# kwargs = {'name': 'Alice', 'age': 30}
```

---

## 10. Common Pitfalls

### Mutable Default Arguments

```python
# ❌ WRONG
def append_to(element, to=[]):
    to.append(element)
    return to

append_to(1)  # [1]
append_to(2)  # [1, 2] - same list!

# ✅ CORRECT
def append_to(element, to=None):
    if to is None:
        to = []
    to.append(element)
    return to
```

### Late Binding in Closures

```python
# ❌ WRONG
functions = []
for i in range(3):
    functions.append(lambda x: x + i)

print(functions[0](1))  # 3 (i=2, not 0!)

# ✅ CORRECT
functions = []
for i in range(3):
    functions.append(lambda x, i=i: x + i)  # Default captures i

print(functions[0](1))  # 1
```

### Is vs ==

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)   # True (same content)
print(a is b)   # False (different objects)
print(a is c)   # True (same object)

# For singletons:
x = None
y = None
print(x is y)   # True (None is a singleton)
```

---

## 11. Performance & Optimization

### Big O Complexity

| Operation | List | Dict | Set  | Tuple |
| --------- | ---- | ---- | ---- | ----- |
| Access    | O(1) | O(1) | N/A  | O(1)  |
| Search    | O(n) | O(1) | O(1) | O(n)  |
| Insert    | O(n) | O(1) | O(1) | N/A   |
| Delete    | O(n) | O(1) | O(1) | N/A   |

### String Concatenation

```python
# ❌ SLOW - creates new string each time
result = ""
for i in range(1000):
    result += str(i)  # O(n^2)

# ✅ FAST
result = "".join(str(i) for i in range(1000))  # O(n)
```

### List Comprehension vs Loop

```python
# ✅ FASTER - list comprehension
squares = [x**2 for x in range(1000)]

# ❌ SLOWER - explicit loop
squares = []
for x in range(1000):
    squares.append(x**2)
```

---

## 12. Testing & Debugging

### Unit Testing

```python
import unittest

class TestMath(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(2 + 2, 4)

    def test_division_by_zero(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0

if __name__ == '__main__':
    unittest.main()
```

### Debugging Tools

```python
# Print debugging
print(f"Variable x: {x}, type: {type(x)}")

# pdb debugger
import pdb
pdb.set_trace()  # Breakpoint
# Commands: n (next), s (step), c (continue), p (print)

# Assertions
assert x > 0, "x must be positive"

# Logging
import logging
logging.debug("Debug message")
logging.error("Error message")
```

---

## Practice Questions

1. Explain the GIL and why it exists
2. When would you use threading vs multiprocessing?
3. What's the difference between shallow and deep copy?
4. Explain the difference between `is` and `==`
5. What are mutable vs immutable objects? Give examples
6. How does Python's garbage collection work?
7. What is a closure? Provide an example
8. Explain decorators and provide a practical example
9. What's the difference between a generator and a list?
10. How would you handle mutable default arguments safely?
11. Explain MRO (Method Resolution Order) in Python
12. What's the difference between `__init__` and `__new__`?
13. How do context managers work?
14. What are properties and when would you use them?
15. Explain the LEGB rule for variable scope

# Python Interview: Practical Code Examples

## 1. Threading Examples

### Basic Threading

```python
import threading
import time

def worker(name, duration):
    print(f"{name} starting at {time.time()}")
    time.sleep(duration)
    print(f"{name} finished at {time.time()}")

# Create threads
t1 = threading.Thread(target=worker, args=("Thread-1", 2))
t2 = threading.Thread(target=worker, args=("Thread-2", 1))

# Start threads
t1.start()
t2.start()

# Wait for completion
t1.join()
t2.join()

print("All threads completed")
```

### Thread Synchronization with Lock

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:  # Critical section
            counter += 1

# Without lock, counter might be < 200000 due to race conditions
t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)

t1.start()
t2.start()
t1.join()
t2.join()

print(f"Final counter: {counter}")  # Should be exactly 200000
```

### GIL Demonstration: I/O Bound vs CPU Bound

```python
import threading
import time
import requests
from multiprocessing import Pool

# I/O-BOUND: Threading helps
def fetch_urls():
    urls = ['http://httpbin.org/delay/1'] * 5

    def fetch(url):
        try:
            requests.get(url, timeout=5)
        except:
            pass

    # Single-threaded
    start = time.time()
    for url in urls:
        fetch(url)
    print(f"Single-threaded I/O: {time.time() - start:.2f}s")

    # Multi-threaded
    start = time.time()
    threads = [threading.Thread(target=fetch, args=(url,)) for url in urls]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    print(f"Multi-threaded I/O: {time.time() - start:.2f}s")

# CPU-BOUND: Multiprocessing helps
def cpu_intensive():
    def fibonacci(n):
        if n < 2:
            return n
        return fibonacci(n-1) + fibonacci(n-2)

    test_cases = [35] * 4

    # Single-threaded
    start = time.time()
    for case in test_cases:
        fibonacci(case)
    print(f"Single-threaded CPU: {time.time() - start:.2f}s")

    # Multi-threaded (slower due to GIL)
    start = time.time()
    threads = [threading.Thread(target=fibonacci, args=(case,)) for case in test_cases]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    print(f"Multi-threaded CPU: {time.time() - start:.2f}s")

    # Multiprocessing (faster - true parallelism)
    start = time.time()
    with Pool(4) as p:
        p.map(fibonacci, test_cases)
    print(f"Multi-process CPU: {time.time() - start:.2f}s")

# Uncomment to test:
# fetch_urls()
# cpu_intensive()
```

### Semaphore Example (Rate Limiting)

```python
import threading
import time

# Limit to 3 simultaneous downloads
semaphore = threading.Semaphore(3)

def download(file_id):
    with semaphore:
        print(f"Downloading file {file_id} at {time.time():.2f}")
        time.sleep(2)
        print(f"Finished file {file_id} at {time.time():.2f}")

threads = [threading.Thread(target=download, args=(i,)) for i in range(9)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

---

## 2. Memory & Reference Examples

### Reference vs Copy

```python
# Reference behavior
a = [1, 2, 3]
b = a  # b references same list
a.append(4)
print(b)  # [1, 2, 3, 4]

# With immutables
x = 5
y = x
x = 10
print(y)  # Still 5 (integers are immutable)

# String interning
s1 = "hello"
s2 = "hello"
print(s1 is s2)  # True (interned by Python)

s3 = "hel" + "lo"
print(s1 is s3)  # May be True or False (implementation detail)
```

### Object Identity with id()

```python
import sys

class MyObject:
    def __init__(self, value):
        self.value = value

obj1 = MyObject(10)
obj2 = MyObject(10)
obj3 = obj1

print(f"obj1 id: {id(obj1)}")
print(f"obj2 id: {id(obj2)}")
print(f"obj3 id: {id(obj3)}")

print(f"obj1 is obj3: {obj1 is obj3}")  # True - same object
print(f"obj1 == obj2: {obj1 == obj2}")  # False (no __eq__ defined)
print(f"obj1 is obj2: {obj1 is obj2}")  # False - different objects

# Reference counting
print(f"Reference count for obj1: {sys.getrefcount(obj1)}")
```

### Shallow vs Deep Copy

```python
import copy

original = [[1, 2], [3, 4]]

# Shallow copy
shallow = copy.copy(original)
shallow[0][0] = 999
print(f"Original after shallow copy modification: {original}")
# Output: [[999, 2], [3, 4]] - nested list changed!

# Deep copy
original = [[1, 2], [3, 4]]
deep = copy.deepcopy(original)
deep[0][0] = 999
print(f"Original after deep copy modification: {original}")
# Output: [[1, 2], [3, 4]] - original unchanged
```

### Circular References and Garbage Collection

```python
import gc
import sys

class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Create circular reference
a = Node(1)
b = Node(2)
a.next = b
b.next = a

print(f"Before deletion, objects: {gc.get_count()}")

del a
del b

print(f"After deletion, objects: {gc.get_count()}")
gc.collect()  # Garbage collect circular references
print(f"After gc.collect(), objects: {gc.get_count()}")

# Check what's in garbage (if collected)
gc.set_debug(gc.DEBUG_SAVEALL)
gc.collect()
print(f"Garbage objects: {len(gc.garbage)}")
```

---

## 3. Scope and Closure Examples

### LEGB Rule Demonstration

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(f"Inside inner: {x}")

    inner()
    print(f"Inside outer: {x}")

outer()
print(f"At module level: {x}")

# Output:
# Inside inner: local
# Inside outer: enclosing
# At module level: global
```

### Closure with Variable Capture

```python
# Problem: Late binding
def make_functions():
    functions = []
    for i in range(3):
        functions.append(lambda x: x + i)  # i is looked up at call time
    return functions

funcs = make_functions()
print(funcs[0](10))  # 12 (i is 2 by the time lambda is called)
print(funcs[1](10))  # 12
print(funcs[2](10))  # 12

# Solution: Use default argument to capture current value
def make_functions_fixed():
    functions = []
    for i in range(3):
        functions.append(lambda x, i=i: x + i)  # i is captured as default
    return functions

funcs = make_functions_fixed()
print(funcs[0](10))  # 10 (captured i=0)
print(funcs[1](10))  # 11 (captured i=1)
print(funcs[2](10))  # 12 (captured i=2)
```

### Global and Nonlocal Keywords

```python
global_var = "global"

def outer():
    enclosing_var = "enclosing"

    def inner():
        nonlocal enclosing_var
        global global_var

        global_var = "modified global"
        enclosing_var = "modified enclosing"

    inner()
    print(f"After inner: {enclosing_var}")  # modified enclosing

outer()
print(f"Global: {global_var}")  # modified global
```

---

## 4. Iterator and Generator Examples

### Iterator Implementation

```python
class CountUp:
    def __init__(self, max):
        self.max = max
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current < self.max:
            self.current += 1
            return self.current
        else:
            raise StopIteration

for num in CountUp(3):
    print(num)  # 1, 2, 3

# Or use next() manually
counter = CountUp(3)
print(next(counter))  # 1
print(next(counter))  # 2
print(next(counter))  # 3
# print(next(counter))  # StopIteration
```

### Generator vs List Comprehension

```python
# Generator - lazy evaluation, memory efficient
def gen_range(n):
    for i in range(n):
        print(f"Generating {i}")
        yield i

print("Generator:")
g = gen_range(3)
print(next(g))  # Generates 0 immediately
print(next(g))  # Generates 1 immediately

# List comprehension - evaluates all at once
print("\nList comprehension:")
l = [i for i in range(3)]
print(l[0])  # All generated at once

# Generator expression
gen_expr = (x**2 for x in range(1000000))
print(f"Generator size: {sys.getsizeof(gen_expr)}")  # Small

list_comp = [x**2 for x in range(1000000)]
print(f"List size: {sys.getsizeof(list_comp)}")  # Large
```

### Infinite Generator

```python
def infinite_sequence():
    num = 0
    while True:
        yield num
        num += 1

gen = infinite_sequence()
print([next(gen) for _ in range(5)])  # [0, 1, 2, 3, 4]
```

---

## 5. Decorator Examples

### Simple Timing Decorator

```python
import time
from functools import wraps

def timing(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} took {elapsed:.4f} seconds")
        return result
    return wrapper

@timing
def slow_function(n):
    time.sleep(n)
    return n

result = slow_function(1)
```

### Decorator with Arguments

```python
from functools import wraps

def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    return f"Hello {name}"

print(greet("Alice"))  # ['Hello Alice', 'Hello Alice', 'Hello Alice']
```

### Class-Based Decorator

```python
class Logger:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print(f"Calling {self.func.__name__}")
        result = self.func(*args, **kwargs)
        print(f"Finished {self.func.__name__}")
        return result

@Logger
def add(a, b):
    return a + b

print(add(2, 3))
```

---

## 6. OOP Concepts

### Magic Methods

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __len__(self):
        return int((self.x**2 + self.y**2)**0.5)

    def __getitem__(self, index):
        return [self.x, self.y][index]

v1 = Vector(3, 4)
v2 = Vector(1, 2)

print(repr(v1))      # Vector(3, 4)
print(str(v1))       # (3, 4)
print(v1 + v2)       # Vector(4, 6)
print(v1 * 2)        # Vector(6, 8)
print(len(v1))       # 5 (magnitude)
print(v1[0])         # 3
```

### Properties and Descriptors

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9

    @property
    def kelvin(self):
        return self._celsius + 273.15

t = Temperature()
print(t.celsius)      # 0
print(t.fahrenheit)   # 32.0

t.fahrenheit = 32
print(t.celsius)      # 0

t.celsius = 100
print(t.fahrenheit)   # 212.0
```

### Inheritance and MRO

```python
class Animal:
    def speak(self):
        return "Some sound"

class Mammal(Animal):
    def speak(self):
        return "Mammal sound"

class Dog(Mammal):
    def speak(self):
        return "Woof"

class Cat(Mammal):
    def speak(self):
        return "Meow"

class Hybrid(Dog, Cat):
    pass

h = Hybrid()
print(h.speak())           # Woof (Dog comes before Cat)
print(Hybrid.__mro__)      # Shows resolution order
# (<class 'Hybrid'>, <class 'Dog'>, <class 'Cat'>, <class 'Mammal'>, <class 'Animal'>, <class 'object'>)
```

### Context Managers

```python
class File:
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
        if exc_type:
            print(f"Exception: {exc_type.__name__}: {exc_val}")
        return False  # Don't suppress exceptions

with File("test.txt", "w") as f:
    f.write("Hello World")

# Simpler way with contextlib
from contextlib import contextmanager

@contextmanager
def open_file(filename, mode):
    f = open(filename, mode)
    try:
        yield f
    finally:
        f.close()

with open_file("test.txt", "r") as f:
    print(f.read())
```

---

## 7. Common Pitfalls

### Mutable Default Arguments

```python
# ❌ WRONG
def append_to_list(element, to=[]):
    to.append(element)
    return to

print(append_to_list(1))  # [1]
print(append_to_list(2))  # [1, 2] - same list!
print(append_to_list(3))  # [1, 2, 3]

# ✅ CORRECT
def append_to_list_fixed(element, to=None):
    if to is None:
        to = []
    to.append(element)
    return to

print(append_to_list_fixed(1))  # [1]
print(append_to_list_fixed(2))  # [2]
print(append_to_list_fixed(3))  # [3]
```

### Using mutable object as dict key

```python
# ❌ WRONG - TypeError
my_dict = {}
my_list = [1, 2, 3]
my_dict[my_list] = "value"  # TypeError: unhashable type

# ✅ CORRECT - use tuple
my_dict = {}
my_tuple = (1, 2, 3)
my_dict[my_tuple] = "value"  # Works fine
```

### Division by Integer (Python 2 vs 3)

```python
# Python 3: / always returns float
print(5 / 2)    # 2.5
print(5 // 2)   # 2 (floor division)
print(5 % 2)    # 1 (modulo)
```

---

## 8. Performance Comparison

### String Concatenation

```python
import time

n = 10000

# ❌ SLOW
start = time.time()
result = ""
for i in range(n):
    result += str(i)
slow_time = time.time() - start

# ✅ FAST
start = time.time()
result = "".join(str(i) for i in range(n))
fast_time = time.time() - start

print(f"String concat: {slow_time:.4f}s")
print(f"Join method: {fast_time:.4f}s")
print(f"Speedup: {slow_time/fast_time:.1f}x")
```

### List Operations

```python
import timeit

# Append
print(timeit.timeit("[].append(1)", number=100000))

# List comprehension vs loop
print(timeit.timeit("[x for x in range(100)]", number=100000))
print(timeit.timeit("l = []; [l.append(x) for x in range(100)]", number=100000))

# Set vs List lookup
print(timeit.timeit("50 in list(range(100))", number=100000))
print(timeit.timeit("50 in set(range(100))", number=100000))
```
