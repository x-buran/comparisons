# Factorial in Buran vs Other Languages

A comparison of how Buran's factorial example stacks up against its ancestral influences and mainstream languages.

## Buran

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

**Characteristics:**

- Pure pattern matching with no keywords (`if`, `return`, `match`)
- Mathematical notation (× for multiplication, − for subtraction, 𝑛 as variable)
- The transformation arrow ↦ makes data flow explicit
- Reads almost like a mathematical definition

---

## Ancestral Languages

### Haskell (1990)

```haskell
factorial 0 = 1
factorial n = n * factorial (n - 1)
```

**Comparison:** Haskell is remarkably similar—pattern matching on function arguments with equation-style definitions. Buran adds the explicit transformation arrow and brackets for patterns, plus Unicode math symbols. Haskell's syntax is slightly more minimal but less visually distinct between pattern and result.

### ML / OCaml (1973)

```ocaml
let rec factorial n =
  match n with
  | 0 -> 1
  | n -> n * factorial (n - 1)
```

**Comparison:** ML requires the `let rec` declaration and explicit `match` keyword. The `->` arrow serves a similar purpose to Buran's `↦`. Buran eliminates the syntactic overhead while keeping the pattern-matching spirit.

### Wolfram Language (1988)

```wolfram
factorial[0] := 1
factorial[n_] := n * factorial[n - 1]
```

**Comparison:** Very close to Buran's approach. Wolfram uses `:=` for delayed assignment and `_` as pattern placeholder. Buran's bracket notation `[pattern] ↦ [result]` is more uniform—Wolfram mixes square brackets for function calls with pattern syntax.

### Refal (1966)

```refal
Factorial {
  0 = 1;
  s.N = <* s.N <Factorial <- s.N 1>>>;
}
```

**Comparison:** As Buran's most direct ancestor, Refal pioneered the pattern-transformation model. However, Refal's syntax is more cryptic: `s.N` denotes a symbol variable, angle brackets for function calls, and prefix notation for arithmetic. Buran modernizes this with familiar infix math and cleaner delimiters.

### Prolog (1972)

```prolog
factorial(0, 1).
factorial(N, F) :-
    N > 0,
    N1 is N - 1,
    factorial(N1, F1),
    F is N * F1.
```

**Comparison:** Prolog requires explicit guards (`N > 0`), intermediate variables, and the awkward `is` operator for arithmetic evaluation. The relational style (two arguments instead of input→output) obscures the computational flow. Buran's directional transformation is clearer.

### Lisp (1958)

```lisp
(defun factorial (n)
  (if (= n 0)
      1
      (* n (factorial (- n 1)))))
```

**Comparison:** Lisp uses conditional branching rather than pattern matching. The fully parenthesized prefix notation is consistent but verbose. Buran's pattern cases are visually separated and read more naturally.

### APL (1966)

```apl
factorial ← {⍵=0: 1 ⋄ ⍵×∇ ⍵-1}
```

**Comparison:** APL shares Buran's embrace of special symbols, but prioritizes extreme terseness. The self-reference symbol `∇` and omega `⍵` for arguments require learning APL's unique vocabulary. Buran opts for readable Unicode (mathematical italic 𝑛) over cryptic glyphs.

---

## Mainstream Languages

### Python

```python
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)
```

**Comparison:** Python is readable but imperative—explicit `if`/`return` control flow rather than declarative pattern matching. The structure obscures the mathematical elegance: you must mentally trace the control flow to understand the function.

### JavaScript

```javascript
function factorial(n) {
    if (n === 0) return 1;
    return n * factorial(n - 1);
}
```

**Comparison:** Nearly identical to Python. The `===` strict equality and semicolons add visual noise. No pattern matching available in the language core.

### C

```c
int factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n - 1);
}
```

**Comparison:** C requires type declarations and uses the same imperative if/return structure. The definition is cluttered with type annotations that don't contribute to understanding the algorithm.

### Java

```java
public static int factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n - 1);
}
```

**Comparison:** Java adds access modifiers (`public static`) and mandatory type annotations. The ceremony-to-logic ratio is high. Pattern matching was only added in recent versions and isn't idiomatic for this use case.

### Rust

```rust
fn factorial(n: u64) -> u64 {
    match n {
        0 => 1,
        n => n * factorial(n - 1),
    }
}
```

**Comparison:** Rust's `match` is closer to Buran's pattern approach. Still requires type annotations and the `fn`/`->` syntax. The `=>` arrow serves a similar role to `↦`.

### Scala

```scala
def factorial(n: Int): Int = n match {
  case 0 => 1
  case n => n * factorial(n - 1)
}
```

**Comparison:** Scala's pattern matching is elegant but still wrapped in `def`/`match`/`case` keywords. Closer to Buran than Python/Java but more verbose than Haskell or Buran.

---

## Analysis

### What Buran Does Well

1. **Mathematical fidelity**: The definition reads like textbook mathematics. A mathematician seeing `[0] ↦ [1]` immediately recognizes "zero maps to one."

2. **No control flow keywords**: No `if`, `match`, `case`, `return`. The patterns *are* the logic.

3. **Visual clarity**: The transformation arrow `↦` and bracketed patterns create clear visual separation between input and output.

4. **Unicode done right**: Unlike APL's cryptic symbols, Buran uses standard mathematical notation (×, −, 𝑛) that mathematicians already know.

5. **Minimal syntax**: Compared to Refal (its direct ancestor), Buran is dramatically cleaner while preserving the pattern-transformation paradigm.

### Trade-offs

1. **Unfamiliar to imperative programmers**: Developers from Python/Java/C backgrounds must shift mental models from "steps to execute" to "patterns to match."

2. **Unicode input**: Typing `↦` and `𝑛` requires keyboard configuration or editor support (though this is increasingly common).

3. **Less explicit than some**: Haskell's equation style is comparably clean; Buran's advantage is more philosophical (everything is pattern transformation) than syntactic.

### The Sweet Spot

Buran occupies an interesting position:

- **More readable than**: APL, Refal, Prolog, Lisp
- **Comparably clean to**: Haskell, Wolfram
- **More declarative than**: Python, C, Java, JavaScript
- **More mathematically native than**: All of the above

The factorial example showcases Buran's core thesis: when pattern transformation is the fundamental operation, even simple functions become elegant declarations rather than procedural recipes.

---

## Line Count Comparison

| Language  | Lines | Non-whitespace chars |
| --------- | ----- | -------------------- |
| **Buran** | 4     | ~65                  |
| Haskell   | 2     | ~45                  |
| Wolfram   | 2     | ~50                  |
| ML/OCaml  | 4     | ~75                  |
| Rust      | 5     | ~70                  |
| Scala     | 4     | ~75                  |
| Python    | 4     | ~70                  |
| C         | 4     | ~75                  |
| Prolog    | 4     | ~95                  |
| Lisp      | 4     | ~75                  |
| Refal     | 4     | ~55                  |
| Java      | 4     | ~85                  |

Haskell and Wolfram win on pure brevity, but Buran's slightly longer form buys explicit pattern delimiters and the transformation arrow that make the data flow unambiguous.
