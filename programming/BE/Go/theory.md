# Go Language Interview Preparation Guide

## 1. Goroutines & Concurrency

### Goroutines vs Threads

**Goroutines:**

- Lightweight abstractions managed by the Go runtime
- Thousands can run efficiently on a single machine
- Multiplexed onto OS threads by the Go scheduler
- Faster context switching than OS threads
- Lower memory overhead (~2KB per goroutine vs ~2MB per thread)

**Threads (OS):**

- Heavy-weight, managed by the operating system
- Context switching is expensive
- Limited number can run efficiently (~thousands)
- Higher memory and CPU overhead

```go
// Creating goroutines is simple
go functionName()     // Runs concurrently
go anonymousFunc()    // Also works
```

**Key Interview Point:** Go abstracts away thread management; you don't create threads, you create goroutines that are scheduled onto threads by the runtime.

### Channels

**What are channels?**

- Typed conduits for communication between goroutines
- Synchronization primitive for passing data safely
- Send and receive operations block until other side is ready
- Prevents race conditions without explicit locks

```go
// Creating channels
ch := make(chan int)           // Unbuffered channel
bufferedCh := make(chan int, 5) // Buffered with capacity 5

// Sending and receiving
ch <- 42       // Send value 42
value := <-ch  // Receive value

// Closing a channel
close(ch)
```

**Unbuffered Channels:**

- Sender blocks until receiver is ready
- Receiver blocks until sender sends
- Synchronous communication

```go
ch := make(chan int)
go func() {
    ch <- 42  // Blocks here until someone receives
}()
value := <-ch  // This unblocks the goroutine
```

**Buffered Channels:**

- Can hold N values without blocking
- Sender only blocks when buffer is full
- Receiver only blocks when buffer is empty
- Asynchronous communication up to buffer capacity

```go
ch := make(chan int, 2)
ch <- 1  // OK
ch <- 2  // OK
// ch <- 3  // Would block - buffer full
```

**Channel Operations:**

```go
// Receive with ok flag (detects closed channels)
value, ok := <-ch
if !ok {
    // Channel is closed and empty
}

// Range over channel (exits when closed)
for value := range ch {
    fmt.Println(value)
}

// Select statement (multiplexing)
select {
case msg1 := <-ch1:
    fmt.Println("Received from ch1:", msg1)
case msg2 := <-ch2:
    fmt.Println("Received from ch2:", msg2)
case ch3 <- value:
    fmt.Println("Sent to ch3")
default:
    fmt.Println("No operation ready")
}
```

### Common Channel Patterns

**Generator Pattern:**

```go
func generate(n int) <-chan int {
    ch := make(chan int)
    go func() {
        for i := 0; i < n; i++ {
            ch <- i
        }
        close(ch)
    }()
    return ch
}

for value := range generate(5) {
    fmt.Println(value)
}
```

**Fan-out, Fan-in:**

```go
// Fan-out: distribute work
func fanOut(input <-chan int, n int) []<-chan int {
    channels := make([]<-chan int, n)
    for i := 0; i < n; i++ {
        ch := make(chan int)
        channels[i] = ch
        go func(out chan int) {
            for val := range input {
                out <- val * val
            }
            close(out)
        }(ch)
    }
    return channels
}

// Fan-in: merge results
func fanIn(channels ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    for _, ch := range channels {
        wg.Add(1)
        go func(in <-chan int) {
            defer wg.Done()
            for val := range in {
                out <- val
            }
        }(ch)
    }
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

**Pipeline Pattern:**

```go
func pipeline() {
    // Producer
    numbers := make(chan int)
    go func() {
        for i := 1; i <= 5; i++ {
            numbers <- i
        }
        close(numbers)
    }()

    // Transform: square numbers
    squares := make(chan int)
    go func() {
        for n := range numbers {
            squares <- n * n
        }
        close(squares)
    }()

    // Consumer
    for sq := range squares {
        fmt.Println(sq)
    }
}
```

### sync Package Primitives

**sync.Mutex (Mutual Exclusion):**

```go
import "sync"

var counter int
var mu sync.Mutex

func increment() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}
```

**sync.RWMutex (Read-Write Mutex):**

```go
var data map[string]int
var rwmu sync.RWMutex

func read(key string) int {
    rwmu.RLock()  // Multiple readers
    defer rwmu.RUnlock()
    return data[key]
}

func write(key string, value int) {
    rwmu.Lock()   // Exclusive writer
    defer rwmu.Unlock()
    data[key] = value
}
```

**sync.WaitGroup:**

```go
var wg sync.WaitGroup

wg.Add(3)  // Expect 3 goroutines

go func() {
    defer wg.Done()
    // Work
}()

wg.Wait()  // Block until all Done() called
```

**sync.Once:**

```go
var once sync.Once
var instance *Singleton

func getInstance() *Singleton {
    once.Do(func() {
        instance = &Singleton{}
    })
    return instance
}
```

**sync.Cond (Condition Variable):**

```go
var mu sync.Mutex
var cond = sync.NewCond(&mu)

// Waiter
cond.L.Lock()
for !ready {
    cond.Wait()  // Releases lock, waits for signal
}
cond.L.Unlock()

// Signaler
cond.L.Lock()
ready = true
cond.Signal()  // Wake one waiter
cond.L.Unlock()
```

### Race Conditions & Race Detector

```go
// ❌ WRONG - Race condition
var counter int
go func() { counter++ }()
go func() { counter++ }()
// Run with: go run -race main.go

// ✅ CORRECT - Using mutex
var counter int
var mu sync.Mutex

go func() {
    mu.Lock()
    counter++
    mu.Unlock()
}()
go func() {
    mu.Lock()
    counter++
    mu.Unlock()
}()
```

**Key Interview Point:** Use `go run -race` to detect race conditions during testing.

---

## 2. Memory Model & Pointers

### Pointers in Go

**Explicit Pointers (unlike Python):**

```go
x := 10
p := &x        // Address of x
fmt.Println(*p) // Dereference: prints 10

*p = 20
fmt.Println(x)  // 20

// Nil pointer
var ptr *int
fmt.Println(ptr == nil)  // true
```

**Pointer to Struct:**

```go
type Person struct {
    name string
    age  int
}

p := &Person{"Alice", 30}
p.name = "Bob"       // Automatic dereferencing
fmt.Println(p.name)  // Bob
```

### Stack vs Heap

**Stack:**

- Local variables, function parameters
- Automatically freed when function returns
- Faster access
- Limited size

**Heap:**

- Dynamically allocated memory
- Managed by garbage collector
- Slower access
- Unbounded size

```go
func stackVar() {
    x := 10  // Stack - freed at function return
}

func heapVar() *int {
    y := 20
    return &y  // OK! Go automatically promotes to heap
}
```

### Escape Analysis

Go compiler determines if variables escape to the heap:

- Return pointer to local variable → escapes to heap
- Store pointer in heap-allocated object → escapes
- Pass to function that may escape → escapes

```go
// Escapes to heap (returned)
func create() *int {
    x := 5
    return &x
}

// Stays on stack (never escapes)
func compute() int {
    x := 5
    y := 10
    return x + y
}
```

### Garbage Collection

**Mark-and-Sweep:**

- Mark phase: trace all reachable objects
- Sweep phase: reclaim unmarked objects
- Generational collection (recent allocations checked more often)
- Concurrent with application (low pause times)

**Key Interview Point:** Go's GC is concurrent and optimized; you rarely need to think about it.

### Memory Leaks

Common causes:

```go
// ❌ Goroutine leak - sender never closes
func leak() {
    ch := make(chan int)
    go func() {
        ch <- 42
    }()
    // Goroutine blocked forever, channel not read
}

// ✅ FIX - Always close or read
func fixed() <-chan int {
    ch := make(chan int)
    go func() {
        ch <- 42
        close(ch)
    }()
    return ch
}

// ❌ Circular references in goroutines
var m = make(map[int]*Node)
for i := 0; i < 1000; i++ {
    node := &Node{val: i}
    m[i] = node  // Keeps reference
    // If node isn't cleaned up, memory leak
}

// ✅ Clean up explicitly
delete(m, key)
```

---

## 3. Interfaces & Polymorphism

### Interface Basics

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Any type with Read method is a Reader (structural typing)
type File struct {
    // ...
}

func (f *File) Read(p []byte) (n int, err error) {
    // Implementation
}

// File now satisfies Reader interface implicitly
```

**Key Point:** Go uses structural typing, not nominal. If it looks like a duck and quacks like a duck, it's a duck.

### Empty Interface

```go
interface{}  // Accepts any type

func PrintAny(v interface{}) {
    fmt.Println(v)
}

// Type assertion
func Assert(v interface{}) {
    str := v.(string)  // Panics if v is not string

    str, ok := v.(string)  // Safe way
    if ok {
        fmt.Println(str)
    }
}

// Type switch
func TypeSwitch(v interface{}) {
    switch v := v.(type) {
    case string:
        fmt.Println("String:", v)
    case int:
        fmt.Println("Int:", v)
    default:
        fmt.Println("Unknown type")
    }
}
```

### Method Sets

```go
type T struct {
    x int
}

// Method on value receiver
func (t T) Method1() {
    t.x = 10  // Only modifies copy
}

// Method on pointer receiver
func (t *T) Method2() {
    t.x = 10  // Modifies original
}

// Rules:
// - Value receiver: can call with T or *T
// - Pointer receiver: only call with *T (method set)
```

### Interface Composition

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

type ReadCloser interface {
    Reader
    Closer
}

// Type implementing ReadCloser must implement both
```

### Common Interfaces

```go
// io.Reader
type Reader interface {
    Read(p []byte) (n int, err error)
}

// io.Writer
type Writer interface {
    Write(p []byte) (n int, err error)
}

// fmt.Stringer
type Stringer interface {
    String() string
}

// error
type error interface {
    Error() string
}

// encoding/json.Marshaler
type Marshaler interface {
    MarshalJSON() ([]byte, error)
}
```

---

## 4. Structs, Methods & Receivers

### Struct Definition

```go
type Person struct {
    Name string
    Age  int
    Email string  // Exported (uppercase)
    phone string  // Unexported (lowercase)
}

// Creating instances
p1 := Person{"Alice", 30, "alice@example.com", "123-456"}
p2 := Person{Name: "Bob", Age: 25}  // Zero values for unspecified
p3 := &Person{Name: "Charlie", Age: 35}  // Pointer

// Accessing fields
fmt.Println(p1.Name)
fmt.Println(p3.Name)  // No need to dereference, Go does it automatically
```

### Methods

```go
// Value receiver - receives copy
func (p Person) FullInfo() string {
    return fmt.Sprintf("%s (%d)", p.Name, p.Age)
}

// Pointer receiver - receives reference (can modify)
func (p *Person) UpdateAge(age int) {
    p.Age = age
}

// Using methods
p := Person{"Alice", 30, "alice@example.com", "123"}
fmt.Println(p.FullInfo())
p.UpdateAge(31)  // Automatic pointer conversion
fmt.Println(p.Age)
```

**When to use pointer receiver:**

- Need to modify receiver
- Large struct (avoid copying)
- Interface requires pointer receiver

### Embedded Structs (Composition)

```go
type Address struct {
    Street string
    City   string
}

type Employee struct {
    Name string
    Address  // Embedded struct
}

e := Employee{
    Name: "Alice",
    Address: Address{
        Street: "123 Main St",
        City: "NYC",
    },
}

fmt.Println(e.Name)    // Alice
fmt.Println(e.City)    // NYC (promoted field)

// Promoted methods from Address are also available on Employee
```

### Struct Tags

```go
type User struct {
    Name  string `json:"name" validate:"required"`
    Email string `json:"email" validate:"email"`
    Age   int    `json:"age" validate:"gt=0"`
}

// Used by reflection and libraries
data := []byte(`{"name":"Alice","email":"alice@example.com","age":30}`)
var u User
json.Unmarshal(data, &u)  // Respects tags
```

---

## 5. Error Handling

### Error Interface

```go
type error interface {
    Error() string
}

// Creating errors
var ErrNotFound = errors.New("not found")

// Custom errors
type ValidationError struct {
    Field string
    Value interface{}
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("invalid %s: %v", e.Field, e.Value)
}
```

### Error Checking Pattern

```go
f, err := os.Open("file.txt")
if err != nil {
    fmt.Println("Error:", err)
    return
}
defer f.Close()

// Always check errors immediately
data, err := ioutil.ReadAll(f)
if err != nil {
    return err
}
```

### Error Wrapping (Go 1.13+)

```go
// Wrapping errors preserves context
if err != nil {
    return fmt.Errorf("failed to open file: %w", err)
}

// Unwrapping and checking
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("File doesn't exist")
}

// Type assertion on errors
var e *MyError
if errors.As(err, &e) {
    fmt.Println("Got MyError:", e)
}
```

### Panic and Recover

```go
// Panic stops execution, defer runs, then program crashes
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered:", r)
    }
}()

panic("something went wrong")  // Program would crash without recover

// Use panic for truly exceptional conditions
// Use error return values for normal error handling
```

---

## 6. Type System

### Basic Types

```go
// Numeric types
var i int        // Platform-dependent (32 or 64 bits)
var i8 int8      // -128 to 127
var u uint       // Non-negative
var f float64    // 64-bit floating point
var c complex128 // Complex numbers

// String and byte
var s string     // UTF-8 encoded
var b byte       // Alias for uint8
var r rune       // Unicode code point (alias for int32)

// Boolean
var flag bool

// Zero values
var x int        // 0
var s string     // ""
var p *int       // nil
var sl []int     // nil slice
var m map[string]int  // nil map
```

### Type Conversion

```go
// Explicit conversion required
var i int = 42
var f float64 = float64(i)
var s string = strconv.Itoa(i)  // Using strconv for complex conversions

// No implicit conversion
// f := 3.14
// i := f  // Compilation error
```

### Type Assertion

```go
var i interface{} = "hello"

// Direct assertion
s := i.(string)

// Safe assertion
s, ok := i.(string)
if ok {
    fmt.Println(s)
}
```

---

## 7. Collections & Data Structures

### Arrays

```go
// Fixed size
var arr [5]int      // Zero-initialized
arr := [5]int{1, 2, 3, 4, 5}
arr := [...]int{1, 2, 3}  // Compiler infers length

// Array is a value type
a := [3]int{1, 2, 3}
b := a                     // Copies array
b[0] = 99
fmt.Println(a[0])          // Still 1
```

### Slices

```go
// Dynamic array
var s []int                    // nil slice
s := []int{1, 2, 3}          // Slice literal
s := make([]int, 5, 10)       // Slice with len=5, cap=10

// Slice is a reference type
s1 := []int{1, 2, 3}
s2 := s1                      // Shares underlying array
s2[0] = 99
fmt.Println(s1[0])            // 99

// Slice operations
s := []int{1, 2, 3, 4, 5}
sub := s[1:4]                 // [2 3 4] - indices 1, 2, 3
sub := s[:3]                  // [1 2 3]
sub := s[2:]                  // [3 4 5]

// Append (may allocate new array)
s := []int{1, 2}
s = append(s, 3)              // [1 2 3]
s = append(s, 4, 5)           // [1 2 3 4 5]

// Capacity and length
s := make([]int, 3, 10)
fmt.Println(len(s), cap(s))   // 3 10
```

**Key Interview Point:** Slices are references to underlying arrays; passing slices to functions passes a reference to the data.

### Maps

```go
// Declaration and creation
var m map[string]int                  // nil map
m := make(map[string]int)             // Empty map
m := map[string]int{"a": 1, "b": 2}  // Map literal

// Accessing
m["key"] = 42
value := m["key"]
value, ok := m["key"]  // Check if key exists

// Deletion
delete(m, "key")

// Iteration
for key, value := range m {
    fmt.Println(key, value)
}

// Maps are reference types
m1 := map[string]int{"a": 1}
m2 := m1           // Same map
m2["a"] = 2
fmt.Println(m1["a"]) // 2
```

**Map comparison:** Maps can't be compared with == (except to nil)

---

## 8. Defer, Panic, Recover

### Defer

```go
// Defer schedules function call to run when current function returns
defer fmt.Println("Deferred")
fmt.Println("Normal")
// Output: Normal, then Deferred

// Common use: resource cleanup
func readFile(filename string) {
    f, err := os.Open(filename)
    if err != nil {
        return
    }
    defer f.Close()  // Guaranteed to run
    // Use f
}

// Deferred functions execute in LIFO order
defer fmt.Println("1")
defer fmt.Println("2")
defer fmt.Println("3")
// Output: 3, 2, 1

// Arguments evaluated immediately, execution deferred
x := 5
defer fmt.Println(x)  // Evaluates x now, prints 5 later
x = 10
```

### Panic and Recover

```go
// Panic terminates execution
panic("something went wrong")

// Recover catches panic
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered from panic:", r)
    }
}()

panic("error")  // Doesn't crash, caught by defer
```

---

## 9. Package & Module System

### Packages

```go
// Every Go file belongs to a package
package main  // Package name

import (
    "fmt"           // Standard library
    "github.com/pkg/lib"  // External package
)

// Exported identifiers (uppercase)
type PublicStruct struct {
    PublicField string
}

func PublicFunction() {}

// Unexported identifiers (lowercase)
type privateStruct struct {
    privateField string
}

func privateFunction() {}
```

### Visibility

```go
// In file: math/math.go
package math

func PublicAdd(a, b int) int {    // Can be imported and used
    return a + b
}

func privateMultiply(a, b int) int {  // Only usable within package
    return a * b
}
```

### Go Modules (Go 1.11+)

```bash
# Create module
go mod init github.com/user/project

# go.mod file:
# module github.com/user/project
# go 1.19
# require github.com/pkg/lib v1.0.0

# Add dependencies
go get github.com/pkg/lib

# Update dependencies
go mod tidy
```

---

## 10. Common Pitfalls & Best Practices

### Goroutine Leaks

```go
// ❌ WRONG - Goroutine blocked forever
func leak() {
    ch := make(chan int)
    go func() {
        ch <- 42
    }()
    // Never read from channel, goroutine stuck
}

// ✅ CORRECT - Always close channels
func fixed() <-chan int {
    ch := make(chan int)
    go func() {
        ch <- 42
        close(ch)
    }()
    return ch
}

// ✅ CORRECT - Use context
func withContext(ctx context.Context) <-chan int {
    ch := make(chan int)
    go func() {
        select {
        case ch <- 42:
        case <-ctx.Done():
            return
        }
        close(ch)
    }()
    return ch
}
```

### Nil Pointer Dereference

```go
// ❌ WRONG
var p *int
fmt.Println(*p)  // Panic!

// ✅ CORRECT
var p *int
if p != nil {
    fmt.Println(*p)
}
```

### Slice Gotchas

```go
// ❌ WRONG - Underlying array modified
s := []int{1, 2, 3, 4, 5}
sub := s[1:3]  // [2 3]
sub[0] = 99
fmt.Println(s)  // [1 99 3 4 5] - s modified!

// ✅ CORRECT - Copy if needed
sub := make([]int, len(s[1:3]))
copy(sub, s[1:3])
sub[0] = 99
fmt.Println(s)  // [1 2 3 4 5] - s unchanged

// Append gotcha
s := make([]int, 3, 5)  // len=3, cap=5
s = append(s, 1, 2)      // Uses capacity, still same backing array
s = append(s, 3)         // New backing array allocated
```

### Interface{} Slicing

```go
// ❌ WRONG - Can't do this
var ints []int = []int{1, 2, 3}
var vals []interface{} = ints  // Compilation error

// ✅ CORRECT - Explicit conversion
ints := []int{1, 2, 3}
vals := make([]interface{}, len(ints))
for i, v := range ints {
    vals[i] = v
}
```

### Receiver Type Mistakes

```go
// ❌ WRONG - Interface expects pointer receiver but uses value
type Reader interface {
    Read(p []byte) (n int, err error)
}

type MyReader struct{}

func (r MyReader) Read(p []byte) (n int, err error) {  // Value receiver
    return len(p), nil
}

var reader Reader = MyReader{}  // OK, but can't modify receiver

// ✅ CORRECT - Use pointer receiver if you need to modify
func (r *MyReader) Read(p []byte) (n int, err error) {  // Pointer receiver
    return len(p), nil
}

var reader Reader = &MyReader{}  // Must use pointer
```

### Closing Channels Multiple Times

```go
// ❌ WRONG - Panic on second close
ch := make(chan int)
close(ch)
close(ch)  // Panic!

// Rule: Only sender should close channel
// If multiple senders, use sync.Once or other mechanism
```

---

## 11. Context Package

### Context Basics

```go
import "context"

// Creating contexts
ctx := context.Background()  // Base context
ctx := context.TODO()        // When unclear which context to use

// Timeout context
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// Deadline context
ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(10*time.Second))
defer cancel()

// Cancel context
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

// Value context
ctx = context.WithValue(context.Background(), "userID", 123)
userID := ctx.Value("userID").(int)
```

### Using Context for Cancellation

```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Cancelled:", ctx.Err())
            return
        default:
            // Do work
            time.Sleep(1 * time.Second)
        }
    }
}

ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()
worker(ctx)
```

---

## 12. Generics (Go 1.18+)

### Generic Functions

```go
// Before Go 1.18 - interface{}
func GetFirst(s []interface{}) interface{} {
    if len(s) == 0 {
        return nil
    }
    return s[0]
}

// With generics (Go 1.18+)
func GetFirst[T any](s []T) T {
    var zero T
    if len(s) == 0 {
        return zero
    }
    return s[0]
}

value := GetFirst([]int{1, 2, 3})  // Type inference: int
```

### Generic Structs and Methods

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

// Using
intStack := &Stack[int]{}
intStack.Push(42)
value, _ := intStack.Pop()
```

### Type Constraints

```go
// Constraint: T must have + operator
type Number interface {
    int | int64 | float64
}

func Add[T Number](a, b T) T {
    return a + b
}

// Constraint: T must implement Reader
type Reader interface {
    Read(p []byte) (n int, err error)
}

func ReadData[R Reader](r R) error {
    // Use R
    return nil
}
```

---

## 13. Performance Tips

### Memory Allocation

```go
// ❌ SLOW - Many allocations
var s []int
for i := 0; i < 1000000; i++ {
    s = append(s, i)  // Reallocates frequently
}

// ✅ FAST - Pre-allocate
s := make([]int, 0, 1000000)
for i := 0; i < 1000000; i++ {
    s = append(s, i)  // Append uses pre-allocated space
}
```

### Avoid Unnecessary Allocations

```go
// ❌ Creates new string (inefficient)
for i := 0; i < n; i++ {
    s := s + str  // O(n) per iteration, total O(n²)
}

// ✅ Efficient
var buf bytes.Buffer
for i := 0; i < n; i++ {
    buf.WriteString(str)
}
result := buf.String()
```

### Inlining and Compiler Optimizations

```go
// Leaf functions are often inlined (no function call overhead)
func Add(a, b int) int {
    return a + b
}

// Non-leaf functions can't be inlined
func Complex(a, b int) int {
    return Add(a, b) * 2
}
```

---

## 14. Testing

### Unit Testing

```go
// math_test.go
package math

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}

func TestSubtract(t *testing.T) {
    tests := []struct {
        name  string
        a, b  int
        want  int
    }{
        {"positive", 5, 3, 2},
        {"negative", 3, 5, -2},
        {"zero", 5, 5, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Subtract(tt.a, tt.b); got != tt.want {
                t.Errorf("Subtract(%d, %d) = %d; want %d",
                    tt.a, tt.b, got, tt.want)
            }
        })
    }
}
```

### Benchmarking

```go
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

// Run: go test -bench=.
```

---

## Practice Questions

1. Explain the difference between goroutines and OS threads
2. What's the difference between buffered and unbuffered channels?
3. How does the GC work in Go?
4. Explain the select statement in Go
5. What are the rules for interface satisfaction in Go?
6. How does error handling work in Go? Why not exceptions?
7. Explain defer, panic, and recover
8. What is escape analysis? Why does it matter?
9. How does the scheduler work in Go?
10. What causes goroutine leaks and how to prevent them?
11. Explain method sets and receiver types
12. What's the difference between value and reference types?
13. How does the context package work?
14. What are generics and when would you use them?
15. How do you handle cancellation in Go?

# Go Interview: Practical Code Examples

## 1. Goroutines and Channels

### Basic Goroutines

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// Sequential execution
	fmt.Println("Sequential:")
	task("1")
	task("2")

	// Concurrent execution
	fmt.Println("\nConcurrent:")
	go task("1")
	go task("2")
	time.Sleep(3 * time.Second)  // Wait for goroutines
}

func task(name string) {
	for i := 0; i < 3; i++ {
		fmt.Printf("Task %s: %d\n", name, i)
		time.Sleep(1 * time.Second)
	}
}
```

### Unbuffered Channels (Synchronous)

```go
package main

import "fmt"

func main() {
	ch := make(chan int)

	// This goroutine will block until someone receives
	go func() {
		fmt.Println("Sending 42...")
		ch <- 42
		fmt.Println("Sent 42")
	}()

	// This receive unblocks the goroutine
	fmt.Println("Receiving...")
	value := <-ch
	fmt.Println("Received:", value)
}
```

### Buffered Channels (Asynchronous)

```go
package main

import "fmt"

func main() {
	// Buffer size of 2
	ch := make(chan int, 2)

	ch <- 1          // OK
	ch <- 2          // OK
	// ch <- 3       // Would block - buffer full

	fmt.Println(<-ch)  // 1
	fmt.Println(<-ch)  // 2
}
```

### Range and Close

```go
package main

import "fmt"

func main() {
	ch := make(chan int)

	// Producer
	go func() {
		for i := 1; i <= 5; i++ {
			ch <- i
		}
		close(ch)  // Signal we're done
	}()

	// Consumer
	for value := range ch {  // Loop exits when channel closed
		fmt.Println("Received:", value)
	}
}
```

### Select Statement

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	// Goroutines that send after delay
	go func() {
		time.Sleep(1 * time.Second)
		ch1 <- "From ch1"
	}()

	go func() {
		time.Sleep(2 * time.Second)
		ch2 <- "From ch2"
	}()

	// Select waits for first ready case
	select {
	case msg1 := <-ch1:
		fmt.Println("Received:", msg1)
	case msg2 := <-ch2:
		fmt.Println("Received:", msg2)
	case <-time.After(3 * time.Second):
		fmt.Println("Timeout")
	}
}
```

### Pipeline Pattern

```go
package main

import "fmt"

func main() {
	// Stage 1: Generate numbers
	numbers := generate(5)

	// Stage 2: Square numbers
	squares := square(numbers)

	// Stage 3: Print squares
	for sq := range squares {
		fmt.Println(sq)
	}
}

func generate(n int) <-chan int {
	ch := make(chan int)
	go func() {
		for i := 1; i <= n; i++ {
			ch <- i
		}
		close(ch)
	}()
	return ch
}

func square(numbers <-chan int) <-chan int {
	ch := make(chan int)
	go func() {
		for n := range numbers {
			ch <- n * n
		}
		close(ch)
	}()
	return ch
}
```

### Fan-Out / Fan-In

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	// Generate numbers
	numbers := make(chan int)
	go func() {
		for i := 1; i <= 5; i++ {
			numbers <- i
		}
		close(numbers)
	}()

	// Fan-out to 3 workers
	results := fanOut(numbers, 3)

	// Fan-in results
	for result := range fanIn(results...) {
		fmt.Println("Result:", result)
	}
}

func fanOut(input <-chan int, workers int) []<-chan int {
	channels := make([]<-chan int, workers)
	for i := 0; i < workers; i++ {
		ch := make(chan int)
		channels[i] = ch
		go func(out chan int, id int) {
			for val := range input {
				out <- val * val  // Square the number
			}
			close(out)
		}(ch, i)
	}
	return channels
}

func fanIn(channels ...<-chan int) <-chan int {
	out := make(chan int)
	var wg sync.WaitGroup
	for _, ch := range channels {
		wg.Add(1)
		go func(in <-chan int) {
			defer wg.Done()
			for val := range in {
				out <- val
			}
		}(ch)
	}
	go func() {
		wg.Wait()
		close(out)
	}()
	return out
}
```

---

## 2. Synchronization Primitives

### Mutex Example

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var counter int
	var mu sync.Mutex

	// Without mutex: race condition
	for i := 0; i < 1000; i++ {
		go func() {
			mu.Lock()
			counter++
			mu.Unlock()
		}()
	}

	// Proper cleanup
	var wg sync.WaitGroup
	wg.Add(1000)
	for i := 0; i < 1000; i++ {
		go func() {
			defer wg.Done()
			mu.Lock()
			counter++
			mu.Unlock()
		}()
	}

	wg.Wait()
	fmt.Println("Counter:", counter)  // Should be exactly 1000
}
```

### RWMutex (Read-Write Mutex)

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var data = make(map[string]int)
	var rwmu sync.RWMutex

	// Multiple concurrent readers
	for i := 0; i < 5; i++ {
		go func(id int) {
			for j := 0; j < 3; j++ {
				rwmu.RLock()
				fmt.Printf("Reader %d: %v\n", id, data)
				rwmu.RUnlock()
				time.Sleep(100 * time.Millisecond)
			}
		}(i)
	}

	// Exclusive writer
	for i := 0; i < 3; i++ {
		time.Sleep(200 * time.Millisecond)
		rwmu.Lock()
		data[fmt.Sprintf("key%d", i)] = i
		fmt.Printf("Writer: wrote key%d\n", i)
		rwmu.Unlock()
	}

	time.Sleep(2 * time.Second)
}
```

### WaitGroup Example

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup

	// Add 3 tasks
	wg.Add(3)

	for i := 1; i <= 3; i++ {
		go func(id int) {
			defer wg.Done()  // Mark task complete
			fmt.Printf("Task %d starting\n", id)
			time.Sleep(time.Duration(id) * time.Second)
			fmt.Printf("Task %d done\n", id)
		}(i)
	}

	wg.Wait()  // Block until all tasks complete
	fmt.Println("All tasks finished")
}
```

### Once Example (Singleton)

```go
package main

import (
	"fmt"
	"sync"
)

type Singleton struct {
	value string
}

var (
	instance *Singleton
	once     sync.Once
)

func GetInstance() *Singleton {
	once.Do(func() {
		instance = &Singleton{value: "initialized"}
		fmt.Println("Creating singleton")
	})
	return instance
}

func main() {
	// Called multiple times but initializes only once
	s1 := GetInstance()
	s2 := GetInstance()
	s3 := GetInstance()

	fmt.Println(s1 == s2 && s2 == s3)  // true
}
```

### Condition Variable Example

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var mu sync.Mutex
	var ready = false
	var cond = sync.NewCond(&mu)

	// Waiter goroutine
	go func() {
		cond.L.Lock()
		for !ready {
			fmt.Println("Waiting...")
			cond.Wait()  // Release lock and wait
		}
		cond.L.Unlock()
		fmt.Println("Proceeding!")
	}()

	// Signaler goroutine
	time.Sleep(2 * time.Second)
	cond.L.Lock()
	ready = true
	cond.Signal()  // Wake one waiter
	cond.L.Unlock()

	time.Sleep(1 * time.Second)
}
```

---

## 3. Memory and Pointers

### Pointers

```go
package main

import "fmt"

func main() {
	x := 10
	p := &x        // Address of x

	fmt.Println("x:", x)
	fmt.Println("p:", p)
	fmt.Println("*p:", *p)

	*p = 20        // Dereference and modify
	fmt.Println("x:", x)  // Also 20

	// Nil pointer
	var nilPtr *int
	fmt.Println("nil pointer:", nilPtr)
	// fmt.Println(*nilPtr)  // Panic!
}
```

### Pointer to Struct

```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func main() {
	// Value
	p1 := Person{"Alice", 30}
	fmt.Println(p1.Name)

	// Pointer (Go auto-dereferences)
	p2 := &Person{"Bob", 25}
	fmt.Println(p2.Name)    // No need for (*p2).Name
	p2.Age = 26             // Can modify through pointer

	// Method receiver matters
	incrementAge(p2)        // Must pass pointer for modifications
	fmt.Println(p2.Age)     // 27
}

// Value receiver - works but doesn't modify original
func incrementAgeValue(p Person) {
	p.Age++
}

// Pointer receiver - modifies original
func incrementAge(p *Person) {
	p.Age++
}
```

### Escape Analysis

```go
package main

import "fmt"

// x escapes to heap (returned)
func createPointer() *int {
	x := 5
	return &x  // x escapes to heap automatically
}

// y stays on stack (never escapes)
func simpleReturn() int {
	y := 10
	return y
}

func main() {
	p := createPointer()
	fmt.Println(*p)  // 5

	v := simpleReturn()
	fmt.Println(v)   // 10

	// Check with: go run -gcflags="-m" main.go
}
```

---

## 4. Interfaces

### Basic Interface

```go
package main

import "fmt"

type Reader interface {
	Read() string
}

type File struct {
	name string
}

func (f *File) Read() string {
	return fmt.Sprintf("Reading %s", f.name)
}

type Database struct {
	query string
}

func (db *Database) Read() string {
	return fmt.Sprintf("Executing: %s", db.query)
}

func main() {
	var r Reader

	r = &File{"data.txt"}
	fmt.Println(r.Read())

	r = &Database{"SELECT * FROM users"}
	fmt.Println(r.Read())
}
```

### Empty Interface and Type Assertion

```go
package main

import "fmt"

func main() {
	var i interface{} = "hello"

	// Direct assertion
	s := i.(string)
	fmt.Println(s)

	// Safe assertion
	s, ok := i.(string)
	if ok {
		fmt.Println("String:", s)
	}

	// Type switch
	switch v := i.(type) {
	case string:
		fmt.Println("It's a string:", v)
	case int:
		fmt.Println("It's an int:", v)
	default:
		fmt.Println("Unknown type")
	}
}
```

### Interface Composition

```go
package main

import "fmt"

type Reader interface {
	Read() string
}

type Closer interface {
	Close() error
}

type ReadCloser interface {
	Reader
	Closer
}

type File struct {
	name string
}

func (f *File) Read() string {
	return fmt.Sprintf("Reading %s", f.name)
}

func (f *File) Close() error {
	fmt.Printf("Closing %s\n", f.name)
	return nil
}

func main() {
	f := &File{"data.txt"}
	var rc ReadCloser = f
	fmt.Println(rc.Read())
	rc.Close()
}
```

### Stringer Interface

```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

// Implementing fmt.Stringer interface
func (p Person) String() string {
	return fmt.Sprintf("%s (age %d)", p.Name, p.Age)
}

func main() {
	p := Person{"Alice", 30}
	fmt.Println(p)  // Uses String() method: Alice (age 30)
	fmt.Printf("%v\n", p)
}
```

---

## 5. Structs and Methods

### Embedded Structs (Composition)

```go
package main

import "fmt"

type Address struct {
	Street string
	City   string
}

type Person struct {
	Name string
	Address
}

func (p Person) String() string {
	return fmt.Sprintf("%s, %s", p.Name, p.City)
}

func main() {
	p := Person{
		Name: "Alice",
		Address: Address{
			Street: "123 Main St",
			City:   "NYC",
		},
	}

	fmt.Println(p.Name)      // Alice
	fmt.Println(p.City)      // NYC (promoted field)
	fmt.Println(p.String())  // Alice, NYC
}
```

### Method Sets

```go
package main

import "fmt"

type Counter struct {
	count int
}

// Value receiver - receives copy
func (c Counter) GetCount() int {
	return c.count
}

// Pointer receiver - receives reference
func (c *Counter) Increment() {
	c.count++
}

func main() {
	// Value
	c1 := Counter{0}
	fmt.Println(c1.GetCount())  // OK
	c1.Increment()              // Works but modifies copy
	fmt.Println(c1.GetCount())  // Still 0

	// Pointer
	c2 := &Counter{0}
	fmt.Println(c2.GetCount())  // OK (method set allows value methods on pointers)
	c2.Increment()              // Modifies original
	fmt.Println(c2.GetCount())  // 1
}
```

### Struct Tags

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
	Age   int    `json:"age"`
}

func main() {
	data := []byte(`{"name":"Alice","email":"alice@example.com","age":30}`)

	var u User
	json.Unmarshal(data, &u)
	fmt.Printf("%+v\n", u)

	// Marshal back
	bytes, _ := json.Marshal(u)
	fmt.Println(string(bytes))
}
```

---

## 6. Error Handling

### Error Interface

```go
package main

import (
	"errors"
	"fmt"
)

func divide(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("division by zero")
	}
	return a / b, nil
}

func main() {
	result, err := divide(10, 2)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("Result:", result)

	// With error
	result, err = divide(10, 0)
	if err != nil {
		fmt.Println("Error:", err)
	}
}
```

### Custom Errors

```go
package main

import (
	"fmt"
)

type ValidationError struct {
	Field string
	Value interface{}
}

func (e *ValidationError) Error() string {
	return fmt.Sprintf("validation error on %s: %v", e.Field, e.Value)
}

func validateAge(age int) error {
	if age < 0 {
		return &ValidationError{"age", age}
	}
	return nil
}

func main() {
	err := validateAge(-5)
	if err != nil {
		fmt.Println(err)
	}
}
```

### Error Wrapping (Go 1.13+)

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func main() {
	// Create error
	err := os.Open("nonexistent.txt")

	// Wrap error with context
	if err != nil {
		wrappedErr := fmt.Errorf("failed to open file: %w", err)
		fmt.Println(wrappedErr)

		// Check unwrapped error
		if errors.Is(err, os.ErrNotExist) {
			fmt.Println("File not found")
		}
	}
}
```

### Defer and Cleanup

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Open("file.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	defer f.Close()  // Guaranteed to run

	// Use file
	fmt.Println("Using file")
	// defer runs here automatically
}
```

---

## 7. Generics (Go 1.18+)

### Generic Function

```go
package main

import "fmt"

// Generic function
func First[T any](s []T) T {
	var zero T
	if len(s) == 0 {
		return zero
	}
	return s[0]
}

// Constraint: must support comparison
func Contains[T comparable](s []T, v T) bool {
	for _, item := range s {
		if item == v {
			return true
		}
	}
	return false
}

func main() {
	fmt.Println(First([]int{1, 2, 3}))           // 1
	fmt.Println(First([]string{"a", "b", "c"}))  // a

	fmt.Println(Contains([]int{1, 2, 3}, 2))           // true
	fmt.Println(Contains([]string{"a", "b"}, "c"))    // false
}
```

### Generic Struct

```go
package main

import "fmt"

type Stack[T any] struct {
	items []T
}

func (s *Stack[T]) Push(item T) {
	s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
	if len(s.items) == 0 {
		var zero T
		return zero, false
	}
	item := s.items[len(s.items)-1]
	s.items = s.items[:len(s.items)-1]
	return item, true
}

func main() {
	// Integer stack
	intStack := &Stack[int]{}
	intStack.Push(1)
	intStack.Push(2)
	intStack.Push(3)

	for {
		val, ok := intStack.Pop()
		if !ok {
			break
		}
		fmt.Println(val)  // 3, 2, 1
	}

	// String stack
	strStack := &Stack[string]{}
	strStack.Push("hello")
	strStack.Push("world")
	val, _ := strStack.Pop()
	fmt.Println(val)  // world
}
```

### Type Constraints

```go
package main

import "fmt"

// Constraint: must be integer type
type Number interface {
	int | int64 | float64
}

func Add[T Number](a, b T) T {
	return a + b
}

func main() {
	fmt.Println(Add(2, 3))          // 5 (int)
	fmt.Println(Add(2.5, 3.5))      // 6.0 (float64)
}
```

---

## 8. Context Package

### Context with Timeout

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func longRunningTask(ctx context.Context) error {
	for i := 1; i <= 5; i++ {
		select {
		case <-ctx.Done():
			fmt.Println("Task cancelled:", ctx.Err())
			return ctx.Err()
		default:
			fmt.Printf("Task step %d\n", i)
			time.Sleep(1 * time.Second)
		}
	}
	fmt.Println("Task completed")
	return nil
}

func main() {
	// Timeout context
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel()

	longRunningTask(ctx)
}
```

### Context with Values

```go
package main

import (
	"context"
	"fmt"
)

type userKey string

const UserIDKey userKey = "userID"

func getUser(ctx context.Context) {
	userID := ctx.Value(UserIDKey)
	fmt.Println("User ID:", userID)
}

func main() {
	ctx := context.WithValue(context.Background(), UserIDKey, 123)
	getUser(ctx)
}
```

---

## 9. Panic and Recover

### Panic Recovery

```go
package main

import "fmt"

func safeDivide(a, b int) (result int) {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered from panic:", r)
			result = 0
		}
	}()

	if b == 0 {
		panic("division by zero")
	}
	return a / b
}

func main() {
	fmt.Println(safeDivide(10, 2))   // 5
	fmt.Println(safeDivide(10, 0))   // 0 (recovered)
}
```

---

## 10. Collections

### Slices

```go
package main

import "fmt"

func main() {
	// Create slice
	s := []int{1, 2, 3, 4, 5}

	// Slice operations
	fmt.Println(s[1:4])    // [2 3 4]
	fmt.Println(s[:3])     // [1 2 3]
	fmt.Println(s[2:])     // [3 4 5]

	// Append
	s = append(s, 6)       // [1 2 3 4 5 6]

	// Length and capacity
	fmt.Println(len(s))    // 6
	fmt.Println(cap(s))    // > 6

	// Copy
	dest := make([]int, len(s))
	copy(dest, s)

	// Iteration
	for i, v := range s {
		fmt.Printf("s[%d] = %d\n", i, v)
	}
}
```

### Maps

```go
package main

import "fmt"

func main() {
	// Create map
	m := make(map[string]int)
	m["Alice"] = 30
	m["Bob"] = 25

	// Or literal
	m = map[string]int{
		"Charlie": 35,
		"Diana":   28,
	}

	// Access
	age := m["Alice"]
	fmt.Println(age)

	// Check existence
	age, ok := m["Alice"]
	if ok {
		fmt.Println("Found:", age)
	}

	// Delete
	delete(m, "Bob")

	// Iteration
	for name, age := range m {
		fmt.Printf("%s: %d\n", name, age)
	}
}
```

---

## 11. Common Pitfalls

### Goroutine Leak

```go
package main

import (
	"context"
	"fmt"
	"time"
)

// ❌ WRONG - Goroutine blocked forever
func leak() {
	ch := make(chan int)
	go func() {
		ch <- 42
	}()
	// Never read from channel - goroutine stuck
}

// ✅ CORRECT - Use context
func fixed(ctx context.Context) {
	ch := make(chan int)
	go func() {
		select {
		case ch <- 42:
		case <-ctx.Done():
			return
		}
	}()

	select {
	case value := <-ch:
		fmt.Println(value)
	case <-ctx.Done():
		fmt.Println("Cancelled")
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel()
	fixed(ctx)
}
```

### Nil Pointer Dereference

```go
package main

import "fmt"

func main() {
	var p *int

	if p != nil {
		fmt.Println(*p)  // Safe
	}
}
```

### Slice Confusion

```go
package main

import "fmt"

func main() {
	// Modifying slice affects underlying array
	s1 := []int{1, 2, 3, 4, 5}
	s2 := s1[1:3]  // [2 3]

	s2[0] = 99
	fmt.Println(s1)  // [1 99 3 4 5] - s1 also changed!

	// To avoid: copy the slice
	s3 := make([]int, len(s1[1:3]))
	copy(s3, s1[1:3])
	s3[0] = 99
	fmt.Println(s1)  // [1 2 3 4 5] - unchanged
}
```

### Closing Channels Multiple Times

```go
// ❌ WRONG
ch := make(chan int)
close(ch)
close(ch)  // Panic!

// ✅ CORRECT - Only sender should close
// If multiple senders, don't close from sender; let receiver close or use context
```

---

## 12. Performance

### Allocation Pre-allocation

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// ❌ SLOW - Many reallocations
	start := time.Now()
	var s []int
	for i := 0; i < 1000000; i++ {
		s = append(s, i)
	}
	fmt.Println("Without pre-allocation:", time.Since(start))

	// ✅ FAST - Pre-allocated
	start = time.Now()
	s = make([]int, 0, 1000000)
	for i := 0; i < 1000000; i++ {
		s = append(s, i)
	}
	fmt.Println("With pre-allocation:", time.Since(start))
}
```

### String Concatenation

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	// ❌ SLOW - Many allocations
	result := ""
	for i := 0; i < 1000; i++ {
		result += fmt.Sprintf("Line %d\n", i)
	}

	// ✅ FAST - Buffer
	var buf bytes.Buffer
	for i := 0; i < 1000; i++ {
		fmt.Fprintf(&buf, "Line %d\n", i)
	}
	result = buf.String()
	fmt.Println(len(result))
}
```

---

## 13. Testing

### Unit Testing

```go
// math_test.go
package main

import "testing"

func TestAdd(t *testing.T) {
	tests := []struct {
		name string
		a, b int
		want int
	}{
		{"positive", 2, 3, 5},
		{"negative", -2, -3, -5},
		{"zero", 0, 0, 0},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got := Add(tt.a, tt.b)
			if got != tt.want {
				t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
			}
		})
	}
}

func Add(a, b int) int {
	return a + b
}
```

### Benchmarking

```go
// math_test.go
func BenchmarkAdd(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Add(1000, 2000)
	}
}

// Run: go test -bench=.
```
