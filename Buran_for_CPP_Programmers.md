# Buran for C++ Programmers

## Introduction

If you know C++, you understand the power of abstraction without sacrificing control. You've mastered templates, RAII, move semantics, and the STL. Buran offers a radically different paradigm: instead of objects with methods, everything is a pattern, and all computation is pattern transformation.

This document will help you transition from C++ to Buran by showing correspondences between familiar constructs and new syntax.

---

## The Paradigm Shift

C++ is multi-paradigm but fundamentally object-oriented and imperative. Buran is purely declarative:

**C++:**

```cpp
int factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n - 1);
}
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

No `if`, no `return`, no type declarations. You declare what patterns transform into, and computation emerges.

---

## No Manual Memory Management

C++ gives you complete control over memory. Buran takes it away entirely:

**C++:**

```cpp
// Raw pointers
int* p = new int(42);
delete p;

// Smart pointers
auto sp = std::make_shared<Widget>();

// RAII
{
    std::lock_guard<std::mutex> lock(mtx);
    // automatically released
}
```

**Buran:**

```
# Just values — no allocation, no deallocation
[42] ↦ value

# No pointers, no references, no smart pointers
# Everything is immutable — no need for locks
```

There are no pointers, no `new`, no `delete`, no smart pointers, no RAII. All values are immutable patterns. The runtime handles memory.

---

## Basic Syntax Correspondences

### Variables

**C++:**

```cpp
int x = 42;
std::string name = "hello";
std::vector<int> items = {1, 2, 3};
const double pi = 3.14159;
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
𝛑    # built-in constant
```

Key differences:

- All values are in square brackets: `[42]`, `["hello"]`
- Arrow `↦` indicates data flow
- No type declarations — types are inferred from patterns
- Mathematical constants like `𝛑` are built-in

### Functions

**C++:**

```cpp
int add(int a, int b) {
    return a + b;
}

std::string greet(const std::string& name) {
    return "Hello, " + name + "!";
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

No return types, no parameter types, no `const`, no references.

### Function Calls

**C++:**

```cpp
int sum = add(2, 3);
std::string msg = greet("World");
```

**Buran:**

```
add([2], [3]) ↦ sum
greet("World") ↦ msg
```

Literal arguments need brackets: `add([2], [3])`. Variables don't: `add(𝑥, 𝑦)`.

---

## Classes → Patterns with Identity

C++ classes encapsulate data and behavior. Buran uses patterns for data and separate functions for behavior:

**C++:**

```cpp
class Color {
public:
    uint8_t r, g, b;

    Color(uint8_t r, uint8_t g, uint8_t b)
        : r(r), g(g), b(b) {}

    int brightness() const {
        return (r + g + b) / 3;
    }

    Color invert() const {
        return Color(255 - r, 255 - g, 255 - b);
    }
};

Color red(255, 0, 0);
int bright = red.brightness();
```

**Buran:**

```
# Define type through identity
⟨[rgb: _, _, _]⟩ ↤ [identity:
    "type": "Color",
    "description": "RGB color with three components"
]

# Create instance
[rgb: 255, 0, 0] ↦ red

# Methods are just functions
brightness {
    [rgb: 𝑟, 𝑔, 𝑏] ↦ [(𝑟 + 𝑔 + 𝑏) ÷ 3]
}

invert {
    [rgb: 𝑟, 𝑔, 𝑏] ↦ [rgb: 255 − 𝑟, 255 − 𝑔, 255 − 𝑏]
}

brightness(red) ↦ bright
```

No constructors, no destructors, no `this` pointer. Data is patterns; behavior is pattern-matching functions.

---

## Inheritance → Pattern Matching

C++ uses inheritance for polymorphism. Buran uses pattern matching:

**C++:**

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() const override {
        return M_PI * radius * radius;
    }
};

class Rectangle : public Shape {
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double area() const override {
        return width * height;
    }
};

void process(const Shape& s) {
    std::cout << s.area() << std::endl;
}
```

**Buran:**

```
# No class hierarchy — just different patterns
area {
    [circle: 𝑟] ↦ [𝛑 × 𝑟²]
    [rectangle: 𝑤, ℎ] ↦ [𝑤 × ℎ]
    [triangle: 𝑎, 𝑏, 𝑐] ↦
        [(𝑎 + 𝑏 + 𝑐) ÷ 2] ↦ 𝑠 ↦
        [√(𝑠 × (𝑠 − 𝑎) × (𝑠 − 𝑏) × (𝑠 − 𝑐))]
}

process {
    shape ↦ area(shape) ↦ [stdout]
}
```

No virtual functions, no vtables, no inheritance hierarchies. Pattern matching dispatches based on structure.

---

## Templates → Implicit Polymorphism

C++ templates provide compile-time polymorphism. Buran patterns are inherently polymorphic:

**C++:**

```cpp
template<typename T>
T first(const std::vector<T>& v) {
    return v.empty() ? T{} : v[0];
}

template<typename T, typename F>
std::vector<T> map(const std::vector<T>& v, F f) {
    std::vector<T> result;
    for (const auto& x : v) {
        result.push_back(f(x));
    }
    return result;
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

No `template<typename T>`, no angle brackets. Patterns match any value with the right structure.

### Constraining Types

When you need type constraints (like C++20 concepts):

**C++:**

```cpp
template<std::integral T>
T sum(const std::vector<T>& v) {
    return std::accumulate(v.begin(), v.end(), T{});
}
```

**Buran:**

```
sum {
    [list: ] ↦ [0]
    [list: ⟨head | "type": "integer"⟩, ⟨tail⟩] ↦ [head + sum(tail)]
}
```

Identity guards (`⟨x | "type": "integer"⟩`) constrain types.

---

## STL Containers → Built-in Patterns

**C++:**

```cpp
std::vector<int> v = {1, 2, 3};
std::map<std::string, int> m = {{"a", 1}, {"b", 2}};
std::set<int> s = {1, 2, 3};
std::optional<int> opt = 42;
```

**Buran:**

```
[list: 1, 2, 3] ↦ v
[map: "a", 1, "b", 2] ↦ m
[set: 1, 2, 3] ↦ s
[42] ↦ opt              # or [] for empty
```

### Container Operations

**C++:**

```cpp
// Vector operations
v.push_back(4);
int first = v[0];
auto it = std::find(v.begin(), v.end(), 2);

// Set operations
s.insert(4);
bool has = s.count(2) > 0;
auto u = set_union(s1, s2);
```

**Buran:**

```
# Create new list (immutable)
[list: ⟨v⟩, 4] ↦ v2

# Destructure
[list: first, ⟨_⟩] ↦ v

# Set operations use mathematical notation
[set: ⟨s⟩, 4] ↦ s2        # new set with 4 added
[2 ∈ s] ↦ has             # membership
[s1 ∪ s2] ↦ u             # union
[s1 ∩ s2] ↦ i             # intersection
```

---

## STL Algorithms → Higher-Order Functions

**C++:**

```cpp
std::vector<int> doubled;
std::transform(v.begin(), v.end(),
               std::back_inserter(doubled),
               [](int x) { return x * 2; });

std::vector<int> evens;
std::copy_if(v.begin(), v.end(),
             std::back_inserter(evens),
             [](int x) { return x % 2 == 0; });

int sum = std::accumulate(v.begin(), v.end(), 0,
                          [](int a, int b) { return a + b; });
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 × 2 }, v) ↦ doubled

filter({ 𝑥 | 𝑥 mod 2 = 0 }, v) ↦ evens

fold({ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 }, [0], v) ↦ sum
```

Pattern blocks `{ ... }` replace lambdas. No iterators, no `begin()`/`end()`.

---

## Lambdas → Pattern Blocks

**C++:**

```cpp
auto add = [](int a, int b) { return a + b; };
auto isPositive = [](int x) { return x > 0; };
auto factorial = [](auto self, int n) -> int {
    return n == 0 ? 1 : n * self(self, n - 1);
};
```

**Buran:**

```
{ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 } ↦ add
{ 𝑥 | 𝑥 > 0 } ↦ is-positive

# Pattern blocks can have multiple clauses
{
    [0] ↦ [1],
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
} ↦ factorial-block
```

---

## Exceptions → Error Patterns

**C++:**

```cpp
double divide(double a, double b) {
    if (b == 0) throw std::runtime_error("division by zero");
    return a / b;
}

try {
    double result = divide(10, 0);
} catch (const std::exception& e) {
    std::cerr << e.what() << std::endl;
}
```

**Buran:**

```
divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

divide([10], [0]) ↦ {
    [error: msg] ↦ msg ↦ [stderr]
    result ↦ process(result)
}
```

No `try`/`catch`/`throw`. Errors are just patterns to match.

### Error Categories

**C++:**

```cpp
try {
    // ...
} catch (const std::ios_base::failure& e) {
    // I/O error
} catch (const std::logic_error& e) {
    // Logic error
} catch (...) {
    // Unknown error
}
```

**Buran:**

```
result ↦ {
    ⟨e | "type": "error", "category": "io"⟩ ↦ handle-io(e)
    ⟨e | "type": "error", "category": "logic"⟩ ↦ handle-logic(e)
    ⟨e | "type": "error"⟩ ↦ handle-unknown(e)
    value ↦ process(value)
}
```

Mirror brackets access error metadata for categorized handling.

---

## References and Pointers → Not Needed

**C++:**

```cpp
void modify(int& x) { x = 42; }
void read(const int& x) { std::cout << x; }
int* findMax(std::vector<int>& v) {
    return &*std::max_element(v.begin(), v.end());
}
```

**Buran:**

```
# No references — return new values
modify {
    _ ↦ [42]
}

# No const — everything is immutable
read {
    𝑥 ↦ 𝑥 ↦ [stdout]
}

# No pointers — return values directly
find-max {
    [list: ] ↦ []
    [list: 𝑥] ↦ 𝑥
    [list: 𝑥, ⟨rest⟩] ↦
        find-max(rest) ↦ {
            [] ↦ 𝑥
            [𝑚 | 𝑚 > 𝑥] ↦ 𝑚
            _ ↦ 𝑥
        }
}
```

No `&`, no `*`, no `const&`, no `&&`. Immutability eliminates the need for reference semantics.

---

## Move Semantics → Not Needed

**C++:**

```cpp
class Buffer {
    std::unique_ptr<char[]> data;
public:
    Buffer(Buffer&& other) noexcept
        : data(std::move(other.data)) {}

    Buffer& operator=(Buffer&& other) noexcept {
        data = std::move(other.data);
        return *this;
    }
};
```

**Buran:**

```
# No move semantics — all values are immutable
# No performance optimization needed for "moving" data
[buffer: data] ↦ buf1
buf1 ↦ buf2    # Conceptually a copy, but runtime optimizes
```

The runtime handles data sharing transparently.

---

## Operator Overloading → Mathematical Notation

C++ lets you overload operators. Buran uses standard mathematical notation:

**C++:**

```cpp
class Complex {
public:
    double re, im;
    Complex operator+(const Complex& other) const {
        return {re + other.re, im + other.im};
    }
    Complex operator*(const Complex& other) const {
        return {re*other.re - im*other.im,
                re*other.im + im*other.re};
    }
};
```

**Buran:**

```
# Mathematical operators are built-in
# For custom types, define functions

complex-add {
    [complex: 𝑎, 𝑏], [complex: 𝑐, 𝑑] ↦ [complex: 𝑎 + 𝑐, 𝑏 + 𝑑]
}

complex-mul {
    [complex: 𝑎, 𝑏], [complex: 𝑐, 𝑑] ↦
        [complex: 𝑎×𝑐 − 𝑏×𝑑, 𝑎×𝑑 + 𝑏×𝑐]
}
```

Built-in mathematical operators: `×`, `÷`, `√`, `²`, `³`, `𝛑`, `≤`, `≥`, `≠`.

---

## File I/O

**C++:**

```cpp
#include <fstream>

// Read
std::ifstream in("data.txt");
std::string content((std::istreambuf_iterator<char>(in)),
                     std::istreambuf_iterator<char>());

// Write
std::ofstream out("output.txt");
out << data;

// JSON (requires library)
#include <nlohmann/json.hpp>
auto j = nlohmann::json::parse(content);
```

**Buran:**

```
# Read
[File: "data.txt"] ↦ content

# Write
data ↦ [File: "output.txt"]

# JSON (built-in)
[File: "config.json" :: json] ↦ config
config ↦ [File: "output.json" :: json]
```

Built-in format support: `:: json`, `:: yaml`, `:: csv`, `:: binary`.

---

## RAII → Automatic

**C++:**

```cpp
class FileHandle {
    FILE* handle;
public:
    FileHandle(const char* path) : handle(fopen(path, "r")) {}
    ~FileHandle() { if (handle) fclose(handle); }
    // Rule of five...
};

{
    FileHandle f("data.txt");
    // automatically closed at scope end
}
```

**Buran:**

```
# Resources managed automatically
[File: "data.txt"] ↦ content
# No explicit cleanup needed
```

No destructors, no scope-based cleanup. The runtime manages resources.

---

## Conditionals and Loops

**C++:**

```cpp
// If-else
if (x > 0) {
    result = "positive";
} else if (x < 0) {
    result = "negative";
} else {
    result = "zero";
}

// For loop
for (int i = 0; i < n; ++i) {
    process(i);
}

// Range-for
for (const auto& item : items) {
    process(item);
}

// While
while (condition) {
    // ...
}
```

**Buran:**

```
# Pattern matching replaces if-else
classify {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}

# Recursion replaces loops
process-range {
    [0] ↦ []
    [𝑛] ↦ process([𝑛 − 1]) ↦ _ ↦ process-range([𝑛 − 1])
}

# Map replaces range-for
map({ item ↦ process(item) }, items)

# Tail recursion replaces while
process-while {
    state | ¬condition(state) ↦ state
    state ↦ process-while(next-state(state))
}
```

No `if`, no `for`, no `while`, no `switch`. Pattern matching and recursion handle all control flow.

---

## Mathematical Notation

Buran uses proper mathematical symbols instead of ASCII approximations:

| C++         | Buran | Meaning               |
| ----------- | ----- | --------------------- |
| `*`         | `×`   | Multiplication        |
| `/`         | `÷`   | Division              |
| `pow(x, 2)` | `𝑥²`  | Square                |
| `sqrt(x)`   | `√𝑥`  | Square root           |
| `cbrt(x)`   | `∛𝑥`  | Cube root             |
| `M_PI`      | `𝛑`   | Pi                    |
| `M_E`       | `𝐞`   | Euler's number        |
| `<=`        | `≤`   | Less than or equal    |
| `>=`        | `≥`   | Greater than or equal |
| `!=`        | `≠`   | Not equal             |
| `&&`        | `∧`   | Logical AND           |
| `\|\|`      | `∨`   | Logical OR            |
| `!`         | `¬`   | Logical NOT           |
| `true`      | `[✔]` | Boolean true          |
| `false`     | `[✘]` | Boolean false         |
| `nullptr`   | `[]`  | Empty/null            |

---

## Comparison Table

| Concept       | C++                        | Buran                              |
| ------------- | -------------------------- | ---------------------------------- |
| Function      | `int f(int x) { }`         | `f { pattern ↦ result }`           |
| Class         | `class C { };`             | `[constructor: fields]` + identity |
| Inheritance   | `class D : public B`       | Pattern matching                   |
| Virtual       | `virtual void f()`         | Pattern dispatch                   |
| Template      | `template<typename T>`     | Implicit polymorphism              |
| Lambda        | `[](int x) { return x; }`  | `{ 𝑥 ↦ 𝑥 }`                        |
| Exception     | `throw`, `try`, `catch`    | `[error: ...]` patterns            |
| Pointer       | `int*`, `->`               | Not needed                         |
| Reference     | `int&`, `const int&`       | Not needed                         |
| Move          | `std::move`                | Not needed                         |
| Smart pointer | `shared_ptr`, `unique_ptr` | Not needed                         |
| RAII          | Destructors                | Automatic                          |
| Vector        | `std::vector<T>`           | `[list: ...]`                      |
| Map           | `std::map<K,V>`            | `[map: k1, v1, ...]`               |
| Set           | `std::set<T>`              | `[set: ...]`                       |
| Optional      | `std::optional<T>`         | Value or `[]`                      |
| Variant       | `std::variant<T...>`       | Different constructors             |

---

## Example: Linked List

**C++:**

```cpp
template<typename T>
struct Node {
    T value;
    std::unique_ptr<Node> next;
};

template<typename T>
int length(const Node<T>* head) {
    if (!head) return 0;
    return 1 + length(head->next.get());
}

template<typename T>
Node<T>* reverse(Node<T>* head, Node<T>* prev = nullptr) {
    if (!head) return prev;
    auto next = std::move(head->next);
    head->next.reset(prev);
    return reverse(next.release(), head);
}
```

**Buran:**

```
length {
    [list: ] ↦ [0]
    [list: _, ⟨rest⟩] ↦ [1 + length(rest)]
}

reverse {
    [list: ] ↦ [list: ]
    [list: head, ⟨rest⟩] ↦ [list: ⟨reverse(rest)⟩, head]
}
```

No pointers, no `nullptr` checks, no memory management.

---

## Example: Expression Evaluator

**C++:**

```cpp
struct Expr {
    virtual ~Expr() = default;
    virtual int eval() const = 0;
};

struct Num : Expr {
    int value;
    Num(int v) : value(v) {}
    int eval() const override { return value; }
};

struct Add : Expr {
    std::unique_ptr<Expr> left, right;
    Add(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r)
        : left(std::move(l)), right(std::move(r)) {}
    int eval() const override {
        return left->eval() + right->eval();
    }
};

struct Mul : Expr {
    std::unique_ptr<Expr> left, right;
    Mul(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r)
        : left(std::move(l)), right(std::move(r)) {}
    int eval() const override {
        return left->eval() * right->eval();
    }
};
```

**Buran:**

```
eval {
    [num: 𝑛] ↦ 𝑛
    [add: left, right] ↦ [eval(left) + eval(right)]
    [mul: left, right] ↦ [eval(left) × eval(right)]
}

# Example expression: (2 + 3) * 4
[mul: [add: [num: 2], [num: 3]], [num: 4]] ↦ expr
eval(expr) ↦ result    # [20]
```

No class hierarchy, no virtual functions, no unique_ptr.

---

## What Buran Trades Away

Coming from C++, you should know what Buran doesn't provide:

1. **Performance control** — No cache optimization, no SIMD, no inline assembly
2. **Memory layout** — No struct packing, no alignment control
3. **Deterministic destruction** — No RAII with precise timing
4. **Zero-overhead abstractions** — Runtime handles everything
5. **Direct hardware access** — No pointers, no memory-mapped I/O
6. **Compile-time computation** — No constexpr, no template metaprogramming

## What Buran Offers Instead

1. **Radical simplicity** — One concept (pattern transformation) for everything
2. **No memory bugs** — No dangling pointers, no use-after-free, no leaks
3. **No undefined behavior** — Everything is well-defined
4. **Mathematical notation** — Write `√𝑥` not `sqrt(x)`
5. **AI-friendly syntax** — Consistent structure for reliable code generation
6. **Unicode identifiers** — Name functions in any language

---

## Philosophical Difference

C++ asks: *"How do I represent this in memory and manipulate it efficiently?"*

Buran asks: *"What pattern does this pattern become?"*

C++ gives you control over every aspect of computation, from memory layout to CPU instructions. Buran takes that control away in exchange for simplicity and safety.

If you appreciate C++'s expressive type system and pattern matching (structured bindings, `std::visit`) but find memory management tedious, Buran offers a different trade-off: simpler semantics at the cost of lower-level control.

For systems programming, embedded systems, or performance-critical code, C++ remains the right choice. For applications where correctness and maintainability matter more than raw performance, Buran offers a compelling alternative.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
