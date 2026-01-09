# Buran for Julia Programmers

## Introduction

If you program in Julia, you appreciate a language that takes scientific computing seriously. Julia gives you mathematical notation with `α`, `β`, `∑`. It gives you multiple dispatch that feels like pattern matching. It gives you high performance without sacrificing expressiveness.

Buran shares Julia's love of mathematical notation and extends it further. Where Julia uses multiple dispatch to select methods, Buran uses pattern matching to select transformations. Where Julia allows Unicode identifiers, Buran uses Unicode operators as first-class syntax. Both languages believe code should look like mathematics.

This guide shows how your Julia intuitions translate to Buran.

---

## Unicode: A Shared Love

Both Julia and Buran embrace Unicode. This isn't superficial — it changes how you think about code.

### Julia

```julia
α = 0.5
β = 1.0 - α
∑ = sum
Δx = x₂ - x₁

# Unicode operators
÷(a, b) = div(a, b)   # Integer division
```

### Buran

```
[0.5] ↦ α
[1.0 − α] ↦ β
[𝑥₂ − 𝑥₁] ↦ Δx

# Unicode operators are built-in
[10 ÷ 3] ↦ [3.33...]    # Division
[10 × 3] ↦ [30]         # Multiplication
[√16] ↦ [4]             # Square root
[2³] ↦ [8]              # Exponentiation
```

Buran goes further — mathematical operators like `×`, `÷`, `√`, `∑`, `∏` are part of the language, not user-defined.

---

## Multiple Dispatch vs Pattern Matching

Julia's signature feature is multiple dispatch. Buran's foundation is pattern matching. They're deeply related.

### Julia Multiple Dispatch

```julia
# Methods dispatched on argument types
area(r::Circle) = π * r.radius^2
area(r::Rectangle) = r.width * r.height
area(t::Triangle) = 0.5 * t.base * t.height

# Dispatch on multiple arguments
collide(a::Circle, b::Circle) = circle_circle_collision(a, b)
collide(a::Circle, b::Rectangle) = circle_rect_collision(a, b)
collide(a::Rectangle, b::Rectangle) = rect_rect_collision(a, b)
```

### Buran Pattern Matching

```
area {
    [circle: radius] ↦ [𝛑 × radius²]
    [rectangle: width, height] ↦ [width × height]
    [triangle: base, height] ↦ [0.5 × base × height]
}

collide {
    [circle: r1], [circle: r2] ↦ circle-circle-collision(r1, r2)
    [circle: r], [rectangle: w, h] ↦ circle-rect-collision(r, w, h)
    [rectangle: w1, h1], [rectangle: w2, h2] ↦ rect-rect-collision(w1, h1, w2, h2)
}
```

The conceptual mapping:

- Julia method signature → Buran pattern clause
- Julia type dispatch → Buran constructor matching
- Julia `::Type` annotation → Buran `[constructor: ...]` pattern

Both select behavior based on the structure of inputs.

---

## Functions

### Julia

```julia
# Short form
f(x) = x^2

# Long form
function factorial(n)
    n == 0 ? 1 : n * factorial(n - 1)
end

# Multiple methods
sign(x::Number) = x > 0 ? 1 : x < 0 ? -1 : 0
sign(x::Complex) = x / abs(x)

# Anonymous functions
map(x -> x^2, [1, 2, 3, 4])
```

### Buran

```
# Single pattern
f {
    [𝑥] ↦ [𝑥²]
}

# Multiple patterns (like multiple methods)
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}

sign {
    [𝑥 | 𝑥 > 0] ↦ [1]
    [𝑥 | 𝑥 < 0] ↦ [−1]
    [0] ↦ [0]
}

# Pattern blocks (anonymous functions)
map({ 𝑥 ↦ 𝑥² }, [list: 1, 2, 3, 4])
```

Julia's multiple methods become Buran's multiple pattern clauses in one function.

---

## do-Block Syntax vs Pattern Blocks

Julia's `do` blocks create anonymous functions inline:

### Julia

```julia
map([1, 2, 3]) do x
    x^2 + 1
end

filter([1, 2, 3, 4, 5]) do x
    x > 2
end

reduce(0, [1, 2, 3, 4]) do acc, x
    acc + x
end
```

### Buran

```
map({ 𝑥 ↦ 𝑥² + 1 }, [list: 1, 2, 3])

filter({ 𝑥 | 𝑥 > 2 }, [list: 1, 2, 3, 4, 5])

fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], [list: 1, 2, 3, 4])
```

Pattern blocks `{ }` are Buran's anonymous functions. Note:

- `filter` uses a guard pattern `{ x | condition }` not a predicate function
- Arguments come after the pattern block in Buran

---

## Type System

### Julia Types

```julia
# Abstract type hierarchy
abstract type Shape end

struct Circle <: Shape
    radius::Float64
end

struct Rectangle <: Shape
    width::Float64
    height::Float64
end

# Parametric types
struct Point{T}
    x::T
    y::T
end

# Type annotations
function process(x::Number)::String
    string(x)
end
```

### Buran Types

```
# Types via identity
⟨[circle: radius]⟩ ↤ [identity: "type": "Circle"]
⟨[rectangle: width, height]⟩ ↤ [identity: "type": "Rectangle"]

# Shared supertype (via identity)
⟨[circle: _]⟩ ↤ [identity: "type": "Circle", "category": "Shape"]
⟨[rectangle: _, _]⟩ ↤ [identity: "type": "Rectangle", "category": "Shape"]

# Type checking via identity guards
process {
    ⟨x | "type": "integer"⟩ ↦ to-string(x)
    ⟨x | "type": "real"⟩ ↦ to-string(x)
}
```

Julia has a rich type hierarchy. Buran types are patterns with identity metadata — flatter but flexible.

---

## Arrays vs Lists

Julia's arrays are central to its performance story:

### Julia

```julia
# Arrays
A = [1, 2, 3, 4, 5]
A[1]              # 1-indexed!
A[2:4]            # Slicing
A .+ 1            # Broadcasting
A'                # Transpose

# Matrix
M = [1 2 3; 4 5 6; 7 8 9]
M[1, 2]           # Element access
M * v             # Matrix multiplication

# Array comprehension
[x^2 for x in 1:10]
[x*y for x in 1:3, y in 1:3]
```

### Buran

```
# Lists
[list: 1, 2, 3, 4, 5] ↦ items

# Pattern matching for access (no indexing)
[list: first, ⟨rest⟩] ↦ items    # First element
[list: _, second, ⟨rest⟩] ↦ items  # Second element

# Transform all elements (like broadcasting)
map({ 𝑥 ↦ 𝑥 + 1 }, items)

# Matrix operations require domain
[𝑨 × 𝑩 :: matrix]
[𝑨ᵀ :: matrix]

# No comprehension syntax — use map
map({ 𝑥 ↦ 𝑥² }, [list: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
```

Key differences:

- Buran lists are immutable
- No numeric indexing — use pattern matching
- Matrix operations require `:: matrix` domain
- No array comprehensions — use `map`

---

## Broadcasting

Julia's broadcasting (`.` operator) is powerful:

### Julia

```julia
A = [1, 2, 3, 4]
A .+ 1        # [2, 3, 4, 5]
A .^ 2        # [1, 4, 9, 16]
sin.(A)       # [sin(1), sin(2), sin(3), sin(4)]

# Custom broadcasting
f(x, y) = x + 2y
f.(A, 10)     # [21, 22, 23, 24]
```

### Buran

```
[list: 1, 2, 3, 4] ↦ items

# map is your broadcast
map({ 𝑥 ↦ 𝑥 + 1 }, items)
map({ 𝑥 ↦ 𝑥² }, items)
map({ 𝑥 ↦ sin(𝑥) }, items)

# Multi-argument via pattern
map({ 𝑥 ↦ 𝑥 + 2 × 10 }, items)

# Zip for element-wise with two lists
zip-with({ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 }, list1, list2)
```

Buran's `map` replaces Julia's broadcasting. Less magical syntax, same idea.

---

## Comprehensions

### Julia

```julia
[x^2 for x in 1:10]
[x^2 for x in 1:10 if x % 2 == 0]
[(x, y) for x in 1:3 for y in 1:3]
Dict(x => x^2 for x in 1:5)
```

### Buran

```
# map replaces simple comprehension
map({ 𝑥 ↦ 𝑥² }, [list: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# filter + map for conditional comprehension
filter({ 𝑥 | 𝑥 mod 2 = 0 }, [list: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]) ↦ evens ↦
map({ 𝑥 ↦ 𝑥² }, evens)

# Nested comprehension via nested map
map({ 𝑥 ↦
    map({ 𝑦 ↦ [pair: 𝑥, 𝑦] }, [list: 1, 2, 3])
}, [list: 1, 2, 3]) ↦ nested ↦
flatten(nested)

# Dict comprehension
fold({ acc, 𝑥 ↦ [map: ⟨acc⟩, 𝑥, 𝑥²] }, [map: ], [list: 1, 2, 3, 4, 5])
```

Buran doesn't have comprehension syntax — `map`, `filter`, and `fold` express the same operations.

---

## Mathematical Notation Comparison

This is where both languages shine:

### Julia

```julia
# Unicode allowed in identifiers and custom operators
α = 0.5
Σ = sum
∈(x, s) = x in s

# But operations still use ASCII or defined operators
sqrt(x)
x^2
pi
exp(x)
```

### Buran

```
# Unicode in identifiers AND built-in operators
[0.5] ↦ α
[∑(𝑖=1 to 𝑛) 𝑖]      # Built-in sum notation
[𝑥 ∈ 𝑆]              # Built-in membership

# Mathematical operators
[√𝑥]                  # Square root
[𝑥²]                  # Exponentiation
[𝛑]                   # Pi constant
[𝐞^𝑥]                 # Exponential

# More built-in notation
[⌊𝑥⌋]                 # Floor
[⌈𝑥⌉]                 # Ceiling
[|𝑥|]                 # Absolute value
[∛27]                 # Cube root
[5!]                  # Factorial
```

Buran makes mathematical notation first-class, not just allowed identifiers.

---

## Mutation and Mutability

### Julia

```julia
# Mutable by default
x = [1, 2, 3]
push!(x, 4)       # Mutates x
x[1] = 10         # Mutates x

# Convention: ! means mutation
sort!(x)          # Mutates
sort(x)           # Returns new

# Explicitly immutable
const PI = 3.14159
```

### Buran

```
# Immutable by design
[list: 1, 2, 3] ↦ x

# "Push" creates new list
[list: ⟨x⟩, 4] ↦ new-x

# "Update" creates new list
map({
    [head, ⟨rest⟩] ↦ [10, ⟨rest⟩]    # First element becomes 10
    row ↦ row
}, [list: x]) ↦ updated-x

# Sort returns new list
sort({ 𝑎, 𝑏 ↦ 𝑎 < 𝑏 }, x) ↦ sorted-x
```

Buran is fully immutable — no `!` convention needed because nothing mutates.

---

## Macros

Julia has powerful metaprogramming:

### Julia

```julia
@time sum(1:1000000)

macro sayhello(name)
    return :(println("Hello, ", $name))
end

@sayhello "Julia"

# Expression manipulation
ex = :(a + b * c)
dump(ex)
```

### Buran

**Buran has no macros.**

Where Julia transforms code with macros, Buran:

- Uses patterns for abstraction
- Uses domains for contextual interpretation
- Keeps the language simpler

---

## Error Handling

### Julia

```julia
try
    risky_operation()
catch e
    println("Error: ", e)
finally
    cleanup()
end

# Or check explicitly
result = try
    parse(Int, "abc")
catch
    nothing
end
```

### Buran

```
risky-operation() ↦ {
    [error: e] ↦ ["Error: ", e]
    result ↦ result
}

# Parse with explicit error
parse-int {
    [s | is-numeric(s)] ↦ string-to-int(s)
    s ↦ [error: [string: "cannot parse: ", s]]
}
```

No exceptions in Buran — errors are patterns you match.

---

## Domains: Context for Evaluation

Julia uses multiple dispatch on types. Buran uses domains for evaluation context:

### Julia Type Dispatch

```julia
# Different behavior based on type
*(a::Matrix, b::Matrix) = matrix_multiply(a, b)
*(a::Number, b::Matrix) = scalar_multiply(a, b)
*(a::Quaternion, b::Quaternion) = quaternion_multiply(a, b)
```

### Buran Domains

```
# Different behavior based on domain
[𝐴 × 𝐵 :: matrix]      # Matrix multiplication
[3 × 𝐵 :: matrix]      # Scalar multiply
[𝑞₁ × 𝑞₂ :: quaternion] # Quaternion multiplication

# Available domains
[expr :: complex]       # Complex number operations
[expr :: symbolic]      # Symbolic computation
[expr :: modular]       # Modular arithmetic
[expr :: statistical]   # Statistics operations
```

Domains tell Buran how to interpret expressions — similar to how Julia's type system guides dispatch.

---

## Scientific Computing Comparison

Julia is designed for scientific computing:

### Julia

```julia
using LinearAlgebra

A = rand(3, 3)
eigenvals = eigvals(A)
det_A = det(A)
inv_A = inv(A)

# Differential equations
using DifferentialEquations
prob = ODEProblem(f, u0, tspan)
sol = solve(prob)
```

### Buran

```
# Matrix operations with domain
[eigvals(𝐴) :: matrix]
[det(𝐴) :: matrix]
[𝐴⁻¹ :: matrix]

# Symbolic calculus
[d⁄d𝑥(𝑥²) :: symbolic] ↦ [2𝑥]
[∫ sin(𝑥) d𝑥 :: symbolic] ↦ [−cos(𝑥) + 𝐶]
```

Julia has a mature ecosystem for scientific computing. Buran has notation support but less library infrastructure (as a newer language).

---

## Complete Example: Numerical Methods

### Julia: Newton-Raphson

```julia
function newton(f, f′, x₀; tol=1e-10, maxiter=100)
    x = x₀
    for i in 1:maxiter
        fx = f(x)
        abs(fx) < tol && return x
        x = x - fx / f′(x)
    end
    error("Did not converge")
end

# Find √2: solve x² - 2 = 0
sqrt2 = newton(x -> x^2 - 2, x -> 2x, 1.0)
```

### Buran

```
newton {
    f, f-prime, x₀, tol, maxiter ↦ newton-loop(f, f-prime, x₀, tol, maxiter, [0])
}

newton-loop {
    f, f-prime, 𝑥, tol, maxiter, iter | iter ≥ maxiter ↦ [error: did not converge]
    f, f-prime, 𝑥, tol, maxiter, iter ↦
        f(𝑥) ↦ fx ↦
        {
            [✔] ↦ 𝑥
            [✘] ↦ newton-loop(f, f-prime, [𝑥 − fx ÷ f-prime(𝑥)], tol, maxiter, [iter + 1])
        } ↤ [|fx| < tol]
}

# Find √2
newton({ 𝑥 ↦ 𝑥² − 2 }, { 𝑥 ↦ 2 × 𝑥 }, [1.0], [1e-10], [100])
```

Similar algorithm, different expression style.

---

## Comparison Table

| Julia              | Buran                                |
| ------------------ | ------------------------------------ |
| `f(x) = x^2`       | `f { [x] ↦ [x²] }`                   |
| `x -> x^2`         | `{ x ↦ x² }`                         |
| `do` block         | `{ pattern ↦ result }`               |
| `struct T ... end` | `⟨[t: ...]⟩ ↤ [identity: ...]`       |
| `::Type` dispatch  | Pattern matching                     |
| Multiple dispatch  | Multiple pattern clauses             |
| `A .+ 1`           | `map({ x ↦ x + 1 }, A)`              |
| `[x^2 for x in A]` | `map({ x ↦ x² }, A)`                 |
| `push!(A, x)`      | `[list: ⟨A⟩, x]`                     |
| `try...catch`      | `[error: e] ↦ ...`                   |
| `nothing`          | `[]`                                 |
| `true`/`false`     | `[✔]`/`[✘]`                          |
| `sqrt(x)`          | `[√x]`                               |
| `x^2`              | `[x²]`                               |
| `pi`               | `𝛑`                                  |
| `sum(A)`           | `fold({ acc, x ↦ acc + x }, [0], A)` |

---

## What Julia Does Better

1. **Performance** — JIT compiled, LLVM optimized
2. **Ecosystem** — Extensive scientific packages
3. **Interop** — Call C, Python, R directly
4. **Arrays** — First-class, fast multi-dimensional arrays
5. **Broadcasting** — Elegant element-wise operations
6. **REPL** — Interactive development

---

## What Buran Offers

1. **Mathematical operators** — `√`, `×`, `÷`, `∑`, `∏` as syntax, not identifiers
2. **Pattern matching** — Unified dispatch mechanism
3. **Immutability** — No mutation, no surprises
4. **Simpler type system** — Patterns with identity
5. **No nil/nothing confusion** — `[]` is explicit
6. **Domains** — Mathematical context switching

---

## Summary

Julia and Buran both embrace mathematical notation and structure-based dispatch. Julia does it through multiple dispatch on a rich type system; Buran does it through pattern matching on constructors.

If you're doing scientific computing with performance requirements: Julia's ecosystem is mature.
If you want pure pattern transformation with mathematical notation: Buran is elegant.

Both languages believe code should look like the mathematics it represents.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
