# Buran for Go Programmers

## Introduction

If you program in Go, you value simplicity. Go deliberately omits features other languages consider essential — no inheritance, no exceptions, no generics (until recently). The philosophy: simple tools that compose well beat complex tools that do everything.

Buran shares this commitment to simplicity, but arrives there from a different direction. Where Go is imperative with explicit control flow, Buran is declarative with pattern transformation. Where Go uses explicit error returns, Buran uses error patterns. Where Go has nil, Buran has explicit emptiness.

Both languages believe less is more. This guide shows how Go concepts map to Buran.

---

## Shared Philosophy

| Principle    | Go                            | Buran                  |
| ------------ | ----------------------------- | ---------------------- |
| Simplicity   | Few keywords, clear semantics | One syntax per concept |
| Explicitness | No hidden control flow        | No implicit coercion   |
| Composition  | Interfaces + embedding        | Pattern matching       |
| Clarity      | gofmt, clear idioms           | Mathematical notation  |
| No magic     | Explicit error handling       | Explicit patterns      |

Go's famous proverb "Clear is better than clever" applies equally to Buran.

---

## Variables and Assignment

### Go

```go
x := 5
var name string = "Alice"
count := count + 1    // mutation

// Multiple assignment
a, b := 1, 2
a, b = b, a           // swap
```

### Buran

```
[5] ↦ x
["Alice"] ↦ name

# No mutation — create new binding
[count + 1] ↦ new-count

# Multiple values via patterns
[1] ↦ a ↦
[2] ↦ b

# "Swap" is just creating new bindings
[pair: a, b] ↦ [pair: new-a, new-b]    # Destructure
[pair: b, a] ↦ [pair: swapped-a, swapped-b]
```

Key difference: Buran values are immutable. You don't change `x`, you create a new binding.

---

## Functions

### Go

```go
func add(a, b int) int {
    return a + b
}

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Multiple return values
func swap(a, b int) (int, int) {
    return b, a
}
```

### Buran

```
add {
    𝑎, 𝑏 ↦ [𝑎 + 𝑏]
}

divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

# Multiple values via constructor
swap {
    𝑎, 𝑏 ↦ [pair: 𝑏, 𝑎]
}
```

Go returns `(result, error)` tuples. Buran returns either a result or an error pattern — they're mutually exclusive through pattern matching.

---

## Error Handling

This is where Go and Buran differ most visibly.

### Go's Explicit Error Checking

```go
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doSomething failed: %w", err)
}

file, err := os.Open("data.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()

// Chain of error checks
a, err := step1()
if err != nil {
    return err
}
b, err := step2(a)
if err != nil {
    return err
}
c, err := step3(b)
if err != nil {
    return err
}
```

### Buran's Error Patterns

```
do-something() ↦ {
    [error: e] ↦ [error: [string: "doSomething failed: ", e]]
    result ↦ continue-with(result)
}

[File: "data.txt"] ↦ {
    [error: e] ↦ handle-error(e)
    content ↦ process(content)
}

# Chain with pattern matching
step1() ↦ {
    [error: e] ↦ [error: e]
    a ↦ step2(a) ↦ {
        [error: e] ↦ [error: e]
        b ↦ step3(b)
    }
}
```

Or define a pipeline helper:

```
chain {
    [error: e], _ ↦ [error: e]
    value, f ↦ f(value)
}

# Then chain operations
step1() ↦ r1 ↦
chain(r1, step2) ↦ r2 ↦
chain(r2, step3)
```

Both approaches make errors explicit. Go uses `if err != nil`; Buran uses pattern matching on `[error: ...]`.

---

## Nil vs Empty Pattern

Go's `nil` causes runtime panics if misused:

### Go

```go
var s []int          // nil slice
var m map[string]int // nil map - will panic on write!
var p *int           // nil pointer

if s == nil {
    s = make([]int, 0)
}

// The billion dollar mistake
func findUser(id int) *User {
    // might return nil
}

user := findUser(123)
user.Name  // PANIC if nil!
```

### Buran

```
# Empty pattern is explicit
[]

# Empty list
[list: ]

# Empty map
[map: ]

# Pattern matching handles presence/absence
find-user(123) ↦ {
    [] ↦ ["user not found"]
    [user: name, email] ↦ ["found: ", name]
}

# No nil pointer panics — can't happen
```

Buran has no nil pointers because it has no pointers. The empty pattern `[]` is a real value you can match and compare.

---

## Structs vs Constructors

### Go Structs

```go
type Point struct {
    X, Y float64
}

type Circle struct {
    Center Point
    Radius float64
}

p := Point{X: 3, Y: 4}
c := Circle{Center: p, Radius: 5}

// Access fields
fmt.Println(p.X, p.Y)

// Methods
func (p Point) Distance() float64 {
    return math.Sqrt(p.X*p.X + p.Y*p.Y)
}
```

### Buran Constructors

```
# Define with identity
⟨[point: 𝑥, 𝑦]⟩ ↤ [identity: "type": "Point"]
⟨[circle: center, radius]⟩ ↤ [identity: "type": "Circle"]

# Create
[point: 3, 4] ↦ p
[circle: p, 5] ↦ c

# Access via pattern matching
p ↦ { [point: 𝑥, 𝑦] ↦ [𝑥, 𝑦] }

# "Methods" are just functions
distance {
    [point: 𝑥, 𝑦] ↦ [√(𝑥² + 𝑦²)]
}
```

Go attaches methods to types. Buran functions pattern-match on constructors — same effect, different mechanism.

---

## Interfaces

Go's interfaces are implicitly satisfied — any type with the right methods implements the interface.

### Go

```go
type Stringer interface {
    String() string
}

// Point implements Stringer implicitly
func (p Point) String() string {
    return fmt.Sprintf("(%v, %v)", p.X, p.Y)
}

// Works with any Stringer
func Print(s Stringer) {
    fmt.Println(s.String())
}
```

### Buran: Pattern-Based Polymorphism

```
# Define string representation for different types
to-string {
    [point: 𝑥, 𝑦] ↦ [string: "(", 𝑥, ", ", 𝑦, ")"]
    [circle: center, radius] ↦ [string: "Circle(", to-string(center), ", r=", radius, ")"]
    [user: name, _] ↦ [string: "User: ", name]
    value ↦ [string: value]    # fallback
}

# Works with any pattern
print-it {
    thing ↦ to-string(thing) ↦ [stdout]
}
```

Buran achieves polymorphism through pattern matching — different patterns, different behavior. No interfaces to declare.

---

## Slices and Maps

### Go

```go
// Slices
nums := []int{1, 2, 3, 4, 5}
nums = append(nums, 6)
first := nums[0]
rest := nums[1:]

// Maps
ages := map[string]int{
    "Alice": 30,
    "Bob":   25,
}
ages["Charlie"] = 35
age, ok := ages["Alice"]
```

### Buran

```
# Lists
[list: 1, 2, 3, 4, 5] ↦ nums

# Append
[list: ⟨nums⟩, 6] ↦ new-nums

# First and rest via pattern matching
[list: first, ⟨rest⟩] ↦ nums

# Maps
[map: "Alice", 30, "Bob", 25] ↦ ages

# Add entry (creates new map)
[map: ⟨ages⟩, "Charlie", 35] ↦ new-ages

# Lookup with pattern matching
lookup {
    key, [map: ] ↦ []    # Not found
    key, [map: k, v, ⟨rest⟩] | k = key ↦ v
    key, [map: _, _, ⟨rest⟩] ↦ lookup(key, [map: ⟨rest⟩])
}
```

Go's slices and maps are mutable. Buran's lists and maps are immutable — operations produce new collections.

---

## Control Flow

### Go

```go
// If-else
if x > 0 {
    fmt.Println("positive")
} else if x < 0 {
    fmt.Println("negative")
} else {
    fmt.Println("zero")
}

// Switch
switch day {
case "Sat", "Sun":
    fmt.Println("weekend")
default:
    fmt.Println("weekday")
}

// Type switch
switch v := i.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
default:
    fmt.Println("unknown")
}
```

### Buran

```
# Pattern matching replaces if-else
classify {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}

# Pattern matching replaces switch
day-type {
    [sat] ↦ ["weekend"]
    [sun] ↦ ["weekend"]
    _ ↦ ["weekday"]
}

# Type matching via identity
type-switch {
    ⟨v | "type": "integer"⟩ ↦ [string: "int: ", v]
    ⟨v | "type": "string"⟩ ↦ [string: "string: ", v]
    _ ↦ ["unknown"]
}
```

Go has `if`, `switch`, and type switches. Buran has pattern matching — one mechanism for all branching.

---

## Loops vs Recursion

### Go

```go
// For loop
sum := 0
for i := 1; i <= 10; i++ {
    sum += i
}

// Range over slice
for i, v := range nums {
    fmt.Println(i, v)
}

// While-style
for count > 0 {
    count--
}
```

### Buran

```
# Sum via fold (no loops)
fold({ acc, 𝑖 ↦ acc + 𝑖 }, [0], [list: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# Process each element via map
map({ 𝑣 ↦ process(𝑣) }, nums)

# Recursion for custom iteration
countdown {
    [0] ↦ ["done"]
    [𝑛] ↦ countdown([𝑛 − 1])
}

# With accumulator for sum
sum-to {
    [𝑛] ↦ sum-loop(𝑛, [0])
}

sum-loop {
    [0], acc ↦ acc
    [𝑛], acc ↦ sum-loop([𝑛 − 1], [acc + 𝑛])
}
```

Go has `for` loops. Buran uses `map`, `filter`, `fold`, and recursion.

---

## Goroutines and Channels

Go's concurrency model is one of its defining features:

### Go

```go
ch := make(chan int)

go func() {
    ch <- computeValue()
}()

result := <-ch

// Select
select {
case v := <-ch1:
    process(v)
case ch2 <- value:
    // sent
case <-time.After(time.Second):
    // timeout
}
```

### Buran

**Buran does not have built-in concurrency primitives.**

Buran is a pure pattern transformation language. Concurrency would be:

- Implementation-specific (how the runtime executes)
- Potentially a future extension via domains
- Currently not part of the language specification

If you need concurrent processing, you'd either:

- Use Buran for the transformation logic, orchestrated by a concurrent runtime
- Wait for future language extensions

This is a significant difference. Go excels at concurrent systems; Buran focuses on transformation logic.

---

## Defer

### Go

```go
func process() {
    file, _ := os.Open("data.txt")
    defer file.Close()

    // work with file
    // Close() called when function returns
}
```

### Buran

Buran doesn't need `defer` because:

- No resources to manually close (immutable data)
- No early returns to manage
- File connections are handled by the runtime

```
# File is read, no cleanup needed
[File: "data.txt"] ↦ content ↦
process(content)
```

The runtime manages connection lifecycles.

---

## Panic and Recover

### Go

```go
func risky() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered:", r)
        }
    }()
    panic("something went wrong")
}
```

### Buran

Buran has no panic/recover — errors are patterns, not exceptions:

```
# Errors flow through pattern matching
risky-operation() ↦ {
    [error: e] ↦ ["recovered: ", e]
    result ↦ result
}
```

No stack unwinding, no surprise control flow. Errors are data.

---

## Embedding vs Pattern Composition

### Go Embedding

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Embedding
type ReadWriter interface {
    Reader
    Writer
}

// Struct embedding
type File struct {
    name string
}

type LoggingFile struct {
    File  // embedded
    log *Logger
}
```

### Buran: Patterns Compose Naturally

```
# Patterns can contain patterns
[logging-file: [file: name], [logger: level]] ↦ lf

# Destructure nested patterns
process {
    [logging-file: [file: name], logger] ↦
        [string: "Processing: ", name]
}

# "Interface" behavior via pattern matching
read {
    [file: name] ↦ [File: name]
    [logging-file: [file: name], _] ↦ [File: name]
    [network: host, port] ↦ [Network: host, port]
}
```

Composition in Buran is structural — patterns contain patterns.

---

## Type System Comparison

| Go                | Buran                 |
| ----------------- | --------------------- |
| Static typing     | Pattern-based typing  |
| Type declarations | Identity metadata     |
| Interfaces        | Pattern matching      |
| Generics (1.18+)  | Implicit polymorphism |
| Type assertions   | Identity guards       |
| Zero values       | Empty pattern `[]`    |

### Go Generics vs Buran

```go
// Go generic function
func Map[T, U any](f func(T) U, slice []T) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = f(v)
    }
    return result
}
```

```
# Buran — naturally polymorphic
map {
    f, [list: ] ↦ [list: ]
    f, [list: head, ⟨tail⟩] ↦ [list: f(head), ⟨map(f, tail)⟩]
}

# Works with any pattern type
map({ 𝑥 ↦ 𝑥 × 2 }, [list: 1, 2, 3])
map({ s ↦ uppercase(s) }, [list: "a", "b", "c"])
```

Buran functions are polymorphic by default — patterns match structurally.

---

## Complete Example: HTTP-style Handler

### Go

```go
type Request struct {
    Method string
    Path   string
    Body   string
}

type Response struct {
    Status int
    Body   string
}

func handler(req Request) Response {
    switch req.Method {
    case "GET":
        if req.Path == "/users" {
            return Response{200, getUsers()}
        }
        return Response{404, "Not found"}
    case "POST":
        if req.Path == "/users" {
            return Response{201, createUser(req.Body)}
        }
        return Response{404, "Not found"}
    default:
        return Response{405, "Method not allowed"}
    }
}
```

### Buran

```
⟨[request: method, path, body]⟩ ↤ [identity: "type": "Request"]
⟨[response: status, body]⟩ ↤ [identity: "type": "Response"]

handler {
    [request: "GET", "/users", _] ↦ [response: 200, get-users()]
    [request: "POST", "/users", body] ↦ [response: 201, create-user(body)]
    [request: "GET", _, _] ↦ [response: 404, "Not found"]
    [request: "POST", _, _] ↦ [response: 404, "Not found"]
    [request: _, _, _] ↦ [response: 405, "Method not allowed"]
}
```

Pattern matching replaces nested switch statements.

---

## What Go Does Better

1. **Concurrency** — Goroutines and channels are exceptional
2. **Performance** — Compiled, optimized, predictable
3. **Tooling** — go build, go test, go fmt, go mod
4. **Standard library** — Comprehensive and consistent
5. **Deployment** — Single binary, no dependencies
6. **Large codebases** — Designed for teams and scale

---

## What Buran Offers

1. **Pattern matching** — One mechanism for all branching
2. **No nil panics** — Empty pattern is explicit
3. **Mathematical notation** — `×`, `÷`, `√`, `𝛑` instead of `*`, `/`, `math.Sqrt`, `math.Pi`
4. **Immutability** — No data races by design
5. **Simpler error handling** — Pattern match, don't check
6. **No loops** — map/filter/fold express intent directly

---

## Translation Quick Reference

| Go                  | Buran                |
| ------------------- | -------------------- |
| `x := 5`            | `[5] ↦ x`            |
| `func f(x int) int` | `f { [x] ↦ result }` |
| `if err != nil`     | `[error: e] ↦ ...`   |
| `nil`               | `[]`                 |
| `struct{X, Y int}`  | `[point: x, y]`      |
| `[]int{1,2,3}`      | `[list: 1, 2, 3]`    |
| `map[string]int{}`  | `[map: ]`            |
| `switch x {}`       | `x ↦ { patterns }`   |
| `for i := range`    | `map({ ... }, list)` |
| `interface{}`       | Pattern matching     |
| `*T` (pointer)      | N/A (no pointers)    |
| `go func(){}`       | N/A (no goroutines)  |
| `<-chan`            | N/A (no channels)    |
| `a + b`             | `[a + b]`            |
| `a * b`             | `[a × b]`            |
| `math.Sqrt(x)`      | `[√x]`               |
| `true` / `false`    | `[✔]` / `[✘]`        |

---

## Summary

Go and Buran share a commitment to simplicity and explicitness. Go achieves this through a minimal imperative language with excellent concurrency. Buran achieves it through pattern transformation with mathematical notation.

If you're building networked services, CLI tools, or concurrent systems: Go is excellent.
If you're transforming data, expressing algorithms, or want mathematical clarity: consider Buran.

Both languages respect your time by keeping things simple.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
