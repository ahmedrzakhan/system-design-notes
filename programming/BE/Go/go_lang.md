# Go OOP Interview Questions & Answers

## 1. What are methods and receivers in Go?

**Q: Explain methods and receivers**

A: Methods are functions attached to a type using a receiver. Receivers go between `func` and method name.

```go
func (r Receiver) MethodName() {
    // r is the receiver
}
```

Two types of receivers:

- **Value receiver**: `func (c Circle) Area() {}` — works on a copy
- **Pointer receiver**: `func (c *Circle) SetRadius(r float64) {}` — modifies original

Use pointer receivers when you need to modify the struct.

---

## 2. What is the difference between interfaces in Go and other languages?

**Q: How do interfaces work in Go?**

A: Go interfaces are implicit and structural (duck typing). Any type automatically satisfies an interface if it implements all its methods. No explicit "implements" keyword needed.

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type File struct {}

func (f File) Read(p []byte) (n int, err error) {
    // File now satisfies Reader without declaring it
    return 0, nil
}
```

Other languages require explicit declaration. Go's approach is more flexible.

---

## 3. What is embedding and how is it different from inheritance?

**Q: Explain struct embedding in Go**

A: Embedding is composition—you embed one struct in another to reuse functionality. It's NOT inheritance.

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() {
    fmt.Println(a.Name, "speaks")
}

type Dog struct {
    Animal  // Embedded
    Breed  string
}

dog := Dog{Animal{"Rex"}, "Labrador"}
dog.Speak()  // Method promotion—Speak is promoted to Dog
```

Dog gets Speak method automatically, but it's composition not inheritance. You can override methods in Dog without affecting Animal.

---

## 4. What is encapsulation in Go?

**Q: How does Go handle public and private members?**

A: Go uses capitalization:

- **Uppercase first letter** = Exported (public) — visible outside package
- **Lowercase first letter** = Unexported (private) — visible only within package

```go
type BankAccount struct {
    Balance float64  // Exported
    pin     string   // Unexported (private)
}

func (b *BankAccount) Withdraw(amount float64) error {
    // Can access b.pin here
    if b.Balance >= amount {
        b.Balance -= amount
        return nil
    }
    return errors.New("insufficient balance")
}
```

No getters/setters needed if you're just accessing fields, but use methods for business logic.

---

## 5. What is the empty interface and when do you use it?

**Q: What is `interface{}` and why is it useful?**

A: The empty interface `interface{}` accepts any type. Used when you don't know or don't care about the type.

```go
func PrintAnything(v interface{}) {
    fmt.Println(v)
}

PrintAnything(42)
PrintAnything("hello")
PrintAnything([]int{1, 2, 3})
```

Use cases:

- Accepting any type (like Python's duck typing)
- Working with JSON (unmarshals to `map[string]interface{}`)
- Generic data structures before Go 1.18 generics

With Go 1.18+, use generics instead for type safety.

---

## 6. Explain pointer receivers vs value receivers

**Q: When should I use pointer receivers?**

A:

- **Use pointer receiver** when you need to modify the receiver
- **Use value receiver** for read-only operations

```go
type Account struct {
    Balance float64
}

// Modifies state — use pointer receiver
func (a *Account) Deposit(amount float64) {
    a.Balance += amount
}

// Read-only — can use value receiver
func (a Account) GetBalance() float64 {
    return a.Balance
}
```

**Rule**: If you use pointer receiver for any method, use it for all methods of that type (consistency).

---

## 7. What is the difference between methods and functions?

**Q: Why does Go have both methods and functions?**

A: A method is a function with a receiver. Functions don't have receivers.

```go
// Function
func Add(a, b int) int {
    return a + b
}

// Method
func (calc Calculator) Add(a, b int) int {
    return a + b
}
```

Methods allow you to:

- Organize code around types (OOP style)
- Use interface polymorphism
- Call like `obj.Method()` instead of `Function(obj)`

---

## 8. What is type assertion and when do you use it?

**Q: Explain type assertion in Go**

A: Type assertion extracts the concrete value from an interface.

```go
var i interface{} = "hello"

s := i.(string)  // Type assertion
fmt.Println(s)   // "hello"

v, ok := i.(int)  // Safe assertion with ok flag
if ok {
    fmt.Println(v)
} else {
    fmt.Println("Not an int")
}
```

Use it when working with `interface{}` or when you need to access type-specific methods:

```go
func Process(r interface{}) {
    switch v := r.(type) {
    case io.Reader:
        // v is Reader
    case io.Writer:
        // v is Writer
    }
}
```

---

## 9. Explain the difference between Go and traditional OOP languages

**Q: How is Go's OOP different?**

A: Go has different philosophy:

| Feature        | Traditional OOP              | Go                         |
| -------------- | ---------------------------- | -------------------------- |
| Inheritance    | Yes (classes inherit)        | No (use embedding)         |
| Classes        | Yes                          | No (use structs + methods) |
| Interfaces     | Explicit declaration         | Implicit (duck typing)     |
| Access control | private/public modifiers     | Capitalization             |
| Polymorphism   | Via inheritance + interfaces | Via interfaces only        |

Go favors **composition over inheritance**. Simpler, more flexible, easier to test.

---

## 10. What is the Reader and Writer interface?

**Q: What are Reader and Writer interfaces?**

A: Fundamental Go interfaces for I/O:

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

Any type implementing `Read()` is a Reader. Any type implementing `Write()` is a Writer.

Examples:

- `*os.File` — implements both
- `*bytes.Buffer` — implements both
- `*os.Stdin` — implements Reader
- `*os.Stdout` — implements Writer

**Why it matters**: You can pass any Reader to functions expecting Reader. Very composable.

```go
func Copy(dst Writer, src Reader) (int64, error) {
    // Works with ANY Reader/Writer
}
```

---

## 11. What are goroutines and how do they relate to OOP?

**Q: What are goroutines?**

A: Lightweight concurrency primitives managed by the Go runtime. Not exactly OOP but important in Go.

```go
go func() {
    fmt.Println("Running concurrently")
}()

// Code continues without waiting
```

Relation to OOP: Methods can run as goroutines:

```go
func (w *Worker) Process() {
    // Do work
}

go w.Process()  // Run method concurrently
```

Use channels for communication between goroutines:

```go
ch := make(chan int)
go func() {
    ch <- 42
}()

value := <-ch
```

---

## 12. What is a nil interface and nil pointer?

**Q: What's the difference between nil interface and nil pointer?**

A: Two different concepts:

```go
var i interface{}      // nil interface (no type, no value)
var p *int             // nil pointer (type *int, but value is nil)

i = p                  // Now i is NOT nil (it contains a nil pointer of type *int)

if i == nil {          // false
    fmt.Println("i is nil")
} else {
    fmt.Println("i is not nil")  // This executes
}
```

**Key point**: An interface containing a nil pointer is NOT nil. Common bug!

```go
func Process(r io.Reader) error {
    if r == nil {  // Only checks if interface itself is nil
        return errors.New("reader is nil")
    }
    // If r was a nil pointer wrapped in interface, it's still here!
}
```

---

## 13. What is the purpose of the defer keyword?

**Q: Explain defer and when to use it**

A: `defer` delays a function call until the surrounding function returns. Useful for cleanup.

```go
func ReadFile(filename string) ([]byte, error) {
    file, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer file.Close()  // Guaranteed to run when function exits

    return ioutil.ReadAll(file)
}
```

Common uses:

- Closing files, database connections
- Unlocking mutexes
- Cleanup operations
- Panic recovery

Defers execute in LIFO order (last deferred, first executed).

---

## 14. What is an error interface and how to implement it?

**Q: What is the error interface?**

A: Built-in interface for error handling:

```go
type error interface {
    Error() string
}
```

Implement it by defining `Error()` method:

```go
type CustomError struct {
    Message string
    Code    int
}

func (e CustomError) Error() string {
    return fmt.Sprintf("Error %d: %s", e.Code, e.Message)
}

// Now CustomError satisfies error interface
var err error = CustomError{Message: "Not found", Code: 404}
fmt.Println(err)  // Error 404: Not found
```

**Best practice**: Always return errors from functions instead of panicking.

---

## 15. What is the difference between concrete and interface types?

**Q: Explain concrete vs interface types**

A:

**Concrete type**: An actual struct or primitive type

```go
type Person struct {
    Name string
}
```

**Interface type**: A contract defining methods

```go
type Speaker interface {
    Speak() string
}
```

You can't create variables of interface type (without concrete implementation):

```go
var s Speaker           // OK - nil interface
s = Person{"John"}      // OK - Person implements Speaker

var p *Person
var s Speaker
s = p  // OK - assigning concrete to interface
```

Interfaces are abstract; concrete types are concrete implementations.

---

## Quick Tips for Interviews

1. **Use examples** — Explain with code, not just words
2. **Mention Go philosophy** — "Go favors simplicity and composition"
3. **Compare to languages you know** — "Unlike Java, Go uses..."
4. **Practical examples** — Talk about production code you've written
5. **Interfaces are key** — Go's answer to polymorphism
6. **Pointer vs value** — Know when to use each
7. **Error handling** — Go's approach is explicit, not exceptions
8. **Goroutines** — Mention concurrency if relevant

---

# Go Lang Interview Questions & Answers

## 1. What is Go and why was it created?

**Q: Tell me about Go language**

A: Go (Golang) is a compiled, statically-typed language created by Google in 2007. Designed by Rob Pike, Robert Griesemer, and Ken Thompson.

**Why it was created**:

- C++ compilation was too slow at Google's scale
- Needed a language combining simplicity (Python) with performance (C)
- Built-in concurrency support
- Fast compilation
- Simple syntax

**Key characteristics**:

- Compiled to machine code (fast)
- Statically typed (type safe)
- Garbage collected (automatic memory management)
- Built-in concurrency (goroutines, channels)
- Small standard library (opinionated)
- Fast startup time
- Single binary output (no runtime dependencies)

---

## 2. What are goroutines and how are they different from threads?

**Q: Explain goroutines vs OS threads**

A: Goroutines are lightweight concurrency units managed by the Go runtime, not OS threads.

| Feature           | OS Threads               | Goroutines           |
| ----------------- | ------------------------ | -------------------- |
| Creation cost     | Expensive (MB of memory) | Cheap (KB of memory) |
| Number            | Limited (1000s)          | Can create 100,000s  |
| Scheduling        | OS kernel                | Go runtime           |
| Context switching | Expensive                | Cheap                |
| Stack             | Fixed size               | Dynamic              |

```go
// Creating goroutine is cheap
go func() {
    fmt.Println("Running concurrently")
}()

// Can create many
for i := 0; i < 100000; i++ {
    go doSomething(i)
}
```

**How it works**: Go runtime multiplexes many goroutines onto a few OS threads (M:N scheduling).

---

## 3. What are channels and how do they work?

**Q: Explain channels and their use cases**

A: Channels are typed conduits for communication between goroutines. They're Go's way to safely pass data between concurrent functions.

```go
ch := make(chan int)      // Unbuffered channel

go func() {
    ch <- 42              // Send value
}()

value := <-ch             // Receive value
fmt.Println(value)        // 42
```

**Unbuffered vs Buffered**:

```go
// Unbuffered — sender blocks until receiver ready
ch := make(chan int)

// Buffered — sender only blocks when full
ch := make(chan int, 10)  // Can hold 10 values
ch <- 1
ch <- 2
// Won't block until 10 values queued
```

**Closing channels**:

```go
close(ch)

// Receiving from closed channel
value, ok := <-ch
if !ok {
    fmt.Println("Channel closed")
}

// Loop until closed
for value := range ch {
    fmt.Println(value)
}
```

**Common patterns**: Worker pools, Fan-out/Fan-in, Timeouts, Pub-sub, Pipeline

---

## 4. What is select and how does it work?

**Q: Explain the select statement**

A: `select` lets a goroutine wait on multiple channel operations. It's like `switch` but for channels.

```go
select {
case <-ch1:
    fmt.Println("Received from ch1")
case <-ch2:
    fmt.Println("Received from ch2")
case ch3 <- value:
    fmt.Println("Sent to ch3")
default:
    fmt.Println("No channels ready")
}
```

**Blocks until one case is ready**. If multiple cases ready, picks randomly.

**Timeout use case**:

```go
select {
case result := <-ch:
    fmt.Println(result)
case <-time.After(5 * time.Second):
    fmt.Println("Timeout!")
}
```

---

## 5. What is the difference between `make` and `new`?

**Q: When do you use make vs new?**

A: Both allocate memory but differently:

**`new(T)`** — allocates memory for type T, returns pointer to zero value

```go
p := new(int)      // *int pointing to 0
*p = 42
```

**`make(T)`** — allocates and initializes slices, maps, channels. Returns initialized value.

```go
s := make([]int, 0, 10)    // Slice
m := make(map[string]int)  // Map
ch := make(chan int, 5)    // Channel
```

**When to use**:

- `new` — rarely used
- `make` — slices, maps, channels ONLY
- `var` — structs, arrays, other types

---

## 6. What is defer and in what order do defers execute?

**Q: How does defer work and execution order?**

A: `defer` postpones function execution until surrounding function returns. Executes in LIFO order.

```go
func main() {
    defer fmt.Println("1")
    defer fmt.Println("2")
    defer fmt.Println("3")
    fmt.Println("4")
}

// Output: 4, 3, 2, 1
```

**Common uses**:

```go
// File cleanup
file, err := os.Open("file.txt")
if err != nil {
    return err
}
defer file.Close()

// Mutex unlock
mutex.Lock()
defer mutex.Unlock()

// Panic recovery
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered:", r)
    }
}()
```

---

## 7. What is panic and recover?

**Q: Explain panic and recover**

A: `panic` stops normal execution and starts unwinding. `recover` catches panic.

```go
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered from:", r)
    }
}()

panic("Something went wrong!")
```

**When to use panic**:

- Unrecoverable programmer errors
- NOT for normal error handling

**Best practice**: Use error returns, not panic/recover for normal errors.

---

## 8. What are buffered and unbuffered channels?

**Q: Buffered vs unbuffered channels**

A:

**Unbuffered**: No capacity, must have receiver ready

```go
ch := make(chan int)
ch <- 1  // Blocks until someone receives
```

**Buffered**: Has capacity, sender only blocks when full

```go
ch := make(chan int, 2)
ch <- 1  // OK
ch <- 2  // OK
ch <- 3  // Blocks!
```

**Use cases**:

- Unbuffered: Synchronization
- Buffered: Decoupling sender/receiver

---

## 9. What is the `context` package and why is it important?

**Q: Explain context and its use**

A: `context` provides a way to pass cancellation signals, deadlines, and values across goroutines.

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

go worker(ctx)

func worker(ctx context.Context) {
    select {
    case <-ctx.Done():
        return
    default:
        doWork()
    }
}
```

**Common context types**:

```go
context.Background()                              // Base context
context.WithCancel(ctx)                           // Cancellation
context.WithDeadline(ctx, time.Now().Add(5*time.Second))  // Timeout
context.WithTimeout(ctx, 5*time.Second)          // Simpler timeout
context.WithValue(ctx, key, value)               // Pass values
```

**Best practice**: Always pass context as first parameter.

---

## 10. What is the `sync` package used for?

**Q: Explain sync package**

A: `sync` provides synchronization primitives.

**Mutex** — mutual exclusion lock:

```go
var mu sync.Mutex
mu.Lock()
criticalSection()
mu.Unlock()
```

**RWMutex** — multiple readers, exclusive writer:

```go
var rw sync.RWMutex
rw.RLock()
readData()
rw.RUnlock()
```

**WaitGroup** — wait for goroutines:

```go
var wg sync.WaitGroup
for i := 0; i < 10; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        doWork()
    }()
}
wg.Wait()
```

**Once** — execute exactly once:

```go
var once sync.Once
once.Do(func() {
    instance = &Singleton{}
})
```

---

## 11. What is a slice and how is it different from an array?

**Q: Arrays vs slices**

A:

**Array** — fixed size

```go
var arr [5]int
```

**Slice** — dynamic size

```go
var s []int
s := []int{1, 2, 3}
s = append(s, 4)
```

**Key difference**: Slices are views into arrays, arrays are fixed.

```go
arr := [5]int{1, 2, 3, 4, 5}
s1 := arr[0:3]  // {1, 2, 3}
s2 := arr[2:5]  // {3, 4, 5}

s1[0] = 99
fmt.Println(arr)  // [99, 2, 3, 4, 5] — shared!
```

**Use slices**, not arrays.

---

## 12. What is `reflect.DeepEqual`?

**Q: How do you compare complex types?**

A: `==` doesn't work for slices and maps. Use `reflect.DeepEqual`:

```go
s1 := []int{1, 2, 3}
s2 := []int{1, 2, 3}

s1 == s2  // Compilation error!
reflect.DeepEqual(s1, s2)  // true
```

**When == works**: primitives, pointers, structs with comparable fields

---

## 13. What are goroutine leaks?

**Q: What is a goroutine leak?**

A: Goroutines created but never terminated, consuming memory forever.

**Common cause**:

```go
// LEAK — blocks forever
go func() {
    value := <-ch  // Never receives
}()
```

**Prevention**:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

go func() {
    select {
    case <-ch:
        // process
    case <-ctx.Done():
        return
    }
}()
```

**Always ensure goroutines have exit conditions**.

---

## 14. What is a pointer and when should you use one?

**Q: Explain pointers**

A: Pointers hold memory addresses.

```go
x := 42
p := &x         // Address of x
*p = 100        // Change x through pointer
```

**When to use**:

1. Modify arguments: `func Increment(p *int)`
2. Avoid expensive copies: `func Process(s *LargeStruct)`
3. nil values: `var p *int`

**Avoid unnecessary pointers** — they complicate code.

---

## 15. What is the blank identifier `_`?

**Q: What is the underscore?**

A: Blank identifier to ignore values.

```go
// Ignore error
result, _ := SomeFunction()

// Ignore index
for _, value := range arr {
    fmt.Println(value)
}

// Import for side effects
import _ "database/sql/driver"
```

---

## 16. What is the `error` interface?

**Q: How does Go handle errors?**

A: Go uses explicit error returns.

```go
type error interface {
    Error() string
}

result, err := SomeFunction()
if err != nil {
    return fmt.Errorf("failed: %w", err)
}
```

**Error wrapping (Go 1.13+)**:

```go
if errors.Is(err, io.EOF) {
    // Handle EOF
}
```

**Philosophy**: Errors are values, handle them explicitly.

---

## 17. What is the difference between `var` and `:=`?

**Q: var vs := declaration**

A:

**`var`** — can be at package level

```go
var x int = 5
```

**`:=`** — only inside functions

```go
x := 5
```

`:=` is convenient shorthand for `var x = 5`.

---

## 18. What is `iota`?

**Q: Explain iota constant**

A: `iota` generates auto-incrementing values.

```go
const (
    Red   = iota  // 0
    Green = iota  // 1
    Blue  = iota  // 2
)
```

**With expressions**:

```go
const (
    KB = 1 << (10 * iota)  // 1024
    MB = 1 << (10 * iota)  // 1M
    GB = 1 << (10 * iota)  // 1G
)
```

---

## 19. Functions vs Methods

**Q: Functions vs methods**

A:

**Function**: Standalone

```go
func Add(a, b int) int {
    return a + b
}
```

**Method**: Attached to type

```go
func (c Calculator) Add(a, b int) int {
    return a + b
}

calc := Calculator{}
calc.Add(2, 3)
```

---

## 20. What is a worker pool?

**Q: Explain worker pool pattern**

A: Limits concurrency by queuing work to fixed number of workers.

```go
jobs := make(chan Job, 100)
results := make(chan Result, 100)

for i := 1; i <= numWorkers; i++ {
    go worker(jobs, results)
}

// Send jobs
go func() {
    for _, job := range allJobs {
        jobs <- job
    }
    close(jobs)
}()

// Collect results
for result := range results {
    handleResult(result)
}
```

**Benefits**: Controls concurrency, efficient resource usage, backpressure.

---

## 21. Channels of channels

**Q: Can you send channels over channels?**

A: Yes, channels are first-class values.

```go
ch := make(chan chan int)

go func() {
    subch := make(chan int)
    ch <- subch
    subch <- 42
}()

received := <-ch
value := <-received
```

---

## 22. Closures and loop pitfalls

**Q: Explain closures in Go**

A: Closures capture variables from surrounding scope.

**Common pitfall — loop variable**:

```go
// WRONG — all see final i
for i := 0; i < 5; i++ {
    go func() {
        fmt.Println(i)  // All print 5
    }()
}

// FIX — pass as parameter
for i := 0; i < 5; i++ {
    go func(idx int) {
        fmt.Println(idx)
    }(i)
}
```

---

## 23. What is `unsafe` package?

**Q: When should you use unsafe?**

A: `unsafe` provides direct memory access. Use only when absolutely necessary.

**When**:

- Calling C code (cgo)
- Performance-critical low-level code

**Never for**:

- Normal programming
- Business logic

**If you think you need unsafe, reconsider your approach**.

---

## 24. What is `init()` function?

**Q: Explain init function**

A: `init()` runs automatically before `main()`.

```go
func init() {
    fmt.Println("init runs first")
}

func main() {
    fmt.Println("main")
}
```

**Execution order**:

1. Package initialization (global variables)
2. `init()` functions
3. `main()` (only in main package)

---

## 25. What is `...` (variadic)?

**Q: Explain variadic functions**

A: `...` means variable number of arguments.

```go
func Sum(numbers ...int) int {
    total := 0
    for _, n := range numbers {
        total += n
    }
    return total
}

fmt.Println(Sum(1, 2, 3, 4))  // 10
```

**Unpack slice**:

```go
nums := []int{1, 2, 3, 4}
fmt.Println(Sum(nums...))  // 10
```

---

## 26. What is struct embedding?

**Q: Explain struct composition**

A: Embed structs to reuse functionality.

```go
type Person struct {
    Name string
}

type Employee struct {
    Person
    ID int
}

e := Employee{Person{"John"}, 123}
fmt.Println(e.Name)  // "John"
```

Fields and methods of Person promoted to Employee.

---

## 27. What is type assertion?

**Q: Explain type assertion**

A: Extract concrete value from interface.

```go
var i interface{} = "hello"
s := i.(string)
fmt.Println(s)  // "hello"

v, ok := i.(int)  // Safe assertion
if !ok {
    fmt.Println("Not an int")
}
```

**Type switch**:

```go
switch v := i.(type) {
case string:
    fmt.Println("String:", v)
case int:
    fmt.Println("Int:", v)
}
```

---

## 28. What is the `time` package?

**Q: Common time package operations**

A:

```go
import "time"

// Current time
now := time.Now()

// Duration
d := 5 * time.Second
time.Sleep(d)

// Timeout
<-time.After(5 * time.Second)

// Ticker (repeated)
ticker := time.NewTicker(1 * time.Second)
<-ticker.C
ticker.Stop()

// Parse/Format
t, _ := time.Parse("2006-01-02", "2024-11-04")
s := t.Format("2006-01-02")
```

---

## 29. What is GOMAXPROCS?

**Q: What is GOMAXPROCS?**

A: Controls number of OS threads Go runtime uses.

```go
import "runtime"

// Set to 4 OS threads
runtime.GOMAXPROCS(4)

// Get current value
numThreads := runtime.GOMAXPROCS(-1)
```

By default, equals number of CPU cores. Only needed for specific tuning.

---

## 30. What is the `sync/atomic` package?

**Q: Explain atomic operations**

A: `atomic` provides lock-free atomic operations on shared variables.

```go
import "sync/atomic"

var counter int64

atomic.AddInt64(&counter, 1)  // Thread-safe increment
val := atomic.LoadInt64(&counter)
```

**Use cases**:

- High-performance counters
- Flags without locks
- Simpler than mutex for simple values

```go
var done int32

go func() {
    doWork()
    atomic.StoreInt32(&done, 1)
}()

for atomic.LoadInt32(&done) == 0 {
    time.Sleep(100 * time.Millisecond)
}
```

---

## Interview Tips

1. **Use examples** — Code beats explanations
2. **Mention philosophy** — "Go favors simplicity"
3. **Know tradeoffs** — No silver bullets
4. **Talk production experience** — "In our system..."
5. **Concurrency is key** — Goroutines/channels are Go's strength
6. **Error handling** — Explicit, not exceptions
7. **Interfaces** — Go's answer to polymorphism
8. **Practical patterns** — Worker pools, pipelines, pub-sub
