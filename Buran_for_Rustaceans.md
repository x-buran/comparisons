# Buran for Rust Programmers

## Introduction

If you know Rust, you already understand pattern matching, algebraic data types, and the power of a strong type system. Buran takes pattern matching further — making it not just a feature, but the *entire* computation model.

This document will help you transition from Rust to Buran by showing correspondences between familiar constructs and new syntax.

---

## The Core Similarity

Rust and Buran both embrace pattern matching. But where Rust uses pattern matching as one tool among many, Buran uses it for *everything*:

**Rust:**

```rust
fn factorial(n: u64) -> u64 {
    match n {
        0 => 1,
        n => n * factorial(n - 1),
    }
}
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

The Buran version *is* the match expression. Functions are just named collections of pattern clauses.

---

## No Ownership, No Borrowing

Rust's ownership system prevents data races and ensures memory safety. Buran takes a different approach: **everything is immutable**.

**Rust:**

```rust
let mut v = vec![1, 2, 3];
v.push(4);  // Mutation requires mut
let r = &v; // Borrowing
```

**Buran:**

```
[list: 1, 2, 3] ↦ v
[list: ⟨v⟩, 4] ↦ v2    # New pattern, v unchanged
```

There's no `mut`, no `&`, no `&mut`, no lifetimes. Patterns transform into new patterns — the original is never modified. This eliminates entire categories of bugs at the cost of different performance characteristics.

---

## Basic Syntax Correspondences

### Variables and Bindings

**Rust:**

```rust
let x = 42;
let name = "hello";
let items = vec![1, 2, 3];
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
```

Key differences:

- All values are in square brackets: `[42]`, `["hello"]`
- Arrow `↦` indicates data flow (bidirectional — `𝑥 ↤ [42]` is equivalent)
- Mathematical italic `𝑥` for math variables, ASCII for general identifiers

### Functions

**Rust:**

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}
```

**Buran:**

```
add {
    𝑎, 𝑏 ↦ [𝑎 + 𝑏]
}

greet {
    name ↦ [string: "Hello, ", name, "!"]
}
```

No type annotations in function signatures — types are determined by pattern structure and identity metadata.

### Function Calls

**Rust:**

```rust
let sum = add(2, 3);
let msg = greet("World");
```

**Buran:**

```
add([2], [3]) ↦ sum
greet("World") ↦ msg
```

Note: Literal arguments need brackets: `add([2], [3])`. Variables don't: `add(𝑥, 𝑦)`.

---

## Pattern Matching: Familiar Territory

Rust's `match` will feel familiar. Buran just makes it the default:

**Rust:**

```rust
fn classify(n: i32) -> &'static str {
    match n {
        n if n > 0 => "positive",
        n if n < 0 => "negative",
        0 => "zero",
        _ => unreachable!(),
    }
}
```

**Buran:**

```
classify {
    [𝑛 | 𝑛 > 0] ↦ ["positive"]
    [𝑛 | 𝑛 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}
```

Guards use `|` instead of `if`, but the concept is identical.

### Destructuring

**Rust:**

```rust
fn first((a, _): (i32, i32)) -> i32 { a }

fn head(v: &[i32]) -> Option<i32> {
    match v {
        [first, ..] => Some(*first),
        [] => None,
    }
}
```

**Buran:**

```
first {
    [𝑎, _] ↦ 𝑎
}

head {
    [list: first, ⟨_⟩] ↦ first
    [list: ] ↦ []
}
```

The `⟨...⟩` mirror brackets capture remaining elements — like Rust's `..` pattern but more powerful.

---

## Enums → Constructors

Rust enums map naturally to Buran constructors:

**Rust:**

```rust
enum Shape {
    Circle(f64),
    Rectangle(f64, f64),
    Triangle(f64, f64, f64),
}

fn area(shape: Shape) -> f64 {
    match shape {
        Shape::Circle(r) => std::f64::consts::PI * r * r,
        Shape::Rectangle(w, h) => w * h,
        Shape::Triangle(a, b, c) => {
            let s = (a + b + c) / 2.0;
            (s * (s-a) * (s-b) * (s-c)).sqrt()
        }
    }
}
```

**Buran:**

```
area {
    [circle: 𝑟] ↦ [𝛑 × 𝑟²]
    [rectangle: 𝑤, ℎ] ↦ [𝑤 × ℎ]
    [triangle: 𝑎, 𝑏, 𝑐] ↦
        [(𝑎 + 𝑏 + 𝑐) ÷ 2] ↦ 𝑠 ↦
        [√(𝑠 × (𝑠 − 𝑎) × (𝑠 − 𝑏) × (𝑠 − 𝑐))]
}
```

Constructors are just pattern names — no enum declaration needed. Type safety comes from identity metadata.

### Defining Types

**Rust:**

```rust
struct Color {
    r: u8,
    g: u8,
    b: u8,
}
```

**Buran:**

```
⟨[rgb: _, _, _]⟩ ↤ [identity:
    "type": "Color",
    "description": "RGB color with three components",
    "constraints": "values between 0-255"
]
```

Types are patterns with identity metadata attached via mirror brackets.

---

## Result and Option → Error Pattern and Empty Pattern

Rust's `Result<T, E>` and `Option<T>` have direct equivalents:

**Rust:**

```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("division by zero".to_string())
    } else {
        Ok(a / b)
    }
}

fn find(items: &[i32], target: i32) -> Option<usize> {
    items.iter().position(|&x| x == target)
}
```

**Buran:**

```
divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

find {
    [list: ], _ ↦ []                           # Empty = None
    [list: target, ⟨_⟩], target ↦ [0]          # Found at index 0
    [list: _, ⟨rest⟩], target ↦
        find(rest, target) ↦ {
            [] ↦ []
            [𝑖] ↦ [𝑖 + 1]
        }
}
```

- `[error: message]` — like `Err(message)`
- `[]` (empty pattern) — like `None`
- Any other pattern — like `Ok(value)` or `Some(value)`

### Error Handling

**Rust:**

```rust
fn process(path: &str) -> Result<Data, Error> {
    let content = std::fs::read_to_string(path)?;
    parse(&content)
}

// Matching on Result
match result {
    Ok(data) => handle_success(data),
    Err(e) => handle_error(e),
}
```

**Buran:**

```
process {
    path ↦ [File: path] ↦ {
        [error: msg] ↦ [error: msg]    # Propagate error
        content ↦ parse(content)
    }
}

# Matching on result
result ↦ {
    [error: e] ↦ handle-error(e)
    data ↦ handle-success(data)
}
```

No `?` operator — error propagation is explicit pattern matching.

### Error Details via Identity

**Buran:**

```
handle {
    ⟨e | "type": "error", "category": "io"⟩ ↦ handle-io(e)
    ⟨e | "type": "error", "category": "parse"⟩ ↦ handle-parse(e)
    ⟨e | "type": "error"⟩ ↦ handle-other(e)
    result ↦ process(result)
}
```

Mirror brackets access error metadata — similar to downcasting error types in Rust.

---

## Traits → Identity

Rust traits define shared behavior. Buran uses identity metadata and pattern matching:

**Rust:**

```rust
trait Area {
    fn area(&self) -> f64;
}

impl Area for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
}

impl Area for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }
}
```

**Buran:**

```
# Define types
⟨[circle: _]⟩ ↤ [identity: "type": "Shape", "variant": "circle"]
⟨[rectangle: _, _]⟩ ↤ [identity: "type": "Shape", "variant": "rectangle"]

# Single function handles all shapes
area {
    [circle: 𝑟] ↦ [𝛑 × 𝑟²]
    [rectangle: 𝑤, ℎ] ↦ [𝑤 × ℎ]
}

# Or dispatch by identity
area {
    ⟨shape | "variant": "circle"⟩ ↦ circle-area(shape)
    ⟨shape | "variant": "rectangle"⟩ ↦ rectangle-area(shape)
}
```

Pattern matching on structure or identity replaces trait dispatch.

---

## Generics → Pattern Polymorphism

Rust generics are explicit. Buran patterns are inherently polymorphic:

**Rust:**

```rust
fn first<T>(items: &[T]) -> Option<&T> {
    items.first()
}

fn map<T, U, F: Fn(T) -> U>(items: Vec<T>, f: F) -> Vec<U> {
    items.into_iter().map(f).collect()
}
```

**Buran:**

```
first {
    [list: head, ⟨_⟩] ↦ head
    [list: ] ↦ []
}

map {
    f, [list: ] ↦ [list: ]
    f, [list: head, ⟨tail⟩] ↦ [list: f(head), ⟨map(f, tail)⟩]
}
```

No `<T>` declarations needed. Patterns match any value with the right structure. Type checking happens through identity guards when needed:

```
# Constrain to integers only
sum {
    [list: ] ↦ [0]
    [list: ⟨head | "type": "integer"⟩, ⟨tail⟩] ↦ [head + sum(tail)]
}
```

---

## Closures → Pattern Blocks

**Rust:**

```rust
let double = |x| x * 2;
let sum = |a, b| a + b;
let is_positive = |x| x > 0;

numbers.iter().map(|x| x * 2).collect()
numbers.iter().filter(|x| **x > 0).collect()
```

**Buran:**

```
{ 𝑥 ↦ 𝑥 × 2 } ↦ double
{ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 } ↦ sum
{ 𝑥 | 𝑥 > 0 } ↦ is-positive

map({ 𝑥 ↦ 𝑥 × 2 }, numbers)
filter({ 𝑥 | 𝑥 > 0 }, numbers)
```

Pattern blocks `{ ... }` are Buran's closures. They can have multiple clauses:

```
{
    [0] ↦ ["zero"],
    [𝑛 | 𝑛 > 0] ↦ ["positive"],
    _ ↦ ["negative"]
}
```

---

## Iterators → Recursion and Higher-Order Functions

Rust's iterator chains become recursive functions or map/filter/fold:

**Rust:**

```rust
let result: i32 = numbers
    .iter()
    .filter(|x| **x > 0)
    .map(|x| x * 2)
    .sum();
```

**Buran:**

```
numbers ↦
filter({ 𝑥 | 𝑥 > 0 }, numbers) ↦ positives ↦
map({ 𝑥 ↦ 𝑥 × 2 }, positives) ↦ doubled ↦
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], doubled) ↦ result
```

Or define the transformation inline:

```
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0],
    map({ 𝑥 ↦ 𝑥 × 2 },
        filter({ 𝑥 | 𝑥 > 0 }, numbers)))
```

---

## Vec, HashMap, HashSet → list, map, set

**Rust:**

```rust
let v: Vec<i32> = vec![1, 2, 3];
let m: HashMap<&str, i32> = [("a", 1), ("b", 2)].into();
let s: HashSet<i32> = [1, 2, 3].into();
```

**Buran:**

```
[list: 1, 2, 3] ↦ v
[map: "a", 1, "b", 2] ↦ m
[set: 1, 2, 3] ↦ s
```

Set operations use mathematical notation:

```
[𝑥 ∈ s]           # membership (like s.contains(&x))
[s1 ∪ s2]         # union
[s1 ∩ s2]         # intersection
[s1 ∖ s2]         # difference
[s1 ⊂ s2]         # proper subset
```

---

## Macros → Just Write Patterns

Rust macros reduce boilerplate. Buran's uniform syntax often eliminates the need:

**Rust:**

```rust
// Need macro for vec!
let v = vec![1, 2, 3];

// Need macro for println!
println!("Hello, {}!", name);

// Need macro for format!
let s = format!("{} + {} = {}", a, b, a + b);
```

**Buran:**

```
# No macros needed — these are just patterns
[list: 1, 2, 3] ↦ v

[string: "Hello, ", name, "!"] ↦ [stdout]

[string: a, " + ", b, " = ", [a + b]] ↦ s
```

The uniform pattern syntax means less need for DSLs.

---

## File I/O

**Rust:**

```rust
use std::fs;

// Read
let content = fs::read_to_string("data.txt")?;

// Write
fs::write("output.txt", data)?;

// JSON
let config: Config = serde_json::from_str(&content)?;
```

**Buran:**

```
# Read
[File: "data.txt"] ↦ content

# Write
data ↦ [File: "output.txt"]

# JSON (built-in)
[File: "config.json" :: json] ↦ config
```

Built-in format support: `:: json`, `:: yaml`, `:: csv`, `:: binary`.

---

## Lifetimes → Not Needed

**Rust:**

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

struct Important<'a> {
    part: &'a str,
}
```

**Buran:**

```
longest {
    𝑥, 𝑦 | length(𝑥) > length(𝑦) ↦ 𝑥
    𝑥, 𝑦 ↦ 𝑦
}

# No lifetime annotations ever
[important: part]
```

No references, no lifetimes, no borrowing. Everything is a value (pattern). This trades Rust's zero-cost abstractions for simpler semantics.

---

## Mathematical Notation

Buran uses proper mathematical symbols:

| Rust                   | Buran | Meaning               |
| ---------------------- | ----- | --------------------- |
| `*`                    | `×`   | Multiplication        |
| `/`                    | `÷`   | Division              |
| `x.pow(2)`             | `𝑥²`  | Exponentiation        |
| `x.sqrt()`             | `√𝑥`  | Square root           |
| `std::f64::consts::PI` | `𝛑`   | Pi                    |
| `<=`                   | `≤`   | Less than or equal    |
| `>=`                   | `≥`   | Greater than or equal |
| `!=`                   | `≠`   | Not equal             |
| `&&`                   | `∧`   | Logical AND           |
| `\|\|`                 | `∨`   | Logical OR            |
| `!`                    | `¬`   | Logical NOT           |
| `true`                 | `[✔]` | Boolean true          |
| `false`                | `[✘]` | Boolean false         |

---

## Comparison Table

| Concept   | Rust                  | Buran                       |
| --------- | --------------------- | --------------------------- |
| Function  | `fn f(x: T) -> U { }` | `f { pattern ↦ result }`    |
| Match     | `match x { ... }`     | Implicit in all functions   |
| Guard     | `if condition`        | `\| condition`              |
| Struct    | `struct S { }`        | `[constructor: fields]`     |
| Enum      | `enum E { A, B }`     | Multiple constructors       |
| Trait     | `trait T { }`         | Identity + pattern matching |
| Generic   | `<T>`                 | Implicit polymorphism       |
| Result    | `Result<T, E>`        | Value or `[error: ...]`     |
| Option    | `Option<T>`           | Value or `[]`               |
| Closure   | `\|x\| x + 1`         | `{ 𝑥 ↦ 𝑥 + 1 }`             |
| Vec       | `vec![1, 2, 3]`       | `[list: 1, 2, 3]`           |
| HashMap   | `HashMap::new()`      | `[map: k1, v1, k2, v2]`     |
| HashSet   | `HashSet::new()`      | `[set: 1, 2, 3]`            |
| Reference | `&x`, `&mut x`        | Not needed                  |
| Lifetime  | `'a`                  | Not needed                  |
| Ownership | Move/Copy/Clone       | All values immutable        |

---

## Example: Binary Tree

**Rust:**

```rust
enum Tree<T> {
    Leaf(T),
    Node(Box<Tree<T>>, Box<Tree<T>>),
}

fn sum(tree: &Tree<i32>) -> i32 {
    match tree {
        Tree::Leaf(n) => *n,
        Tree::Node(left, right) => sum(left) + sum(right),
    }
}

fn map<T, U, F: Fn(T) -> U>(tree: Tree<T>, f: &F) -> Tree<U> {
    match tree {
        Tree::Leaf(x) => Tree::Leaf(f(x)),
        Tree::Node(l, r) => Tree::Node(
            Box::new(map(*l, f)),
            Box::new(map(*r, f)),
        ),
    }
}
```

**Buran:**

```
sum {
    [leaf: 𝑛] ↦ 𝑛
    [node: left, right] ↦ [sum(left) + sum(right)]
}

tree-map {
    f, [leaf: 𝑥] ↦ [leaf: f(𝑥)]
    f, [node: l, r] ↦ [node: tree-map(f, l), tree-map(f, r)]
}
```

No `Box`, no explicit recursion types, no lifetime annotations.

---

## Example: State Machine

**Rust:**

```rust
enum State {
    Idle,
    Running { task: String },
    Paused { task: String, progress: u32 },
    Completed { result: String },
}

fn transition(state: State, event: Event) -> State {
    match (state, event) {
        (State::Idle, Event::Start(task)) =>
            State::Running { task },
        (State::Running { task }, Event::Pause(progress)) =>
            State::Paused { task, progress },
        (State::Paused { task, .. }, Event::Resume) =>
            State::Running { task },
        (State::Running { .. }, Event::Complete(result)) =>
            State::Completed { result },
        (state, _) => state,
    }
}
```

**Buran:**

```
transition {
    [idle], [start: task] ↦ [running: task]
    [running: task], [pause: progress] ↦ [paused: task, progress]
    [paused: task, _], [resume] ↦ [running: task]
    [running: _], [complete: result] ↦ [completed: result]
    state, _ ↦ state
}
```

Pattern matching on tuples of state and event — no enum declarations needed.

---

## What Buran Trades Away

Coming from Rust, you should know what Buran doesn't provide:

1. **Zero-cost abstractions** — Buran prioritizes simplicity over performance
2. **Memory control** — No allocation control, no unsafe, no raw pointers
3. **Compile-time guarantees** — Type checking is softer, identity-based
4. **Concurrency primitives** — No `Send`/`Sync`, no ownership-based safety
5. **Ecosystem** — No crates.io equivalent (yet)

## What Buran Offers Instead

1. **Radical simplicity** — One concept (pattern transformation) for everything
2. **No lifetime puzzles** — Immutability eliminates the need
3. **Mathematical notation** — Write `√𝑥` not `x.sqrt()`
4. **AI-friendly syntax** — Consistent structure for reliable code generation
5. **Unicode identifiers** — Name functions in any language

---

## Philosophical Alignment

Rust asks: *"Who owns this data, and for how long?"*

Buran asks: *"What pattern does this pattern become?"*

Both languages value correctness, but achieve it differently:

- Rust uses ownership and borrowing to prevent data races at compile time
- Buran uses immutability to make data races impossible by design

If you appreciate Rust's pattern matching and algebraic types but find lifetimes frustrating, Buran offers a different trade-off: simpler semantics at the cost of lower-level control.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
