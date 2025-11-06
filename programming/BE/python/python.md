# Python Interview Questions & Answers

## 1. What is Python and why is it popular?

**Q: Explain Python**

A: Python is an interpreted, dynamically-typed, high-level programming language emphasizing code readability.

**Why it's popular**:

- Simple, readable syntax
- Beginner-friendly
- Powerful standard library
- Versatile (web, data science, automation, AI/ML)
- Large community
- Fast development cycle

**Use cases**: Web (Django, Flask), Data science (NumPy, Pandas), ML (TensorFlow), Automation, DevOps, Testing

---

## 2. Python 2 vs Python 3

**Q: Key differences**

A: Python 3 is current standard. Python 2 is deprecated (EOL 2020).

**Key differences**:

- `print "text"` → `print("text")`
- `/` integer division → `/` float division (use `//` for int)
- `unicode` → `str` is unicode by default
- Exception `except Exception, e` → `except Exception as e`
- Type hints supported in Python 3

**Use Python 3** — always.

---

## 3. What are data types in Python?

**Q: Explain data types**

A: Python has built-in types.

**Numeric**: `int`, `float`, `complex`
**Sequence**: `str`, `list`, `tuple`, `range`
**Mapping**: `dict`
**Set**: `set`, `frozenset`
**Boolean**: `True`, `False`
**Special**: `None`

```python
type(42)           # <class 'int'>
isinstance(42, int)  # True
```

---

## 4. List vs Tuple vs Set

**Q: Differences**

A:

| List        | Tuple       | Set           |
| ----------- | ----------- | ------------- |
| `[1, 2, 3]` | `(1, 2, 3)` | `{1, 2, 3}`   |
| Mutable     | Immutable   | Mutable       |
| Ordered     | Ordered     | Unordered     |
| Duplicates  | Duplicates  | No duplicates |

```python
lst = [1, 2, 3]
lst[0] = 99  # Works

tup = (1, 2, 3)
tup[0] = 99  # TypeError

st = {1, 2, 2, 3}  # {1, 2, 3}
1 in st  # Fast lookup
```

---

## 5. What are list comprehensions?

**Q: Explain list comprehensions**

A: Concise way to create lists.

```python
# Traditional
squares = []
for i in range(10):
    squares.append(i ** 2)

# Comprehension
squares = [i ** 2 for i in range(10)]

# With condition
even_squares = [i ** 2 for i in range(10) if i % 2 == 0]

# Dict comprehension
d = {i: i**2 for i in range(5)}

# Set comprehension
unique = {i % 3 for i in range(10)}
```

---

## 6. What are functions?

**Q: Explain functions**

A:

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("John")  # Uses default greeting
```

**Variable arguments**:

```python
def sum_numbers(*args):
    return sum(args)

def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

def func(a, b, *args, **kwargs):
    pass
```

**Unpacking**:

```python
args = (1, 2, 3)
func(*args)

kwargs = {'name': 'John'}
func(**kwargs)
```

---

## 7. What are lambda functions?

**Q: Explain lambdas**

A: Single-expression anonymous function.

```python
add = lambda a, b: a + b
add(1, 2)  # 3

# Common uses
numbers = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))
sorted(students, key=lambda x: x[1])  # Sort by age
```

---

## 8. What are decorators?

**Q: Explain decorators**

A: Function that modifies another function.

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@decorator
def greet(name):
    return f"Hello, {name}!"

greet("John")
# Calling greet
# Hello, John!
```

**Timing decorator**:

```python
import time

def time_it(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"Took {time.time() - start} seconds")
        return result
    return wrapper

@time_it
def slow_function():
    time.sleep(2)
```

---

## 9. What are classes and objects?

**Q: Explain OOP**

A:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hello, I'm {self.name}"

p = Person("John", 30)
p.greet()  # "Hello, I'm John"
```

**Class vs Instance attributes**:

```python
class Car:
    wheels = 4  # Class attribute

    def __init__(self, brand):
        self.brand = brand  # Instance attribute
```

---

## 10. What is inheritance?

**Q: Explain inheritance**

A: Child class inherits from parent.

```python
class Animal:
    def speak(self):
        return "Some sound"

class Dog(Animal):
    def speak(self):
        return "Woof!"

dog = Dog()
dog.speak()  # "Woof!"
```

**super()** — call parent method:

```python
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)
        self.breed = breed
```

**MRO** — Method Resolution Order:

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

D.mro()  # [D, B, C, A, object]
```

---

## 11. @staticmethod and @classmethod

**Q: Explain static and class methods**

A:

**Instance method** — operates on instance (`self`):

```python
class Person:
    def greet(self):
        return f"Hello, {self.name}"
```

**@staticmethod** — no instance or class needed:

```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b

Math.add(2, 3)  # 5
```

**@classmethod** — operates on class (`cls`):

```python
class Person:
    count = 0

    def __init__(self, name):
        self.name = name
        Person.count += 1

    @classmethod
    def get_count(cls):
        return cls.count

    @classmethod
    def from_string(cls, string):
        name, age = string.split(',')
        return cls(name)

Person.get_count()  # 0
```

---

## 12. What are properties?

**Q: Explain @property decorator**

A: Make method accessible as attribute.

```python
class Person:
    def __init__(self, first_name, last_name):
        self._first_name = first_name
        self._last_name = last_name

    @property
    def full_name(self):
        return f"{self._first_name} {self._last_name}"

    @full_name.setter
    def full_name(self, name):
        first, last = name.split()
        self._first_name = first
        self._last_name = last

p = Person("John", "Doe")
print(p.full_name)  # "John Doe" — accessed as property!

p.full_name = "Jane Smith"
print(p.full_name)  # "Jane Smith"
```

---

## 13. Exception handling

**Q: How do you handle errors?**

A:

```python
try:
    x = 1 / 0
except ZeroDivisionError:
    print("Can't divide by zero")
```

**Multiple exceptions**:

```python
try:
    data = {'a': 1}
    print(data['b'])
except (KeyError, ValueError):
    print("Error")
except Exception as e:
    print(f"General error: {e}")
```

**else and finally**:

```python
try:
    x = 1 / 1
except ZeroDivisionError:
    print("Error")
else:
    print("No error")
finally:
    print("Cleanup")  # Always runs
```

**Raise exception**:

```python
def divide(a, b):
    if b == 0:
        raise ValueError("Divisor can't be zero")
    return a / b
```

---

## 14. File handling

**Q: How do you read/write files?**

A:

```python
# Read file
with open('file.txt', 'r') as f:
    content = f.read()

# Read lines
with open('file.txt', 'r') as f:
    for line in f:
        print(line.strip())

# Write file
with open('file.txt', 'w') as f:
    f.write("Hello, World!")

# Append
with open('file.txt', 'a') as f:
    f.write("\nNew line")
```

**File modes**: `'r'` (read), `'w'` (write, overwrite), `'a'` (append), `'x'` (create)

**with statement** — auto-closes file.

---

## 15. What are generators?

**Q: Explain generators**

A: Function that yields values lazily.

```python
def get_numbers():
    for i in range(1000000):
        yield i

# Memory efficient — yields one at a time
for num in get_numbers():
    print(num)
```

**Generator expression**:

```python
squares = (x**2 for x in range(1000000))
next(squares)  # Get next value
```

**Benefits**: Memory efficient, lazy evaluation, can be infinite.

---

## 16. What are context managers?

**Q: Explain with statement**

A: Context manager handles setup/cleanup.

```python
# Manual cleanup
f = open('file.txt')
content = f.read()
f.close()

# Context manager — auto-closes
with open('file.txt') as f:
    content = f.read()
```

**Custom context manager**:

```python
class MyContext:
    def __enter__(self):
        print("Setup")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Cleanup")

with MyContext():
    print("Inside")
```

---

## 17. Iterables vs iterators

**Q: Difference**

A:

**Iterable** — object you can iterate over:

```python
my_list = [1, 2, 3]  # Iterable
for item in my_list:
    print(item)
```

**Iterator** — object that produces values:

```python
iterator = iter([1, 2, 3])
next(iterator)  # 1
next(iterator)  # 2
```

**Create custom iterator**:

```python
class CountUp:
    def __init__(self, max):
        self.max = max
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        self.current += 1
        if self.current > self.max:
            raise StopIteration
        return self.current
```

---

## 18. \*args and \*\*kwargs

**Q: Variable arguments**

A:

```python
def func(*args):
    print(args)  # Tuple

func(1, 2, 3)  # (1, 2, 3)

def func(**kwargs):
    print(kwargs)  # Dict

func(name="John", age=30)  # {'name': 'John', 'age': 30}
```

**Both**:

```python
def func(a, *args, **kwargs):
    pass

func(1, 2, 3, name="John")
```

**Unpacking**:

```python
args = [1, 2, 3]
func(*args)

kwargs = {'name': 'John'}
func(**kwargs)
```

---

## 19. Built-in functions

**Q: Common built-ins**

A:

```python
# Type conversion
int("42"), float("3.14"), str(42), bool(1)

# Aggregation
sum([1, 2, 3]), max([1, 2, 3]), min([1, 2, 3]), len([1, 2, 3])
all([True, True]), any([False, True])

# Iteration
range(5), enumerate(['a', 'b']), zip([1, 2], ['a', 'b'])

# Sequence operations
sorted([3, 1, 2]), reversed([1, 2, 3])
map(lambda x: x**2, [1, 2, 3])
filter(lambda x: x > 1, [1, 2, 3])

# Other
abs(-5), round(3.7), pow(2, 3), divmod(7, 3)
isinstance(5, int), type(5)
```

---

## 20. What is slicing?

**Q: Explain slicing**

A:

```python
s = "Hello World"
lst = [0, 1, 2, 3, 4, 5]

# s[start:stop:step]
s[0:5]      # "Hello"
s[6:]       # "World"
s[:5]       # "Hello"
s[::2]      # "HloWrd"
s[::-1]     # "dlroW olleH" (reverse)

lst[1:4]    # [1, 2, 3]
lst[-2:]    # [4, 5]
lst[::2]    # [0, 2, 4]
```

---

## 21. String methods

**Q: Common string methods**

A:

```python
s = "  Hello World  "

s.upper()           # "  HELLO WORLD  "
s.lower()           # "  hello world  "
s.strip()           # "Hello World"
s.replace("World", "Python")
s.split(' ')        # ['', '', 'Hello', 'World', '', '']
'-'.join(['a', 'b', 'c'])  # 'a-b-c'
s.find("World")     # 8
s.startswith("  ")  # True
"a,b,c".split(',')  # ['a', 'b', 'c']
```

---

## 22. String formatting

**Q: Format strings**

A:

```python
# f-strings (modern, preferred)
name = "John"
age = 30
f"Hello {name}, you are {age}"
f"Age: {age + 5}"

# .format()
"Hello {}, you are {}".format("John", 30)

# Old style (%)
"Hello %s, you are %d" % ("John", 30)
```

**Use f-strings** — cleaner and faster.

---

## 23. Dictionary operations

**Q: Dict methods**

A:

```python
d = {'name': 'John', 'age': 30}

d['name']           # 'John'
d.get('name')       # 'John'
d.get('missing', 'default')

d['city'] = 'NYC'
d.update({'name': 'Jane'})

del d['name']
d.pop('age')
d.popitem()

d.keys(), d.values(), d.items()

for key, value in d.items():
    print(f"{key}: {value}")

'name' in d         # True
len(d)
d.clear()
```

---

## 24. Set operations

**Q: Set operations**

A:

```python
s = {1, 2, 3}

s.add(5)
s.remove(5)
s.discard(5)
s.pop()

a = {1, 2, 3}
b = {2, 3, 4}

a | b               # Union: {1, 2, 3, 4}
a & b               # Intersection: {2, 3}
a - b               # Difference: {1}
a ^ b               # Symmetric difference: {1, 4}

a.union(b)
a.intersection(b)
a.issubset(b)
```

---

## 25. List methods

**Q: List operations**

A:

```python
lst = [3, 1, 4, 1, 5]

lst.append(9)
lst.extend([2, 6])
lst.insert(0, 10)

lst.remove(1)
lst.pop()
lst.pop(0)

lst.sort()
lst.reverse()

lst.count(1)
lst.index(1)

len(lst)
lst.clear()
```

---

## 26. Type hints

**Q: Explain type hints**

A: Optional annotations for clarity (Python 3.5+).

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

def add(a: int, b: int) -> int:
    return a + b

from typing import List, Dict, Optional

def process_list(items: List[int]) -> int:
    return sum(items)

def get_value() -> Optional[str]:
    return None
```

**Benefits**: Better IDE autocomplete, catch errors early, better documentation.

---

## 27. The pass statement

**Q: What does pass do?**

A: Null operation — placeholder.

```python
def not_implemented():
    pass

class MyClass:
    pass

if condition:
    pass
```

---

## 28. Assertions

**Q: Explain assertions**

A: Check if condition is true.

```python
def divide(a, b):
    assert b != 0, "Divisor cannot be zero"
    return a / b

divide(5, 0)  # AssertionError
```

**Disable in production**: `python -O script.py`

---

## 29. if **name** == '**main**'

**Q: What does this do?**

A: Check if script is run directly vs imported.

```python
# module.py
def greet():
    return "Hello"

if __name__ == '__main__':
    print(greet())  # Runs only if executed directly
```

**Import**: if block doesn't run.
**Run directly**: if block runs.

---

## 30. Modules and packages

**Q: Explain modules and packages**

A:

**Module** — Python file:

```python
# math_utils.py
def add(a, b):
    return a + b
```

**Import**:

```python
import math_utils
math_utils.add(1, 2)

from math_utils import add
add(1, 2)
```

**Package** — directory with **init**.py:

```
mypackage/
    __init__.py
    module1.py
    subpackage/
        __init__.py
```

**Import from package**:

```python
from mypackage import module1
from mypackage.module1 import func
```

---

## 31. Docstrings

**Q: Explain docstrings**

A: Documentation string.

```python
def add(a, b):
    """
    Add two numbers.

    Args:
        a: First number
        b: Second number

    Returns:
        Sum of a and b
    """
    return a + b

print(add.__doc__)
help(add)
```

---

## 32. global and nonlocal

**Q: Explain scope**

A:

**Local scope**:

```python
def func():
    x = 5  # Local
    print(x)
```

**Global scope**:

```python
x = 10  # Global

def func():
    print(x)  # Accesses global x
```

**Modify global**:

```python
x = 10

def modify():
    global x
    x = 20

modify()
print(x)  # 20
```

**Nested functions** (nonlocal):

```python
def outer():
    x = 10

    def inner():
        nonlocal x
        x = 20

    inner()
    print(x)  # 20
```

---

## 33. What is zip?

**Q: Explain zip**

A: Combine iterables element-wise.

```python
names = ['John', 'Jane', 'Bob']
ages = [30, 25, 35]

list(zip(names, ages))
# [('John', 30), ('Jane', 25), ('Bob', 35)]

for name, age in zip(names, ages):
    print(f"{name}: {age}")

names_new, ages_new = zip(*list(zip(names, ages)))
```

---

## 34. What is enumerate?

**Q: Explain enumerate**

A: Get index and value.

```python
names = ['John', 'Jane', 'Bob']

for i, name in enumerate(names):
    print(f"{i}: {name}")

# Start from 1
for i, name in enumerate(names, 1):
    print(f"{i}: {name}")
```

---

## 35. any() and all()

**Q: Explain any() and all()**

A:

**any()** — True if any element is true:

```python
any([False, False, True])  # True
any([False, False])        # False
```

**all()** — True if all elements are true:

```python
all([True, True, True])    # True
all([True, False, True])   # False
```

**With conditions**:

```python
numbers = [2, 4, 6, 8]
all(n % 2 == 0 for n in numbers)  # True
any(n > 10 for n in numbers)      # False
```

---

## Python Interview Tips

1. **Know data structures** deeply — list, tuple, dict, set
2. **List comprehensions** — clean and efficient
3. **Decorators** — understand how they work
4. **OOP concepts** — inheritance, @property, @staticmethod
5. **Exception handling** — try/except/finally
6. **Generators** — memory efficient
7. **Built-in functions** — map, filter, zip, enumerate
8. **Type hints** — write clean, documented code
9. **File handling** — with statement
10. **Production code** — talk about real projects
